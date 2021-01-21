# 5. StudyGroupWordBook 도메인

그룹에서 사용하는 단어장이다. 유저 단어장과 기능은 동일 하다.

추가, 수정, 삭제, 조회가 가능하다

1. Create

```json
Request

POST http://localhost:8080/api/v1/users/1/study-groups/6/wordbooks
Body : 
{
    "name": "testWordBook"
}
```

```json
Response

status : Created(201)
Body : 
{
    "id": 10,
    "isUsing": true,
    "name": "testWordBook",
    "_links": {
        "self": {
            "href": "http://localhost:8080/api/v1/users/1/study-groups/6/wordbooks/10"
        },
        "update_studyGroupWordBook": {
            "href": "http://localhost:8080/api/v1/users/1/study-groups/6/wordbooks/10"
        },
        "delete_StudyGroupWordBook": {
            "href": "http://localhost:8080/api/v1/users/1/study-groups/6/wordbooks/10"
        },
        "add_word": {
            "href": "http://localhost:8080/api/v1/users/1/study-groups/6/wordbooks/10"
        },
        "get_studyGroup": {
            "href": "http://localhost:8080/api/v1/users/1/study-groups/6"
        }
    }
}
```

그룹 단어장은 그룹 관리자만 생성할 수 있으며 관리자에게는  update_studyGroupWordBook, delete_StudyGroupWordBook 상태로 갈 수 있는 링크가 반환된다.  이외 add_word, get_studyGroup은 모든 그룹 참여자에게 반환 한다.