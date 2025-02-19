# RFC 2326 : RTSP 1.0

## Entity

요청(Request), 응답(Response)에는 엔티티를 전송 할 수도, 안 할 수도 있다.

이 때, 엔티티는 헤더와 바디로 이루어져 있으며, 몇몇 응답은 엔티티 헤더만을 포함 할 수도 있다.

### Entity Header

엔티티의 헤더는 엔티티 바디에 대한 메타정보, 바디가 없다면, 요청에 대한 자원을 식별정보임

#### Allow

해당 URI 리소스에 대해 지원하는 메소드를 나열하는 헤더

`Public`과 차이점은 `Public`은 `OPTION` 요청에 대한 응답이라면, 해당 헤더는 지원하지 않는 메소드를 요청 했을 때의 반환이다.

즉, "405 Method not allowed" 응답에 반드시 있어야하는 헤더이다.

#### Content-Base

entity-body의 리소스 base-URL

```
Content-Base: http://www.example.com/video/
```

#### Content-Encoding

엔티티 바디에 적용된 인코딩의 종류를 나타내는 헤더

이 인코딩을 이용하여 클라이언트는 디코딩을 진행해야하며, 여러 인코딩인 적용된 경우 인코딩이 적용된 순서대로 나열된다.

프록시는 `no-transform` 지시어가 없을 때만 인코딩을 변경 할 수 있다.

이 때, 서버가 요청 메시지를 인코딩을 처리 할 수 없으면, "415 Unsupported Media Type" 코드로 응답해야 함

``` 
Content-Encoding: gzip, default
```

#### Content-Language

엔티티 바디가 사용하는 자연 언어를 지칭

#### Content-Length

컨텐츠의 길이를 나타내는 헤더(바이트 단위)

이 때, 컨텐츠는 헤더의 마지막에 오는 연속된 2개의 CRLF 이후의 것을 의미한다.

HTTP와는 다르게, 헤더 이후의 메시지의 길이를 반드시 전달해야 한다. 만약 생략되었을 경우, 기본 값인 0이 적용된다.

#### Content-Location

요청에 대해 해당 자원이 위치하는 실제 위치를 나타낸다

요청된 URL와는 별도로 현재 반환된 엔티티의 실제 위치를 나타낸다.

이는 URI와 다른 위치에서 엔티티를 제공할 때, 명확한 위치를 나타내기 위함으로 예를들면, 웹사이트에서 `/home`을 요청 했을 때, `/home/ko.html`을 응답 받을 수 있다.

#### Content-Type

엔티티 바디의 미디어 타입을 지정하는 헤더

```
Content-Type: application/sdp
```

#### Expires

해당 데이터가 안정적이라고 할 수 있는 시간을 지정하는 헤더

주로 DESCRIBE 메소드에 대한 응답에 사용되며, 특정 날짜와 시간 이후에 특정 미디어의 설명이 더이상 유효하지 않음을 의미

날짜는 RFC 1123의 날짜형식으로 제공됨

특정 미래의 날짜로 지정하면 미디어 스트림이 캐시가능한 것으로 취급된다. 하지만 `Cahce-Control`헤더가 별도로 존재한다면, 그 헤더의 설정에 따라 처리된다.

```
Expires: Thu, 01 Dec 1994 16:00:00 GMT
```

#### Last-Modified

서버의 프레젠테이션의 설명 혹은 미디어 스트림이 마지막으로 수정된 시점을 나타내는 헤더

서버가 해당 자원에 대해 최근의 수정 시점을 알려주는 역할을 수행

이는 클라이언트와 서버 간 유효성을 검사하기 위한 것으로 클라이언트가 가지고 있는 정보가 변경되었는지 확인하고, 캐시된 데이터를 사용 할 수 있다. 

이 때, `Last-Modified`로 받은 날짜를 `If-Modified-Since` 헤더를 이용하여 확인하고, 변경되지 않았따면, "304 Not Modified"응답을 받으면 캐시된 데이터를 사용한다.

변경되지 않더라도, 서버 측에서 이 값을 갱신 할 수도 있음

```
Last-Modified: Tue, 15 Nov 1994 12:45:26 GMT
```

### Entity Body

요청이나 응답에서 실제로 전송되는 데이터

이 때, 메시지 바디(message-body)나 엔티티 바디(entity-body)는 용어는 다르지만 동일한 개념으로 취급함

- Entity-body

  RTSP에서 전송되는 실제 데이터 부분을 의미

- Message-body

  이 데이터가 전송되는 형식과 전송방식(인코딩 및 전송방식)에 대한 세부 사항을 포함하는 넓은 개념

엔티티 본문은 OCTECT(byte)으로 이루어져 있음

#### Type

엔티티 바디의 데이터 타입은 `Content-Type`과 `Content-Encoding` 헤더에 의해 정의된다.

- Content-Type

  엔티티 본문의 실제 미디어 타입을 기술

  비디오 스트리밍의 경우 `application/sdp`와 같이 프리젠테이션의 설명을 담을 수도 있고,

  `application/mp4`와 같이 mp4비디오를 스트리밍 할 수 있다.

- Content-Encoding

  위에 `Content-Type`으로 정의된 미디어 타입에 추가적인 인코딩을 적용한 경우에 나타내며 위에 설명되어 있다.

#### Length

Entity Length는 메시지 본문의 길이를 의미한다. 이는 인코딩이 적용되기 이전 데이터의 실 길이를 의미한다.

`Content-Length`헤더로 정의되는 부분이다.

> <strong>`Transfer-Encoding`</strong>
>
> 이 헤더는 RTSP에서 다루지 않는다.
