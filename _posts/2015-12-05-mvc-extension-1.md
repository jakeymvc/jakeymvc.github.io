---
title:  "MVC 확장 포인트 (1/10) - Action Result"
date:   2015-12-05
categories: mvc
tags: 
author: Jake Ryu
permalink: /mvc-extension-action-result
---

MVC 애플리케이션을 다루다 보면 확장이 쉽다는 것을 알 수 있다. 단위 테스트를 염두에 두고 프레임워크 자체가 모듈화되었기 때문에 얻는 잇점이라고 할 수 있다. 좋은 구조로 설계하고 구현하는 것은 시간과 정성이 필요한 일이지만 여러 가지로 그 노력에 대한 보상이 돌아온다.

앞으로 MVC의 10가지 확장 포인트를 정리할 계획이다. 그 첫번째로 Custom Action Result를 살펴보자.

###XMLResult###

목록 1은 리마인드 이메일을 보내기 위해 그 대상을 추출하는 액션이다. 이메일 서비스와 XML 포맷으로 통신하기 때문에 이메일 대상을 아예 XML로 받고자 한다. 

``` csharp
public ActionResult GetApplicantsForReminders()
{
    var applicants = _context.Applicants.ToList();
    var vmApplicants = new List<ApplicantVM>();
    foreach(var app in applicants)
    {
        vmApplicants.Add(Mapper.Map<ApplicantVM>(app));
    }

    return new XMLResult(vmApplicants);
}
```

<목록 1> 이메일 대상자를 XML 형식으로 반환하는 액션

Entity Framework의 데이터베이스 컨텍스트를 통해 데이터를 가져오고 AutoMapper를 통해 뷰모델로 변환하는 것 외에는 특별한 것이 없다. 그 결과를 XML을 반환하고 있다는 것까지 코드로 알 수 있다. 

컨트롤러의 코드는 이렇게 단순해야 한다. 어떤 모델을 사용해야 할지 결정하고 해당 모델을 돌려 주면된다. 

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<ArrayOfApplicantVM>
    <ApplicantVM>
        <FirstName>test</FirstName>
        <LastName>test</LastName>
        <Dob>2015-11-02T00:00:00</Dob>
        <Phone>54343</Phone>
        <Email>asdfs</Email>
        <MaritalStatus>Single</MaritalStatus>
        <HighestEducation>1</HighestEducation>
        <LicenseStatus>Valid</LicenseStatus>
        <YearsLicensed>1</YearsLicensed>
    </ApplicantVM>
</ArrayOfApplicantVM>
```
<목록 2> 액션의 결과로 생성된 XML


목록 2는 액션의 결과물이다. `ApplicationVM`을 액션내에서 XML로 시리얼라이즈한다면 코드가 한참이나 늘어날 테고 재사용하지도 못할 것이다. `XMLResult` 클래스가 어떻게 구현되었는지 목록 3에서 알아보자.

``` csharp
using System;
using System.Linq;
using System.Web.Mvc;
using System.Xml.Serialization;

public class XMLResult : ActionResult
{
    private object _data;

    public XMLResult(object data)
    {
        _data = data;
    }

    public override void ExecuteResult(ControllerContext context)
    {
        XmlSerializer serializer = new XmlSerializer(_data.GetType());
        var response = context.HttpContext.Response;
        response.ContentType = "text/xml";
        serializer.Serialize(response.Output, _data);
    }
}
```

<목록 3> XMLResult - Custom Action Result

Custom Action Result를 구현하기 위해서는 `ActionResult` 클래스를 상속해야 하는데, 이 클래스는 `ExecuteResult` 메서드를 abstract 로 정의하고 있어 자식 클래스에서 구현하도록 하고 있다. 데이터는 생성자를 통해 받았고 `ExcuteResult` 메서드에서는 닷넷 `XmlSerializer`를 사용해 HTTP Response를 작성했다. 군더더기가 없다. 

####커맨드 패턴####

전체 과정을 보면 컨트롤러에서는 어떤 행동(반환값)이 필요한지 결정하여 그 행동을 구현한 객체에 실행을 위임하고 있다. `ActionResult` 클래스가 갖는 중요한 의미는 `ExecuteResult` 메서드를 강제화하여 프레임워크에서 호출될 수 있도록 할 뿐이다. 

이렇게 개별 행동을 하나의 명령 객체로 설계하면 명령을 결정하는 부분과 명령을 실행하는 부분을 분리할 수 있어 명령 추가가 쉽다. `XMLResult`를 추가하는 것이 이렇게 쉬운 이유는 `ActionResult`가 커맨드 패턴을 따른 덕이다.   

###CSVResult###
이번에는 데이터 교환용으로 많이 사용하고 엑셀에서 열어볼 수 있는 csv 포맷을 반환하는 Custom Action Result를 작성해 보자.  


``` csharp
public class CSVResult : FileResult
{
    private IEnumerable _data;

    public CSVResult(IEnumerable data, string fileName) : base("text/csv")
    {
        _data = data;
        FileDownloadName = fileName;
    }

    protected override void WriteFile(HttpResponseBase response)
    {
        var builder = new StringBuilder();
        var stringWriter = new StringWriter(builder);

        foreach (var item in _data)
        {
            var properties = item.GetType().GetProperties();
            foreach (var prop in properties)
            {
                stringWriter.Write(GetValue(item, prop.Name));
                stringWriter.Write(", ");
            }
            stringWriter.WriteLine();
        }

        response.Write(builder);
    }

    public static string GetValue(object item, string propName)
    {
        return item.GetType().GetProperty(propName).GetValue(item, null).ToString() ?? "";
    }
}
```

<목록 4> FileResult를 상속하는 CSVResult

이전과 달리 `CSVResult`는 `FileResult`를 상속하고 있다. `FileResult`는 MVC 프레임워크가 기본적으로 지원하는 Action Result 중의 하나이다. `FileResult`의 구현과 그 구조에 대해서는 [Zip 파일 반환하는 ActionResult 만들기](/action-result-to-return-zip-file/) 라는 이전 포스팅을 참조하자.

파일을 다루는 Action Result가 이미 있기 때문에 이 것을 상속함으로써 안정적으로 파일을 처리할 수 있게 되었고 새로운 포맷에만 집중하여 쉽게 Custom Action Result를 추가할 수 있다. 액션의 반환 형식이 파일이라면 FileResult를 활용하는 것을 항상 최우선적으로 생각해야 할 것이다.

`CSVResult`를 구현하면서 몇 가지 특징적인 면을 아래에 정리했다.

* 생성자에서 content type을 부모 클래스 생성자에 전달하고 있다. `FileResult`에는 parameterless 생성자가 없다.
* `FileResult`는 `ExecuteResult` 메서드에서 `WriteFile`을 호출하도록 구현되어 있다. `WriteFile`은 abstract로 정의되어 있어 자식 클래스에서 구현해야 하는데 실제로 파일의 내용을 생성해서 Response로 반환하는 역할을 한다.
* 데이터를 object로 받기 때문에 리플렉션 기법으로 속성 값을 추출한다.

`CSVResult`를 사용하는 예제는 목록 5에서 볼 수 있다.

``` csharp
public ActionResult GetQuotesCSV()
{
    var applicants = _context.Applicants.ToList();
    var mappedApplicants = new List<ApplicantVM>();
    foreach (var app in applicants)
    {
        mappedApplicants.Add(Mapper.Map<ApplicantVM>(app));
    }

    return new CSVResult(mappedApplicants, "TestCSV");
}
```

<목록 5> CSVResult를 사용하는 액션

###Built-in Action Result###

목록 6는 MVC 프레임워크에서 지원하는 Action Result와 이에 대응하는 헬퍼 메서드를 보여주고 있다.

* ViewResult  -  View
* PartialViewResult  -  PartialView
* RedirectToRouteResult  -  RedirecToAction, RedirectToActionPermanent, RedirectToRoute, RedirectToRoutePermanent
* RedirectResult - Redirect
* ContentResult - RedirectPermanent, Content
* FileResult - File
* JsonResult - Json
* JavaScriptResult - JavaScript
* HttpUnauthorizedResult - None
* HttpNotFoundResult - HttpNotFound
* HttpStatusCodeResult - None
* EmptyResult - None

<목록 6> 내장된 ActionResult 형식

여기서 헬퍼 메서드를 소개하는 이유는 실제로 컨트롤러 액션에서는 주로 이 것을 사용하기 때문이다. 헬퍼 메서드를 사용할 때 궁극적으로는 ActionResult의 인스턴스가 반환되는 것을 이해하면 목록 1과 목록 5에서 `new`를 사용하는 것이 더 이상 부자연스럽지 않을 것이다.

목록 7은 View와 관련된 헬퍼 메서드들인데 매개변수의 조합에 따른 여러 개의 오버라이드가 있고 모두 마지막 메서드에 의존하고 있는 걸 확인할 수 있다. 마지막 메서드는 예상대로 ViewResult의 인스턴스를 반환한다. 

``` csharp
protected internal ViewResult View()
{
    return View(viewName: null, masterName: null, model: null);
}

protected internal ViewResult View(object model)
{
    return View(null /* viewName */, null /* masterName */, model);
}

protected internal ViewResult View(string viewName)
{
    return View(viewName, masterName: null, model: null);
}

protected internal ViewResult View(string viewName, string masterName)
{
    return View(viewName, masterName, null /* model */);
}

protected internal ViewResult View(string viewName, object model)
{
    return View(viewName, null /* masterName */, model);
}

protected internal virtual ViewResult View(string viewName, string masterName, object model)
{
    if (model != null)
    {
        ViewData.Model = model;
    }

    return new ViewResult
    {
        ViewName = viewName,
        MasterName = masterName,
        ViewData = ViewData,
        TempData = TempData,
        ViewEngineCollection = ViewEngineCollection
    };
}
```

<목록 7> Controller에서 구현하고 있는 View 메서드

###정리###

MVC의 확장 포인트에 대한 첫 번째 주제로 Custom Action Result에 대해 알아봤다. 

* 객체를 XML 형식으로 시리얼라이즈 하는 예제
* 객체를 CSV 형식의 파일로 다운로드 하는 예제

를 코드와 함께 살펴봤다. 더불어, 명령을 객체화 한다는 커맨드 패턴도 알게 되었다.