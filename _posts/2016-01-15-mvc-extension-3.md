---
title:  "MVC 확장 포인트 (3/10) - HTML 헬퍼"
date:   2016-01-15
categories: mvc
tags: extension
author: Jake Ryu
permalink: /mvc-extension-html-helper
---

Custom 객체를 만들거나 헬퍼 메서드를 작성하는 데에는 크게 두 가지 이점이 있다. 첫 째로, 반복되는 내용을 캡슐화하여 코드를 간결하게 유지할 수 있고 두 번째로, 캡슐화된 기능은 쉽게 재사용할 수 있다.

####기본 HTML 헬퍼####

MVC 프레임워크는 Razor 뷰에서 사용하는 유용한 Html 헬퍼들을 제공한다. 이런 헬퍼들을 사용하면 컨트롤러와 액션 메서드의 이름을 사용하여 애플리케이션 내에서 라우팅 시스템에 근간한 안전한 네비게이션할 수 있고, 뷰와 바인딩된 모델 덕에 인텔리센스의 도움으로 모델의 속성을 strong typed 형태로 폼 항목에 사용할 수도 있다. 

![HTML 헬퍼](/assets/mvc/html-helpers.png)

<표 1> Html 헬퍼 예시와 그 혜택

그러나, 기본으로 제공되는 HTML 헬퍼가 많지 않다. 대신 두 가지 방법으로 사용자 HTML 헬퍼를 쉽게 만들 수 있다.

뷰에 직접 정의하는 Inline Razor 헬퍼는 생성과 사용이 직관적이고 쉽지만 다른 뷰에서 사용할 수 없다는 단점이 있다.

![Inline Razor 헬퍼](/assets/mvc/inline-helper.png)

<목록 1> Inline Razor 헬퍼

특정 뷰에서만 사용하는 것이 아니라면 재사용성 때문에 확장 메서드 기법을 이용해 아래와 같이 사용자 HTML 헬퍼를 만드는 방법이 더 보편적이다.

![확장 메서드 방식의 HTML 헬퍼](/assets/mvc/extension-method.png)

<목록 2> 확장 메서드 방식의 HTML 헬퍼

이렇게 정의된 HTML 헬퍼는 어떤 뷰에서든지 아래와 같은 Razor 문장으로 쉽게 사용할 수 있다. 

``` csharp
@Html.DisplayList(data)
```

####부분 뷰 / 자식 액션####

HTML 렌더링을 모듈화하고 재사용성을 높인다는 관점에서 보면 HTML 헬퍼 말고도 부분 뷰 / 자식 액션이 있다. 어떻게 다르고 언제 사용해야 하는지 알아보자.

![부분 뷰, 자식액션과 헬퍼 메서드 비교](/assets/mvc/partial-view-child-action.png)

<표 2> 부분 뷰, 자식액션과 헬퍼 메서드 비교

하나의 HTML 요소를 사용한다면 또는 `<ol>`, `<li>` 처럼 서로 연관된 HTML 요소에 대한 작업이라면 헬퍼 메서드를 사용하자. 다소 복잡한 영역을 대상으로 하거나 컨트롤러의 액션이 사용된다면 부분 뷰 / 자식 액션을 사용해야 한다.

뷰에 비지니스 로직을 두지 않아야 한다는 MVC 패턴의 취지에 맞게 헬퍼 메서드는 HTML 렌더링만 취급해야 할 것이다.

####태그 헬퍼####

MVC 6에서는 태그 헬퍼가 새로 도입되었다. MVC 6에서 HTML 헬퍼를 대치하는 개념이지만 HTML 헬퍼는 별도 설정 없이 사용할 수 있는 반면, 태그 헬퍼는 지시어을 통해 그 사용을 명시해야 하는 추가적인 기능이다.

태그 헬퍼에 대한 개념적인 내용은 이전 글, [태그 헬퍼 (TagHerpers)](/taghelpers/) 에서 알아보자.

HTML 헬퍼에 비해 태그헬퍼는 어떤 장점을 갖고 있을까. 아래 두 개의 소스를 비교해 보자.

![HTML 헬퍼를 사용하는 뷰](/assets/mvc/html-helper-source.png)

<목록 3> HTML 헬퍼를 사용하는 뷰

![태그 헬퍼를 사용하는 뷰](/assets/mvc/tag-helper-source.png)

<목록 4> 태그 헬퍼를 사용하는 뷰

회색으로 하일라이트된 부분이 폼의 입력 항목이다. 모델의 속성을 사용하는 법이 HTML 스러워지고 HTML 소스를 보기가 훨씬 편해졌다.

[Introduction to Tag Helpers](https://docs.asp.net/projects/mvc/en/latest/views/tag-helpers/intro.html#what-are-tag-helpers) 글에 따르면 태그 헬퍼는 HTML 프렌들리하고 풍부한 인텔리센스 지원을 받기에 개발자의 생산성을 높일 수 있다고 한다.

게다가 친절하게도 `ConfirmPassword` 이라는 모델 속성이 label 요소에 사용될 때, 렌더링된 결과에 단어 사이에 자동으로 공백을 삽입해 준다. 대문자로 시작되는 곳에 공백을 삽입해 주는 작은 기능이지만, 이런 작은 배려에 개발자들은 감동 받는다. 

``` csharp
// Taghelper
<label asp-for="ConfirmPassword"></label>

// 렌더링된 HTML
<label for="ConfirmPassword">Confirm Password</label>
```

####정리####

코드를 간결하게 유지하고 재사용성을 높일 수 있는 차원에서 HTML 헬퍼를 알아봤다. 웹 페이지를 모듈화 하기위한 방법으로 부분 뷰와 HTML 헬퍼를 병행해서 사용한다면 좋은 효과를 볼 것이다.

MVC6 에서 소개된 태그 헬퍼와 뷰 컴포넌트는 이전과는 전혀 다른 스타일을 보여주고 있다.

