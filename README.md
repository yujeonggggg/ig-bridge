# ig-bridge

인스타그램 DM 등 **인앱 브라우저(웹뷰)** 에서 열린 링크를, 외부 브라우저로 내보내거나 목적지로 랜딩시키는 브릿지 페이지.

- 배포: GitHub Pages (`gh-bridge/index.html` → 리포지토리 루트의 `index.html` 로 서빙)
- 버전: v0.2 (자동 이동 방식)

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
| **Android** | `intent://` (package 미지정) 로 **기본 브라우저** 열기 시도. App Links 검증된 앱이 있으면 디바이스가 자동으로 앱 실행. 미처리 시 intent 내부 `browser_fallback_url` 로 인앱 랜딩 |
| **iOS** | `googlechromes://` 스킴으로 **크롬** 열기 시도. 크롬 설치 시 크롬 실행(+목적지 앱 딥링크는 디바이스가 처리). 미설치 시 폴백 타이머가 **인스타 인앱 브라우저 안에서 목적지로 랜딩** |
| **PC/기타** | 곧바로 목적지로 이동 |

### 폴백
- 외부 앱 전환 실패/스킴 차단 대비, **2.2초 타이머** 후 목적지로 인앱 랜딩(`location.replace`).
- 외부 앱이 떠서 페이지가 백그라운드로 가면(`visibilitychange`/`pagehide`) 타이머를 취소.

## 제약 / 주의 (검증 필요 항목)

- **iOS 는 Safari(기본 브라우저) 강제 실행 공식 API 가 없음.** 그래서 iOS 는 크롬 타겟으로만 탈출 시도하며, 크롬 미설치 시 인앱 랜딩으로 폴백한다.
- **무제스처(자동) 스킴/intent 호출**은 인앱 브라우저에서 차단될 수 있어, 실기기(iOS/Android, 크롬·앱 설치/미설치)별 검증이 필요하다. (추정)
- Android 의 "앱 선택 얼럿"은 기본 브라우저가 이미 지정돼 있으면 뜨지 않고 바로 열린다.
