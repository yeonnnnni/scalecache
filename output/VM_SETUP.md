# M5 Mac: Ubuntu VM 설치 후 SSH로 넘기기

2026-11-25: UTM 직접 조작으로 VM 생성·Ubuntu 설치·ISO 분리·첫 부팅·yeon 로그인을 완료했다.
아래는 재설치하거나 직접 조작할 때 참고할 절차이며, 현재 화면은 STATUS.md에 기록한다.

## 선택한 초기 구성

| 항목 | 설정 |
| --- | --- |
| 방식 | UTM ARM64 하드웨어 가상화 |
| 생성 경로 | + → Virtualize → Linux |
| 백엔드 | QEMU + UEFI 부팅. Use Apple Virtualization 옵션이 보이면 해제 |
| CPU | 4 vCPU |
| 메모리 | 6144 MiB, 즉 6 GiB |
| 시스템 디스크 | 128 GiB |
| 네트워크 | Shared Network / macOS Shared Network |
| 설치 이미지 | Ubuntu Server 24.04 LTS, 일반 ARM64 ISO |
| VM 이름 예시 | ScaleCache-ARM64 |

이 구성은 M5·RAM 16 GB에서 개발을 시작하기 위한 선택이다.
확인한 cc-core는 컴파일러 원자 연산을 사용하며 직접 x86 명령어를 강제하지 않아
우선 ARM64를 선택했다. 전체 pxt4 경로의 ARM 동작까지 검증한 결과는 아니다.
x86 연구실 서버에서의 최종 재검증은 별도로 진행한다.

## 1. 설치 ISO 받기

[Ubuntu 공식 ARM 이미지 목록](https://cdimage.ubuntu.com/releases/24.04/release/)에서
이름이 `live-server-arm64.iso`로 끝나는 24.04 LTS 이미지를 받는다.

- 일반 `arm64` 서버 설치 ISO를 선택한다.
- `arm64+largemem`, `amd64`, Raspberry Pi용 `raspi` 이미지는 이 구성에 사용하지 않는다.
- 다운로드한 ISO는 UTM에서 설치 디스크로 선택한다.

## 2. UTM에서 VM 만들기

1. UTM을 열고 **+ → Virtualize → Linux**를 선택한다.
2. ISO에서 부팅하는 구성을 사용한다. **Use Apple Virtualization** 옵션이 보이면 해제한다.
3. 위 표대로 CPU 4개, 메모리 6144 MiB, 디스크 128 GiB를 설정한다.
4. 공유 폴더는 우선 생략한다.
5. 저장 후 네트워크가 **Shared Network**인지 확인하고 VM을 실행한다.

목적은 수정 커널을 나중에 VM의 부트로더에서 선택해 부팅하는 것이다.
초기에는 순정 Ubuntu 커널로 설치하면 된다.

## 3. Ubuntu 설치하기

1. 언어·키보드는 기본값 또는 편한 설정을 선택한다.
2. 네트워크는 자동 DHCP를 사용한다. 프록시는 사용하지 않으면 비워둔다.
3. 저장 장치에는 **VM 안에 생성한 128 GiB 가상 디스크**를 선택한다. LVM을 사용하면 기본값이 디스크 일부만 루트 파일시스템에 배정할 수 있다. `/` 할당 크기를 확인하고, 부족하면 SSH 연결 후 남은 공간을 확장한다.
4. 일반 사용자 계정을 만든다. 사용자 이름 예시는 `scuser`다.
5. SSH 설정에서 **Install OpenSSH server**를 선택한다.
6. 인증을 위해 필요한 경우 초기 설치에서는 SSH 비밀번호 로그인을 허용하고, 이후 키 인증을 설정한다.
7. 추가 서버 패키지는 우선 선택하지 않는다.
8. 설치가 끝나면 재부팅한다. 설치 화면으로 다시 돌아오면 UTM의 ISO를 꺼낸 뒤 부팅한다.

설치 화면에 일반 터미널만 나오는 것은 Ubuntu Server의 정상 동작이다.

## 4. VM 안에서 상태 확인

설치한 계정으로 로그인한 뒤 실행한다.

```bash
whoami
hostname -I
uname -m
```

`uname -m` 결과는 `aarch64`여야 한다.
`hostname -I`가 보여주는 VM IP와 사용자 이름을 알려주면 된다.

설치 중 OpenSSH를 선택하지 않았다면 VM 안에서 실행한다.

```bash
sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
```

## 5. Mac에서 접속 확인

Mac 터미널에서 실제 사용자 이름과 IP로 바꾸어 실행한다.

```bash
ssh scuser@VM_IP
```

첫 접속의 호스트 확인은 로컬 VM에 접속하는지 확인하고 처리한다.
비밀번호가 필요하면 Mac 터미널에서 직접 입력한다.

성공하면 이 세션에 다음만 알려준다.

```text
VM 설치 완료
사용자 이름:
VM IP:
Mac 터미널에서 SSH 접속: 성공 / 오류 내용
```

이후 Codex의 SSH 실행 환경에서도 접속 가능한지 확인하고, 필요한 공개키 등록을 안내한다.
사용자의 Mac 터미널에서 접속된 사실만으로 Codex의 접속도 성공했다고 표시하지 않는다.

## 이후 Codex가 맡을 일

- 개발 패키지와 세션 유지 도구 설치
- 저장소와 작업 브랜치 준비
- ARM64 기본 설정에 필요한 프로젝트 옵션을 반영한 커널 설정 준비
- 기준 커널 빌드·설치·부팅 및 복구 방법 준비
- 담당 5개 원본 커밋 포팅과 필수 추가 수정
- 전용 가상 테스트 디스크 또는 loopback 이미지 준비
- jbd3/pxt4와 I/O·벤치마크 검증
- 실행 결과와 남은 일 문서화 및 커밋

## 저장 공간과 벤치마크

128 GiB 시스템 디스크는 초기 개발용이다.
기존 README의 Filebench fileserver 설정은 초기 데이터만 약 384 GiB를 사용한다.
현재 Mac 여유 공간을 고려하면 OS·소스·빌드 결과까지 포함한 원본 설정의 실행 공간을
확보했다고 볼 수 없다. 초기 기능 테스트는 축소하고 변경값을 기록한다.
원본 크기 실행이 필요하면 외부 저장 공간 등 추가 용량을 확보해야 하며,
축소 실행만으로 공지의 전체 벤치마크 완료를 선언하지 않는다.

## 참고

- [UTM Ubuntu 설치 안내](https://docs.getutm.app/guides/ubuntu/)
- [UTM 네트워크 모드](https://docs.getutm.app/settings-qemu/devices/network/network/)
- [Ubuntu ARM 이미지 목록](https://cdimage.ubuntu.com/releases/24.04/release/)
- [Ubuntu OpenSSH 설치 안내](https://documentation.ubuntu.com/server/how-to/security/openssh-server/)

현재 VM 구성·DHCP·미러 테스트·파티션·계정 설정과 설치 완료·첫 부팅·로그인은 화면에서 확인했다. SSH 접속은 세션 네트워크 권한 차단으로 아직 미검증이다.

