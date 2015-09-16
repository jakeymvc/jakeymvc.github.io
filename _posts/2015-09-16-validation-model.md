---
title:  "유효성 검사 모델 (MVC Validation)"
date:   2015-09-16 16:15:00
categories: mvc
tags: validaton
author: Jake Ryu
permalink: /validation-model/
---

웹 페이지를 통해 사용자 입력 값을 받다보면 꼭 필요하면서도 성가신 것이 유효성 검사(validation)이다. ASP.NET MVC와 Web API는 아주 유연하면서도 사려 깊은 유효성 검사 모델을 제공한다.

이번 글에서는 실무에서 조금 복잡하게 느껴졌던 유효성 검사를 어떤 방식으로 해결했는지 살펴보고, MVC와 Web API에서 사용할 수 있는 다양항 유효성 검사 모델을 살펴볼 것이다.

시나리오는 이렇다. 생산된 자동차가 보관되는 야적장에서 재고관리 차원에서 주기적인 검사가 필요한데,

- 2개월마다 배터리 / 외관 / 타이어 공기압 검사를 실시한다
- 5개월마다 배터리 / 외관 / 타이어 공기압 / 시동 / 인테리어 / 엔진실 / 차량 이동 검사를 실시한다
- 9개월마다 배터리 / 외관 / 타이어 공기압 / 시동 / 차량 이동 / 클러치 검사를 실시한다
- 12개월마다 배터리 / 외관 / 타이어 공기압 / 시동 / 인테리어 / 엔진실 / 브레이크 패드 검사를 실시한다

야적작에서 사용하는 모바일 기기를 통해 입력된 정보는  Web API을 사용하여 서버에 저장되고 Web API로 전달되는 입력 모델은 목록 1과 같다.

<목록 1> 재고 관리용 Web API 입력 모델 

{% highlight c# %}
public class StockMaintenanceRequestModel
{
    [Required, MaxLength(17), MinLength(17)]
    public string Vin { get; set; }     // vehicle identification number

    [Required]
    public int Interval { get; set; }

    [MaxLength(20)]
    public string BatteryInspection { get; set; }

    [MaxLength(20)]
    public string ExteriorInspection { get; set; }

    [MaxLength(512)]
    public string ExteriorInspectionComments { get; set; }

    [MaxLength(20)]
    public string TyrePressureCheck { get; set; }

    [MaxLength(512)]
    public string TyrePressureCheckComments { get; set; }

    [MaxLength(20)]
    public string StartingWarmUp { get; set; }

    [MaxLength(20)]
    public string InteriorInspection { get; set; }

    [MaxLength(512)]
    public string InteriorInspectionComments { get; set; }

    [MaxLength(20)]
    public string UnderBonnet { get; set; }

    [MaxLength(20)]
    public string MoveVehicle { get; set; }

    [MaxLength(20)]
    public string CheckClutch { get; set; }

    [MaxLength(20)]
    public string CheckBrakePads { get; set; }
}
{% endhighlight %}

속성별 어트리뷰트를 통해 기본적인 제약사항을 정했지만 검사 주기별로 필수 입력값이 달라져서 슬프게도 Interval 값을 분석해서 필수값 입력 여부를 처리해야만 한다. 현재 수준에서는 자동차의 고유 ID(Vin)와 검사 주기(Interval)를 필수 입력 값으로 지정하고 있다. 

목록 2의 Post 메서드가 호출되는 순간 기본 모델 바인더가 동작해서 요청 정보 - 폼, 라우트, 쿼리스트링 등 - 를 토대로  `StockMaintenanceRequestModel` 의 인스턴스를 생성해서 메서드에 돌려 준다. 이때, 모델 바인더는 `StockMaintenanceRequestModel` 클래스의 속성 수준에서 정의한 제약사항에 위반된 사실을 발견하면 즉시 그 내용을 ModelState에 담아 클라이언트에 돌려준다. 모델 바인딩에 문제가 없었다면 이제 목록 2에서 처럼 필수 입력 값을 확인한다.


<목록 2> Post 메서드 - 검사 주기별로 필수 값에 대한 유효성 검사

{% highlight c# %}
public class StockMaintenanceController : ApiController
{
    public IHttpActionResult Post(StockMaintenanceRequestModel req)
    {
        CallResult result = null;

        if (req == null)
            return BadRequest();

        // validation
        switch (req.Interval)
        {
            case 2:
                if(string.IsNullOrEmpty(req.BatteryInspection))
                    ModelState.AddModelError("BatteryInspection", 
                        "Battery inspection value is required");

                if(string.IsNullOrEmpty(req.ExteriorInspection))
                    ModelState.AddModelError("ExteriorInspection", 
                        "Exterior inspection value is required");

                if(string.IsNullOrEmpty(req.TyrePressureCheck))
                    ModelState.AddModelError("TyrePressureCheck", 
                        "Tyre pressure check value is required");

                break;

            case 5:
                if(string.IsNullOrEmpty(req.BatteryInspection))
                    ModelState.AddModelError("BatteryInspection", 
                        "Battery inspection value is required");

                if(string.IsNullOrEmpty(req.ExteriorInspection))
                    ModelState.AddModelError("ExteriorInspection", 
                        "Exterior inspection value is required");

                if(string.IsNullOrEmpty(req.TyrePressureCheck))
                    ModelState.AddModelError("TyrePressureCheck", 
                        "Tyre pressure check value is required");

                if(string.IsNullOrEmpty(req.StartingWarmUp))
                    ModelState.AddModelError("StartingWarmUp", 
                        "Starting & Warm Up value is required");

                if(string.IsNullOrEmpty(req.InteriorInspection))
                    ModelState.AddModelError("InteriorInspection", 
                        "Interior inspection value is required");

                if(string.IsNullOrEmpty(req.UnderBonnet))
                    ModelState.AddModelError("UnderBonnet", 
                        "Under bonnet value is required");

                if(string.IsNullOrEmpty(req.MoveVehicle))
                    ModelState.AddModelError("MoveVehicle", 
                        "Move vehicle value is required");

                break;

            case 9:

                if(string.IsNullOrEmpty(req.BatteryInspection))
                    ModelState.AddModelError("BatteryInspection", 
                        "Battery inspection value is required");

                if(string.IsNullOrEmpty(req.ExteriorInspection))
                    ModelState.AddModelError("ExteriorInspection", 
                        "Exterior inspection value is required");

                if(string.IsNullOrEmpty(req.TyrePressureCheck))
                    ModelState.AddModelError("TyrePressureCheck", 
                        "Tyre pressure check value is required");

                if(string.IsNullOrEmpty(req.StartingWarmUp))
                    ModelState.AddModelError("StartingWarmUp", 
                        "Starting & Warm Up value is required");

                if(string.IsNullOrEmpty(req.MoveVehicle))
                    ModelState.AddModelError("MoveVehicle", 
                        "Move vehicle value is required");

                if(string.IsNullOrEmpty(req.CheckClutch))
                    ModelState.AddModelError("CheckClutch", 
                        "Check clutch value is required");

                break;

            case 12:

                if(string.IsNullOrEmpty(req.BatteryInspection))
                    ModelState.AddModelError("BatteryInspection", 
                        "Battery inspection value is required");

                if(string.IsNullOrEmpty(req.ExteriorInspection))
                    ModelState.AddModelError("ExteriorInspection", 
                        "Exterior inspection value is required");

                if(string.IsNullOrEmpty(req.TyrePressureCheck))
                    ModelState.AddModelError("TyrePressureCheck", 
                        "Tyre pressure check value is required");

                if(string.IsNullOrEmpty(req.StartingWarmUp))
                    ModelState.AddModelError("StartingWarmUp", 
                        "Starting & Warm Up value is required");

                if(string.IsNullOrEmpty(req.InteriorInspection))
                    ModelState.AddModelError("InteriorInspection", 
                        "Interior inspection value is required");

                if(string.IsNullOrEmpty(req.UnderBonnet))
                    ModelState.AddModelError("UnderBonnet", 
                        "Under bonnet value is required");

                if(string.IsNullOrEmpty(req.CheckBrakePads))
                    ModelState.AddModelError("CheckBrakePads", 
                        "Check brake pads value is required");

                break;
        }

        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }

        // 이하 생략
    }
}
{% endhighlight %}



<목록 2>를 보면 검사 주기별로 필요한 입력 값을 조사하여 빠진 항목이 있다면 `ModelState.AddModelError` 메서드를 사용해 에러 내용을 입력하고 있다. 그리고 마지막으로 `ModelState.IsValid` 속성을 조사하여 에러 내용이 하나라도 있으면 400 Bad Request 를 반환한다.

보기에 별로 좋지 않은 코드지만 딱히 좋은 방법이 떠오르지 않는다. 현재의 유효성 검사는 속성 수준에서 간단하게 처리할 수 없는데 그 이유는 속성간에 의존성이 있기 때문이다. 그래서 모델 수준에서 접근해야 하는데 몇 가지 방법 중 `IValidatableObject` 인터페이스를 구현하는 것으로 방향을 잡았다. 다른 방법으로는 사용자 정의 유효성 검사 어트리뷰트를 작성하는 것인데 다른 글에서 보다 자세하게 알아 보도록 한다.

현재의 유효성 검사 방식은, 전달 받은 StockMaintenanceRequestModel 모델의 유효성을 메서드 수준에서 확인하고 있다. 만약, 다른 메서드에서 StockMaintenanceRequestModel에 대한 유효성을 검사한다고 하면 Copy & Paste를 피하기 위한 고민이 시작될 것이다. 

특정 모델에 대한 유효성 검사를 수행할 때, 모델 자신 보다 더 나은 장소는 없어 보인다. 모델을 정의하면서 그 유효성까지 함께 정의한다면 그 모델을 사용하는 모든 곳에서는 유효성 검사에 대한 부담을 지지 않아도 된다. 다만, 유효성 검사를 트리거하는 장치가 필요한데, MVC 프레임워크가 이 부분을 책임지고 있으니 정말 다행이다.

<목록 3> IValidatableObject 인터페이스를 구현하도록 모델 변경

{% highlight c# %}
public class StockMaintenanceRequestModel : IValidatableObject
{
    [Required, MaxLength(17), MinLength(17)]
    public string Vin { get; set; }     // vehicle identification number

    [Required]
    public int Interval { get; set; }

    [MaxLength(20)]
    public string BatteryInspection { get; set; }

    // 나머지 속성 생략

    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        List<ValidationResult> errors = new List<ValidationResult>();

        // Interval 속성은 [Required] 어트리뷰트로 수식되어 있어 현재 메서드 호출 전에 유효성이 검사된다 

        // Interval에 따른 유효성 검사
        switch (Interval)
        {
            case 2:
                if (string.IsNullOrEmpty(BatteryInspection))
                    errors.Add(new ValidationResult("Battery inspection value is required"));

                if(string.IsNullOrEmpty(ExteriorInspection))
                    errors.Add(new ValidationResult("Exterior inspection value is required"));

                if(string.IsNullOrEmpty(TyrePressureCheck))
                    errors.Add(new ValidationResult("Tyre pressure check value is required"));

                break;

            case 5:
                if(string.IsNullOrEmpty(BatteryInspection))
                    errors.Add(new ValidationResult("Battery inspection value is required"));

                if(string.IsNullOrEmpty(ExteriorInspection))
                    errors.Add(new ValidationResult("Exterior inspection value is required"));

                if(string.IsNullOrEmpty(TyrePressureCheck))
                    errors.Add(new ValidationResult("Tyre pressure check value is required"));

                if(string.IsNullOrEmpty(StartingWarmUp))
                    errors.Add(new ValidationResult("Starting & Warm Up value is required"));

                if(string.IsNullOrEmpty(InteriorInspection))
                    errors.Add(new ValidationResult("Interior inspection value is required"));

                if(string.IsNullOrEmpty(UnderBonnet))
                    errors.Add(new ValidationResult("Under bonnet value is required"));

                if(string.IsNullOrEmpty(MoveVehicle))
                    errors.Add(new ValidationResult("Move vehicle value is required"));

                break;

            case 9:

                if(string.IsNullOrEmpty(BatteryInspection))
                    errors.Add(new ValidationResult("Battery inspection value is required"));

                if(string.IsNullOrEmpty(ExteriorInspection))
                    errors.Add(new ValidationResult("Exterior inspection value is required"));

                if(string.IsNullOrEmpty(TyrePressureCheck))
                    errors.Add(new ValidationResult("Tyre pressure check value is required"));

                if(string.IsNullOrEmpty(StartingWarmUp))
                    errors.Add(new ValidationResult("Starting & Warm Up value is required"));

                if(string.IsNullOrEmpty(MoveVehicle))
                    errors.Add(new ValidationResult("Move vehicle value is required"));

                if(string.IsNullOrEmpty(CheckClutch))
                    errors.Add(new ValidationResult("Check clutch value is required"));

                break;

            case 12:

                if(string.IsNullOrEmpty(BatteryInspection))
                    errors.Add(new ValidationResult("Battery inspection value is required"));

                if(string.IsNullOrEmpty(ExteriorInspection))
                    errors.Add(new ValidationResult("Exterior inspection value is required"));

                if(string.IsNullOrEmpty(TyrePressureCheck))
                    errors.Add(new ValidationResult("Tyre pressure check value is required"));

                if(string.IsNullOrEmpty(StartingWarmUp))
                    errors.Add(new ValidationResult("Starting & Warm Up value is required"));

                if(string.IsNullOrEmpty(InteriorInspection))
                    errors.Add(new ValidationResult("Interior inspection value is required"));

                if(string.IsNullOrEmpty(UnderBonnet))
                    errors.Add(new ValidationResult("Under bonnet value is required"));

                if(string.IsNullOrEmpty(CheckBrakePads))
                    errors.Add(new ValidationResult("Check brake pads value is required"));

                break;
        }



        return errors;
    }
}
{% endhighlight %}

장문의 유효성 검사 로직이 모델안으로 들어왔다. `IValidatableObject` 인터페이스에 정의된 `Validate` 메서드를 구현했는데 이 메서드는 모델 바인딩 후 MVC 프레임워크에 의해 자동으로 호출된다. 

컨트롤러에서 ModelState를 다룬 것과 유사한 방식으로 `ValidationResult` 를 다루고 있다. `Validate` 메서드의 반환 값으로 `ValidationResult`의 리스트를 돌려주면 목록 4와 같이 단정한 모습으로 브라우저에 오류를 알려준다.


<목록 4> JSON 포맷의 에러 메시지 

{% highlight json %}
{
    "Message":"The request is invalid.",
    "ModelState":
    {
        "req":[
            "Starting & Warm Up value is required",
            "Interior inspection value is required",
            "Under bonnet value is required",
            "Check brake pads value is required"
        ]
    }
}
{% endhighlight %}

위에서 다룬 예는 WebAPI를 위한 것이고 클라이언트 단 유효성 검사를 염두에 두지 않았기 때문에 만족스럽게 동작한다. 만약 MVC라면 어떤 유효성 검사라도 클라이언트 단에서 먼저 동작하기를 원할텐데 이를 위해 `ValidationAttribute` 클래스를 상속해서 사용자 정의 모델 어트리뷰트를 작성하는 것이 더 좋은 방법일 수 있다.

모델 바인딩과 모델 유효성 검사는 비슷하지만 다르게 구분해야 한다. 모델 정의에 따라 서로 얽혀서 동작하기 때문에 그 개념과 동작순서를 잘 알고 있어야 사용자 정의가 필요할 때, 사용자 정의 모델 바인더를 만들지, 사용자 정의 유효성 검사 어트리뷰트를 만들지, 자체적으로 유효성 검사를 수행하는 모델을 만들지 판단할 수 있기 때문이다. 

