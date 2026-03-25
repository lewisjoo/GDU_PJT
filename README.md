# ☁ 근두운 (筋斗雲) - Gunduwoon

### `gdu` v0.3.0 — Lightweight QEMU Network Lab Emulator

> 손오공의 근두운처럼, 어디서나 가볍게 네트워크 랩을 펼칩니다.
> Run anywhere. Any hardware. Simple.

---

## 왜 근두운인가?

EVE-NG로 CCIE 풀 랩을 돌리면 RAM 90GB를 먹습니다.
근두운은 KSM + Balloon 메모리 최적화로 **동일 랩을 48~63GB에서 구동**합니다.

| 환경 | CCIE 풀 랩 RAM |
|------|---------------|
| EVE-NG (기존) | ~90 GB |
| gdu + KSM/Balloon | **~48~63 GB** |
| 절감률 | **30~47%** |

YAML 한 장으로 토폴로지를 정의하고, 한 명령으로 전체 랩을 올립니다.
CLI 기반이라 가볍고, VNC/SPICE로 ISE 같은 GUI 어플라이언스도 브라우저에서 접속합니다.

---

## 설치

### Ubuntu (권장: 22.04 LTS)
```bash
tar xzf gdu-v0.3.0.tar.gz -C ~/gdu && cd ~/gdu
sudo ./install.sh
```

### Windows (Native)
```powershell
Set-ExecutionPolicy Bypass -Scope Process
.\install-windows.ps1
```

### WSL2 (Windows + Linux)
```bash
sudo ./install-wsl2.sh
sudo gdu-portfwd setup    # 최초 1회: Windows localhost 접속 활성화
```

### 플랫폼 비교

| 기능 | Ubuntu | WSL2 | Windows |
|------|--------|------|---------|
| KVM 가속 | ✅ | ✅ (5.10+) | △ WHPX |
| Bridge/TAP | ✅ | ✅ | △ TAP-Win |
| VNC/noVNC | ✅ | ✅→Win 브라우저 | ✅ |
| KSM 메모리 최적화 | ✅ | ✅ | ✗ |
| 성능 | ◎ 최고 | ○ 좋음 | △ 보통 |
| 권장도 | ★★★ | ★★☆ | ★☆☆ |

---

## 빠른 시작

```bash
sudo gdu shell

gdu> load examples/enterprise-gui-lab.yaml
gdu> start
gdu> list
gdu> console R1        # serial (라우터)
gdu> vnc ISE           # VNC viewer (ISE GUI)
gdu> browser ISE       # noVNC 웹 브라우저 접속
gdu> spice WIN-DC      # SPICE viewer (Windows)
gdu> memory            # 메모리 최적화 상태
gdu> estimate          # 메모리 예측
gdu> stop
```

---

## QEMU 이미지 설정

이미지 기본 경로: `/opt/qemu-images/`

```
/opt/qemu-images/
├── routers/       cat8000v, CSR1000v, IOSv
├── switches/      n9kv
├── security/      vFTD, PA-VM, FortiGate
├── appliances/    ISE, DNAC, WLC
├── hosts/         Ubuntu, Windows Server
└── iso/           설치 CD, virtio 드라이버
```

EVE-NG에서 기존 이미지 가져오기:
```bash
scp root@eve-ng:/opt/unetlab/addons/qemu/cat8000v-17.12.01/virtioa.qcow2 \
    /opt/qemu-images/routers/cat8000v-17.12.qcow2
```

`snapshot=on` 모드로 실행하므로 원본 이미지는 변경되지 않습니다.

---

## 토폴로지 YAML

### Serial 노드 (라우터/스위치)
```yaml
R1:
  image: "/opt/qemu-images/routers/cat8000v-17.12.qcow2"
  node_type: router
  console_type: serial
  memory: 4096
```

### GUI 노드 (ISE, Palo Alto, Windows)
```yaml
ISE:
  image: "/opt/qemu-images/appliances/ise-3.3.qcow2"
  node_type: appliance
  console_type: vnc
  memory: 16384
  vcpus: 4
  display:
    type: vnc
    password: "cisco123"
  disk:
    bus: ide
    extra_disks:
      - path: "/opt/qemu-images/appliances/ise-3.3-disk2.qcow2"
  interfaces:
    - name: Eth0
      bridge: link_mgmt
      model: e1000
```

### 메모리 최적화 설정
```yaml
defaults:
  memory: 4096
  vcpus: 2
  console_base_port: 5000
  optimization:
    ksm: true
    ksm_profile: aggressive    # conservative | balanced | aggressive | maximum
    balloon: true
    hugepages: false
    balloon_headroom_pct: 20
```

---

## CLI 명령어

| 명령어 | 설명 |
|--------|------|
| `gdu start <topo.yaml>` | 토폴로지 시작 (KSM/Balloon 자동 활성화) |
| `gdu stop` | 전체 중지 (KSM 원상 복구) |
| `gdu list` | 노드 상태 + 할당 메모리 |
| `gdu console <node>` | 자동 감지 콘솔 (serial/VNC/SPICE) |
| `gdu vnc <node>` | VNC 뷰어 연결 |
| `gdu spice <node>` | SPICE 뷰어 연결 |
| `gdu browser <node>` | noVNC 브라우저 접속 |
| `gdu validate <topo>` | 토폴로지 검증 |
| `gdu shell` | 대화형 셸 |

## Shell 명령어

```
── 토폴로지 ──
load <file.yaml>       토폴로지 로드
start                  전체 시작
stop                   전체 중지
list / ls              노드 상태
bridges / net          네트워크 브리지 상태
topo                   토폴로지 상세 + 총 메모리

── 콘솔 ──
console <node>         자동 감지 (serial/VNC/SPICE)
serial <node>          강제 시리얼 (telnet)
vnc <node>             VNC 뷰어
spice <node>           SPICE 뷰어
browser <node>         noVNC 웹 브라우저

── 메모리 최적화 ──
memory / mem           호스트 + KSM + Hugepages 상태
ksm                    KSM 통계 (절감된 메모리)
balloon <node>         VM balloon 상태 조회
balloon <node> <mb>    VM 메모리 축소/확장
estimate               토폴로지 메모리 사용량 예측

── 기타 ──
quit / exit            전체 중지 후 종료
help / ?               도움말
```

---

## 지원 이미지

| 이미지 | 콘솔 | 최소 RAM | 디스크 버스 | 비고 |
|--------|------|---------|-----------|------|
| Cat8000v / CSR1000v | serial | 4 GB | virtio | |
| IOSv / vIOS | serial | 512 MB | virtio | 가장 경량 |
| n9kv (NX-OS) | serial | 8 GB | virtio | 부팅 5~10분 |
| Cisco ISE 3.x | vnc | 16 GB | **ide** | NIC: e1000, 디스크 2개 |
| Cisco DNAC | vnc | 32 GB | ide | |
| Cisco vFTD / FMC | vnc | 8 GB | virtio | boot order 필요 |
| Cisco WLC 9800-CL | serial | 4 GB | virtio | |
| Palo Alto PA-VM | vnc | 6 GB | virtio | |
| FortiGate VM | serial | 2 GB | virtio | |
| Windows Server | spice | 8 GB | virtio | usb-tablet 추가 권장 |
| Ubuntu / Alpine | serial | 1~2 GB | virtio | |

---

## WSL2 전용 도구

| 도구 | 설명 |
|------|------|
| `gdu-browser <port>` | Windows 기본 브라우저에서 noVNC 열기 |
| `sudo gdu-portfwd setup` | WSL2 → Windows 포트 포워딩 설정 |
| `sudo gdu-portfwd status` | 포워딩 상태 확인 |
| `sudo gdu-portfwd clean` | 포워딩 제거 |

---

## 메모리 최적화 엔진

### 3-Layer Architecture

```
Layer 1: KSM (Kernel Same-page Merging)
  커널이 동일 메모리 페이지를 탐지 → 하나로 병합
  Cat8000v 10대 = OS 커널 페이지 55% 공유

Layer 2: Virtio-Balloon
  VM이 실제로 안 쓰는 메모리를 호스트에 반환
  할당 4GB → 실사용 1.5GB → 2.5GB 반환

Layer 3: Hugepages (선택)
  2MB 대형 페이지로 TLB miss 감소
  성능 10~15% 향상 (메모리 절감 아님)
```

### KSM 프로파일

| 프로파일 | scan pages | sleep | CPU 부하 | 병합 속도 |
|---------|-----------|-------|---------|----------|
| conservative | 200 | 100ms | 낮음 | 느림 |
| balanced | 1,000 | 20ms | 보통 | 보통 |
| aggressive | 5,000 | 10ms | 높음 | 빠름 |
| maximum | 10,000 | 5ms | 매우 높음 | 최대 |

### estimate 출력 예시 (CCIE 풀 랩)

```
  ☁ Memory Estimate
  ─────────────────────────────────────────────
  Nodes: 22 (5 unique images)

  IMAGE                     COUNT  PER-VM     SUBTOTAL
  ───────────────────────────────────────────────────────
  cat8000v-17.12            10     4096 MB    40960 MB
  n9kv-10.3.5               4     8192 MB    32768 MB
  ise-3.3                    1     16384 MB   16384 MB
  vFTD-7.4                   2     8192 MB    16384 MB
  ubuntu-22.04               4     1024 MB     4096 MB

  Raw allocation:    110592 MB
  KSM savings:       -49766 MB (45%)
  Balloon savings:   -16588 MB (15%)
  ────────────────────────────
  Estimated:          44238 MB
  Total savings:     ~60%
  ─────────────────────────────────────────────
```

---

## 프로젝트 구조

```
gdu/
├── Cargo.toml                    # Rust 프로젝트 (v0.3.0)
├── install.sh                    # Ubuntu 설치
├── install-windows.ps1           # Windows 설치
├── install-wsl2.sh               # WSL2 설치
├── uninstall.sh / .ps1           # 삭제
├── examples/
│   ├── ccie-lab.yaml             # CCIE Serial 전용 랩
│   └── enterprise-gui-lab.yaml   # GUI 포함 엔터프라이즈 랩
└── src/                          # 2,134줄
    ├── main.rs                   # CLI + Interactive Shell (572줄)
    ├── topology/mod.rs           # YAML 파서 + 최적화 설정 (264줄)
    ├── vm/mod.rs                 # QEMU VM 관리 + VNC/SPICE (416줄)
    ├── memory/mod.rs             # KSM/Balloon/Hugepages 엔진 (459줄)
    ├── network/mod.rs            # Linux Bridge/TAP 관리 (167줄)
    └── console/mod.rs            # Serial/VNC/SPICE 콘솔 (256줄)
```

---

## Changelog

### v0.3.0 — Memory Optimization Engine
- KSM (Kernel Same-page Merging) 엔진 추가
  - 4단계 프로파일: conservative / balanced / aggressive / maximum
  - 실시간 KSM 통계 (pages_shared, pages_sharing, saved MB)
  - 기존 KSM 상태 백업/복원
- Virtio-Balloon 통합
  - QEMU에 `virtio-balloon-pci,deflate-on-oom=on` 자동 주입
  - QEMU 모니터 경유 balloon 조회/설정 (`balloon <node> <mb>`)
- Hugepages 지원 (선택적)
  - 2MB 대형 페이지 자동 할당/해제
  - QEMU `-mem-path /dev/hugepages` 자동 적용
- `estimate` 명령어: 토폴로지 메모리 사용량 예측
  - 이미지 그룹별 KSM 절감률 자동 계산
  - Balloon 예상 절감 포함
- Shell 명령어 추가: `memory`, `ksm`, `balloon`, `estimate`
- 토폴로지 YAML에 `optimization` 섹션 추가
- VM별 할당 메모리 `list`에 표시

### v0.2.0 — GUI Display + Multi-Platform
- VNC/SPICE GUI 콘솔 지원 (ISE, Palo Alto, Windows Server)
- noVNC 웹 브라우저 접속 (websockify 자동 실행)
- `console_type: vnc | spice | serial` 자동 감지
- `display` 설정: password, listen 주소, web_port
- `disk` 설정: bus (virtio/ide/scsi), extra_disks, format
- `boot` 설정: order, cdrom
- `node_type: appliance` 추가
- Shell 명령어 추가: `vnc`, `spice`, `browser`, `serial`
- GUI 의존성 자동 체크 (websockify, tigervnc, virt-viewer)
- Windows Native 설치 스크립트 (`install-windows.ps1`)
- WSL2 설치 스크립트 (`install-wsl2.sh`)
  - `gdu-portfwd`: WSL2 → Windows 포트 포워딩
  - `gdu-browser`: Windows 브라우저 자동 실행
  - `.wslconfig` 자동 생성 (nestedVirtualization)

### v0.1.0 — Initial Release
- Dynamips/Dynagen에서 영감을 받은 CLI 기반 QEMU 네트워크 랩 매니저
- YAML 토폴로지 정의 (nodes, links, interfaces)
- QEMU VM 생성/삭제/관리
- Linux Bridge + TAP 자동 생성/정리
- Telnet 시리얼 콘솔 접속
- Interactive Shell (`gdu shell`)
- Snapshot 모드 (원본 이미지 보존)
- 결정론적 MAC 주소 생성
- Ubuntu 설치 스크립트 + systemd 서비스
- Bash 자동완성

---

## 요구사항

| 환경 | 요구사항 |
|------|---------|
| Ubuntu | 22.04 LTS, QEMU 6.0+, Rust 1.70+, KVM, root |
| WSL2 | Windows 11, 커널 5.10+, `nestedVirtualization=true` |
| Windows | Windows 10/11, QEMU for Windows, WHPX 또는 HAXM |

---

## License

MIT
