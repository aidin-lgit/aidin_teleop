+++
title = "Meta Quest 셋업"
weight = 4
+++

## 초기 설정

1. Quest 부팅 후 동일 Wi-Fi (5GHz) 접속
2. 개발자 모드 활성화 (Meta Quest 개발자 계정 필요)


## 시점 영상 수신

1. Quest 내 **OpenVR(WebXR) 지원 브라우저 앱**(예: Meta Quest Browser, Wolvic 등)을 실행합니다.
2. 주소창에 텔레옵 PC 에서 실행 중인 **스트리밍 서버 주소**를 입력해 접속합니다.

   ```
   https://<teleop_pc_ip>:<port>
   ```

   예: `https://192.168.2.31:8443`

3. 페이지가 정상 로드되면 좌/우 시점 영상이 헤드셋 안에서 스테레오로 표시됩니다.

{{% notice style="warning" title="HTTPS 보안 인증서 오류" %}}
스트리밍 서버는 로컬 자체 서명(self-signed) 인증서를 사용하므로 브라우저가 **"이 사이트는 안전하지 않습니다"** 또는 **`NET::ERR_CERT_AUTHORITY_INVALID`** 등 보안 경고를 표시합니다.

경고 화면에서 **Advanced** (고급) → **Proceed to `<주소>` (unsafe)** (해당 사이트로 계속 진행) 를 눌러 진행하세요. 신뢰된 내부 망에서만 사용하므로 안전하며, 한 번 허용해두면 같은 도메인에 대해 다음 접속부터는 경고가 표시되지 않습니다.
{{% /notice %}}

