---
title: IDOR — id만 바꿔 남의 자료 보기
tags: [security, web]
---

**IDOR**(Insecure Direct Object Reference, 안전하지 않은 직접 객체 참조) = 요청에 들어 있는 **id(번호·UUID·파일명)를 바꾸기만 하면** 서버가 "이 사람이 이걸 봐도 되나"를 따지지 않고 그대로 내주는 구멍. OWASP Top 10 의 1위 항목 「Broken Access Control(접근 통제 실패)」의 가장 흔한 꼴이다.

## 왜 생기나
개발자는 보통 "화면에 안 보이니 못 누른다"고 생각한다. 그런데 주소·요청은 사람이 직접 고칠 수 있다.
- `GET /invoices/1001` 이 내 청구서면 `1002` 는 옆 사람 것. 서버가 `invoice.owner == me` 를 안 보면 그대로 나온다.
- `GET /files/report_2026_11.pdf` — 파일명만 맞히면 된다.
- `?user_id=37` 을 `38` 로 — 쿼리도 같다.
- 「로그인했느냐」(인증, authentication)는 통과했지만 「이걸 볼 자격이 있느냐」(인가, authorization)를 안 본 것. 둘은 다른 검사다.

## 코드로 보면 (FastAPI 예)

취약한 코드 — 로그인만 확인하고 주인은 안 본다.
```python
@app.get("/invoices/{invoice_id}")
def get_invoice(invoice_id: int, user=Depends(current_user)):
    return db.get(Invoice, invoice_id)
```

고친 코드 — 조회 조건에 소유 범위를 넣고, 남의 것은 "없음"으로 답한다.
```python
@app.get("/invoices/{invoice_id}")
def get_invoice(invoice_id: int, user=Depends(current_user)):
    inv = (
        db.query(Invoice)
        .filter(Invoice.id == invoice_id, Invoice.org_id == user.org_id)
        .first()
    )
    if inv is None:
        raise HTTPException(status_code=404)
    return inv
```

목록도 서버가 거른다 — 전부 주고 화면에서 거르면 IDOR 다.
```python
@app.get("/invoices")
def list_invoices(user=Depends(current_user)):
    return db.query(Invoice).filter(Invoice.org_id == user.org_id).all()
```

검사는 엔드포인트마다 손으로 쓰기보다 **한 곳**(의존성·미들웨어·조회 헬퍼)에 모아 둔다. 새 엔드포인트가 그 길을 안 거치면 바로 구멍이 된다.

## 막는 법 (원칙)
1. **모든 요청에서 서버가 소유·권한을 다시 본다.** 화면의 버튼 숨김·`isAdmin()` 분기는 편의일 뿐 울타리가 아니다.
2. 남의 것은 **403 이 아니라 404** 로 — "있다/없다"조차 알려주지 않는다(존재 여부도 정보).
3. 추측 가능한 순번(1, 2, 3…) 대신 **UUID** — 단, UUID 는 추측을 어렵게 할 뿐 1번을 대신하지 못한다. "id 를 모르면 안전"은 보안이 아니다. 통과 조건은 "id 를 알아도 안 됨"이다.
4. 목록 API 는 **보이는 범위를 서버가 거른다**(자기 조직 자료만).
5. 쓰기(수정·삭제)도 같은 검사 + 역할 검사(보기 전용 역할은 POST/PATCH/DELETE 자체를 거부).
6. 파일 다운로드·미리보기·하위 경로(`/items/{id}/file`)처럼 **곁가지 엔드포인트**도 똑같이 검사한다. 빠지기 가장 쉬운 곳이다.
7. 테스트는 **"남의 id 를 알고 있는 공격자"로 실제 요청**을 보내 본다. 코드만 읽어선 빠진 곳을 못 찾는다.

## 테스트 예 (pytest)
```python
def test_other_org_invoice_is_404(client, user_a, invoice_of_b):
    r = client.get(f"/invoices/{invoice_of_b.id}", headers=user_a.auth)
    assert r.status_code == 404

def test_own_invoice_is_200(client, user_a, invoice_of_a):
    r = client.get(f"/invoices/{invoice_of_a.id}", headers=user_a.auth)
    assert r.status_code == 200

def test_no_token_is_401(client, invoice_of_a):
    assert client.get(f"/invoices/{invoice_of_a.id}").status_code == 401
```
배포 뒤에도 같은 시나리오를 실제 서버에 돌려 본다.

## 점검 체크리스트
- [ ] 계정 A 로 B 의 id 를 넣어 조회·수정·삭제 → 전부 404(또는 403)
- [ ] 자기 것은 200
- [ ] 토큰 없음 → 401
- [ ] 보기 전용 역할로 변경 요청 → 거부
- [ ] 목록·검색·내보내기 API 에 남의 자료가 섞이지 않음
- [ ] 파일·미리보기·하위 경로도 같은 결과
- [ ] 관리자 전용 API 를 일반 계정으로 호출 → 403

## 헷갈리기 쉬운 것
- **UUID 가 주소에 보이는 것 ≠ IDOR.** id 는 이름표다. 문제는 서버가 대조를 안 하는 것이다.
- **내부 키(내부 코드·파일 경로·내부 이름)가 응답에 보이는 것은 IDOR 가 아니라 정보 노출.** 만든 방식을 드러내니 따로 숨긴다.
- **주소에 절대 넣지 않는 것** = 토큰·비밀번호·개인정보. UUID 는 넣어도 된다.
- 로그인 없이 링크만으로 여는 기능(공유 링크)을 만들면, 그때는 UUID 만으로는 안 되고 **만료되는 서명**(서명된 URL)이 필요하다.

관련: [[SSH 터널 — -L·-R·ProxyJump]] · [[브라우저 캐시와 정적 파일]]
