# RFC 2326 : RTSP 1.0

## Response

HTTP 응답에서 HTTP-Version을 RTSP-Version으로 바꾸면 된다.

그리고 추가적인 상태 코드 값도 존재한다

```
Response	=	Statue-Line		;
			*(	general-header	;
			|	response-header	;
        	|	entity-header )	;
        	CRLF
        	[message-body]		;
```

### Status-Line

```
Status-Line	=	RTSP-Version SP Status-Code	SP Reason-Phrase CRLF
```

### Status Code and Reason Phrase

Status-Code는 3자리 숫자로 이루어져있으며, 요청에 대한 결과 값이다.

RTSP에서 사용하는 특정 상태코드는 x50으로 시작하니 HTTP 코드와 사용시 유의

Reason-Phrase는 Status-Code에 대한 짧은 설명이다.

상태 값은 뒤에 상세히 설명하고, 여기서는 앞자리에 따른 대략적인 내용을 기술

- 1xx: Informational

  요청을 받았으며, 처리 중

- 2xx: Success

  요청을 성공적으로 받았고(received), 인지했으며(understood), 처리했다(accepted).

- 3xx: Redirection

  요청을 성공적으로 처리하기 위해서는 추가 작업이 필요함

- 4xx: Client Error

  요청이 문법적으로 잘못되었거나 충족되지 않음

- 5xx: Server Error

  서버측 오류로 요청을 성공적으로 처리하지 못했음

#### Status Code

- 1XX

  | Status code | Reason Phrase | RTSP Usage |
  | ----------- | ------------- | ---------- |
  | 100         | Continue      | ALL        |

- 2xx

  | Status code | Reason Phrase        | RTSP Usage |
  | ----------- | -------------------- | ---------- |
  | 200         | OK                   | ALL        |
  | 201         | Created              | RECORD     |
  | 250         | Low on Storage Space | RECORD     |

- 3xx

  | Status code | Reason Phrase     | RTSP Usage |
  | ----------- | ----------------- | ---------- |
  | 300         | Multiple Choices  | ALL        |
  | 301         | Moved Permanently | ALL        |
  | 302         | Moved Temporarily | ALL        |
  | 303         | See Other         | ALL        |
  | 304         | Not Modified      | -          |
  | 305         | Use Proxy         | ALL        |

- 4xx

  | Status code | Reason Phrase                    | RTSP Usage      |
  | ----------- | -------------------------------- | --------------- |
  | 400         | Bad Request                      | ALL             |
  | 401         | Unauthorized                     | ALL             |
  | 402         | Payment Required                 | ALL             |
  | 403         | Forbidden                        | ALL             |
  | 404         | Not Found                        | ALL             |
  | 405         | Method Not Allowed               | ALL             |
  | 406         | Not Acceptable                   | ALL             |
  | 407         | Proxy Authentication Required    | ALL             |
  | 408         | Request Timeout                  | ALL             |
  | 410         | Gone                             | ALL             |
  | 411         | Length Required                  | ALL             |
  | 412         | Precondition Failed              | DESCRIBE, SETUP |
  | 413         | Request Entity Too Large         | ALL             |
  | 414         | Request-URI Too Large            | ALL             |
  | 415         | Unsupported Media Type           | ALL             |
  | 451         | Invalid parameter                | SETUP           |
  | 452         | Invalid Conference Identifier    | SETUP           |
  | 453         | Not Enough Bandwidth             | SETUP           |
  | 454         | Session Not Found                | ALL             |
  | 455         | Method Not Valid in This State   | ALL             |
  | 456         | Header Field Not Valid           | ALL             |
  | 457         | Invalid Range                    | PLAY            |
  | 458         | Parameter Is Read-Only           | SET_PARAMETER   |
  | 459         | Aggregate Operation Not Allowed  | ALL             |
  | 460         | Only Aggregate Operation Allowed | ALL             |
  | 461         | Unsupported Transport            | ALL             |
  | 462         | Destination Unreachable          | ALL             |

- 5XX

  | Status code | Reason Phrase              | RTSP Usage |
  | ----------- | -------------------------- | ---------- |
  | 500         | Internal Server Error      | ALL        |
  | 501         | Not Implemented            | ALL        |
  | 502         | Bad Gateway                | ALL        |
  | 503         | Service Unavailable        | ALL        |
  | 504         | Gateway Timeout            | ALL        |
  | 505         | RTSP Version Not Supported | ALL        |
  | 551         | Option not support         | ALL        |

### Response Header

#### Location

클라이언트가 요청한 URI와 다른 위치로 이동하거나, 새로운 리소스를 식별하도록 지시하는데 사용

- 리다이렉션

  3xx 상태 코드와 함께 응답 시, 해당 헤더는 리다이렉션해야 하는 URI를 명시

- 리소스 생성

  "201 Created" 응답에서는 새롭게 생성된 리소스의 URI

```
Location: http://www.w3.org/pub/WWW/People.html
```

> <strong>`Content-Location`과의 차이점</strong>
>
> 요청에 포함된 실제 엔티티의 본 위치
>
> 즉 Location은 새로운 리소스를, Content-Location은 제공된 데이터의 출처를 나타낸다

#### Proxy-Authenticate

해당 헤더는 "407 Proxy Authentication Required" 응답과 함께 사용해야 한다.

이는 프록시 서버에서 인증이 필요하다는 것을 나타내며, 클라리언트가 인증 시 필요한 인증 스킴과 관련 매개변수를 제공해야함

이 때, 해당 헤더는 현재 연결에만 유효하고, 다른 요청 혹은 하위 클라이언트에 전달되지 않아야 한다.

```
Proxy-Authenticate: Basic realm="ExampleProxy"
```

#### Public

OPTIONS 요청에 대해 어떤 메소드를 지원하는지 알려주는 헤더

```
C->S:
OPTIONS rtsp://example.com/media.mp4
CSeq: 1

S->C:
RTSP/1.0 200 OK
CSeq: 1
Public: OPTIONS, DESCRIBE, SETUP, PLAY, PAUSE, TEARDOWN
```

#### Retry-After

"503 Service Unavailable" 응답에 대해 재요청 가능 시각을 나타내는 헤더

```
Retry-After: Fri, 31 Dec 1999 23:59:59 GMT
Retry-After: 120 // means 2 minutes
```

#### Server

해당 요청을 처리한 서버에 대한 정보를 담는 헤더

```
Server	= "Server" ":" 1*( product | comment )
```

```
Server: CERN/3.0 libwww.2.17
```

#### Vary

요청 헤더의 값에 따라 캐싱을 결정하는 헤더

캐시는 Vary 헤더에 명시된 헤더 값이 동일한 경우에만 캐싱되어 있는 응답을 재사용

```
Vary: Accept-Language, User-Agent
```

위 예시의 경우에는 위 두 헤더가 일치하는 캐싱된 응답만을 재사용 할 수 있다.

`*` 값이라면, 캐시는 캐싱된 응답을 재사용 할 수 없다.

이는 서버가 클라이언트의 요청 헤더를 보고 적절한 응답을 제공해야 할 때 사용되는데, 이를 남발할 경우 캐싱의 이점을 제한할 수 있다.

#### WWW-Authenticate

"401 Unauthorized" 응답에 반드시 포함되어야 하는 헤더

클라이언트가 사용할 수 있는 인증방식을 포함한다.

```
WWW-Authenticate = "WWW-Authenticate" ":" 1#challenage
```

`challenage`에는 인증방식이 들어가며, 인증 스키마에 따른 필요한 추가정보가 매개변수로 붙는다

```
WWW-Authenticate = Basic realm="Access to the site"

WWW-Authenticate: Digest realm="Secure Area", qop="auth", nonce="dcd98b7102dd2f0e8b11d0f600bfb0c093", opaque="5ccc069c403ebaf9f0171e9517f40e41"
```

복수 개의 인증 스키마를 넣을 수도 있다. 따라서 이를 고려하여 파싱해야 한다
