---
title: 'HTTP 1.1 Reference - Message Syntax and Routing (2) (최종 수정 날짜 : 2021-03-10)'
tags: Web HTTP1.1
categories: http
author: CY
key: HTTP 1.1 Reference - Message Syntax and Routing (2)
---
# 참고 자료
* 이병록님이 번역하신 HTTP 1.1 Message Syntax and Routing 번역본  
  [https://roka88.dev/106](https://roka88.dev/106)
* MDN Web Docs  
  [https://developer.mozilla.org/ko/docs/Web/HTTP](https://developer.mozilla.org/ko/docs/Web/HTTP)

이외 자료는 내용 중 링크 추가
##### 틀린 내용, 부족한 내용 꼭 지적 부탁드립니다!

# 3. Message Format
모든 HTTP/1.1 메시지는 인터넷 메시지 형식과 유사한 형식의 일련의 octest 시퀀스 뒤에 start-line으로 구성한다.
0 또는 여러 헤더필드 (집합적으로 "헤더" 또는 "헤더 부문"이라고 함), 헤더 부문이 끝을 나타내는 빈줄 및 선택적 메시지 본문으로 구성된다. 

```h
HTTP-message = start-line
               *(header-field CRLF)
               CRLF
               [ message-body ]

PUT /create_page HTTP/1.1     (start-line)
Host: localhost:8000          (header-field)
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Content-Type: text/html
Content-Length: 345
                              (CRLF)
Body line 1                   (message-body)
Body line 2
...
```

HTTP 메시지를 분석하는 일반적인 절차는 start-line을 읽고, 각 헤더 필드를 빈 행까지 필드 이름으로 
해시 테이블로 읽은 다음 분석된 데이터를 사용하여 메시지 본문이 필요한지 여부를 확인하는 것이다. 
메시지 본문이 표시된 경우, 메시지 본문 길이와 동일한 octet의 양을 읽거나 커넥션을 닫을 때까지 스트림으로 읽는다. 

수신자는 반드시 HTTP 메시지를 US-ASCII의 상위 집합인 인코딩에서 octet로 굿문 분석해야 한다. 
특정 인코딩에 관계 없이 HTTP 메시지를 유니코드 문자의 스트림으로 구문 분석하면 octet LF(%x0A)를 포함하는 유효하지 않은 
다중 바이트 문자 시퀀스를 처리하는 문자열 처리 라이브러리의 다양한 방식 때문에 보안 취약성이 발생한다. 
문자열 기반 분석기는 메시지 분석이 개별 필드를 분석 후 헤더 field-value와 같이 메시지에서 요소를 추출한 후에만 
프로토콜 요소 내에서 안전하게 사용할 수 있다.

> 옥텟이란 말이 계속 나오는데 옥텟(octet)이란 컴퓨팅에서 8개의 비트가 한곳에 모인것을 말하고 바이트와 같은 의미이다.

HTTP 메시지는 스트림으로 구문 분석하여 증분 처리하거나 다운 스트림으로 전송할 수 있습니다. 
그러나 일부 구현에서는 네트워크 효율성, 보안 검사 또는 페이 로드 변환을 위해 
메시지 전달을 버퍼링 하거나 지연시키기 때문에 수신자는 부분 메시지의 증분 전달에 의존할 수 없다.

> Incremental processing이라는 말이 나오는데 간단히 찾아본 결과 Incremental processing는 전체 데이터 집합을 
> 기존 데이터가 이미 처리 된 경우 다시 처리하는 대신 새로 추가 된 데이터만 처리하는 방법이다. 이미 처리 된 데이터를 재 처리하는 
> 오버 헤드를 제거하여 효율성을 향상시킨 처리 기술이다.

발신자는 start-line과 첫 번째 헤더 필드 사이에 공백을 보내면 안 된다. 
start-line과 첫 번째 헤더 필드 사이의 공백을 수신하는 수신자는 메시지를 유효하지 않은 것으로 거부하거나 
메시지를 더 이상 처리하지 않고 각 공백이 지정된 줄을 소비해야 한다. 
(i.e., 공백이 오는 모든 후속 줄과 함께, 올바르게 형성된 헤더필드가 수신되거나 헤더 부문이 끝날 때까지, 전체 라인을 무시한다)

요청에 이러한 공백이 있으면 서버가 해당 필드를 무시하도록 속이거나 새 요청으로 처리하려고 시도한 것일 수 있으며, 
요청 체인의 다른 구현에서 동일한 메시지를 다르게 해석할 경우 보안 취약성이 발생할 수 있다. 
마찬가지로, 응답에 이러한 공백이 있으면 일부 클라이언트가 무시하거나 다른 사용자가 구문 분석을 중지할 수 있다.

## 3.1. Start Line
HTTP 메시지는 클라이언트에서 서버로 요청하거나 서버에서 클라이언트로 응답하는 것일 수 있다. 
동기적으로, 두 유형의 메시지는 start-line(요청의 경우) 또는 status-line(응답의 경우)과 메시지 본문의 길이를 
결정하기 위한 알고리즘에서만 다르다.

이론적으로 클라이언트는 요청을 수신할 수 있고 서버는 응답을 수신할 수 있으며, 이를 서로 다른 start-line 형식으로 구분할 수 있지만, 
실제로 서버가 요청만 예상하도록 구현된다. (응답은 알 수 없거나 잘못된 요청 메서드로 해석됨). 클라이언트는 응답만을 예상하도록 구현 된다.

``` h
start-line = request-line / status-line
```

### 3.1.1. Request Line
request-line은 method 토큰을 시작으로 공백(SP), request-target, 공백, 프로토콜 버전, 그리고 CRLF를 끝으로 한다.
```h
request-line = method SP request-target SP HTTP-version CRLF
```

method 토큰(GET, POST, HEAD 등...)은 대상 리소스를 실행하기 위한 요청 메서드를 표시한다. 요청 메서드는 대 소문자를 구분한다.
```h
method = token 
```

request-target은 요청을 적용할 대상 리소스를 식별한다.

세 구성 요소에는 공백이 허용되지 않기 때문에 일반적으로 수신자는 공백을 분할하여 request-line을 구성 요소 부분으로 분할한다. 
불행하게도 일부 사용자 에이전트는 하이퍼 텍스트 참조에서 발견된 공백을 제대로 인코딩하거나 제외하지 못하여 허용되지 않는 문자가 request-target으로 전송된다.

유효하지 않은 request-line의 수신자는 400(Bad Request) 오류 또는 301(Moved Permanently) 리다이렉트로 응답해야 하며, 
request-target은 적절히 인코딩되어야 한다. 유효하지 않은 request-line은 요청 체인을 따라 보안 필터를 우회하도록 의도적으로 조작될 수 있으므로 
수신자는 리다이렉트 없이 요청을 자동 수정하고 처리해서는 안 된다.

HTTP는 request-line의 길이에 대해 미리 정의된 제한을 두지 않는다. 구현한 것보다 더 긴 메서드를 수신한 서버는 
501(Not Implemented) 상태 코드로 응답해야 한다. request-target을 구문 분석하려는 URI보다 긴 414(URI Too Long) 상태 코드로 응답해야 한다.

실제로 request-line의 길이에 대한 다양하고 특별한 목적의 제한 사항이 발견된다. 모든 HTTP 발신자와 수신자는, 
최소 8000 octet 길이의 request-line을, 지원하는 것을 권장한다.

### 3.1.2. Status Line
응답 메시지의 첫 번째 줄은 status-line이며, 프로토콜 버전, 공백(SP), status-code, 공백, status-code를 설명하는 빈 텍스트 구문, 
그리고 끝으로 CRLF이다.

```h
status-line = HTTP-version SP status-code SP reason-phrase CRLF
```

status-code 요소는 서버가 클라이언트의 해당 요청을 이해하고 충족하려고 시도한 결과를 설명하는 세 자리의 정수 코드이다. 
응답 메시지의 나머지 부분은 해당 상태 코드에 대해 정의된 의미론을 고려하여 해석된다.

```h
status-code = 3DIGIT
```

reason-phrase 요소는 숫자 상태 코드와 관련된 텍스트 설명을 제공하는 유일한 목적으로 존재하며, 
대부분 대화형 텍스트 클라이언트에서 더 자주 사용되었던 이전의 인터넷 애플리케이션 프로토콜에 대한 경의를 배제한다. 
클라이언트는 reason-phase 내용을 무시해야 한다.

```h
reason-phrase = *(HTAB/SP/VCHAR/obs-test)
```
> reason-phrase 는 만약 상태코드가 100이라면 Continue, 200이라면 OK와 같이 상태 코드를 간략하게 설명해주는 문자열이다. 


## 3.2. Header Fields
각 헤더 필드는 대소문자를 구분하지 않는 필드 이름 뒤에 콜론(":"), 선택적 앞의 공백(OWS), 필드값, 선택적 뒤의 공백(OWS)으로 구성된다.

```h
header-field = field-name ":" OWS field-value OWS

field-name = token
field-value = *(field-content / obs-fold)
field-content = field-vchar [1*(SP/HTAB) field-vchar]
field-vchar = VCHAR / obs-test
obs-fold = CRLF 1*(SP / HTAB)
          ;obsolete line folding
```

field-name 토큰은 해당 field-value를 헤더 필드에서 정의한 의미론을 갖는 것으로 레이블을 지정한다. 
예를 들어 Date 헤더 필드는 메시지의 시작 타임 스탬프를 포함하는 필드로 정의 된다.

### 3.2.1. Field Extensibility
헤더 필드는 완전히 확장 가능하다. 새 필드 이름의 도입, 각각 새로운 의미론을 정의하는데 제한이 없다. 
또한 지정된 메시지에 사용되는 헤더 필드의 개수에도 제한이 없다. 기존 필드는 이 명세의 각 부분과 이 문서 모음 외 다른 많은 명세에 정의되어 있다. 

새로운 헤더 필드는 수신자가 이해할 때 이전에 정의된 헤더 필드의 해석을 재정의하거나 향상시키거나, 
요청 평가에 대한 전제 조건을 정의하거나, 응답의 의미를 구체화하도록 정의할 수 있다. 

field-name이 Connection 헤더 필드에 나열되어 있지 않거나 프록시가 이러한 필드를 
차단하거나 변환하도록 특별히 구성되어 있지 않는 경우, 프록시는 인식하지 못하는 헤더 필드를 전달해야 한다. 
다른 수신자는 인식할 수 없는 헤더 필드를 무시해야 한다. 이러한 요구 사항을 통해 배치된 중개자의 사전 업데이트를 요구하지 않고도 
HTTP의 기능을 강화할 수 있다.

### 3.2.2. Field Order
다른 필드 이름과 헤더 필드가 수신하는 순서는 중요하지 않다. 
그러나 요청 시 Host, 응답 시 Date 등 제어 데이터가 포함된 헤더필드를 먼저 보내 구현 시 메시지를 처리하지 않을 시기를 결정하는 것이 좋다. 
서버는 헤더 필드에는 조건, 인증 자격 증명 또는 요청 처리에 영향을 미칠 수 있는 의도적으로 잘못된 중복 헤더 필드가 포함될 수 있으므로 
전체 요청 헤더 부문을 받을 때 까지 대상 리소스에 요청을 적용하면 안 된다. 발신자는 헤더 필드의 전체 필드 값이 쉼표로 구분된 목록으로 
정의되어 있거나 헤더필드가 잘 알려진 예외가 아닌 한, 메시지에 동일한 필드 이름을 가진 헤더 필드를 여러개 생성해서는 안 된다. 

수신자는 각 후속 필드 값을 쉼표로 구분하여 결합된 필드 값에 순서대로 추가하여 메시지의 의미론을 변경하지 않고 동일한 필드 이름을 가진 여러 헤더 필드를 
하나의 "field-name : field-value" 쌍으로 결합할 수 있다. 따라서 동일한 필드 이름의 헤더 필드를 수신하는 순서는 결합된 필드 값을 해석하는 데 중요하다. 
메시지를 전달할 때 프록시는 이러한 필드 값의 순서를 변경해서는 안 된다. 