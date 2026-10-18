# imjang-location-guide

임장(현장 방문)을 앞두고 입지 분석이 막막한 사람을 위한 Claude 스킬입니다.
단지명이나 주소를 주면 반경 1km 입지를 쉬운 말로 분석하고, **현장 체크리스트와 동적지도가 담긴 HTML 보고서**를 만듭니다.

## 이런 분께

- 첫 집·신혼·전월세를 알아보는 분
- 학령기 자녀가 있어 학교·통학로가 중요한 분
- 자녀 독립·은퇴를 앞두고 병원·마트·평지 여부를 보는 분
- 호재 일정의 불확실성까지 따져 보려는 분

나이가 아니라 **상황(프로필)** 기준으로 읽어 줍니다. 말하지 않으면 '일반 실거주'로 봅니다.

## 결과물

1. 채팅 요약: 지하철·마트·시장·공원·초등학교·주변 단지 (거리 + 걸어서 몇 분)
2. HTML 보고서: 한눈에 보기, 내 상황 기준 읽기, 교통, 생활 편의, 교육, 주변 단지, 호재, 지도, 임장 체크리스트
3. 호재마다 신뢰도 등급(🟢 확정 / 🟡 계획 / ⚠️ 보도만·번복 이력)과 출처
4. 지도 저장: 링크가 만료돼도 남는 약식 지도와 마커 데이터(JSON) 저장·재발급 요청문 복사

## 준비물

| 항목 | 필수 여부 | 비고 |
|---|---|---|
| ipzitalk(입지톡) MCP 연결 | 필수 | 주소·시설·단지·지도 데이터 출처. 아래 설명 참고 |
| 웹 검색 | 필수 | 호재 조사에만 사용 |
| 코드 실행·파일 생성 | 권장 | HTML 보고서 파일 생성용 |

### ipzitalk MCP란

카카오가 만든 MCP가 아니라 **시너지랩스(Synergy Labs)가 운영하는 서드파티 베타 서비스**입니다. 카카오 로컬, K-apt, 청약홈, 네이버 지오코딩·지도 데이터를 묶어 도구 10개로 제공합니다. 이 스킬은 입지 요소를 이 서버의 결과로만 채웁니다.

연결은 두 가지 중 하나를 고르세요. 동시에 켜지 마세요.

| 방식 | 필요한 것 | 비고 |
|---|---|---|
| Remote (호스티드) | 카카오 계정 로그인 | 가장 간단. 안내 페이지 기준 무료 플랜 하루 500회 |
| OSS 로컬 | Node.js 18+, API 키 4개 (카카오 REST, 네이버 지도 ID·Secret, data.go.kr) | 직접 운영. 키 발급처 정책을 따름 |

Claude Code에서는 아래 순서로 설치하고, 이어서 나오는 `setup`에서 모드를 고릅니다.

```
/plugin marketplace add chatdaeri/ipzitalk
/plugin install ipzitalk@ipzitalk
/reload-plugins
/ipzitalk:setup
```

Claude 앱에서는 커스텀 커넥터를 추가합니다. 이름은 `ipzitalk`, URL은 `https://ipzi-talk.synergylabs.kr/mcp`이고 로그인 방식으로 인증합니다.

**이용약관:** ipzitalk은 [서비스 이용약관](https://ipji-talk.com/legal/terms)을 따릅니다. 서비스로 얻은 정보의 영리 이용은 사전 서면 동의가 필요하다고 정해져 있으니(제15조 3항), 보고서를 판매·광고 콘텐츠로 쓰려면 서비스 쪽에 먼저 확인하세요. 이 레포의 MIT 라이선스는 이 레포의 파일에만 적용됩니다.

서비스는 베타라 요금·한도·도구 이름이 바뀔 수 있습니다. 데이터 출처, 도구별 한도, 알려진 제약은 [ipzitalk 가이드](docs/IPZITALK.md)에 정리했고, 최신 내용은 [ipzitalk 안내 페이지](https://ipzi-talk.synergylabs.kr)와 [GitHub](https://github.com/chatdaeri/ipzitalk)를 확인하세요.

## 설치

**Claude 앱 (Pro / Max / Team / Enterprise)**

1. `skills/imjang-location-guide` 폴더를 ZIP으로 압축합니다. 폴더가 ZIP 최상위에 있어야 합니다.
2. 설정 > Capabilities에서 코드 실행 및 파일 생성을 켭니다.
3. 같은 화면의 Skills 섹션에서 Upload skill로 ZIP을 올립니다.

**Claude Code**

```bash
# 개인용 (모든 프로젝트에서 사용)
mkdir -p ~/.claude/skills
cp -r skills/imjang-location-guide ~/.claude/skills/
```

팀에 공유하려면 프로젝트의 `.claude/skills/`에 같은 폴더를 두고 커밋합니다.

자세한 설치와 문제 해결은 [사용 매뉴얼](docs/MANUAL.md)을 보세요.

## 사용

```
래미안안양메가트리아 임장 준비해줘
평촌어바인퍼스트 입지 분석, 학령기 자녀 기준으로
/imjang-location-guide 평촌어바인퍼스트
```

## 한계

- 도보·차량 시간은 직선거리 기반 추정치입니다. 실제 길은 더 걸릴 수 있습니다.
- 시세·실거래가·대출 정보는 다루지 않습니다.
- 학교 수준 평가, 치안, 소음 수치는 데이터가 없어 단정하지 않고 현장 체크 항목으로 넘깁니다.
- 정보 제공 목적이며 투자·계약 조언이 아닙니다.

## 라이선스

[MIT](LICENSE)
