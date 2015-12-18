테이블 형식의 데이터를 전달하면 OpenOfficeXml을 사용해서 엑셀 컨텐트를 byte[] 로 반환한다.
MVC의 File() 오버라이드 중 하나를 이용하면 쉽게 엑셀을 다운로드 시킬 수 있다.


이건 MVC 확장 시리즈의 Action Result 에 추가할 만한 내용이다. 
FileResult를 상속해서 엑셀 결과를 리턴하는 방법을 소개하자.
csv 버전이 이미 있기 때문에 엑셀은 또 하나의 adtion이다.

