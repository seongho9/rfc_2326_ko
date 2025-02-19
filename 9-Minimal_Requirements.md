# RFC 2326 : RTSP 1.0

## Minimal Requirements

### Server

RTSP 서버의 최소한의 구현은 다음과 같다.

- 메소드

  SETUP, TEARDOWN, OPTIONS, PLAY(재생 서버라면), RECORD(녹화, 녹취 서버라면)

  RECORD가 구현되면, ANNOUNCE도 구현되어야 한다.

- 응답헤더

  Connection, Content-Length, Content-Type, Content-Language, Content-Encoding, Transport, Public.

  RECORD가 구현되면 Location은 반드시 있어야함

  RTP 호환 구현이면 RTP-Info도 있어야 함

- 요청헤더

  Connection, Session, Transport, Require

구현이 권장되는 사항은 다음과 같다.

- RTP/AVP/UDP를 유효한 전송방식으로 구현

  RTP : 실시간 멀티미디어 데이터를 전송하기 위한 프로토콜

  AVP : 오디오-비디오 프로파일, RTP에서 멀티미디어 스트리밍을 위한 표준 세트

  UDP: UDP

- Server 헤더 포함

- DESCRIBE 메서드 구현

- SDP 세션 정보의 생성

### Basic Playback

단순 재생을 위해서는 다음 동작을 수행해야 한다.

- Range 헤더를 인지해야 하며, 유효한 범위가 아닐시 에러를 반환해야 함
- PAUSE 메소드를 구현해야 한다.

추가적으로 권장 구현 사항이다

- Range 헤더 파싱시 NPT 단위 말고도 SMPTE단위로도 지원해야 한다.

- 미디어 재생길이 정보 포함

  미디어 초기화 시 미디어 프레젠테이션의 전체 길이를 포함하여 클라이언트에 제공해야함

- RTP 타임스탬프를 NPT로 변환하는 매핑 제공

  RTP를 사용하는 경우, RTP-Info 필드의 `rtptime`을 활용해 RTP 타임스탬프와 NPT를 매핑



