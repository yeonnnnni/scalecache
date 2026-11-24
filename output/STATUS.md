# 현재 상태 — 2026-11-25

## 결론

**GitHub 쓰기 권한이 실제 커밋으로 확인되었다. 추가 GitHub 권한 설정은 현재 필요하지 않다.**

- 인증 계정: `yeonnnnni`
- 개인 포크의 `permissions.push=true` 확인
- 문서 브랜치 `docs/porting-session` 생성 성공
- 문서 파일 생성·커밋 성공 후 커밋 내용 재조회 완료
- 첫 커밋: [5e4e690a7e63](https://github.com/yeonnnnni/scalecache/commit/5e4e690a7e6353bca392058d0464f91f08b31a50)
- 커밋 메시지: `docs: initialize ScaleCache porting session records`
- 해당 커밋은 `output/README.md`를 추가한다. 커널 코드는 바꾸지 않았다.

## 작업 기준

**팀의 `sc-7.2.8-integ`에서 이어서 작업한다.**

기준 SHA는 `d6d55a2b2f8d91252f3f7b6d8a32a79250e33caf`이며,
cc-core 1부 PR #4의 병합 커밋이다. Makefile에서 7.2.8을 직접 확인했다.
기본 브랜치 `sc-6.6.8-integ`는 포팅 기준이 아니다.
팀의 최신 브랜치를 사용자가 따로 찾아줄 필요는 없다.

| 선행 작업 | 상태 |
| --- | --- |
| pxt4-support | PR #1 병합 완료 |
| dflush | PR #2 병합 완료 |
| cc-core 1부 | PR #4 병합 완료 |
| cc-core 2부 | 확인한 통합 HEAD에는 미병합 |
| 이정연 담당 원본 | update 2개·lookup 3개, 적용 순서 확인 |

자세한 근거는 [BRANCHES_AND_COMMITS.md](BRANCHES_AND_COMMITS.md)를 본다.

## VM 설치 진행 상황

UTM 직접 조작 연결이 복구되어 설치를 진행했다.
중복된 UTM 앱 식별자 대신 `/Applications/UTM.app`의 전체 경로로 연결했다.
이전 `utmctl` 실패는 설치 진행을 막는 현재 장애가 아니다.

| 항목 | 현재 확인 상태 |
| --- | --- |
| VM | `ScaleCache-ARM64`, ARM64 QEMU 가상화, 4 vCPU, RAM 6 GiB, 디스크 128 GiB |
| Mac | Apple M5, RAM 16 GB, 논리 CPU 10개 확인 |
| 네트워크 | Shared Network, 설치 후에도 `192.168.64.2`, `enp0s1` 확인 |
| Ubuntu 미러 | 연결 테스트 통과 |
| OS 파티션 | LVM 없이 root 126.948 G ext4 + EFI 1.049 G |
| 계정 | `yeon`, 호스트 이름 `scalecache-arm64`; 사용자 비밀번호 설정 후 다음 화면 진입 확인 |
| OpenSSH | 사용자 동의 후 선택, 설치 로그에서 패키지 설치 완료 확인 |
| OS 설치·첫 부팅 | 설치 완료, ISO 분리 후 설치 디스크로 부팅 성공 |
| 로그인 | `yeon` 로그인 성공 |
| 실행 중인 커널 | Ubuntu 기본 `6.8.0-151-generic`, `aarch64` — 목표 7.2.8은 아직 빌드 전 |
| 루트 파일시스템 | 로그인 화면에서 총 124.40 GB, 사용률 5.5% 확인 |
| SSH 실제 접속·sudo | 아직 미검증 |

## 현재 막힌 부분: 세션의 실행 권한

VM 설치와 계정 로그인까지는 완료했다. SSH 접속 확인은 현재 세션의 권한에 막혔다.

- 기본 실행 환경에서 `192.168.64.2:22` 소켓 연결: `PermissionError: [Errno 1] Operation not permitted`.
- 읽기 전용 SSH 포트 확인을 위해 권한 상승을 요청했으나, 자동 승인 정책이 요청 자체를 거부했다.
- 거부 이유: 현재 granular 정책의 `sandbox_approval=false`. 이 세션은 권한 상승을 요청할 수 없다.
- 컴퓨터 조작 도구의 Mac Terminal 앱 접근도 안전상 이유로 명시적으로 차단됐다.
- 이 차단을 우회하는 다른 실행 경로는 시도하지 않는다.

따라서 SSH 서버의 실제 응답·원격 인증·명령 실행 성공 여부는 아직 판단하지 않았다.
현재 차단만으로 VM의 SSH 서버가 고장 났다고 결론 내리지 않는다.

허용된 해결 경로는 이 세션의 VM 네트워크 접근 권한을 사용자가 조정하는 것이다.
사용자가 Mac 터미널에서 직접 아래를 실행하면 VM 측 접속 가능 여부를 별도로 확인할 수 있다.

```bash
ssh yeon@192.168.64.2
```

이 사용자 직접 접속은 Codex 실행 환경의 SSH 접속 성공을 대신하지 않는다.

## 진행 기록과 PR 역할

- 원본 `ScaleCache_porting_TODO.md`에도 완료한 항목을 체크하며 계속 갱신한다.
- 완료 항목 수는 원본 TODO의 최신 체크 표시를 기준으로 한다.
- Codex: 구현·검증·커밋·push, 최종 PR 제목과 본문 Markdown 준비.
- 이정연: 최종 PR 생성·제출.

## 다음에 Codex가 진행할 일

세션의 네트워크 실행 권한을 확보한 뒤 SSH 접속을 확인한다. 접속이 확보되면 개발 도구 설치,
저장소 준비, 기준 커널 빌드·부팅, 담당 포팅, 모듈·파일시스템 테스트를 순서대로 진행한다.
부팅·벤치마크 결과는 실제 실행한 뒤 이 폴더에 갱신한다.

## 아직 미실행

- SSH 서버 active 상태·원격 접속·명령 실행 및 sudo 권한 확인
- ARM64 기준 커널 빌드·설치·부팅
- 담당 코드 cherry-pick·수정
- jbd3/pxt4 빌드·로드·제거, mount·umount 및 I/O 테스트
- 제공 벤치마크
- 연구실 서버 실측, PR 제목·본문 작성 및 사용자의 PR 제출

