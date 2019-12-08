---
layout: post
title: Wireshark로 HTTPS 암호화 검증하기
categories: [HTTPS, Security]
description: proving https with Wireshark
keywords: HTTPS, Server, Security
---

최근 HTTPS 관련 작업을 하다가 문득 정말로 암호화가 되기는 하는 걸까? 라는 의문이 들기 시작하였습니다.

(~~사실 HTTPS를 적용하는 방법이 너무 간단해서 더 의심이 됐다는..~~)

이론상으로는 비대칭키를 이용한 암호화 통신을 통해 중간에서 정보가 탈취 되더라도 알아볼 수 없다 라고는 하는데 여전히 의구심이 들어 직접 확인해 보기로 했습니다.

### Wireshark

> Wireshark(와이어샤크)란 오픈 소스 패킷 분석 프로그램입니다. 현재 내 컴퓨터에서 주고받는 네트워크 패킷을 캡쳐 해서 세부 내용들을 확인해 볼 수 있으며 직접적인 연결 뿐만 아니라 브로드캐스트, 멀티캐스트 등의 내용도 확인할 수 있습니다.

실제로 컴퓨터에 연결된 케이블에서 무슨 일이 일어나는지 알아낼 수 있는 아주아주아주 중요하고 좋은 툴입니다.

과거 보안 관련 일을 할 때 DDOS바이러스에 걸린 컴퓨터가 특정 IP로 heartbeat(서버에 클라이언트가 살아있다는 정보)를 보내는 패킷을 캡쳐 해서 무슨 정보를 주고받고 있으며, 해당 컴퓨터 내 무슨 프로세스가 바이러스로 동작 하는지 파악할 수 있었습니다.

이제 본론으로 들어가자면,,

### HTTP 방식 GET REQUEST

먼저 HTTP로 통신하는 API서버에 GET REQUEST를 해보겠습니다. 통신 순서는 단순하게 클라이언트에서 GET REQUEST를 보내면 서버에서 해당 값을 RESPONSE해주는 1차원 적인 방식입니다.

![](/images/posts/https/http.png)

![](/images/posts/https/http-wireshark.png)

테스트용으로 만든 API서버에 HTTP GET REQUEST를 해보았습니다.

결과의 첫번째 줄은 클라이언트 → 서버 REQUEST, 두번째 줄은 서버 → 클라이언트 RESPONSE입니다

![](/images/posts/https/http-wireshark-request.png)

클라이언트 → 서버 GET REQUEST 패킷을 열어 보면 다음과 같이 어느 URL로 무슨 정보를 REQUEST했는지 볼 수 있습니다. 그 이외에도 네트워크 관련 많은 정보들을 보여주는데 이는 추후 Wireshark에 관련해서 별도로 다뤄 보도록 하겠습니다.

간혹 가다가 HTTP에서 header나 post를 이용하여 accessToken과 같은 중요한 정보를 숨기면 되지 않느냐는 질문을 하시는 분들이 계시는데 이는 URL에서만 보이지 않을 뿐 보안 적인 기능을 전혀 하지 않는 방법입니다.

실제로 테스트용 API서버는 accessToken을 header로만 주고받도록 되어 있는데 위와 같이 모든 정보들이 그대로 보여 지게 됩니다.

![](/images/posts/https/http-wireshark-response.png)

다음은 서버 → 클라이언트 RESPONSE내용 입니다. JSON형식으로 반환되는 데이터들이 key-value 별로 아주 예쁘게(?) 정리되어 있습니다.ㅎㅎ

### HTTPS 통신 방식

HTTPS의 통신 방식은 다음과 같습니다.

![](/images/posts/https/https-connection-sequence-diagram-968x624.png)

[이미지 출처](https://love2dev.com/blog/how-https-works/)

서버와 클라이언트에서 서로 인증서와 키를 주고 받은 다음 암호화된 데이터로 통신을 하는 방식이며, HTTPS 통신에 대한 기본 개념에 대해 잘 모르시는 분들은 [이전포스트](https://cydp0127.github.io/2019/12/04/understanding-of-https/) 를 확인해 주세요.

### HTTPS 방식 GET REQUEST

HTTPS로 동일한 API에 GET REQUEST를 해보았습니다.

![](/images/posts/https/https-wireshark.png)

가장 먼저 보이는 차이점은 Protocol이 이전에는 HTTP였다면 지금은 TCP와 [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security)로만 통신을 하게 됩니다.

위에서 부터 순서대로 서버와 클라이언트가 인증서와 키를 주고받은 다음 암호화 통신에 필요한 기본적인 정보들을 주고 받게 됩니다. HandShake 까지 완료가 되면 그 뒤로 REQUEST 와 RESPONSE가 진행됩니다.

![](/images/posts/https/https-wireshark-details.png)

Application Data 패킷을 열어 보면 Port 역시 HTTP가 사용하는 80포트가 아닌 443 포트를 사용하는 것을 볼 수 있으며, 모든 내용은 예상과 같이 암호화가 되어 보여 집니다.

### 마치는 글

이렇게 실제로 HTTPS가 암호화되어 통신을 하는지 확인해 보았습니다.

예전 같았으면 당연히 내부적으로 잘 동작 하겠지 라고 생각했겠지만 최근들어 당연한 것도 한번 더 확인해 보는 습관이 생긴 것 같습니다. 조금 귀찮긴 하지만 자세히 확인해보면 몰랐던 내용도 많이 알 수 있어서 좋은 경험이 된 것 같습니다.

질문이나 피드백이 있다면 언제든지 연락 주세요 .!

### Reference

[https://www.wireshark.org/docs/wsug_html_chunked/ChapterIntroduction.html#ChIntroWhatIs](https://www.wireshark.org/docs/wsug_html_chunked/ChapterIntroduction.html#ChIntroWhatIs)
