# RFC 2326 : RTSP 1.0

## Protocol Parameters

### RTSP Version

```
RTSP-Version = "RTSP" "/" 1*DIGIT "." 1*DIGIT
```

### RTSP URL

`rtsp`와 `rtspu`가 사용된다.

```
rtsp_URL = ("rtsp:" | "rtspu:") "//" host [":" port] [abs_path]
host = < A legal Internet host domain name of IP address(dotted decimal form) >
port = *DIGIT
abs_path = "/" [ path ] [";" params] ["?" query] (Defined in RFC1808 2.2)
```

`rtsp`는 TCP를 통해 전송되는 경우 `rtspu`는 UDP를 통해 전송되는 경우 사용

port 번호가 비어있는 경우 :554를 기본적으로 사용

### Conference Identifiers



### Session Identifiers

session identifier는 임의 길이의 문자열이다.

임의로 생성되며, 최소 8 octet(64bit)길이를 만족해야한다.

```
session-id = 1*( ALPHA | DIGIT | safe)
```

### SMPTE Relative Timestamps

clip의 시작 시점으로 부터 상대적인 timestamps

프레임 단위로 접근하는 방식으로 `hours:minutes:seconds:frames.subframes`의 형태로 표현됨

기본 SMPTE 형식은 SMPTE 30 drop 형식으로 프레임 속도는 29.97fps(`smpte`)

SMPTE-25와 같은 형식도 지원함(25fps)

``` 
smpte-range = smpte-type "=" smpte-time "-" [ smpte-time ]
smpte-type 	= "smpte" | "smpte-30-drop" | "smpte-25"
						; other timecodes may be added
smpte-time 	= 1*2DIGIT ":" 1*2DIGIT ":" 1*2DIGIT [ ":" 1*2DIGIT]
					[ "." 1*2DIGIT]
```

ex.

```
smpte = 10:12:33:20-
smpte = 10:07:33-
smpte = 10:07:00-10:07:33:05.01
smpte-25 = 10:07:00-10:07:33:05.01
```

### Normal Play Time

Presentation의 시작에 대한 절대적인 위치를 표현하는 timestamp

소수점 이하의 숫자까지 포함하여 표현되며, 소수점 윗부분은 시간, 분, 초를 표현하며, 소수점 부분은 초이하의 부분을 표현한다.

Presentation의 시작 부분은 0이고, 음수 표현은 정의되지 않는다. 특수 상수 `now`는 현 시점을 나타내며, 실시간 이벤트에서만 사용이 가능하다

```
npt-range 	= ( npt-time "-" [ npt-time ]) | ( "-" npt-time )
npt-time	= "now" | npt-sec | npt-hhmmss
npt-sec		= 1*DIGIT [ "." *DIGIT ]
npt-hhmmss	= npt-hh ":" npt-mm ":" npt-ss [ "." *DIGIT]
npt-hh		= 1*DIGIT	; any positive number
npt-mm		= 1*2DIGIT	; 0-59
npt-ss		= 1*2DIGIT	; 0-59
```

ex.

```
npt=123.45-125
npt=12:05:35.3-
npt=now-
```

### Absolute Time

UTC 시간을 이용한 ISO 8601 timestamp (GMT)

```
utc-range	= "clock" "=" utc-time "-" [ utc-time ]
utc-time	= utc-date "T" utc-time "Z"
utc-date	= 8DIGIT					; <YYYYMMDD>
utc-time	= 6DIGIT[ "."" fraction ]	; <HHMMSS.fraction>
```

ex.

```
19961108T143720.25Z
```

### Option Tags

유일한 식별자로 RTSP에서 새로운 옵션을 지정하기 위해 사용됨

`Require`, `Proxy-Require` 헤더에 사용됨

```
option-tag		=	1*xchar
```

> <strong>추가 Option Tag 등록(IANA)</strong>
>
> 추가적으로 RTSP 옵션을 추가하려면 다음을 따라야 한다.
>
> - 옵션의 이름의 길이는 상관 없지만 20 character를 초과하는 것은 지양
> - 이름은 공백, 제어문자, 마침표들을 포함해서는 안됨
> - 옵션을 변경한 단체를 표기

