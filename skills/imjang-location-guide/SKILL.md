---
name: "imjang-location-guide"
description: "임장 준비자(첫집·실거주·투자)를 위해 단지·주소 입지를 쉬운 말로 분석하고, 현장 체크리스트와 동적지도가 담긴 HTML 보고서를 만든다."
---

# 임장 입지분석 가이드

임장(현장 방문)을 앞두고 입지 분석이 막막한 사람이, 단지 주변을 쉬운 말로 파악하고 현장에서 무엇을 확인할지까지 챙기도록 돕는 범용 스킬.

트리거: '임장 준비', '입지 분석', '[단지명] 어때', '이 동네 살기 어때', '/imjang [단지명 또는 주소]'

산출물은 두 가지이며 순서를 지킨다.
1. 채팅 요약 (먼저)
2. HTML 보고서 (나중) — 동적지도 iframe과 현장 체크리스트 포함

## 0. 시작 규칙
- 입력은 단지명 또는 주소 1~3곳. 4곳 이상이면 3곳씩 나눠 진행하자고 한 줄로 안내한다.
- 목적·상황을 말하지 않았으면 묻지 말고 '실거주·일반 가구'로 가정해 바로 시작한다. 가정은 요약과 보고서 상단에 한 줄로 밝히고, 바꾸고 싶으면 말해 달라고 안내한다.
- 질문은 위치 후보가 여럿이라 단지를 특정할 수 없을 때만, 한 번에 하나만 한다.
- 동네·역 이름만 주어지면(예: '범계역 근처 아파트') 후보 단지를 먼저 찾아 보여주고 고르게 한다.

## 1. 표현 원칙 (20대~60대 누구나 읽을 수 있게)
- 전문용어는 처음 나올 때 괄호로 풀이한다. 예: 초품아(초등학교를 큰길 건너지 않고 갈 수 있는 단지), 역세권(역까지 걸어서 갈 만한 거리).
- 거리는 항상 '미터 + 걸어서 몇 분'을 함께 쓴다.
- '좋다/나쁘다' 단정 대신 '이런 분께는 유리 / 이런 분께는 불리'로 쓴다.
- 나이·성별로 사람을 가정하지 않는다. 아래 생애 단계 프로필을 쓰고, 해당하는 것을 본인이 고르게 한다.

## 2. 상황별 관점 (프로필)
프로필을 지정하지 않으면 '일반 실거주'로 본다. 사용자가 말하면 해당 프로필의 '먼저 볼 것'을 요약·보고서의 '내 상황 기준 읽기'에 반영한다. 여러 개 선택도 가능하다.

| 프로필 | 먼저 볼 것 | 흔한 함정 |
|---|---|---|
| 첫 집·신혼·전월세 | 출퇴근 역까지 거리, 마트·편의점·배달 편의, 밤길 | 지도상 가까워도 실제 걷는 길이 멀거나 가파름 |
| 학령기 자녀 | 초등학교 거리·통학로(큰길·건널목), 공원·놀이터, 학원가 접근 | 학교가 가깝다는 것과 배정된다는 것은 다름 (학구도 확인) |
| 자녀 독립·은퇴 준비 | 병원·약국, 마트·시장 도보, 평지 여부, 조용함, 주차·엘리베이터 | 지도에 없는 경사·계단 |
| 투자 관점 | 호재 일정의 불확실성, 주변 공급, 임차 수요(역·직장), 세대수 | 호재는 일정이 밀리거나 번복될 수 있음 |

투자 관점이어도 가격 전망·수익 보장은 쓰지 않는다.

## 3. 데이터 수집 (입지 요소는 ipzitalk MCP 결과만 사용)
웹서치·추정값으로 입지 요소를 대체하지 않는다. 반경은 1000m, 요소별 상위 3개까지.

### 3-1. 위치 확정
1. 단지 정보(준공연도, 세대수, 동수, 시공사, 난방, 학교 안내, 지하철 도보 안내): 연결된 ipzitalk 서버의 도구 목록에서 get_complex_info_batch(items=[{complex_query: 단지명}], detail=true)를 먼저 찾는다. 없으면 get_complex_info(complex_query=단지명, detail=true)를 쓴다. 기본정보만 필요하면 get_complex_info_by_query(complex_query=단지명)도 된다. 서버마다 도구 이름이 다르므로(베타) 스키마를 확인한 뒤 호출한다. 학교·지하철 안내는 상세(detail) 조회에만 들어 있다.
2. get_address(query=단지명 또는 주소): 위경도. 카테고리가 '부동산 > 주거시설 > 아파트'이고 주소가 일치하는 항목을 center로 쓴다. 상가·어린이집·정문 등은 center로 쓰지 않는다.
3. 단지가 하나로 특정되면 묻지 말고 진행한다. 후보가 여럿이면 AskUserQuestion으로 고르게 한다.

### 3-2. 주변 요소
- search_by_nearby_category(categories=['subway','mart_large','school_elementary'], center_lat, center_lng, radius_m=1000)
- 공원·시장·병원·약국·편의점·주변 단지는 search_by_nearby_keyword(query=..., radius_m=1000, deduplicate_by='name')를 쓴다. 주변 단지는 preset='apartment'. 호출당 결과는 최대 50개로 고정되므로, 거리순 상위 중 업종이 맞는 것만 채택한다.
- 키워드 검색은 노이즈가 많다. 결과의 kakao_category_name으로 해당 업종만 채택한다. 예: 공원 검색에 전기차 충전소·주차장이 섞이면 제외, 시장 검색에 음식점·빵집이 섞이면 제외(상호에 '시장'이 들어간 가게는 시장이 아님).
- 대형마트 카테고리에 슈퍼마켓이 잡히면 '슈퍼마켓'으로 정직하게 표기한다. 대형마트가 없으면 '1km 내 확인되지 않음'으로 쓴다.
- 병원·약국·편의점은 결과를 보고 판단하되 업종이 확실한 것만 채택한다.

### 3-3. 지하철 폴백
반경 1km 안에 지하철역이 없으면 query='지하철역', radius_m=2000으로 다시 조회한다. 반경 밖 최근접역은 '반경 1km 밖 — 참고용'으로 분리 표기하고, 핵심 지표에는 '1km 내 0'으로 쓴다.

### 3-4. 보조 도구 (필요할 때만)
동네만 주어졌을 때의 후보 찾기 find_complexes_near_point, 지역코드 get_region_code, 주변 분양 공고 search_presale_notices_by_filter가 있다. 이 도구들은 ToolSearch로 스키마를 먼저 불러온 뒤 도구 설명대로 쓴다. 파라미터를 추측해서 호출하지 않는다.

### 3-5. 도구 오류 처리
'No approval received' 등으로 거절되면 같은 호출을 다시 시도한다. 끝내 안 되면 그 항목을 '조회 실패'로 명시하고 나머지로 진행한다. 값을 지어내지 않는다.
도구 이름이 없다는 오류가 나면(ipzitalk은 베타라 이름이 바뀔 수 있다) 연결된 서버의 도구 목록에서 같은 기능의 도구를 찾아 쓴다. 이때도 파라미터를 추측하지 않고 스키마를 확인한다.

### 3-6. 시간 추정 규칙
- 도보: 직선거리 ÷ 75m(분속), 반올림. 1km를 넘으면 '도보 15분+'로 쓴다.
- 차량: 직선거리 1km당 대략 3~4분(신호·우회 감안, '대략'이라 표기).
- 직선거리 기준이라 실제 길(경사·신호·우회)은 더 걸릴 수 있음을 보고서에 명시하고, 현장 체크리스트에서 직접 걸어보게 한다.

## 4. 호재 조사 (웹서치, 3단계에서 안 나오는 것만)
- 대상: 신설 지하철·광역철도, 산업단지, 재개발·정비사업, 대형 상업·업무시설.
- 신뢰도 등급을 반드시 붙인다. 🟢 착공·확정 / 🟡 계획·협의 중 / ⚠️ 보도만 있거나 일정이 번복된 이력.
- 개통 예정 연도가 자료마다 다르면 최신 자료를 쓰되 '자료 간 편차·순연 이력'을 병기한다. 확정처럼 쓰지 않는다.
- 단지에서 1km 이상 떨어진 호재는 '광역 접근성 참고'로 구분한다.
- 출처를 문서에 남긴다. 인용은 짧게, 원문 그대로 길게 옮기지 않는다.

## 5. 지도
get_map_embed_url(center, markers, radius_m=1000, title='[단지명] 입지분석 지도')
- 마커 색: 사업지 Red / 지하철 Blue / 마트 Green / 공원 Purple / 초등학교 Orange / 시장 Yellow / 주변 단지 Gray.
- 마커 이름에 거리를 병기한다. 예: '이마트 안양점 (504m)'.
- 결과의 expires_at(약 7일)을 기록하고 보고서에 '○○까지 유효, 만료 시 재발급 필요'를 명시한다.
- 여러 단지 비교 시에는 단지마다 지도 하나씩 만든다.

### 5-1. 지도 저장 (링크가 만료돼도 남게)
지도 링크는 약 7일 뒤 만료되고 서버 쪽 저장이라 사용자가 직접 보관할 수 없다. 그래서 보고서에 지도 데이터를 같이 담는다.
- get_map_embed_url에 넣은 center, radius_m, title, markers(name·lat·lng·color)와 expires_at, 지도 URL을 JSON 한 덩어리로 보고서에 내장한다.
- 아래 블록을 보고서 '입지분석 지도' 섹션의 iframe 바로 뒤에 그대로 넣는다. `__MAP_JSON__`만 위 JSON으로 바꾼다(문자열의 `</`는 `<\/`로 이스케이프). iframe에는 id="map-frame"을 단다.
- 이 블록이 하는 일: 만료와 무관한 약식 지도(SVG, 번호 마커+목록), 마커 데이터 .json 저장 버튼, 재발급 요청문 복사 버튼, 만료일이 지나면 iframe을 숨기고 안내를 띄운다.
- 약식 지도는 직선 위치 관계만 보여 주므로 실제 길과 다르다는 문구를 지운다. 외부 라이브러리는 쓰지 않는다.
- 재발급 요청을 받으면 사용자가 붙여 준 JSON의 center, radius_m, markers, title을 그대로 get_map_embed_url에 넣어 새 링크를 만들고, 새 만료일을 알린다.
- 사용자가 지도 이미지를 달라고 하면, 연결된 서버에 정적 지도 도구(예: get_static_map)가 있을 때만 쓴다. 없으면 약식 지도 SVG를 쓰라고 안내한다.

```html
<section id="map-save">
<h3>지도 저장 (링크가 만료돼도 남습니다)</h3>
<p id="map-expired" class="note" hidden>지도 링크가 만료됐습니다. 아래 약식 지도를 쓰거나, 재발급 요청문을 복사해 다시 만들어 달라고 하세요.</p>
<svg id="map-svg" viewBox="0 0 640 640" role="img" aria-label="약식 입지 지도" style="width:100%;max-width:640px;height:auto;border:1px solid #E3E3E3;border-radius:10px;background:#fff"></svg>
<ol id="map-legend"></ol>
<p class="screen-only"><button type="button" id="btn-json" style="min-height:44px;padding:0 14px;margin-right:8px">마커 데이터 저장(.json)</button><button type="button" id="btn-copy" style="min-height:44px;padding:0 14px">지도 재발급 요청문 복사</button> <span id="map-msg" role="status"></span></p>
<small>약식 지도는 직선 위치 관계만 보여 주며 실제 길과 다릅니다. 번호는 위 목록과 같습니다.</small>
<script type="application/json" id="map-data">__MAP_JSON__</script>
<script>
(function(){
var D=JSON.parse(document.getElementById('map-data').textContent);
var C={Red:'#e03131',Blue:'#1c7ed6',Green:'#2f9e44',Purple:'#7048e8',Orange:'#f76707',Yellow:'#fcc419',Gray:'#868e96'};
var kx=111320*Math.cos(D.center.lat*Math.PI/180),ky=110540;
function xy(m){return [(m.lng-D.center.lng)*kx,(D.center.lat-m.lat)*ky];}
var far=D.radius_m*1.1;
D.markers.forEach(function(m){var p=xy(m);far=Math.max(far,Math.hypot(p[0],p[1])*1.08);});
var s=290/far,NS='http://www.w3.org/2000/svg',svg=document.getElementById('map-svg'),lg=document.getElementById('map-legend');
function el(n,a){var e=document.createElementNS(NS,n);for(var k in a)e.setAttribute(k,a[k]);return e;}
svg.appendChild(el('circle',{cx:320,cy:320,r:D.radius_m*s,fill:'none',stroke:'#2D7D6F','stroke-dasharray':'6 6'}));
var t=el('text',{x:320,y:320-D.radius_m*s-6,'text-anchor':'middle','font-size':14,fill:'#595959'});t.textContent='반경 '+D.radius_m+'m';svg.appendChild(t);
D.markers.forEach(function(m,i){var p=xy(m),x=320+p[0]*s,y=320+p[1]*s;
 svg.appendChild(el('circle',{cx:x,cy:y,r:11,fill:C[m.color]||m.color||'#868e96',stroke:'#fff','stroke-width':2}));
 var n=el('text',{x:x,y:y+5,'text-anchor':'middle','font-size':13,'font-weight':700,fill:(m.color==='Yellow'?'#1F1F1F':'#fff')});n.textContent=i+1;svg.appendChild(n);
 var li=document.createElement('li');li.textContent=m.name;lg.appendChild(li);});
if(D.expires_at&&Date.now()>Date.parse(D.expires_at)){document.getElementById('map-expired').hidden=false;var f=document.getElementById('map-frame');if(f)f.hidden=true;}
var msg=document.getElementById('map-msg');
document.getElementById('btn-json').onclick=function(){var b=new Blob([JSON.stringify(D,null,1)],{type:'application/json'}),a=document.createElement('a');a.href=URL.createObjectURL(b);a.download=D.title+'.json';a.click();};
document.getElementById('btn-copy').onclick=function(){var txt='아래 데이터로 입지톡 지도를 다시 만들어줘. center, radius_m, markers를 그대로 쓰고 title도 같게 해줘.\n'+JSON.stringify(D);
 if(navigator.clipboard&&navigator.clipboard.writeText){navigator.clipboard.writeText(txt).then(function(){msg.textContent='복사됨';},function(){msg.textContent='복사 실패. JSON 저장 버튼을 쓰세요';});}else{msg.textContent='복사 실패. JSON 저장 버튼을 쓰세요';}};
})();
</script>
</section>
```

## 6. 채팅 요약 (HTML보다 먼저)
1. 단지 한 줄 소개와 가정(프로필) 한 줄.
2. 표: 구분 | 내용 | 직선거리 | 도보 추정 (지하철·마트·시장·공원·초등학교·주변 단지).
3. 내 상황 기준 읽기: 유리한 점 / 주의할 점 각 1~3줄 (데이터에 근거한 것만).
4. 호재 (신뢰도 등급 포함) 와 데이터 한계 1~2줄.
5. 마지막 줄: '이어서 HTML 보고서를 만들겠습니다.'

## 7. HTML 보고서
파일명: [단지명]_임장가이드.html (/mnt/user-data/outputs/). 단일 파일, 인라인 CSS, vanilla JS만.

### 접근성·휴대 기준 (모든 연령 대상)
- 본문 17px 이상, 줄간격 1.7, 글자 대비 4.5:1 이상. 작은 글씨(12px 이하)는 출처·주석에만 쓴다.
- 모바일 우선. 터치 영역 44px 이상. 표는 가로 스크롤 없이 세로로 쌓이게 한다.
- 인쇄 CSS(@media print): 현장에 들고 갈 수 있게 체크리스트가 한 장에 들어오도록 하고, 지도 iframe은 숨긴 뒤 '지도 링크' 텍스트를 보여준다.
- 기본 팔레트는 중립(배경 #FFFFFF, 본문 #1F1F1F, 강조 #2D7D6F, 경고 #C25400). 브랜드 색이 있으면 :root 변수만 교체한다.
- 폰트: Pretendard (jsdelivr CDN), fallback으로 -apple-system, 'Apple SD Gothic Neo', sans-serif.

### 구성 (순서 고정)
1. 한눈에 보기: 결론 3줄 + 핵심 수치 4개 (지하철 / 마트 / 시장 또는 공원 / 최근접 초등학교).
2. 내 상황 기준 읽기: 선택한 프로필의 유리·주의 요약.
3. 교통: 서술 + 지하철 리스트 + 주의 블록. 반경 밖 역은 참고용으로 분리.
4. 생활 편의: 마트·시장·병원·약국·편의점·공원 (거리·도보시간).
5. 교육: 초등학교 3곳과 '초품아 여부'. 단지 부지 내 학교가 아니면 '초품아 아님'으로 정직하게 쓴다. 학교 수준·학업 평가는 다루지 않는다(데이터 없음). 배정은 교육청 학구도 확인을 안내한다.
6. 주변 단지: 거리·특징 (세대수 등은 조회한 경우만).
7. 호재: 연도 | 제목 | 설명 | 신뢰도 등급(🟢🟡⚠️) | 출처.
8. 입지분석 지도: iframe(높이 520px, id="map-frame") + 마커 색 범례 + 만료일 주석 + 5-1의 지도 저장 블록.
9. 임장 체크리스트 (아래).
10. 한계와 확인처 / 푸터.

### 임장 체크리스트 (체크박스, 단순 input[type=checkbox])
데이터로 알 수 없는 것을 현장으로 넘기는 섹션이다. 단지 특성에 맞게 항목을 고르고, 데이터에서 나온 주의점(예: 역이 멀다, 학교가 건너편이다)은 해당 항목을 강조 표시한다.
- 시간대 방문: 평일 아침 출근 시간 / 주말 낮 / 평일 밤.
- 걸어보기: 지하철역·마트·초등학교까지 실제로 걸어서 시간 재기, 경사·계단·횡단보도·큰길 여부.
- 소음·환경: 도로·철도·학교 운동장·상가 소음, 밤 조명·분위기, 냄새.
- 단지 상태: 저녁 주차 만차 여부, 분리수거장·쓰레기, 엘리베이터 대기, 외벽·지하주차장 균열·누수 흔적, 일조·조망.
- 질문 목록(중개사·관리사무소): 관리비 수준, 하자·누수 이력, 층간소음, 재건축·정비 계획, 학구 배정, 주차 대수.
- 체감: 택시·배달 호출, 야간 편의점·약국.

### 한계와 확인처 문구
- 도보·차량 시간은 직선거리 기반 추정치이며 실제와 다를 수 있다.
- 이 보고서에는 시세·실거래가·대출 정보가 없다. 국토교통부 실거래가 공개시스템, 교육청 학구도 안내서비스, 해당 지자체 도시계획 공고에서 직접 확인을 권한다.
- 정보 제공 목적이며 투자·계약 조언이 아니다.

## 8. 비교 모드 (2~3곳)
- 보고서 맨 위에 비교표: 준공·세대수 / 지하철 / 마트 / 시장·공원 / 초등학교 / 주변 단지 수.
- 우열 판정을 쓰지 않는다. '○○ 프로필에는 A단지가 유리한 항목' 형태로 쓴다.
- 이후 단지별 섹션을 같은 구성으로 반복하고, 체크리스트는 마지막에 한 번만 둔다.

## 9. 금지·정직 규칙
- 입지 요소는 ipzitalk 결과만 쓴다. 없으면 '확인되지 않음' 또는 '조회 실패'로 쓴다.
- 가격·수익 전망, 학교 우열, 치안·범죄율·소음 수치처럼 데이터에 없는 것은 단정하지 않고 현장 체크 항목으로 넘긴다.
- 반경 밖 항목은 반드시 '참고용'으로 분리한다.
- 호재 일정은 확정처럼 쓰지 않는다. 등급과 편차를 함께 쓴다.
- 특정 단지를 사라/사지 말라고 권유하지 않는다.

## 10. 완료 전 검증
- [ ] 모든 거리·시설명이 조회 결과와 일치하는가
- [ ] 반경 밖 항목에 '참고용'을 달았는가
- [ ] 마트 오분류(슈퍼마켓 등)를 바로잡았는가
- [ ] 초품아 여부를 정직하게 썼는가
- [ ] 호재마다 신뢰도 등급과 출처가 있는가
- [ ] 지도 iframe URL과 만료일이 맞는가
- [ ] 지도 저장 블록의 JSON 마커가 get_map_embed_url에 보낸 값과 같은가
- [ ] 체크리스트가 단지 특성(주의점)을 반영했는가
- [ ] 요약 텍스트가 HTML보다 먼저 나갔는가

파일은 SendUserFile로 전달하고, 마지막에 지도 링크 만료일을 한 줄로 알린다.