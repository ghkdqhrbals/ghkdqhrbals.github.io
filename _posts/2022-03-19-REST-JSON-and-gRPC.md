---
title: "REST, JSON RPC, and gRPC"
categories:
  - Server
date: 2022-03-19 15:00:25 +0900
tags:
  - RESTFUL
  - JSON RPC
  - gRPC
---
# From MA(Monolithic Architecture) to MSA(Micro Service Architecture)
* 옛날에는 모놀리식 아키텍처로 설계를 하였음으로 하나의 거대한 어플리케이션으로 제작되었었다.    
  ex) 뱅킹 서비스 + UI + AD로직 + DB엑세스 로직 + ... = Application   

* 하지만 최근 다양한 서비스들을 유동적으로 제공하기 위한 MSA(Micro Service Architecture)를 채택하다보니 다양한 언어로 제작되어있다.    
    ex) 뱅킹 서비스 = API(Application Programming Interface), UI 관리 = API, DB 엑세스 = API, API1+2+3 = Application    
    ex) ID 관리 및 제어 서비스(C++) 서버, 뱅킹 서비스 서버(Golang), UI 관리 서버(Python), ...    

> 즉, 각각의 서비스가 독립적으로 동작하며, 팀 단위로 빌드/테스트/배포 가능. 그리고 이러한 서비스들은 서로 HTTP 통신을 수행함으로써 정보교환이 가능하다. HTTP는 API를 제공함으로써 서로 패킷데이터를 라우팅 및 처리하는데, 이러한 API는 여러가지 아키텍처형태로 제공가능하다(**REST**, **JSON-RPC**, **gRPC**).
{: .prompt-info}

-----------------
# API Architectures(REST, JSON-RPC, gRPC)
## REST API
REST는 HTTP 프로토콜을 효과적으로 사용하기 위한 아키텍처이다. 그리고 이러한 REST 아키텍처로 제공되는 API를 REST API라 한다.   
REST API는 다음으로 구성되어있다.   
* **자원** [URI]
> **URI**:Uniform Resource Identifier   
> **URL**:Uniform Resource Location   
* **행위**
> GET, POST, PUT, DELETE, ...     
> **GET** vs **POST**    
> * GET은 요청하는 데이터를 HTTP Header의 URI에 포함되어 전송된다. 예를 들어 구글에 오늘의 날씨를 검색하면, https://www.google.com/search?q=오늘의날씨 로 URL에 뜨는 것을 확인할 수 있다. 즉, 공격자가 Header의 자원을 스내핑하여 클라이언트가 어떠한 정보를 전송했는지 확인할 수 있어 보안적인 측면을 고려해보았을 때 적합하지 않다.    
> * POST는 요청하는 데이터를 HTTP BODY에 포함하여 전송한다. 보안적인 측면에서 딱히 GET방식과 낫다라고 표현할 수는 없다. 이도 마찬가지로 공격자가 스내핑하여 관찰할 수 있기 때문이다. 하지만, BODY는 Header와 별개로 따로 이전에 암호키를 교환하고, 암호화하여 전송할 수 있기에 보안 scalability가 존재한다. 즉, 암호화할 수 있어 보안적으로 선호되는 방식이다.     

    > 물론, TLS가 적용된다면 엔드포인트 보안이 설정됨으로 Man in the Middle와 같은 공격에 대해 GET이나 POST는 같은 보안수준을 제공할 수 있다.
    {: .prompt-info}

* **표현**
> JSON, XML, TEXT, ...    

REST 방식의 API 구현은 **CRUD**(Create/Read/Update/Delete)에 초점이 맞추어져있다. 이는 구체적인 동작을 나타내기에는 쉽지 않다. 다음의 예를 확인하자.    
> * 만일 클라이언트는 서버로부터 특정 유저의 ID를 가져오고 싶다라고 가정하자.    
> * 이를 위해서 클라이언트는 서버에게 다음의 형태로 HTTP를 전송한다.   
    **[자원] : /user, [행위] : GET, [표현] : JSON{"limit",...}**  
> * 클라이언트는 전체 유저목록을 반환 받고, 이 중 원하는 특정 유저의 ID 필드를 검색하여 가져간다.  
> * 이 때, 클라이언트가 반환받는 **Payload가 상당히 커진다**. 이는 **세부적인 표현이 힘든 REST API가 가지는 일반적인 딜레마**이다.    

> 즉, REST형태로 API를 설계할 떄, 일반적인 CRUD 행위가 아닌 추가적인 행위가 필요할 때 클라이언트의 부담이 커지는 경향이 존재한다.   
{: .prompt-warning}

-----------------

## JSON-RPC
* JSON-RPC는 REST보다 더욱 구체적인 API를 제공할 수 있다. 예로 JSON-RPC는 GetUser(String username)와 JSON형태의 데이터를 http body에 담아 전송함으로써 구체적인 표현이 가능하다.    
> ex) BODY: {"**jsonrpc**":"2.0", "**method**":"GetUser", "username":"Hwangbo Gyumin"}      
* 따라서 JSON-RPC는 High performance이며, Payload가 작다. 반면 서버 마음대로 표준을 생성하기에 다음의 **단점**이 존재한다.   
  1. 표준화를 할 수 없다.   
  2. 실제 function이 노출되는 위험이 존재한다(이는 거꾸로 말하면 API를 노출시키기 좋다라는 것임).       
 
-----------------

## gRPC
JSON-RPC는 JSON 형태로 주고받지만, gRPC는 Protocol-Buffer 형태로 주고받는다. 차이점은 아래와 같다.   

| 기능                 | gRPC                                   | JSON-RPC                        |
| -------------------- | -------------------------------------- | ------------------------------- |
| 계약                 | 필수(.proto)                           | 선택 사항(OpenAPI)              |
| 프로토콜             | HTTP/2                                 | HTTP                            |
| Payload              | Protobuf(소형, 이진), JSON도 사용 가능 | JSON(대형, 사람이 읽을 수 있음) |
| 규범                 | 엄격한 사양                            | 느슨함. 모든 HTTP가 유효합니다. |
| 스트리밍             | 클라이언트, 서버, 양방향               | 클라이언트, 서버                |
| 브라우저 지원        | 아니요(gRPC-웹 필요)                   | 예                              |
| 보안                 | 전송(TLS)                              | 전송(TLS)                       |
| 클라이언트 코드 생성 | 예                                     | OpenAPI + 타사 도구             |


> gRPC는 MSA 및 Docker-based application들의 RPC 설계를 단순화 시킴으로써 충분히 사용가치가 뛰어나다.
{: .prompt-info}