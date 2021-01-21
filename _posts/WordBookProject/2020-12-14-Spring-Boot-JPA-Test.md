---
title: 'Spring Boot JPA Test'
tags: WordBookProject SpringBoot TestCode Junit SpringBootDataJPA
categories: wordbook-project
author: CY
key: Spring Boot JPA Test
---
# 2020-12-14 :: Spring Boot JPA Test

JPA를 테스트 하는 방법을 알아보겠다. 우리는 특정한 목적을 위해 쿼리를 작성할 일이 생기고 해당 쿼리가  문제 없이 올바르게 동작하는지 테스트할 필요가 있다. Springboot에서는 @DataJpaTest라는 어노테이션을 이용하여 JPA를 테스트 할수 있게 해준다.

```java
//레퍼지토리
public interface UserRepository extends JpaRepository<User, Long> {

    @Query("SELECT u FROM user u where u.id = :id")
    Optional<User> findById2(@Param("id") Long id);
}

//테스트 클래스
@RunWith(SpringRunner.class)
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
public class UserRepositoryTest {
    @Autowired
    private UserRepository userRepository;

    @Test
    public void User_save(){
        //given
        User userTemp = User.builder()
                .userId("admin")
                .pw("admin")
                .name("lcy")
                .age(25)
                .build();
        userTemp = userRepository.save(userTemp);

        //when
        User user = userRepository.findById2(userTemp.getId()).orElse(null);

        //then
        assertThat(user)
                .isNotNull()
                .usingRecursiveComparison()
                .isEqualTo(userTemp);
    }
}
```

@AutoConfigureTestDatabase 테스트시 데이터베이스와 관련된 설정을 하는 어노테이션이다. 현재처럼 None으로 지정하게되면 application.properties에 설정된 db를 사용한다. 또한 테스트시에 테스트 디비를 따로 사용하도록 설정 할 수도 있다. 자세한 내용은 문서를 찾아보자

테스트 방식은 앞에서 공부했던 내용과 유사하다. 테스트 내용을 설명하자면 간단하게 findById와 같은 동작을 하는 쿼리를 작성하였다. 이 쿼리가 올바르게 동작하는지 알기 위해 먼저 새로운 유저를 저장한다. 작성한 findById2를 이용하여 저장된 유저의 아이디를 불러온다. 저장한 userTemp와 불러온 user의 모든 속성이 같은지를 확인한다.

---

DataJpaTest를 실습해보면서 @AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.Any)를 사용하여 내장된 DB를 사용할때 문제가 발생하였다.

기본적으로 위 어노테이션을 사용하거나 아무것도 사용하지 않는다면 DataJpaTest는 인 메모리 디비를 사용한다. 이때 우리가 application.properties에 db관련된 설정들과 충돌을 일으키면서 DDL이 동작하지 않는 오류가 발생한다. 

임시적인 해결법으로는 application.properties에 DB 접속과 관련된 부분을 주석을 걸어준다.
또는 @AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)를 이용하여 실제 디비 환경에서 테스트를 진행한다. 

테스트 환경에 맞는 application.properties를 구성하는 방법이 있을건데 이건 더 찾아봐야겠다.
