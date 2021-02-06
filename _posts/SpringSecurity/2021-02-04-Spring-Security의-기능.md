---
title: 'Spring Security의 기능 (최종 수정 날짜 : 2021-02-04)'
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
