---
title:  "값 공급자 (Value Provider)"
date:   2015-10-02
categories: mvc
tags: value-provider
author: Jake Ryu
permalink: /value-provider
---

입력 값의 유효성을 검사하거나 또는 그 값을 조정하기 위해 이전 글, [모델 바인딩 (Custom Model Binding)](/model-binding)에서 모델 바인딩의 개념과 과정을 알아보고 사용자가 개입하는 방법을 알아봤다. 모델 바인더의 목표가 모델을 제공하는데 있지만 모델을 구성하는 값을 어디서 어떻게 가져오는지는 알지 못한다. HTTP 요청에서 값을 추출하는 일은 값 공급자의 몫이다.

###모델 바인더와 값 공급자###

모델 바인더(MB)와 값 공급자(VP)의 대화

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0; margin-bottom:20px;}
.tg td{font-size: 0.8em;padding:5px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;}
.tg .tg-yw4l{vertical-align:top}
.tg .bold{font-weight:bold;}
</style>
<table class="tg">
  <tr>
    <td class="tg-yw4l bold">MB</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">요새 컨트롤러가 내 덕에 일이 많이 줄었어. 그렇게 생각하지 않니?</td>
  </tr>
  <tr>
    <td class="tg-yw4l bold">VP</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">그렇긴 하지, 우리가 만들어준 데이터를 쓰기만 하면 되잖아.</td>
  </tr>
  <tr>
    <td class="tg-yw4l bold">MB</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">우리? 왜 우리라고 생각하지? 값이든 객체든 컨트롤러 입맛에 맞게 만들어 주는건 난데.</td>
  </tr>
  <tr>
    <td class="tg-yw4l bold">VP</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">그렇게 생각해? 입력 값은 어디서나 오는지 알고 그러는거야?</td>
  </tr>
  <tr>
    <td class="tg-yw4l bold">MB</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">그야 물론, HTTP 요청에서 가져오는 거잖아.</td>
  </tr>
  <tr>
    <td class="tg-yw4l bold">VP</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">원칙적으로 말하면 그렇긴 하지. 그런데 HTTP 요청이 어떻게 구성되어 있나 생각해 보라구. A 라는 값이 폼 바디에 있는지, 쿼리스트링 또는 라우트 데이터에 있는지, 심지어 헤더 또는 쿠키에 박혀 있는 값인지 어떻게 알고 가져올 수 있을까?</td>
  </tr>
  <tr>
    <td class="tg-yw4l bold">MB</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">음... 그런 걸 내가 꼭 알 필요는 없잖아? 팩토리 아저씨에게 물어보면 그만인데.</td>
  </tr>
  <tr>
    <td class="tg-yw4l bold">VP</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">어라, 우리 아빠 이름이 팩토리인데.</td>
  </tr>
  <tr>
    <td class="tg-yw4l bold">MB</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">헉!!!</td>
  </tr>
  <tr>
    <td class="tg-yw4l bold">VP</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">아빠는 우리 형제들한테 항상 퀴즈를 내, A라는 값을 누가 찾을 수 있냐고. 하지만, 기회가 불공평해. 나는 막내라서 답을 찾을 기회가 거의 없어.</td>
  </tr>
  <tr>
    <td class="tg-yw4l bold">MB</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">그게 무슨 뜻이야?</td>
  </tr>
  <tr>
    <td class="tg-yw4l bold">VP</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">먼저, 첫째 형이 폼 바디에 들어 있는 name-value 쌍을 보고 A라는 name을 찾지. 운좋게 찾으면 그걸로 끝이야, 아빠한테 A에 해당하는 값을 알려주고 끝나는 거지.</td>
  </tr>
  <tr>
    <td class="tg-yw4l bold">MB</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">만약 못 찾으면?</td>
  </tr>
  <tr>
    <td class="tg-yw4l bold">VP</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">그럼, 둘째 형 순서가 되지. 둘째 형은 라우트 데이터만 살펴봐. 거기 없으면 셋째 형 순서가 되지.</td>
  </tr>
  <tr>
    <td class="tg-yw4l bold">MB</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">재밌네. 셋째 형은 어디서 값을 찾을까?</td>
  </tr>
  <tr>
    <td class="tg-yw4l bold">VP</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">셋째 형은 쿼리스트링, 나는 파일, 이렇게 서열대로 찾는 곳이 각각 다르다구. 파일까지 오는 일이 거의 없어 아주 슬프다구.</td>
  </tr>
<tr>
    <td class="tg-yw4l bold">MB</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">그럼, 찾는 순서를 바꿀 수는 없는거야?</td>
  </tr>
  <tr>
    <td class="tg-yw4l bold">VP</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">아빠한테 물어봐. ㅋㅋ</td>
  </tr><tr>
    <td class="tg-yw4l bold">MB</td>
    <td class="tg-yw4l"></td>
    <td class="tg-yw4l">내가 어떻게 해볼 수 있을 것 같기도 한데. 음...</td>
  </tr>
</table>

액션 메서드에 매개변수가 정의되어 있으면 모델 바인더가 개입하여 HTTP 요청으로 전달된 값을 매개변수에 매핑한다. 이 과정을 자세히 들여다 보면 HTTP 요청에서 값을 찾는 일과 매개변수 형식의 객체를 만드는 두 가지 일로 구분할 수 있다. 

모델 바인더가 최전방에서 액션 메서드와 함께 동작하기 때문에 그 이면에서 값 공급자가 하는 중요한 일을 놓치기 쉽다. 값 공급자의 존재를 안다고 해도 때로는 사용자 로직을 모델 바인더에 넣어야 할지, 값 공급자로 구현해야 할지 막막할 때가 있다. 

###값 공급자는 언제 사용할까###

모델 바인딩의 동작원리를 알아 보면서 모델 바인더와 값 공급자의 역할을 구분지었다. 값 공급자는 언제 사용하면 유용할까?

값 공급자를 모델 바인더와 구분짓는 가장 큰 요소는 값의 출처를 캡슐화하고 있다는 것이다. 예를 들어, 사용자 정보를 액션 메서드에서 사용하고자 할 때, 그 것을 찾는 위치가 복잡해서 고민이 된다면 또는, 그 위치 때문에 단위 테스트에 지장이 있다면 매개변수로 지정하는 것을 고려해 보자.

아래 목록 1의 코드는 `Controller` 베이스 클래스의 `User` 속성을 통해 사용자명에 접근하고 있다. 


<목록 1> Welcome 액션 메서드

```c#
public ActionResult Welcome()
{
    string username = User.Identity.Name;
    ViewBag.WelcomeMessage = "Welcome " + username;
    return View();
}
```        

`Controller` 클래스의 `User` 속성을 찾아가 보면 목록 2에서처럼 HttpContext에서 정보를 받고 있다.

<목록 2> Controller 클래스의 User 속성

```c#
public System.Security.Principal.IPrincipal User
{
    get
    {
        if (this.HttpContext == null)
        {
            return null;
        }
        return this.HttpContext.User;
    } 
}
```

사용자의 상호 작용없이 Welcome 액션 메서드를 테스트하려면 어떻게 해야 할까? 테스트 메서드에서 컨트롤러를 생성하고 `Welcome` 메서드를 호출하면 `User` 속성에서 당장 null reference 에러가 발생한다. HttpContext에서 사용자 정보를 가져올 수 없기 때문이다. 이를 방지하려면 테스트 메서드에서 HttpContext 객체를 mock 해야 하는데 성가신 일일뿐만 아니라 HttpContext와의 종속성을 피할 수 없다.

값 공급자의 의미를 되새겨 HttpContext에서 사용자 정보를 가져오는 일을 캡슐화 해보자. 사용자 정보를 매개변수로 정의하는 것이다.

<목록 3> 사용자를 매개변수로 추가한 Welcome 액션 메서드

```c#
public ActionResult Welcome(IPrincipal parameterizedUser)
{
    string username = parameterizedUser.Identity.Name;
    ViewBag.WelcomeMessage = "Welcome " + username;

    return View();
}
```

목록 3에서 사용자(IPrincipal)를 매개변수로 받도록 Welcome 액션 메서드를 수정했다. 이제 모델 바인더가 `parameterizedUser` 매개변수를 처리하는 값 공급자를 찾을 수 있도록 사용자 값 공급자와 팩토리를 만들어 보자.

<목록 4> 사용자 정의 값 공급자

```c#
public class IdentityValueProvider : IValueProvider
{
    public bool ContainsPrefix(string prefix)
    {
        return "parameterizedUser".Equals(prefix, StringComparison.OrdinalIgnoreCase);
    }

    public ValueProviderResult GetValue(string key)
    {
        return ContainsPrefix(key) 
            ? new ValueProviderResult(HttpContext.Current.User, 
                HttpContext.Current.User.Identity.Name, 
                System.Globalization.CultureInfo.CurrentCulture)
            : null;
    }
}
```

값 공급자를 만들려면 `IValuePrivider` 인터페이스를 구현해야 한다. 팩토리는 이 인터페이스를 구현하는 클래스들을 차례로 호출하면서 `ContainsPrefix` 메서드를 호출하는데 prefix를 통해 액션 메서드에 정의한 매개변수명(여기서는  parameterizedUser)을 전달한다. 그 때문에 처리여부를 판단하기 위해 매개변수명을 하드코드로 비교하고 있다.

`IValuePrivider` 인터페이스가 정의하는 또 하나의 메서드는 `GetValue` 메서드이다. 이 메서드에서는 팩토리용으로 제공했던 `ContainsPrefix` 메서드를 통해 자신이 값 제공자인지 다시 한번 확인한 후,  HttpContext의 현재 사용자를 반환하고 있다. 반환 형식 `ValueProviderResult` 에 맞게 값을 구성하는데, 두 번째 인자인 AttemptedValue 는 시스템에 의해 오류시  overwrite 될 수 있기 때문에 null 로 할당해도 무방하다. 만약, `GetValue` 메서드가  null을 반환하면 팩토리 내의 다음 값 공급자에게 기회가 돌아간다. 이런 체인 방식은 값을 다른 곳에도 중복 정의할 수 있는 유연함을 주지만 반면에, 팩토리가 호출되는 순서도 신경써야 할 것이다.

팩토리를 정의하는 것은 아주 간단하다. 목록 5와 같이 값 공급자의 인스턴스를 반환하기만 하면 된다.

<목록 5> 사용자 정의 값 공급자를 위한 팩토리

```c#
public class IdentityValueProviderFactory : ValueProviderFactory
{
    public override IValueProvider GetValueProvider(ControllerContext controllerContext)
    {
        return new IdentityValueProvider();
    }
}
```

마지막으로 사용자 정의 팩토리를 MVC 프레임워크에 알려 줄 일만 남았다. 

<목록 6> Global.asax 파일에 팩토리 등록

```c#
public class MvcApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
        AreaRegistration.RegisterAllAreas();
        RouteConfig.RegisterRoutes(RouteTable.Routes);

        ValueProviderFactories.Factories.Add(new IdentityValueProviderFactory());
    }
}
```

 `ValueProviderFactories.Factories` 속성은 값 공급자 팩토리들의 컬렉션이다. 이 곳에 우리가 작성한 팩토리를 등록한다. 만약 팩토리가 사용되는 우선 순위를 높이고 싶다면 `Add` 대신 `Insert` 메서드를 사용해서 인덱스를 지정할 수도 있다.



###단위 테스트###

값 공급자를 사용함으로써 HttpContext를 의존하고 취급해야 하는 부담을 덜었다. 이 것은 또한 테스트 면에서도 대단히 유용하다. 

<목록 7> 단위 테스트

```c#
[TestMethod]
public void TestMethod1()
{
    // Arrange
    var controller = new HomeController();
    GenericPrincipal user = new GenericPrincipal(new GenericIdentity("iamtestuser"), null);

    // Act
    ViewResult result = controller.Welcome(user) as ViewResult;

    // Assert
    Assert.AreEqual(result.ViewData["WelcomeMessage"], "Welcome iamtestuser");
}
```

액션 메서드에 추가적인 매개변수를 정의하는 것이 깔끔해 보이지 않을 수도 있겠지만 테스트 과정에서 보상 받는 듯한 느낌이 든다.

###정리###

HTTP 요청을 탐색하고 필요한 정보를 가져오는데 값 공급자가 큰 역할을 하고 있다. 모델 바인더는 이를 토대로 해서 액션 메서드에 정의한 매개변수의 객체를 제공한다. 

모델 바인더와 값 공급자의 대화에서 값을 탐색하는 장소에 우선 순위가 있다고 했다. 이는 MVC 프레임워크가 제공하는 기본 모델 바인더에 국한된 이야기다. 이 순서를 조정하는 일은 팩토리가 FormValueProvider를 먼저 호출하느냐, QueryStringValueProvider를 먼저 호출하느냐의 문제이므로 팩토리 컬렉션을 어떻게 관리하느냐에 따라 값 공급자 간의 우선 순위를 조정할 수 있는 것이다.

MVC 가 테스트를 염두에 두고 만들어진 프레임워크라는 것을 값 공급자를 통해 다시 한번 깨달았다. 테스트의 중요성을 알면서도 매번 간과하는 것은 MVC 개발자로서 창피한 일인 것 같다 (물론, 내 이야기다).

간단하기는 하지만 매개변수명을 갖고 하드코드로 비교하는 것은 좋은 방법이 아닌 것 같다. 매개변수에 어트리뷰트를 적용해서 특정 값 공급자랑 연결되도록 하는 것이 나은 방법일 것 같다.   

####참고 자료####
[How to create a custom Value Provider for MVC](http://donovanbrown.com/post/2011/09/23/How-to-create-a-custom-Value-Provider-for-MVC)

[What’s the Difference Between a Value Provider and Model Binder?](
http://haacked.com/archive/2011/06/30/whatrsquos-the-difference-between-a-value-provider-and-model-binder.aspx/)
