---
title:  "닷넷의 미래"
date:   2016-05-05
categories: dotnet
tags: dotnet
author: Jake Ryu
permalink: /dotnet-tomorrow
---

Microsoft의 최근 행보를 알고 있는 개발자라면, 닷넷 개발자에게 흥미 진진한 시대가 펼져치고 있다는 것을 체감하고 있을 것이다. 최근 빌드 행사를 통해 닷넷과 C#의 로드맵이 확실해 졌다. Microsoft는 닷넷을 어느 곳에서든지 개발하고 운영할 수 있도록 새로운 플랫폼을 개발해 왔고 기존 플랫폼을 포함하여 모든 플랫폼을 정돈하는 과정에 있다. 이런 변화가 닷넷 개발자에게 어떤 의미를 갖는가?

####프레임워크와 라이브러리####

모든 것이 확연히 좋아졌다고 느끼는 시점이 되기전에는 항상 그렇듯이 뭔가 정책적으로 명료하지 않다는 느낌을 받는다. 최근 Xamarin 추가에 대한 발표는 당연히 환영할 만한 일이지만 닷넷 생태계에서 어떤 역할을 할지 의문을 갖게 한다. 흥미 진진한 시간인 동시에 불분명한 시간이기도 하다.

####현재의 닷넷####

현재 닷넷의 기본 라이브러리는 아래 그림과 같이 세 가지로 요약된다. 각각의 BASE LIBRARY가 지원하는 애플리케이션 모델이 제각기 존재한다. 따라서, BCL 기반에서 동작하는 코드가 MONO 기반에서 동작한다는 보장은 없다.

[![.NET Today](/assets/dotnet/dotnet-today.png)](/assets/dotnet/dotnet-today.png)

<그림 1> 현재의 닷넷 플랫폼들

전체 닷넷 생태계에서 내 코드를 동작시킨다는 것은은 세 개의 BASE LIBRARY - BCL(.NET Framework), .NET Core, Mono Class Library(Xamarin) - 에서 모두 동작해야 한다는 것을 의미한다.

닷넷 내에서 크로스 플랫폼 코드를 작성하고 싶다면 PCL(Portable Class Library) 을 참조하는 것이 현재의 방법이다. PCL은 동일 기능을 제공함에도 불구하고 각자의 구현체를 갖고 있는 다른 BASE LIBRARY 들의 기능을 참조함으로써 공통의 인터페이스를 제공한다. 때문에 PCL 기반위에서 구현한 코드는 닷넷의 다른 플랫폼에서도 동작할 수 있다.

[![PCL](/assets/dotnet/dotnet-today-pcl.png)](/assets/dotnet/dotnet-today-pcl.png)

<그림 2> PCL의 참조 구현

PCL 이 제공하는 기능은 닷넷 생태계의 교집합적인 기능이므로 Full .NET 프레임워크에 비해서는 제한적일 수 밖에 없는 단점이 존재한다. 또한, PCL이 각각 다른 플랫폼에서 어떻게 동작하는지가 명료하지 않아서 참조한 기능이 의도하지 않게 플랫폼에 따라 다르게 동작할 수 있는 위험도 있다. 

이런 사항을 염두에 두어야 한다는 것은 개발자에게는 부담이 될 수밖에 없고 결국에는 3 + 1 개의 플랫폼을 전부 알아야 한다. 플랫폼 입장에서도 불만이 있다. PCL에 추가된 새 기능을 지원해야 할 때, 각 플랫폼마다 사정도 다르고 복잡도도 다를테니 구현 기간에 차이가 생길 수 밖에 없고 이는 부담으로 이어진다. 
 
####닷넷의 미래####

다행이도 Microsoft는 이 문제를 인식하고 명확한 비전을 수립했다. PCL 방식을 버리고 플랫폼간에 공통 API를 지원하는 계약(CONTRACT) 방식으로 전환한다는 전략이다. 교집합 성격의 PCL이 합집합 성격의 계약으로 바뀌고 이 계약을 지키는 플랫폼과 그렇지 못한 플랫폼을 개발자에게 알려 준다.

이 새로운 계약기반 API를 .NET Standard Library라고 한다. NET Standard Library 는 구현 차원에서 두 가지 성격이 공존하는데 크로스 플랫폼적인 Full Implementation과 플랫폼 종속적인 Reference Implementation(기존의 PCL 방식)의 조합이다.

[![.NET Future](/assets/dotnet/dotnet-future.png)](/assets/dotnet/dotnet-future.png) 

<그림 3> 닷넷의 혁신적인 미래상

용어에 조금 차이가 있기는 하지만 아래 표의 .NET Platform Standard는 .NET Standard Library 를 의미하는 것 같다. 라이브러리가 제공하는 범위가 지속적으로 변경될 것이므로 버전으로 그 차이를 구분하고 있다.



아래 표를 보자면, Windows Phone 8.1은 .NET Platform Standard 1.2 이후에 추가된 API는 지원하지 않는다는 것을 알 수 있다. 또한, .NET Core 1.0과 .NET Framework 4.6.2 는 플랫폼 표준을 최초로 지원하기 시작했으며 그 플랫폼 표준의 버전은 1.5이다.

[![.NET Platform Standard](/assets/dotnet/dotnet-platform-standard.png)](/assets/dotnet/dotnet-platform-standard.png)

<표 1> [닷넷 플랫폼 표준](https://github.com/dotnet/corefx/blob/master/Documentation/architecture/net-platform-standard.md)

.NET Standard Library 의 API는 지속적으로 증가할 텐데 이 것을 지속적으로 구현하여 따라 잡는 플랫폼과 그렇지 않은 플랫폼이 구분된다. 특정 시점에서 보면 구현에 있어 뒤쳐지는 플랫폼이 있겠지만 shipping 기간의 차이가 있을 뿐, 최종적으로는 구현이 되어야 닷넷 표준 라이브러리라는 의미가 빛을 발할 것이다. 

####마무리####

<그림 3>에서 볼 수 있듯이 .NET Standard Library는 App Model, Base Library 그리고 Tooling을 decouple하고 있다. Decoupled 된 시나리오는 코드 재사용성을 훨씬 높혀주고 다른 플랫폼 때문에 익혀야할 학습 곡선을 훨씬 낮추어 준다. 결국, 닷넷 개발자는 어떤 것이든(모바일 장치에서 동작하는 앱이던, 데스크탑 앱이던, 웹 앱이던) 자신의 프로그래밍 경험을 일관되게 유지하면서 다른 종류의 앱을 개발할 수 있게 될 것이다.



