# RFC 2326 : RTSP 1.0

## RTSP Message

RTSP는 텍스트 기반의 프로토콜로 UTF-8 인코딩 기반의 ISO 10646 character set을 사용한다.

CRLF로 줄의 끝을 명시하며, 이를 기반으로 라인의 끝을 구분해야 한다.

> <strong>CRLF</strong>
>
> CR = Carriage Return (\r), 커서를 라인의 시작부분으로 옮김
>
> LF = Line Feed(\n), 커서를 다음줄로 이동
>
> 옛 타자기에 기반한 문자

### Message Types

HTTP/1.1(RFC 2616)의 `4.1 Message Types`를 참고

Request Message 혹은 Response Message 둘 중 하나이며, 이들은 `generic-message` 포맷을 따른다.

```
generic-message = 	start-line
					*(message-header CRLF)
					CRLF
					[ message-body ]
start-line		=	Request-Line | Status-Line
```

이 때, Request Message가 들어 올 것으로 예상되면 빈 라인은 무시하는데, 이 말인 즉, CRLF가 먼저 읽히면 이를 무시하는 것과 유사하다

### Message Headers

HTTP/1.1(RFC 2616)의 `4.2 Message Headers`를 참고

request-header, response-header, generic-header, entity-header들은 아래 `message-header`포맷을 따른다

```
message-header  = field-name ":" [field-value]
field-name		= token
field-value		= *( field-content | LWS(Linear White Space) )
field-content	= <the OCTECTs making up the field-value and consisting of 
				either *TEXT or combinations of token, separators, and
				quoted-sting
```

### Message Body

HTTP/1.1(RFC 2616)의 `4.3 Message Body`를 참고

```
message-body = entity-body
			 | < entity-body encoded as per Transfer-Encoding >
```

### Message Length

메시지 바디의 길이는 다음 3가지 중 하나에 의해 결정된다.

- 상태코드가 1XX, 204, 304 같은 응답 메시지는 메시지 바디를 포함하면 안된다.

  이 경우 entity-header가 존재하는 것에 관계없이 헤더 필드 이후 첫번째 빈 줄(empty line)로 끝나야 한다.

  (빈줄 : CRLF로만 구성된 줄)

- 만약 Content-Length 필드가 헤더에 존재하면 메시지 바디에 존재하는 바이트 단위의 길이임

  이 필드가 헤더에 존재하지 않으면, 0이라고 추정하면 된다.

- 서버와의 연결을 끊을 때, request body의 끝을 지정하지 않는데, 이는 서버로부터 응답을 받지 않기 때문

RTSP는 HTTP/1.1의 `chunked`를 지원하지 않으며, 이 때문에 Contetnt-Length 헤더 필드가 존재해야 한다.
