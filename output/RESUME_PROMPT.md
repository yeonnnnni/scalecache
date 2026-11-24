# ScaleCache 포팅 실행 세션 인계

## 최신 확인: VM Codex CLI 로그인 상태

- VM에 Linux ARM64용 Codex CLI `0.172.0`을 설치했다.
- 사용자 수동 인증 파일 전송 후 원격 `codex login status`에서 `Logged in using ChatGPT`를 확인했다.
- 이 문서를 전달받은 새 CLI 세션에서 먼저 모델 응답과 현재 정책에서 허용되는 읽기 전용 셸 도구 실행을 확인한다.
- 모델 응답·도구 실행·sudo·clone·개발 패키지 설치·포팅·빌드는 아직 확인하지 않았다.
- `/home/yeon` 아래 프로젝트 작업 폴더를 만들고 그 안에서 소스·빌드·로그를 관리한다. VM을 재설치하지 않는다.
- 공개 코드 읽기와 VM 작업부터 진행한다. 기존 ChatGPT 세션의 GitHub 연결 권한이 자동으로 이전된다고 가정하지 않는다.
- sudo와 push 인증이 필요하면 해당 환경의 정상 승인 절차를 따른다. 인증 정보는 출력·문서·커밋에 넣지 않는다.
- 함께 전달된 `ScaleCache_porting_TODO.md`와 아래 기존 조사 내용을 읽고 작업을 이어간다. PR은 사용자가 직접 생성한다.

확인일: 2026-11-25. 이 문서를 읽고 기존 상태를 유지한 채 작업을 이어간다.
이전 ChatGPT 작업 모드에서는 VM 네트워크 실행이 차단되어 코드 포팅·커널 빌드를 시작하지 못했다.
차단된 정책을 우회하지 말고, 현재 개발 환경의 정상적인 실행·승인 절차를 따른다.

## 목표와 사용자 지시

- 담당자 이정연. 범위는 cc-filemap-update, cc-filemap-lookup.
- Linux 6.6.8 ScaleCache 변경을 이미 병합된 7.2.8 통합 코드 위에 포팅한다.
- 필요한 환경 준비·코드 수정·검증·커밋·push를 진행한다.
- **PR은 사용자가 직접 만든다.** 최종 PR 제목·본문을 Markdown으로 작성한다.
- 완료한 항목만 TODO에 체크한다. 예정·미실행·실패를 성공으로 적지 않는다.
- 사용자에게 필요한 결정·진행 상태·재현 절차를 output/ 문서에 남긴다.
- 기존 VM을 다시 설치하거나 초기화할 필요는 없다.

## 현재 VM

| 항목 | 값 |
| --- | --- |
| VM 이름 | ScaleCache-ARM64 |
| 호스트 | MacBook Pro M5, 논리 CPU 10개, RAM 16 GB |
| VM | UTM QEMU ARM64 하드웨어 가상화, 4 vCPU, RAM 6 GiB, 시스템 디스크 128 GiB |
| OS | Ubuntu Server 24.04.3 LTS ARM64 |
| 실행 커널 | 6.8.0-151-generic aarch64. 목표 7.2.8은 아직 미설치 |
| 계정 | yeon |
| 호스트 이름 | scalecache-arm64 |
| IP | 192.168.64.2, enp0s1, Shared Network |
| 디스크 | LVM 없이 root ext4 약 126.948 G + EFI 1.049 G. MOTD root 124.40 GB |
| SSH | OpenSSH 설치, 사용자 Mac에서 비밀번호 인증 후 셸 진입 확인 |
| 미확인 | sudo 동작, 키 인증, Codex의 실제 원격 명령 실행 |

비밀번호·토큰·개인키는 이 문서에 넣지 않는다. 인증이 필요하면 사용자가 직접 처리한다.
사용자가 SSH에 성공한 사실과 현재 에이전트 실행 환경의 접속 가능 여부를 구분한다.

## 저장소와 작업 기준

- 개인 포크: https://github.com/yeonnnnni/scalecache.git
- 팀 저장소: https://github.com/MoonSunMoor/scalecache.git
- 원본 artifact: https://github.com/syslab-CAU/ScaleCache-6.6.8
- 기준 브랜치: sc-7.2.8-integ
- 확인한 기준 HEAD: d6d55a2b2f8d91252f3f7b6d8a32a79250e33caf
- 기본 브랜치 sc-6.6.8-integ에서 포팅하지 않는다.
- 선행 PR #1(pxt4-support), #2(dflush), #4(cc-core 1부)는 병합됨.
- 이전 PR #3은 병합 없이 닫힘. cc-core 2부는 현재 기준 HEAD에 없음.
- 문서 브랜치 docs/porting-session에는 현재 조사·설치 기록만 있으며, 담당 코드 브랜치와 분리한다.
- VM 안의 저장소 clone·개발 패키지 설치·코드 포팅·빌드는 아직 시작하지 않았다.

## 담당 원본 커밋

| 순서 | 원본 topic | SHA | 역할 |
| --- | --- | --- | --- |
| U1 | topic/cc-filemap-update | d00b40251d16802f15c91e15290130c67d0abfe9 | page-cache 삭제 |
| U2 | topic/cc-filemap-update | 6168953cb7373dfa0a017417eae41325e6e015b1 | folio 삽입·교체 |
| L1 | topic/cc-filemap-lookup | 6f44dbd37215c7da6549de9d9aeae5dbbc0bb175 | 단일 조회 |
| L2 | topic/cc-filemap-lookup | bd9c1905e6b878726113781d476e74bd677060df | 일괄 조회 |
| L3 | topic/cc-filemap-lookup | 3f4874348c3b5b08472a846dcf5be0395b3257b1 | seek hole/data |

각 topic의 순서는 유지하되 상호 적용 순서는 실제 diff·의존성으로 판단한다.
read-batch·fault-around 등 다른 topic의 커밋을 무심코 포함하지 않는다.

## 반드시 읽을 호출 규약

기준 코드의 Documentation/core-api/cc-xarray.rst와 팀 PR #4의 리뷰를 읽는다.

- 현재 실행 범위는 CC mapping의 order-0. large folio와 DAX는 mapping allocation policy에서 제한한다.
- CONFIG_TRANSPARENT_HUGEPAGE·CONFIG_XARRAY_MULTI를 전역으로 끄지 않는다.
- 7.2.8 원래 함수와 일대일 대응시키고 large-folio 코드와 비-CC 경로를 보존한다.
- U2 삽입은 conflict 검사에서 관찰한 old를 cc_xas_store_expected()에 전달한다.
- 소유한 folio 삭제·교체는 계약에 맞게 cc_xas_store()를 사용한다.
- 정상·오류·조기 반환의 error, traversal reference, cursor, unused allocation 정리를 각각 확인한다.
- page_cache_delete_batch()의 원본 CC dispatch 누락은 재현한 뒤 별도 필수 수정 커밋으로 처리한다.
- 안전한 quiescent point 없이 GC나 일반 XArray free로 노드를 회수하지 않는다.

## 다음 실행 순서

1. 현재 실행 위치가 Mac인지 VM인지 확인하고, 허용된 경로에서 VM의 uname, id, 디스크, 메모리, SSH·sudo 상태를 확인한다.
2. 필요한 빌드 패키지와 tmux 등 개발 환경을 준비한다.
3. 팀 통합 HEAD를 얕게 clone하고 담당 원본 topic과 config 참조를 가져온다.
4. ARM64용 기준 커널 설정·빌드·부팅과 복구 경로를 먼저 검증한다.
5. 담당 커밋을 포팅하고 필요한 별도 수정 및 의미 있는 검증을 수행한다.
6. 커널·외부 모듈·파일시스템·제공 워크로드를 통합 검증한다.
7. 최종 변경을 커밋·push하고, 검증 근거와 제한을 담은 PR 제목·본문을 작성한다.

## 초기 빌드 준비 참고

- Ubuntu GCC 13은 확인한 커널 GCC 최소 8.1 요건을 충족한다. GCC 9.4로 낮출 필요는 없다.
- sc-7.2.8-config는 독립 config-only 브랜치이며 제공 설정은 x86_64용이다. ARM에 그대로 복사하거나 코드에 병합하지 않는다.
- ARM64 virtconfig를 시작점으로 EFI·VirtIO 디스크/네트워크·콘솔·EXT4·MODULES·MODULE_UNLOAD·MEMCG·CPUSETS·CPUSETS_V1 등을 검토한다.
- 초기 VM 빌드는 디버그 정보·BTF를 끄는 구성을 고려해 공간과 pahole 의존성을 줄인다.
- cc_xarray.o는 lib/Makefile의 obj-y에 있다. 존재하지 않는 CONFIG_SCALECACHE 옵션을 가정하지 않는다.
- 도구 검증 예: tools/testing/radix-tree의 cc-shadow-reclaim. 커널·pxt4 실행 검증을 대신하지 않는다.
- 팀 통합 커널에는 fs/pxt4와 fs/jbd3 전체 소스가 없다. PR #1은 지원 exports·headers다. 외부 모듈은 원본 artifact의 pxt4/에서 별도로 확보하고 7.2.8 호환성을 확인한다.
- 원본 artifact에서 필요한 pxt4/·workloads/만 sparse checkout하면 두 번째 커널 전체 다운로드를 줄일 수 있다.
- Filebench 원본 fileserver 초기 데이터는 약 384 GiB. VM 128 GiB 디스크로 원본 실행을 완료했다고 하지 않는다.
- ARM VM의 기능 확인과 x86 연구실 서버의 실제 성능 검증을 구분한다.

## 문서 유지

- 원본 종합 체크리스트: ScaleCache_porting_TODO.md. 현재 18/103개 완료.
- Mac의 기존 문서 폴더: /Users/leejeongyeon/Documents/Codex/2026-11-24/new-chat-5
- VM 원격 세션에서는 위 Mac 경로가 로컬 경로라고 가정하지 않는다.
- 이 문서와 STATUS.md, BRANCHES_AND_COMMITS.md, TODO.md를 읽어 현재 상태를 확인한다.
- 새 실행 환경에서 직접 접근하지 못하는 기존 문서는 사용자에게 전달하거나 동기화 방법을 확인한다.
- 실제 실행 결과와 변경 SHA를 기록해 원래 ChatGPT 문서 세션에서도 진행 상태를 갱신할 수 있게 한다.

