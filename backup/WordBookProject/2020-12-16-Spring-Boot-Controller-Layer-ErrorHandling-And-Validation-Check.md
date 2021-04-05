---
title: 'Spring Boot Controller Layer ErrorHandling And Validation Check'
tags: WordBookProject SpringBoot Validation ErrorHandling
categories: wordbook-project
author: CY
key: Spring Boot Controller Layer ErrorHandling And Validation Check
---
# 2020-12-16 :: Spring Boot Controller Layer ErrorHandling And Validation Check

Spring Boot ErrorHandling에 대해서 알아 보겠다. 컨트롤러 단에서 발생하는 에러들을 처리해야할 일이 많아 진다. 그때 해당 컨트롤러 안에서 발생하는 에러들을 핸들링 하겠다는것이 @ExceptionHandler('exception.class')이다. 해당 어노테이션을 붙인 함수를 컨트롤러 안에 선언하면 어노테이션에 추가한 exception에 대해서 핸들러 함수가 처리를 해준다. 코드를 확인 해 보자

```java
@RestController("/api/v1")
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping(path = "/user")
    public ResponseEntity<Object> createUser(@Valid @RequestBody UserDTO createUserDTOJson) {
            Long id = userService.createUser(createUserDTOJson);

            HttpHeaders httpHeaders = new HttpHeaders();
            httpHeaders.add("location", String.valueOf(id));

            return new ResponseEntity<>(httpHeaders, HttpStatus.OK);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<String> ValidationExceptionHandle(MethodArgumentNotValidException methodArgumentNotValidException) {
        BindingResult bindingResult = methodArgumentNotValidException.getBindingResult();
        for (ObjectError objectError : bindingResult.getAllErrors()){
            logger.error(objectError.getDefaultMessage());
        }

        return new ResponseEntity<>(HttpStatus.UNPROCESSABLE_ENTITY);
    }
```

위와 같은 코드가 있을때 UserControllerd에서 발생하는 MethodArgumentNotValidException에 대해서 @ExceptionHandler 어노테이션이 붙은 ValidationExceptionHandle 함수가 처리하게 된다.

더 나아가서 우리는 여러 컨트롤러들이 동일한 예외를 낸다면 위처럼 에러를 처리하기엔 많은 중복 코드가 발생하게 된다. 이런 문제를 해결하기 위해 우리는 전역 컨트롤러에서 발생하는 동일한 에러를 처리하는 클래스를 만들고 이 클래스에서 모든 에러를 처리하도록 구현할 수 있다. 아래의 코드를 보자.

```java
//유저 컨트롤러
@RestController("/api/v1")
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping(path = "/user")
    public ResponseEntity<Object> createUser(@Valid @RequestBody UserDTO createUserDTOJson) {
            Long id = userService.createUser(createUserDTOJson);

            HttpHeaders httpHeaders = new HttpHeaders();
            httpHeaders.add("location", String.valueOf(id));

            return new ResponseEntity<>(httpHeaders, HttpStatus.OK);
    }
}

//RestController에서 발생하는 모든 에러를 담당하는 클래스 -------------------------------------------------------------------------------
@RestControllerAdvice(annotations = RestController.class)
public class RestControllerExceptionHandler {
    private static final Logger logger = 
				LoggerFactory.getLogger(UserExceptionController.class);

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<String> ValidationExceptionHandle(MethodArgumentNotValidException methodArgumentNotValidException) {
        BindingResult bindingResult = methodArgumentNotValidException.getBindingResult();
        for (ObjectError objectError : bindingResult.getAllErrors()){
            logger.error(objectError.getDefaultMessage());
        }

        return new ResponseEntity<>(HttpStatus.UNPROCESSABLE_ENTITY);
    }
}
```

위에서 처리한 에러는 VaildException으로 파라메타들의 값이 유효하게 들어왔는지를 확인한다. 이러한 에러는 여러 컨트롤러에서 발생할 수 있으며 각 컨트롤러마다 만들 필요없이 한번에 처리하도록 만드는것이 좋다. 먼저 @ControllerAdvice (annotations = 'class') 어노테이션을 이용하여  (annotations = 'class')에서 선언한 어노테이션들을 가진 클래스들의 에러를 전역으로 관리한다. 에러를 핸들링 하는 부분은 위와 같다.

추가 적으로 객체에 대한 유효성 검증에 대해서 알아보자. 

유효성 검증 라이브러리는 `spring-boot-starter-validation`를 사용하였다. 간단하게 유효성을 검증할 Entity나 DTO에 아래와 같이 @Null이나 @NotBlank와 같이 해당하는 어노테이션을 작성해주면 된다. 이후 위에 코드 처럼 유효성 검증할 변수 앞에 @Vaild를 작성해주면 된다.

```java
//어떤 변수에 대해서 유효성을 체크할건지 선언하는 부분 @Valid
public ResponseEntity<Object> createUser(**@Valid** @RequestBody UserDTO createUserDTOJson)

//어떻게 체크할건지 선언하는 부분 @Null, @NotBlank, @Min(value = 2, message = "so young...") 등...
public class UserDTO {
    *@Null*
    private Long id;

    @NotBlank
    private String userId;

    @NotBlank
    private String pw;

    @NotBlank
    private String name;

    @Min(value = 2, message = "so young...")
    private Integer age;
}

```

마지막으로 제대로 동작하는지 확인 하기위해 아래와 같은 테스트 코드를 작성.

```java
@Test
public void createUser_ValidationExceptionTest() throws Exception {
    //given
    userDTO.setUserId(null);
    userDTO.setPw("");
    userDTO.setName(null);
    userDTO.setAge(1); //Min Age 2
    String userDTOJson = new ObjectMapper().writeValueAsString(userDTO);

    given(userService.createUser(any(UserDTO.class))).willReturn(0L);

    //when
    ResultActions resultActions = mockMvc.perform(
            post("/user")
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(userDTOJson)
    );

    //then
    resultActions
            .andDo(print())
            .andExpect(status().isUnprocessableEntity());
}
```

실행 결과

```java
2020-12-17 02:59:44.935 ERROR 12088 --- [    Test worker] c.e.s.e.RestControllerExceptionHandler   : must not be blank
2020-12-17 02:59:44.936 ERROR 12088 --- [    Test worker] c.e.s.e.RestControllerExceptionHandler   : must not be blank
2020-12-17 02:59:44.936 ERROR 12088 --- [    Test worker] c.e.s.e.RestControllerExceptionHandler   : too young...
2020-12-17 02:59:44.936 ERROR 12088 --- [    Test worker] c.e.s.e.RestControllerExceptionHandler   : must not be blank
...
BUILD SUCCESSFUL in 6s
```

유효성 검사에서 에러를 발생했기 때문에 controller에서 422(unprocessableEntity)에러를 반환하여 정삭적으로 유효성 검사가 되는지 테스트 하였다.

---

Vaild 체크 어노테이션이나 @RestControllerAdvice의 옵션들은 문서를 참고하자

[Java Bean Validation Basics Baeldung](https://www.baeldung.com/javax-validation)

[RestControllerAdvice - spring-web 5.1.1.RELEASE javadoc](https://javadoc.io/doc/org.springframework/spring-web/5.1.1.RELEASE/org/springframework/web/bind/annotation/RestControllerAdvice.html#annotations--)