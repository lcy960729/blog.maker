---
title: 'Spring Security의 기능 (최종 수정 날짜 : 2021-02-22)'
tags: Web SpringSecurity Spring
categories: spring-security
author: CY
key: Spring Security의 기능
---
# 참고 자료
* 스프링 시큐리티 레퍼런스  
  [https://docs.spring.io/spring-security/site/docs/5.4.2/reference/html5/](https://docs.spring.io/spring-security/site/docs/5.4.2/reference/html5/)


이외 자료는 내용 중 링크 추가
##### 틀린 내용, 부족한 내용 꼭 지적 부탁드립니다!

# 1. 스프링 시큐리티의 기능
스프링 시큐리티는 인증, 권한 부여, 및 보호에 대한 종합적인 지원을 하며 단순하게 사용하기 위해 다른 라이브러리와의 통합을 제공한다.

## 1.1. 인증 (Authentication)
인증은 특정 리소스에 접근하려는 주체의 신원을 확인하는 것이다. 사용자를 인증하는 일반적인 방법은 사용자에게 이름과 비밀번호를 입력하도록 요구하는것이다. 인증이 수행되면 사용자의 신원(아이디와 비밀번호)을 알게 되며 신원을 인증 할 수 있게 된다. 스프링 시큐리티는 사용자 인증을 위한 기능을 내장하여 제공한다. 

## 1.2. Password Storage 
스프링 시큐리티의 PasswordEncoder 인터페이스는 비밀번호를 안전하게 저장할 수 있도록 비밀번호의 단방향 변환을 수행하는데 사용된다. PasswordEncoder는 단방향 변환이므로 양방향이 필요한 경우(데이터 베이스 인증에 사용되는 자격 증명 저장)에는 사용할 수 없다. 일반적으로 PasswordEncoder는 인증시 사용자가 제공한 암호와 비교해야하는 암호를 저장하는데 사용 된다. 

### Password Storage History
처음의 암호는 일반 텍스트로 저장되었다. 텍스트로 저장하던 때의 암호는 데이터 저장소에 저장 되었으며 이 저장소에 접근 하기위해선 자격 증명이 필요했기 때문에 개발자들은 충분히 안전하다고 생각했다. 그러나 악의적인 사용자가 SQL Injection과 같은 공격을 사용하여 대규모의 사용자 정보를 얻는 방법이 나타났고 이후부터 암호를 더 강력히 보호 해야함을 깨달았다.

이후 SHA-256과 같은 단방향 해시를 통해 암호를 저장하도록 권장되었다. 해시로 주어진 암호는 전수공격하여 계산하기에 큰 어려움이 있었기 때문에 안전하다 생각하였다(현재 하드웨어 기준으론 아님). 하지만 해시를 이용한 방법도 Rainbow Tables(해시 함수로 변환 가능한 모든 해시 값을 저장시켜 놓은 조회 테이블)로 인해 해시로 저장된 비밀번호만 있으면 원래의 비밀번호를 추출할 수 있는 문제가 생겼다.

개발자들은 이러한 Rainbow Tables의 효과를 막기 위해 개발자는 솔트된 암호를 사용하였다. 솔트된 암호란 해시 함수에 대한 입력으로 암호만 사용하는 대신 모든 사용자의 암호에 대해 임의의 바이트(솔트)가 생성 된다. 솔트와 입력된 사용자의 비밀번호는 해시 함수를 통해 고유한 해시값을 생성하게 된다. 사용자가 인증을 시도하면 사용자 입력 암호와 솔트를 이용하여 해시 함수를 통해 해시 값을 만들어낸다. 만들어진 해시 값을 저장소에 저장된 고유한 해시값과 비교하게 된다. 솔트 값은 모든 암호마다 다르기 때문에 RainbowTables를 이용하여 비밀번호를 복호화 할 수 없게 된다.

현재는 이러한 해시 함수 또한 하드웨어의 발전으로 안전하지 않게 되었다(SHA-256을 초당 백만, 천만번 씩 수행할 수 있게 됨). 개발자들은 Adaptive one-way function을 이용하여 암호를 저장하는 방식을 발전하였다. Adaptive one-way function을 사용하는 암호의 유효성 검사는 의도적으로 하드웨어 리소스를 사용하여 암호화 시간을 조절 할 수 있다. 암호화 작업 시간은 1초가 적당하다. Spring Security는 사용자가 직접 암호화 작업 시간을 정의 하도록 하였다. 스프링 시큐리티가 제공하는 Adaptive one-way function을 사용하는 암호화 방식은 bcrypt, PBKDF2, scrypt, argon2가 있다.

위에 소개한 내용으로 스프링 시큐리티가 왜 이러한 암호화 방식을 채택했는지 이해 됐을것이다. 이후엔 스프링 시큐리티가 PasswordEncoder에 대해서 알아보겠다. 

### 1.2.1. DelegatingPasswordEncoder
스프링 시큐리티 5.0 이전에는 기본 PasswordEncoder가 일반 텍스트 암호가 필요한 NoOpPasswordEncoder였다. 그러나 Password Storage History에 따라서 기본 PasswordEncoder는 BCryptPasswordEncoder를 사용할 수도 안할 수도 있다. 기본 PasswordEncoder가 변경됨에 따라서 3가지 큰 문제를 일으키는데 첫번째로 이전 암호 인코딩을 사용하는 많은 프로그램이 있어 쉽게 마이그레이션을 할 수 없다. 두번째, 암호 저장에 대한 모범 사례가 다시 변경 된다. 세번째 프레임 워크로서 자주 변경이 일어나면 안된다. 이 문제를 해결하기 위해 DelegatingPasswordEncoder를 도입하였고 DelegatingPasswordEncoder는 아래와 같은 기능으로 위의 문제들을 해결하였다. 첫번째로 현재 암호 저장소가 권장하는 방식을 사용하여 인코딩 되는지 확인한다. 두번째는 최신 및 레거시 형식의 암호 유효성 검사를 허용한다. 세번째는 향후 인코딩 업그레이드를 허용한다.

DelegatingPasswordEncoder는 PasswordEncoderFactories를 이용하여 인스턴스를 생성할 수 있다.

##### 기본 DelegatingPasswordEncoder를 생성하는 방법이다.
```java
PasswordEncoder passwordEncoder =
    PasswordEncoderFactories.createDelegatingPasswordEncoder();
```

##### 커스텀 DelegatingPasswordEncoder를 생성하는 방법이다.
```java
String idForEncode = "bcrypt";
Map encoders = new HashMap<>();
encoders.put(idForEncode, new BCryptPasswordEncoder());
encoders.put("noop", NoOpPasswordEncoder.getInstance());
encoders.put("pbkdf2", new Pbkdf2PasswordEncoder());
encoders.put("scrypt", new SCryptPasswordEncoder());
encoders.put("sha256", new StandardPasswordEncoder());

PasswordEncoder passwordEncoder =
    new DelegatingPasswordEncoder(idForEncode, encoders);
```

#### Password Storage Format
```
{id}encodePassword
```
id는 PasswordEncoder의 식별자이고 encodedPassword는 인코딩된 password를 뜻한다. id를 찾을 수 없으면 null로 반환된다.

#### Password Matching
매칭은 {id} 나 생성자에서 제공된 PasswordEncoder의 ID 매핑을 기반으로 수행된다. 패스워드나 맵핑되지 않은 ID를 사용하여 match를 호출하면 IllegalArgumentException이 발생한다. 이러한 오류를 예방하기 위해서 `DelegatingPasswordEncoder.setDefaultPasswordEncoderForMatches (PasswordEncoder)`
을 이용하여 기본값을 지정할 수 있다. 또한 암호가 저장되는 방식을 파악하여 아래처럼 명시적으로 제공을 해주면 된다.
`$2a$10$dXJ3SW6G7P50lGmMkkmwe. ...` => `{bcrypt}$2a$10$dXJ3SW6G7P50lGmMkkmwe. ...`

##### withDefaultPasswordEncoder Example
```java
User user = User.withDefaultPasswordEncoder()
  .username("user")
  .password("password")
  .roles("user")
  .build();
System.out.println(user.getPassword());
// {bcrypt}$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG
```

##### withDefaultPasswordEncoder Reusing the Builder
```java
UserBuilder users = User.withDefaultPasswordEncoder();
User user = users
  .username("user")
  .password("password")
  .roles("USER")
  .build();
User admin = users
  .username("admin")
  .password("password")
  .roles("USER","ADMIN")
  .build();
```
위 두 예제는 암호를 해시하지만 암호는 여전히 메모리와 컴파일된 소스에 노출되고 있다.
이러한 문제는 큰 보안 사고로 이어질 수 있으므로 실제 프로덕션일 경우는 암호를 외부에서 해시 하여야 한다. 외부에서 인코딩하는 방법 중 하나를 아래에 설명하겠다.

#### Encode with Spring Boot CLI
비밀번호를 외부에서 해시하는 방법 중 쉬운 방법으로는 Spring Boot CLI를 사용한다. 

1. 아래처럼 `DelegatingPasswordEncoder`로 암호화할 암호를 Spring CLI에 아래와 같이 입력 후 인코딩된 암호를 생성한다.  
![1.png](/assets/images/2020-02-04-Spring-Security의-기능/1.png)

2. 반환 되는 암호를 프로젝트의 application.properties에 user 비밀번호에 추가해 준다.  
![2.png](/assets/images/2020-02-04-Spring-Security의-기능/2.png)

1. 이후 컨트롤러를 생성후 아무 API을 만들고 요청을 하면 아래와 같은 로그인 화면이 나타나고 user의 이름과 패스워드를(pas1234) 입력한다.  
![3.png](/assets/images/2020-02-04-Spring-Security의-기능/3.png)

4. 정상적으로 로그인이 되며 인증이 진행된걸 확인할 수 있다.  
![4.png](/assets/images/2020-02-04-Spring-Security의-기능/4.png)

---
아래에 소개하는 인코더들은 암호 크래킹의 내성을 위해 의도적으로 느리게 설계되어있으며 약 1초정도 걸리도록 조정하는것이 바람직하다. 자체 시스템에서 매개 변수를 조정하여 테스트하여 1초가 걸리도록 하면 된다. 알고리즘이 궁금하다면 알아서 찾아보도록...

### 1.2.2. BCryptPasswordEncoder
BCryptPasswordEncoder는 bdcrypt 알고리즘을 사용하여 암호를 해시한다.
```java
BCryptPasswordEncoder encoder = new BCryptPasswordEncoder(16);
String result = encoder.encode("myPassword");
assertTrue(encoder.matches("myPassword", result));
```
### 1.2.3. Argon2PasswordEncoder
Argon2PasswordEncoder는 Argon2 알고리즘을 사용하여 암호를 해시한다.
```java
Argon2PasswordEncoder encoder = new Argon2PasswordEncoder();
String result = encoder.encode("myPassword");
assertTrue(encoder.matches("myPassword", result));
```
### 1.2.4. Pbkdf2PasswordEncoder
Pbkdf2PasswordEncoder는 PBKDF2 알고리즘을 사용하여 암호를 해시한다.
```java
Pbkdf2PasswordEncoder encoder = new Pbkdf2PasswordEncoder();
String result = encoder.encode("myPassword");
assertTrue(encoder.matches("myPassword", result));
```
### 1.2.5. SCryptPasswordEncoder
SCryptPasswordEncoder SCrypt 알고리즘을 사용하여 암호를 해시한다.
```java
SCryptPasswordEncoder encoder = new SCryptPasswordEncoder();
String result = encoder.encode("myPassword");
assertTrue(encoder.matches("myPassword", result));
```
### 1.2.6. Other PasswordEncoders
이전 버전과의 호환성을 위해 구현된 PasswordEncoder들이 있다. 더 이상 안전하지 않기 때문에 사용되지 않는다.

### 1.2.7. Password Storage Configuration
NoOpPasswordEncoder를 사용하기 위해선 Spring bean으로 등록하여 사용한다.
```java
@Bean
public static NoOpPasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance();
}
```
## 5.2. Protection Against Exploits (악용에 대한 보호)
스프링 시큐리티는 일반적인 악용에 대한 방어를 제공한다. 가능하면 보호 기능은 기본적으로 활성화 되어 있다. 

### 5.2.1. Cross Site Request Forgery (CSRF)
스프링 시큐리티는 CSRF 공격을 보호하는 기능을 제공한다.

#### What is a CSRF Attack?
은행 웹 사이트에서 현재 로그인 한 사용자의 금액을 다른 은행 계좌로 이체할 수 있는 양식을 제공한다고 가정하자.
```html
<form method="post" action="/transfer">
  <input type="text" name="amount"/>
  <input type="text" name="routingNumber"/>
  <input type="text" name="account"/>
  <input type="submit" value="Transfer"/>
</form>
```
HTTP 요청은 아래와 같다
```
POST /transfer HTTP/1.1
Host: bank.example.com
Cookie: JSESSIONID=randomid
Content-Type: application/x-www-form-urlencoded

amount=100.00&routingNumber=1234&account=9876
```
다음은 은행 웹사이트에 인증한 것처럼 가장한다. 은행 사이트에서 로그아웃하지 않고 악의적인 웹사이트를 접속한다. 악의적인 웹 사이트는 아래와 같다.
```html
<form method="post" action="https://bank.example.com/transfer">
  <input type="hidden" name="amount" value="100.00"/>
  <input type="hidden" name="routingNumber" value="evilsRoutingNumber"/>
  <input type="hidden" name="account" value="evilsAccountNumber"/>
  <input type="submit" value="Win Money!"/>
</form>
```
아마도 의심이 없는 사람이라면 돈을 벌기 위해 제출 버튼을 누를것이고 이 과정에서 의도하지 않은 evilsAccountNumber로 100달러를 보내게 될것이다. 이것은 악의적인 사이트가 사용자의 쿠키를 볼 수는 없지만 은행과 관련된 쿠키는 여전히 요청과 함께 전송되기 때문에 가능하게 된다.

최악의 경우 이 프로세스가 javaScript를 이용하여 자동화 되었다면 접속과 동시에 송금이 될것이다. 또한 [XSS](https://ko.wikipedia.org/wiki/%EC%82%AC%EC%9D%B4%ED%8A%B8_%EA%B0%84_%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8C%85)공격으로 실제 정직한 은행 사이트를 방문 할 때도 악의적인 송금을 발생시킬 수 있다. 

#### Protecting Against CSRF Attacks
CSRF 공격이 가능한 이유는 피해자 웹 사이트의 HTTP 요청과 공격자 웹 사이트의 요청이 정확히 동일하기 때문이다. 즉, 악의적인 웹 사이트에서 오는 요청을 거부하고 은행 웹 사이트에서 오는 요청을 허용 할 방법이 없다. CSRF 공격으로부터 보호하려면 요청에 악의적인 사이트가 제공 할 수 없는 것이 있는지 확인하여 두 요청을 구분할 수 있어야 한다.
스프링은 CSRF 공격으로부터 보호하기 위해 아래 두가지 메커니즘을 제공한다. 
* Synchronizer Token Pattern
* 쿠키에 SameSite Attribute을 지정

#### Safe Methods Must be Idempotent
CSRF에 대한 보호가 작동하려면 어플리케이션이 safe한 메소드가 멱등성을 보장해야한다. 
(멱등성이란 어떠한 연산을 여러번 하더라도 결과가 달라지지 않는 성질이다.)
HTTP 메서드 중 GET, HEAD, OPTIONS, TRACE를 사용하는 요청은 리소스의 상태를 변경해서는 안된다.

#### Synchronizer Token Pattern
CSRF 공격으로부터 보호하는 가장 잘알려진 방법은 Synchronizer Token을 사용하는 것이다. 이 방법은 각 HTTP 요청에 세션 쿠키 외 CSRF 토큰이라는 임의 생성 값을 넣어 보내는 것이다.
HTTP 요청이 제출되면 서버는 예상되는 CSRF 토큰을 찾아 HTTP 요청의 CSRF 토큰과 비교한다. 값이 일치하지 않으면 해당 요청을 거부한다.
Synchronizer Token 방법의 핵심은 CSRF 토큰이 브라우저에 의해 자동으로 포함되지 않는 HTTP 요청의 일부에 있어야한다는 것이다(쿠키에 CSRF 토큰이 있으면 안됨 이후 설명). 예를 들어 HTTP 매개변수 또는 HTTP 헤더에 실제 CSRF 토큰을 요구하면 CSRF 공격으로부터 보호된다. 쿠키는 브라우저의 HTTP 요청에 자동으로 포함되므로 쿠키에 CSRF 토큰을 요구하는 것은 동작하지 않는다.
리소스 상태를 업데이트하는 HTTP 요청에 대해 CSRF 토큰만 필요하도록 할 수 있다. 이것이 동작하려면 우리는 HTTP 메소드가 멱등성을 보장해야한다. 이는 외부 사이트의 링크를 사용하여 당사 웹사이트로의 링크를 허용하기 때문에 사용성을 향상 시킨다. 또한 토큰이 유출 될 수 있으므로 HTTP GET에 토큰을 포함하지 않아야 한다.

Synchronizer Token Pattern을 사용하는 예제를 확인해보자
``` html
<form method="post" action="/transfer">
  <input type="hidden" name="_csrf" value="4bfd1575-3ad1-4d21-96c7-4ef2d9f86721"/>
  <input type="text" name="amount"/>
  <input type="text" name="routingNumber"/>
  <input type="hidden" name="account"/>
  <input type="submit" value="Transfer"/>
</form>
```
양식에 CSRF 토큰값이 숨겨져 있다. Same Origin Policy이 악의적 사이트가 응답을 읽을 수 없도록 보장하기 떄문에 외부 사이트는 CSRF 토큰을 읽을 수 없다.
```
POST /transfer HTTP/1.1
Host: bank.example.com
Cookie: JSESSIONID=randomid
Content-Type: application/x-www-form-urlencoded

amount=100.00&routingNumber=1234&account=9876&_csrf=4bfd1575-3ad1-4d21-96c7-4ef2d9f86721
```
HTTP 요청서에 _csrf 매개 변수가 포함되어 있는것을 확인할 수 있다. 악의적인 웹사이트는 _csrf 매개변수에 대한 올바른 값을 제공 할 수 없으며 서버가 CSRF 토큰을 예상 된 CSRF 토큰과 비교할 때 전송이 실패한다.

#### SameSite Attribute
CSRF 공격으로 부터 보호하는 새로운 방법은 쿠키에 SameSite 속성을 지정하는 것이다. 서버는 쿠키를 설정할 때 SameSite 속성을 지정하여 외부 사이트에서 올 때 쿠키를 보내지 않아야 함을 나타낼 수 있다.
> 스프링 시큐리티는 세션 쿠키 생성을 직접 제어하지 않으므로 SameSite 속성에 대한 지원을 제공하지 않는다. 스프링 세션은 서블릿 기반 어플리케이션에서 SameSite 속성에 대한 지원을 제공한다.

SameSite 속성이 있는 HTTP 응답 헤더를 보자
```
Set-Cookie: JSESSIONID=randomid; Domain=bank.example.com; Secure; HttpOnly; SameSite=Lax
```
SameSite 속성의 값은 아래와 같다.
* Strict - 지정된 경우 동일한 사이트에서 들어오는 모든 요청에 쿠키가 포함된다. 동일한 사이트가 아니라면 쿠키가 HTTP 요청에 포함되지 않는다
* Lax - 동일한 사이트거나 요청이 최상위 수준 탐색에서 오고 메서드가 멱등성인 경우 지정된 쿠키가 전송된다. 그렇지 않으면 쿠키가 HTTP 요청에 포함되지 않는다.
  <a href>, <link href>, <form method=get>은 예외 나머지는 Strict와 동일.

SameSite 속성을 사용하여 은행 송금 예제를 보호하는 방법을 알아보자. 은행 어플리케이션은 세션 쿠키에 SameSite 속성을 지정하여 CSRF로 부터 보호 할 수 있다.
세션 쿠키에 SameSite 속성이 설정되어 있으면 브라우저는 은행 웹 사이트에서 오는 요청과 함께 JSESSIONID 쿠키를 계속 보낸다. 그러나 악의적인 웹 사이트에서 들어오는 전송 요청은 JSESSIONID 쿠키를 보내지 않는다. 세션이 더 이상 악의적인 웹사이트에서 오는 전송 요청에 존재하지 않으므로 애플리케이션은 CSRF 공격으로부터 보호된다. 

SameSite 속성을 Strict로 설정하면 더 강력한 방어가 제공되지만 사용자에게 혼란을 줄 수 있다. https://social.example.com에서 호스팅 되는 소셜 미디어 사이트에 로그인 되어 있는 사용자를 생각해보자. 사용자는 https://email.example.org에서 소셜 미디어 사이트에 대한 링크가 포함된 이메일을 받는다. 사용자가 링크를 클릭하면 소셜 미디어 사이트에 대한 인증을 받을 것으로 기대한다. 하지만 SameSite 속성이 Strict이면 쿠키가 전송되지 않으므로 사용자가 인증되지 않는다.

또 다른 고려 사항은 SameSite 속성이 사용자를 보호하려면 브라우저가 SameSite 속성을 지원해야한다. 이러한 이유로 CSRF 공격에 대한 보호로 유일한 방법을 사용하는것보다 여러 방법을 사용하는것이 권장된다.

#### When to use CSRF protection
일반 사용자가 브라우저에서 처리할 수 있는 모든 요청에 대해 CSRF 보호가 되야 한다. 브라우저가 아닌 클라이언트에서 사용하는 서비스만 만드는 경우 CSRF 보호를 비활성화 할 수 있다. (세션을 사용하지 않거나 브라우저를 사용하지 않으면 쿠키를 사용할 일이 없기 때문.)

#### CSRF protection and JSON
JSON 요청에 영향을 줄 수 있는 CSRF 위협이 있으므로 주의해야한다. 예를 들어 악의적인 사용자는 아래와 같이 CSRF를 만들 수 있다.
```html
<form action="https://bank.example.com/transfer" method="post" enctype="text/plain">
    <input name='{"amount":100,"routingNumber":"evilsRoutingNumber","account":"evilsAccountNumber", "ignore_me":"' value='test"}' type='hidden'>
    <input type="submit"
        value="Win Money!"/>
</form>
```
위처럼 name에 json을 양식을 작성하고 value값과 연결되는 =을 없애기 위해 `"ignore_me":"' value='test"}'`이런식으로 끝나도록 `<input >`을 작성한다.
결과 값은 아래와 같은 JSON 구조가 된다.
```
{ "amount": 100,
"routingNumber": "evilsRoutingNumber",
"account": "evilsAccountNumber",
"ignore_me": "=test"}
```
어플리케이션이 Content-Type의 유효성을 검사하지 않으면 이 악용에 노출된다. 설정에 따라 Content-Type을 검증하는 Spring MVC 프로그램은 아래와 같이 .json으로 끝나도록 URL 접미사를 업데이트하여 악용할 수 있다.
```html
<form action="https://bank.example.com/transfer.json" method="post" enctype="text/plain">
    <input name='{"amount":100,"routingNumber":"evilsRoutingNumber","account":"evilsAccountNumber", "ignore_me":"' value='test"}' type='hidden'>
    <input type="submit"
        value="Win Money!"/>
</form>
```

#### CSRF and Stateless Browser Applications
어플리케이션이 stateless 하더라도 CSRF의 공격에 대해 안전한것은 아니다. 실제로 사용자가 특정 요청에 대해 웹브라우저에서 어떠한 작업도(SameSite 쿠키를 이용한다거나 Synchronizer Token과 같은 작업) 수행할 필요가 없다면 CSRF 공격에 여전히 취약 할 수 있다.
예를 들어 JSESSIONID 대신 인증을 위해 모든 상태를 포함하는 사용자 지정 쿠키를 사용하는 어플리케이션이 있다. CSRF 공격이 발생하면 이전 예제에서 JSESSIONID 쿠키가 전송 된 것과 동일한 방식으로 사용자 지정 쿠키가 요청과 함께 전송된다. 이런 어플리케이션은 CSRF 공격에 취약하다.

#### CSRF Considerations
CSRF 공격에 대한 보호를 구현할 때 고려해야 할 몇 가지 특별한 고려사항이 있다.

#### Logging In
위조 된 로그인 요청으로부터 보호하려면 로그인 HTTP 요청을 CSRF 공격으로부터 보호해야한다. 위조 로그인 요청으로부터 보호하여 악의적 인 사용자가 피해자의 민감한 정보를 읽을 수 없도록 해야한다. 로그인시 공격은 아래와 같은 방식으로 이루어진다.

* 악의적인 사용자는 악의적인 사용자의 자격 증명을 사용하여 CSRF 로그인을 수행한다. 피해자는 이제 악의적인 사용자로 인증된다.(로그인을 위해 필요한 자격 증명을 악의적인 사용자의 것을 이용한다는 것이다. 여기서 설명하는 자격 증명은 csrf토큰이나 쿠키의 samesite 속성과 같이 HTTP 요청이 올바른 곳에서 요청 됐다는것을 나타낼수 있는 정보를 나타내는것 같다.)
* 이후 악의적인 사용자가 피해자를 속여 악의적인 웹 사이트를 방문하도록 하고 민감한 정보를 입력하도록 한다.
* 정보는 악의적인 사용자의 계정과 연결되어 있으므로 악의적인 사용자는 자신의 자격 증명으로 로그인하여 피해자의 민감한 정보를 볼 수 있다. (악의적인 사용자의 자격 증명으로 인해 악의적인 사이트에서도 로그인 요청이 가능하게 됨.)

로그인 HTTP 요청이 CSRF 공격으로부터 보호되도록 하는데 발생할 수 있는 문제는 세션 타임 아웃 때문에 사용자가 요청이 거부되는 경험을 할 수 있다. 세션 타임아웃은 로그인하기 위해 세션이 필요하지 않을것으로 예상하는 사용자들을 놀랍게 한다.

#### Logging Out
위조된 로그아웃 요청으로부터 보호하려면 로그아웃 HTTP 요청을 CSRF 공격으로부터 보호해야한다. 악의적인 사용자가 피해자의 민감한 정보를 읽을 수 없도록 위조 로그아웃 요청으로부터 보호해야한다. 공격에 대한 내용은 아래와 같은 방식으로 이루어진다.

ApplicationX는 login/logout CSRF를 가진 OAUTH 제공자라고 가정한다. ApplicationY는 ApplicationX와의 연결을 구현했다. 연결되어있는 ApplicationX의 계정을 ApplicationY의 권한 인증하는데 사용할 것이다.

1. 공격자 CSRF가 ApplicationX로 부터 피해자를 로그아웃한다.
2. 공격자 CSRF가 ApplicationX에 피해자를 로그인한다. (피해자는 이제 ApplicationX에서 공격자로 로그인 되었다)
3. 공격자는 피해자에게 ApplicationX 연결 기능을 보낸다
4. 공격자의 ApplicationX 계정을 피해자의 ApplicationY 계정으로 연결한다.
5. 공격자는 ApplicationX 로그인을 사용하여 ApplicationY에서 피해자로 인증을 한다.

이외 추가적인 내용은 [여기를 참고](https://labs.detectify.com/2017/03/15/loginlogout-csrf-time-to-reconsider/https://labs.detectify.com/2017/03/15/loginlogout-csrf-time-to-reconsider/)

로그아웃 HTTP 요청이 CSRF 공격으로부터 보호되도록 하기 위해 발생할 수 있는 문제는 세션 타임 아웃 때문에 사용자가 요청이 거부되는 경험을 할 수 있다. 세션 타임아웃은 로그아웃하기 위해 세션이 필요하지 않을 것으로 예상하는 사용자들을 놀랍게 한다.

#### CSRF and Session Timeouts
서버에서는 CSRF 토큰이 세션에 저장 된다. 이는 세션이 만료되면 서버가 해당하는 CSRF 토큰을 찾지 못하고 HTTP 요청을 거부 한다. 타임 아웃을 해결하는 데는 여러가지 옵션이 있으며 각 옵션 마다 장단점이 존재한다.
* 타임 아웃을 완화하는 가장 좋은 방법은 JavaScript를 사용하여 양식 제출시 CSRF 토큰을 요청하는것이다. 그런다음 양식이 CSRF 토큰으로 업데이트 되고 제출 된다.
* 다른 옵션은 사용자에게 세션이 만료 될 예정임을 알리는 JavaScript를 사용하는 것이다. 사용자는 버튼을 클릭하여 세션을 지속하거나 재부여할 수 있다.
* 마지막으로 CSRF 토큰을 쿠키에 저장할 수 있다. 이를 통해 CSRF 토큰이 세션보다 오래 지속될 수 있다.

CSRF 토큰이 기본적으로 쿠키에 저장되지 않는 이유를 물을 수 있다. 다른 도메인에서 헤더(쿠키 지정)를 설정할 수 있는 알려진 공격이 있다. 이는 Ruby on Rails가 헤더 X-Requested-With가있을 때 더 이상 CSRF 검사를 건너 뛰지 않는 이유와 같다. 또 다른 단점은 상태 (즉, 시간 제한)를 제거하면 토큰이 손상된 경우 강제로 무효화 할 수있는 기능이 없다.

#### Multipart (file upload)
CSRF 공격이 발생하지 않도록 하려면 CSRF 토큰을 얻기 위해 HTTP 요청 본문을 읽어야한다. 그러나 본문을 읽는다는 것은 파일이 업로드 된다는 것을 의미하므로 외부 사이트에서 파일을 업로드 할 수 있다.
이렇기 때문에 MUltipart/form-data에서 CSRF 보호를 사용하는 두가지 옵션이 존재한다.

##### Place CSRF Token in the Body
첫번째 옵션은 요청 본문에서 실제 CSRF 토큰을 포함하는 것이다. CSRF 토큰을 본문에 배치하면 권한이 수행되기 전에 본문을 읽게 된다. 즉, 누구나 서버에 임시 파일을 저장할 수 있다. 그러나 승인 된 사용자의 요청만 파일을 제출할 수 있다. 일반적으로 임시 파일 업로드는 대부분의 서버에 거의 영향을 미치지 않기 때문에 권장되는 접근 방식이다. 

##### Include CSRF Token in URL
권한이 없는 사용자가 임시 파일을 업로드 하도록 허용하는 것이 허용되지 않는 경우, 대안은 서버의 CSRF 토큰을 form 속성에 쿼리 매개변수로 포함하는 것이다. 이 방식의 단점은 쿼리 매개 변수가 유출 될 수 있다는 것이다. 일반적으로 중요한 데이터는 유출되지 않도록 본문 또는 헤더 내에 배치하는 것이 모범 사례이다.

##### HiddenHttpMethodFilter
일부 어플리케이션에서는 form 매개 변수를 사용하여 HTTP 메서드를 재정의 할 수 있다. 예를 들어 아래처럼 HTTP 메서드를 POST가 아닌 DELETE로 처리할 수 있다
```html
<form action="/process" method="post">
    <!-- ... -->
    <input type="hidden" name="_method" value="delete"/>
</form>
```
HTTP 메서드 오버라이딩은 필터에서 발생한다. 해당 필터는 Spring Security 이전에 배치 되어야 한다. 오버라이딩은 POST에서만 발생하므로 실제로 문제가 발생하지 않을 가능성이 있다. 그러나 Spring Security의 필터 앞에 배치 되도록 하는것이 모범 사례이다.

### 5.2.2. Security HTTP Response Headers
웹 어플리케이션의 보안을 강화하는데 사용할 수 있는 많은 HTTP 응답 헤더가 있다. Spring Security가 명시적으로 지원하는 다양한 HTTP 응답 헤더를 설명한다. 필요한 경우 사용자 지정 헤더를 제공하도록 Spring Security를 구성할 수 있다.

#### Default Security Headers
스프링 시큐리티는 기본적인 보안 값을 제공하기 위해 보안 관련 HTTP 응답 헤더의 기본 집합을 제공한다.
스프링 시큐리티의 기본값은 다음 헤더를 포한한다.
```
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=31536000 ; includeSubDomains
// Strict-Transport-Security는 HTTPS 요청에만 추가됩니다.
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
```
기본값이 요구 사항을 충족하지 않는 경우 이러한 기본값에서 헤더를 쉽게 제거, 수정 또는 추가 할 수 있다. 

#### Cache Control
스프링 시큐리티의 기본값은 사용자 컨텐츠를 보호하기 위해 캐싱을 비활성화 하는 것이다.
사용자가 민감한 정보를 보기 위해 인증한 다음 로그아웃하는 경우 악의적인 사용자가 뒤로 버튼을 클릭하여 민감한 정보를 볼 수 없도록 한다.
기본적으로 전송되는 캐시 제어 헤더는 다음과 같다.
```
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
```
기본적인 보안을 유지하기 위해 스프링 시큐리티는 기본적으로 헤더를 추가한다. 그러나 응용 프로그램이 자체 캐시 제어 헤더를 제공하는 경우
스프링 시큐리티는 중단된다. 이를 통해 어플리케이션은 CSS 및 JavaScript와 같은 정적 리소스를 캐시 할 수 있다.

#### Content Type Options
지금까지 Internet Explorer를 포함한 브라우저는 컨텐츠 스니핑(데이터의 파일 형식을 추론하는 방법)을 사용하여 요청의 content-type을 추측하려 했다. 
이를 통해 브라우저는 content-type을 지정하지 않은 리소스의 content-type을 추측하여 사용자 경험을 개선 할 수 있었다. 
예를 들어, 브라우저가 content-type이 지정되지 않은 JavaScript파일을 발견한 경우 content-type을 추측한 다음 실행할 수 있다.

컨텐츠 업로드를 허용 할 때해야 할 많은 추가 작업이 있따.(예를 들어 고유한 도메인에 문서만 표시, content-type 헤더 설정 확인, 문서 삭제 등). 
그러나 이러한 조치는 스프링 시큐리티가 제공하는 범위를 벗어난다. 컨텐츠 스니핑을 비활성화 할 때 제대로 작동하려면 content-type을 지정해야한다는 점을 지적하는것도 중요하다.

컨텐츠 스니핑의 문제점은 악의적인 사용자가 다중 컨텐츠(즉, 여러 컨텐츠 유형으로 유효한 파일)를 사용하여 XSS 공격을 수행 할 수 있다는 것입니다. 
예를 들어, 일부 사이트에서는 사용자가 유효한 postscript document를 웹 사이트에 제출하고 볼 수 있도록 허용 할 수 있다. 
악의적인 사용자가 유효한 JavaScript 파일 인 postscript document를 만들어 XSS 공격을 수행 할 수 있다.
[postscript document 취약점](https://asec.ahnlab.com/ko/1047/https://asec.ahnlab.com/ko/1047/)을 참고.

스프링 시큐리티는 HTTP 응답에 다음 헤더를 추가하여 기본적으로 컨텐츠 스니핑을 비활성화 한다.
```
X-Content-Type-Options: nosniff
```

#### HTTP Strict Transport Security (HSTS)
https 프로토콜을 생략하면 중간자 공격에 잠재적으로 취약하다. 웹 사이트가 https로 리디렉션을 수행하더라도 악의적 인 사용자가 
초기 HTTP 요청을 가로 채서 응답을 조작 할 수 있다. (예 : https://악의적인 사이트.com으로 리디렉션하게 하고 자격 증명을 훔칠 수 있음)

많은 사용자가 https 프로토콜을 생략하기 때문에 HTTP Strict Transport Security (HSTS)가 만들어졌다.
예를 들어 mybank.example.com이 HSTS 호스트로 추가 되면 브라우저는 mybank.example.com에 대한 모든 요청이 
https://mybank.example.com으로 해석되어야 함을 미리 알 수 있다. 이렇게 하면 중간자 공격이 발생할 가능성이 크게 줄어 든다.

RFC 6797에 따라 HSTS 헤더는 HTTPS 응답에만 삽입된다. 브라우저가 헤더를 확인하려면 먼저 연결에 사용 된 SSL 인증서에 서명한 CA
(SSL 인증서 뿐만 아니라)를 신뢰해야한다.

사이트를 HSTS 호스트로 표시하는 한 가지 방법은 호스트를 브라우저에 미리 로드하는 것이다. 또 다른 방법은 Strict-Transport-Security 헤더를 
응답에 추가하는 것이다. 예를 들어, Spring Security의 기본 동작은 브라우저가 도메인을 1년 동안 HSTS 호스트로 처리하도록 지시하는 다음 헤더를 추가하는 것이다.
(1 년에 약 31536000초).
```
Strict-Transport-Security: max-age=31536000 ; includeSubDomains ; preload
```
선택 사항 인 IncludeSubDomains 지시문은 하위 도메인 (예 : secure.mybank.example.com)도 HSTS 도메인으로 처리되어야 함을 브라우저에 지시합니다.
선택적 preload 지시문은 도메인이 HSTS 도메인으로 브라우저에 사전로드 되어야 함을 브라우저에 지시합니다. 사용자가 HSTS Lists를 가지고 있지 않다면 
Web Browser는 HTTP Protocol을 사용하여 접속을 하게 되는데 이때 SSL Stripping 공격을 받을 가능성이 있기 때문에 이것을 방지하기 위해 preload를 한다.
Preloaded HSTS Lists에 포함되도록 하려면, "hstpreload.org" Site에 신청을 해야 한다. 또한 Web Browser 사용자가 직접 자신의 Web Browser에
특정 Web Site를 HSTS List에 추가할 수 있도록 기능을 제공하기도 한다(예를 들어 크롬).

#### HTTP Public Key Pinning (HPKP)
패시브 상태를 유지하기 위해 스프링 시큐리티는 여전히 서블릿 환경에서 HPKP에 대한 지원을 제공하지만 위에 나열된 이유로 인해 보안팀에서 더이상 HPKP를 권장하지 않는다.
HPKP는 위조 된 인증서로 MTM(Man in the Middle) 공격을 방지하기 위해 특정 웹서버에서 사용할 공개 키를 웹 클라이언트에 지정한다.
HPKP가 올바르게 사용되면 손상된 인증서에 대한 보호를 할 수 있다. 그러나 HPKP의 복잡성으로 인해 많은 전문가들은 더 이상 사용을 권장하지 않으며 Chrome은 더 이상 지원하지 않는다.

#### X-Frame-Options
웹 사이트를 프레임에 추가하는 것은 보안 문제가 될 수 있다. 예를 들어, 영리한 CSS 스타일을 사용하는 사용자는 의도하지 않은 것을 클릭하도록 속일 수 있다.
예를 들어 은행에 로그인 한 사용자가 다른 사용자에게 엑세스 권한을 부여하는 버튼을 클릭 할 수 있다. 이러한 종류의 공격을 클릭 재킹이라고 한다.
(클릭 재킹 ClickJacking이란 사용자가 인식하는것과 다른것을 클릭 하도록 사용자를 속여 잠재적으로 
기밀 정보를 노출하거나 다른 사람이 컴퓨터를 제어하도록 허용 하는 악의 적인 기술)

클릭 재킹을 처리하는 또 다른 현대적인 접근 방식은 컨텐츠 보안 정책(CSP)을 사용하는 것이다.

클릭 재킹 공격을 완화하는 방법에는 여러가지가 있다. 예를 들어, 기존 브라우저를 클릭 재킹 공격으로부터 보호하기 위해 프레임 브레이킹 코드를 사용할 수 있다.
완벽하지는 않지만 프레임 분리 코드는 레거시 브라우저에서 할 수 있는 최선의 방법이다.

클릭 재킹을 해결하기 위한 현대적인 방법은 X-Frame-Options 헤더를 사용하는것이다. 기본적으로 Spring Security는 다음 헤더를 사용하여
iframe 내에서 페이지 렌더링을 비활성화 한다.
```
X-Frame-Options: DENY
```

#### X-XSS-Protection
일반 브라우저는 XSS 공격을 필터링하는 기능을 내장하고 있다. 이것은 절대적으로 안전하지는 않지만 XSS 보호에 도움이 된다.
필터링은 기본적으로 활성화 되어 있으므로 헤더를 추가하면 일반적으로 활성화되도록 보장하고 XSS 공격이 탐지 될 때 브라우저에 수행할 작업을 지시한다.
예를 들어 필터는 모든 것을 렌더링하기 위해 가장 덜 침습적인 방식으로 컨텐츠를 변경하려 할 수 있다. 때떄로 이러한 유형의 교체는 그 자체로 XSS 취약점이 될 수 있다.
대신 컨텐츠를 수정하는 것보다 차단하는것이 가장 좋다. 기본적으로 Spring Security는 다음 헤더를 사용하여 컨텐츠를 차단한다.
```
X-XSS-Protection: 1; mode=block
```

#### Content Security Policy (CSP)
CSP(컨텐츠 보안 정책)은 웹 애플리케이션이 XSS와 같은 컨텐츠 주입 취약성을 완화하기 위해 활용할 수 있는 메커니즘이다. 
CSP는 웹 응용 프로그램 작성자가 웹 응용 프로그램이 리소스를 로드 할 것으로 예상하는 소스에 대해 선언하고 궁극적으로 클라이언트에게 
알리는 기능을 제공하는 선언적 정책이다.

CSP는 모든 컨텐츠 삽입 취약점을 해결하기 위한 것이 아니다. 대신 CSP를 활용하여 컨텐츠 삽입 공격으로 인한 피해를 줄일 수 있다.
첫번째 방어선으로 웹 애플리케이션 작성자는 입력의 유효성을 검사하고 출력을 인코딩해야한다.

웹 애플리케이션은 응답에 다음 HTTP 헤더 중 하나를 포함하여 CSP를 사용 할 수 있다.
* `Content-Security-Policy`
* `Content-Security-Policy-Report-Only`
이러한 각 헤더는 클라이언트에 보안 정책을 전달하는 메커니즘으로 사용 된다. 보안 정책에는 특정 리소스 representation에 대한 제한을 
선언하는 보안 정책 지시문 집합이 포함된다.

예를 들어 웹 애플리케이션은 응답에 다음 헤더를 포함하여 신뢰할 수 있는 특정 소스에서 스크립트를 로드 할 것으로 예상한다고 선언 할 수 있다.
```
Content-Security-Policy: script-src https://trustedscripts.example.com
```
`script-src`지시문에 선언된 것 이외의 다른 소스에서 스크립트를 로드하려는 시도는 user-agent에 의해 차단된다. 
또한 report-url 지시문이 보안 정책에 선언 된 경우 사용자 에이전트는 선언 된 URL에 위반 사항을 보고한다.

예를 들어 웹 애플리케이션이 선언 된 보안 정책을 위반하는 경우 다음 응답 헤더는 정책의 report-uri 지시문에 지정된 URL로 위반 보고서를 보내도록 
user-agent에 지시한다.
```
Content-Security-Policy: script-src https://trustedscripts.example.com; report-uri /csp-report-endpoint/
```
위반 보고서는 웹 애플리케이션 자체 API 또는 https://report-uri.io/와 같이 공개적으로 호스팅 되는 CSP 위반 보고 서비스에서 캡처 할 수 있는 표준 JSON 구조이다.

Content-Security-Policy-Report-Only 헤더는 웹 어플리케이션 작성자 및 관리자가 보안 정책을 시행하는 대신 모니터링 할 수 있는 기능을 제공한다. 
이 헤더는 일반적으로 사이트에 대한 보안 정책을 실험 및 개발 할 때 사용된다. 정책이 유효한 것으로 간주되면 대신 Content-Security-Policy 헤더 필드를 사용하여 적용 할 수 있다.
```
Content-Security-Policy-Report-Only: script-src 'self' https://trustedscripts.example.com; report-uri /csp-report-endpoint/
```

사이트가 이 정책을 위반하는 경우 evil.com에서 스크립트를 로그하려고 시도하면 user-agent는 report-uri 지시문에 선언 된 URL로 위반 보고서를 보내지만
그래도 위반 리소스가 로드되도록 허용한다.

#### Referrer Policy
Referrer Policy은 웹 어플리케이션이 사용자가 마지막을 방문한 페이지를 포함하는 referrer 필드를 관리하는데 활용할 수 있는 메커니즘이다.
Spring Security의 접근 방식은 다른 정책을 제공하는 Referrer-Policy 헤더를 사용하는 것이다.
```
Referrer-Policy: same-origin
```
Referrer-Policy 응답 헤더는 대상이 사용자가 이전에 있었던 소스를 알도록 브라우저에 지시한다.

#### Feature Policy
Feature Policy은 웹 개발자가 브라우저에 특정 API 및 웹 기능의 동작을 선택적으로 활성화, 비활성화 및 수정할 수 있도록하는 메커니즘이다.
```
Feature-Policy: geolocation 'self'
```
Feature Policy을 통해 개발자는 브라우저가 사이트 전체에서 사용되는 특정 기능에 적용할 'policies'집합을 선택할 수 있다. 
이러한 정책은 사이트가 특정 기능에 대한 브라우저의 기본 동작에 엑세스 하거나 수정할 수 있는 API를 제한한다.

#### Clear Site Data
Clear Site Data는 HTTP 응답에 다음 헤더가 포함 된 경우 브라우저 측 데이터(쿠키, 로컬 저장소 등)를 제거 할 수 있는 메커니즘이다.
```
Clear-Site-Data: "cache", "cookies", "storage", "executionContexts"
```
이것은 로그아웃시 수행 할 수 있는 좋은 정리 작업이다.

#### Custom Headers
Spring Security는 어플리케이션에 보안 헤더를 편리하게 추가 할 수 있는 메커니즘이 있다. 
그러나 사용자 지정 헤더를 추가할 수 있는 후크도 제공한다.

### 5.2.3. HTTP
정적 리소스를 포함한 HTTP 기반 통신은 TLS를 사용하여 보호해야 한다.

프레임 워크로서 Spring Security는 HTTP 연결을 처리하지 않으므로 HTTPS를 직접 지원하지 않는다. 
그러나 HTTPS 사용에 도움이 되는 여러 기능을 제공한다.

#### Redirect to HTTPS
클라이언트가 HTTP를 사용할 때 HTTPS로 리디렉션하도록 구성할 수 있다.

#### Strict Transport Security
Spring security는 Strict Transport Security에 대한 지원을 제공하며 기본적으로 활성화 한다.

#### Proxy Server Configuration
프록시 서버를 사용하는 경우 어플리케이션을 올바르게 구성했는지 확인 하는 것이 중요하다. 
예를 들어 많은 어플리케이션에는 적절한 구성 없이 http://192.168.1:8080에 어플리케이션 서버로 요청을 전달하여 https://example.com/ 요청에 응답하는 로드 밸런서가 있다.
어플리케이션 서버는 로드 밸런서가 존재하는지 모르고 요청을 https://192.168.1:8080이 클라이언트에서 요청한 것처럼 처리한다.

이 문제를 해결하려면 RFC7239를 사용하여 로드 밸런서를 사용하도록 지정할 수 있다. 어플리케이션이 이를 인식하도록 하려면 X-Forwarded 헤더를 인식하는 어플리케이션 서버를 구성해야 한다.
예를 들어 Tomcat은 RemoteValve를 사용하고 Jetty는 FowardedRequestCustomizer를 사용한다. Spring 사용자는 ForwardedHeaderFilter를 활용할 수 있다.

Spring Boot 사용자는 server.user-forward-headers 속성을 사용하여 어플리케이션을 구성할 수 있다.
