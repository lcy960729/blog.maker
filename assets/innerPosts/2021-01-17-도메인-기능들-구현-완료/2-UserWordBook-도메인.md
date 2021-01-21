# 2. UserWordBook 도메인

사용자 개인 단어장이다. 

추가, 수정, 삭제, 조회가 가능하다.

1. Create

```json
Request

POST http://localhost:8080/api/v1/users/1/wordbooks

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
    "id": 2,
    "isUsing": true,
    "name": "testWordBook",
    "_links": {
        "self": {
            "href": "http://localhost:8080/api/v1/users/1/wordbooks/2"
        },
        "update_userWordBook": {
            "href": "http://localhost:8080/api/v1/users/1/wordbooks/2"
        },
        "delete_userWordBook": {
            "href": "http://localhost:8080/api/v1/users/1/wordbooks/2"
        },
        "add_word": {
            "href": "http://localhost:8080/api/v1/users/1/wordbooks/2/words"
        },
        "get_user": {
            "href": "http://localhost:8080/api/v1/users/1"
        }
    }
}
```

아래와 같이 정상적으로 추가 됐다면 다음 상태에 있는 get_user를 요청 해보자. 

```json
{
    "id": 1,
    "name": "testName123",
    "email": "testEmail@test.com",
    "wordBookDTOList": [
        {
            "id": 2,
            "name": "testWordBook",
            "_links": {
                "get_userWordBook": {
                    "href": "http://localhost:8080/api/v1/users/1/wordbooks/2"
                }
            }
        }
    ],

	...
}
```

아래 생성한 유저를 요청한 결과이다 밑에 다음 상태를 설명해주는 부분은 생략을 하였다. 

달라진 점은 없던 wordBookDTOList가 생겼다. 그리고 해당 필드가 가지는 다음 상태들을 나타냈다.