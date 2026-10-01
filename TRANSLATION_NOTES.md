# browser.md 번역 작업 메모 (임시 — PR 올리기 전 이 파일 삭제)

## 현재 상태
- 대상: `src/content/reference/react-dom/browser.md` (canary API, upstream main에 아직 영문판 없음 → **새 파일 추가 PR**)
- 이슈: https://github.com/reactjs/ko.react.dev/issues/1562 (jjamming 할당됨, 2026-09-30 메인테이너 hg-pyun 코멘트)
- **번역 전 구간 완료** (섹션 1~8 + Sandpack 주석)

## 남은 일
1. 전체 1회 통독 (로컬 `yarn && yarn dev` → `localhost:3000/reference/react-dom/browser`)
2. 맞춤법 검사 1회 — https://nara-speller.co.kr/speller/
3. textlint 통과 확인 (`yarn lint` 또는 CI)
4. **이 파일(TRANSLATION_NOTES.md) 삭제**
5. 커밋 squash → 1개로 정리 → push
6. PR 생성 (base: `reactjs/ko.react.dev` main, 본문에 `Closes #1562`, 레포 PR 템플릿 체크리스트 전부 체크)

## 확정 컨벤션

### 손대지 않음
frontmatter / 헤딩 앵커 `{/*...*/}` / JSX 태그(`<Intro>` `<Canary>` `<InlineToc />` `<Note>` `<Sandpack>`) / 코드블록 / API·식별자(`browser` `use` `onBrowserBailout` `componentStack` `reason` `cause` 등) / 링크 URL

### 영문 유지 (음차 금지)
- **React** (리액트 ❌)
- **Suspense** (서스펜스 ❌ — Suspense.md 104:0)
- **Hook** (훅 ❌ — "커스텀 Hook")

### 용어 사전 강제
| 원문 | 번역 |
|---|---|
| Reference | 레퍼런스 |
| Parameters | 매개변수 |
| Returns | 반환값 |
| Caveats | 주의 사항 |
| Usage | 사용법 |
| directive | 지시어 |
| Browser / Component | 브라우저 / 컴포넌트 |
| render, rendering | **렌더링**(하다) — "렌더" 단독 금지(textlint) |
| server rendering | **서버 렌더링** (서버 사이드/서버 측 ❌ — react-dom 이웃 문서 관례) |
| server renderer | 서버 렌더러 |
| boundary | 경계 (바운더리 ❌ — Suspense.md 관례) |
| Conditional / Callback | 조건부 / 콜백 |
| mark A as B | **A를 B로 표시**하다 (useTransition.md 선례) |
| abort | 중단 |
| content | 콘텐츠 (컨텐츠 ❌) |
| expensive | 비용이 크다 |
| opaque value | 불투명한 값 |
| draft(예제 도메인어) | 초안 |

### 형식
- `**optional**` → `` `identifier`**(선택사항)**: `` (이슈 #1478 / PR #1502로 통일)
- "See more examples below." → `[아래에서 더 많은 예시를 확인하세요.](#usage)`
- **코드 문자열 리터럴 = 영문 유지**, **코드 주석 = 한글 번역** (useState·useEffect 관례)

### best-practices (wiki/best-practices-for-translation.md)
문장 끝 콜론 ❌ → 마침표 / 불필요한 쉼표 제거 / 당신·여러분·우리 생략 / 불필요한 피동 자제 / "만약" 생략 가능 / `~들` 복수 남발 금지 / must = "반드시 ~해야 합니다"(≠ "만")
자주 틀린 띄어쓰기: "되어야**하는**"→"되어야 하는", "렌더링**동안**"→"렌더링 동안"

## 자주 빠진 함정 (리뷰 시 재확인)
- `leaves ... in its place` = **그 자리에 남긴다** (이동 ❌)
- reason → Error → cause **방향**: reason=입력, Error=React가 만든 봉투, cause=출력
- `Do not throw it` = **throw 하지 마세요** ("잊지 마세요" ❌)
- `instead of A, B, or C` = A·B·C **전부**를 대체
- `pass X as the reason to Y` = **Y에 X를** 전달
- `recovered` 누락 주의

## 참고
- upstream sync PR은 6/23(#1534) 이후 머지 정체. 기다리지 말 것 — main 기준 새 파일로 PR.
