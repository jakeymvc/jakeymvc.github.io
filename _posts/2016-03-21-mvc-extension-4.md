---
title:  "MVC 확장 포인트 (4/10) - 테마 지원을 위한 Custom View Engine"
date:   2016-03-21
categories: mvc
tags: extension
author: Jake Ryu
permalink: /mvc-extension-theme
---

커머셜 웹 사이트에서는 매출 증대, 고객 전환률등을 높이기 위해 다양한 시도를 한다. 중요 버튼의 색상과 크기를 변경하는 것, 입력 요소들을 세로 배열에서 가로 배열로 변환하는 등 다양한 시도를 하는데 이를 Conversion Optimization 이라고 한다. 이런 개선의 노력이 항상 좋은 결과로 이어지는 것은 아니므로 이전 버전으로 쉽게 돌아갈 수 있어야 한다.

####테마(Theme) 지원####

간단한 CSS 변경으로도 여러 가지 변화를 관리할 수 있겠지만 화면 구성요소의 위치를 변경하는 것과 같은 큰 변경은 뷰를 직접 수정해야 할 것이다. 만약, 뷰를 직접 수정하는 변경이라면 원래 상태로 되돌리기 위해서 다시 수정해야 하므로 관리의 문제뿐만 아니라 비효율적이고 위험하다. 뷰의 복사본을 작성하여 관리하거나 소스 관리 도구를 통해 운영의 묘를 살릴수도 있겠으나 그 보다 더 좋은 방법은 테마를 지원하는 것이다.

테마를 지원하기로 결정했다면 아래 요건을 만족해야 한다.

1. 테마를 변경하는 것이 쉬워야 한다
2. 단순한 css 변경이 아니라 뷰가 관리의 대상이다
3. 테마에서 뷰를 찾을 수 없다면 기본 테마의 뷰를 사용할 수 있어야 한다 (Fallback system)

####Custom View Engine####

MVC에서 사용할 뷰 엔진을 새로 작성하는 것은 가능한 일이다. 그러나 이미 있는 바퀴를 다시 발명하려고 노력할 필요가 없듯이 기존의 Razor 뷰 엔진을 사용하면서 테마를 사용하도록 확장해 보자. 확장을 위해서는 프레임워크가 뷰를 찾는 메커니즘을 이해하고 그 중간에 간섭하여 테마에 포함된 뷰가 사용되도록 하는 것이다.

아래 그림과 같이 `RazorViewEngine`은 다른 뷰 엔진에 기반하고 있다.

![Razor View Engine Class](/assets/mvc/RazorViewEngineClass.png)

<그림 1> Razor View Engine 클래스의 상속관계

뷰를 찾는 위치가 `VirtualPathProviderViewEngine` 클래스에 속성으로 정의되어 있으므로 해당 속성을 customize함으로써 테마와 연관된 뷰의 위치를 삽입하는 것이 custom view engine을 작성하는 핵심이다.
 
[![Razor Search Location](/assets/mvc/razor-search-location.png)](/assets/mvc/razor-search-location.png)

<표 1> Razor View 엔진의 뷰 경로 탐색
 
Views 폴더 아래에 Themes 라고 모든 테마를 모아 둘 폴더를 새로 만들고 각 테마의 이름으로 폴더를 작성하여 뷰를 위치시킨다. Views 폴더 하단에 보이던 구조 (컨트롤러 명 폴더와 Shared 폴더) 가 그대로 테마 이름 폴더 하단에 위치해야 한다. 이 개념을 코드로 구현한 것이 목록 1이다.

```csharp
public class ThemeViewEngine : RazorViewEngine
{
    public ThemeViewEngine( string activeThemeName)
    {

        ViewLocationFormats = new[]
        {
            "~/Views/Themes/" + activeThemeName + "/{1}/{0}.cshtml" ,
            "~/Views/Themes/" + activeThemeName + "/Shared/{0}.cshtml"
        };

        PartialViewLocationFormats = new[]
        {
            "~/Views/Themes/" + activeThemeName + "/{1}/{0}.cshtml" ,
            "~/Views/Themes/" + activeThemeName + "/Shared/{0}.cshtml"
        };

        AreaViewLocationFormats = new[]
        {
            "~Areas/{2}/Views/Themes/" + activeThemeName + "/{1}/{0}.cshtml" ,
            "~Areas/{2}/Views/Themes/" + activeThemeName + "/Shared/{0}.cshtml"
        };

        AreaPartialViewLocationFormats = new[]
        {
            "~Areas/{2}/Views/Themes/" + activeThemeName + "/{1}/{0}.cshtml" ,
            "~Areas/{2}/Views/Themes/" + activeThemeName + "/Shared/{0}.cshtml"
        };
    }

}

```

<목록 1> ThemeViewEngine

목록 1은 `ViewLocationFormats`을 포함한 네 개의 속성을 테마를 고려하여 재정의하고 있다. 이렇게 작성한 custom view engine은 global.asax 파일의 `Application_Start` 이벤트에서 MVC 프레임워크에 등록해 주어야 한다.

```csharp
protected void Application_Start()
{
    if (!string.IsNullOrEmpty(ConfigurationManager .AppSettings["ActiveTheme"]))
    {
        var activeTheme = ConfigurationManager.AppSettings["ActiveTheme" ];
        ViewEngines.Engines.Insert(0, new ThemeViewEngine (activeTheme));
    };

      //...
    }
```

<목록 2> global.asax

web.config 파일에서 ActiveTheme 라는 키로 테마를 지정하면 그 테마가 custom view engine에 전달된다. 여기서 주의할 점은, MVC 프레임워크는 다수의 뷰 엔진을 사용할 수 있기 때문에 custom view engine을 Insert 메서드로 첫 번째 엔진으로 설정해야 한다. 여러 개의 뷰 엔진을 사용할 수 있는 능력 덕분에 default fallback 뷰 시스템을 구현할 수 있다.

![theme view engine search](/assets/mvc/theme-view-engine-search-1.png)

<그림 2> Theme view engine 뷰 검색 성공

![razor view engine search](/assets/mvc/theme-view-engine-search-2.png)

<그림 3> Razor view engine 뷰 검색 성공 - fallback system

####정리####

간단히 `RazorViewEngine` 클래스를 상속하는 것으로 custome view engine을 준비하고 뷰를 검색하는 경로를 customize하여 새로운 뷰 엔진을 작성해 봤다. 이런 방식으로 MVC에서 테마를 지원할 수 있다. 또한, 테마에 포함되어 있지 않은 뷰에 대한 요청은 다른 뷰 엔진에 의해 MVC 프레임워크 기본 방식으로 처리되므로 테마 때문에 모든 컨트롤러를 위한 뷰를 담지 않아도 된다. 