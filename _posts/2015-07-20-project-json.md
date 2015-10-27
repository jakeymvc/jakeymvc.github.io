---
title:  "project.json"
date:   2015-07-20 10:00:00
categories: aspnet5
author: Jake Ryu
permalink: /project-json/
---

project.json 파일은 ASP.NET 5 프레임워크에서 새로 도입된 프로젝트 설정 파일이다.

ASP.NET 5 프로젝트는 DNX(.NET Execution Environment) 프로젝트라고도 말할 수 있다. DNX는 애플리케이션의 실행을 위해 CLR을 호스트하고 종속성을 관리하며 애플리케이션의 실행지점(Startup 클래스)을 찾아서 실행한다. JSON 포맷의 project.json 이라는 가볍고 단순한 파일이 실행환경에 필요한 모든 정보를 제공하는데 DNX 프로젝트는 이 파일에 의해 정의된다.

ASP.NET 5 애플리케이션을 실행할 때, 비주얼 스튜디오를 사용하지 않고 윈도우 커맨드 창에서 DNX 명령어를 사용할 수도 있는데, 이 때 project.json 파일이 포함된 폴더 경로를 반드시 지정해야 한다. project.json 파일이 프로젝트를 정의한다는 의미를 다시 한번 확인할 수 있는 대목이다.

[ASP.NET 5 솔루션 구조](/aspnet5-solution-structure/)라는 포스팅에서 기존의 프로젝트 파일(.csproj)이 새 프로젝트 파일(.xproj)과 project.json 파일로 대치되었다고 했다. 단순하게 프로젝트 파일의 크기만 비교해 보면 15KB에서 1KB로 줄었으니 그 차이만큼의 내용이 project.json 파일로 옮겨 갔다고 생각할 수 있겠다.

[![프로젝트 속성 비교][1]][1]
<그림 1> ASP.NET 5 프로젝트(상단)와 ASP.NET 4.5 프로젝트(하단)의 속성 비교

그림 1은 ASP.NET 5 프로젝트와 이전 프로젝트의 속성을 비교하고 있다. 전체적인 카테고리의 수도 1/4로 줄었지만 빌드라는 속성, 한 가지만 보더라도 큰 변화를 느낄 수 있다. 

이와 같이 프로젝트 속성 창에서 사라진 모든 속성을 포함하여 project.json 파일에는 수 많은 설정이 있지만 그 중 대표적인 몇 가지만 살펴보기로 한다.

####1. DEPENDENCIES####

NuGet 패키지를 관리하는 섹션이다. 패키지 명과 버전에 대한 인텔리센스를 지원하며 특정 버전을 선택할 수 있어 편리하다. 기존처럼 “Manage NuGet Packages…” 메뉴를 선택해 GUI 툴을 사용할 수도 있지만 툴을 사용하여 변경한 내용도 결국 이 곳에 업데이트 된다. 모든 변경은 실시간으로 반영되는데 변경 내용 저장시 솔루션 탐색기의 References 폴더를 보고 있으면 일시적으로 restoring 하는 순간을 포착할 수 있다.

####2. FRAMEWORKS####

애플리케이션이 실행되는 대상 프레임워크를 명시하는 섹션이다. 아래 코드에서는 Full CLR인 .NET Framework 4.5.1과 ASP.NET 5에서 등장한 CoreCLR을 함께 사용하도록 설정하고 있다.

<리스트 1> 다중 프레임워크 설정

{% highlight json %}
"frameworks": {
    "dnx451": { },
    "dnxcore50": { }
}
{% endhighlight %}

비주얼 스튜디오의 코드 편집창은 그림 2와 같이 대상 프레임워크를 모두 고려하여 친절하게 인텔리센스를 지원한다.

[![다중 프레임워크 인텔리센스][2]][2]

<그림 2> 비주얼 스튜디오의 다중 프레임워크 인텔리센스 지원
 
한쪽 프레임워크에서만 제공되는 메서드를 어쩔 수 없이 사용해야 하는 상황에서는 리스트 2와 같은 구문을 사용해 조건부 컴파일 할 수 있다.

<리스트 2> 조건부 컴파일 구문

{% highlight json %}
#if DNX451
    Debug.Print("This method is not avaliable in .NET Core.");
#endif
{% endhighlight %}

####3. COMMANDS####

명령어 창에서 dnx를 사용하여 애플리케이션을 실행할 때 사용할 수 있는 명령어를 지정하는 섹션이며 dnx를 사용하는 문법은 아래와 같다.

`dnx <project.json 폴더경로> <command>`

project.json 파일에서 명령어를 리스트 3과 같이 구성한 후, 명령어 창을 띄워 프로젝트가 위치한 폴더로 이동했다고 가정하자. 

<리스트 3> project.json 파일의 명령어 설정

{% highlight json %}
"commands": {
    "web": "Microsoft.AspNet.Hosting --server Microsoft.AspNet.Server.WebListener --server.urls http://localhost:5000",
    "kestrel" : "Microsoft.AspNet.Hosting --server Kestrel --server.urls http://localhost:5004"
}
{% endhighlight %}

윈도우 환경이라면 “dnx . web”을 실행해 웹 애플리케이션을 셀프 호스팅할 수 있고, 맥 또는 리눅스 환경이라면 “dnx . kestrel”을 실행하여 kestrel이라는 HTTP 서버를 사용한  셀프 호스팅이 가능하다.     

애플리케이션의 대상 프레임워크 설정은 이전 프로젝트라면 web.config 파일에 선언되어 있었다. 새로운 프로젝트 설정 파일의 등장으로 web.config 파일에 생긴 변화를 다음 포스팅에서 알아보기로 한다.


[1]: /assets/aspnet5/project-property.png
[2]: /assets/aspnet5/intelisense-two-frameworks.png
