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
| 네트워크 | Shared Network, 설치 중 DHCP `192.168.64.2`, `enp0s1` |
| Ubuntu 미러 | 연결 테스트 통과 |
| OS 파티션 | LVM 없이 root 126.948 G ext4 + EFI 1.049 G |
| 계정 | `yeon`, 호스트 이름 `scalecache-arm64`; 사용자 비밀번호 설정 후 다음 화면 진입 확인 |
| 현재 화면 | SSH configuration: OpenSSH 서버 설치 선택 직전 |
| 설치 완료·첫 부팅·SSH | 아직 미확인 |

## 지금 필요한 확인

SSH 서버 설치를 선택하면 VM에 원격 로그인 경로가 생긴다.
컴퓨터 조작 도구의 액션 시점 확인 규칙에 따라, 현재 이 선택에 대한 사용자 확인을 기다리고 있다.
설치·접속이 실제로 끝나기 전에는 원본 TODO의 해당 항목을 체크하지 않는다.

## 진행 기록과 PR 역할

- 원본 `ScaleCache_porting_TODO.md`에도 완료한 항목을 체크하며 계속 갱신한다.
- 현재 원본 TODO에서 준비·조사 항목 14개를 완료로 표시했다.
- Codex: 구현·검증·커밋·push, 최종 PR 제목과 본문 Markdown 준비.
- 이정연: 최종 PR 생성·제출.

## 다음에 Codex가 진행할 일

SSH 접속이 가능한지 먼저 확인한다. 접속이 확보되면 개발 도구 설치,
저장소 준비, 기준 커널 빌드·부팅, 담당 포팅, 모듈·파일시스템 테스트를 순서대로 진행한다.
부팅·벤치마크 결과는 실제 실행한 뒤 이 폴더에 갱신한다.

## 아직 미실행

- Linux VM 설치 완료 확인·첫 부팅 및 SSH 접속
- ARM64 기준 커널 빌드·설치·부팅
- 담당 코드 cherry-pick·수정
- jbd3/pxt4 빌드·로드·제거, mount·umount 및 I/O 테스트
- 제공 벤치마크
- 연구실 서버 실측, PR 제목·본문 작성 및 사용자의 PR 제출

