---
title: 'Spring Boot Controller Test'
tags: WordBookProject SpringBoot TestCode MVC Junit Mockito
categories: wordbook-project
author: CY
key: Spring Boot Controller Test
---
# 2020-12-13 :: Spring Boot Controller Test

Spring Boot에서 컨트롤러 테스트를 위해 여러 방법을 찾아보았다. 그중 하나를 설명하겠다.

먼저 테스트 클래스 어노테이션이다

1. @RunWith(SpringRunner.class)

    JUnit 프레임워크가 테스트 실행시 SpringRunner.class를 같이 실행하라는 의미

2. @WebMvcTest(controllers = ?)

    컨트롤러 레이어를 테스트 하기위한 어노테이션. 컨트롤러 레이어에 선언된 bean만 가져옴

    Service 레이어의 빈들은 받아오지 못함으로 @MockBean으로 가짜 서비스를 만들어 주어야함

    @SpringBootTest는 전체 프로그램의 bean을 가져옴. 통합 테스트를 진행할때 적합.

다음은 작성해본 테스트 코드 예제 이다

```java
@RunWith(SpringRunner.class)
@WebMvcTest(controllers = UserController.class)
public class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    private String userDTOJson;

    @Before
    public void setUp() throws JsonProcessingException {
        UserDTO userDTO = new UserDTO();
        userDTO.setName("lcy");
        userDTO.setUserId("admin");
        userDTO.setPw("admin");
        userDTO.setAge(25);

        ObjectMapper objectMapper = new ObjectMapper();

        userDTOJson = objectMapper.writeValueAsString(userDTO);
    }

    @Test
    public void createUser_locationIsNotNull() throws Exception {
        //given
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
                .andExpect(status().isOk())
                .andExpect(header().exists("location"));
    }

    @Test
    public void testCreateUser() {
    }
}
```

@MockMvc는 스프링 mvc 동작을 재현할 수 있는 클래스이다

@MockBean은 실제로 구현한 객체가 아닌 가짜 객체이다. 현재 UserController는 UserService와 의존성이 있으므로 UserController에 대해서만 단위 테스트를 하기 위해선 가짜 UserService를 만들어준후 테스트 해야한다. 

만약 컨트롤러 단위 테스트에서 실제 UserService의 객체를 이용한 테스트가 진행되면 UserService가 제대로 동작하는지 같이 확인하게 됨으로 컨트롤러와 서비스 모두 테스트를 하게 된다. 이렇게 의존성이 생긴 테스트는 잘못된것이므로 테스트 코드를 수정할 필요가 있다. 

`createUser_locationIsNotNull` 은 유저를 생성하고 결과값으로 location 헤더에 값이 있는지를 확인한다.

테스트 코드가 어떻게 돌아가는지는 알았지만 어떤 상황에 어떤식으로, 또 어디까지 테스트를 해야하는지 아직 감이 잡히지 않는다. 많은 예제를 보고 익히도록 해야겠다.