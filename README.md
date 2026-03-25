gdu/
├── Cargo.toml               # name = "gdu"
├── README.md                # ☁ 근두운 브랜딩
├── install.sh               # Ubuntu
├── install-windows.ps1      # Windows
├── install-wsl2.sh          # WSL2
├── uninstall.sh / .ps1
├── examples/
│   ├── ccie-lab.yaml
│   └── enterprise-gui-lab.yaml
└── src/
    ├── main.rs              # gdu> 프롬프트, 근두운 배너
    ├── topology/mod.rs
    ├── vm/mod.rs            # gdu-<node> QEMU 이름
    ├── network/mod.rs
    └── console/mod.rs       # gdu - <node> 타이틀

설치 & 실행 : 
    # Ubuntu / WSL2
sudo ./install.sh
sudo gdu shell

gdu> load examples/enterprise-gui-lab.yaml
gdu> start
