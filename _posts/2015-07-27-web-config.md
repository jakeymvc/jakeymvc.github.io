---
title:  "Bye web.config"
date:   2015-07-27 12:00:00
categories: aspnet5
author: Jake Ryu
permalink: /bye-web-config/
---

ASP.NET 프레임워크의 변화에 따라 web.config의 설정 내용도 조금씩 변해 왔지만 개발자들에게 web.config 파일의 첫 번째 용도는 appSettings 섹션에서 애플리케이션에 필요한 정보를 관리하는 것이었다. 또한, web.config 파일은 프로젝트 수준에서의 대상 프레임워크 버전, 종속 어셈블리 목록 등을 관리하고 있어 여러 내용을 모두 포함하는 단일 설정 파일로 사용되어 왔다. 

ASP.NET 5와 함께 web.config 파일에 있던 프로젝트 관련 설정은 project.json 파일로 이동하였고, 개발자를 위해서는 Key-Value의 쌍을 관리하는 깔끔한 설정(configuration) 모델이 새롭게 도입되었다. 이렇게 web.config 파일은 그 역할을 나눠주고 이제 역사의 뒤안길로 사라졌다. 

새 설정 모델은 외부 소스로서 XML, JSON, INI, 세 가지의 파일 양식을 기본적으로 지원하고 인메모리(in-memory) 방식과 사용자 정의 설정 제공자(configuration provider)도 작성할 수 있는 유연한 모델이다. 보다 중요한 변화로, 운영환경 자체에 정의된 환경 변수(environment variables)를 사용할 수 있게 되어 프로젝트 외부에서도 설정이 가능해 졌다. 외부 소스에 의존하지 않는 인메모리 방식의 설정은 리스트 1와 같다.

<리스트 1> 인메모리 방식의 설정

{% highlight C# %}
var config = new Configuration();
config.Add(new MemoryConfigurationSource());
config.Set("somekey", "somevalue");

string setting = config.Get("somekey); 	// returns "somevalue"
string setting2 = config["somekey"]; 	// also returns "somevalue"
{% endhighlight %}

파일을 사용하는 방식은 웹사이트(Web Site) 템플릿에 의해 생성되는 config.json 파일이 좋은 예다. 

<리스트 2> config.json 파일. JSON 포맷의 다층 구조가 읽기 편하다.

{% highlight json %}
// config.json 파일
{
  "AppSettings": {
    "SiteTitle": "ConfigDemo"
  },
  "Data": {
    "DefaultConnection": {
      "ConnectionString": "Server=(localdb)\\mssqllocaldb;Database=aspnet5-ConfigDemo;"
    }
  }
}
{% endhighlight %}

이렇게 설정한 값들은 애플리케이션이 시작되는 시점에 로드되는데 여러 소스로 구성할 수 있기 때문에 추가하는 순서에 따라 우선순위가 정해지고, 나중에 추가되는 소스가 높은 우선순위를 갖는다.

<리스트 3> 외부 파일(config.json)과 환경 변수로 구성한 예
{% highlight C# %}
var configFluent = new Configuration()
                  .AddJsonFile("config.json")
                  .AddEnvironmentVariables();
{% endhighlight %}

`Configuration` 클래스는 여러 소스에 대한 컬렉션이다. 리스트 3의 예는, config.json에 정의된 키 값이 환경 변수에도 정의되어 있을 경우, 환경 변수의 키 값이 최종적으로 사용된다. 리스트 4에서는 환경 변수를 활용하여 web.config의 transform과 유사한 기능을 구현하고 있다.

<리스트 4> 환경 변수를 활용해 web.config의 transform과 유사한 기능 구현

{% highlight C# %}
var configuration = new Configuration()
    		  .AddJsonFile("config.json")
    		  .AddJsonFile($"config.{env.EnvironmentName}.json", optional: true);
{% endhighlight %}

<리스트 4>의 마지막 줄을 보면 C# 6의 새로운 문자열 포매팅 기능을 사용해 파일명을 표현하고 있는데 `env.EnvironmentName` 값이 production 이라면 config.production.json 파일을 로드 할 것이다. 애플리케이션을 배포할 서버의 환경 변수를 고려한 설정 파일을 만들 수 있어 서버 특화된 설정이 가능한 것이다.

####정리####

ASP.NET 5 프로젝트에서는 Web.config를 대신하는 새로운 구성(configuration) 모델이 도입되었다. 이 글을 통해 그 특징을 살펴보았는데 클라우드 호스팅을 고려하여 서버 환경에 특화된 구성이 가능하고 다양한 외부 파일 소스를 지원하는 점이 매우 흥미롭다.