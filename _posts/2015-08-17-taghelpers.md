---
title:  "태그 헬퍼 (TagHelpers)"
date:   2015-08-17 22:00:00
categories: aspnet5
author: Jake Ryu
permalink: /taghelpers/
---

TagHelpers는 이전 버전의 HtmlHelpers와 비교할 수 있는데 둘 다 HTML을 동적으로 생성한다는 동일한 목표를 갖고 있다. 

HTML을 동적으로 생성해야 하는 상황은 조회 데이터를 출력 한다거나 반복되는 HTML 마크업을 효율적으로 처리하기 위한 경우가 많다. 기능상으로 봤을 때, 웹 폼의 서버 컨트롤과도 유사하다.

두 가지 헬퍼 모두 사용목적과 결과가 같지만 그 사용 방식은 클라이언트 개발자 또는 웹 디자이너의 관점에서 봤을 때 TagHelpers가 훨씬 우아하다. 

<리스트 1> 하이퍼링크를 표현하는 두 가지 방법

[HtmlHelpers]

`@Html.ActionLink("Back Home", "Index", "Home", new { @class = “linkstyle” })`

[TagHelpers]

`<a asp-action="Index" asp-controller="Home" class=”linkstyle”>Back Home</a>`

[결과]

`<a href="/" class=”linkstyle”>Back Home</a>`

리스트 1에서는 동일한 결과를 만드는 두 가지 헬퍼를 비교하고 있다. HtmlHelpers를 사용한 문장은 웹 디자이너들이 직관적으로 이해하기 어렵다는 것이 문제였다. HTML 속성을 설정하기 위해 익명 객체를 사용해야 하고 게다가 C#과 키워드가 겹치는 경우에는 @를 사용해서 escape 처리까지 해야하니 웹 디자이너들에게는 외계어처럼 보였을 것이다.

TagHelpers를 사용하면 AngularJS의 디렉티브를 사용하는 것처럼 asp- 접두어로 시작하는 지시어를 HTML 태그에 직접 삽입할 수 있다. 사용 방식을 보니 태그 헬퍼라는 이름이 꽤 직관적이다. Razor 뷰 엔진은 이런 지시어를 만나면 이를 포함하는 HTML 요소와 직접적으로 연결된 태그 헬퍼가 있는지 조사해서 HTML을 렌더링한다. 

제공되는 태그 헬퍼들은 아래와 같다. 그러나, 사용자 정의 태크 헬퍼도 만들 수 있다는 것을 기억하자.

*	Anchor (하이퍼 링크 생성)
*	Cache (부분 페이지 캐싱 관리)
*	Environment (실행 환경에 기반하여 컨텐트 조정)
*	Form (폼 요소 생성)
*	Input (폼의 입력 요소 생성)
*	Label (라벨 요소 생성)
*	Link (외부 스크립트, CSS 파일등을 참조하는 링크 처리)
*	Option (드롭다운리스트에서 각각의 선택 항목이 대상)
*	Script (스크립트 태그 처리)
*	Select (드롭다운리스트 생성)
*	TextArea (textarea 태그 처리)
*	ValidationMessage (개별 항목에 대한 유효성 검사 에러 메시지 생성)
*	ValidationSummary (유효성 검사의 전체 결과 메시지 생성)

TagHelpers도 MVC를 추가했던 것과 마찬가지로 종속성으로 프로젝트에 추가해야 한다. project.json 파일의 dependencies 영역에 다음 한 줄을 추가하면 된다.

`"Microsoft.AspNet.Mvc.TagHelpers": "6.0.0-beta4"`

asp- 접두어로 시작하는 지시어를 HTML 편집시 쉽게 사용하려면 툴링에 대한 종속성도 추가해야 한다.

`"Microsoft.AspNet.Tooling.Razor": "1.0.0-beta4"`

이제 비주얼스튜디오에서 HTML 태그를 편집하다 'asp-' 를 타입하면 인텔리센스가 지시어를 추천해 줄 것이다.

[![TagHelpers 인텔리센스][1]][1]

<그림 1> Razor 툴링이 제공하는 TagHelpers 인텔리센스

TagHelpers는 뷰에서 선택적으로 사용하는 기술이기 때문에 `@addTagHelper`라는 지시어로  TagHelpers의 사용을 뷰에 명시적으로 선언해야 한다. TagHelpers를 글로벌하게 사용하려면 'Views\Shared\_ViewImports.csthml' 파일에 아래와 같이 선언하면 된다.

`@addTagHelper "*, Microsoft.AspNet.Mvc.TagHelpers"`

TagHelpers를 사용한다고 선언하면서 와일드카드(*)를 사용했다. TagHelpers가 제공하는 모든 태그 기능을 사용한다는 의미이다. 특정 헬퍼를 배제하려면 아래와 같은 문장을 사용할 수 있어 태그 헬퍼들을 선택적으로 구성할 수 있다.

{% highlight c# %}
@removeTagHelper "Microsoft.AspNet.Mvc.TagHelpers.AnchorTagHelper, Microsoft.AspNet.Mvc.TagHelpers"
{% endhighlight %}

####정리####

TagHelpers를 사용하면 깔끔한 HTML을 유지할 수 있어 웹 디자이너와의 협업에 도움이 될 뿐만 아니라 웹 폼 스타일의 사용자 컨트롤도 작성할 수 있기 때문에 개발자들에게 큰 인기를 끌 것 같다. 

유독 태그 헬퍼에 대해 많은 내용을 다루고 있는 Dave Paquette 라는 블로거가 있으니 TagHelpers에 관심이  많다면 그의 [블로그][2]를 참조하자.


[1]: /assets/aspnet5/taghelpers-intellisense.png
[2]: http://www.davepaquette.com/
