# 10주차 활동지 / Week 10 Worksheet

**막힘 진단 · 데모 시나리오 초안 / Blockers and demo scenario draft**

- 작성일 / Date: 2026-11-11
- 참여자 / Present: 정준영, 허정민, 장동호, 김현빈

---

## ① 착수 점검표 / Start checklist

수업 시작 후 10분 안에 확인합니다. 체크되지 않은 항목이 있으면 그것부터 해결하고 개발을 시작합니다.
*Check within the first 10 minutes. Fix any unchecked item before you start coding.*

- [o] 초기 파일이 저장소에 올라와 있다 / Initial files are in the repository
  - `junyoung0423/aix-2026f-team10_Fit_Ruler_v2` main. README, CONTRIBUTING, 3주 계획, 결정 기록(`docs/decisions.md`), CI(Python 3.10·3.12, web)
- [o] 7주차 이슈가 GitHub에 등록되어 있고 담당자가 지정되어 있다 / Week 7 issues are on GitHub with owners
  - 이슈 #1~#13은 등록됨. **GitHub 담당자(Assignees)는 아직 비어 있음** → 이번 주에 3주 계획 역할(A·B·C·D)대로 지정
- [o] 팀원 모두 같은 환경에서 실행된다 / Everyone can run it in the same environment
  - `pip install -e ".[dev]"` + CI로 같은 환경 확인. 다른 팀원 PC에서 처음부터 설치하는 시험은 아직 안 함(3주차 B6에서 재현)
- [o] 브랜치 규칙을 정했다 / Branch rule agreed — `종류/이슈번호-요약`(예: `feat/3-aihub-adapter`), 커밋 `#번호 내용`, PR 제목 `[역할][#번호] 내용`, 리뷰어 승인 + CI 통과 후 **squash merge만**(저장소 설정으로 고정)

---

## ② 개발 일정 / Development schedule

5주차 **작업 순서**와 7주차 **이슈·마일스톤**을 바탕으로 이슈를 주차별로 배치합니다.
*Place your issues week by week, based on the Week 5 work order and Week 7 issues and milestones.*

- 11주차에 MVP가 동작해야 합니다. / Fill backwards from the deadlines.
- 선행 이슈가 끝나기 전에 뒤 이슈를 넣지 않습니다. / Never schedule an issue before the ones it depends on.
- 한 사람에게 몰리지 않았는지 확인합니다. / Check that no one is overloaded.

| 주차 Week | 수업 일정 Class | 끝낼 이슈 # Issues to close | 담당 Owner |
|---|---|---|---|
| 9주차 | 개발 착수 | #1 기준, #2 핏 계산기(종료). 뼈대·스파이크: #9(A1·A2), #3(C1), #4 #8(C2), #7(D2), #10 #11(D1) | A·B·C·D |
| 10주차 | 막힘 진단 · 데모 시나리오 초안 | #3(C3 의류 10~20벌), #4(C3 견본 10개 이상), #5(B3 견본 검색), #6(C4 최적화), #7(D3 몸 생성), #8(C4 착장·D4 점검), #12(B3 작업 API·B4 실제 어댑터 연결), #9(A3·A4 화면 연결) | B: #5 #12 · C: #3 #4 #6 #8 · D: #7 #8 · A: #9 |
| 11주차 | **MVP 완료 · 중간 데모** | MVP(핏 리포트 + 3D 착장) 데모. #10(C5 렌더 클립·D5 실사화), #11(D6 검증), #12(B5 실패 처리), #13(B6 통합 실행), #9(A5 동의·A6 결과 화면) | C: #10 · D: #10 #11 · B: #12 #13 · A: #9 |
| 12주차 | 핵심 경로 테스트 | 기준 사례 피팅 전체 재현, 사용성 테스트(3~5명) 반영, 단계별 실패 주입 테스트 | 전원 |
| 13주차 | **배포 · 실행 패키지** | #13 설치 안내·실행 명령 정리, 다른 PC 재현 확인, `PROMPTS.md` 최종 정리 | B(통합)·A(설치 따라 하기 리뷰) |

> 일정이 맞지 않으면 기능을 줄입니다. 뺀 기능은 5주차 Won't 목록에 옮깁니다.
> *If it doesn't fit, cut scope and move the feature to your Week 5 Won't list.*
>
> 축소 순서(3주 계획): 등록 의류 20벌 → 10벌, 웹 3D 뷰어 → 정지 이미지, 사용성 테스트 5명 → 3명. 견본 10개·의류 10벌·실사화·검증은 유지.

---

## ③ 이번 주 목표 이슈 / This week's issues

| # | 제목 Title | 담당 Owner | 완료 Done |
|---|---|---|---|
| #3 #4 | C3: AI Hub 티셔츠 10벌 이상 등록·누락 점검 + 패턴 견본집 10개 이상 | C | [ ] |
| #5 #12 | B3: 견본 검색 + 피팅 작업 API(`POST/GET /api/fittings`) | B | [ ] |
| #6 #8 | C4: 견본을 실제 의류 치수에 맞게 최적화 + Blender 착장 어댑터 | C | [ ] |
| #7 | D3: 사용자 치수로 SMPL 몸 생성 | D | [ ] |
| #8 #10 | D4: 착장 점검(관통) + fal/LTX 실사화 어댑터 | D | [ ] |
| #9 | A3·A4: 실제 API 연결, 진행 상태·3D 미리보기·관리 화면 | A | [ ] |
| #12 #13 | B4: 가짜 ↔ 실제 어댑터 교체, 기준 사례 1회 실행 스크립트 | B | [ ] |

---

## ④ 오늘 진행한 것 / Progress today

**머지 (1주차 PR 8개 모두 main에)**
- B1(#14) 공통 스키마·포트·가짜 어댑터, B2(#15) 핏 판정 계산기·핏 API → #2 종료
- A1(#16) 와이어프레임·웹 뼈대·가짜 API, A2(#17) 치수 입력 폼·핏 리포트 화면
- C1(#19) AI Hub 변환 어댑터, 의류 `color_hex`·`fabric` 필드
- C2(#18) GarmentCode → Blender 스파이크 코드 이전. 다시 돌린 결과가 이전과 같음(관통 0)
- D1(#20) fal/LTX 실사화 1회 호출: 1회 $0.07, 생성 약 43초
- D2(#21) SMPL 평균 체형 A자세, 신체 측정 함수(measure_v1)
- 테스트 약 620개, CI에 garment·realize·body 선택 의존성 추가

**확인**
- 실제 서버(`uvicorn server.app:app`) + 웹 실제 모드로 입력 → 티셔츠 선택 → 핏 리포트 흐름 동작
- AI Hub 71501 반팔 라운드넥 3,223벌 중 2,303벌 등록 가능. 7벌을 로컬에 등록해 실측값으로 리포트 확인

**막힘 진단 / Blockers**

| 막힘 | 영향 | 대응 |
|---|---|---|
| AI Hub 이용정책: 승인 없는 열람·제공 금지, 학습용만 | 변환 의류를 공개 저장소에 못 올림 | 로컬 폴더 + `FIT_GARMENTS_DIR`, `.gitignore`로 막음. 필요한 사람은 각자 신청. 교수님께 공개·데모 사용 범위 확인 |
| AI Hub 데이터 설명서 미확인 | 가슴·허리 둘레/단면, 앞길이 기준점(HPS) 불확실 → 총장 비움 | 값 범위로 둘레 판단([제안]). 설명서 받아 확인 후 다시 등록 |
| 어깨 기준표 경계 | 몸 어깨(직선, SMPL 평균 38~40 cm)와 옷 어깨(43~50 cm) 차이로 대부분 "어깨선 내려옴" | C3에서 AI Hub 실측을 보고 `fit_rules_v1`로 조정 |
| 총장 판정 불가 | 입력에 몸통 길이가 없음 | 판정 안 함 + 실측만 표시. 추정(키 × 비율) 사용 여부는 A3 전에 결정 |
| SMPL 라이선스 1인용 | 몸 메시 공유 애매 | 쓰는 사람이 각자 가입해 `tools.smpl_body`로 생성 |
| D1 테스트가 한글 사용자 경로에서 실패 | 로컬 pytest 1개 실패(CI는 통과) | 테스트에서 `cv2.imdecode(np.fromfile(...))`로 읽게 수정 |
| PR이 merge commit으로 머지됨 | main 히스토리가 규칙과 다름 | 저장소 설정을 squash merge만 허용으로 변경 |

**데모 시나리오 초안**
1. 입력 화면: "재는 법"(measure_v1), 범위 밖 값 오류
2. 티셔츠 선택: 실측 표시, 외부 전송 동의 체크
3. 결과: 핏 리포트 즉시(가슴 +8 cm 기본 여유, 어깨 +2 cm 기본 범위, 소매·총장 판정 안 함) → 몸 → 착장 → 렌더 → 실사화 → 검증 진행
4. 다른 경우: 작은 옷(여유 부족 예상), 동의 안 함(실사화 건너뜀), 실사화 실패(`?mockFail=realize`)여도 핏 리포트·3D 유지
5. 실제 서버 모드: AI Hub 실측 티셔츠로 같은 흐름

- [o] 이번 활동에서 AI를 사용했다면 `PROMPTS.md`에 기록했습니다 / Logged any AI use in `PROMPTS.md`

---

> 수업 종료 시 커밋하세요 / Commit this at the end of class
> `git add docs/week-10.md && git commit -m "docs: 10주차 활동지 작성"`
