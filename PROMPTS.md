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

**4week**

**프롬프트**
WHILE 사용자가 기존 보유 옷을 등록한 후 새로 선택한 옷과의 조합을 확인하는 상태인 동안 THE 시스템은 SHALL 색상 조합 매칭을 분석하여 기존 옷과 함께 착용한 가상 아바타 피팅 이미지를 제공한다. 이 문장에 대한 EARS 판정 방법을 첨부한 예시 자료를 참고해서 작성해줘

**결과**
판정 방법 : 기존 보유 옷을 등록한 상태에서 새로운 옷을 선택해 조합 확인 화면에 진입한 후, 색상 조합 매칭 분석 결과 및 기존 옷과 함께 착용한 가상 아바타 피팅 이미지가 정상적으로 화면에 표시되는지 확인

**프롬프트**
WHEN 사용자가 기존에 보유하고 있던 옷을 착장한 사진을 찍어올리면 THE 시스템은 SHALL 가상의 아바타가 해당 옷을 착장한 이미지를 보여준다. 에 대한 판정방법 제시해줘.

**결과**
옷 종류가 서로 다른 착장 사진 5장을 준비해 하나씩 업로드한다.
1. 업로드하면 시스템이 에러 없이 아바타 이미지를 생성해 표시하는지 확인한다.
2. 팀원이 아닌 제3자 평가자에게 원본 사진과 생성된 아바타 이미지를 보여주고 같은 옷을 입은 것으로 알아볼 수 있는가에 대한 답을 예/아니오로 듣는다.                     3. 3건 이상에서 "예"가 나오면 통과로 판정한다.

**프롬프트**
내가 찾는 옷이 없을 때 그 옷의 수치와 디자인을 고려하여 최대한 비슷한 다른 옷을 추천해줍니다. 이게 ac라고 생각하면 판정방법 어떻게 쓸래

**결과**
1. 입력 데이터:
- 찾는 옷: 검은색 긴팔 티셔츠, XL 사이즈
- 패턴: 단색, 둥근 넥

2. 실행:
- 검색 API 호출 → 결과 0개 반환
- 추천 시스템 자동 실행

3. 예상 결과:
- 비슷한 옷 5개 이상 추천
- 각 상품의 유사도 점수 80% 이상

4. 확인 방법:
- API 응답에서 추천 상품 확인
- 각 상품의 수치 비교:
* 사이즈: XL (정확) 또는 L/2XL (1단계 오차 허용)
* 색상: 검은색 또는 어두운 색
* 소매: 긴팔 계열
- UI에서 "비슷한 상품" 섹션에 표시되는지 확인

**프롬프트**
EARS 패턴의 Optional인 ⁠WHERE⁠ 구문 구조를 활용해, 상·하의 매칭 웹 서비스에서 특정 기능이 포함될 때의 정상 경로 요구사항 문장과 판정 방법을 작성해줘

**결과**
판정 How to check

매칭된 화면에서 저장 버튼을 클릭했을 때 마이페이지 북마크/보관함 목록에 해당 코디가 정상적으로 등록되는지 확인

**5week**
 
**프롬프트**
[5주차 활동지 양식 첨부]
우리가 확정한 주제는 공공 의류 데이터셋에서 내가 가진옷과 비슷한 옷을 찾아주고, 거기에 매치하고 싶은 옷의 색상, 사이즈들을 입력하면 그에 맞는 옷을 추천해주고, 피팅된 이미지까지 제공하는 거야.
 
**결과**
4주차 사용자 스토리는 볼 수 없어서 태스크는 주제에 맞춰 가정했고, 담당은 팀원 4명에게 2개씩 임의로 나눴어요.
(태스크 8개, 의존 관계 그래프, MoSCoW, Walking Skeleton이 채워진 활동지 초안)
 
**프롬프트**
[5주차 강의자료 PDF 첨부]
이거 피피티 한장한장 설명해줘 잘 모르겠어
 
**결과**
- SDD: 요구가 바뀌면 완성된 벽을 부수지 말고 ①번 문서(명세)를 고쳐서 다시 만든다는 뜻이에요.
- Atomic Task: "백엔드 개발"은 언제 끝났는지 알 수 없고, "제출 현황 조회 기능 — 과제 ID로 목록 반환"은 끝났는지 눈으로 확인돼요.
- 사이클: 화면과 데이터 구조가 서로 기다리면 아무도 시작을 못 해요. 둘 다 필요한 결정을 맨 앞으로 떼어내요.
- Won't: 다국어·고대비는 특정 사용자가 아예 못 쓰게 되는 배제형이고, 추이 그래프는 편의형이에요.
**프롬프트**
갯수는 신경쓰지말고 최적화된 거로 만들어줘
 
**결과**
개수보다 수업 기준(따로 검증 가능, 담당자 1명, 사이클 없음)에 맞춰 9개로 나눴습니다.
- 화면과 추천이 서로 결과 형식을 기다리지 않도록 "결과 데이터 형식 합의"(#1)를 맨 앞에 뒀어요.
- 가장 불확실한 AI-Hub 접속과 피팅 생성은 선행 태스크 없이 처음부터 시작해요.
**프롬프트**
핵심 시나리오: 내 옷 사진 1장을 올리고 색상·사이즈를 입력하면 → AI-Hub 데이터에서 비슷한 옷 중 조건에 맞는 옷을 추천하고 → 피팅 이미지 1장이 화면에 나온다 -> 근데 이거맞아? 내 옷사진 올리면 그거와 최대한 비슷한 옷을 찾아서 아바타에 피팅하고 이후에 내가 사고자하는 옷의 조건(색상, 디자인)을 입력하면 그에 맞는 옷들이 기존의 내옷을 입고있는 아바타에게 입혀지는거지. 그렇게 되면 내가 가지고 있는 옷과 내가 사고자하는 옷의 조합을 확인 할 수 있으니까
 
**결과**
아니요, 제가 쓴 시나리오는 틀렸어요. 말씀하신 흐름은 내 옷과 새 옷의 조합을 미리 보는 것이에요.
핵심 시나리오: 내 옷 사진을 올리면 비슷한 옷을 찾아 아바타에 입히고 → 색상·디자인 조건을 입력하면 추천된 옷이 내 옷을 입은 아바타에 겹쳐 입혀진 이미지가 나온다.
 
**프롬프트**
초기 설정: 성별·연령대·키·몸무게 입력 → 핏 판정에 필요한 세부 치수(가슴·어깨·팔길이·허리·엉덩이) 자동 추정 → 사용자가 확인·수정
내 옷 등록: 사진 촬영(배경 제거 + 카테고리·색상 자동 추정 후 확인) 또는 드롭다운으로 카테고리·색상만 선택. 치수 입력 없음
새 옷 입력: 상품 이미지 + 사이즈표 캡처 → 실측 자동 추출 → 확인
핏 확인 (필수): 신체 치수와 새 옷 실측으로 부위별 여유분 계산 → 데이터 기반 기준으로 타이트/적당/여유 판정, 사이즈가 여러 개면 가장 맞는 사이즈 추천
코디 추천: 새 옷 카테고리에 따라 슬롯 구성(상의 → 하의·아우터, 하의 → 상의·아우터, 아우터 → 상의·하의, 원피스 → 아우터) → 슬롯별 후보 전체를 Jev에 넘겨 최적 조합 선택. 확신도 높으면 1개, 낮으면 2~3개 제시
피팅 이미지 생성: 체형 묘사 + 핏 판정 결과 + 조합 옷 이미지로 프롬프트 생성 → Higgsfield 호출(나노바나나, gpt image 등) → 결과를 기기에 저장 // 이거로 하려고하는데 조금 보완해줘
 
**결과**
- 쇼핑몰 사이즈표는 보통 단면 길이라서 몸 치수(둘레)와 그대로 빼면 모든 옷이 "타이트"로 판정돼요. ×2로 둘레 환산이 필요해요.
- "데이터 기반 기준"이 비어 있어요. 부위별 판정 기준표와 출처를 정해야 해요.
- 아우터를 샀을 때 원피스와 조합하는 경우가 빠져 있고, 슬롯에 등록된 옷이 없으면 오류가 날 수 있어요.
- Higgsfield API는 크레딧이 차감되고, 원조 나노바나나는 API가 종료돼서 모델 버전을 명시해야 해요.
**프롬프트**
아니 지금은 너무 큼지막해졌고 음 넘 구체적인수치 적지말고 [위 기획 흐름] 이런식으로 큰 흐름만
 
**결과**
주신 6단계 흐름을 그대로 태스크로 옮겼어요. 완료 조건도 주신 문장을 거의 그대로 쓰고 구체적인 숫자는 뺐어요.
| # | 태스크 | 선행 |
| 1 | 데이터 형식 정하기 | 없음 |
| 2 | 초기 설정 | 1 |
| 3 | 내 옷 등록 | 1 |
| 4 | 새 옷 입력 | 1 |
| 5 | 핏 확인 | 2, 4 |
| 6 | 코디 추천 | 3, 4 |
| 7 | 피팅 이미지 생성 | 없음 |
| 8 | 화면 만들기 | 1 |
 
**프롬프트**
왜 피팅이미지 생성이 2번이야 마지막쯤아냐?
 
**결과**
맞아요, 마지막이 맞아요. 선행 칸이 "없음"으로 되어 있어서 맨 앞으로 올라갔던 거예요. 피팅 이미지는 핏 판정 결과와 코디 조합이 있어야 프롬프트를 만들 수 있으니, 선행을 핏 확인·코디 추천으로 바꾸고 마지막으로 옮겼어요.
1단계: 데이터 형식 → 2단계: 화면, 초기 설정, 내 옷 등록, 새 옷 입력 → 3단계: 핏 확인, 코디 추천 → 4단계: 피팅 이미지 생성
 
**프롬프트**
15주차에 발표장에서 시연할 수 있는 형태인가? 이부분은 뭐라고 답하는게좋을까
 
**결과**
- 가능하다. 특수 장비 없이 휴대폰(또는 노트북) 하나로 시연하고, 화면을 프로젝터에 띄워 보여준다.
- 자동 추정·인식·추출이 틀려도 사용자가 확인·수정하는 단계가 있어 흐름이 끊기지 않는다.
- 인터넷 연결이 안 되거나 크레딧이 부족하면, 미리 만들어 둔 결과 이미지로 이어서 보여준다.
**프롬프트**
근데 지금 저거 다 좋은데 가지고있는 옷 사진과 사이즈표를 캡쳐해서 올리면 위에 보내줬던 AI-HUB 의류 데이터셋에서 가장 유사한 옷의 정보를 가져와서 아바타에 피팅시키고 새옷도 마찬가지고 AIHUB데이터에서 가져와서 하는거로 수정해서 그것만 추가해서 다시 만들어주라
 
**결과**
"AI-Hub 의류 데이터 준비"와 "비슷한 옷 찾기" 태스크를 추가했어요. 내 옷 등록과 새 옷 입력이 둘 다 쓰는 기능이라 하나로 떼어 앞에 뒀어요.
1단계: 데이터 형식, AI-Hub 데이터 준비 → 2단계: 화면, 초기 설정, 비슷한 옷 찾기 → 3단계: 내 옷 등록, 새 옷 입력 → 4단계: 핏 확인, 코디 추천 → 5단계: 피팅 이미지 생성
 
