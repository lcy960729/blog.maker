# 1. User 도메인

먼저 유저이다. 일단 아직 Spring security를 사용하지 않았기 때문에 세션이 존재하지는 않는다.

추가, 수정, 조회가 가능하다.

1. create

```json
Request

POST http://localhost:8080/api/v1/users

Body : 
{
    "email": "testEmail@test.com",
    "pw": "testPw",
    "name": "testName123"
}
```

```json
Response

status : Created(201)
Body : 
{
    "id": 1,
    "name": "testName123",
    "email": "testEmail@test.com",
    "_links": {
        "self": {
            "href": "http://localhost:8080/api/v1/users/1"
        },
        "update_user": {
            "href": "http://localhost:8080/api/v1/users/1"
        },
        "delete_user": {
            "href": "http://localhost:8080/api/v1/users"
        },
        "create_studyGroup": {
            "href": "http://localhost:8080/api/v1/users/1/study-groups"
        },
        "create_userWordBook": {
            "href": "http://localhost:8080/api/v1/users/1/wordbooks"
        }
    }
}
```

위와 같은 결과 값이 나오며 다음 상태로 갈 수 있는 정보가 나오게 된다.