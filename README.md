# ig-bridge

인스타그램 DM 등 **인앱 브라우저(웹뷰)** 에서 열린 링크를, 외부 브라우저로 내보내거나 목적지로 랜딩시키는 브릿지 페이지.

- 배포: GitHub Pages (`gh-bridge/index.html` → 리포지토리 루트의 `index.html` 로 서빙)
- 버전: v0.8 (Android referer 세탁 — 크롬에서 브릿지 경유 후 no-referrer 목적지 이동)

## 사용법

```
https://<pages-domain>/?url=<목적지 URL>
```

- `?url=` : 목적지 링크. 생략 시 기본값(쿠팡 상품)으로 동작.
- 목적지 URL 에 `&` 가 포함되면 `encodeURIComponent` 로 인코딩해서 전달.

예시
```
https://<pages-domain>/?url=https://www.coupang.com/vp/products/6731245116
https://<pages-domain>/?url=https://example.com
```

## 동작 (v0.2 — 자동 이동, 버튼 없음)

페이지 진입 시 "잠시 후 이동합니다…" 안내 후 **자동으로** 아래를 수행한다.

| 플랫폼 | 동작 |
|---|---|
| **Android** | 인스타 인앱이면 `intent://...;package=com.android.chrome;` 로 **크롬에서 "브릿지 페이지 자신"을 다시 염**. 크롬에서 재로드되면 `inApp=false` 가 되어 `no-referrer` 로 목적지 이동 → 인스타(`android-app://...`) referer 가 제거된 깨끗한 요청(=크롬 주소창 직접 입력과 동일)이라 목적지 봇 차단을 통과. 크롬 미설치/실패 시 1.5초 폴백으로 인앱 랜딩 |
| **iOS** | `googlechromes://` 스킴으로 **크롬** 열기 시도. 크롬 설치 시 크롬 실행(+목적지 앱 딥링크는 디바이스가 처리). 미설치/실패 시 1.5초 폴백 타이머가 **인스타 인앱 브라우저 안에서 목적지로 랜딩** |
| **PC/기타** | 곧바로 목적지로 이동 |

### 폴백
- (iOS) 외부 앱 전환 실패/스킴 차단 대비, **1.5초 타이머** 후 목적지로 인앱 랜딩(`location.replace`).
- 외부로 **실제 페이지 이탈**(`pagehide`)이 일어난 경우에만 타이머 취소(중복 이동 방지). `visibilitychange` 는 확인창/앱전환만으로도 발동해 무한 스피너를 유발하므로 사용하지 않는다.

## 제약 / 주의 (검증 필요 항목)

- **iOS 는 Safari(기본 브라우저) 강제 실행 공식 API 가 없음.** 그래서 iOS 는 크롬 타겟으로만 탈출 시도하며, 크롬 미설치 시 인앱 랜딩으로 폴백한다.
- **무제스처(자동) 스킴/intent 호출**은 인앱 브라우저에서 차단될 수 있어, 실기기(iOS/Android, 크롬·앱 설치/미설치)별 검증이 필요하다. (추정)
- Android 의 "앱 선택 얼럿"은 기본 브라우저가 이미 지정돼 있으면 뜨지 않고 바로 열린다.
- `?url=` 값이 `http(s)://` 로 시작하지 않으면(예: 플레이스홀더를 그대로 입력) 멈추지 않고 기본값으로 이동한다.
- 목적지 서버(예: 쿠팡)가 봇/비정상 접근을 차단하면 "사용 권한이 없습니다(Ref:...)" 가 뜰 수 있다. 원인은 인스타에서 넘어온 `android-app://com.instagram.android` referer 가 **새로 열린 크롬 창까지 따라가** 차단되는 것. `<meta referrer>` 만으로는 "앱→크롬 최초 로드"의 referer 를 못 지우므로, Android 는 **크롬에서 브릿지를 한 번 경유(referer 세탁)** 한 뒤 `no-referrer` 로 목적지 이동한다. (iOS 스킴 유입은 원래 referer 가 안 붙어 직접 이동)
