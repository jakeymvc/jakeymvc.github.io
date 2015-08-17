---
title:  "태스크 러너 (Task Runner)"
date:   2015-08-03 12:00:00
categories: aspnet5
author: Jake Ryu
permalink: /task-runner/
---

자바스크립트의 비약적인 발전으로 현대의 웹 개발은 클라이언트 단에서 다양한 일을 효과적으로 수행하고 있다. 모던 웹을 작성한다고 할 때 어떤 일들이 필요한가.

>* LESS 또는 SASS 파일을 CSS로 컴파일하기

>* CoffeeScript 또는 TypeScript를 자바스크립트 파일로 컴파일 하기

>* 자바스크립트 파일들을 묶고(bundle) 축소(minify)하기

>* 자바스크립트가 코딩 규칙을 따르는지 JSHint 툴로 검사하기

아마 박스 속 작업들이 웹 개발의 일부가 되어 애플리케이션마다 반복적으로 수행될 것이다. 이런 일련의 작업들을 자동화 하기 위해 태스크 러너(이하 task runner)라는 하나의 앱을 사용하는데 비주얼 스튜디오 2015(Visual Studio 2015)는 task runner로서 가장 인기 있는 자바스크립트 기반의 Gulp와 Grunt를 지원한다. 

Gulp가 비교적 최근에 개발되었고 현대적인 문법과 디스크 I/O가 아닌 스트림을 사용한 빠른 성능 때문인지 ASP.NET 5 Web Site 템플릿에 의해 기본적으로 사용된다. Gulp는 노드 패키지로서 NPM(Node.js Package Manger) 설정 파일인 package.json 에 의해 설치된다. 그리고, 자동화 작업(task)은 gulpfile.js 파일에 작성한다. 리스트 1은 Assets 폴더 하위의 모든 자바스크립트 파일을 wwwroot/js 폴더로 복사하는 예제다. 

<리스트 1> Assets 폴더 하위의 모든 자바스크립트 파일을 wwwroot/js 폴더로 복사

{% highlight js %}
var gulp = require('gulp');
var paths = {
    src: "./Assets/**/*.js",
    dest: "./wwwroot/js/"
}
gulp.task('default', function () {		// default 는 작업명
    return gulp.src(paths.src)         	// Returns a stream
        	 .pipe(gulp.dest(paths.dest))   	// Pipes the stream somewhere
});
{% endhighlight %}

이렇게 default라고 정의한 작업을 Task Runner 탐색기를 통해 실행할 수도 있지만 그림 1에서처럼 일반적으로 빌드 이벤트에 바인딩하여 자동으로 수행하게 한다.

[![default 작업을 빌드 이벤트에 바인딩][1]][1]

<그림 1> default 작업을 프로젝트 빌드 후에 수행하도록 설정

Task runner의 지원이 꼭 필요한 Bower라는 또 다른 노드 패키지가 있다. Bower는 Git 위에서 동작하며 jquery, bootstrap, angular를 포함한 35,000 이상의 패키지를 관리하는 웹 패키지 매니저로 알려져 있는데, 생태계의 규모와 task runner와의 좋은 궁합 때문에 클라이언트 단에서 NuGet 패키지보다 Bower가 추천된다. 

Bower.json 파일에 등록된 클라이언트 라이브러리들은 bower_components 폴더에 다운로드 되고 task runner를 이용해 wwwroot 폴더로 복사하는 것이 ASP.NET 5 애플리케이션의 전형적인 관리 방식이다.

[1]: /assets/aspnet5/task-binding.png