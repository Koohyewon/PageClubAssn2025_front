# Page — 성결대 동아리연합회 협업 플랫폼

## 프로젝트 개요

| 항목 | 내용 |
|------|------|
| 프로젝트명 | Page (페이지) |
| 소속 | 성결대학교 동아리연합회 |
| 유형 | 팀 프로젝트 (프론트엔드) |
| 서비스 내용 | 동아리 물품 대여·예약, 강의실 조회, 동아리원증 발급, 활동 점수 관리 |
| API 서버 | `https://pageback.sku-sku.com` |

동아리 활동에 필요한 물품을 온라인으로 예약하고, 동아리원 신분증을 디지털로 발급받으며, 관리자가 동아리와 회원 정보를 통합 관리할 수 있는 웹 서비스입니다. 사용자용 인터페이스와 관리자용 인터페이스를 분리하여 역할 기반으로 기능을 제공합니다.

---

## 기술 스택

| 분류 | 기술 |
|------|------|
| 프레임워크 | React 19 |
| 라우팅 | React Router DOM 7 |
| HTTP 클라이언트 | Axios |
| 인증 | JWT (localStorage 저장, jwt-decode로 만료 검증) |
| 스타일링 | Tailwind CSS |
| 상태 관리 | React 내장 `useState` / `useEffect` |
| 기타 | react-icons, react-modal, Lottie (dotlottie-react) |

별도의 전역 상태 관리 라이브러리 없이 컴포넌트 단위의 로컬 상태와 Axios 호출을 조합하여 데이터 흐름을 관리했습니다.

---

## 전체 기능 요약

**사용자 영역**

- 로그인 / 개인정보 동의 / 로그인 실패 모달
- 마이페이지 (동아리 정보 조회·변경)
- 동아리원증 페이지 (디지털 신분증)
- 물품 예약 현황 / 물품 대여 현황
- 활동 점수 조회 (전체 순위)
- 강의실 조회

**관리자 영역**

- 동아리원 관리 (추가·삭제, 엑셀 일괄 업로드)
- 등록 동아리 관리 (추가·수정·삭제)
- 활동 점수 관리 (전체 동아리 점수 등록·수정·삭제)
- 물품 관리 (추가·수정·삭제)
- 물품 예약·대여 현황 관리 (수령 처리·반납 처리·취소)

---

## 내가 담당한 기능

### 1. 인증 및 온보딩

**로그인 페이지** (`src/pages/User/Login/Login.jsx`)

학번과 이름을 입력받아 `/login` API를 호출하고, 응답 상태 코드에 따라 세 가지 흐름을 분기했습니다. 성공 시 JWT 토큰과 역할(`role`)을 `localStorage`에 저장하고 홈으로 이동합니다. 400 오류(등록되지 않은 사용자)는 로그인 실패 모달을 표시하고, 401 오류(개인정보 미동의)는 개인정보 동의 페이지로 이동하면서 입력한 학생 정보를 `state`로 전달합니다.

**개인정보 동의 페이지** (`src/pages/User/Login/UserAgreement.jsx`)

수집 이용 항목·수집 이용 목적·보유 기간 세 가지 항목에 대한 체크박스를 독립적으로 관리하고, 전체 동의 체크박스 클릭 시 세 항목이 일괄 전환됩니다. 개별 항목을 모두 선택했을 때 전체 동의가 자동으로 활성화되며, 모든 항목에 동의해야 확인 버튼이 활성화됩니다. 각 항목은 펼침·접힘 토글로 상세 내용을 확인할 수 있습니다. 확인 버튼 클릭 시 `/agree` API를 호출해 토큰을 발급받고 서비스에 진입합니다.

**로그인 실패 모달** (`src/pages/User/Login/LoginFailureModal.jsx`)

`react-modal` 기반으로 구현했습니다. 로그인 시도에서 400 응답이 오면 부모 컴포넌트가 `showModal` 상태를 `true`로 전환해 모달을 표시하고, 입력 필드를 초기화합니다.

---

### 2. 마이페이지 및 동아리원증

**마이페이지** (`src/pages/User/Login/MyPage.jsx`)

`/mypage`와 `/joined-list` 두 API를 `Promise.all`로 병렬 호출하여 현재 사용자 정보와 가입된 동아리 목록을 동시에 받아옵니다. 서버에서 전달되는 로고 이미지는 Base64 문자열로 제공되며, `data:image/png;base64,` 접두사가 없는 경우 자동으로 변환해 표시합니다. 사용자가 동아리를 선택하면 `/changeIconClub` API에 해당 동아리의 고유 `clubId`를 전달하고, 로컬 상태도 `clubId·clubName·logo`를 함께 업데이트합니다.

**동아리원증 페이지** (`src/pages/User/ClubMemberCard.jsx`)

마이페이지와 동일한 API 구조로 데이터를 조회하고, 성결대 동아리연합회 디자인 규격에 맞는 카드 형태로 렌더링합니다. 동아리 로고, 동아리명, 사용자 이름을 조합하여 "위 동아리원증은 성결대학교 동아리원임을 증명합니다" 문구와 함께 표시합니다.

---

### 3. 물품 예약·대여 현황

**물품 예약 현황** (`src/pages/User/ClickStatus/Reservation.jsx`)

`/item-rent/book-list` API로 현재 예약 중인 물품 목록을 받아옵니다. 여러 건의 예약이 있을 때를 대비하여 캐러셀(carousel) 방식으로 한 건씩 표시하며, 이전·다음 버튼과 `현재 번호/전체 수` 형태의 페이지 표시를 제공합니다. 예약 취소는 현재 표시 중인 항목의 `itemRentId`를 DELETE 요청으로 전달하며, 성공 후 해당 항목을 목록에서 제거하고 인덱스를 보정합니다.

**물품 대여 현황** (`src/pages/User/ClickStatus/RentalClick.jsx`)

예약 현황과 동일한 캐러셀 구조에 연체 여부 표시를 추가했습니다. 현재 시각이 반납 기한을 초과한 경우 "연체" 텍스트를 강조색으로 표시하고, 정상 대여 중이면 대여일과 반납 기한을 함께 표시합니다.

---

### 4. 관리자 — 동아리원 관리

**동아리원 조회·삭제** (`src/pages/Admin/ClubMember/ClubMember.jsx`)

키워드 검색으로 회원을 조회(`/admin/join-club/search`)하고, 삭제 확인 모달을 거쳐 `/admin/join-club` DELETE 요청을 보냅니다. 삭제 성공 후 목록은 `studentId`를 기준으로 필터링하여 UI에서 즉시 제거합니다.

**회원 개별 추가** (`src/pages/Admin/ClubMember/AddMember.jsx`)

학번·이름·역할(`ROLE_ADMIN` / `ROLE_MEMBER`)을 입력받아 추가합니다. 추가 전 `/admin/member/find` API로 중복 여부를 먼저 확인하고, 이미 등록된 경우 사용자에게 안내 메시지를 표시합니다.

**동아리별 회원 추가 및 엑셀 업로드** (`src/pages/Admin/ClubMember/AddClubMember.jsx`)

특정 동아리에 여러 회원을 한 번에 추가할 수 있습니다. 선택된 모든 회원에 대해 `/admin/join-club/add`를 `Promise.all`로 병렬 호출합니다. 또한 `.xlsx` / `.xls` 형식의 파일을 검증한 뒤 `FormData`로 `/admin/excel/upload`에 전송하는 엑셀 일괄 업로드 기능도 구현했습니다.

---

### 5. 관리자 — 등록 동아리 관리

**목록 조회 및 삭제** (`src/pages/Admin/ClubManagement/ClubManagement.jsx`)

`/admin/club/all`에서 전체 동아리 목록을 불러와 `club.id`를 `key`로 렌더링합니다. 삭제 시 확인 모달을 거쳐 `{ id: selectedClub.id }`를 DELETE 요청의 `data`로 전달하고, 완료 후 목록을 재조회합니다.

**동아리 추가** (`src/pages/Admin/ClubManagement/ClubAdd.jsx`)

동아리명과 로고 이미지를 `FormData`로 묶어 `/admin/club/add`에 POST 요청을 보냅니다. 이미지 선택 시 `URL.createObjectURL`로 미리보기를 즉시 표시합니다.

**동아리 수정** (`src/pages/Admin/ClubManagement/ClubEdit.jsx`)

기존 정보를 초기값으로 채운 폼을 제공하며, 수정할 내용을 `FormData`(id, name, logo)로 묶어 PUT 요청을 보냅니다. 로고가 서버에서 Base64로 제공될 경우 `data:image/png;base64,` 접두사를 붙여 `<img>`에 표시합니다.

---

### 6. 관리자 — 활동 점수 관리

**점수 목록 조회** (`src/pages/Admin/ClubScore/ClubScoreManagement.jsx`)

`/admin/club-scores/all`에서 전체 동아리 점수를 받아 동점 처리를 포함한 순위 계산을 클라이언트에서 수행합니다. 점수가 같은 동아리에는 동일 순위를 부여하고, 그 다음 순위는 공동 수만큼 건너뛰어 매깁니다. 계산된 전체 목록을 `scores.map()`으로 렌더링합니다.

**점수 등록·수정·삭제** (`src/pages/Admin/ClubScore/UpdateClubScore.jsx`)

동아리별 점수 항목을 추가·수정·삭제할 수 있는 폼입니다. 각 항목은 `{ clubId, clubName, score }` 구조로 상태를 관리하며, 삭제 시 해당 항목의 `clubId`로 `/admin/club-scores/delete/{clubId}` DELETE 요청을 보냅니다. 저장 시에는 `/admin/club-scores/add-or-update`에 분기 처리를 서버에 위임합니다.

---

## 문제 해결 경험

### 1. 인덱스 기반 상태 관리로 인한 데이터 정합성 문제

활동 점수 수정 페이지(`UpdateClubScore.jsx`)를 개발하면서 목록 항목의 식별 방식이 데이터 정합성에 미치는 영향을 직접 경험했습니다.

**문제 상황:** 초기 구현에서 각 점수 항목은 `scores` 배열의 인덱스만으로 식별되었습니다. 예를 들어 3개의 항목 중 2번째를 삭제하면 배열이 재인덱싱되어 기존 3번째 항목이 2번째 위치를 차지합니다. 이 상태에서 "2번째 항목을 삭제하라"는 API 요청을 보내면 원래 의도한 항목이 아닌 다른 항목이 삭제되는 정합성 오류가 발생했습니다.

**개선 방향:** 각 항목이 배열 내 위치와 무관하게 자신의 실체를 항상 알 수 있도록, `scores` 상태의 객체 구조를 `{ clubId, clubName, score }`로 정의해 서버 식별자인 `clubId`를 항목 자체에 포함시켰습니다.

**구체적인 적용:** `handleRemoveClub(index)` 함수는 `const scoreItem = scores[index]`로 삭제할 항목의 정보를 먼저 캡처한 뒤, `filter`로 배열에서 제거합니다. 이후 API 호출은 배열 상태와 무관하게 캡처된 `scoreItem.clubId`를 사용합니다.

```javascript
// src/pages/Admin/ClubScore/UpdateClubScore.jsx (124~155행)
const handleRemoveClub = async (index) => {
  const scoreItem = scores[index]; // 먼저 항목 정보 캡처

  // UI 즉시 반영 (배열 재인덱싱 발생)
  setScores((prevScores) => prevScores.filter((_, idx) => idx !== index));

  // clubId로 API 호출 — 재인덱싱의 영향을 받지 않음
  if (scoreItem.clubId) {
    await axios.delete(
      `${API_URL}/admin/club-scores/delete/${scoreItem.clubId}`,
      ...
    );
  }
};
```

동아리원 관리(`ClubMember.jsx`)도 같은 원칙을 적용했습니다. 삭제 후 목록 갱신 시 `prevResults.filter((result) => result.studentId !== selectedMember.studentId)`처럼 고유 식별자(`studentId`) 기반으로 필터링하여 인덱스 이동에 의한 오류를 방지했습니다.

이 경험을 통해 배열을 UI 표시 순서가 아니라 "어떤 항목인지"를 식별하는 수단으로 다룰 때는, 인덱스가 아닌 서버 ID를 항목에 포함해 안정적인 식별자로 활용해야 한다는 것을 체득했습니다.

---

### 2. 디자인 시안에 드러나지 않은 요구사항 검증 — 활동 점수 관리 전체 목록 표시

**상황:** 활동 점수 관리 페이지를 개발할 때, 받은 디자인 시안에는 1·2·3위 세 개의 동아리만 표시되어 있었습니다. 시안대로라면 상위 3개 항목만 렌더링하면 충분했고, 처음에는 그렇게 구현했습니다.

**확인 과정:** 구현 후 기능을 검토하는 과정에서 "이 페이지가 왜 관리자에게 필요한가"라는 질문을 스스로 던졌습니다. 관리자가 활동 점수를 관리하는 목적은 단순한 시상 결과 확인이 아니라 전체 동아리의 점수 현황을 파악하고 수정하기 위한 것이었습니다. 상위 3개만 보여 준다면 나머지 동아리의 점수는 화면에서 아예 확인할 수 없는 상태가 됩니다. 팀원에게 이 점을 확인했고, 전체 동아리를 표시해야 한다는 사실을 확인했습니다.

**수정 내용:** 하드코딩된 3개 항목 렌더링 대신, API 응답 전체를 `scores.map()`으로 동적 렌더링하도록 수정했습니다. 동점 처리를 포함한 순위 계산도 서버 응답의 전체 배열을 순회하여 수행하므로, 동아리 수에 관계없이 올바르게 동작합니다.

```javascript
// src/pages/Admin/ClubScore/ClubScoreManagement.jsx (94~124행)
{scores.map((item, index) => (
  <div key={`${item.ranking}-${item.clubName}-${index}`}>
    <p>{item.ranking}등</p>
    <p>{item.clubName}</p>
    <p>{item.score}점</p>
  </div>
))}
```

이 경험은 디자인 시안이 기능의 완전한 명세가 아닐 수 있다는 점을 상기시켜 주었습니다. 특히 관리자 기능처럼 운영 목적이 뚜렷한 화면은 시안의 시각적 표현 너머에 있는 실제 사용 목적을 함께 검토해야 한다는 것을 배웠습니다.

---

## 배운 점 / 성과

**담당 범위와 일관성 유지**
인증부터 관리자 기능까지 서비스의 여러 레이어를 담당하면서, 반복되는 API 호출 패턴(토큰 헤더 첨부, Base64 이미지 변환, 에러 응답 분기)을 컴포넌트마다 일관되게 적용하는 경험을 쌓았습니다.

**상태와 서버 데이터의 동기화**
삭제·추가 후 서버 재조회와 로컬 상태 업데이트 중 어느 방식이 더 적절한지 맥락에 따라 판단했습니다. 목록이 단순한 경우 로컬 필터링으로 즉시 반영하고, 순서나 집계가 중요한 경우 재조회를 선택했습니다.

**요구사항의 명확화**
디자인 시안이 모든 요구사항을 표현하지 못할 수 있다는 점을 실무적으로 경험했습니다. 구현하기 전에 "이 기능의 목적이 무엇인가"를 확인하는 습관이 오류를 예방하는 데 효과적이라는 것을 배웠습니다.

**역할 기반 라우팅 이해**
JWT의 만료 시각(`exp`)을 클라이언트에서 검증하고, `role` 값에 따라 사용자 라우트와 관리자 라우트를 분리하는 보호 라우트 패턴(`ProtectedRoute` / `PrivateRoute`)을 직접 구현하면서 토큰 기반 인증 흐름 전반을 이해했습니다.
