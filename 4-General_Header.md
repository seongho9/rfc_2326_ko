# RFC 2326 : RTSP 1.0

## General-Header

### Cache-Control

Cache 지시어는 반드시 프록시 또는 게이트웨이를 통해 전달되어야 한다.

따라서 Cache 지시어는 특정 캐시를 지정하여 설정 할 수 없고, 전역적으로 관리된다. 즉, 개별 캐시의 특성에 맞춰 조정 할 수 없다.

Cache-Control은 `SETUP` 요청과 응답에만 기술된다. 이 때, Cache-Control은 응답에 대한 캐싱은 제어하지 않으며, `SETUP`요청으로 식별된 미디어 스트림의 캐싱만 제어한다.

`DESCRIBE`를 제외한 RTSP의 응답은 캐싱하지 않는다.

```
Cache-Control			= "Cache-Control" ":" !#cache-directive
cache-directive			= cache-request-directive
						| cache-response-directive
cache-request-directive = "no-cache"
						| "max-stale"
						| "min-fresh"
						| "only-if-cached"
						| cache-extension
cache-response-directive = "public"
						| "private"
						| "no-cache"
						| "no-transform"
						| "must-revalidate"
						| "proxy-revalidate"
						| "max-age" "=" delta-seconds
cache-extension			= token [ "=" ( token| quoted-string )]
```

- no-cache

  Media stream이 어느 곳에라도 캐싱되면 안됨을 의미

- public

  Media stream이 어느 곳에도 캐싱될 수 있음을 의미

- private

  한명의 유저를 위한 Media stream으로 공유가능한(shared) 공간에 캐싱되면 절대 안되며, 공유되지 않는(non-shared) 사설 캐시에 캐싱되어야 함

- no-transform

  프록시와 같은 중간 캐쉬는 미디어의 타입을 특정 스트림으로 바꾸는 기능이 존재한다(트래픽 부화 감소 및 용량 저감 목적)

  의학 이미지나 과학용 이미지와 같은 이미지나 암호화 된 이미지의 경우 이러한 기능이 동작하면 안된다.

  따라서 해당 내용은 캐싱되면 안됨을 의미한다.

- only-if-cached

  네트워크 연결이 불안정한 경우 미디어 스트림을 오리진 서버가 아닌 캐싱된 데이터만을 원할 수 있다.

  이 때 해당 요청을 추가하면 된다. 데이터가 없으면 504 Gateway Timeout을 응답한다.

- max-stale

  해당 값을 수명을 초과한 캐시데이터만을 가져옴

  만약 값이 있으면 해당 값을 초과한 데이터만을, 값이 없으면 accept a stable response of any age

- min-fresh

  현 시점에서 지정된 시점까지 유효한 데이터만을 가져옴

  즉, 지정 시점까지유효한 데이터만을 가져온다

- must-revalidate

  캐시가 유효시간이라면 캐시를 이용, 만료 후 라면 최초 조회시 원 서버에 검증

### Connection

전송 이후 네트워크 접속을 유지할지 말지를 제어

```
Connection = "Connection" ":" 1#(Connection-token)
connection-token = token
```

프록시는 메시지를 포워딩 하기 이전에 반드시 해당 내용을 파싱해야 하며, 중복되는 값은 제거한다.

해당 헤더는 `Cache-Control`과 같은 end-to-end 헤더에 포함되지 않아야 한다.

### Date

```
Date = "Date" ":" HTTP-date
```

ex.

```
Date: Tue, 15 Nov 1994 08:12:32 GMT
```

Origin 서버는 반드시 해당 필드를 아래 예외를 제외한 모든 응답에 포함해야 한다.

- 응답의 상태코드가 `100`, `101`인 경우
- `500` 에러와 같은 서버 에러인 경우
- 서버가 현 시각을 측정 할 수 없는 경우

### Via

사용된 프록시 프로토콜을 기술하는 부분

```
Via 				= "Via" ":" 1#( received-protocol received-by [ comment ])
received-protocol 	= [ protocol-name "/" ] protocol-version
received-by			= ( host [ ":" port]) | pseudonym
```

