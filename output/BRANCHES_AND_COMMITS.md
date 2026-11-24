# 기준 브랜치와 원본 커밋

확인일: 2026-11-24. 원본: `MoonSunMoor/scalecache`, 개인 포크: `yeonnnnni/scalecache`.

## 기준 브랜치 판단

**`sc-7.2.8-integ` @ `d6d55a2b2f8d91252f3f7b6d8a32a79250e33caf`를 사용한다.**

판단 근거:

1. [PR #1](https://github.com/MoonSunMoor/scalecache/pull/1), [PR #2](https://github.com/MoonSunMoor/scalecache/pull/2), [PR #4](https://github.com/MoonSunMoor/scalecache/pull/4)의 병합 대상이 모두 이 브랜치다.
2. 현재 HEAD는 cc-core 1부 PR #4의 실제 병합 커밋이다.
3. [해당 SHA의 Makefile](https://github.com/MoonSunMoor/scalecache/blob/d6d55a2b2f8d91252f3f7b6d8a32a79250e33caf/Makefile)에서 VERSION=7, PATCHLEVEL=2, SUBLEVEL=8을 확인했다.
4. 이번 세션의 초기 조회에서 원본·개인 포크의 기존 14개 브랜치 이름과 HEAD SHA가 모두 일치했다.
5. 따라서 최초 공지의 순정 `v7.2.8`에서 다시 시작하거나, 기본 브랜치인 6.6.8에서 작업할 이유가 없다.

브랜치가 앞으로 갱신되면 작업 시작 전에 다시 fetch하여 병합 내용을 확인한다.

## 병합 내역

| PR | 작업 | 확인 상태 | 병합 SHA |
| --- | --- | --- | --- |
| [#1](https://github.com/MoonSunMoor/scalecache/pull/1) | pxt4-support | 병합됨: 2026-11-12 UTC | `010947a6d2ecc4de1fd7f78031ca3854caaf8b2d` |
| [#2](https://github.com/MoonSunMoor/scalecache/pull/2) | dflush | 병합됨: 2026-11-08 UTC | `2433ae1a98c1d3c98b060bdc034d9a1fb46424a1` |
| [#3](https://github.com/MoonSunMoor/scalecache/pull/3) | 예전 cc-core 포팅 | 병합 없이 닫힘 | 적용 대상으로 사용하지 않음 |
| [#4](https://github.com/MoonSunMoor/scalecache/pull/4) | cc-core 1부와 호출 규약 | 병합됨: 2026-11-16 UTC | `d6d55a2b2f8d91252f3f7b6d8a32a79250e33caf` |

조회된 팀 PR은 위 4개이며 열린 PR은 없었다.
cc-core 2부는 현재 통합 HEAD에 포함되지 않는다.
현재 호출 규약을 지키는 order-0 포팅은 이 기준에서 시작할 수 있지만,
large folio·DAX 활성화까지 끝난 것으로 취급하면 안 된다.

## 담당 원본 커밋 순서

6.6.8의 원본 topic 이름은 `topic/cc-filemap-update`, `topic/cc-filemap-lookup`이다.
두 topic은 `topic/cc-core`의 `df662620ebe34007cfae00cb48b15c1d23a6cb28`에서 갈라진다.

| 순서 | 파트 | 원본 SHA | 변경 목적 |
| --- | --- | --- | --- |
| U1 | update | `d00b40251d16802f15c91e15290130c67d0abfe9` | page-cache 삭제 |
| U2 | update | `6168953cb7373dfa0a017417eae41325e6e015b1` | folio 삽입·교체 |
| L1 | lookup | `6f44dbd37215c7da6549de9d9aeae5dbbc0bb175` | 단일 항목 조회 |
| L2 | lookup | `bd9c1905e6b878726113781d476e74bd677060df` | 일괄 조회 |
| L3 | lookup | `3f4874348c3b5b08472a846dcf5be0395b3257b1` | seek hole/data |

각 topic 안의 순서는 유지한다. update와 lookup의 상호 적용 순서는
실제 diff와 의존성을 보고 결정한다. read-batch·fault-around 등 다른 topic의
커밋을 담당 lookup에 무심코 추가하지 않는다.

## 현재 cc-core 호출 규약

반드시 [기준 SHA의 cc-xarray.rst](https://github.com/MoonSunMoor/scalecache/blob/d6d55a2b2f8d91252f3f7b6d8a32a79250e33caf/Documentation/core-api/cc-xarray.rst)를 기준으로 구현한다.

- 원본 7.2.8 함수와 CC 함수를 일대일 대응시키고 비-CC 경로를 보존한다.
- 현재 실행 대상은 order-0이다. CC mapping의 allocation policy에서 large folio와 DAX를 끈다.
- `CONFIG_TRANSPARENT_HUGEPAGE`나 `CONFIG_XARRAY_MULTI`를 전역으로 끄지 않는다.
- large-folio 처리 코드는 삭제하지 않는다. 현재 실행하지 않는 것과 코드 제거를 구분한다.
- U2의 삽입에는 conflict 검사에서 관찰한 old를 넘기는 `cc_xas_store_expected()`가 필요하다.
- 소유한 folio를 삭제·교체하는 경로는 규약에 맞게 `cc_xas_store()`를 유지한다.
- 정상·오류·조기 반환에서 error, traversal reference, cursor, unused allocation을 각각 정리한다.
- `page_cache_delete_batch()`에는 원본부터 CC dispatch 누락이 알려져 있다.
  삭제·해제 경로를 재현하고 별도 필수 수정 커밋으로 처리한다.
- shadow의 논리 삭제와 노드의 물리 회수는 다르다. safe point 없이 임의 GC나 일반 XArray free를 호출하지 않는다.

## 브랜치 운영

- `docs/porting-session`: 이 세션의 문서와 권한 확인 커밋.
- 담당 포팅 브랜치: 기준 통합 SHA에서 별도로 만든다.
- 브랜치 생성과 GitHub 쓰기는 확인했지만, 아직 담당 코드 수정·빌드·부팅은 하지 않았다.
- 코드 PR에는 담당 변경과 필요한 의존성만 넣고 문서 권한 확인용 커밋을 불필요하게 섞지 않는다.
