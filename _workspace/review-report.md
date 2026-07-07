# Review Report

- 대상 커밋 초안: `_workspace/commit-draft.md`
- 대상 diff: `git diff --cached` (.claude/skills/commit-message/SKILL.md, _workspace/commit-draft.md, _workspace/review-report.md)

## 판정: PASS

## 사유:
1. 형식: `fix(skill): commit-message 스킬의 git diff 명령어 오타 수정` — `type(scope): subject` 패턴을 정확히 준수 (type=fix, scope=skill, 콜론 뒤 공백 1개, 제목 총 49자로 길이 적정, 말미 마침표 없음). 제목과 본문 사이 빈 줄 존재.
2. scope 적절성: 핵심 변경이 `.claude/skills/commit-message/SKILL.md`에서 발생했으므로 scope `skill`은 실제 변경 위치와 부합. type `fix`도 오타 수정이라는 변경 성격과 일치.
3. 사실 일치: staged diff에서 SKILL.md의 precondition 체크 명령어가 `git diff --cached --quied` → `git diff --cached --quiet`로 수정되었고, 본문 1문장이 이를 정확히 서술함. 또한 diff에 `_workspace/commit-draft.md`, `_workspace/review-report.md` 두 신규 파일 추가가 포함되어 있고, 본문 2문장("스킬 실행 결과물인 커밋 초안과 리뷰 리포트를 _workspace/에 추가했다")이 이를 정확히 서술함. 과장·누락·허위 서술 없음.

## 수정지시 : 해당 없음 (PASS)
