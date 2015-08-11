---
title:  "간단한 SSO #2 - 라우팅"
date:   2015-01-25 12:30:00
categories: mvc
author: Jake Ryu
permalink: /sso-routing/
---

MVC 애플리케이션은 `RouteConfig` 클래스에서 라우팅에 대한 모든 것을 관리한다. Global.asax 파일의 `Application_Start` 메서드를 보면 아래 코드와 같이 `RouteConfig` 클래스의 정적 메서드, `RegisterRoutes` 를 호출 하면서 `RouteTable` 의 `Routes` 속성을 전달하는데 이 속성은 `RouteCollection` 클래스의 인스턴스이다.  `RegisterRoute` 메서드에서 진행할 라우트 구성을 위해 컬렉션을 전달하는 시점이므로 컬렉션의 내용은 비어있다. 

{% highlight C# %}
protected void Application_Start()
{
    AreaRegistration.RegisterAllAreas();
    RouteConfig.RegisterRoutes(RouteTable.Routes);
}
{% endhighlight %}

`RegisterRoutes` 메서드에서는 여러 가지 방법으로 라우트를 등록할 수 있는데 가장 손쉬운 방법은 아래와 같이 `RouteCollection`의 확장 메서드인 `MapRoute`를 사용하는 것이다. 라우트를 구별할 때사용하는 이름, URL 패턴(이하 URL) 그리고 세그먼트 변수에 대한 기본값을 주로 정의하지만 제약조건, 네임스페이스까지 상세하게 지정할 수 있도록 6개의 `MapRoute` override가 제공된다. 

{% highlight C# %}
public static void RegisterRoutes(RouteCollection routes)
{
    routes.MapRoute(
        name: "Default",
        url: "{controller}/{action}/{id}",
        defaults: new { controller = "Home", action = "Index", id = UrlParameter.Optional }
    );
}
{% endhighlight %}

한편으로, `RouteCollection`에 대해 잘 이해하고 있다면, `RouteBase`를 상속하는 라우트를 생성하고 컬렉션에 등록하는 수동적인 방법도 있다. 이 방법은 라우트 핸들러까지 함께 지정할 수 있기 때문에 사용자 정의한 라우트 핸들러를 사용한다면 이 방법을 사용해야만 한다. 위 코드에서 처럼 `MapRoute` 메서드에서는 라우트 핸들러를 지정할 수 없는데, MVC 프레임워크가 제공하는 기본 라우트 핸들러 `MvcRouteHandler`가 암시적으로 사용된다. 

{% highlight C# %}
public static void RegisterRoutes(RouteCollection routes)
{  
    Route myRoute = new Route("{controller}/{action}", new MvcRouteHandler());
    routes.Add("MyRoute", myRoute);    
}
{% endhighlight %}

라우트 컬렉션에는 `RouteBase`를 상속하는 클래스만 등록할 수 있다. MVC 프레임워크의 `Route` 클래스는 이 `RouteBase` 클래스를 상속한, 프레임워크에서 미리 정의한 클래스이다. `Route` 클래스는 라우트의 기본 임무인 URL과 HTTP 요청을 비교해서 매치여부를 판단하고 매치라고 판단하는 경우에만 `RouteData`를 반환한다. 이 결과로, HTTP 컨텍스트에 라우트 데이터가 포함되고 MVC 프레임워크는 라우트 찾는 작업을 중단한다. 이런 작업은 라우트 컬렉션에 등록된 모든 라우트에 대해 행해지는데, 매치되지 않는 경우에는 null을 반환함으로써 라우트 컬렉션을 계속 탐색할 수 있는 원리이다. 

라우트가 URL을 분석하고 `RouteData`를 제공하면, 라우트 핸들러는 `RouteData`를 이용해서 컨트롤러와 액션 메서드를 생성하고 (`RouteData`에 컨트롤러명과 액션 메서드명도 있기 때문) 뷰를 렌더해서 최종 응답을 작성하는 후반 작업을 책임진다. 라우트 핸들러는 이렇게 라우팅 시스템과 MVC 프레임워크를 연결하는 중요한 역할을 하기 때문에 라우트의 일부로 함께 정의되어야 한다. <그림 1>을 보면 ASP.NET의 라우팅 처리가 끝나는 시점(matching route entry)에서 (`IRouteHandler`를 구현하는) 사용자 정의 라우트 핸들러가 발견되지 않으면 MVC 프레임워크의 기본 라우트 핸들러인 `MvcRouteHandler`가 사용되는 걸 확인할 수 있다. 

![ASP.NET MVC 요청 처리 파이프라인](/assets/mvc/mvc-request-pipeline.png)

<그림 1> ASP.NET MVC 요청 처리 파이프라인

MSDN [ASP.NET Routing - Class Reference]

위 링크는 MSDN에서 ASP.NET 라우팅에 관련된 클래스만 모아둔 곳이니 참고하자. **[TIP]** MSDN을 링크할 때, 문제가 되는 Azure 서버가 있다고 한다. MSDN 링크로 이동시 502 Server Error가 발생하면 주소를 복사해서새 창에 주소를 붙여넣고 실행하면 접속된다! 

# 사용자 정의 라우팅

Route 클래스와 MvcRouteHandler 클래스를 설명하면서 MVC 프레임워크가 제공하는 기본 개체라고 설명한데에는 사용자 정의도 가능하다는 뜻이 내포되어 있었다. 앞서 라우트와 라우트 핸들러의 역할 분담을 설명했다.

> 라우트가 URL을 분석하고 RouteData를 제공하면, 라우트 핸들러는 RouteData를 이용해서 컨트롤러와 액션 메서드를 생성하고 (RouteData에 컨트롤러명과 액션 메서드명도 있기 때문) 뷰를 렌더해서 최종 응답을 작성하는 후반 작업을 책임진다.

라우트와 라우트 핸들러 중 어디에서 암호화-복호화를 어디에서 구현하는 것이 좋을까?

사용자 ID는 URL을 통해 전달되지만 암호화되어 있어야 하고 컨트롤러의 액션 메서드로 가기전에 바로 복호화할 수 있으면 액션 메서드 입장에서는 추가 작업이 필요없다. 액션 메서드에서 입력 변수를 찾는 장소 중 하나는 `RouteData`이다. 나가는 URL을 만들 때도 `RouteData`의 사용자 ID 값을 암호화한다. 반면, 라우트 핸들러는 데이터를 처리하기 보다는 요청과 응답의 원활한 사이클을 만들기 위한 작업을 한다.

사용자 정의 라우트를 작성해서 문제를 풀어가 보기로 한다. 두 가지 방법으로 구현할 수 있는데 어떤 클래스를 상속할 것인가의 문제이다. `RouteBase`를 상속하여 완전히 새로운 개체를 구현할 것인가, MVC 프레임워크의 기본 라우트인 Route 개체를 상속해서 그 기능을 응용할 것인가.

`RouteBase`를 상속하는 새로운 개체를 만들고자 했을 때, 처음 부딪히는 상황은 요청 URL에 대한 분석이다. "/"를 구분자로 세그먼트 개수를 파악하고 세그먼트의 순서에 따라 URL 패턴에 정의된 내용과 HTTP 요청을 비교해야 하는 것이다. 불행히도 `Route` 클래스는 System.Web.Routing 네임스페이스에 있어서 그 소스도 공개되어 있지 않다. Do NOT reinvent the wheel!

Microsoft가 어떻게 URL을 파싱하고 패턴에 매치시키는지 구현부를 볼 수는 없지만 사용할 수는 있다. 기본 라우트인 `Route`를 상속하여 기본적인 처리를 맡기고 사용자 클래스에서는 암호화-복호화만 처리하기로 한다. 이러고 보니 구현할 양이 훨씬 줄었다.


# 사용자 정의 라우트 구현

`Route` 클래스를 상속하는 `CryptoRoute`라는 클래스를 작성할텐데 주요 기능을 부모 클래스에 의존하고 있으므로 아래와 같이 생성자를 정의한다. URL 패턴, `RouteValueDictionary`, 라우트 핸들러가 필요하다.

{% highlight C# %}
public class CryptoRoute : Route
{
    public CryptoRoute(string url, RouteValueDictionary defaults, IRouteHandler routeHandler)
        : base(url, defaults, routeHandler)
    {
    }
}
{% endhighlight %}

라우트가 되려면 두 개의 메서드를 반드시 재정의해야 하는데 암호화 하는 과정을 먼저 보기 위해 GetVirtualPath 메서드를 먼저 보리고 한다.

{% highlight C# %}
public override VirtualPathData GetVirtualPath(RequestContext requestContext, RouteValueDictionary values)
{
    // Get id from route data
    string id = values["id"] == null ? string.Empty : values["id"].ToString();
    if (!string.IsNullOrWhiteSpace(id))
    {
        // Encrypt and url encode id
        AesCryptography helper = new AesCryptography();
        byte[] encryptedId = helper.EncryptStringToBytes(id);
        values["id"] = HttpServerUtility.UrlTokenEncode(encryptedId);
    }
    return  base.GetVirtualPath(requestContext, values);
}
{% endhighlight %}


예제의 시나리오는, 웹 애플리케이션 전체에 걸쳐 네비게이션할 때, ID가 있다면 암호화 하는 것이다. 라우팅 시스템이 전달해준 `RouteValueDictionary`에서 ID를 찾을 수 없다면 부모 클래스에 이후 과정을 일임한다. `base.GetVirtualPath`는 URL과 라우트 값들이 매치될 때만 나가는 URL을 작성해서 반환하고 그렇지 않으면 null을 반환하여 라우팅 시스템이 다음 라우트를 확인하도록 한다.

id 값이 있는 경우에만 그 값을 암호화한다. `AesCryptography` 는 사용자 클래스이고 구현 내용은 다음 연재글에서 자세하게 설명한다. 일단 암호화된 결과는 바이트 배열로 반환되는데, 이 것을 쿼리스트링으로 전달하기 위해서는 어찌되었건 텍스트 형태로 변경해야 한다.

URL로 표현하는 가장 일반적인 방법은 URL 인코딩(`UrlEncode`)이지만 IIS 동작 방식 때문에 오류 가능성이 있다. URL 인코딩 결과에 `'%2F'` 라는 값이 있을 경우  IIS 가 간섭하여 `'/'` 문자로 디코드 하기 때문인데, 쿼리스트링의 값을 `'/'` 문자로 바꿔버리면 URL 세크먼트가 늘어나는 부작용이 있어 라우팅 시스템을 헷갈리게 한다. 

차선책으로 Base64 인코딩이 있는데 이 방식도 결과 값에 `'/'` 문자를 포함할 수 있기 때문에 사용할 수 없다. 아래 링크에서 문제의 원인과 대안을 설명하고 있다. 

[Base64 Url Encoding](http://brockallen.com/2014/10/17/base64url-encoding/)

다행히도 닷넷 프레임워크의 `HttpServerUtility.UrlTokenEncode` 메서드는 이런 문제를 완벽하게 해결해 주었다. 다만, 이 방식은 ASP.NET 기반의 사이트에서는 완벽한 솔루션이지만 타 기술로 만들어진 사이트와 통신할 때는 `'/'` 문자를 대치하는 원리를 공유해야 할 것이다. 

두 번째로 오버라이드해야 할 메서드는 들어오는 URL을 처리하는 `GetRouteData` 메서드이다. 라우팅 시스템이 `GetVirtualPath` 메서드를 호출할 때는 친절하게도 `RouteData`를 만들어 보내주지만 이번 경우는 Http 컨텍스트의 URL을 참조해서 직접 `RouteData`를 생성해야 한다. 역시 부모 클래스의 메서드를 호출하여 복잡한 작업을 처리한다.

{% highlight C# %}
public override RouteData GetRouteData(HttpContextBase httpContext)
{
    // Get the base class to build the route data
    var routeData = base.GetRouteData(httpContext);
    // Url not matched
    if (routeData == null) return null;
    // All ids are supposed to be encrypted. Decrypt it!
    if (routeData.Values["id"] != System.Web.Mvc.UrlParameter.Optional)
    {
        string encryptedId = (string)routeData.Values["id"];
        byte[] byteId = HttpServerUtility.UrlTokenDecode(encryptedId);
        if (byteId == null) return null;
        AesCryptography helper = new AesCryptography();
        string id = helper.DecryptStringFromBytes(byteId);

        // Modify id value for controller to see it as normal
        routeData.Values["id"] = id;
    }
    return routeData;
}
{% endhighlight %}

`RouteData`를 확보해야 하기 때문에 이번에는 부모 클래스의 `GetRouteData` 메서드를 먼저 호출한다. `null`이 반환되면 매치되지 않는 경우이므로 똑같이 `null`을 리턴하여 라우팅 시스템이 라우트 컬렉션에서 다음 라우트의 `GetRouteData` 메서드를 호출하도록 한다.

id 라는 세그먼트 변수에 아무것도 할당되지 않았다면 라우트 설정에서 기본값으로 지정한 `UrlParameter.Optional` 이 반환된다. id 값이 있는 경우에만 복호화를 진행한다. 먼저, 암호화된 텍스트를 `HttpServerUtility.UrlTokenDecode` 메서드로 디코드해서 바이트 배열을 준비해야 하는데 중간에 이 텍스트 값이 변조되면 디코드를 할 수 없어 `UrlTokenDecode` 메서드는 null을 반환한다. 이런 경우에는 더 이상 작업을 진행할 필요가 없으므로 `null`을 반환한다. 여기서는 URL 매치에도 불구하고 의도적으로 건너뛰기를 했다고 할 수 있다. 이후 라우트에서도 이 요청을 매치하여 처리할 수 없다면 최종적으로 `404 Not Found` 에러가 반환될 것이다.

id 값을 정상적으로 복호화할 수 있다면 `routeData`에 있는 id 값을 덮어쓰면 된다. 이후 컨트롤러의 액션 메서드는 복호화된 원문을 매개변수로 받게 된다. 다만, id가 정수형이라 해도 암호화-복호화 과정에서 plain text로 간주하여 처리하므로 액션메서드에서도 문자열 형식으로 받아야 한다. 필요하다면 형변환은 액션 메서드 내부에서 해야 할 것이다.

# 라우트 등록하기

이렇게 간단하게 두 개의 메서드를 오버라이드해서 `CryptoRoute`라는 새 라우트를 만들었으니 라우팅 컬렉션에 등록할 일만 남았다.

{% highlight C# %}
public static void RegisterRoutes(RouteCollection routes)
{
    var customRoute = new CryptoRoute("{controller}/{action}/{id}",
        new RouteValueDictionary( new { controller = "Home", action = "Index", id = UrlParameter.Optional } ),
        new MvcRouteHandler());
    routes.Add("Default", customRoute);
}
{% endhighlight %}


익숙한 URL 패턴과 기본 라우트 핸들러를 사용해서 `CryptoRoute`의 인스턴스를 라우트 컬렉션에 추가한다.

# 예제 구현

라우팅 시스템이 제대로 동작하는지 알아보기 위해 랜딩 페이지(landing page)와 상세 페이지만 있는 간단한 예제를 작성했다. 먼저, 랜딩 페이지를 살펴보자.

![상세정보 페이지로의 링크가 있는 랜딩 페이지](/assets/mvc/landing-page.png)

<그림 2> 상세정보 페이지로의 링크가 있는 랜딩 페이지

페이지의 뷰 소스는 아래와 같다.

{% highlight html %}
@{
    ViewBag.Title = "Index";
}

<h2>HR Portal</h2>
<div style="margin-top:50px;">
    <ul>
        <li>@Html.ActionLink("View my details", "Detail", "User", new { id = 3 }, null)</li>
    </ul>
</div>
{% endhighlight %}

View my details 라는 링크를 생성하면서 id 3을 수동으로 지정했다. 라우팅 시스템을 통해 나가는 링크가 페이지에 생성되었고 링크에 마우스 포인트를 가져가면 아래와 같이 URL이 작성되었음을 확인할 수 있다.

- http://localhost:51993/User/Detail/**jGM2myydtQEO8yAO4qFcKA2**

이제, 링크를 클릭하여 상세정보 페이지로 이동해 보자.

![복호화된 ID를 보여주는 사용자 상세정보 페이지](/assets/mvc/detail-page.png)

<그림 3> 복호화된 ID를 보여주는 사용자 상세정보 페이지

위의 뷰를 렌더링하는 액션 메서드의 소스를 살펴보자.

{% highlight C# %}
public class UserController : Controller
{
    UserRepository repo = new UserRepository();
    public ActionResult Detail(string id)
    {
        User model = null;
        if (id == null)
            return HttpNotFound();
        // Decode id and convert it to the original type
        int decodedId;
        if(Int32.TryParse(id, out decodedId))
        {
            model = repo.Users.Where(u => u.Id == decodedId).SingleOrDefault();
        }

        if(model == null)
            return HttpNotFound();

        return View(model);
    }
}
{% endhighlight %}
        
        
문자열로 넘어온 id를 정수형으로 바꾸는 작업이 길어보이지만 그 외에 복호화 하는 과정은 없다. 이미 RouteValue에는 원래의 id가 복호화 되었기 때문이다. 아래의 뷰 소스를 보면 모델의 내용을 단순히 출력만 하고 있다.

{% highlight C# %}
@model TamperProof.Models.User
@{
    ViewBag.Title = "Detail";
}
<h2>ID:  @Model.Id</h2>
<h2>@Model.Name</h2>
<h2>@String.Format("{0:C}", Model.Salary)</h2>
{% endhighlight %}

만약에 원래 URL의 ID 세그먼트 변수 부분을 수정하면 어떻게 될까?

- http://localhost:51993/User/Detail/jGM2myydtQEO8yAO4qFcKA2**AddFunnyThings**

`HttpServerUtility.UrlTokenDecode` 메서드는 제대로 인코딩되지 않은 내용을 디코드할 수 없기 때문에 라우트는 'null'을 반환함으로써 자신의 할 일의 넘겨 버린다. 이후 라우트에서도 처리되지 않는다면 아래와 같이 `Not Found` 에러가 반환된다. 사용자 정보를 보는 화면에서는 id가 없는 경우에도 이런 결과를 반환하기 때문에 맥락상 이해할 수 있는 결과이다.
        
![ID를 변형했을 때, 라우트 매치를 찾을 수 없어 Not Found 에러 발생](/assets/mvc/not-found.png)

<그림 4> ID를 변형했을 때, 라우트 매치를 찾을 수 없어 Not Found 에러 발생

다음 연재글에서는 실제 암호화-복호화에 관한 알고리듬을 알아보겠다.

<br />


*연재 글*

- [간단한 SSO #1 - 접근법](/sso-approach/)
- **간단한 SSO #2 - 라우팅**
- [간단한 SSO #3 - 암호화](/sso-encryption/)
- [간단한 SSO #4 - 위조방지](/sso-hmac/)

[ASP.NET Routing - Class Reference]: https://msdn.microsoft.com/en-us/library/cc668201(v=vs.140).aspx#class_reference
