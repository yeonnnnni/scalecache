# 현재 상태 — 2026-11-24

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

## 현재 막힌 부분

UTM은 Mac에 설치되어 있고 명령줄 도구 파일도 존재한다.
그러나 이 세션에서는 다음 자동제어 시도가 성공하지 않았다.

- `utmctl list`: 프로세스가 signal 6으로 종료됨
- `utmctl --help`: 종료 코드 134
- UTM 앱 자동제어 연결: 시간 초과

이는 이 세션에서 자동제어가 안 됐다는 관찰이다.
UTM 자체의 고장이나 사용자 설정 오류로 단정할 근거는 없다.
Linux VM 생성·설치·SSH 접속은 아직 하지 못했다.

## 지금 사용자가 할 일

[VM_SETUP.md](VM_SETUP.md)에 적힌 설정으로 VM을 만들고 Ubuntu를 설치한 뒤,
일반 사용자 이름과 VM IP를 알려준다. 비밀번호를 채팅에 보내지는 않는다.
SSH 키 등록 등 인증이 필요하면 로컬에서 입력할 명령으로 안내한다.

VM이 이미 있다면 새로 만들기 전에 현재 VM의 CPU 구조와 설치 상태를 알려준다.

## 다음에 Codex가 진행할 일

SSH 접속이 가능한지 먼저 확인한다. 접속이 확보되면 개발 도구 설치,
저장소 준비, 기준 커널 빌드·부팅, 담당 포팅, 모듈·파일시스템 테스트를 순서대로 진행한다.
부팅·벤치마크 결과는 실제 실행한 뒤 이 폴더에 갱신한다.

## 아직 미실행

- Linux VM 설치 및 SSH 접속
- ARM64 기준 커널 빌드·설치·부팅
- 담당 코드 cherry-pick·수정
- jbd3/pxt4 빌드·로드·제거, mount·umount 및 I/O 테스트
- 제공 벤치마크
- 연구실 서버 실측 및 담당 코드 PR
