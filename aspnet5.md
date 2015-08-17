---
layout: post
title: ASP.NET 5
permalink: /aspnet5/
---

새롭게 나오는 기술은 언제나 흥미롭다. 아직 주어진 환경을 벗어나지 못하는 개발자로서 새로운 개발 프레임워크가 나오면 하는 일에 조금이라도 도움이 되기를 바랄 뿐이다.

ASP.NET 5에 대한 전반적인 내용을 접한 건, 2015년 5월에 런던에서 개최된 SDD(Software Design & Development) 컨퍼런스에서다. [Brock Allen][1]의 종일 세션에서 ASP.NET의 신세계를 경험했다고 할까. 그는 거침없이 ASP.NET 프레임워크의 REBOOT라고 표현했다.  

공부도 하고 정리도 할겸, [ASP.NET Korean User Group][2] 이라는 페이스북 그룹에 ASP.NET 5의 변화를 열 가지로 정리하는 연재 비슷한 걸 했다. 페이스북에서 읽기에 괜찮을 정도로 내용을 정말 짧게 요약해서 핵심만 전달하고 싶었다. 그러다 보니 예제를 다룰 수도 없고 결국에는 뭔가 아쉬운 결과물만 남더라.

덜어낸 내용이 아깝기도 하고 좀더 완성된 내용을 전달하기 위해 마이크로소프트웨어 잡지에 기고하기 시작했다. 3회 연재로 열 가지 내용을 요약하는 것이었다. 그러나, 잡지 연재도 지면 수를 맞추기 위해 내용을 넣다 뺏다하는 일이 쉽지는 않더란... 

여기 모아둔 내용은 그렇게 정리의, 정리의 과정을 거치면서 ASP.NET 5의 변화를 요약한 것들이다. 그 많은 변화중 중요하다고 생각한 것만 정리했기에 다루지 못한 내용들도 많다. ASP.NET 5의 정식 릴리즈를 기다리면서 그 동안 못 다뤘던 주제나 흥미로운 주제가 생기면 블로그를 통해 계속 추가할 생각이다.

부족한 글이지만 여러분께 도움이 되길 바라며 대부분이 개념적인 내용이라 가급적 순서대로 읽으시기를 추천한다.

<br />
* * *
<br />
{% assign aspnet5posts = site.categories.aspnet5 | sort: "date" %}
<div>
  <ul class="post-list">
    {% for post in aspnet5posts %}
      <li>
        <h2>
          <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
        </h2>
      </li>
    {% endfor %}
  </ul>
</div>

* * *


[1]: https://www.develop.com/technicalstaff/details/brock-allen
[2]: https://www.facebook.com/groups/AspxKorea/
