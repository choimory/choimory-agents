# Histories

- .agents/histories는 프로젝트의 작업 히스토리들을 작성하는 곳.
- 작업 사항들을 histories에 기록해가며 작성하고 작업 내역을 남겨, 향후 작업사항들을 참고할 수 있도록 한다.
- 내가 직접 문서를 만들어 놓을 때도 있지만, 그렇지 않을 때는 문서 디렉토리 양식대로 문서를 생성하며 작업을 진행한다.
- 해당 디렉토리 안에서 작업 개요 및 분석과 계획은 plan1~.md, 작업 내용은 task1~.md, 작업 완료 요약은 done1~.md에 작성한다.
- 문서 마크다운 파일은 utf-8로 작성한다.
- 자세한 사항은 하단 Histories 디렉토리 양식란 참고.
- 디렉토리명은 영문으로 작성한다

## Histories 디렉토리 구조

```markdown
# Histories 디렉토리 구조 (.agents/histories/calendar.md, .agents/histories/{year}/{month}/{day}/{task-agenda}/plan1~.md, task1~.md, done1~.md)

.agents/histories/calendar.md
.agents/histories/{year}/{month}/{day}/{task-agenda}/plan1.md
		                                             plan2.md
		                                             task1.md
		                                             task2.md
		                                             task3.md
		                                             done1.md
		                                             done2.md

---

# 예시 (.agents/histories/calendar.md, .agents/histories/2025/01/01/ssl-apply/plan1.md, plan2.md, task1.md, task2.md, task3.md, done1.md)

.agents/histories/calendar.md
.agents/histories/2025/01/01/ssl-apply/plan1.md
                                       plan2.md
                                       task1.md
                                       task2.md
                                       task3.md
                                       done1.md
                                       done2.md
```

---

## Histories 문서 작성 규칙

- 기본적으로는 대화로 이야기하며, 문서 작성은 내가 작성 요청할 때만 작성한다.
- 문서 작성 요청할 땐 기본적으로 기존 글들을 수정하지 않는다.
- 질문과 대화를 1:1 형식으로 `--`로 구분선을 그어 구분해가며, 밑에 새로 추가하는 형식으로 적을 것.
- 내 질문 내용을 적을땐 요약하거나 변경하지 말고 그대로 적을것.
- 내 질문 내용 중, “지금 내용 문서에 추가해줘” 같은 내용은 적을 필요가 없고, 해당 답변이 나오게끔 물어본 질문을 적어야 함.
- 문서는 H1 > H2 > H3 형식으로 주제를 구분하며 적을 것.
- 문서 맨 위에 목차도 적고, 대화 내용을 추가할 때마다 업데이트 할 것.
- 문서 마크다운 파일은 utf-8로 작성한다.
- 민감한 정보는 마스킹 한다. (api-secret-key, ssh-private-key, token-secret-key, terraform-resource-id...)
- 작성할 문서의 양식은 하단 문서 작성 양식란 참고.

## Histories 작성 양식

```markdown
# 목차

- (목차내용)(마크다운 링크)

---

# 개요

- (이러이러한 상황)
- (이러이러한 작업을 해야함)
- (이런저런것들을 참고해서 진행)
- (그 외 작업과 관련된 간략한 내용들)

---

# (사용자 질문 주제1)

> (질문내용. 내가 작성한 내용 그대로 적어줄것)

## (답변)

(답변내용)

## (주제1)

(답변내용)

## (주제2)

(답변내용)

### (주제2-1)

(답변내용)

### (주제2-2)

(답변내용)

## (주제3)

(답변내용)

---

# (사용자 질문 주제2)

> (질문내용. 내가 작성한 내용 그대로 적어줄것)

## (답변)

(답변내용)

## (주제1)

(답변내용)

## (주제2)

(답변내용)

### (주제2-1)

(답변내용)

### (주제2-2)

(답변내용)

## (주제3)

(답변내용)

---

...
```

---

## calendar.md 작성 규칙

- 문서 디렉토리의 일자별 작업 내용을 하나의 문서에 한 줄로 정리하고자 함.
- 작업 일자별 디렉토리들의 작업 사항을 CALENDAR.md에 추가한다.
- 분석 작업을 시작할땐 calendar.md에 PLAN:[Simple-agenda][(Directory-name)]을 추가한다.
- 구현 작업을 시작할땐 calendar.md에서 상태를 TASK로 바꿔서 TASK:[Simple-agenda][(Directory-name)]으로 변경한다.
- 모든 작업을 마무리할땐 calendar.md에서 상태를 DONE으로 바꿔서 DONE:[Simple-agenda][(Directory-name)]으로 변경한다.
- 작업 주제 알려주면서 “(Agenda) 작업할거야” 하면, 내용 없이 목차와 개요만 존재하는 plan1 빈 문서를 만들어주고 CALENDAR.md에 PLAN:[Simple-agenda][(Directory-name)]으로 작업을 추가한다.
- plan 문서를 작성하면서 분석하던 것을 마무리하고, 작업을 시작해달라 하면 CALENDAR.md에 해당 작업을 TASK:[Simple-agenda][(Directory-name)]로 변경하고 작업을 시작함.
- 작업이 모두 종료 되어서 “작업 마무리하자” 하면, 작업 전체 요약을 done1~.md에 적고, CALENDAR.md에 해당 작업을 DONE:[Simple-agenda][(Directory-name)]로 변경하고, README.md에 변경사항을 반영하는 것으로 마무리한다.
- calendar는 디렉토리별로 하나씩만 작성한다.
- 과거 디렉토리의 작업을 다시 재개하거나 추가 진행할때엔, calendar에 동일한 디렉토리를 다른 일자로 새로 추가할 필요가 없으며, 기존 디렉토리와 calendar의 일자를 수정할 필요도 없다. 기존 calendar의 Process만 현재 진행상태에 맞게 변경하면 된다.
- 자세한 사항은 하단 calendar.md 양식란 참고.

## calendar.md 양식

```markdown
# 2025

- yyyy-MM-dd: [Process->PLAN(분석),TASK(구현),DONE(완료)]:[Simple-agenda][(Directory-name)]
- 2025-01-02: PLAN:SSL 적용작업(ssl-apply)
- 2025-01-03: TASK:회원 api 리팩토링 작업(member-api-refactor), DONE:프론트 마이페이지 적용 작업(front-mypage-apply)

# 2026
...
```

---