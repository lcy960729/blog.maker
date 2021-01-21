---
title: 'Service Layer Validator 테스트 문제점'
tags: WordBookProject SpringBoot TestCode Junit SpringBootDataJPA
categories: wordbook-project
author: CY
key: Service Layer Validator 테스트 문제점
---
# 2020-12-29 :: Service Layer Validator 테스트 문제점

Spring Boot에서 spring validator는 컨트롤러에서만 동작을 한다. 이유로는 spring mvc에는 서비스 레이어라는것이 존재하지 않는다. 서비스 어노테이션은 그저 단순한 빈의 형태이며 컨트롤러 레이어에서 validator가 동작할 수 있는 이유는 Handler 메소드에서 사용할 인수를 처리하기 전에 유효성 검사를 수행하는 ModelAttributeMethodProcessor가 있기때문이다. 그렇기 때문에  validator가 서비스 레이어에서도 동작을 할수 있도록 하기위해 아래 코드와 같이 서비스 클래스 위에 @Validated라는 어노테이션을 추가해준다. 

```java
@Service
@Validated
public class CreateUserWordBookService {

    private final UserWordBookRepository userWordBookRepository;

    private final ModelMapper modelMapper = new ModelMapper();

    public CreateUserWordBookService(UserWordBookRepository userWordBookRepository) {
        this.userWordBookRepository = userWordBookRepository;
    }

    public Long create(@Valid UserWordBookRequestDTO.Create createUserWordBookDTO) {
        UserWordBook userWordBook = modelMapper.map(createUserWordBookDTO, UserWordBook.class);

        userWordBook = userWordBookRepository.save(userWordBook);

        return userWordBook.getId();
    }
}
```

문제점은 테스트 코드를 작성할때 발생한다. 서비스 레이어에서 유닛 테스트를 진행하게 되면 @Valid가 트리거 되지않는다. 이유로는 mvc테스트가 아니므로 DispatcherServlet이 없기 때문이다(아래 사진은 유효성 검증이 어떤식으로 이루어지는지 디버깅해본 사진이다). 그렇기 때문에 서비스레이어에서 유효성 검사를 완전히 하기 위해서는 통합 테스트로 진행을 하던가 유효성 검사부분을 직접 작성해야한다. 

![2020-12-29%20Service%20Layer%20Validator%20%E1%84%90%E1%85%A6%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%AE%E1%86%AB%E1%84%8C%E1%85%A6%E1%84%8C%E1%85%A5%E1%86%B7%2038eb5e110e344929bb8f2b0ec408ae46/Untitled.png](2020-12-29%20Service%20Layer%20Validator%20%E1%84%90%E1%85%A6%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%AE%E1%86%AB%E1%84%8C%E1%85%A6%E1%84%8C%E1%85%A5%E1%86%B7%2038eb5e110e344929bb8f2b0ec408ae46/Untitled.png)

이 부분에서 많은 시간을 투자했는데 디버깅해보면서 따라가봤으면 금방 해결할 수 있는 문제였다. 안되는걸 하려고하니 시간만 날리는 문제가 발생했는데 다른 방법을 찾아봐야겠다.