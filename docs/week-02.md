# 2주차 활동지 — 코딩 에이전트와 컨텍스트

| | |
| :-- | :-- |
| 팀명 | AIMAX |
| 작성일 | 2026.09.09 |
| 참여자 | 정준영,허정민,장동호,김현빈 |

---

## 0. 준비

`memo-seed` 저장소를 엽니다. 다음 파일이 있는지 확인하세요.

- [ ] `schema.sql`
- [ ] `service.js`
- [ ] `routes.js`
- [ ] `CONVENTIONS.md`

---

## 1. 조 나누기

팀을 두 조로 나눕니다. (4인 → 2:2 / 3인 → 1:2)

| 조 | 참여자 |
| :-- | :-- |
| A조 | 허정민,김현빈 |
| B조 | 정준영,장동호 |

**두 조는 같은 과제를 동시에 수행합니다.** 서로의 화면을 보지 마세요.

### 오늘의 과제 (두 조 공통)

> 메모 검색 기능을 추가하라. 제목과 본문에서 키워드로 찾을 수 있어야 한다.

---

## 2. 에이전트에게 준 것

### A조— 이것만 붙여넣습니다

```
메모 검색 기능 만들어줘. 제목이랑 본문에서 키워드로 찾을 수 있게.
```

파일은 **하나도 주지 않습니다.**

### B조 — 네 칸을 모두 채웁니다

```
[지시]
메모 검색 기능을 추가해줘. 제목과 본문에서 키워드로 검색된다.

[규약]
(CONVENTIONS.md 내용 전체를 붙여넣기)

[근거]
(schema.sql, service.js, routes.js 내용 전체를 붙여넣기)

[종료조건]
- GET /memos/search?q=키워드 로 호출된다
- 제목 또는 본문에 키워드가 포함된 메모만 반환한다
- 본인 메모만 반환한다
- q가 비어 있으면 400과 { ok: false, error } 를 반환한다
```

### 실제로 붙여넣은 것 (원문 그대로, 요약 금지)

```
#A조

import streamlit as st
from datetime import datetime

def search_memos(memos, keyword):
    """제목과 본문에서 키워드를 검색"""
    if not keyword:
        return memos

    keyword_lower = keyword.lower()
    results = []

    for memo in memos:
        if (keyword_lower in memo['title'].lower() or
                keyword_lower in memo['content'].lower()):
            results.append(memo)

    return results

def init_session_state():
    """세션 상태 초기화"""
    if 'memos' not in st.session_state:
        st.session_state.memos = []
    if 'next_id' not in st.session_state:
        st.session_state.next_id = 1

def add_memo(title, content):
    """메모 추가"""
    memo = {
        'id': st.session_state.next_id,
        'title': title,
        'content': content,
        'created_at': datetime.now().strftime('%Y-%m-%d %H:%M')
    }
    st.session_state.memos.append(memo)
    st.session_state.next_id += 1

def delete_memo(memo_id):
    """메모 삭제"""
    st.session_state.memos = [m for m in st.session_state.memos if m['id'] != memo_id]

def main():
    st.set_page_config(page_title="메모 검색", page_icon="📝", layout="wide")
    init_session_state()

    st.title("📝 메모 검색 앱")

    col1, col2 = st.columns([2, 3])

    with col1:
        st.subheader("✏️ 새 메모 작성")
        with st.form("add_memo_form", clear_on_submit=True):
            title = st.text_input("제목", placeholder="메모 제목을 입력하세요")
            content = st.text_area("내용", placeholder="메모 내용을 입력하세요", height=150)
            submitted = st.form_submit_button("메모 추가", use_container_width=True)

            if submitted:
                if title and content:
                    add_memo(title, content)
                    st.success("메모가 추가되었습니다!")
                    st.rerun()
                else:
                    st.error("제목과 내용을 모두 입력해주세요.")

    with col2:
        st.subheader("🔍 메모 검색")
        search_keyword = st.text_input(
            "검색어",
            placeholder="제목 또는 내용에서 검색...",
            label_visibility="collapsed"
        )

        filtered_memos = search_memos(st.session_state.memos, search_keyword)

        st.markdown(f"**검색 결과: {len(filtered_memos)}개**")
        st.divider()

        if not filtered_memos:
            if search_keyword:
                st.info("검색 결과가 없습니다.")
            else:
                st.info("저장된 메모가 없습니다. 왼쪽에서 메모를 추가해보세요!")
        else:
            for memo in reversed(filtered_memos):
                with st.container():
                    col_title, col_delete = st.columns([5, 1])

                    with col_title:
                        st.markdown(f"### {memo['title']}")

                    with col_delete:
                        if st.button("🗑", key=f"delete_{memo['id']}", help="삭제"):
                            delete_memo(memo['id'])
                            st.rerun()

                    st.markdown(memo['content'])
                    st.caption(f"작성일: {memo['created_at']}")
                    st.divider()

if __name__ == "__main__":
    main()

#B조

const express = require('express');
const service = require('./service');

const router = express.Router();

// 메모 목록 조회
router.get('/memos', async (req, res) => {
  const memos = await service.listMemos(req.user.id);
  res.json({ ok: true, data: memos });
});

// 메모 단건 조회
router.get('/memos/:id', async (req, res) => {
  const memo = await service.getMemo(req.user.id, req.params.id);

  if (!memo) {
    return res.status(404).json({ ok: false, error: 'MEMO_NOT_FOUND' });
  }

  res.json({ ok: true, data: memo });
});

// 메모 생성
router.post('/memos', async (req, res) => {
  const { title, body } = req.body;

  if (!title || !body) {
    return res.status(400).json({ ok: false, error: 'TITLE_AND_BODY_REQUIRED' });
  }

  const result = await service.createMemo(req.user.id, title, body);
  res.status(201).json({ ok: true, data: { id: result.lastID } });
});

module.exports = router;

-- memo-seed 데이터베이스 스키마
CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  email TEXT NOT NULL UNIQUE,
  name TEXT NOT NULL,
  created_at TEXT NOT NULL
);

CREATE TABLE memos (
  id INTEGER PRIMARY KEY,
  user_id INTEGER NOT NULL,
  title TEXT NOT NULL,
  body TEXT NOT NULL,
  created_at TEXT NOT NULL,
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE INDEX idx_memos_user ON memos(user_id);

const db = require('./db');

/**
 * 사용자의 메모 목록을 최신순으로 조회한다.
 */
function listMemos(userId) {
  return db.all(
    `SELECT id, title, created_at
       FROM memos
      WHERE user_id = ?
      ORDER BY created_at DESC`,
    [userId]
  );
}

/**
 * 메모 한 건을 조회한다. 본인 메모가 아니면 null을 반환한다.
 */
function getMemo(userId, memoId) {
  return db.get(
    `SELECT id, title, body, created_at
       FROM memos
      WHERE id = ? AND user_id = ?`,
    [memoId, userId]
  );
}

/**
 * 메모를 생성한다.
 */
function createMemo(userId, title, body) {
  return db.run(
    `INSERT INTO memos (user_id, title, body, created_at)
     VALUES (?, ?, ?, datetime('now'))`,
    [userId, title, body]
  );
}

module.exports = { listMemos, getMemo, createMemo };
```

> 요약하지 마세요. 나중에 이 기록이 무엇이 결과를 만들었는지 확인하는 근거가 됩니다.

---

## 3. 결과 확인

### A조

| | 확인 항목 | 결과 |
| :-: | :-- | :-- |
| ① | 실행 성공까지 걸린 시간 | 1분 |
| ② | 없는 함수·컬럼을 지어낸 개수 | 5개 |
| | → 지어낸 이름 | search_memos(), init_session_state(), add_memo(), delete_memo() 등 |
| ③ | `CONVENTIONS.md` 위반 개수 | 2개 |
| | → 무엇을 어겼는가 | 1. 함수명 규칙 위반 / 2. 권한 규칙 위반 |
| ④ | 사람이 직접 고친 지점 | 0곳 |
| | → 어디를 어떻게 | |
| ⑤ | **본인 메모만 반환되는가** | 아니오 |

### B조

| | 확인 항목 | 결과 |
| :-: | :-- | :-- |
| ① | 실행 성공까지 걸린 시간 | 1분 |
| ② | 없는 함수·컬럼을 지어낸 개수 | 4개 |
| | → 지어낸 이름 | searchMemos/all/get/run |
| ③ | `CONVENTIONS.md` 위반 개수 | 1개 |
| | → 무엇을 어겼는가 | 1. 함수명 규칙 위반 |
| ④ | 사람이 직접 고친 지점 | 0곳 |
| | → 어디를 어떻게 | |
| ⑤ | **본인 메모만 반환되는가** | 예 |

### ⑤번을 반드시 확인하세요

생성된 SQL에 `user_id` 조건이 들어 있는지 보세요.

없다면 **코드는 정상 동작하지만 남의 메모까지 검색됩니다.** 에러도 나지 않습니다.

---

## 4. 두 조의 결과 비교

작업이 끝나면 두 조가 만든 코드를 나란히 놓고 함께 답하세요.

**4-1. 두 결과의 가장 큰 차이는 무엇입니까?**

```
정확한 결과: routes.js + service.js 수정만
규약 자동 준수: 모든 컨벤션 만족
최소한의 작업: 10~15줄 추가로 완료
검증 가능: 체크리스트로 확인

방식 1로 요청했을 때 실제로 일어난 일 = 저는 기존 Express.js 프로젝트를 무시하고 새로운 Streamlit 앱을 만들었습니다.
방식 2라면 = 기존 프로젝트에 딱 필요한 검색 엔드포인트만 추가했을 것입니다.
```

**4-2. A조의 실패는 모델 탓입니까, 우리가 주지 않은 탓입니까? 근거를 들어 적으세요.**

```
주지않은 탓입니다. 오히려 모델은 말한것보다 더 과한 성능을 만족시켜주었습니다 원하는 조건을 다 만족시켜주고 삭제기능이나 메모추가기능같은것이 더 생겼는데 실패의 이유는 주지않은탓입니다
```

**4-3. B조가 준 자료 중 결과를 가장 크게 바꾼 것 하나를 꼽는다면 무엇입니까? 왜 그렇게 생각합니까?**

```
근거가 가장 중요합니다. 기술 스택을 명확히 할 수 있고 코드 패턴을 직접보여주고 아키텍처의 구조를 이해할 수 있었기 때문입니다
```

---

## 5. PROMPTS.md 기록

위 2번의 프롬프트 원문을 저장소의 `PROMPTS.md`에 추가하고 커밋하세요.

```markdown
## 2026-__-__ · 메모 검색 기능 (2주차 활동)

**지시**
(붙여넣은 프롬프트 원문)

**채택 여부**
(전체 채택 / 일부 채택 — 무엇을 어떻게 수정했는지 / 미채택)

**참고**
(있으면)
```

- [O] `PROMPTS.md`에 추가하고 커밋했습니다

---

## 6. 제출 확인

- [O] 이 활동지를 저장소에 커밋했습니다
- [O] `PROMPTS.md`를 커밋했습니다
