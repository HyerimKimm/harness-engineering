# 이 폴더는 비어 있어야 합니다

Next.js가 `src/pages`를 Pages Router로 오인하는 것을 막기 위한 폴더입니다.

App Router를 사용하더라도 이 폴더가 없으면 Next.js는 `src/pages`(FSD의 pages layer)를 Pages Router 디렉터리로 인식해 빌드가 실패합니다.

- 라우팅 파일은 프로젝트 루트의 `app/`에 둡니다.
- 화면 구현은 `src/pages/`(FSD pages layer)에 두고, `app/`의 라우트 파일에서 가져와 연결합니다.
- **이 폴더에는 어떤 파일도 추가하지 마세요.**

참고: `_references/01_아키텍처/FSD-아키텍처.md`
