# can-i-remove-this

[English](./README.md) | [한국어](./README.ko.md)

> 이 JavaScript 의존성을 번들에서 제거해도 될까요?

`can-i-remove-this`는 브라우저로 전달되는 JavaScript 의존성을 제거 관점에서
검토하는 근거 중심 Agent Skill입니다. 검사 대상 프로젝트의 `package.json`에서
시작하지만, manifest만으로 의존성이 실제 사용되는지, 사용자에게 전달되는지,
비용이 큰지, 네이티브 기능으로 대체 가능한지 판단하지 않습니다.

## 이 Skill이 실행되는 관점

### `package.json`은 지도로 사용합니다

프로덕션 의존성은 조사 후보일 뿐입니다. Skill은 import, re-export, 공통 wrapper,
호출부, workspace와 브라우저/서버 경계를 추적한 뒤 실제 사용자에게 전달되는
의존성인지 판단합니다.

반대로 import를 찾지 못했다고 곧바로 미사용 의존성으로 판단하지도 않습니다.
Generated code, alias, framework convention, runtime loading이 실제 경로를 숨길 수
있습니다.

### 패키지 크기보다 실제 전송량을 봅니다

Tree-shaking 이후의 프로덕션 번들 근거를 우선합니다. Bundlephobia나 registry
크기는 후보를 찾는 데 유용하지만 프로젝트의 실제 build로 확인하기 전까지는
이론적 비용으로만 취급합니다.

일부 호출부를 네이티브 API로 바꿔도 정적 import가 하나라도 남아 패키지가
번들에 포함된다면 패키지 단위의 즉시 절감량은 0입니다.

### 추상적인 Baseline 배지보다 실제 사용자층을 봅니다

Baseline은 현재 상호운용성 근거로 사용하지만 모든 프로젝트에 적용되는 제거
허가로 취급하지 않습니다. 네이티브 기능의 지원 범위를 프로젝트의 Browserslist
또는 제공된 사용자 analytics와 비교합니다.

Baseline Widely는 대체로 강한 근거입니다. Baseline Newly도 최신 브라우저만
사용하는 내부 서비스에서는 안전할 수 있습니다. 하지만 둘 다 중요한 WebView,
downstream browser 또는 보조공학 테스트를 대신하지 않습니다.

### 비슷한 API보다 프로젝트의 실제 사용 방식을 봅니다

두 API의 겉모습이 비슷하다는 이유만으로 교체하지 않습니다. 프로젝트가 실제로
의존하는 interceptor, retry, parsing, timezone, locale data, focus restoration,
upload progress, error semantics와 plugin을 확인합니다.

예를 들어 `fetch`가 Widely available이어도 Axios interceptor가 token refresh와
요청 재실행을 담당한다면 Axios를 바로 제거할 수 없습니다.

### 교체 비용에는 새로 생기는 코드도 포함합니다

Wrapper, polyfill, fallback, 추가 테스트와 다시 구현해야 하는 동작까지 비용에
포함합니다. Temporal polyfill이 제거하려는 날짜 라이브러리보다 크다면 라이브러리
유지가 더 작은 선택일 수 있습니다.

### 점진적 향상은 실제 전송량을 줄여야 합니다

Fallback 라이브러리를 정적으로 import한 상태에서 feature detection만 추가하면
JavaScript 전송량은 줄지 않습니다. `PROGRESSIVE`는 fallback이 필요 없거나 별도
chunk로 지연 로딩되는 경우에만 사용하며 초기 chunk와 fallback 비용을 분리해
보고합니다.

### 알 수 없는 것은 알 수 없다고 보고합니다

번들 데이터가 없다고 0바이트인 것은 아닙니다. 사용자층 정보가 없다고 모든
브라우저에서 안전한 것도 아닙니다. 대체 후보 지도에 있다는 사실도 제거 허가가
아닙니다. 근거가 부족하면 `INVESTIGATE`로 판정하고 어떤 근거가 필요한지
명시합니다.

### 감사는 기본적으로 읽기 전용입니다

Skill은 조사 결과와 필요한 검증을 보고합니다. 사용자가 별도의 구현을 요청하지
않으면 코드를 수정하거나 analyzer를 설치하거나 패키지를 제거하지 않습니다.
Deploy, release, publish script도 실행하지 않습니다.

## 세 가지 판단 질문

모든 후보는 같은 관점으로 평가합니다.

1. 네이티브 대체 기능이 이 프로젝트의 실제 사용자층에 안전한가?
2. Wrapper, polyfill, fallback을 포함해도 실제 JavaScript가 줄어드는가?
3. 네이티브 기능이 프로젝트가 사용하는 동작을 모두 지원하는가?

세 질문에 모두 긍정적인 근거가 있어야 `REMOVE`로 판정합니다.

## 판정 종류

- `REMOVE`: 지금 브라우저 번들에서 의존성을 완전히 제거할 수 있습니다.
- `REPLACE_PARTIALLY`: 일부 호출부는 교체할 수 있지만 의존성은 남습니다.
- `PROGRESSIVE`: 네이티브 기능과 지연 로딩 또는 단순 fallback으로 전송량을
  줄일 수 있습니다.
- `KEEP`: 호환성, 기능 또는 비용 측면에서 유지할 근거가 있습니다.
- `INVESTIGATE`: 안전한 판단에 필요한 근거가 부족합니다.

## 내 `package.json`으로 이 점검을 돌리는 방법

대체 후보 지도에 있는 묶음은 출발점일 뿐입니다. 의존성은 프로젝트마다 다르기
때문에 Skill은 전체 프로덕션 의존성에 동일한 5단계 절차를 반복 적용합니다.

### 1단계. 프로덕션 의존성 나열하기

사용자에게 실제로 전달될 가능성이 있는 의존성부터 확인합니다. npm
프로젝트에서는 다음 명령으로 시작합니다.

```bash
npm ls --omit=dev --depth=0
```

그다음 `package.json`, lockfile과 workspace를 읽고 import, wrapper, 호출부,
브라우저/서버 build 경계를 추적합니다. 프로덕션 목록에 있다는 이유만으로 실제
브라우저에 전달된다고 가정하지 않습니다.

### 2단계. 각각의 비용 재기

[Bundlephobia](https://bundlephobia.com/)에서 배포된 패키지의 minified 및 gzip
크기를 빠르게 확인할 수 있지만 이 값은 이론적 수치로 취급합니다. Tree-shaking과
중복 제거 이후의 실제 비용은 프로젝트의 프로덕션 build와 이미 설치된
`source-map-explorer`, Vite 프로젝트의 `vite-bundle-visualizer` 같은 analyzer를
우선 사용해 측정합니다.

Skill은 고정되지 않은 analyzer를 조용히 내려받아 실행하지 않습니다. 안전하게
사용할 수 있는 기존 측정 방법이 없다면 실제 번들 비용을 unknown으로 보고하고,
측정에 필요한 명령이나 설정을 함께 제시합니다.

### 3단계. 각 대체 수단의 현재 Baseline 상태 확인하기

후보마다 라이브러리를 대체할 플랫폼 기능을 찾고
[webstatus.dev](https://webstatus.dev/), 공식 `web-features` 데이터 또는 해당
기능의 MDN Baseline 배지에서 현재 상태를 확인합니다. Skill에 복사된 상태를
믿지 않고 조회 출처와 날짜를 기록합니다.

### 4단계. 세 가지 질문 돌리기

플랫폼 대체 수단이 있는 라이브러리마다 다음을 묻습니다.

1. Browserslist 또는 제공된 analytics와 비교했을 때 실제 사용자층에 안전한가?
2. Wrapper, polyfill, fallback과 다시 구현할 동작을 포함한 실제 교체 비용은
   얼마인가?
3. 플랫폼 기능이 이 프로젝트에서 라이브러리를 사용하는 방식을 모두 지원하는가?

대부분의 결정은 패키지 이름보다 사용자층의 지원 범위와 실제 사용 방식 조사에서
결정됩니다.

### 5단계. 필요한 곳에서는 점진적 향상 뒤에 두고 교체하기

Widely available 기능도 나머지 두 질문을 통과해야 강한 제거 후보가 됩니다.
Newly available 기능이라면 사용자층이 충분히 최신인지 확인하거나 feature
detection과 fallback을 유지합니다.

실제 JavaScript를 줄이려면 fallback이 필요 없거나 더 단순하거나 별도 chunk로
지연 로딩되어야 합니다. 정적으로 import된 fallback은 지원 브라우저에도 계속
전송되므로 `PROGRESSIVE`로 판정하지 않습니다.

최종 보고서는 각 후보의 판정, 신뢰도, 예상 절감량과 필요한 검증을 기록합니다.
초기 지도에 없는 의존성도 Web Platform 기능과 역할이 겹친다면 조사합니다.

## 사용 방법

에이전트가 지원하는 Skill 디렉터리에 저장소를 복제하거나 복사합니다. Codex를
예로 들면 다음과 같습니다.

```bash
git clone https://github.com/gwagjiug/can-i-remove-this.git ~/.codex/skills/can-i-remove-this
```

별도의 설치나 빌드 단계는 없습니다. 다음과 같이 요청할 수 있습니다.

```text
$can-i-remove-this를 사용해서 이 프로젝트를 검사해줘. package.json에서
시작해서 어떤 브라우저 JavaScript 의존성을 제거할 수 있는지, 대체 기능과
실제 또는 현재 측정 가능한 절감량을 보고해줘.
```

## 저장소 구조

```text
can-i-remove-this/
├── SKILL.md
├── references/
│   ├── decision-rules.md
│   ├── replacements.md
│   └── report-template.md
├── README.md
├── README.ko.md
└── LICENSE
```

- `SKILL.md`: 진입점, 작업 흐름과 반드시 지켜야 할 검사 규칙
- `decision-rules.md`: 근거 우선순위와 최종 판정 기준
- `replacements.md`: 네이티브 대체 후보와 주요 기능 차이
- `report-template.md`: 최종 감사 보고서 형식

## 참고 자료

- [WebDX Baseline](https://web-platform-dx.github.io/)
- [`web-features`](https://github.com/web-platform-dx/web-features)
- [`browserslist-config-baseline`](https://github.com/web-platform-dx/browserslist-config-baseline)
- [`baseline-browser-mapping`](https://github.com/web-platform-dx/baseline-browser-mapping)
- [How Baseline Can Help You Ship Less JavaScript](https://www.smashingmagazine.com/2026/08/how-baseline-can-help-ship-less-javascript/)

## 라이선스

MIT
