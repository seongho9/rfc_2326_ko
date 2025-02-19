# RFC 2326 : RTSP 1.0

## Request

client에서 server로 그리고 server에서 client로 보내는 모든 Request에는 Resource에 적용할 메서드, Resource 식별자 및 사용 중인 프로토콜 버전이 포함된다.

HTTP와는 다르게 언제나 절대 URL(absolute URL)이 사용되어야 한다

> <strong>absolule URL</strong>
>
> 스키마, 호스트 주소, 포트까지 전부 포함하는 URL

```
Request	=
		Request-Line		;
	*(	general-header		;
	|	request-header		;
	|	entity-header )		;
		CRLF
		[ message-body ]	;
```

### Request Line

```
Request-Line = Method SP Request-URI SP RTSP-Version CRLF
```

```
RTSP-Version 	= 	"RTSP" "/" 1*DIGIT "." 1*DIGIT
```

```
Request-URI		=	"*" | absolute_URI
```

### Method

```
Method			=	"DESCRIBE"			;
                |	"ANNOUNCE"			;
                |	"GET_PARAMETER"		;
                |	"OPTIONS"			;
                |	"PAUSE"				;
                |	"PLAY"				;
                |	"RECORD"			;
                |	"REDIRECT"			;
                |	"SETUP"				;
                |	"SET_PARAMETER"		;
                |	"TEARDOWN"			;
                |	extension-method
extension-method = token
```

#### DESCRIBE

요청한 URL로 식별되는 자원에 대한 정보를 검색하는 메소드

Accept 해더에는 받아오는 정보를 기술할 포맷을 지정해야 한다.

RTSP 연결 시작부분에서 사용하며, response는 반드시 모든 미디어에 대한 초기 연결 정보를 제공해야 한다.

서버는 DESCRIBE 응답을 이용해 미디어를 전송하는 간접 수단으로 이용하면 안되며, <strong>최소한의 기능만을 하도록 구현하는것이 좋다</strong>.

추가적으로 미디어 연결에 필요한 정보를 굳이 DESCRIBE로 전달 받을 필요는 없으며 아래 3가지 방법이 존재한다.

- RTSP DESCRIBE 메서드
- HTTP, email 첨부와 같은 다른 프로토콜
- command line 혹은 표준 입력

```
C->S : 
DESCRIBE rtsp://server.example.com/fizzle/foo RTSP/1.0CRLF
CSeq: 312CRLF
Accept: application/sdp, application/rtsl, application/mhegCRLF
CRLF
CRLF

S->C : 
RTSP/1.0 200 OK
CSeq: 312
Date: 23 Jan 1997 15:35:06 GMT
Content-Type: application/sdp
Content-Length: 376
v=0
o=mhandley 2890844526 2890842807 IN IP4 126.16.64.4
s=SDP Seminar
i=A Seminar on the session description protocol
u=http://www.cs.ucl.ac.uk/staff/M.Handley/sdp.03.ps
e=mjh@isi.edu (Mark Handley)
c=IN IP4 224.2.17.12/127
t=2873397496 2873404696
a=recvonly
m=audio 3456 RTP/AVP 0
m=video 2232 RTP/AVP 31
m=whiteboard 32416 UDP WB
a=orient:portrait
```

#### ANNOUNCE

요청을 보내는 주체가 누구냐에 따라 다르게 동작한다.

- client -> server

  미디어에 대한 description을 전송

- server -> client

  미디어에 대한 정보를 실시간으로 업데이트

```
C->S : 
ANNOUNCE rtsp://server.example.com/fizzle/foo RTSP/1.0
CSeq: 312
Date: 23 Jan 1997 15:35:06 GMT
Session: 47112344
Content-Type: application/sdp
Content-Length: 332

v=0
o=mhandley 2890844526 2890845468 IN IP4 126.16.64.4
s=SDP Seminar
i=A Seminar on the session description protocol
u=http://www.cs.ucl.ac.uk/staff/M.Handley/sdp.03.ps
e=mjh@isi.edu (Mark Handley)
c=IN IP4 224.2.17.12/127
t=2873397496 2873404696
a=recvonly
m=audio 3456 RTP/AVP 0
m=video 2232 RTP/AVP 31

S->C:
RTSP/1.0 200 OK
CSeq: 312
```

#### GET_PARAMETER

URI에 있는 스트림이나 presentation에 필요한 파라미터 값을 찾는 메소드

값의 표현은 특정하지 않고, 구현에 따른다.

body를 넣지 않으면 서버 ping 요청과 같이 동작

```
S->C:
GET_PARAMETER rtsp://example.com/fizzle/foo RTSP/1.0
CSeq: 431
Content-Type: text/parameters
Session: 12345678
Content-Length: 15

packets_received
jitter

C->S:
RTSP/1.0 200 OK
CSeq: 431
Content-Length: 46
Content-Type: text/parameters

packets_received: 10
jitter: 0.3838
```

#### OPTIONS

요청 URI에 대해 가능한지 여부를 확인하는 메소드(캐싱되지 않음)

만약 해당 요청에 body가 있다면, Content-Type 필드에 미디어 타입이 반드시 명시되어 있어야 한다. 이는 OPTION 메소드에서 사용하지는 않지만 추후 버전에서 필요로 할 수도 있고, 서버 구현에서 요구 할 수 있기 때문

요청 URI가 * 인 경우 특정 리소스에 대한 요청이 아닌 서버에 일반적인 경우에 대한 요청이다.

```
C->S:
OPTIONS * RTSP/1.0
CSeq: 1
Require: implicit-play
Proxy-Require: gzipped-messages

S->C:
RTSP/1.0 200 OK
CSeq: 1
Public: DESCRIBE, SETUP, TEARDOWN, PLAY, PAUSE
```

#### PAUSE

스트림의 전달을 일시적으로 멈추는 메소드

이 때, URI가 지칭하는 스트림의 녹화와 재생에 대해서만 멈춘다.

만약 presentation이나 스트림 그룹의 경우에는 모두 멈춘다.

이후에 다시 재생 혹은 녹취하는 경우에는 presentation이나 스트림 그룹에 대한 트랙 동기화는 반드시 유지해야 한다.

이 요청이 들어오더라도 서버는 해당 리소스를 유지하고 있어야 하며, 만약 SETUP 메소드에서 timeout이 설정되어 있다면, PAUSE이후 해당 기간만큼 지나면 리소스 할당을 해지한다.

```
C->S:
PAUSE rtsp://example.com/fizzle/foo RTSP/1.0
CSeq: 834
Session: 12345678

S->C:
RTSP/1.0 200 OK
CSeq: 834
Date: 23 Jan 1997 15:35:06 GMT
```

이 때, Range 헤더를 사용하면, 특정 지점에서 멈출 수 있다. 이 때, Range헤더의 값은 범위가 아닌 정확히 1개의 값만을 할당 할 수 있으며, NPT(Normal Paly Time)으로 기술해야 한다. 

위의 예시와 같이 Range헤더가 없으면 현재 재생 중인 부분에서 중지하며 해당 지점이 중단점이 된다.

Range 해더의 값이 범위를 벗어난 값이면 서버는 457 Invalid Range 오류를 반환한다.

해당 요청이 있은 이후 PLAY 요청에 의해 queue에 들어간 요청들은 전부 버려진다. 하지만 중단점은 유지한다.

```
ex1:
대기 중인 PLAY 요청
Range: 10-15
Range: 20-29
PAUSE 요청
Range: NPT=21 -> 서버는 2번째 PLAY요청을 시작하고 21에서 멈춤
Range: NPT=12 -> 서버가 14에서 재생 중이라면, 즉시 멈춤
Range: NPT=16 -> 첫 번째 요청인 10-15가 끝낸 후 멈추고, 두번재 요청은 폐기

ex2:
대기 중인 PLAY 요청
Range: 10-15
Range: 13-20
PAUSE 요청
Range: NPT=14 -> 서버가 첫 번째 요청을 재생 중이라면 14에서 멈추고, 두 번째 요청은 폐기
```

서버가 중단점 이후의 데이터를 전송 했다면, 클라이언트 단에서 폐기했다고 가정하고, 다음 PLAY요청이 있을 경우 중단점 이후부터 재생이 시작

#### PLAY

SETUP을 통해 설정된 데이터의 전송을 요청하는 메소드

클라이언트는 SETUP을 통해 데이터를 설정하는 것이 성공하지 않으면 해당 메소드로 요청을 보내면 안된다.

시작 지점은 Range 헤더에 지정 되어 있으며, Range의 끝 부부에 도달 할 때까지 데이터 스트림을 전송한다.

PLAY 요청은 파이프라인화가 가능한데, 여러 PLAY요청을 한번에 보낼 수 있음을 의미한다.

클라이언트가 보내는 요청은 서버에 의해 큐(queue)로 관리된다. 이 때, 처리하는 순서는 큐에 쌓인 순서대로 요청을 실행하고, 이전 요청이 아직 활성 상태(재생 중)이라면, 이후 요청은 지연된다. 즉 이전 요청이 끝나야 다음 요청이 실행 된다.

```
C->S:
PLAY rtsp://audio.example.com/audio RTSP/1.0
CSeq: 835
Session: 12345678
Range: npt=10-15

C->S:
PLAY rtsp://audio.example.com/audio RTSP/1.0
CSeq: 836
Session: 12345678
Range: npt=20-25

C->S:
PLAY rtsp://audio.example.com/audio RTSP/1.0
CSeq: 837
Session: 12345678
Range: npt=30-
```

위와 같이 요청이 들어오면, 10-15초 구간을 실행하고, 이후 20-25초, 마지막으로 30초부터 끝까지 실행한다.

만약 Range헤더가 없으면, PAUSE요청이 오지 않는 이상 시작부터 수행된다.

이전에 PAUSE 요청으로 멈춘 상태에서 PLAY요청이 들어오면 중단점에서 부터 스트림이 전달된다.

스트림이 이미 재생 중인 경우라면, 추가 동작은 수행되지 않는다. 이 때, 클라이언트는 PLAY요청을 통해 서버의 활성여부(liveness)를 확인 할 수 있다.

Range 헤더는 npt말고도 UTC 절대시간도 사용 할 수 있다.

이는 다중 스트립의 동기화를 위해 사용한다.

온디멘트 스트림의 경우 요청한 재생범위가 실 재생 범위와 다를 수 있다. 첫째로 요청한 범위의 경계가 실 데이터 스트림의 경계와 다를 경우 서버는 실제 값을 반환한다. 둘째로 범위가 없는 경우 현 위치에서 바로 재생을 시작한다. 요청은 위와 같이 npt, utc 절대시간, smpte 단위를 사용한다.

요청한 범위를 다 재생한 경우 미디어 재생은 PAUSE요청이 발생 했을 때와 같이 자동으로 멈춘다.

```
C->S:
PLAY rtsp://audio.example.com/twister.en RTSP/1.0
CSeq: 833
Session: 12345678
Range: smpte=0:10:20-;time=19970123T153600Z
S->C:
RTSP/1.0 200 OK
CSeq: 833
Date: 23 Jan 1997 15:35:06 GMT
Range: smpte=0:10:22-; time=19970123T153600Z
```

위 예시는 smpte시간으로 0:10:20에서 끝까지 1997년 01월 23일 15시 36분에 재생을 시작하라는 요청과 그에 따른 응답이다.

```
C->S:
PLAY rtsp://audio.example.com/meeting.en RTSP/1.0
CSeq: 835
Session: 12345678
Range: clock=19961108T142300Z-19961108T143520Z
S->C:
RTSP/1.0 200 OK
CSeq: 835
Date: 23 Jan 1997 15:35:06 GMT
```

이는 clock 단위의 Range헤더를 이용해서 녹화된 라이브 프레젠테이셔니의 특정 시간 국나을 재생하는 방법을 보여준다.

Range는 절대시간(UTC)로 지정하며, 1996년 11월 8일 14:23:00(UTC)에서 1996년 11월 8일 14:35:20(UTC)사이 재생 범위를 요청한다.

`clock`단위 사용은 녹화된 라이브 프레젠테이션처럼 절대 시간 정보가 있는 스트림에서 특정 시간 구간을 재생하려고 할때 유용하다.

#### RECORD

이 기능은 presentation의 Description에 따라 미디어 데이터를 저장하도록 하는 메소드

 Range 헤더에 UTC를 이용해 기록할 시간을 지정 할 수 있으며, 만약 Range헤더가 없다면, presentation에 기술된 description에 따라 시작과 종료 시잔을 설정한다.

저장 위치를 요청한 URI 혹은 다른 URI에 저장하도록 할 수 있는데, 다른 URI에 저장하도록 했다면, 201 Created 응답을 반환하고, 요청의 상태를 기술하고 저장한 위치를 반환해야한다.

이 때, 서버는 RECORD요청에 대해 반드시 clock 기반의 절대 시간으로만 지원해야 한다. 상대적인 smpte나  npt는 해당 요청에 맞지않은 방식임 

```
C->S: 
RECORD rtsp://example.com/meeting/audio.en RTSP/1.0
CSeq: 954
Session: 12345678
Conference: 128.16.64.19/32492374

S->C:
RTSP/1.0 200 OK
CSeq: 954
Date: 2025-01-11T15:00:00Z

S->C:
RTSP/1.0 201 Created
CSeq: 954
Date: 2025-01-11T15:00:00Z
Location: rtsp://example.com/recordings/audio12345
Content-Type: application/sdp
Content-Length: 123

(저장한 리소스에 대한 sdp)
```

이 때, `Conference`헤더의 경우 해당 서버가 특정 회의에 참여하고 있다면, 해당 회의의 ip와 포트번호를 기술해야 하며, 단순 자원에 대한 녹화요청이라면 사용하지 않아도 된다.

#### REDIRECT

클라이언트에게 다른 서버로 연결을 변경하도록 알리는 역할을 하는 메소드

이 때, REDIRECT에는 리다이렉트 할 새로운 서버의 URI를 명시하고, 클라이언트는 해당 URI로 연결을 설정해야 한다(Location).

어느 지점에서 REDIRECT가 일어났는지 알려주는 헤더도 필요하지만, 이는 상황에 따라 다르다(Range)

이 때, 클라이언트는 해당 서버와 TEARDOWN으로 연결을 끊고, 이후 SETUP을 이용 다른 서버와 연결을 생성한다.

```
S->C:
REDIRECT rtsp://example.com/fizzle/foo RTSP/1.0
CSeq: 732
Location: rtsp://bigserver.com:8001
Range: clock=19960213T143205Z-
```

#### SETUP

스트리밍 미디어를 사용하기 위해 URI에 전송 매커니즘을 설정하는 메소드

초기 설정 외에도 서버가 지원한다면, 이미 재생 중인 스트림에 대해서 전송 매개변수를 변경 할 수 있다. 만약 서버가 지원하지 않는다면, "455 Method Not Valid In This State"를 반환한다.

SETUP요청에 포함되어 있는 전송 정보는 방화벽이나 네트워크 장치가 데이터 전송을 설정하는데 도움을 준다.  이는 DESCRIBE 응답을 사용하는 것보다 네트워크 장치들에게 유리하게 데이터를 준비 할 수 있게 한다.

> <strong> SETUP과 DESCIRBE의 차이</strong>
>
> DESCIRBE는 해당 URI에 있는 자원에 대한 미디어 정보를 얻는데 사용된다.
>
> 즉, 미디어의 형식, 코덱, 해상도 등의 정보를 제공한다.
>
> SETUP은 네트워크에 가까운 정보가 포함되어 있다. 
>
> 포트 번호 및 프로토콜등의 정보를 제공한다. 

Transport 헤더는 클라이언트가 사용 할 수 있는 데이터 전송방식을 기술하는 헤더이다.

이에 대한 서버의 응답은 session 식별자를 생성하고 이를 SETUP 요청에 대한 응답으로 전송한다.

이 세션 식별자는 반드시 응답에 넣어야 하며, 안된다면 "459 Aggregate Operation Not Allowed" 에러를 반환한다.

```
C->S:
SETUP rtsp://example.com/foo/bar/baz.rm RTSP/1.0
CSeq: 302
Transport: RTP/AVP;unicast;client_port=4588-4589

S->C:
RTSP/1.0 200 OK
CSeq: 302
Date: 23 Jan 1997 15:35:06 GMT
Session: 47112344
Transport: RTP/AVP;unicast;client_port=4588-4589;server-port=6256-6257
```

#### SET_PARAMETER

URI에 의해 특징되는 스트림이나 표현(presentation)의 파라미터를 설정하는 메소드

1개의 요청에 1개의 파라미터 설정이 권장되며, 그 이유는 특정 파라미터의 설정이 실패 했을 경우에 대해 대비하기 위함이다.

하지만 카메라의 팬 틸트 설정과 같은 설정은 여러 파라미터 값을 일괄적으로 설정해야 하는데, 이는 원자적(atomic)으로 처리해야 한다(하나의 값이라도 실패하면 전부 실패로 처리).

서버는 같은 값으로 파라미터를 설정하는 것도 허용해야 한다. 하지만 특정 파라미터의 값은 변경을 제한 할 수도 있는데, 한번 설정되면 이후에는 변경할 수 없도록 제한할 수도 있다.

특히 네트워크 전송 관련 설정 값은 반드시 SETUP에서만 해야하며, 이는 방화벽을 고려한 설계이다

```ㅐ, 
C->S:
SET_PARAMETER rtsp://example.com/fizzle/foo RTSP/1.0
CSeq: 421
Content-length: 20
Content-type: text/parameters

barparam: barstuff

S->C:
RTSP/1.0 451 Invalid Parameter
Cseq: 421
Content-length: 10
Content-type: text/parameters

barparm
```

이 때, Content-type은 예시이며, 이는 구현에 따라 다른 포맷을 사용 할 수 있다.

#### TEARDOWN

해당 URI에 대한 스트림 전송을 멈추는 메소드

해당 URI에 대해 할당된 자원을 해지하는 동작을 수행한다.

해당 요청을 전송 한 이후에는 해당 세션 식별자는 더 이상 유효하지 않게 된다.

다시 미디어를 재생하려면 SETUP부터 시작해야함

```
C->S:
TEARDOWN rtsp://example.com/fizzle/foo RTSP/1.0
CSeq: 892
Session: 12345678

S->C:
RTSP/1.0 200 OK
CSeq: 892
```

### Request Header

```
request-header 	= 	Accpet				;
				|	Accept-Encoding		;
				|	Accept-Language 	;
				|	Authorization		;
				|	From				;
				|	If-Modified-Since	;
				|	Range				;
				|	Refer				;
				|	User-Agent			;
```

#### Accept

응답으로 받을 값의 표현을 지정하는 헤더

MIME 타입으로 받을 것

```
Accept: application/rtsl, application/sdp;
```

> <strong>application/sdp MIME</strong>
>
> 별도 필수, 추가 파라미터는 없음

#### Accept-Encoding

Accept 해더와 유사하며, 클라이언트가 사용가능한 인코딩 형식을 지정하는 헤더

클라이언트가 사용가능하며 서버로 부터 받을 수 있는 내용 압축방식을 지정한다.

```
Accept-Encoding: compress, gzip
Accept-Encoding: 
Accept-Encoding: *
Accept-Encoding: compress;q=0.5, gzip;q=1.0
Accept-Encoding: gzip;q=1.0, identity; q=0.5, *;q=0
```

q 파라미터는 우선순위를 나타내며, 높은 숫자가 우선권을 갖는다.

이 때, identity는 원본 데이터를 의미하며, 이는 `identity;q=0` 혹은 `*;q=0`으로 거부되는 경우를 제외하면 항상 허용된다.

#### Accept-Language

Accept 해더와 유사하며, 클라이언트가 사용가능한 자연언어(en, ko..)를 지정하는 헤더

Accept-Encoding과 유사하게 q파라미터로 우선순위를 지정 할 수 있다.

```
Accept-Language: da, en-gb;q=0.8, en;q=0.7
```

#### Authorization

클라이언트가 서버에 인증 정보를 제공하여 자신을 인증하는데 사용

보통 "401 Unauthorized" 응답을 받은 후 재요청 시 포함

```
Authorization: <credentials>
```

#### From

요청을 제어하는 사용자의 이메일 주소를 포함하는 해더

주소 형식은 RFC 822 혹은  RFC 1123에서 정의된 "mailbox" 형식을 따라야 함

클라이언트의 경우 From 헤더를 사용해서는 안되며, 로봇의 경우 포함하여 요청을 보내야 한다.

#### If-Modified-Since

DESCRIBE와 SETUP 메서드에서 사용되며, 요청을 조건부 요청으로 만듦

요청한 리소스가 지정 시간 이후에 수정되지 않았으면, 서버는 리소스의 설명을 반환하지 않거나, 스트림을 설정하지 않으며, "304 Not Modified" 응답을 반환

시각 지정은 HTTP-date를 사용하며 아래와 같이 기술한다.

```
If-Modified-Since: Sat, 29 Oct 1994 19:43:31 GMT
```

#### Range

smpte, npt, clock으로 나타내며, byte range는 의미 없으니 사용하지 말 것

이 값은 PLAY, RECORD에서 사용되는 헤더이다.

이는 반은 닫힌 구간(half-open interval)로 표현하며, 시작시간은 포함하고 종료시간은 포함하지 않는 형태로 사용된다.

time 매개변수를 UTC로 지정하여 스트림 전송을 언제 실행될지를 설정 할 수 있다.

> <strong> Half-Open Interval</strong>
>
> 수학적인 표현으로 [a, b)의 형태

```
Range: clock=19960213T143205Z-;time=19970123T143720Z
```

#### Referer

클라리언트가 요청하는 자원이 어디에서 왔는지를 표현하는 헤더

주로 오류추적, 백링크 추적, 캐시 최적화를 위해 사용된다.

보안상 유의해야 하며, 일부 경우에는 의도적으로 숨기기도 한다.

```
Referer: http://www.example.com/audio
Referer: /page
```

이는 RTSP 단독으로 사용되기 보다 HTTP를 통해 사용된다.

#### User-Agent

요청을 생성한 user agent(브라우저, 앱 등)에 대한 정보를 포함하는 헤더

통계, 프로토콜 위반 추적 및 user agent에 맞춘 최적화된 응답을 제공하기 위해 사용

```
User-Agent: CERN-LineMode/2.15 libwww/2.17b3
```

