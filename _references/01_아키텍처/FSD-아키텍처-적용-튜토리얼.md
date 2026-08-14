# FSD 아키텍처 적용 튜토리얼

- 참고문서 : https://fsd.how/kr/docs/get-started/tutorial/
- 개념 정리는 [FSD-아키텍처.md](./FSD-아키텍처.md)를 먼저 읽으세요. 이 문서는 개념을 **실제 폴더/파일로 옮기는 순서**만 다룹니다.
- 예시는 이 저장소의 `projects/` (Next.js App Router + SASS + Zustand + React Query, "미진이네 파스타 가게") 기준입니다.

---

## 0. 큰 그림

FSD 적용은 코드부터 짜는 게 아니라 **설계 → 스캐폴딩 → 구현 → 정리** 순서로 진행합니다.

```
1단계  설계      어떤 페이지가 있는가 → 페이지를 pages 슬라이스로 적는다
2단계  스캐폴딩   레이어 폴더를 만든다 (Next.js 충돌 처리 포함)
3단계  구현      pages부터 만들고, 중복이 생기면 아래 레이어로 내린다
4단계  정리      public API(index.ts)와 import 방향을 점검한다
```

> [!IMPORTANT]
> FSD의 핵심 규칙은 단 하나입니다. **슬라이스의 모듈은 자신보다 엄격히 아래에 있는 레이어의 슬라이스만 import할 수 있다.**
> 같은 레이어의 옆 슬라이스도 import 금지입니다. 이 규칙만 지키면 나머지는 대부분 취향의 문제입니다.

---

## 1단계 · 설계 : 페이지를 먼저 적는다

FSD 설계는 "이 앱에 어떤 화면이 있는가"를 나열하는 것에서 시작합니다. 화면 하나가 `pages` 레이어의 슬라이스 하나가 됩니다.

미진이네 파스타 가게의 경우:

| 화면        | 슬라이스              | 라우트     |
| ----------- | --------------------- | ---------- |
| 인트로/메뉴 | `pages/intro`         | `/`        |
| 면 고르기   | `pages/noodle-select` | `/noodle`  |
| 소스 고르기 | `pages/sauce-select`  | `/sauce`   |
| 결과        | `pages/result`        | `/result`  |

이 시점에서는 폴더만 정하고 내부 구조는 고민하지 않습니다. **슬라이스는 비즈니스 도메인 단위로 자르며, 이름에는 제한이 없습니다.**

### 어디까지 미리 설계할까

- `entities`, `features`, `widgets`는 **미리 만들지 않습니다.** 처음엔 전부 `pages` 안에 두고, 두 페이지 이상에서 같은 코드가 필요해질 때 아래 레이어로 내립니다.
- 처음부터 확실히 공용인 것(버튼, API 클라이언트, 환경설정)만 `shared`에 둡니다.

---

## 2단계 · 스캐폴딩 : 레이어 폴더 만들기

### Next.js App Router 충돌 처리

Next.js의 `app/`은 라우팅용, FSD의 `app`은 전역 설정용이라 이름이 겹칩니다. **루트 `app/`은 라우팅 전용으로 두고, FSD는 `src/` 안에서 시작합니다.**

```
projects/
├── app/                    # Next.js 라우팅 전용 (page.tsx는 껍데기만)
│   ├── layout.tsx
│   └── page.tsx
├── pages/                  # Pages Router 오인 방지용 빈 폴더
│   └── README.md
└── src/                    # ← 여기서부터 FSD
    ├── app/                # FSD app layer : provider, 전역 스타일
    ├── pages/              # 실제 페이지 구현
    ├── widgets/
    ├── features/
    ├── entities/
    └── shared/
```

> [!WARNING]
> 루트의 빈 `pages/` 폴더를 지우면 Next.js가 `src/pages`를 Pages Router로 인식해 빌드가 깨집니다. 왜 비어 있는지 설명하는 `README.md`를 함께 두세요.

### import 별칭

`tsconfig.json`에 이미 `@/* → ./src/*`가 설정되어 있습니다. 모든 레이어 간 import는 이 별칭을 씁니다.

```ts
import { IntroPage } from "@/pages/intro";
```

---

## 3단계 · 구현 ① : 페이지 슬라이스와 라우트 연결

### 슬라이스 내부 구조

슬라이스 안은 **역할별 세그먼트**로 나눕니다. 자주 쓰는 5개면 충분합니다.

| 세그먼트 | 담는 것                                      |
| -------- | -------------------------------------------- |
| `ui`     | 컴포넌트, 스타일                             |
| `api`    | 서버 요청 함수, 응답 타입                    |
| `model`  | 스토어(Zustand), 도메인 타입, 비즈니스 로직  |
| `lib`    | 이 슬라이스 안에서만 쓰는 헬퍼               |
| `config` | 상수, 설정값                                 |

> [!NOTE]
> 세그먼트 이름은 **본질이 아니라 목적**을 설명해야 합니다. `components`(무엇인가)가 아니라 `ui`(무엇을 위한 것인가)입니다. 같은 이유로 `utils`, `helpers` 같은 이름은 피합니다.

### 인트로 페이지 예시

```
src/pages/intro/
├── ui/
│   ├── IntroPage.tsx
│   └── IntroPage.module.scss
├── model/
│   └── useIntroStep.ts      # gomin → aha → menu 단계 상태
└── index.ts                 # public API
```

```ts
// src/pages/intro/index.ts
export { IntroPage } from "./ui/IntroPage";
```

라우트 파일은 **껍데기만** 둡니다. 화면 구현은 전부 FSD 쪽에 있습니다.

```tsx
// app/page.tsx  (Next.js 라우팅 전용)
export { IntroPage as default } from "@/pages/intro";
```

이 패턴 덕분에 라우트 구조를 바꿔도 페이지 구현은 그대로 재사용됩니다.

---

## 4단계 · 구현 ② : shared로 공용 코드 내리기

`shared`는 슬라이스가 없고 세그먼트로만 구성됩니다. **비즈니스 도메인을 모르는 코드**만 들어갑니다.

```
src/shared/
├── ui/            # Button, Modal — 파스타를 몰라도 되는 컴포넌트
├── api/           # fetch 래퍼, React Query client
├── config/        # 라우트 경로, 환경변수
├── lib/           # 범용 훅, 포맷터
└── styles/        # SASS 변수, mixin
```

```ts
// src/shared/config/routes.ts
export const ROUTES = {
  intro: "/",
  noodle: "/noodle",
  result: "/result",
} as const;
```

판별 기준: **"이 파일 이름에 도메인 용어(파스타, 면, 소스)가 들어가면 shared가 아니다."**

---

## 5단계 · 구현 ③ : entities / features / widgets로 승격

페이지를 만들다 보면 중복이 보입니다. 그때 아래 레이어로 내립니다.

### entities — 명사

프로젝트가 다루는 비즈니스 개체. 데이터 모양과 그것을 보여주는 기본 UI를 담습니다.

```
src/entities/pasta/
├── model/
│   ├── types.ts        # Noodle, Sauce, Pasta 타입
│   └── recipes.ts      # 조합 규칙
├── ui/
│   └── PastaCard.tsx   # 파스타 하나를 그리는 표현 전용 컴포넌트
└── index.ts
```

### features — 동사

사용자가 수행하는 액션. 보통 "무엇을 한다"로 이름 붙습니다.

```
src/features/select-noodle/    # 면을 고른다
src/features/cook-pasta/       # 파스타를 조리한다
```

features는 entities를 import할 수 있지만, 반대는 **금지**입니다.

### widgets — 조합된 화면 영역

여러 feature/entity를 묶어 만든 독립적인 블록입니다. 재사용되는 큰 덩어리(예: 헤더, 재료 선택 패널)만 widgets로 올리고, **한 페이지에서만 쓰는 덩어리는 그냥 페이지 안에 둡니다.**

> [!TIP]
> 승격 판단은 "재사용될 것 같아서"가 아니라 **"실제로 두 번째 사용처가 생겼을 때"** 합니다. 미리 만든 레이어는 대부분 빈 껍데기로 남습니다.

---

## 6단계 · 정리 : public API와 import 방향

### public API : index.ts

각 슬라이스는 `index.ts`로 **밖에 공개할 것만** 내보냅니다. 내부 파일을 직접 경로로 import하지 않습니다.

```ts
// ✅ 좋음
import { PastaCard } from "@/entities/pasta";

// ❌ 나쁨 — 내부 구조에 의존하게 된다
import { PastaCard } from "@/entities/pasta/ui/PastaCard";
```

이렇게 하면 슬라이스 내부를 자유롭게 리팩터링해도 바깥 코드가 깨지지 않습니다.

### 최종 점검 체크리스트

- [ ] 위 레이어 → 아래 레이어 방향으로만 import하는가? (`app > pages > widgets > features > entities > shared`)
- [ ] 같은 레이어의 옆 슬라이스를 import하는 곳은 없는가?
- [ ] 모든 슬라이스에 `index.ts`가 있고, 외부에서는 그것만 통해 접근하는가?
- [ ] 슬라이스 내부 파일을 경로로 직접 import하는 곳은 없는가?
- [ ] `shared`에 도메인 용어가 들어간 파일이 있지 않은가?
- [ ] Next.js 라우트 파일(`app/**/page.tsx`)이 껍데기 역할만 하는가?

> 이 체크리스트는 `architecture-reviewer` 에이전트가 검사하는 항목과 같습니다. 기능을 추가한 뒤 한 번씩 돌려보세요.

---

## 자주 하는 실수

| 실수                                      | 왜 문제인가                                        | 어떻게                                        |
| ----------------------------------------- | -------------------------------------------------- | --------------------------------------------- |
| 시작부터 6개 레이어를 다 만든다           | 빈 폴더만 남고 어디에 둘지 매번 헷갈린다           | `pages` + `shared`로 시작해서 필요할 때 늘린다 |
| `shared/lib/pastaUtils.ts`                | shared가 도메인을 알게 되어 재사용성이 깨진다      | `entities/pasta/lib/`로 옮긴다                |
| 옆 슬라이스 import (`features/a → features/b`) | 슬라이스 간 결합이 생겨 독립 수정이 불가능해진다 | 공통 부분을 아래 레이어로 내려서 공유한다      |
| 컴포넌트 하나마다 슬라이스를 만든다       | 슬라이스는 도메인 단위지 파일 단위가 아니다        | 관련 컴포넌트를 한 슬라이스의 `ui`에 모은다   |
