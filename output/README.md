# ScaleCache 포팅 문서

이 폴더에서 현재 상태, 작업 기준, 필요한 사용자 조작, 다음 작업을 확인한다.
대상은 이정연 담당 `cc-filemap-update`·`cc-filemap-lookup`의 Linux 6.6.8 → 7.2.8 포팅이다.

## 먼저 볼 문서

| 문서 | 내용 |
| --- | --- |
| [STATUS.md](STATUS.md) | 완료한 일, 현재 막힌 부분, 다음 행동 |
| [VM_SETUP.md](VM_SETUP.md) | M5 Mac에서 Ubuntu VM을 만들고 SSH로 넘기는 절차 |
| [BRANCHES_AND_COMMITS.md](BRANCHES_AND_COMMITS.md) | 기준 브랜치 판단 근거, 병합 내역, 담당 원본 커밋 |
| [TODO.md](TODO.md) | 앞으로 진행할 체크리스트 |

## 저장소와 환경

- 팀 저장소: [MoonSunMoor/scalecache](https://github.com/MoonSunMoor/scalecache)
- 개인 포크: [yeonnnnni/scalecache](https://github.com/yeonnnnni/scalecache)
- 포팅 기준: `sc-7.2.8-integ` @ `d6d55a2b2f8d91252f3f7b6d8a32a79250e33caf`
- 문서 브랜치: `docs/porting-session`
- 사용자 환경: MacBook Pro M5 / RAM 16 GB / UTM 설치·실행 완료
- VM 계획: ARM64 Ubuntu Server 24.04 LTS / 4 vCPU / RAM 6 GiB / 시스템 디스크 128 GiB
- VM의 ARM 호환성은 코드 검토에 따른 초기 선택이며, 빌드·부팅·pxt4 실검증 결과는 아직 없다.

## 기록 원칙

- 성공한 확인과 예정 작업을 분리하고, 미실행한 검증을 통과했다고 적지 않는다.
- 코드와 설정 변경에는 이유, 기준 SHA, 확인 결과를 남긴다.
- 담당 커널 변경은 포팅 브랜치에 커밋하고, 세션 문서는 이 폴더에 유지한다.
- 이 문서들은 개인 포크에도 커밋해 보관한다. 코드 PR에 문서 브랜치를 통째로 병합할 필요는 없다.
- 인증 정보와 기기 일련번호는 저장하지 않는다.

기존 루트의 `ScaleCache_porting_TODO.md`는 최초 계획이며, 진행 상태는 이 폴더의 문서를 우선한다.

최종 업데이트: 2026-11-24.
