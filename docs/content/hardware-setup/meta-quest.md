+++
title = "Meta Quest 셋업"
weight = 4
+++

## 초기 설정

1. Quest 부팅 후 동일 네트워크의 Wi-Fi (5GHz) 접속
2. 개발자 모드 활성화 (Meta Quest 개발자 계정 필요)


## 시점 영상 수신

영상 송출은 **NVIDIA CloudXR** 기반이며, 헤드셋 측 클라이언트는 CloudXR.js WebXR 페이지입니다. 별도 앱 설치 없이 브라우저 URL 접속만으로 동작합니다.

1. Quest 내 **WebXR 지원 브라우저 앱**(예: Meta Quest Browser, Wolvic 등)을 실행합니다.
2. 먼저 **WSS 프록시 주소**에 접속해 인증서 경고를 수락합니다. 시그널링 채널 TLS 를 헤드셋에 신뢰 등록하는 단계로, 건너뛰면 다음 단계에서 연결이 실패합니다.

   ```
   https://<teleop_pc_ip>:48322
   ```

3. 이어서 **web client 페이지**에 접속해 페이지 인증서 경고를 수락합니다.

   ```
   https://<teleop_pc_ip>:8080
   ```

   예: `https://192.168.2.31:8080`

4. Server IP = `<teleop_pc_ip>`, Port = 비움(→ 48322), Immersive Mode = **VR** 로 설정하고 **CONNECT** → **Enter XR** 을 누르면 좌/우 시점 영상이 헤드셋 안에서 스테레오로 표시됩니다.

전체 조작 절차와 런치 인자는 [원격제어 → 머리 카메라 제어 시 이미지 수신 방법](../../teleoperation/operation/#머리-카메라-제어-시-이미지-수신-방법) 을 참고하세요.

{{% notice style="warning" title="HTTPS 보안 인증서 오류" %}}
CloudXR 스택은 로컬 자체 서명(self-signed) 인증서를 사용하므로 브라우저가 **"이 사이트는 안전하지 않습니다"** 또는 **`NET::ERR_CERT_AUTHORITY_INVALID`** 등 보안 경고를 표시합니다.

경고 화면에서 **Advanced** (고급) → **Proceed to `<주소>` (unsafe)** (해당 사이트로 계속 진행) 를 눌러 진행하세요. 신뢰된 내부 망에서만 사용하므로 안전하며, 한 번 허용해두면 같은 도메인에 대해 다음 접속부터는 경고가 표시되지 않습니다.

**`48322` → `8080` 순서**로 두 주소 모두에서 수락해야 합니다. 인증서 SAN 에 서버 IP 가 포함되어 있지 않으면 수락해도 헤드셋이 신뢰하지 않습니다.
{{% /notice %}}

