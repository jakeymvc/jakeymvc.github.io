---
title:  "MVC 확장 포인트 (2/10) - Action Filter"
date:   2015-12-09
categories: mvc
tags: extension
author: Jake Ryu
permalink: /mvc-extension-action-filter
---

MVC 애플리케이션을 작성하면서 가장 빈번하게 들여다 보는 곳이 컨트롤러가 아닐까 싶다. 그 이름에서 알 수 있듯이 흐름을 제어하기 때문이다. 컨트롤러의 핵심 요소인 액션과 액션 결과(Action Result)에는 필터 어트리뷰트를 적용할 수 있는데 이를 액션 필터, 결과 필터라고 한다. 이 필터들을 MVC 요청 파이프라인의 흐름에 간섭할 수 있는 확장 포인트로 삼을 수 있다.  

####MVC 요청 라이프 사이클####

아래 그림은 MVC 요청이 액션에 전달되고 액션 결과를 생성하기까지의 라이프 사이클을 보여준다.

![MVC Request Life Cycle](/assets/mvc/mvc-action-result-filters.png)

<그림 1> MVC 요청 라이프 사이클

이전 글 [Action Result](/mvc-extension-action-result/)에서 알아 봤듯이 액션 메서드는 명령(액션 결과의 종류)을 결정하고 이 명령은 해당 객체(액션 결과)에 의해 실행된다. 분리되어 있는 구조 덕에 확장 또는 간섭의 포인트를 더 많이 갖게 되었다. 

액션 필터와 결과 필터를 모두 사용한다면 그림 1과 같이 네 곳에서 요청과 응답에 관련된 작업을 할 수 있다.

####필터 구현에 필요한 인터페이스####

``` csharp
public interface IActionFilter
{
    void OnActionExecuting(ActionExecutingContext filterContext);
    void OnActionExecuted(ActionExecutedContext filterContext);
}
```

<목록 1> 액션 필터를 구현하기 위한 인터페이스

``` csharp
public interface IResultFilter
{
    void OnResultExecuting(ResultExecutingContext filterContext);
    void OnResultExecuted(ResultExecutedContext filterContext);
}
```

<목록 2> 결과 필터를 구현하기 위한 인터페이스

목록 1과 2는 각각 액션 필터와 결과 필터를 구현할 때, 구현해야 하는 인터페이스이다. 참고로, 두 가지 필터 모두 필요하다면 목록 3에 있는 `ActionFilterAttribute`를 대신 사용할 수도 있다.

``` csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, Inherited = true, AllowMultiple = false)]
public abstract class ActionFilterAttribute : FilterAttribute, IActionFilter, IResultFilter
{
    // The OnXxx() methods are virtual rather than abstract so that a developer need override
    // only the ones that interest him.

    public virtual void OnActionExecuting(ActionExecutingContext filterContext)
    {
    }

    public virtual void OnActionExecuted(ActionExecutedContext filterContext)
    {
    }

    public virtual void OnResultExecuting(ResultExecutingContext filterContext)
    {
    }

    public virtual void OnResultExecuted(ResultExecutedContext filterContext)
    {
    }
}
```

<목록 3> 액션 필터 어트리뷰트


####워크플로우####

쇼핑몰에서 주문을 확인하고 배송지 입력, 결재 정보 입력, 최종 확인 후 주문을 내는 것과 같이 일련의 작업을 순서대로 진행하는 것을 워크플로우라고 한다. 정의된 작업 순서라고 볼 수 있겠다.

이번 글에서는 액션 필터만 사용해서 이런 워크플로우를 구현해 보려고 한다. 강제화된 작업 순서는 사용자 입력뿐만 아니라 프로세스 과정에도 많이 찾아볼 수 있다. 

이런 작업을 구현하려면 각 작업마다 정의된 상태가 부여되어야 하고 사용자 흐름에 맞게 작업 상태를 지속적으로 업데이트해야 한다. Step 1, 2, 3, 4를 순서대로 진행한다고 할 때, 상태를 확인하고 업데이트하는 비슷한 코드가 각 단계마다 중복될 것이 충분히 예상된다. 이런 코드를 액션 필터로 추상화해보자.

####로직 설계####

각 스텝은 하나의 액션 메서드에서 수행하는 작업이라고 가정하고, 액션 메서드를 수행하기 전과 후에 어떤 일이 필요한지 생각해보자.

![워크플로우](/assets/mvc/action-filter-workflow.png)

<그림 2> 요청 처리 워크플로우

사용자가 최초 스텝을 수행했다면 추적을 위한 세션 값을 할당할 것이다. 이 값은 사용자 정보의 일부로 데이터베이스에도 저장될 것이다. 따라서 사용자 세션 값이 있다는 의미는 최소한 Step 1을 거쳤다는 의미이고 이 세션 값을 찾을 수 없다면 사용자를 처음 스텝으로 리다이렉트한다.

각 액션 메서드는 실행에 앞서 최소한 완료해야 할 스텝을 완료했는지 확인해야 한다. 만약, Step 4에서 최소 완료 스텝을 2로 설정하면 Step 3는 건너뛸 수 있다는 것을 의미한다. 최소 완료 스텝을 사용함으로써 좀더 유연한 분기 로직을 구현할 수 있다.

이렇게 이전 상태를 확인하고 조건에 충족하지 않으면 리다이렉트하는 로직은 액션 필터의 OnActionExecuting 메서드에서 구현한다.

액션 메서드를 수행한 후에는 사용자 정보에 완료한 스텝을 업데이트해야 할 것이다. 액션 메서드 실행 후의 작업이므로 OnActionExecuted 메서드에서 구현할 것이다.


####액션 필터 구현####

액션 필터에서 사용할 세 가지 상태 값은 아래와 같다.

* 현재 단계
* 최소 (완료 요구된) 단계
* 사용자가 완료한 마지막 단계

현재 스텝과 최소 완료 스텝은 필터를 액션 메서드에 적용할 때 액션 메서드에 맞게 설정할 수 있지만 사용자가 마지막으로 완료한 단계는 데이터베이스에서 관리된다. 이 세 가지 상태를 사용하면 액션 메서드의 호출 순서 즉, 사용자 입장에서 다시 표현할 때, 이동하는 페이지의 순서를 강제화할 수 있다. 목록 4은 액션 필터의 기본 골격을 보여주고 있다.

``` csharp
public class WorkflowFilter : FilterAttribute, IActionFilter
{
    private int _highestCompletedStep;

    public int MinRequiredStage { get; set; }
    public int CurrentStage { get; set; }

    public void OnActionExecuting(ActionExecutingContext filterContext)
    {
    }
    
    public void OnActionExecuted(ActionExecutedContext filterContext)
    {
    }
}
```

<목록 4> 워크플로우 액션 필터의 기본 구조

목록 4을 보면 액션 필터를 작성하기 위해 필요한 `FilterAttribute`와 `IActionFilter`를 상속하고 있다. 클래스 멤버로 세 가지 상태 값을 두고 있고 `IActionFilter` 인터페이스에 따른 `OnActionExecuting`, `OnActionExcuted`, 두 개의 메서드를 확인할 수 있다.

컨트롤러의 액션 메서드에 액션 필터를 적용하는 방식은 목록 5와 같다.

``` csharp
[WorkflowFilter(
    MinRequiredStage = (int)WorkflowValues.Begin,
    CurrentStage = (int)WorkflowValues.Step1
)]
public ActionResult Step1()
{
    var stepData = new Step1();
    if (Session["Tracker"] != null)
    {
        var tracker = (Guid)Session["Tracker"];
        stepData = _context.Step1s.FirstOrDefault(x => x.User.Tracker == tracker);
    }

    return View(stepData);
}

[HttpPost]
[WorkflowFilter(
    MinRequiredStage = (int)WorkflowValues.Begin,
    CurrentStage = (int)WorkflowValues.Step1
)]
public ActionResult Step1(Step1 model)
{
    //... Step1 에 관련된 데이터 저장
}

[WorkflowFilter(
    MinRequiredStage = (int)WorkflowValues.Step1,
    CurrentStage = (int)WorkflowValues.Step2
)]

public ActionResult Step2()
{
    //... 생략
}
```

<목록 5> 액션 메서드에 액션 필터 적용

WokflowValues는 Begin = 0 부터 Step1 = 1, ... ,Step4 = 4 까지 enum으로 정의되어 있다. 각 스텝마다 최소 완료 스텝과 현재 스텝을 액션 필터로 수식하고 있다. 목록 5에서는 다른 스텝들이 하나의 컨트롤러에 있기 때문에 액션 메서드마다 필터를 적용하고 있지만 Step1 만 처리하는 컨트롤러를 사용한다면 컨트롤러 수준에서 필터를 적용할 수 있다. 또한, 필요하다면 글로벌로 적용하는 것도 가능하다.

이제 액션 필터의 구현부를 살펴보자.

``` csharp
public void OnActionExecuting(ActionExecutingContext filterContext)
{
    var applicantId = filterContext.HttpContext.Session["Tracker"];
    if (applicantId != null)
    {
        Guid tracker;
        if (Guid.TryParse(applicantId.ToString(), out tracker))
        {
            var _context = DependencyResolver.Current.GetService<AppDbContext>();
            _highestCompletedStep = _context.Users.FirstOrDefault(x => x.Tracker == tracker).WorkflowStage;
            if (MinRequiredStage > _highestCompletedStep)
            {

                switch (_highestCompletedStep)
                {
                    case (int)WorkflowValues.Step1:
                        filterContext.Result = GenerateRedirectUrl("Step1", "PreProcess");
                        break;

                    case (int)WorkflowValues.Step2:
                        filterContext.Result = GenerateRedirectUrl("Step2", "PreProcess");
                        break;

                    case (int)WorkflowValues.Step3:
                        filterContext.Result = GenerateRedirectUrl("Step3", "PreProcess");
                        break;
                }
            }
        }
    }
    else
    {
        if (CurrentStage != (int)WorkflowValues.Step1)
        {
            filterContext.Result = GenerateRedirectUrl("Step1", "PreProcess");
        }
    }
}

private RedirectToRouteResult GenerateRedirectUrl(string action, string controller)
{
    return new RedirectToRouteResult(new RouteValueDictionary(new { action = action, controller = controller }));
        }
```

<목록 6> 액션 메서드가 실행되기 전, 요청을 처리하는 `OnActionExecuting` 메서드

로직으로 잡은 내용이 목록 6의 메서드에 구현되어 있다. 먼저, 추적을 위한 세션 값이 존재하는지 확인하고 없다면 어떤 스텝도 완료하지 않았으므로 처음 스텝으로 보낸다. 그러나, 현재 단계가 처음 스텝일 수도 있기 때문에 그렇지 않은 경우에만 리다이렉트한다.

그 다음엔 데이터베이스에서 사용자가 완료한 마지막 스텝을 조회하여 최소로 요구되는 스텝과 비교한다. 최소 요구 스텝보다 완료한 스텝이 적다면 액션 메서드로 가는 조건에 부합하지 않기 때문에 최근 완료한 스텝으로 리다이렉트한다. `filterContext.Result` 는 액션 결과 형식이고 이 속성에 `RedirectToRouteResult` 인스턴스를 할당하여 다른 액션으로 이동할 수 있다.

이 로직을 무사히 통과하면 액션 메서드가 실행되고 이어서 액션 필터의 `OnActionExecuted` 메서드가 호출된다.

``` csharp
public void OnActionExecuted(ActionExecutedContext filterContext)
{
    var _context = DependencyResolver.Current.GetService<AppDbContext>();
    var sessionId = filterContext.HttpContext.Session["Tracker"];
    if (sessionId != null)
    {
        Guid tracker;
        if (Guid.TryParse(sessionId.ToString(), out tracker))
        {
            if (filterContext.HttpContext.Request.RequestType == "POST" 
                && CurrentStage >= _highestCompletedStep)
            {
                var user = _context.Users
                           .FirstOrDefault(x => x.Tracker == tracker);
                user.WorkflowStage = CurrentStage;
                _context.SaveChanges();
            }
        }
    }
}
```

<목록 7> 액션 메서드가 실행된 후, 사후 처리하는 `OnActionExecuted` 메서드

`OnActionExecuted` 메서드의 목적은 사용자가 완료한 스텝을 데이터베이스에 반영하는 것이다. 따라서, 값을 저장할 때 사용하는 POST 액션 메서드가 호출되었는지, 현재 스텝이 이미 완료한 스텝보다 같거나 큰지를 확인하고 사용자가 완료한 워크플로우 단계를 업데이트한다. 이렇게 수정된 값은 다음 스텝을 호출할 때, 최고 요구 스텝을 충족하는지 알아보기 위해 사용된다. 

####다른 방법####

한편으로, MVC 컨트롤러는 이미 `IActionFilter` 인터페이스를 상속하고 있어 액션 호출전과 호출 후에 할 작업을 정의할 수 있다.액션 필터를 구현하지 않고도 컨트롤러 자체에서 처리가 가능한 것이다. 그러나, 구현 내용을 다른 컨트롤러에 재사용할 수 없고, 호출된 액션 메서드에 따라 다른 처리를 하려면 `filterContext` 가 제공하는 라우트 데이터를 참고해야 한다. 컨트롤러에 공통으로 적용할 일이라면 이 방법을 사용하는 것도 고려해 볼 일이다.

참고로, `filterContext`는 라우트 데이터 이외에도 ControllerContext, ActionResult, ActionDescriptor, Request 객체에 대한 접근을 제공한다.

``` csharp
public abstract class Controller : ControllerBase, IActionFilter, IAuthenticationFilter, IAuthorizationFilter, IDisposable, IExceptionFilter, IResultFilter, IAsyncController, IAsyncManagerContainer
{
    //...
    protected virtual void OnActionExecuting(ActionExecutingContext filterContext)
    {
    }

    protected virtual void OnActionExecuted(ActionExecutedContext filterContext)
    {
    }
    //...
}
```

<목록 8> MVC Contoller

목록 8에서 확인할 수 있듯이 IActionFilter 인터페이스를 구현하고 있는 실제 내용은 없다. 두 개의 메서드를 virtual로 정의하여 자식 클래스에서 오버라이드할 수 있도록 하고 있다.


###정리###

사용자가 완료한 단계를 데이터베이스에 업데이트 하면서 액션 필터를 통해 현재 스텝과 최소 요구 스텝을 정의하여 워크플로우를 구현해봤다. 

액션 필터는 이렇게 MVC 파이프라인에 사용자 로직을 주입할 때 사용할 수 있다. 결과 필터는 다루지 않았지만 이렇게 여러 필터를 통해 사용자 로직을 구현하고 흐름을 제어할 수 있다는 것이 매력적이다.