# ☁ 근두운 (gdu) — QEMU 이미지 설정 가이드

## 이미지 디렉토리 구조

gdu 설치 후 기본 이미지 경로는 `/opt/qemu-images/` 입니다.

```
/opt/qemu-images/
├── routers/
│   ├── cat8000v-17.12.qcow2          # Cisco IOS-XE (Cat8000v)
│   ├── csr1000v-17.03.qcow2          # Cisco CSR1000v (레거시)
│   └── vios-adventerprisek9.qcow2    # Cisco IOSv (경량)
├── switches/
│   ├── n9kv-10.3.5.qcow2             # Cisco Nexus 9000v
│   └── n9kv-10.4.1.qcow2
├── security/
│   ├── vFTD-7.4.qcow2                # Cisco Firepower FTD
│   ├── vFMC-7.4.qcow2                # Firepower Management Center
│   ├── PA-VM-KVM-11.1.qcow2          # Palo Alto
│   └── FortiGate-VM64-KVM-7.4.qcow2  # Fortinet
├── appliances/
│   ├── ise-3.3.qcow2                 # Cisco ISE (+ disk2 필요)
│   ├── ise-3.3-disk2.qcow2           # ISE 2nd disk
│   ├── dnac-2.3.qcow2                # Cisco DNA Center
│   └── wlc9800-17.09.qcow2           # Cisco WLC 9800-CL
├── hosts/
│   ├── ubuntu-22.04-server.qcow2     # Linux 서버
│   ├── win2022-eval.qcow2            # Windows Server
│   └── alpine-3.18.qcow2             # 경량 Linux (테스트용)
└── iso/
    ├── ise-setup.iso                  # ISE 초기 설정 CD
    └── virtio-win.iso                 # Windows virtio 드라이버
```

## 이미지 넣는 법

### 방법 1: 직접 복사

```bash
# 디렉토리 생성
sudo mkdir -p /opt/qemu-images/{routers,switches,security,appliances,hosts,iso}

# 이미지 복사
sudo cp ~/Downloads/cat8000v-17.12.qcow2 /opt/qemu-images/routers/
sudo cp ~/Downloads/n9kv-10.3.5.qcow2 /opt/qemu-images/switches/
sudo cp ~/Downloads/ise-3.3.qcow2 /opt/qemu-images/appliances/

# 권한 설정
sudo chmod 644 /opt/qemu-images/**/*.qcow2
```

### 방법 2: WSL2에서 Windows → Linux 복사

```bash
# Windows Downloads에서 직접 복사
sudo cp /mnt/c/Users/<유저명>/Downloads/cat8000v-17.12.qcow2 /opt/qemu-images/routers/

# 또는 Windows 공유 폴더 (install-wsl2.sh가 자동 생성)
# C:\Users\<유저명>\qdyn-images\ 에 넣으면
# /opt/qemu-images/windows-shared/ 로 접근 가능
```

### 방법 3: SCP로 원격 전송

```bash
# 다른 서버에서 이미지 가져오기
scp user@eve-ng-server:/opt/unetlab/addons/qemu/cat8000v-17.12/virtioa.qcow2 \
    /opt/qemu-images/routers/cat8000v-17.12.qcow2
```

## EVE-NG에서 이미지 가져오기

기존 EVE-NG 서버에 이미지가 있다면 그대로 사용 가능합니다.

```bash
# EVE-NG 이미지 기본 경로
# /opt/unetlab/addons/qemu/<이미지명>/virtioa.qcow2

# 예시: Cat8000v
scp root@eve-ng:/opt/unetlab/addons/qemu/cat8000v-17.12.01/virtioa.qcow2 \
    /opt/qemu-images/routers/cat8000v-17.12.qcow2

# 예시: Nexus 9000v
scp root@eve-ng:/opt/unetlab/addons/qemu/n9kv-10.3.5/virtioa.qcow2 \
    /opt/qemu-images/switches/n9kv-10.3.5.qcow2

# 예시: ISE (디스크 2개)
scp root@eve-ng:/opt/unetlab/addons/qemu/ise-3.3/virtioa.qcow2 \
    /opt/qemu-images/appliances/ise-3.3.qcow2
scp root@eve-ng:/opt/unetlab/addons/qemu/ise-3.3/virtiob.qcow2 \
    /opt/qemu-images/appliances/ise-3.3-disk2.qcow2
```

## CML(VIRL)에서 이미지 가져오기

```bash
# CML 이미지 경로
# /var/local/virl2/dropfolder/ 또는 /var/lib/libvirt/images/

scp admin@cml-server:/var/lib/libvirt/images/cat8000v-17.12.qcow2 \
    /opt/qemu-images/routers/
```

## 토폴로지 YAML에서 참조

이미지를 넣은 후 토폴로지 YAML에서 경로를 지정합니다.

```yaml
nodes:
  R1:
    image: "/opt/qemu-images/routers/cat8000v-17.12.qcow2"
    node_type: router
    console_type: serial
    memory: 4096

  SW1:
    image: "/opt/qemu-images/switches/n9kv-10.3.5.qcow2"
    node_type: switch
    memory: 8192

  ISE:
    image: "/opt/qemu-images/appliances/ise-3.3.qcow2"
    node_type: appliance
    console_type: vnc
    memory: 16384
    disk:
      bus: ide
      extra_disks:
        - path: "/opt/qemu-images/appliances/ise-3.3-disk2.qcow2"
```

## 이미지별 주의사항

### Cat8000v / CSR1000v
- 디스크 버스: `virtio` (기본값, 변경 불필요)
- 최소 메모리: 4096 MB
- vCPU: 2개면 충분

### Nexus 9000v (n9kv)
- 디스크 버스: `virtio`
- 최소 메모리: **8192 MB** (이하로 내리면 부팅 안 됨)
- 첫 부팅 시간: 5~10분 (loader에서 대기)

### Cisco ISE
- 디스크 버스: **`ide`** (virtio 안 됨)
- NIC 모델: **`e1000`** (virtio 안 됨)
- 최소 메모리: **16384 MB**
- 디스크 2개 필수 (virtioa + virtiob)
- 첫 부팅: 15~20분

```yaml
ISE:
  image: "/opt/qemu-images/appliances/ise-3.3.qcow2"
  console_type: vnc
  memory: 16384
  vcpus: 4
  disk:
    bus: ide
    extra_disks:
      - path: "/opt/qemu-images/appliances/ise-3.3-disk2.qcow2"
  interfaces:
    - name: Eth0
      bridge: link_mgmt
      model: e1000
```

### Palo Alto PA-VM
- 디스크 버스: `virtio`
- 최소 메모리: 6144 MB
- 관리 인터페이스가 첫 번째 NIC

### Cisco vFTD / FMC
- 최소 메모리: 8192 MB
- 부트 순서 지정 필요

```yaml
FW1:
  image: "/opt/qemu-images/security/vFTD-7.4.qcow2"
  memory: 8192
  vcpus: 4
  boot:
    order: "c"
```

### Windows Server
- 디스크 버스: `virtio` (virtio 드라이버 ISO 필요)
- 또는 `ide` (드라이버 없이 바로 사용)
- USB 태블릿 디바이스 추가 권장 (마우스 동기화)

```yaml
WIN-DC:
  image: "/opt/qemu-images/hosts/win2022-eval.qcow2"
  console_type: spice
  memory: 8192
  disk:
    bus: virtio
  extra_qemu_args:
    - "-device"
    - "usb-tablet"
    - "-cdrom"
    - "/opt/qemu-images/iso/virtio-win.iso"
```

## 이미지 검증

```bash
# 이미지 정보 확인
qemu-img info /opt/qemu-images/routers/cat8000v-17.12.qcow2

# 출력 예시:
# image: cat8000v-17.12.qcow2
# file format: qcow2
# virtual size: 8 GiB
# disk size: 1.2 GiB

# 토폴로지 검증 (이미지 존재 여부 체크)
gdu validate my-lab.yaml
```

## 이미지 용량 참고

| 이미지 | 가상 크기 | 실제 디스크 | 비고 |
|--------|----------|-----------|------|
| Cat8000v | 8 GB | ~1.2 GB | |
| IOSv | 2 GB | ~200 MB | 가장 경량 |
| n9kv | 8 GB | ~1.5 GB | |
| ISE 3.x | 300 GB | ~8 GB | disk1 + disk2 합계 |
| vFTD | 50 GB | ~3 GB | |
| PA-VM | 60 GB | ~3 GB | |
| Win2022 | 40 GB | ~8 GB | |

gdu는 `snapshot=on` 모드로 실행하므로 원본 이미지는 변경되지 않습니다. VM이 쓰는 변경분만 임시 파일에 기록됩니다.
