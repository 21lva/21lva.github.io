---
layout: post
title:  "ControllerAdvice"
date:   2022-07-12 02:02:59
author: Inhyuk
category: spring
tags: spring
cover:  "/assets/instacode.png"
name: controllerAdvice.md
---

ControllerAdvice
================

- Controller에서 발생한 예외를 통합처리하기 위한 기능
- 아래 예제의 구성은 다음과 같다
  - CustomExceptionHandler: Exception들을 처리하는 handler. 처리를 원하는 class만 지정 가능
  - ErrorResponse: 응답에 담길 Error에 관한 정보들
  - ErrorCode: exception에 관한 정보를 갖는 enum
  - CustomException: exception class의 기본 구조. RuntimeException을 상속.

```kt
@RestController
class SampleController {
    @GetMapping("/a")
    suspend fun getList(@RequestParam isE: String): String{
        when(isE){
            "1" -> throw InvalidParamException()
            "2" -> throw UnauthorizedException()
            "3" -> throw Exception("not custom exception")
            else -> return "a"
        }
    }
}

@ControllerAdvice
class CustomExceptionHandler{
    @ExceptionHandler(CustomException::class)
    suspend fun handleInvalidParamExceptionHandler(e: CustomException): ResponseEntity<ErrorResponse>{
        return ResponseEntity(ErrorResponse(e.message?:"", e.status), HttpStatus.resolve(e.status)?: HttpStatus.INTERNAL_SERVER_ERROR)
    }

    @ExceptionHandler(Exception::class)
    suspend fun handleException(e: Exception): ResponseEntity<ErrorResponse>{
        return ResponseEntity(ErrorResponse(e.message?:"", HttpStatus.INTERNAL_SERVER_ERROR.value()), HttpStatus.INTERNAL_SERVER_ERROR)
    }
}

data class ErrorResponse(
    val message: String,
    val status: Int,
)

enum class ErrorCode(val status: Int, val message: String){
    INVALID_PARAM(400, "invalid format"),
    UNAUTHORIZED(401, "unauthrozied to do"),
}

open class CustomException(errorCode: ErrorCode): RuntimeException(errorCode.message){
    val status = errorCode.status
}
class InvalidParamException: CustomException(ErrorCode.INVALID_PARAM)
class UnauthorizedException: CustomException(ErrorCode.UNAUTHORIZED)
```

- 이를 시행하고 여러 request를 보내보면 아래와 같은 결과를 얻을 수 있다.


```
//1
curl -X GET "localhost:8080/a?isE=1" -i
HTTP/1.1 400 Bad Request
Content-Type: application/json
Content-Length: 41

{"message":"invalid format","status":400}%

//2
curl -X GET "localhost:8080/a?isE=2" -i
HTTP/1.1 401 Unauthorized
Content-Type: application/json
Content-Length: 45

{"message":"unauthrozied to do","status":401}%

//3
curl -X GET "localhost:8080/a?isE=3" -i
HTTP/1.1 500 Internal Server Error
Content-Type: application/json
Content-Length: 47

{"message":"not custom exception","status":500}%

//4
curl -X GET "localhost:8080/a?isE=4" -i
HTTP/1.1 200 OK
Content-Type: text/plain;charset=UTF-8
Content-Length: 1

a%
```
