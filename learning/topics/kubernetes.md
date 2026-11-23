# Kubernetes

<!--
새 주제 파일 템플릿. `learning/topics/<slug>.md`로 복사해 쓴다.
- <...> 자리표시자를 채우고, 아직 내용이 없는 섹션은 "(아직 없음)"으로 둔다.
- 실제 대화에서 일어난 일만 기록한다. 날짜는 YYYY-MM-DD.
- "학습함"은 이해의 증거가 아니다. "이해가 확인된 내용"에는 근거가 반드시 있어야 한다.
- 이 HTML 주석들은 지우지 말고 남겨 둔다(이후 업데이트의 형식 안내 역할).
-->

- slug: `kubernetes`
- 시작일: 2026-10-23
- 마지막 학습: 2026-11-23
- 상태: 학습 중
- 학습 목표: 쿠버네티스를 공부하고 싶다. 먼저 기초 개념부터 진단 인터뷰로 현재 수준을 파악해 시작점을 찾는다 (구체 목표는 인터뷰 중 정함)

## 학습한 개념 (Concepts studied)

<!-- 세션에서 실제로 다룬 개념. 형식: - YYYY-MM-DD — 개념: 한 줄 요약 -->

- 2026-10-23 — 진단 인터뷰 시작: Docker 실무 경험 확인(Dockerfile, compose, ECR→ECS CI/CD 배포). 컨테이너 vs VM 질문에서 "커널 공유" 개념이 빠진 것 발견.
- 2026-10-23 — 컨테이너의 실체: 호스트 커널을 공유하는 보통 프로세스 + namespace(보이는 범위 제한). VM은 별도 커널. `unshare --pid`로 실행해 확인함(호스트 ps에는 보이고, namespace 안에서는 PID 1로 보임).
- 2026-11-23 — 진단 인터뷰(k8s 개념 층): Pod, Node, Deployment/ReplicaSet, Service, kubelet을 본인 말로 설명. 용어와 전체 구조는 이미 알고 있음. 질문은 kubectl이었는데 kubelet을 답함 → kubectl(API server에 요청하는 CLI 클라이언트) vs kubelet(node agent) 구분을 짚음.
- 2026-11-23 — 진단 인터뷰(원리 층): `kubectl delete pod` 시나리오 예측. kubectl→API server(HTTP)→etcd→kubelet이 삭제→ReplicaSet이 수를 맞추려 새 Pod 생성→kubelet이 실행, 으로 설명. 제어 루프(desired state로 수렴) 직관 있음. 빠진 것: scheduler 단계, "etcd는 API server만 접근하고 나머지는 API server를 watch"라는 구조.

## 이해가 확인된 내용 (Verified understanding)

<!--
원리를 내 말로 설명했거나, 코드 결과 예측, 버그 찾기, 새 상황 적용에서
올바른 추론으로 맞힌 경우에만 추가한다. "이해했어"라는 말만으로는 추가하지 않는다.
형식: - YYYY-MM-DD — 개념 — 근거: 어떤 확인을 어떻게 통과했는지
-->

- 2026-11-23 — Node의 역할 — 근거: 진단 인터뷰에서 "EC2 등이 node가 되고, kubelet이 control plane과 통신해 pod를 배치한다"고 본인 말로 설명 (역할 설명 수준; 적용 확인은 아직)
- 2026-11-23 — kubelet의 역할 — 근거: "node에 있으며 control plane과 소통해 배치할 pod가 있으면 띄우고 pod 상태를 계속 보고한다"고 본인 말로 설명 (역할 설명 수준; 적용 확인은 아직)
- 2026-11-23 — ReplicaSet의 reconciliation(삭제된 Pod를 누가, 왜 대체하는가) — 근거: `kubectl delete pod` 시나리오에서 "replicaset이 원하는 수의 파드를 띄우기 위해 다시 쓴다"고 행위 주체와 이유를 맞게 예측. Deployment를 끌어들이지 않은 것도 맞음
- 2026-11-23 — kubectl → API server(HTTP) → etcd 저장 → kubelet 실행 흐름 — 근거: 같은 시나리오에서 순서대로 설명(단, 아래 "통신 구조" gap 참고)

## 아직 이해가 부족한 개념 (Gaps and misconceptions)

<!--
틀렸거나 흔들린 답, 드러난 오해, 내가 모르겠다고 한 것.
형식: - 개념 — 부분 이해/부족 — 구체적인 오해나 빠진 부분 (발견: YYYY-MM-DD)
나중에 확인되면 지우지 말고 "→ YYYY-MM-DD 확인됨"을 덧붙인다.
-->

- 컨테이너의 실체(호스트 커널 공유, PID namespace) — 부족 — 컨테이너/VM 차이를 "자원 공유"로 설명(VM도 자원은 공유하므로 구분이 안 됨). 호스트 `ps`에서 컨테이너 프로세스가 보이는지 예측 못 함("잘 모르겠어"). 설명은 들었으나 아직 확인 전 (발견: 2026-10-23)
- Pod가 왜 "컨테이너"가 아니라 별도 단위인가 — 부분 이해 — Pod를 "node에 배치되는 프로세스, 이미지가 올라가는 곳"으로 설명(컨테이너 1개인 경우엔 실용적으로 맞음). 여러 컨테이너가 network namespace(같은 IP, localhost)와 volume을 공유하는 단위라는 점과 그 단위가 존재하는 이유가 빠짐 (발견: 2026-11-23)
- Deployment가 ReplicaSet 위에 따로 존재하는 이유 — 부분 이해 — "ReplicaSet으로 pod 수를 보장하는 장치"로 설명. 수 보장은 ReplicaSet의 일이고 Deployment 고유 역할(rollout: 새 ReplicaSet 생성·점진 전환·rollback·history)이 빠짐. ECS Service는 이 둘을 한 객체가 하므로 분리 이유가 안 보였을 수 있음 (발견: 2026-11-23)
- Service의 존재 이유 — 부족 — "pod에 네트워크를 붙이는 기능"으로 설명. Pod는 이미 IP를 가짐; Pod IP가 수시로 바뀌기 때문에 필요한 고정 이름/가상 IP + label selector 기반 load balancing이라는 "왜"가 빠짐 (발견: 2026-11-23)
- scheduler의 역할 — 부족 — Pod 삭제/재생성 흐름에서 scheduler를 빠뜨림. ReplicaSet이 만든 새 Pod는 node가 정해지지 않은 상태(nodeName 없음)이고, scheduler가 node를 골라 bind해야 그 node의 kubelet이 받음 (발견: 2026-11-23)
- 컴포넌트들의 통신 구조 — 부분 이해 — "etcd에 써지면 kubelet이 가져간다"고 설명. 실제로 etcd에 접근하는 건 API server뿐이고 kubelet·controller·scheduler는 모두 API server를 watch해 변화를 받음(API server = hub). "모든 컴포넌트는 API server를 watch하며 원하는 상태로 수렴시키는 독립 루프"라는 일반 원리를 명시적으로 알지는 못함("정확히 모르겠어") (발견: 2026-11-23)

## 해결되지 않은 질문 (Open questions)

<!--
형식: - [ ] 질문 (YYYY-MM-DD)
해결되면: - [x] 질문 → 짧은 답 (YYYY-MM-DD)
-->

- [x] 쿠버네티스 공부 전에 Linux를 먼저 공부해야 하나? → 아니오. 프로세스/namespace/cgroup 정도의 얇은 조각만 필요하며, k8s 개념이 필요로 할 때 같이 배운다 (2026-10-23)

## 선수 지식과 개념 연결 (Prerequisites and connections)

<!--
- 선수 지식: 개념 — 확인됨 / 기록 없음 / 부족
- C#/.NET 대응: 개념 ↔ C# 대응 — 같은 점 / 다른 점(비유가 깨지는 지점)
- 관련 주제: `other-slug` — 관계
-->

- 선수 지식: Docker 사용(Dockerfile, compose) — 실무 경험 있음(사용법; 원리는 미확인)
- 선수 지식: AWS ECS 배포(ECR, CI/CD) — 실무 경험 있음 → k8s 개념을 ECS 개념에 대응시키는 다리로 활용
- 선수 지식: Linux 프로세스/namespace/cgroup — 부족 (발견: 2026-10-23)
- ECS 대응(2026-11-23 제시, 다리로 쓸 것): Pod ↔ Task, Deployment(+ReplicaSet) ↔ ECS Service(desired count + rolling deploy), Node ↔ container instance(EC2)/Fargate, kubelet ↔ ECS agent, kubectl ↔ `aws ecs` CLI, Service ↔ (대략) target group + service discovery — 깨지는 지점: ECS Service 하나가 k8s의 Deployment·ReplicaSet·Service 역할을 나눠 가짐

## 다음에 공부할 내용 (Next steps)

<!-- 형식: 1. 내용 — 이유 (필수/선택) -->

1. 진단 인터뷰 완료(2026-11-23). 시작점과 첫 학습 순서 확정(learning-coach 자문 중) — 필수
