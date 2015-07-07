---
title:  "Zip 파일 반환하는 ActionResult 만들기"
date:   2015-01-04 00:00:00
categories: mvc
author: Jake Ryu
---

이미지 뷰어를 구현하다 다운로드 기능이 필요해졌다. 한 번 클릭에 뷰어에서 재생되는 모든 이미지를 압축하여 다운로드 한다. 아무래도 압축 풀기의 번거로움이 있으니 이미지가 하나만 있다면 압축 없이 다운로드할 수 있게 해보자.

# DotNetZip

압축 후 다운로드 (zip and download)를 구현하기 위해 다른 개발자들은 어떻게 하고 있는지 알아보자.

[DotNetZip] 은 NuGet 패키지로서 2011년 2월부터 최근까지 꾸준히 업데이트 되었고 다른 패키지에 대한 의존성이 없다. 또한, 많은 개발자들이 다운로드한 것으로 보아 신뢰할 수 있겠고 `Microsoft Public License` 하에 기업에서도 사용하는데 문제가 없기 때문에 훌륭한 솔루션이다.

그럼에도 불구하고 패키지에 종속되지 않게 구현할 수 없을까 하여 조사해 보니, 닷넷 프레임워크 4.5에서 `ZipArchive`와 `ZipFile` 이라는 압축과  관련된 클래스가 추가된 것을 찾을 수 있었다. 이 클래스들을 사용하는 예제를 보니 `DotNetZip` 패키지를 사용하는 것 만큼이나 아주 간단하고 편리하게 압축 파일을 다룰 수 있다.

MVC 컨트롤러에는 파일을 반환하는 Action Result가 있으니 이를 참고하여 Zip 파일을 반환하는 Action Result를 만들어 보자.

# FileResult는 어떻게 구현되었나?

MVC 프레임워크에서 액션 메서드는 `Response` 개체를 직접 다루지 않는다. 대신, `ActionResult` 클래스를 상속하는 개체를 반환하도록 설계되었는데 이 것은 목표를 결정하는 것과 목표를 실행하는 것을 분리하는 일종의 Command 패턴이다.

> Command 패턴은 어떤 행동(명령)을 개체로 표현함으로써 명령을 실행하는 클라이언트와 명령 실행에 필요한 요소(데이터, 로직 등)를 분리한다. 모든 명령은 인터페이스를 상속하여 구현하므로 Logging, Validation, Undo 기능을 적용하기에 용이하다. 새로운 명령이 필요할 때, 기존 코드를 수정하지 않고 새 명령 클래스를 추가하기 때문에 Design Principal 중 하나인 Open for extension, but closed for modification 의 개념을 잘 따르고 있다. ActionResult가 추상클래스로서 ExecuteResult 메서드 하나만을 갖고 있고 MVC 프레임워크(클라이언트)에 의해 호출되어 실행된다는 것이 Command 패턴에서 Command가 갖는 개념과 일치한다.

MVC 프레임워크는 액션 메서드로부터 `ActionResult` 객체를 받는 경우, 그 객체에 정의되어 있는 `ExecuteResult` 메서드를 호출하고, 이 메서드는 실행 결과를 Response의 출력 스트림에 반영한다. 이해를 돕기 위해 `ActionResult` 중 한 가지인 파일을 반환하는 방법에 대해 상세히 알아보자.

MVC 컨트롤러는 파일을 반환하는 6개의 오버로드 메서드를 제공한다. ([MSDN 참조 - Controller.File Method])

{% highlight C# %}
protected internal FileContentResult File(byte[] fileContents, string contentType)
{
    return File(fileContents, contentType, null /* fileDownloadName */);
}
protected internal virtual FileContentResult File(byte[] fileContents, string contentType, string fileDownloadName)
{
    return new FileContentResult(fileContents, contentType) { FileDownloadName = fileDownloadName };
}
protected internal FileStreamResult File(Stream fileStream, string contentType)
{
    return File(fileStream, contentType, null /* fileDownloadName */);
}
protected internal virtual FileStreamResult File(Stream fileStream, string contentType, string fileDownloadName)
{
    return new FileStreamResult(fileStream, contentType) { FileDownloadName = fileDownloadName };
}
protected internal FilePathResult File(string fileName, string contentType)
{
    return File(fileName, contentType, null /* fileDownloadName */);
}
protected internal virtual FilePathResult File(string fileName, string contentType, string fileDownloadName)
{
    return new FilePathResult(fileName, contentType) { FileDownloadName = fileDownloadName };
}
{% endhighlight %}

이들 메서드는 파일을 만들거나 읽을 때, 편의를 위해 세 가지 다른 종류의 개체(`FilePathResult`, `FileStreamResult`, `FileContentResult`)를 사용한다. 그 이름에서 짐작할 수 있듯이 파일의 위치, 파일 스트림, 바이트 배열을 다루면서 궁극적으로 `Resopnse`의 출력 스트림에 그 내용을 작성한다. 이렇게 다른 소스를 다루어야 하기 때문에 FileResult 라는 추상 클래스에 `WriteFile` 라는 추상 메서드를 정의하여 자식 클래스에서 `WriteFile` 메서드를 구현하도록 한다. 클래스 간의 관계는 아래 클래스 다이어그램으로 쉽게 이해할 수 있다.

![FileResult 클래스의 상속관계](/assets/mvc/file-result-class.png)

[그림-1] FileResult 클래스의 상속관계

ASP.NET MVC 는 오픈 소스인 덕에 FileResult 클래스가 어떻게 구현되었는지 알아볼 수 있다.

{% highlight C# %}
public abstract class FileResult : ActionResult
{
    private string _fileDownloadName;
    protected FileResult(string contentType)
    {
        if (String.IsNullOrEmpty(contentType))
        {
            throw new ArgumentException(MvcResources.Common_NullOrEmpty, "contentType");
        }
        ContentType = contentType;
    }
    public string ContentType { get; private set; }
    public string FileDownloadName
    {
        get { return _fileDownloadName ?? String.Empty; }
        set { _fileDownloadName = value; }
    }
    public override void ExecuteResult(ControllerContext context)
    {
        if (context == null)
        {
            throw new ArgumentNullException("context");
        }
        HttpResponseBase response = context.HttpContext.Response;
        response.ContentType = ContentType;
        if (!String.IsNullOrEmpty(FileDownloadName))
        {
            // From RFC 2183, Sec. 2.3:
            // The sender may want to suggest a filename to be used if the entity is
            // detached and stored in a separate file. If the receiving MUA writes
            // the entity to a file, the suggested filename should be used as a
            // basis for the actual filename, where possible.
            string headerValue = ContentDispositionUtil.GetHeaderValue(FileDownloadName);
            context.HttpContext.Response.AddHeader("Content-Disposition", headerValue);
        }
        WriteFile(response);
    }
    protected abstract void WriteFile(HttpResponseBase response);
}
{% endhighlight %}

`content type` 은 파일의 정체성을 결정하는 중요 요소이므로 생성자에서 반드시 필요하다. `ExecuteResult` 메서드에서는 `Response` 개체에 접근하기 위해 컨트롤러 컨텍스트의 유효성 여부를 먼저 확인한 후, 파일이 다운로드될 수 있도록 `Response` 의 헤더를 조정한다. 참고로 `ContentDispositionUtil.GetHeaderValue` 메서드는 닷넷 프레임워크 4.0에서 파일명에 유니코드가 포함되었을 때 발생하는 `Content-Disposition` 헤더 작성 문제를 보완하기 위해 사용하고 있다.

이후 작성할 `ZipResult` 라는 사용자 정의 액션결과에서는 응답 헤더에 아래 내용을 추가할 것이다.

```
Content-Type: application/zip
content-disposition: attachment; filename=SampleFiles.zip
```

`ExecuteResult` 메서드는 프로세스의 마지막에 `WirteFile` 메서드에 `Response` 객체를 전달하는 것으로 종료되었다. 이 메서드는 추상 메서드로 정의되었기 때문에, 최종 처리는 자식 클래스 - `FilePathResult`, `FileStreamResult`, `FileContentResult` - 의 `WriteFile` 메서드에 의존하고 있는 것이다. 자식 클래스에서 `WriteFile` 메서드를 어떻게 구현했는지 살펴보자.

{% highlight C# %}
// FilePathResult
protected override void WriteFile(HttpResponseBase response)
{
    response.TransmitFile(FileName);
}
{% endhighlight %}

{% highlight C# %}
// FileStreamResult
protected override void WriteFile(HttpResponseBase response)
{
    // grab chunks of data and write to the output stream
    Stream outputStream = response.OutputStream;
    using (FileStream)
    {
        byte[] buffer = new byte[BufferSize];
        while (true)
        {
            int bytesRead = FileStream.Read(buffer, 0, BufferSize);
            if (bytesRead == 0)
            {
                // no more data
                break;
            }
            outputStream.Write(buffer, 0, bytesRead);
        }
    }
}
{% endhighlight %}

{% highlight C# %}
// FileContentResult
protected override void WriteFile(HttpResponseBase response)
{
    response.OutputStream.Write(FileContents, 0, FileContents.Length);
}
{% endhighlight %}

파일명을 사용하는 경우에는 'Response' 개체가 제공하는 메서드를 통해 파일을 처리하지만, 파일 스트림과 바이트 배열을 다룰 때는 'Response'의 출력 스트림에 그 내용을 직접 쓰고 있다.

`ActionResult` - `FileResult` - `FileResult의 자식 클래스`들까지 예상보다 복잡한 단계를 살펴봐야 했지만, 액션 메서드에서 File 관련 메서드를 사용할 때, MVC 프레임워크에서 어떻게 처리하는지 구체적으로 알게 되었다.

# 사용자 정의 ActionResult

이제 `FileResult`의 동작 원리를 이해했으니 `Zip` 파일을 반환하는 `ActionResult`를 만들어 보자. 이것은 MVC 프레임워크의 일부처럼 동작할 것이기에 프로젝트 루트 아래 `Infrastructure` 라는 폴더를 만들고 `ZipResult` 라는 클래스를 준비하여 아래 코드와 같이 편집하자.

{% highlight C# %}
public class ZipResult : ActionResult
{
    private string zipFileName;
    private List<string> filesToZip;
    public string ZipFileName { 
        get { return zipFileName ?? "unnamed.zip";} 
        set {zipFileName = value;}
    }
    public ZipResult(string zipFileName, List<string> filesToZip)
    {
        if (filesToZip == null)
        {
            throw new ArgumentException("No file is specified to compress and download.");
        }
        this.filesToZip = filesToZip;
        if (!string.IsNullOrEmpty(zipFileName))
        {
            zipFileName = zipFileName;
        }
    }
    public override void ExecuteResult(ControllerContext context)
    {
        using (MemoryStream ms = new MemoryStream())
        {
            using (ZipArchive fileContainer = new ZipArchive(ms, ZipArchiveMode.Create, true))
            {
                foreach (string filePath in filesToZip)
                {
                    if (File.Exists(filePath))
                    {
                        fileContainer.CreateEntryFromFile(filePath, Path.GetFileName(filePath));
                    }
                }
            }  // ZipArchive object is disposed and automatically backed in the memory stream
            var response = context.HttpContext.Response;
            response.ContentType = "application/zip";
            response.AppendHeader("content-disposition", "attachment; filename=" + ZipFileName);
            ms.WriteTo(response.OutputStream);
        }
    }
}
{% endhighlight %}

액션 메서드에서 반환할 수 있는 `ActionResult`가 되기 위해 `ZipResult` 클래스는 `ActionResult` 클래스를 상속하고 `ExecuteResult` 메서드를 재정의한다. 이 메서드의 마지막 과정을 먼저 보자면, `FileResult`의 자식 클래스들에서 본 것과 같이 `Response` 헤더에 zip 파일이라는 컨텐트 형식과 원하는 파일명을 명시하고, 압축 파일 자체를 `Response` 의 출력 스트림에 복사하고 있다.

`FileResult`의 `ExecuteResult`에서는 `WriteFile` 메서드를 한번 더 호출해서 다른 파일 소스(파일 경로, 파일 스트림, 바이트 배열)에 따라 다르게 동작하도록 다형성을 구현하였으나, `ZipResult` 액션 결과는 간단히 메모리 스트림만을 사용하기에 응답 처리까지 포함시키도록 구현했다. 만약, 다른 종류의 동작이 필요하다면 `FileResult` 클래스처럼 추상 클래스로 만들어 자식 클래스에 로직 구현을 위임하면 된다.

# 압축 구현하기

다시 코드로 돌아가서 압축과 관련된 내용을 살펴보자. 닷넷 프레임워크에서 압축 관련 클래스는 `System.IO.Compression` 네임스페이스에 있는데, 이 네임스페이스는 세 개의 어셈블리에 구현되어 있다. 여기서는 `System.IO.Compression.dll` 과 `System.IO.Compression.FileSystem.dll` 이라는 두 가지 어셈블리에 대한 참조를 추가해야 한다.

![닷넷 어셈블리 참조](/assets/mvc/compression-references.png)

[그림-2]  닷넷 어셈블리 참조

`ZipResult` 클래스의 `ExecuteResult` 메서드에서 구현할 핵심 사항은 여러 개의 파일을 하나의 zip 파일로 생성하는 것이다. 그러기 위해 압축에 포함될 대상 파일들과 최종 zip 파일의 이름 정도를 알아야 한다. 위의 소스 코드를 보면, 생성자에 정의한 두 개의 매개변수가 그 것이다. 이렇게 전달 받은 정보는 `ExecuteResult` 메서드에서 사용된다.

`ExecuteResult` 메서드를 구현하면서 zip 파일을 서버에 남기지 않기 위해 메모리 스트림을 사용했다. 이 메모리 스트림은 `ZipArchive` 객체를 생성할 때 전달되고, `ZipArchive` 객체가 소멸된 후에도 남아 있어 `Response.OuputStream`에 그 결과를 복사한다. 이를 위해, `ZipArchive` 생성자의 세 번째 매개변수를 `true`로 지정하여 `ZipArchive` 객체가 소멸되어도 스트림 객체를 `open` 상태로 남겨 두도록 설정했다. `ZipArchive` 객체를 정의한 `using`문의 끝에서 `ZipArchive`가 소멸되면서 `Dispose` 메서드가 호출되고 `ZipArchive` 객체의 내용은 자동으로 `backing store stream`인 메모리 스트림에 저장된다.

마지막으로 메모리 스트림의 내용을 `Response`의 출력 스트림에 복사하여 최종 압축 파일을 다운로드 시킨다.

# 컨트롤러에서 사용하기

컨트롤러에서는 `ZipResult` 객체의 생성에 필요한 압축대상 파일 리스트를 조회한다. `ZipResult` 객체를 반환하면 MVC 프레임워크에서는 `ExecuteResult` 메서드를 호출하여 최종 응답을 생성한다.

{% highlight C# %}
 public ActionResult Download()
{
    var fileStorage = new FileStorage();
    List<string> filePaths = fileStorage.GetFiles();
    return new ZipResult("SampleFiles.zip", filePaths);
}
{% endhighlight %}

# 정리

이미지를 네비게이트할 수 있는 뷰어 페이지에 `Zip and download` 기능을 추가한다는 간단한 요구사항에서 출발했다. 닷넷 프레임워크의 `BCL(Base Class Library)`을 활용한 압축 방법과 사용자 정의 `ActionResult`를 어떻게 작성하는지 살펴 보았다. 오픈 소스인 ASP.NET MVC 의 `FileResult` 구현을 보면서 `ActionResult`가 Command 패턴으로 구현된 것을 확인할 수 있었다.

전체소스는 [GitHub]에서 찾아볼 수 있다.


[DotNetZip]: https://www.nuget.org/packages/DotNetZip/
[MSDN 참조 - Controller.File Method]: https://msdn.microsoft.com/en-us/library/system.web.mvc.controller.file(v=vs.118).aspx
[GitHub]: https://github.com/JakeRyu/BlogSamples/tree/master/ZipAndDownload