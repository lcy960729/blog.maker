# 3. Word Of UserWordBook 도메인

유저 개인 단어장의 단어들이다.

추가, 수정, 조회, 삭제가 가능하다.

1. Create

```json
Request

POST http://localhost:8080/api/v1/users/1/wordbooks/2/words

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
    "id": 3,
    "voca": "apple",
    "meaning": "사과",
    "_links": {
        "update_word": {
            "href": "http://localhost:8080/api/v1/users/1/wordbooks/2/words/3"
        },
        "delete_word": {
            "href": "http://localhost:8080/api/v1/users/1/wordbooks/2/words/3"
        },
        "get_userWordBook": {
            "href": "http://localhost:8080/api/v1/users/1/wordbooks/2"
        }
    }
}
```

단어를 2개 정도 생성해보고 다시 get_userWordBook을 호출하게 되면 아래와 같이 단어가 추가되고 해당 단어에서 갈 수 있는 상태인 수정과 삭제 요청을 반환한다.

```json
{
    "id": 2,
    "isUsing": true,
    "name": "testWordBook",
    "wordDTOList": [
        {
            "id": 3,
            "voca": "apple",
            "meaning": "사과",
            "_links": {
                "update_word": {
                    "href": "http://localhost:8080/api/v1/users/1/wordbooks/2/words/3"
                },
                "delete_word": {
                    "href": "http://localhost:8080/api/v1/users/1/wordbooks/2/words/3"
                }
            }
        },
        {
            "id": 4,
            "voca": "banana",
            "meaning": "바나나",
            "_links": {
                "update_word": {
                    "href": "http://localhost:8080/api/v1/users/1/wordbooks/2/words/4"
                },
                "delete_word": {
                    "href": "http://localhost:8080/api/v1/users/1/wordbooks/2/words/4"
                }
            }
        }
    ],
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