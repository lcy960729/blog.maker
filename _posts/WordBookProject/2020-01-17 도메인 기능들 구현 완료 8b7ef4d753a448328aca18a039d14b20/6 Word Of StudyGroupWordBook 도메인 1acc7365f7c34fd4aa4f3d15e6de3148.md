# 6. Word Of StudyGroupWordBook 도메인

그룹 단어장의 단어이다. 유저 단어장의 단어 기능과 동일 하다.

추가, 수정, 삭제, 조회가 가능하다

1. Create

```json
Request

POST http://localhost:8080/api/v1/users/1/study-groups/6/wordbooks/10
Body : 
{
    "voca": "apple",
    "meaning": "사과"
}
```

```json
Response

status : Created(201)
Body : 
{
    "id": 11,
    "voca": "apple",
    "meaning": "사과",
    "_links": {
        "update_word": {
            "href": "http://localhost:8080/api/v1/users/1/study-groups/6/wordbooks/10/words/11"
        },
        "delete_word": {
            "href": "http://localhost:8080/api/v1/users/1/study-groups/6/wordbooks/10/words/11"
        },
        "get_studyGroupWordBook": {
            "href": "http://localhost:8080/api/v1/users/1/study-groups/6/wordbooks/10"
        }
    }
}
```

유저 단어장의 단어와 유사하다 수정과 삭제를 할 수 있으며 이전 상태인 get_studyGroupWordBook 링크를 반환 한다.