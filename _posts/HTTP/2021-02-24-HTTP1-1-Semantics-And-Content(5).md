---
title: 'HTTP 1.1 Reference - Semantics and Content (5) (최종 수정 날짜 : 2021-02-25)'
tags: Web HTTP1.1
categories: http
author: CY
key: HTTP 1.1 Reference - Semantics and Content (5)
---
# 참고 자료
* 이병록님이 번역하신 HTTP 1.1 Semantics and Content 번역본  
  [https://roka88.dev/106](https://roka88.dev/106)
* MDN Web Docs  
  [https://developer.mozilla.org/ko/docs/Web/HTTP](https://developer.mozilla.org/ko/docs/Web/HTTP)

이외 자료는 내용 중 링크 추가
##### 틀린 내용, 부족한 내용 꼭 지적 부탁드립니다!

# 7. Response Header Fields
응답 헤더 필드는 서버가 status-line에 배치된 정보를 넘어 응답에 대한 추가 정보를 전달할 수 있도록 한다. 
이러한 헤더 필드는 서버에 대한 정보, 대상 리소스에 대한 추가 액세스 또는 관련 리소스에 대한 정보를 제공한다.

각 응답 헤더 필드에는 정의된 의미가 있지만, 일반적으로 정확한 의미론은 요청 메서드 및/또는 응답 상태 코드의 의미에 의해 더욱 세분화될 수 있다.

## 7.1. Control Data
응답 헤더 필드는 상태 코드를 보충하거나 캐시를 지시하거나 클라이언트에게 다음은 어디로 가야 하는지 지시하는 제어 데이터를 제공할 수 있다. 
* Age
* Cache-Control
* Expires
* Date
* Location
* Retry-After
* Vary
* Warning

### 7.1.1 Origination Date
#### 7.1.1.1. Data/Time Formats
1995년 이전에는 서버가 타임스탬프를 통신하기 위해 일반적으로 사용하는 세 가지 다른 형식이 있었다. 
기존 구현과의 호환성을 위해 세 가지 모두 여기에 정의되어 있따. 선호하는 형식은 인터넷 메시지 형식에서 
사용하는 날짜 및 시간 규격의 fixed-length 및 single-zone 부분 집합이다.

HTTP-date = IMF-fixdate /obs-date
선호하는 형식의 예는
```
Sun, 06 Nov 1994 08:49:38 GMT; IMF-fixdate
```

사용되지 않는 두형식의 예는 다음과 같다.
```
Sunday, 06-Nov-94 08:49:38 GMT; obsolete format
Sun Nov 6 08:49:38 1994; ANSI C`s asctime() format
```
HTTP 헤더 필드에서 타임스탬프 값을 구문 분석하는 수신자는 세 가지 HTTP-date 형식을 모두 수용해야 한다.
발신자가 HTTP-date로 정의된 하나 이상의 타임스탬프를 포함하는 헤더 필드를 생성할 때, 발신자는 반드시 IMF-fixdate 형식으로 타임스탬프를 생성해야 한다.

HTTP 날짜 값은 UTC(Coordinated Universal Time)의 인스턴스로서의 시간을 나타낸다. 처음 두 형식은 UTC 이름의 이전 버진인 
"GMT"의 3글자 약어로 UTC를 나타내며, asctime 형식의 값은 UTC로 가장헌다. 로컬 클럭에서 HTTP 날짜 값을 생성하는 발신자는 시계를 
UTC에 동기화하기 위해 NTP 또는 이와 유사한 프로토콜을 사용해야 한다.
```
IMF-fixdate = day-name "," SP date 1 SP time-of-day SP GMT
            ;fixed length/zone/capitalization subset of the format
day-name = %x4D.6F.6E; :"Mon", case-sensitive
         / %x54.75.65; :"Tue", case-sensitive
         / %x57.65.64 ; "Wed", case-sensitive
         / %x54.68.75 ; "Thu", case-sensitive
         / %x46.72.69 ; "Fri", case-sensitive
         / %x53.61.74 ; "Sat", case-sensitive
         / %x53.75.6E ; "Sun", case-sensitive

date 1 = day SP month SP year
        ;e.g.,02 Jun 1982

day = 2DIGIT
month = %x4A.61.6E ; "Jan", case-sensitive
      / %x46.65.62 ; "Feb", case-sensitive 
      / %x4D.61.72 ; "Mar", case-sensitive 
      / %x41.70.72 ; "Apr", case-sensitive 
      / %x4D.61.79 ; "May", case-sensitive 
      / %x4A.75.6E ; "Jun", case-sensitive 
      / %x4A.75.6C ; "Jul", case-sensitive
      / %x41.75.67 ; "Aug", case-sensitive 
      / %x53.65.70 ; "Sep", case-sensitive 
      / %x4F.63.74 ; "Oct", case-sensitive 
      / %x4E.6F.76 ; "Nov", case-sensitive 
      / %x44.65.63 ; "Dec", case-sensitive

year = 4DIGIT

GMT = %x47.4D.54 ; "GMT", case-sensitive

time-of-day = hour ":" minute ":" second
              ; 00:00:00 - 23:59:60 (leap second)
hour = 2DIGIT 
minute = 2DIGIT 
second = 2DIGIT
```

쇠퇴한 형식(참고만 하자)
```
obs-date = rfc850-date / asctime-date

rfc850-date = day-name-l "," SP date2 SP time-of-day SP GMT 
date2 = day "-" month "-" 2DIGIT
      ; e.g., 02-Jun-82

day-name-l = %x4D.6F.6E.64.61.79 ; "Monday", case-sensitive
           / %x54.75.65.73.64.61.79 ; "Tuesday", case-sensitive
           / %x57.65.64.6E.65.73.64.61.79 ; "Wednesday", case-sensitive 
           / %x54.68.75.72.73.64.61.79 ; "Thursday", case-sensitive
           / %x46.72.69.64.61.79 ; "Friday", case-sensitive
           / %x53.61.74.75.72.64.61.79 ; "Saturday", case-sensitive
           / %x53.75.6E.64.61.79 ; "Sunday", case-sensitive
asctime-date = day-name SP date3 SP time-of-day SP year 
date3 = month SP ( 2DIGIT / ( SP 1DIGIT ))
      ; e.g., Jun 2
```

HTTP-date는 대소문자를 구분한다. 발신자는 문법에 SP로 구체적으로 포함된 공간을 초과하여 HTTP-date에 추가 공백을 생성해서는 안 된다.
day-name, day, month, year 및 time-of-day의 의미는 해당 이름과 Internet Message Format 구성에 대해 정의된 것과 동일 하다.

두 자릿수를 사용하는 rfc850-date 형식의 타임스탬프 값을 받는 사람은 2-digit가 가장 최근 연도를 나타내는 것으로 해석해야 한다.

타임스탬프 값의 수신자는 필드 정의에 의해 달리 제한되지 않는 한 타임스탬프를 구문 분석할 때 견고한 것을 권장한다. 
예를 들어, Internet Message Format에 의해 정의된 날짜 및 시간 사양을 생성할 수 있는 비 HTTP 소스에서 메시지를 HTTP를 통해 전달하는 경우가 있다.

참고 : date/time 스탬프 형식에 대한 HTTP 요구 사항은 프로토콜 스트림 내의 사용에만 적용된다. 사용자 프레젠테이션, 요청 기록 등에 이러한 형식을 사용할 필요는 없다.

#### 7.1.1.2. Date
"Date" 헤더 필드는 Origination Date Field와 동일한 의미를 가지며 메시지가 발생한 날짜와 시간을 나타낸다. 
필드 값은 HTTP-date이다.

```
Date = HTTP-date

예를 들어
Date: Tue, 15 Nov 1994 08:12:31 GMT
```

Date 헤더 필드가 생성될 떄, 발신자는 메시지 생성 날짜와 시간에 대한 가장 근사치로서 필드 값을 생성해야 한다. 
이론적으로, 날짜는 페이로드가 생성되기 바로 전의 순간을 나타내야 한다. 실제로 그 날짜는 메시지 작성 중에 언제라도 생성될 수 있다.

원서버는 UTC에서 현재 인스턴스의 합당한 근사치를 에제공할 수 있는 시계가 없는 경우 Date 헤더 필드를 보내서는 안 된다. 
응답이 상태 코드의 1xx(Informational) 또는 5xx(Server Error) 등급인 경우 원 서버는 Date 헤더 필드는 보낼 수 있다.
원서버는 다른 모든 경우 Date 헤더 필드를 전송해야 한다.

Date 헤더 필드 없이 응답 메시지를 수신하는 시계가 있는 수신자는 수신된 시간을 기록하고 해당 Date 헤더 필드가 캐시 되거나 
다운스트림에서 전달되는 경우 메시지의 헤더 부문에 해당 Date 헤더 필드를 추가해야 한다.

사용자 에이전트는 요청으로 Date 헤더 필드를 보낼 수 있지만, 서버에 유용한 정보를 전달하는 것으로 간주되지 않는 한 일반적으로 그렇게 하지 않는다. 
예를 들어, HTTP의 사용자 정의 어플리케이션은 서버가 사용자 에이전트와 서버 시계 간의 차이를 기반으로 사용자 요청에 대한 해석을 조정할 것으로 기대되는 경우 Date를 전달할 수 있다.

### 7.1.2. Location
"Location" 헤더 필드는 응답과 관련된 특정 리소스를 참조하기 위해 일부 응답에 사용된다. 
관계의 유형은 요청 메서드와 상태 코드 의미론의 조합에 의해 정의 된다.

```
Location = URI-reference
```

필드 값은 단일 URI-reference로 구성된다. 상대적 참조의 형태를 가지고 있을 때 유효한 요청 URI의 값을 가진다.

201(Created) 응답의 경우, Location 값은 요청에 의해 생성된 기본 리소스를 말한다. 
3xx (Redirection) 응답의 경우, Location 값은 요청을 자동으로 리디렉션하기 위한 기본 대상 리소스를 가르킨다.

3xx (Redirection) 응답에 제공된 Location 값에 Fragment(#)가 없는 경우, 사용자 에이전트는 그 값이 요청 대상을 생성하는 데 
사용된 URI 참조의 Fragment(#) (즉, 리디렉션은 원래 참조의 파편을 상속하는 경우)를 상속하는 것처럼 리디렉션을 처리해야 한다.

예를 들어 URI 참조 "http://www.example.org/~tim"에 대해 생성된 GET 요청은 헤더 필드를 포함하는 303(See Other) 응답을 받을 수 있다.
```
Location: /People.html#tim
```
사용자 에이전트가 "http://www.example.org/People.html#tim"으로 리디렉션됨을 암시한다.

마찬가지로, URI 참조 "http://www.example.org/index.html#larry"에 대해 생성된 GET 요청은 
다음과 같은 헤더 필드를 포함하는 301 (Moved Permanently) 응답을 초래할 수 있다.
```
Location: http://www.example.net/index.html
```
이는 사용자가 에이전트가 Fragment(#)를 보존하면서 "http://www.eample.net/index.html#larry"으로 리디렉션을 제안한다.

Location 값의 Fragment(#)가 적절하지 않은 상황이 있다. 예를 들어 201(Created) 응답의 Location 헤더 필드는 생성된 리소스와 관련된 URI를 제공하도록 되어 있다. 
참고: 일부 수신자는 유효한 URI 참조가 아닌 Location 필드에서 복구를 시도한다. 이 명세는 그러한 처리를 위임하거나 정의하지 않지만, 견고성을 위해 허용된다.

참고 : Content-Location 헤더 필드는 Content-Location 이 동봉된 표현에 해당하는 가장 구체적인 리소스를 지칭한다는 점에서 Loaction과 다르다. 
따라서 응답에 Loaction 및 Content-Location 헤더 필드가 모두 포함될 수 있다.

### 7.1.3. Retry-After
서버는 "Retry-After" 헤더 필드를 보내 사용자 에이전트가 후속 요청을 하기 전에 얼마나 기다려야 하는지 표시한다. 
503 (Service Unavailable) 응답과 함께 전송된 경우 Retry-After에는 클라이언트가 서비스를 사용할 수 없을 것으로 예쌍되는 기간이 표시된다. 
3xx (Redirection) 응답과 함께 전송되는 경우 Retry-After에는 리디렉션 요청을 실행하기 전에 사용자 에이전트가 대기하도록 요청하는 최소 시간이 표시된다.

이 필드의 값은 HTTP-date 또는 응답 수신 후 지연 시간(초)이 될 수 있다.

Retry-After = HTTP-date / delay-seconds

delay-seconds 값은 음이 아닌 소수 정수로 시간(초)을 나타낸다.

delay-seconds = 1*DIGIT

```
Retry-After: Fri, 31 Dec 1999 23:59:59 GMT 
Retry-After: 120 (2분 지연)
```

예제를 만들어 동작을 시켜보았다.

![Retry-After](/assets/images/http1-1SemanticsAndContent/RetryAfter.png)

Retry-After값을 5를 줘서 5초 이후에 리디렉션이 되도록 구현하였지만 동작을 하지 않았다. 
이유를 찾아보니 웹브라우저(크롬, 파이어폭스, IE 등)은 무시한다고한다.

### 7.1.4. Vary
응답의 "Vary" 헤더 필드는 메서드, Host 헤더 필드 및 요청 대상을 제외하고 
요청 메시지의 어떤 부분이 이 응답을 선택하고 나타내는 원서버의 프로세스에 영향을 미칠 수 있는 지 설명 한다. 
값은 단일 별표("*") 또는 헤더 필드 이름 목록(대소문자 구분 없음)으로 구성 된다.

```
Vary = "*" / 1 #field-name
```

"*" 필드 값은 요청에 대한 모든 것이 메시지 구문(클라이언트의 네트워크 주소) 외부에 있는 요소를 포함하여 
응답 representation을 선택하는데 역할을 할 수 있음을 나타낸다. 수신자는 요청을 원 서버로 전달하지 않고는 
이 응답이 이후 요청에 적합한지 여부를 결정할 수 없을 것이다.
프록시는 "*" 값을 가진 Vary 필드를 생성해서는 안 된다.

comma-separated로 구분된 이름 목록으로 구성된 Vary 필드 값은 선택 헤더 필드라고 알려진 
명명된 요청 헤더 필드가 representation을 선택하는 데 역할을 할 수 있음을 표시한다. 잠재적 선택 헤더 필드는 
이 명세에 의해 정의된 필드로 제한되지 않는다.

예를 들어 Vary가 포함된 응답은

```
Vary: accept-encoding, accept-language
```

원서버가 이 응답에 대한 내용을 선택하면서 요청의 Accept-Encoding 및 
Accept-Language 필드(또는 해당 필드 없음)를 결정 요인으로 사용했을 수 있음을 나타낸다.

원 서버는 두 가지 목적을 위해 필드 목록과 함께 Vary를 보낼 수 있다.

1. 캐시 수신자에게, 이후의 요청이 원래 요청과 동일한 값을 가지지 않는 한, 
이후의 요청을 만족 시키기 위해 이 응답을 사용해서는 안 된다는 것을 통지 한다.
즉, Vary는 저장된 캐시 항목으로 새 요청을 일치시키는 데 필요한 캐시 키를 확장한다.
2. 사용자 에이전트 수신자에게 이 응답은 컨텐츠 협상의 대상이며, 
나열된 헤더 필드에 추가 파라미터가 제공될 경우 후속 요청으로 다른 표현이 전송될 수 있음을 통지한다. 

- Vary 헤더를 이용하여 브라우저 캐시를 확인하려고했는데; 동작을 하지 않는다. 방법을 한번 찾아봐야할듯. 

## 7.2. Validator Header Fields
검증자 헤더 필드는 선택된 representation에 대한 메타 데이터를 전송한다. 
안전한 요청에 대한 응답에서 검증자 필드는 응답을 처리하는 동안 원서버에 의해 선택된 representation을 설명한다. 
상태 코드 의미에 따라 특정 응답에 대해 선택된 representation은 응답 페이로드로 동봉된 
representation과 반드시 동일하지 않는다는 점에 유의한다.

상태 변경 요청에 대한 성공적인 응답에서 검증자 필드는 요청을 처리한 결과 이전에 선택된 
representation을 대체한 새로운  representation을 설명한다.

예를 들어 201(Created) 응답의 ETag 헤더 필드는 새로 생성된 리소스의 representation에 대한 
entity-tag를 전달하여, 이후 조건부에 요청에 사용되어 "lost update" 문제를 방지할 수 있다.

* ETag
* Last-Modified

## 7.3. Authentication Challenges
인증 문제는 클라이언트가 향후 요청 시 인증 자격 증명을 제공할 수 있는 메커니즘을 나타낸다.

## 7.4. Response Context
나머지 응답 헤더 필드는 이후 요청에 사용할 수 있는 대상 리소스에 대한 자세한 정보를 제공 한다.
* Accept-Ranges
* Allow
* Server

### 7.4.1. Allow
"Allow" 헤더 필드에는 대상 리소스가 지원하는 것으로 알려진 메서드 집합이 나열된다. 
이 필드의 목적은 수신자에게 리소스와 관련된 유효한 요청 메서드를 알리는 것이다.

```
Allow = #method
Allow : GET, HEAD, PUT
```

허용되는 메서드의 실제 집합은 각 요청 시 원서버에 의해 정의된다. 
원서버는 405(Method Not Allow) 응답에서 Allow 필드를 생성해야 하며, 
다른 응답에서도 Allow 필드를 생성할 수 있다. 405 상태 코드에서 빈 Allow 필드 값은 리소스가 메서드를 허용하지 않음을 나타내며 
리소스가 일시적으로 비활성화된 경우를 나타낸다.

프록시는 Allow 헤더 필드를 수정해서는 안 되며, 일반적인 메시지 처리 규칙에 따라 처리하기 위해 표시된 모든 메서드를 이해할 필요는 없다.

### 7.4.2. Server
"Server" 헤더 필드에는 요청을 처리하기 위해 원서버가 사용하는 소프트웨어에 대한 정보가 들어 있는데, 
이 소프트웨어는 보고된 상호 운용성 문제의 범위를 식별하는 데 도움을 주고, 
특정 서버 제한을 피하기 위해 작업하거나 요청을 맞춤화하며, 
서버 또는 운영체제 사용과 관련된 분석을 위해 클라이언트가 종종 사용한다. 
원서버는 응답에 Server 필드를 생성할 수 있다.

```
Server = product *(RWS(product/comment))
Server: CERN/3.0 libwww/2.17
```
Server field-value는 하나 이상의 product 식별자로 구성되며, 각각 0개 이상의 주석이 뒤따르며 
원서버 소프트웨어와 그 중요한 하위 제품을 함께 식별한다. product 식별자는 원서버 소프트웨어를 식별하기 위한 중요도의 감소 순서로 나열된다.

원서버는 불필요한 세부 정보가 포함된 Server 필드를 생성해서는 안 되며, 서드 파티의 하위 제품 추가를 제한해야 한다. 
지나치게 길고 세부적인 Server 필드 값은 응답 대기 시간을 증가시키고 공격자가 알려진 보안 구멍을 더 쉽게 찾고 이용할 수 있는 
내부 구현 세부 정보를 잠재적으로 노출시킬 수 있다.

> 설명되지 않은 헤더들에 대해선 추후 문서를 읽어보면서 알아보도록 하겠다.  
> 1~7번까지의 설명을 알아보았다. 8번의 내용은 새로운 메서드를 등록하거나 상태 코드를 등록하는 방법에 대해서 설명하는데 현재 내 기준에선 기록하면서 만큼 알아볼 정도는 아니여서 추후에 필요하다면 학습하도록 하자.

