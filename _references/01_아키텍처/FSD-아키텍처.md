# FSD 아키텍처

- 참고문서: [https://fsd.how/kr/docs/get-started/overview/](https://fsd.how/kr/docs/get-started/overview/)

## 개요

- Feature-Sliced Design (FSD) 는 프론트엔드 애플리케이션의 코드를 구조화하기 위한 아키텍처 방법론입니다.
- 이 방법론의 목적은 요구사항이 바뀌어도 코드 구조가 무너지지 않고, 새 기능을 쉽게 추가할 수 있는 프로젝트를 만드는 것입니다.
- FSD는 코드를 얼마나 많은 책임을 가지는지와 다른 모듈에 얼마나 의존하는지에 따라 계층화합니다.

## 구조 예시

- 간단한 FSD 구조는 다음과 같습니다.

```
├── app/
├── pages/
└── shared/
```

- 이 상위 폴더들이 Layer입니다. Layer는 표준화된 이름을 가지며, 각각 명확한 역할을 담당합니다.

```
├── app/
│   ├── routes/
│   └── analytics/
├── pages/
│   ├── home/
│   ├── article-reader/
│   │   ├── ui/
│   │   └── api/
│   └── settings/
└── shared/
    ├── ui/
    └── api/
```

- 📂 pages 내부의 home, article-reader, settings는 Slice입니다.
- slice는 비즈니스 도메인(이 예시에서는 각 페이지) 단위로 코드를 구분합니다.
- 각 Slice 안에는 ui, api, model 등의 Segment가 있습니다. Segment는 코드의 역할이나 기능에 따라 분류됩니다.

ui - UI Components
api - REST/GraphQL Client, Fetchers
model - State, Types, Selectors

## 개념

- FSD는 다음과 같은 3단계 계층 구조를 따릅니다:

```
     Layers                Slices              Segments

  ┌────────────┐
  │    app     │
  ├────────────┤
  │   pages    │
  ├────────────┤
  │  widgets   │      ┌────────────┐      ┌────────────┐
  ├────────────┤      │    user    │      │     ui     │
  │  features  │      ├────────────┤      ├────────────┤
  ├────────────┤──────┤    post    │──────┤   model    │
  │  entities  │      ├────────────┤      ├────────────┤
  ├────────────┤      │  comment   │      │    api     │
  │   shared   │      └────────────┘      └────────────┘
  └────────────┘
```

- 위 다이어그램은 FSD의 계층 구조를 시각적으로 보여줍니다.
- 세 개의 수직 블록 그룹은 각각 Layer, Slice, Segment를 나타냅니다.
- 왼쪽의 Layer 블록에는 app, pages, widgets, features, entities, shared가 포함됩니다.
- entities Layer 안에는 여러 개의 Slice가 존재하며, 예시로는 user, post, comment 등이 있습니다.
- Slice는 비즈니스 도메인별(user, post, comment)로 나뉘며, 각 Slice 안의 Segment들은 코드의 역할(예: UI, 데이터, 상태) 에 따라 구성됩니다.
- Slice에는 ui, model, api Segment가 포함됩니다.

## Layers 계층

- Layer는 모든 FSD 프로젝트의 표준 최상위 폴더입니다.
- 상위 Layer는 자신보다 하위 Layer를 참조 할 수 있지만, 하위 Layer가 상위 Layer를 참조하는 것은 허용되지 않습니다.
- 예를 들어 pages는 features나 entities의 모듈을 참조할 수 있지만, features가 pages를 참조하는 것은 금지됩니다.

```
1. App - Routing, Entrypoint, Global Styles, Provider 등 앱을 실행하는 모든 요소
   - Next.js에서는 app 폴더가 라우트 구조로 사용되기 때문에, 프로젝트 최상위에 app 폴더와 src 폴더를 분리하고, src 폴더 내에서 FSD 아키텍처를 준수하여 개발합니다. (Next.js app layer에서 FSD와 Next.js의 충돌 참고)
2. Pages - Route 기준으로 구성된 주요 화면 단위
3. Widgets - 여러 UI 요소를 조합해 구성한 독립적인 화면 영역
4. Features - 사용자에게 비즈니스 가치를 제공하는 액션을 구현한 재사용 가능한 제품 기능 단위
5. Entities - 프로젝트가 다루는 비즈니스 Entity
6. Shared - 모든 Layer에서 재사용되는 코드(라이브러리, 유틸리티 등)
```

> [!NOTE]
> App/Shared Layer는 Slice 없이 Segment로 구성됩니다.

## Slices 계층

- Slice는 Layer 내부를 비즈니스 도메인별로 나눕니다.
- 이름/개수에 제한이 없으며, 같은 Layer 내 다른 Slice를 참조할 수 없습니다. - - 이 규칙이 높은 응집도와 낮은 결합도를 보장합니다.

## Segments 계층

Slice와 App/Shared Layer는 Segment로 세분화되어, 코드의 역할(예: UI, 데이터 처리, 상태 관리 등)에 따라 코드를 그룹화합니다. 일반적으로 다음과 같은 Segment를 사용합니다

- ui - UI components, date formatter, styles 등 UI 표현과 직접 관련된 코드
- api - request functions, data types, mappers 등 백엔드 통신 및 데이터 로직
- model - schema, interfaces, store, business logic 등 애플리케이션 도메인 모델
- lib - 해당 Slice에서 여러 모듈이 함께 사용하는 공통 library code
- config - configuration files, feature flags 등 환경/기능 설정

대부분의 Layer에서는 위 다섯 Segment로 충분합니다. 필요하다면 App 또는 Shared Layer에서만 추가 Segment를 정의하세요.

## Next.js app layer에서 FSD와 Next.js의 충돌

- 참고문서: [https://fsd.how/kr/docs/guides/tech/with-nextjs/](https://fsd.how/kr/docs/guides/tech/with-nextjs/)
- App Router에서 app 폴더는 라우트 구조로 사용됩니다. 반면 FSD에서 app 폴더는 애플리케이션 전역을 담당하는 layer입니다. 두 구조는 app 폴더의 역할이 다르고, 같은 위치에 둘 경우 구조 충돌이 발생합니다.
- 이 충돌을 피하려면 Next.js의 app 폴더는 프로젝트 루트에 두고, 라우팅에 필요한 파일만 배치합니다.
  페이지는 src 하위의 FSD pages layer에 두고, 루트 app 폴더의 각 라우트 파일에서 이를 가져와 연결합니다.
- 프로젝트 루트에는 빈 pages 폴더도 함께 두어야 합니다. 그렇지 않으면 App Router를 사용하더라도 Next.js가 src/pages를 Pages Router로 인식하므로 빌드가 실패합니다.
- 또한 비어 있는 폴더이더라도 왜 필요한지 설명하는 README.md를 이 루트 pages 폴더 안에 함께 두는 것이 좋습니다.

- 폴더 구조 예시

  ```
  ├── app/                    # Next.js의 App folder (라우팅 전용)
  │   ├── api/
  │   │   └── get-example/
  │   │       └── route.ts
  │   └── example/
  │       └── page.tsx
  ├── pages/                  # Next.js가 Pages Router로 오인하지 않도록 두는 빈 폴더
  │   └── README.md           # 이 폴더가 왜 비어 있는지 설명
  └── src/
      ├── app/                # FSD app layer (전역 설정, provider 등)
      ├── pages/              # FSD pages layer (실제 페이지 구현)
      ├── widgets/
      ├── features/
      ├── entities/
      └── shared/
  ```
