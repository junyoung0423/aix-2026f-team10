**A조**

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
                        if st.button("🗑️", key=f"delete_{memo['id']}", help="삭제"):
                            delete_memo(memo['id'])
                            st.rerun()

                    st.markdown(memo['content'])
                    st.caption(f"작성일: {memo['created_at']}")
                    st.divider()

if __name__ == "__main__":
    main()


**B조**

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
  id         INTEGER PRIMARY KEY,
  email      TEXT NOT NULL UNIQUE,
  name       TEXT NOT NULL,
  created_at TEXT NOT NULL
);

CREATE TABLE memos (
  id         INTEGER PRIMARY KEY,
  user_id    INTEGER NOT NULL,
  title      TEXT NOT NULL,
  body       TEXT NOT NULL,
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


**결과**


A조	확인 항목	결과
①	실행 성공까지 걸린 시간	:1분
②	없는 함수·컬럼을 지어낸 개수	:5개
→ 지어낸 이름	search_memos(),init_session_state(),add_memo(),delete_memo(),main()
③	CONVENTIONS.md 위반 개수:2개
→ 무엇을 어겼는가	1. 함수명 규칙 위반
2. 권한 규칙 위반
④	사람이 직접 고친 지점:	0곳	
⑤	본인 메모만 반환되는가:	아니오

B조	확인 항목	결과
①	실행 성공까지 걸린 시간:1분
②	없는 함수·컬럼을 지어낸 개수:4개
→ 지어낸 이름:searchMemos/all/get/run
③	CONVENTIONS.md 위반 개수:1개
→ 무엇을 어겼는가:1. 함수명 규칙 위반
④	사람이 직접 고친 지점:0곳	
⑤	본인 메모만 반환되는가:예
