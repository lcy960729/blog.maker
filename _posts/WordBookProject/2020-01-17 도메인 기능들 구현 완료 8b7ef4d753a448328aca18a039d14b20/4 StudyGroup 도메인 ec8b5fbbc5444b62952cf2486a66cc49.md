# 4. StudyGroup 도메인

개인 유저들이 모여서 그룹을 만드는 도메인이다.

아직 그룹 참여 기능은 구현되지 않았다. 해당 기능을 구현하기 위해선 어떤 방식으로 그룹에 참여할지에 대한 설계가 필요할것 같다. 추후 회의를 통해서 정하도록 해야겠다.

추가, 수정, 삭제, 조회가 가능하다

1. Create

```json
Request

POST http://localhost:8080/api/v1/users/1/study-groups
Body : 
{
    "name": "testStudyGroup"
}
```

```json
Response

status : Created(201)
Body : 
{
    "id": 6,
    "name": "testStudyGroup",
    "userDTOList": [
        {
            "id": 1,
            "name": "testName123"
        }
    ],
    "_links": {
        "self": {
            "href": "http://localhost:8080/api/v1/users/1/study-groups/6"
        },
        "update_studyGroup": {
            "href": "http://localhost:8080/api/v1/users/1/study-groups/6"
        },
        "delete_studyGroup": {
            "href": "http://localhost:8080/api/v1/users/1/study-groups/6"
        },
        "create_studyGroupWordBook": {
            "href": "http://localhost:8080/api/v1/users/1/study-groups/6/wordbooks"
        },
        "get_user": {
            "href": "http://localhost:8080/api/v1/users/1"
        }
    }
}
```

그룹이 생성되며 그룹 사용자와 그룹 단어장이 반환된다. 그룹 단어장은 현재 없는 상태이므로 반환되지 않도록 하였다. 또한 그룹 생성자는 그룹 관리자 권한을 가지게 되며 해당 관리자가 아닌 유저가 그룹 단어장에 접근 하게 되면 update_studyGroup,  delete_studyGroup, create_studyGroupWordBook 해당 링크들을 반환하지 않도록 구현하였다.

이후 다시 get_User를 요청해보겠다. 아래와 같은 결과값이 반환되며 이외 부분은 모두 생략하였다. 각 그룹을 조회 할 수 있는 링크 정보가 담겨 있다.

```json
{
    "id": 1,
    "name": "testName123",
    "email": "testEmail@test.com",
    "studyGroupDTOList": [
        {
            "id": 6,
            "name": "testStudyGroup",
            "_links": {
                "get_studyGroup": {
                    "href": "http://localhost:8080/api/v1/users/1/study-groups/6"
                }
            }
        }
    ],
    ...
}
```