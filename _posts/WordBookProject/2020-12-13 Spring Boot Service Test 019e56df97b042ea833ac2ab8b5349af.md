---
title: 'Spring Boot Service Test'
tags: WordBookProject SpringBoot TestCode Junit Mockito
categories: wordbook-project
author: CY
key: Spring Boot Service Test
---
# 2020-12-13 :: Spring Boot Service Test

Service Layer Test와 Controller Layer Test는 구현하는데 차이점이 있다. 서비스 레이어는 웹 레이어와 관련이 없으므로 WebMvcTest 어노테이션을 사용하지 않는다.

서비스 레이어 테스트 코드 작성 방식중 SpringBootTest(classes = *)를 사용하는 방법과 mockito를 사용하는 방식이 있다. mockito를 사용하면 서비스 레이어에서만 독립적으로 테스트 할 수 있어 객체간 의존성이 사라진다. 또 SpringBootTest를 사용하지 않음으로서 서버를 동작하지 않고 테스트를 진행할 수 있다.

Mockito를 이용하여 작성한 방법을 설명하겠다. 먼저 작성한 테스트 코드이다.

```java
@RunWith(SpringRunner.class)
@ExtendWith(MockitoExtension.class)
public class UserServiceTest {
    @InjectMocks
    private UserService userService;

    @Mock
    private UserRepository userRepository;

    private User user;
    private UserDTO userDTO;

    private static final Long newID = 0L;

    @Before
    public void setUp() {
        userDTO = new UserDTO();
        userDTO.setUserId("admin");
        userDTO.setPw("admin");
        userDTO.setName("lcy");
        userDTO.setAge(25);

        user = User.builder()
                .id(newID)
                .userId(userDTO.getUserId())
                .pw(userDTO.getPw())
                .name(userDTO.getName())
                .age(userDTO.getAge())
                .build();
    }

    @Test
    public void createUser() {
        //mocking, given
        given(userRepository.save(any())).willReturn(user);
//        when(userRepository.save(any())).thenReturn(user);

        //when
        Long id = userService.createUser(userDTO);

        //then
        verify(userRepository).save(any());
        assertThat(id).isNotNull();
    }
}
```

@ExtendWith(MockitoExtension.class) 어노테이션을 사용하여 @Mock 어노테이션이 붙은 객체를 목객체로 초기화하고 @InjectMocks가 붙어있는 객체에는 목객체로 의존을 주입한다.

서비스와 의존성이 있는 UserRepository를 Mock객체로 선언한다.

서비스는 InjectMocks로 의존을 주입한다.

@Befors

- 테스트 코드 동작 이전에 수행되는 함수이다.

@Test

- given 영역에서는 mocking을 해준다. 해당 목객체가 어떤 함수 동작시 반환할 값을 정해준다. 보면 give-willReturn과 whe-thenReturn이 있는데 이 둘의 동작은 같지만 차이점으론 when으로 선언한 방식이 BDD 테스트 방식을 추구한다고 한다.
- when 영역에서는 내가 테스트할 시점에 대해서 작성한다.
- then 영역에서는 service에서 userRepository.save가 호출되었는지 검증을 하고 반환값인 id가 Null이 아닌지 확인한다.

then 영역에서 assert문을 사용할때는 assertj의 assertThat을 사용하는것이 가독성이 좋고 사용하기도 편하다고 한다.  간단한 사용방법에 대해선 아래의 사이트를 참고하고 보다 나은 기능은 문서를 참조하도록 하자.

[[AssertJ] JUnit과 같이 쓰기 좋은 AssertJ 필수 부분 정리](https://pjh3749.tistory.com/241)

[AssertJ - fluent assertions java library](https://assertj.github.io/doc/)