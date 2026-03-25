# ☁ 근두운 (筋斗雲) - Gunduwoon

### `gdu` v0.2.0 — Lightweight QEMU Network Lab Emulator

> 손오공의 근두운처럼, 어디서나 가볍게 네트워크 랩을 펼칩니다.
> Run anywhere. Any hardware. Simple.

## 설치

### Ubuntu (권장: 22.04 LTS)
```bash
tar xzf gdu-v0.2.0.tar.gz -C ~/gdu && cd ~/gdu
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
sudo gdu-portfwd setup    # 최초 1회
```

## 플랫폼 비교

| 기능 | Ubuntu | WSL2 | Windows |
|------|--------|------|---------|
| KVM 가속 | ✅ | ✅ (5.10+) | △ WHPX |
| Bridge/TAP | ✅ | ✅ | △ TAP-Win |
| VNC/noVNC | ✅ | ✅→Win 브라우저 | ✅ |
| 성능 | ◎ | ○ | △ |

## 빠른 시작

```bash
sudo gdu shell

gdu> load examples/enterprise-gui-lab.yaml
gdu> start
gdu> list
gdu> console R1        # serial (라우터)
gdu> vnc ISE           # VNC viewer (ISE GUI)
gdu> browser ISE       # noVNC 웹 접속
gdu> spice WIN-DC      # SPICE viewer (Windows)
gdu> stop
```

## 토폴로지 YAML

### Serial 노드 (라우터/스위치)
```yaml
R1:
  image: "/opt/qemu-images/cat8000v.qcow2"
  node_type: router
  console_type: serial
  memory: 4096
```

### GUI 노드 (ISE, Palo Alto, Windows)
```yaml
ISE:
  image: "/opt/qemu-images/ise-3.3.qcow2"
  node_type: appliance
  console_type: vnc
  memory: 16384
  vcpus: 4
  display:
    type: vnc
    password: "cisco123"
  disk:
    bus: ide
```

## CLI 명령어

| 명령어 | 설명 |
|--------|------|
| `gdu start <topo.yaml>` | 토폴로지 시작 |
| `gdu stop` | 전체 중지 |
| `gdu list` | 노드 상태 |
| `gdu console <node>` | 자동 감지 콘솔 |
| `gdu vnc <node>` | VNC 뷰어 연결 |
| `gdu spice <node>` | SPICE 뷰어 연결 |
| `gdu browser <node>` | noVNC 브라우저 |
| `gdu validate <topo>` | 토폴로지 검증 |
| `gdu shell` | 대화형 셸 |

## 지원 이미지

| 이미지 | 콘솔 | 메모리 |
|--------|------|--------|
| Cat8000v / CSR1000v / IOSv | serial | 4GB |
| n9kv (NX-OS) | serial | 8GB |
| Cisco ISE 3.x | vnc | 16GB |
| Cisco DNAC / FMC | vnc | 16-32GB |
| Palo Alto PA-VM | vnc | 6GB |
| Windows Server | spice | 8GB |

## WSL2 전용 도구

| 도구 | 설명 |
|------|------|
| `gdu-browser <port>` | Windows 브라우저에서 noVNC 열기 |
| `sudo gdu-portfwd setup` | WSL2 → Windows 포트 포워딩 |
| `sudo gdu-portfwd status` | 포워딩 상태 확인 |
| `sudo gdu-portfwd clean` | 포워딩 제거 |

## 프로젝트 구조

```
gdu/
├── Cargo.toml
├── install.sh                # Ubuntu
├── install-windows.ps1       # Windows
├── install-wsl2.sh           # WSL2
├── uninstall.sh / .ps1
├── examples/
│   ├── ccie-lab.yaml
│   └── enterprise-gui-lab.yaml
└── src/
    ├── main.rs               # CLI + Shell
    ├── topology/mod.rs        # YAML 파서
    ├── vm/mod.rs              # QEMU + VNC/SPICE
    ├── network/mod.rs         # Bridge/TAP
    └── console/mod.rs         # 콘솔 관리
```
