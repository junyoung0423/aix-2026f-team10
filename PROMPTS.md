**2week**

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

**3week**

**프롬프트**

| 사용자 Who | 옷을 온라인으로 구매하려는 사람 | 평소 수면이 부족하다고 느끼는 사람 | 커플 또는 가족 |
| 상황 When/Where | 마음에 드는 옷을 발견했지만, 실제로 입었을 때 어떨지 그리고 이미 가지고 있는 옷들과 잘 어울릴지 알 수 없는 상황 | 수면 패턴이 무너지고 규칙적이지 못하다 | 재미로 혹은 관계 개선이 필요하다 혹은 그 사람을 더 이해하고싶다. |
| 페인포인트 What — 무엇이 **얼마나** | 실제 착용 모습과 기존 옷과의 조합을 확인할 수 없어 구매 후 안 입거나 반품하게 됨. | 피로한 상황이 이어지고아침에 일어나기 힘듦. | 그 사람에 대한 이해도 부족, 관계 개선 혹은 재미 |
| 성공 기준 Done | 	가상 아바타에 피팅된 이미지를 제공함과 동시에 기존 보유 옷과의 매칭 결과를 제공하여 구매 결정에 도움을 줌 | 개인에 맞춘 수면패턴 설정, 규칙적인 삶 확보 | 상대방의 성향 및 관계 향상을 위한 분석리포트 |
위 주제에 대해 인터뷰 질문을 작성해 주는데 항목당 3개씩 작성해줘. 그리고 질문의 형태는 과거형으로 해줘(예 : 최근에 그런 적 있으셨어요?)

**결과**

제공해주신 image_b92f9a.png 파일의 각 가설 항목을 바탕으로, 과거형으로 작성된 인터뷰 질문을 3개씩 정리했습니다.
항목 1 (후보 A): 온라인 의류 쇼핑 및 반품 경험   
- 최근 온라인으로 옷을 구매하셨을 때, 화면에서 본 것과 실제 착용감이 달라서 반품을 고민하셨거나 실제로 반품하신 적이 있으셨나요?
- 온라인으로 산 옷이 집에 있는 다른 옷들과 어울리지 않아서 결국 한 번도 입지 않고 옷장에 방치해 두신 적이 있으셨나요?
- 옷을 반품하시거나 입지 않기로 결정하셨을 때, 구체적으로 어떤 점이 가장 큰 이유로 작용했었나요?
항목 2 (후보 B): 불규칙한 수면 패턴과 일상 지장  
- 최근 불규칙한 수면 패턴 때문에 아침에 일어나기 힘들어서 지각을 하거나 중요한 일정을 놓치신 적이 있으셨나요?
- 수면 부족이나 불규칙한 기상 시간으로 인해 하루 종일 피곤함을 느끼고 일상생활이나 학업에 지장을 겪으신 적이 있으셨나요?
- 그렇게 수면 패턴이 무너졌을 때, 일상생활의 지장을 극복하기 위해 특별히 시도해보셨던 대처 방법이 있으셨나요?
항목 3 (후보 C): 커플/가족 간의 오해와 갈등   
- 최근 가족이나 연인과 대화를 나누던 중, 서로의 생각이나 성향을 잘 몰라서 예상치 못한 오해가 생겼던 적이 있으셨나요?
- 상대방의 의도를 평소와 다르게 파악하여 사소한 문제로 갈등이나 다툼이 발생했던 적이 있으셨나요?
- 그러한 오해나 갈등이 발생했을 때, 두 분은 어떤 방식으로 대화를 풀어나가고 문제를 해결하셨었나요?

