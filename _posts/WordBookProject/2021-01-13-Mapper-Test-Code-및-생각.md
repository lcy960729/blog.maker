---
title: 'Mapper Test Code 및 생각'
tags: WordBookProject TestCode
categories: wordbook-project
author: CY
key: Mapper Test Code 및 생각
---
# 2021-01-13 :: Mapper Test Code 및 생각

하루 전에 생각했던 Mapper Test code를 작성해보았다. 커스텀한 맵핑 규칙에 대해 테스트를  동작 하였고 전체적인 맵핑 메소드를 테스트 하였다. 아래는 작성해본 코드이다.

테스트 코드 작성하면서 입력이 Null로 들어왔을때에 대한 생각인데 Null일때 에러를 내는것이 좋은지 유연하게 Blank 값을 넣어주는게 좋은지 생각해봐야겠다. 

또한 Jpa Entity 내 맵핑하는 변수가 Collection 객체일 경우를 생각해서 모두 final 변수로 두어 null값이 들어가지 않게 초기화하였다. 이 방법이 문제가 될만한 점이 있는지 생각해봐야겠다...

```java
@SpringBootTest(classes = {DomainFactory.class, ObjectMapper.class, UserMapperImpl.class})
class UserMapperTest {

    private static final Logger logger = LoggerFactory.getLogger(UserMapperTest.class);

    @Autowired
    private DomainFactory domainFactory;

    @Autowired
    private ObjectMapper objectMapper;

    @Autowired
    private UserMapper userMapper;

    @Test
    @DisplayName("createUserDTOToEntity 맵핑이 정삭적으로 동작 하는 테스트")
    void createUserDTOToEntity() throws JsonProcessingException {
        //given
        CreateUserRequestDTO createUserRequestDTO = CreateUserRequestDTO.builder()
                .name("testName")
                .email("testEmail")
                .pw("testPw")
                .build();

        //when
        User user = userMapper.createUserDTOToEntity(createUserRequestDTO);

        //then
        logger.info(objectMapper.writeValueAsString(user));

        assertThat(user.getName()).isEqualTo(createUserRequestDTO.getName());
        assertThat(user.getEmail()).isEqualTo(createUserRequestDTO.getEmail());
    }

    @Test
    @DisplayName("userToUserDetailDTO 맵핑이 정삭적으로 동작 하는 테스트")
    void entityToUserDetailDTO() throws JsonProcessingException {
        //given
        long userId = 0L;
        User user = domainFactory.createUser(userId);

        int length = 3;
        IntStream.range(0, length).forEach((i) -> {
            StudyGroup studyGroup = domainFactory.createStudyGroup(i);
            UserWordBook userWordBook = domainFactory.createUserWordBook(i);

            Study study = Study.builder()
                    .id((long) i)
                    .studyGroup(studyGroup)
                    .user(user)
                    .build();

            user.joinToStudy(study);
            user.getUserWordBookList().add(userWordBook);
        });

        //when
        UserDetailResponseDTO userDetailResponseDTO = userMapper.entityToUserDetailDTO(user);

        //then
        logger.info(objectMapper.writeValueAsString(userDetailResponseDTO));

        assertThat(userDetailResponseDTO.getId()).isEqualTo(user.getId());
        assertThat(userDetailResponseDTO.getEmail()).isEqualTo(user.getEmail());
        assertThat(userDetailResponseDTO.getName()).isEqualTo(user.getName());

        IntStream.range(0, length).forEach((i) -> {
            UserDetailResponseDTO.StudyGroupDTO studyGroupDTO = userDetailResponseDTO.getStudyGroupList().get(i);
            StudyGroup studyGroup = user.getStudyList().get(i).getStudyGroup();

            assertThat(studyGroupDTO.getId()).isEqualTo(studyGroup.getId());
            assertThat(studyGroupDTO.getName()).isEqualTo(studyGroup.getName());

            UserDetailResponseDTO.WordBookDTO wordBookDTO = userDetailResponseDTO.getWordBookDTOList().get(i);
            UserWordBook userWordBook = user.getUserWordBookList().get(i);

            assertThat(wordBookDTO.getId()).isEqualTo(userWordBook.getId());
            assertThat(wordBookDTO.getName()).isEqualTo(userWordBook.getName());
        });
    }

    @Test
    @DisplayName("User를 StudyGroupDTOList 맵핑이 정삭적으로 동작 하는 테스트")
    void mapToStudyGroupDTOList() throws Exception {
        //given
        User user = domainFactory.createUser(0L);

        int length = 5;
        IntStream.range(0, length).forEach((i) ->{
            StudyGroup studyGroup = domainFactory.createStudyGroup(i);

            Study study = Study.builder()
                    .id((long)i)
                    .user(user)
                    .studyGroup(studyGroup)
                    .build();

            user.getStudyList().add(study);
        });

        //when
        List<UserDetailResponseDTO.StudyGroupDTO> studyGroupDTOList = userMapper.mapToStudyGroupDTOList(user);

        //then
        IntStream.range(0, length).forEach((i) ->{
            UserDetailResponseDTO.StudyGroupDTO studyGroupDTO = studyGroupDTOList.get(i);
            StudyGroup studyGroup = user.getStudyList().get(i).getStudyGroup();

            assertThat(studyGroupDTO.getId()).isEqualTo(studyGroup.getId());
            assertThat(studyGroupDTO.getName()).isEqualTo(studyGroup.getName());
        });
    }

    @Test
    @DisplayName("userWordBookList를 userWordBookDTOList로 맵핑이 정삭적으로 동작 하는 테스트")
    void mapToWordBookDTOList() {
        //given
        List<UserWordBook> userWordBookList = new ArrayList<>();

        int length = 5;
        IntStream.range(0, length).forEach((i) ->{
            userWordBookList.add(domainFactory.createUserWordBook(i));
        });

        //when
        List<UserDetailResponseDTO.WordBookDTO> wordBookDTOList = userMapper.mapToWordBookDTOList(userWordBookList);

        //then
        IntStream.range(0, length).forEach((i) ->{
            UserDetailResponseDTO.WordBookDTO wordBookDTO = wordBookDTOList.get(i);
            UserWordBook userWordBook = userWordBookList.get(i);

            assertThat(wordBookDTO.getId()).isEqualTo(userWordBook.getId());
            assertThat(wordBookDTO.getName()).isEqualTo(userWordBook.getName());
        });
    }
}
```