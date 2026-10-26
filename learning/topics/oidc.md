# OIDC (OpenID Connect)

<!--
새 주제 파일 템플릿. `learning/topics/<slug>.md`로 복사해 쓴다.
- <...> 자리표시자를 채우고, 아직 내용이 없는 섹션은 "(아직 없음)"으로 둔다.
- 실제 대화에서 일어난 일만 기록한다. 날짜는 YYYY-MM-DD.
- "학습함"은 이해의 증거가 아니다. "이해가 확인된 내용"에는 근거가 반드시 있어야 한다.
- 이 HTML 주석들은 지우지 말고 남겨 둔다(이후 업데이트의 형식 안내 역할).
-->

- slug: `oidc`
- 시작일: 2026-10-26
- 마지막 학습: 2026-10-26
- 상태: 학습 중
- 학습 목표: GitLab runner(CI/CD)를 OIDC 방식으로 전환하는 작업에 앞서 OIDC를 정확히 이해하기

## 학습한 개념 (Concepts studied)

<!-- 세션에서 실제로 다룬 개념. 형식: - YYYY-MM-DD — 개념: 한 줄 요약 -->

- 2026-10-26 — 큰 그림: OIDC = OAuth 2.0 위의 identity layer, AuthN vs AuthZ, access token vs ID token
- 2026-10-26 — CI/CD에서 쓰는 OIDC 범위: ID token + discovery + JWKS만 사용, 브라우저 flow(auth code, PKCE, nonce)는 해당 없음
- 2026-10-26 — 역할 매핑: GitLab 인스턴스 = issuer(서명), runner = 전달자, 클라우드 STS = relying party(검증·교환), 클라우드 API = 임시 자격증명만 봄
- 2026-10-26 — ID token 검증 흐름: iss → discovery → jwks_uri → kid로 공개키 선택 → 서명·iss·aud·exp 검증 → trust 조건(sub) 매칭
- 2026-10-26 — 왜 비대칭(RS256)이어야 하는가: 제3자 검증자가 위조할 수 없어야 함
- 2026-10-26 — OIDC가 OAuth 2.0에 더한 것: client가 수신자(aud)인 별도 토큰, JWT·서명 의무화, sub 의미 규정(iss+sub 불변 식별자), discovery/JWKS 표준화, 검증 규칙, nonce 등 브라우저 flow 장치
- 2026-10-26 — "OAuth로 로그인" 공격(access token 재사용)이 ID token의 aud로 막히는 이유
- 2026-10-26 — hash vs signature: hash만으로는 위조 방지가 안 됨 (PyJWT 예제를 실제로 실행함)
- 2026-10-26 — EC2 runner로 S3 업로드하는 전체 흐름(ASCII 그림): 사전 설정, job 실행 시 토큰 발급, STS 교환, S3 PutObject. EC2 instance profile이 OIDC를 우회하는 경로가 될 수 있음
- 2026-10-26 — AWS IAM OIDC provider의 역할: 외부 issuer를 신뢰한다는 계정 단위 등록(신뢰할 issuer, 키를 가져올 위치, 허용 aud, TLS fallback thumbprint). 토큰을 발급하지 않음. 권한은 role이 정함. provider 1개가 role N개와 연결됨
- 2026-10-26 — 서명이 막는 것과 못 막는 것: 변조는 막지만 원본 토큰 재사용(replay)은 exp 전까지 못 막음
- 2026-10-26 — AWS 자격증명 탐색 순서(환경 변수 → web identity → ... → instance profile)와 `aws sts get-caller-identity`로 실제 신원 확인하기
- 2026-10-26 — 신뢰 사슬: 미리 등록한 iss URL → HTTPS(TLS) discovery → jwks_uri → kid → 서명. 일반 OIDC에서는 token endpoint TLS나 HS256(client_secret)으로도 검증할 수 있지만, CI에서는 토큰이 job을 거쳐 오므로 JWKS 서명 검증만이 근거
- 2026-10-26 — CI 전체 흐름(AWS 예시): push → pipeline → runner가 job 수령 → GitLab이 ID token 서명·주입 → STS AssumeRoleWithWebIdentity → discovery/JWKS로 검증 → trust policy → 임시 자격증명 → SigV4로 API 호출

## 이해가 확인된 내용 (Verified understanding)

<!--
원리를 내 말로 설명했거나, 코드 결과 예측, 버그 찾기, 새 상황 적용에서
올바른 추론으로 맞힌 경우에만 추가한다. "이해했어"라는 말만으로는 추가하지 않는다.
형식: - YYYY-MM-DD — 개념 — 근거: 어떤 확인을 어떻게 통과했는지
-->

- 2026-10-26 — ID token 발급 주체는 GitLab 서버(runner 아님) — 근거: 전체 흐름을 직접 설명하며 "job이 있으면 GitLab이 private key로 ID token을 만들어 내려준다"고 서술
- 2026-10-26 — 서명 키 방향(private key로 서명, JWKS의 public key로 검증) — 근거: 같은 흐름 설명에서 올바르게 서술 (앞서 "public key로 만든다"고 했던 오해를 스스로 바로잡음)
- 2026-10-26 — 검증 주체는 STS — 근거: 두 확인 질문 답변 모두에서 STS가 JWKS를 가져와 서명을 검증한다고 스스로 서술
- 2026-10-26 — 변조된 토큰은 STS의 서명 검증 단계에서 차단됨 — 근거: "ref:main → ref:feature 변조" 질문에 "JWT가 바뀌었으니 STS가 public key로 서명을 검증할 때 막힌다"고 답함. 재서명에 private key가 필요하다는 점은 명시하지 않았지만 추론은 맞음
- 2026-10-26 — 검증받는 쪽(job)의 자기 검증은 보안 통제가 아님 — 근거: "AWS가 검증하지 않고 job의 JSON을 믿는다면?" 질문에 "job이 거짓으로 검증했다고 하면 무조건 통과하니 위험하다"고 답함. 구체적인 공격 경로(검증 단계 삭제, job 밖에서 직접 STS 호출)는 설명하지 않음
- 2026-10-26 — ID token은 암호화되지 않아 STS가 iss를 읽고 well-known → JWKS로 public key를 가져옴 — 근거: 두 번째 전체 흐름 설명에서 스스로 서술 ("ID token은 암호화돼 있지 않아 누구든 iss를 볼 수 있다")
- 2026-10-26 — runner가 job을 실제로 실행함(Docker executor로 실행 준비) — 근거: 두 번째 흐름 설명
- 2026-10-26 — EC2 runner → S3 업로드 OIDC 전체 흐름 — 근거: 세 번째 흐름 설명에서 8단계를 모두 올바르게 서술(polling, GitLab의 private key 서명, runner의 Docker 실행, script가 ID token으로 AssumeRoleWithWebIdentity 호출, STS의 iss→JWKS 검증과 claim 검사, 임시 자격증명 발급, 환경 변수 저장, S3 업로드). 빠진 세부: exp 검사, sub는 role trust policy가 검사, S3 단계의 permission policy 검사

## 아직 이해가 부족한 개념 (Gaps and misconceptions)

<!--
틀렸거나 흔들린 답, 드러난 오해, 내가 모르겠다고 한 것.
형식: - 개념 — 부분 이해/부족 — 구체적인 오해나 빠진 부분 (발견: YYYY-MM-DD)
나중에 확인되면 지우지 말고 "→ YYYY-MM-DD 확인됨"을 덧붙인다.
-->

- OIDC와 OAuth 2.0의 차이 — 부분 이해 — "기존 OAuth에 iss/aud/sub만 강제한 것"이라고 봄. 빠진 부분: access token에 claim을 추가한 게 아니라 client가 수신자인 별도 토큰이라는 점, sub의 의미 규정, discovery/JWKS로 키를 배포하는 방식 (발견: 2026-10-26) → 2026-10-26 다시 정리: "JWT + sub/iss/aud 강제 + JWKS 공개키". CI 관점에서는 거의 맞음. 남은 보정: claim은 있는 것만이 아니라 의미와 RP의 검증 의무까지 정해짐, JWKS는 교환이 아니라 게시이며 신뢰의 뿌리는 iss URL + TLS
- JWT 검증 방식 — 표현 확인 필요 — "JWT의 해시를 이용한 검증"이라고 표현함. 실제 검증 근거는 private key 서명. 본인 이해인지 단순 표현 문제인지 아직 미확인 (발견: 2026-10-26)
- 서명 키의 방향 — 부분 이해 — "GitLab이 public key로 JWT를 만든다"고 말함. 실제로는 GitLab이 private key로 서명하고 검증자(AWS)가 public key로 검증함. 같은 답에서 "암호화가 아니라 변조되지 않았고 해당 키로 만들어졌음을 확인하는 과정"이라는 부분은 맞았으나, "내 키"라고 해 키 소유자(GitLab)와 검증자를 혼동함 (발견: 2026-10-26) → 2026-10-26 키 방향은 확인됨
- 검증 주체 — 부족 — job이 JWKS로 토큰을 검증하고 그 결과를 STS에 보낸다고 이해함. 실제로는 job은 원본 토큰을 STS에 제출만 하고 iss, JWKS, 서명, aud, exp, sub 검사는 모두 STS가 함. 빠진 원리: 신뢰를 결정하는 쪽이 직접 검증해야 함 (발견: 2026-10-26) → 2026-10-26 확인됨
- 서명 검증 vs 복호화 — 부분 이해 — "public key로 ID token을 복호화해 본다"는 표현이 다시 나옴(본인도 "복호화니 signature 비교니"로 헷갈려 함). payload는 키 없이도 읽히고, public key는 서명 검증에만 쓰임 (발견: 2026-10-26) → 2026-10-26 이후 답변에서는 "서명 검증"으로 올바르게 표현함(완전 확인은 아님) → 2026-10-26 확인됨
- runner의 역할 — 부분 이해 — "runner는 중계만 하고 실행하지 않는다"고 함. 토큰에 대해서는 전달만 하는 게 맞지만, job을 실제로 실행하는 주체는 runner(Docker executor로 컨테이너를 만들고 script를 돌림) (발견: 2026-10-26) → 2026-10-26 확인됨
- STS 호출 주체와 시점 — 부분 이해 — "실행 전에 ID token을 STS에 주고 검증을 맡긴다"고 함. 실제로는 runner나 GitLab이 자동으로 호출하지 않고, job이 실행되는 중에 script(aws CLI/SDK)가 직접 호출함. 그 줄이 없으면 STS 교환은 일어나지 않음 (발견: 2026-10-26) → 2026-10-26 본인 질문("assume-role-with-web-identity에 job이 가져온 ID token을 넣어 호출하는 거야?")으로 이해를 보임. 확인 질문으로는 아직 미확인 → 2026-10-26 확인됨(세 번째 흐름 설명 4단계)
- 용어: STS vs 임시 자격증명 — 부분 이해 — "job에서 sts를 준다/받는다"고 표현함. STS는 서비스 이름이고, 받는 것은 임시 자격증명(AccessKeyId, SecretAccessKey, SessionToken) (발견: 2026-10-26) → 2026-10-26 확인됨("STS가 임시 출입증을 발급"으로 올바르게 구분)
- instance profile 위험의 본질 — 부분 이해 — "script에서 STS 호출을 빠뜨리면?" 질문에 (a) 실패, (b) 성공은 맞혔지만, 위험을 "profile에 있는 곳으로 올라간다(목적지가 바뀐다)"로 봄. 실제로 목적지는 명령에 쓴 경로 그대로이고 바뀌는 건 신원(instance role)이라, 모든 프로젝트와 branch가 sub 조건 없이 같은 권한을 씀. 대응책으로 든 "환경 변수의 AWS key 삭제"는 IMDS로 받는 instance profile 자격증명은 막지 못함 (발견: 2026-10-26)
- assume-role-with-web-identity CLI 결과의 적용 방식 — 부분 이해 — 호출만 하면 다음 줄의 S3 접근이 자동으로 된다고 봄. CLI는 자격증명을 출력만 하므로 export하거나, AWS_ROLE_ARN과 AWS_WEB_IDENTITY_TOKEN_FILE을 설정해 SDK가 자동 교환하게 해야 함 (발견: 2026-10-26) → 2026-10-26 확인됨(세 번째 흐름 설명 7단계: 환경 변수로 저장)
- job 안 사전 검증의 보안 가치 — 부분 이해 — "보안이 강해지지 않는다"는 결론은 맞았지만 근거를 "STS가 이미 하니 중복이고 느려진다"로 댐. 빠진 원리: 검증받는 쪽(job)의 코드는 공격자가 통제할 수 있어 보안 통제가 될 수 없음. 속도 차이는 핵심이 아님 (발견: 2026-10-26) → 2026-10-26 확인됨
- 탈취된 토큰을 변조하지 않고 재사용하는 경우 — 미답변 — 진짜 서명이라 STS의 모든 검사를 통과함. 방어는 masked 변수, 짧은 exp, 좁은 aud와 sub, 최소 권한 (발견: 2026-10-26)

## 해결되지 않은 질문 (Open questions)

<!--
형식: - [ ] 질문 (YYYY-MM-DD)
해결되면: - [x] 질문 → 짧은 답 (YYYY-MM-DD)
-->

- [ ] 회사 GitLab은 gitlab.com인가 self-managed인가? 버전은? discovery/JWKS가 공개 인터넷에서 접근 가능한가? (2026-10-26)
- [ ] 대상이 AWS / GCP / Azure / Vault 중 무엇인가? (2026-10-26)
- [ ] "runner를 OIDC로 전환"이 job 단위 ID token(`id_tokens`)인가, runner 인프라 identity(IRSA, instance profile)인가? (2026-10-26)

## 선수 지식과 개념 연결 (Prerequisites and connections)

<!--
- 선수 지식: 개념 — 확인됨 / 기록 없음 / 부족
- C#/.NET 대응: 개념 ↔ C# 대응 — 같은 점 / 다른 점(비유가 깨지는 지점)
- 관련 주제: `other-slug` — 관계
-->

- 선수 지식: JWT, 서명, 해싱, 대칭/비대칭키 — 기록 없음 (본인 언급, 2026-10-26 미검증)
- 선수 지식: OAuth 2.0 역할과 토큰 종류, AuthN vs AuthZ — 기록 없음
- 선수 지식: 대상 클라우드 STS와 trust policy — 기록 없음 (대상 미정)
- C#/.NET 대응: `AddJwtBearer`(Authority, Audience, TokenValidationParameters) ↔ 클라우드 STS의 ID token 검증 — 같은 점: discovery·JWKS 캐시·kid refresh / 다른 점: STS는 검증 후 임시 자격증명으로 교환, API가 토큰을 직접 쓰지 않음
- C#/.NET 대응: `[Authorize(Policy)]` + RequireClaim ↔ trust policy의 `sub` 조건 — authN과 authZ 분리
- C#/.NET 대응: Azure Managed Identity ↔ runner 호스트 identity — 다른 점: 호스트 전체 단위, job/branch 단위가 아님

## 다음에 공부할 내용 (Next steps)

<!-- 형식: 1. 내용 — 이유 (필수/선택) -->

1. JWT 자기 보고 검증 + 1단계 이해 확인 (역할 매핑 설명) — 필수 → 2026-10-26 1단계는 전체 흐름 설명으로 확인됨, 다음 세션은 2번 또는 범위 확정 후 3·4번
2. ID token 검증 실습: gitlab.com discovery/JWKS 조회, Python PyJWT로 검증, 변조 케이스 예측 — 필수
3. GitLab `id_tokens` 세부: claim, `sub` 형식, 수명, CI_JOB_JWT 제거(17.0) — 필수
4. 대상 클라우드 하나의 trust 설정과 토큰 교환 — 필수
5. 보안 설계와 전환 계획: protected branch/environment, 최소 권한, self-managed 네트워크 도달성 — 필수
6. 심화: OAuth 브라우저 flow, RFC 8693, k8s SA token federation — 선택
