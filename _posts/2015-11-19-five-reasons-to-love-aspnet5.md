---
title:  "ASP.NET 5가 좋은 다섯 가지 이유"
date:   2015-11-19
categories: aspnet5
tags: 
author: Jake Ryu
permalink: /five-reasons-to-love-aspnet5
---

ASP.NET 5 RC1 이 예정대로 2015년 11월에 Connect() 행사 시점에 맞추어 배포되었다. RC 버전에 들어서면 사용자 피드백을 통한 안정화 단계에 접어든 것이므로 기능적인 추가와 변경이 없다고 한다. 베타버전이라 아직 망설였다면 이제 슬슬 시작해보는 것은 어떨까. ASP.NET 5가 좋은 이유를 다섯 가지로 정리해 봤다.

###1. 오픈소스###

나는 오픈소스를 좋아한다. 누군가 아이디어를 구체화해서 다른 사람과 공유한다는 것이 좋고 집단 지성이 만들어낸 훌륭한 코드를 볼 수 있다는 것은 굉장한 매력이다. 

GitHub에는 ASP.NET과 관련된 [ASP.NET 리포지토리](https://github.com/aspnet/)와 닷넷 코어처럼 프레임워크와 관련된 [.NET Fouddation 리포지토리](https://github.com/dotnet/)가 공개되어 있다. 며칠 전 마이크로소프트의 Connect() 행사에서는 [비주얼 스튜디오 코드의 리포지토리](https://github.com/Microsoft/vscode)도 키노트중에 공개되었다. 

마이크로소프트의 기술로 개발하고 있다면 오프소스를 프로젝트에 함께 설치하여 프레임워크에서 동작하는 부분까지 디버깅할 수 있다. 또한, 확장 모듈과 사용자 정의 모듈을 만들때 프레임워크에 구현된 방식을 참고할 수도 있어 도움이 된다. 

마이크로소프트는 TypeScript를 공개했고 구글은 TypeScript를 기반으로 AngularJS 2를 작성하고 있으며 역시 공개하고 있다. 당신이 마이크로소프트나 구글에 취업하고 싶다면 이런 오픈소스 프로젝트에 이슈를 등록하고 Pull Request까지 도전해 보자. 당신의 열정과 실력을 증명할 수 있는 가장 좋은 방법일 것이다.

###2. IIS에 목매지 않는다, Kestrel###

IIS는 ASP.NET 웹 애플리케이션을 호스팅하는 성숙한(matured) 플랫폼이다. 반면, 규모가 작은 웹 애플리케이션의 경우에는 필요보다 과다한 기능을 제공하는 것이 문제되어 왔다. 그리고, ASP.NET과의 지나친 결속력은 성능면에서 이득일수도 있었으나 애플리케이션의 독립성을 저해하는 요소로 평가되어 왔다. 단적인 예로, [web.config 파일의 상속 구조](https://msdn.microsoft.com/en-us/library/ms178685.aspx)만 봐도 이 둘의 관계가 얼마나 돈독한지 알 수 있다. 

웹 서버와 웹 애플리케이션이 서로 종속성을 갖는 것에 대해 무엇이 문제인지 답하기란 쉽지않다. 리눅스 서버를 갖고 있는데 ASP.NET과 C#을 하고 싶다면 이건 개인의 욕심인가, 기술의 장벽인가. 되야 맞는건가, 안되도 이해할 수 있는 문제인가. 주어진 상황에 따라 문제 삼으면 문제요, 아니면 아닌 것이다.

그래도 호스팅 환경에 대한 종속성이 문제라고 생각했기 때문에 [OWIN](http://owin.org/)이란 표준이 나오지 않았나 싶다. OWIN은 닷넷 웹 애플리케이션과 웹 서버가 대화하는 방식을 표준화한 규격이고 이 규격대로 닷넷 애플리케이션을 호스팅할 수 있는 서버를 작성할 수 있다.   

[Katana 프로젝트](http://katanaproject.codeplex.com/documentation)는 OWIN 표준을 구현한 새로운 웹 애플리케이션 아키텍쳐다. [Katana의 탄생배경](http://www.asp.net/aspnet/overview/owin-and-katana/an-overview-of-project-katana)을 읽어 보면, 특히 그 아키텍쳐와 각 컴포넌트의 정의를 이해하는 것이 지금의 ASP.NET을 이해하는데 많은 도움이 된다.

![Katana Architecture](/assets/aspnet5/katana-architecture.png)

<그림 1> Katana 아키텍쳐

일반적으로 말하는 웹 서버는 Host와 Server의 역할을 같이 한다. 그와 비교해서 셀프 호스팅을 예로 들면, 커맨드 프로그램을 호스트로, 소켓을 열고 HTTP 요청을 기다리는 리스너(listener)를 서버로 생각할 수 있다. 이제부터 서버라는 표현이 나오면 HTTP 요청을 잡아서 사용자가 지정한 OWIN 파이프라인으로 보내는 컴포넌트라고 생각하자.

Katana가 있음에도 불구하고 마이크로소프트는 왜 [Kestrel](https://github.com/aspnet/Home/wiki/Servers)이라는 서버를 ASP.NET 5 프로젝트에 포함시켜야 했을까? 크로스플랫폼을 지원하기 위해서 코드 작성의 호환성과 함께 서버도 필요했기 때문이다. 닷넷 코드가 CoreCLR을 통해 서로 다른 운영체제에서 동일하게 해석된다 하더라도 ASP.NET 5는 웹 애플리케이션이니 만큼 호스트와 서버 없이는 동작할 수 없다. 따라서, Katana와 Kestrel이 구분되는 한 가지 요소는 크로스플랫폼이다.

Kestrel(그리고, WebListener)를 사용하면 셀프 호스팅이 가능한데 이는 웹 애플리케이션에 날개를 달아주는 격이다. ASP.NET 5 애플리케이션은 라즈베이 파이에서도 운영할 수 있다.

###3. 모듈화###

>현재의 모놀리식 닷넷 프레임워크도 유지되지만 동시에 점차 NuGet 패키지로 모듈화되어 CoreFX 라이브러리를 구성해갈 것이다... 이렇게 애플리케이션에 필요한 기능만을 가져다 사용하는 pay-as-you-play 모델이라 가볍고 배포상의 이점이 있다...  - [닷넷코어](/net-core/) 중에서 

덩치 큰 닷넷 프레임워크가 잘게 모듈화되고 있는 중이다(현재 진행형). 각각의 기능들이 NuGet에서 다운로드할 수 있는 개별 패키지로 분리되면서 버그 수정 및 업데이트 주기도 빨라진다. 이를 CoreFX 라이브러리라고 하는데 여러 런타임(.NET Nateive, CoreCLR)과 여러 애플리케이션 모델(Windows Store, ASP.NET 5)에서 공통으로 사용되므로 다른 종류의 애플리케이션을 작성하는 것이 완전히 새로운 일은 아닐 것이다.

###4. 벤치마크는 계속 된다, 빠른 성능###

ASP.NET 팀은 성능 개선에 큰 비중을 두어왔다. 다른 플랫폼과 지속적인 [벤치마크](https://github.com/aspnet/benchmarks#plain-text-with-http-pipelining)를 통해 NodeJS 보다 빠르게 만들겠다는 확고한 의지를 실천했다. 재밌는 점은 Kestrel과 NodeJS 모두 [libuv](https://github.com/libuv/libuv) 라는 비동기 I/O 기술을 지원하는 라이브러리에 기반하고 있다는 것이다. 

[![benchmark](/assets/aspnet5/benchmark.png)](https://github.com/aspnet/benchmarks#plain-text-with-http-pipelining)

<그림 2> Plain text를 HTTP 파이프라인에서 벤치마크한 결과

HTTP 파이프라인을 가볍게 가져갈 수 있었던 데는 미들웨어도 한 몫했다. 미들웨어는 복잡하고 어려웠던 OWIN을 좀더 쉽게 접근할 수 있도록 추상화하여 개발자가 요청 파이프라인를 통제할 수 있도록 해주고 파이프라인을 완전히 비워 버렸다.

>ASP.NET에서는 System.Web 을 사용하여 http 요청을 처리해 왔고 데이터 운반책인 HttpContext.Current는 32KB의 메모리를 차지했다. 이 것은 요청하는 웹 페이지의 복잡도에 상관없이 하나의 세션당 무조건 필요한 메모리 크기이므로 다소 불합리하다고 할 수 있었다. ASP.NET 5의 http 요청 처리는 Node.js 스타일의 raw socket 과 유사하게 변경되었고 단지 3KB의 메모리만을 차지한다. 이렇게 90% 감소된 메모리 사용은 로딩 타임을 줄여 사용자 요청에 빠르게 응대하고 동시에 더 많은 요청을 처리할 수 있게 해준다. 이런 성능 향상의 배경에 미들웨어가 있다.  - [ASP.NET 5 미들웨어](/middleware/) 중에서 

    
###5. 클라우드 최적화###

[새로운 Configuration 모델](/bye-web-config/)은 아주 유연하다. Json 타입의 설정파일과 환경 변수를 활용하는 것이 눈에 띄는 변화이다. 특히 민감한 정보를 하드코딩으로 사용하다가 GitHub 같은 곳에서 노출하지 않으려면 환경변수는 좋은 대안이다.

호스팅 환경에 대한 독립성과 유연한 애플리케이션 설정 모델은 클라우드에서 애플리케이션을 운영하기 위한 준비라고 할 수 있다. 이와 더불어 ASP.NET 5 애플리케이션은 모듈화된 구조덕에 시장에서 검증된 제품 또는 기술들로 특정 기능(예: 캐싱)을 대치할 수 있다. 클라우드 스케일에 걸맞는 애플리케이션의 기능과 성능을 구현할 수 있는 것이다. 

###마치며###

ASP.NET 5의 변화된 내용들을 보고 있자면 성숙하고도 당당해진 느낌을 받는다.

* 마이크로소프트는 개발 프레임워크 뿐만 아니라 개발 도구들도 개방하고 있다
* [live.asp.net](https://live.asp.net/)을 통해 개발자들과 대화하고 피드백을 제품에 반영한다
* 다른 커뮤니티들이 잘하는 것을 인정하고 도입한다
- [다큐멘테이션](http://docs.asp.net/)도 잘 정리되어 있고 개발자들의 컨트리뷰트도 받는다
* 맥북에서 마이크로소프트 기술을 사용해서 안드로이드르 앱을 개발할 수 있을 정도로 개발자들의 다양함을 인정한다

ASP.NET 5는 모던 웹 애플리케이션을 개발하는데 첫 번째로 고려해 볼만한 기술이 될 것같다.

####[참고자료]####
[Mastering ASP.NET 5 without growing a beard](http://blogs.msdn.com/b/tess/archive/2015/11/12/mastering-asp-net-5-without-growing-a-beard.aspx)
