# ipzitalk(입지톡) MCP 가이드

이 스킬은 입지 데이터를 **ipzitalk(입지톡) MCP**에서만 가져옵니다. 이 문서는 ipzitalk이 무엇인지, 어떻게 연결하는지, 이 스킬이 어떤 도구를 어떻게 쓰는지 정리한 참고서입니다.

확인 시점: 2026-10-18. 출처는 [ipzitalk 안내 페이지](https://ipzi-talk.synergylabs.kr), [GitHub chatdaeri/ipzitalk](https://github.com/chatdaeri/ipzitalk), 그리고 연결된 서버가 노출한 도구 설명입니다. 베타 서비스라 바뀔 수 있으니, 연결 전에 위 두 페이지를 한 번 확인하세요.

## 1. ipzitalk은 무엇인가

- **카카오 MCP가 아닙니다.** 시너지랩스(Synergy Labs)가 운영하는 서드파티 베타 서비스입니다. 카카오 데이터를 쓰지만 카카오가 제공하는 MCP가 아닙니다.
- 부동산 입지분석, 단지 정보, 청약 공고 조회용 커넥터이며 도구 10개를 제공합니다.
- 서버 주소: `https://ipzi-talk.synergylabs.kr/mcp`

### 데이터 출처

서버가 도구 설명에서 밝힌 출처 기준입니다.

| 데이터 | 출처 | 쓰는 도구 |
|---|---|---|
| 장소명·단지명 → 주소·좌표 | 카카오 로컬 | `get_address` |
| 도로명·지번 주소 → 좌표 | 네이버 지오코딩 (실패 시 VWorld 보조), 법정동 코드는 카카오 | `get_geocode` |
| 주변 시설(지하철·마트·학교 등) | 카카오 로컬 카테고리 검색 | `search_by_nearby_category` |
| 주변 키워드 검색(공원·시장·아파트 등) | 카카오 로컬 키워드 검색 | `search_by_nearby_keyword` |
| 공동주택 단지 정보 | K-apt(공동주택관리정보시스템) | `get_complex_info_*`, `find_complexes_near_point` |
| 분양·청약 공고 | 청약홈(ApplyHome) | `search_presale_notices_by_filter`, `get_region_code` |
| 공유용 지도 | 네이버 지도 기반 자체 지도 페이지 | `get_map_embed_url` |

## 2. 연결 방법

두 가지 방식이 있습니다. **동시에 켜지 마세요.** (공식 안내)

### A. Remote (호스티드, 가장 간단)

카카오 계정으로 로그인하는 방식입니다. API 키를 따로 만들 필요가 없습니다.

**Claude 앱 (웹·데스크톱)**
1. 커스텀 커넥터를 추가합니다.
2. 이름은 `ipzitalk`, URL은 `https://ipzi-talk.synergylabs.kr/mcp`로 입력합니다.
3. 인증은 로그인 방식을 선택하고, 팝업에서 가입·인증을 마칩니다.

**Claude Code** (공식 플러그인 마켓플레이스, 순서대로)

```
/plugin marketplace add chatdaeri/ipzitalk
/plugin install ipzitalk@ipzitalk
/reload-plugins
/ipzitalk:setup
```

설치 후 Claude를 완전히 종료했다가 다시 실행하라고 안내되어 있습니다. `setup`에서 Remote 모드를 고릅니다.

**요금·한도** (안내 페이지 기준)
- 무료 플랜: 모든 도구 사용 가능, 하루 500회 호출. Pro 플랜은 출시 예정입니다.
- GitHub 페이지에는 한도가 적혀 있지 않습니다. 실제 한도와 약관은 서비스 쪽 안내가 우선합니다.
- Remote 백엔드와 호스티드 서비스는 오픈소스가 아니며 시너지랩스가 권리를 가집니다. 이 레포의 MIT 라이선스는 이 레포의 파일에만 적용됩니다.

### B. OSS 로컬 (직접 운영)

호스티드 서비스에 의존하지 않고 본인 컴퓨터에서 오픈소스 `presale-mcp`(도구 10개)를 돌리는 방식입니다. `setup`에서 OSS 모드를 고릅니다.

- 필요한 것: Node.js 18 이상, API 키 4개
  - 카카오 REST API 키
  - 네이버 지도 Client ID와 Secret
  - 공공데이터포털(data.go.kr) 서비스 키
- 키는 채팅창·명령어·설정 파일에 붙여 넣지 마세요. Claude Code의 보안 저장소(userConfig)에 저장하도록 안내되어 있습니다.
- 호출 한도와 비용은 각 키 발급처의 정책을 따릅니다.

### 참고: ipzitalk 플러그인의 자체 스킬

ipzitalk 플러그인에는 자체 스킬이 함께 설치될 수 있고, 입지 리포트 기능이 이 스킬과 겹칠 수 있습니다. 이 스킬은 쉬운 말 풀이, 상황별 관점(프로필), 현장 체크리스트에 초점을 둔 별개의 스킬입니다.

## 3. 이 스킬이 쓰는 도구

| 도구 | 하는 일 | 한도·주의 |
|---|---|---|
| `get_address` | 장소명·단지명을 카카오 주소 후보와 좌표로 변환 | 후보 기본 5개, 최대 15개. 단지 본체를 고를 때 카테고리와 주소를 확인 |
| `get_geocode` | 도로명·지번 주소를 좌표로 변환 | 역·학교 같은 장소명은 `get_address`를 사용 |
| `get_complex_info_by_query` | 단지명·주소 하나로 K-apt 기본정보 조회 | 모호하거나 미등록이면 추측 없이 반환. 상세정보 옵션 없음 |
| `get_complex_info_batch` | 여러 단지의 K-apt 정보를 한 번에 조회 | 최대 20개. `detail=true`는 주차·엘리베이터·CCTV·교통·편의시설 추가, 이때 최대 10개 |
| `find_complexes_near_point` | 좌표·주소 주변 K-apt 단지 목록 | 페이지당 50개, 반경 기본 3km·최대 20km, 거리순 또는 최근 입주순 |
| `search_by_nearby_category` | 지하철·마트·학교 등 카테고리별 주변 시설 | 반경 기본 2,000m·최대 20,000m, 소스 상한 45건 |
| `search_by_nearby_keyword` | 키워드로 주변 장소 검색 (공원·시장·아파트 등) | 호출당 50개, `offset`으로 이어 조회. 프리셋은 `apartment`·`officetel`·`library` |
| `get_map_embed_url` | 마커·중심·반경을 담은 공유용 지도 URL 생성 | 마커 최소 1개, 색은 `Red/Blue/Orange/Yellow/Green/Purple/Gray` 또는 `#RRGGBB` |
| `get_region_code` | 주소를 법정동·시군구·청약홈 지역코드로 변환 | 청약 공고 검색용. 이 스킬의 기본 흐름에서는 쓰지 않음 |
| `search_presale_notices_by_filter` | 청약홈 분양 공고 조회 | 이 스킬의 기본 흐름에서는 쓰지 않음 |

### 스킬 본문의 옛 도구 이름

스킬 본문(`SKILL.md`)에는 `get_complex_info(complex_query, detail=true)`라는 이름이 남아 있습니다. 현재 서버에는 이 이름이 없고 다음 둘로 나뉘어 있습니다. Claude가 도구 목록을 보고 맞는 쪽을 고르면 되지만, 직접 설정한다면 아래 대응을 따르세요.

| 스킬 본문 표기 | 현재 도구 |
|---|---|
| `get_complex_info(complex_query)` | `get_complex_info_by_query(complex_query)` |
| `get_complex_info(complex_query, detail=true)` | `get_complex_info_batch(items=[{complex_query}], detail=true)` |

## 4. 알려진 제약과 주의

- **카테고리 검색은 소스 상한이 45건입니다.** 결과가 많은 지역은 반경을 줄이세요.
- **키워드 검색은 노이즈가 많습니다.** 공원 검색에 전기차 충전소가, 시장 검색에 상호에 '시장'이 들어간 음식점이 섞입니다. 결과의 `kakao_category_name`으로 걸러야 합니다. 이 스킬은 이 규칙을 따릅니다.
- **'대형마트' 카테고리에 슈퍼마켓이 잡힐 수 있습니다.** 이 스킬은 실제 업종대로 표기합니다.
- **지도 링크는 약 7일 뒤 만료됩니다.** 응답의 `expires_at`으로 확인하며, 만료되면 다시 만들어야 합니다.
- **지도에 보낸 마커·좌표는 ipzitalk 서버에 저장되고, 링크를 아는 사람은 열어 볼 수 있습니다.** 자택 같은 민감한 위치는 넣지 마세요.
- **베타라서 도구 이름과 파라미터가 바뀔 수 있습니다.** 실제로 `get_complex_info`가 위 두 도구로 나뉜 것이 확인됐습니다. 이상하면 연결된 서버의 도구 목록을 먼저 보세요.
- **직선거리 기반입니다.** 길찾기 기능이 없어서 도보 시간은 추정치입니다.

## 5. 카카오 공식 MCP와의 차이

카카오가 제공하는 MCP(PlayMCP의 KakaoMap)도 있지만 기능이 다릅니다. 확인한 범위에서는 장소 키워드 검색과 도보·자전거·대중교통 길찾기를 제공합니다. 반경·카테고리 검색, K-apt 단지 정보, 공유 지도 링크가 없어 이 스킬의 대체재는 되지 못합니다. 다만 도보 길찾기는 직선거리 추정을 보완하는 용도로 생각해 볼 수 있습니다. 현재 스킬에는 반영되어 있지 않습니다.
