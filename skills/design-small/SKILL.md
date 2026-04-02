---
name: design-small
description: "웹소설의 작은 설계(25화 단위 세부 설계)를 수행하는 오케스트레이터. 세부 캐릭터 시트 + 세부 플롯 훅 가이드를 생성한다. 큰 설계 문서(부트스트랩, 캐릭터 시트, 플롯 훅 가이드)가 전제 조건. 자동 리서치(domain-researcher 서브에이전트)로 해당 아크의 세부 자료를 수집한 후 설계를 진행한다. '작은 설계', '세부 설계', '25화 설계', 'N~M화 설계', '에피소드별 플롯', '회차별 훅' 요청에 이 스킬을 사용할 것. 큰 설계(전체 소설)가 필요하면 design-big을, 통합 설계나 모호한 요청은 design 라우터를, 단일 영역만 필요하면 bootstrap/character/plot-hook을 사용하라."
---

# Novel Design Small — 작은 설계 오케스트레이터

웹소설의 25화 단위 세부 설계를 수행한다. 큰 설계 산출물을 토대로, domain-researcher 서브에이전트가 에피소드 수준의 구체적 전문 지식과 실제 사건 디테일을 자동 리서치한다.

## 실행 모드: 에이전트 팀

## 에이전트 구성

| 팀원 | 에이전트 파일 | 역할 | 스킬 | 출력 |
|------|-------------|------|------|------|
| character-architect | `${CLAUDE_PLUGIN_ROOT}/agents/character-architect.md` | 세부 캐릭터 설계 | character (모드 B) | 세부 캐릭터 시트 |
| plot-hook-engineer | `${CLAUDE_PLUGIN_ROOT}/agents/plot-hook-engineer.md` | 세부 플롯/훅 설계 | plot-hook (모드 B) | 세부 플롯 훅 가이드 |

**domain-researcher는 팀원이 아닌 서브에이전트**로, 팀 구성(TeamCreate) 전에 실행된다.

## 전제 조건

큰 설계 문서 3종이 존재해야 한다:
- `{작품가제}_부트스트랩.md`
- `{작품가제}_캐릭터시트.md`
- `{작품가제}_플롯훅가이드.md`

## 공유 레퍼런스

- **genre-dna-framework.md 위치**: `${CLAUDE_PLUGIN_ROOT}/skills/design/references/genre-dna-framework.md` (라우터 하위 — big/small 공용)

## 권한 안내

이 스킬은 팀원(서브에이전트)이 `.claude/`, `design/`, `_workspace/` 하위 파일을 Read합니다.
최초 실행 시 `Read(//**)` 권한을 허용하면 반복 승인 없이 진행됩니다.

## 워크플로우

### Phase 1: 범위 확인 및 큰 설계 문서 점검

1. 대상 화수 구간 확인 (권장: 25화 단위. 예: 1~25화, 26~50화)
2. 큰 설계 문서 3종 존재 확인 (novel-config.md가 있으면 `design_dir`에서, 없으면 프로젝트 루트와 `_workspace/`에서 탐색):
   - `{DESIGN_DIR}/{작품가제}_부트스트랩.md` (또는 `_workspace/01_*`)
   - `{DESIGN_DIR}/{작품가제}_캐릭터시트.md` (또는 `_workspace/02_*`)
   - `{DESIGN_DIR}/{작품가제}_플롯훅가이드.md` (또는 `_workspace/03_*`)
   - novel-config.md가 없으면 `design/` 및 프로젝트 루트를 순차 탐색
   - **누락 시**: 사용자에게 큰 설계를 먼저 완료하도록 안내하고 중단
3. 큰 설계 문서에서 해당 아크의 개요를 추출:
   - 아크 제목, 핵심 갈등, 주요 적대자
   - 해당 구간의 핵심 역량 모듈 항목
   - 등장 예정 캐릭터 (VIP, 신규 캐릭터)
4. **아크 범위 정합성 검증**: 요청 화수 구간이 큰 설계 아크 구조와 불일치하면 아크 경계를 사용자에게 안내하고 구간 조정 제안
5. 아크 개요를 사용자에게 확인 → Phase 1.5로 진행

### Phase 1.5: 자동 리서치 (서브에이전트)

> domain-researcher 서브에이전트를 호출하여 해당 아크의 세부 리서치를 자동 수행한다. 사용자 대기 없이 즉시 진행한다.

**서브에이전트: domain-researcher**
- subagent_type: `general-purpose`
- 리서치 항목:
  - **R7 전문 기술/지식 디테일**: 해당 아크에서 활용되는 구체적 전문 지식, 기술, 장면 디테일
  - **R8 사건 상세 타임라인**: 해당 아크 시간대의 실제 사건 상세 (날짜, 인물, 결과, 파급 효과)

- 프롬프트:
```
당신은 domain-researcher 서브에이전트입니다.
다음 리서치를 수행하세요:

1. R7 전문 기술/지식 디테일:
   - 전문 분야: {전문 분야}
   - 아크 범위: {N}~{M}화
   - 분석 항목: 구체적 기술명, 절차, 용어, 리액션 묘사에 활용할 디테일

2. R8 사건 상세 타임라인:
   - 시대: {해당 아크의 시간대}
   - 분석 항목: 정확한 날짜, 전조 신호, 파급 효과, 핵심 인물, 사회적 반응

큰 설계 문서를 참조하세요:
- {작품가제}_부트스트랩.md (핵심 역량 모듈, 전문 분야)
- {작품가제}_플롯훅가이드.md (해당 아크 개요)

출력:
- _workspace/00_research/R7_전문지식_{N}~{M}화.md
- _workspace/00_research/R8_사건상세_{N}~{M}화.md
```

- 리서치 결과는 `_workspace/00_research/`에 저장
- 리서치 완료 후 즉시 Phase 2로 진행

### Phase 2: 팀 구성

> ⚠️ **TeamCreate 전 안전 점검**:
> 기존 팀("design-big-team" 등)이 남아있으면 TeamCreate가 실패한다.
> "Already leading team" 오류 발생 시:
> 1. `TeamDelete("{기존 팀 이름}")`을 먼저 실행한다
> 2. TeamDelete 성공 후 아래 TeamCreate를 진행한다

### Phase 2 시작: 사전 로드 (TeamCreate 전)

리더는 TeamCreate 전에 팀원에게 전달할 컨텍스트를 사전 로드한다.

**Step 2-0a: 리서치 파일 로드**
- R7_CONTENT = Read("_workspace/00_research/R7_전문지식_{N}~{M}화.md") 또는 "(R7 리서치 미완료)"
- R8_CONTENT = Read("_workspace/00_research/R8_사건상세_{N}~{M}화.md") 또는 "(R8 리서치 미완료)"

**Step 2-0b: 큰 설계 문서 아크 섹션 추출**
- BOOTSTRAP_EXCERPT = Phase 1에서 추출한 아크 관련 섹션 (핵심 역량 모듈, 세계관 규칙, 유료 전환)
- CHARACTER_EXCERPT = 캐릭터시트에서 해당 아크 등장 캐릭터, 적대자, VIP 추출
- PLOT_EXCERPT = 플롯훅가이드에서 해당 아크 개요, 카타르시스 리듬 추출

리더는 TeamCreate 전에 `_workspace/00_research/` 내 리서치 결과 파일 존재 여부를 Glob으로 확인. 존재하는 파일만 프롬프트에 포함.

```
TeamCreate(
  team_name: "design-small-team",
  members: [
    {
      name: "character-architect",
      agent_type: "general-purpose",
      prompt: "당신은 character-architect 에이전트입니다.
        ${CLAUDE_PLUGIN_ROOT}/agents/character-architect.md를 읽고 역할을 숙지하세요.
        ${CLAUDE_PLUGIN_ROOT}/skills/character/SKILL.md를 읽고 모드 B(작은 설계) 절차와 출력 템플릿을 따르세요.
        ${CLAUDE_PLUGIN_ROOT}/skills/design/references/genre-dna-framework.md를 읽고 장르 DNA 프레임워크를 숙지하세요.

        ★ 큰 설계 요약 (Read 불필요 — 리더가 사전 로드):
        --- 부트스트랩 요약 ---
        {BOOTSTRAP_EXCERPT}
        --- 캐릭터시트 요약 ---
        {CHARACTER_EXCERPT}
        --- 플롯훅가이드 요약 ---
        {PLOT_EXCERPT}

        ★ 자동 리서치 결과 (Read 불필요 — 리더가 사전 로드):
        --- R7 전문지식 ---
        {R7_CONTENT}
        --- R8 사건상세 ---
        {R8_CONTENT}

        (요약이 불충분하면 큰 설계 문서 원본을 Read할 수 있으나, 권한 승인이 필요할 수 있음)

        대상 화수 구간: {N}~{M}화
        세부 캐릭터 시트를 작성하세요.

        ★ 집단 캐릭터 이름 부여 규칙:
        - R1/R2/R3/R4, 인턴, 수간호사 등 직급 캐릭터 중
          해당 화수 구간에서 대사가 2회 이상인 인물은 반드시 한국어 이름을 부여하라.
        - '이름' 열에는 실제 이름을, '관계' 열에는 직급(R3 레지던트 등)을 기입하라.
        - plot-hook-engineer가 에피소드별 비트에서 직급 코드 대신 이름을 사용할 수 있게 한다.

        출력: _workspace/04_character-architect_detail.md
        완료 후 리더에게 SendMessage로 완료를 보고하세요:
        '세부 캐릭터 시트 작성 완료. _workspace/04_character-architect_detail.md에 저장.'
        (리더가 plot-hook-engineer에게 핸드오프합니다. PHE에게 직접 SendMessage하지 마세요.)"
    },
    {
      name: "plot-hook-engineer",
      agent_type: "general-purpose",
      prompt: "당신은 plot-hook-engineer 에이전트입니다.
        ${CLAUDE_PLUGIN_ROOT}/agents/plot-hook-engineer.md를 읽고 역할을 숙지하세요.
        ${CLAUDE_PLUGIN_ROOT}/skills/plot-hook/SKILL.md를 읽고 모드 B(작은 설계) 절차와 출력 템플릿을 따르세요.
        ${CLAUDE_PLUGIN_ROOT}/skills/design/references/genre-dna-framework.md를 읽고 장르 DNA 프레임워크를 숙지하세요.

        ★ 큰 설계 요약 (Read 불필요 — 리더가 사전 로드):
        --- 부트스트랩 요약 ---
        {BOOTSTRAP_EXCERPT}
        --- 캐릭터시트 요약 ---
        {CHARACTER_EXCERPT}
        --- 플롯훅가이드 요약 ---
        {PLOT_EXCERPT}

        리더로부터 SendMessage를 수신하는 즉시 작업을 시작하세요. 별도 확인이나 대기 없이 즉시 시작합니다.
        _workspace/04_character-architect_detail.md를 Read하여 세부 캐릭터 시트를 숙지한 뒤
        세부 플롯/훅 가이드를 작성하세요.

        ★ 자동 리서치 결과 (Read 불필요 — 리더가 사전 로드):
        --- R7 전문지식 ---
        {R7_CONTENT}
        --- R8 사건상세 ---
        {R8_CONTENT}

        (요약이 불충분하면 큰 설계 문서 원본을 Read할 수 있으나, 권한 승인이 필요할 수 있음)

        대상 화수 구간: {N}~{M}화

        ★ 이름 사용 규칙:
        에피소드별 비트·클리프행어에 등장하는 인물은 직급 코드(R1, R3 등)가 아닌
        세부 캐릭터 시트에 정의된 실제 이름으로 표기하라.

        출력: _workspace/05_plot-hook-engineer_detail.md"
    }
  ]
)
```

작업 등록:
```
TaskCreate(tasks: [
  { title: "세부 캐릭터 시트 작성 ({N}~{M}화)", assignee: "character-architect" },
  { title: "세부 플롯/훅 가이드 작성 ({N}~{M}화)", assignee: "plot-hook-engineer",
    depends_on: ["세부 캐릭터 시트 작성 ({N}~{M}화)"] }
])
```

### Phase 3: 작은 설계 수행

**실행 방식:** 리더 중재 파이프라인

**Step 3-1**: character-architect 완료 대기
- `_workspace/04_character-architect_detail.md` 존재 여부를 Bash로 확인
- 미완료 시 60초 대기 후 재확인 (최대 5회)

**Step 3-2**: 리더 핸드오프
character-architect 완료 확인 즉시:
1. 리더가 `_workspace/04_character-architect_detail.md`를 Read
2. 핵심 내용 요약 추출:
   - 주인공 감정 곡선 요약
   - 신규 캐릭터 리스트와 등장 화수
   - 적대자 활동 타임라인
3. 리더가 plot-hook-engineer에게 직접 SendMessage:
   "character-architect 작업 완료.
   세부 캐릭터 시트: _workspace/04_character-architect_detail.md
   [핵심 요약 첨부]
   즉시 세부 플롯/훅 가이드 작성을 시작하세요.
   출력: _workspace/05_plot-hook-engineer_detail.md"

**Step 3-3**: plot-hook-engineer 완료 대기
- `_workspace/05_plot-hook-engineer_detail.md` 존재 여부를 Bash로 확인 (최대 5회, 60초 간격)
- 2회 확인 후에도 미생성이면 리더가 재차 SendMessage로 작업 시작 요청
- 4회 확인 후에도 미생성이면 에러 핸들링 (PHE 실패로 간주)

**작은 설계 중 모순 발견 시:**
- concept-builder는 참여하지 않으므로, 부트스트랩 수정이 필요하면 리더가 직접 `{작품가제}_부트스트랩.md`를 수정
- 수정 내역을 `_workspace/06_bootstrap_amendments.md`에 기록
- 부트스트랩은 여전히 source of truth

**산출물 저장:**

| 팀원 | 출력 경로 |
|------|----------|
| character-architect | `_workspace/04_character-architect_detail.md` |
| plot-hook-engineer | `_workspace/05_plot-hook-engineer_detail.md` |

### Phase 4: 통합 검증 및 정리

1. 각 팀원의 산출물을 Read로 수집
2. **일관성 검증 체크리스트:**
   - [ ] 세부 캐릭터의 등장 화수가 세부 플롯의 에피소드 배치와 일치하는가
   - [ ] 주인공의 감정 곡선이 단조롭지 않은가
   - [ ] 신규 캐릭터가 3명 이내인가 (초반 과다 등장 방지)
   - [ ] VIP 조우 이벤트가 10~15화 간격으로 배치되었는가
   - [ ] 적대자 활동이 카타르시스 리듬과 연동되는가
   - [ ] 전문 지식 활용 장면이 리서치 자료의 실제 기술/사례에 기반하는가
   - [ ] 핵심 역량 모듈 활용 장면의 날짜/인물/결과가 사건 상세 자료와 일치하는가
   - 모순 발견 시: 큰 설계 문서 기준으로 수정

3. 최종 산출물을 `{DESIGN_DIR}`에 복사 (novel-config.md의 경로와 일치시킴):

| 중간 산출물 | 최종 경로 |
|-----------|----------|
| `_workspace/04_*_detail.md` | `{DESIGN_DIR}/{작품가제}_세부캐릭터시트_{N}~{M}화.md` |
| `_workspace/05_*_detail.md` | `{DESIGN_DIR}/{작품가제}_세부플롯훅가이드_{N}~{M}화.md` |

   > **경로 일관성 원칙**: novel-config.md의 `design_dir`과 실제 파일 위치를 반드시 일치시킨다.

4. **novel-config.md 자동 업데이트** (작은 설계 산출물을 다운스트림 스킬에 연결):

   novel-config.md를 Read한 후, 아래 필드를 자동으로 추가/갱신한다:

   ```
   업데이트 항목:
   a) EP 범위별 설정문서 테이블의 해당 행에 세부 문서 경로 추가:
      - 세부 플롯 가이드 열: {DESIGN_DIR}/{작품가제}_세부플롯훅가이드_{N}~{M}화.md
      - 세부 캐릭터 시트 열: {DESIGN_DIR}/{작품가제}_세부캐릭터시트_{N}~{M}화.md
      (기존 행의 EP 범위가 일치하면 해당 열만 갱신, 범위가 없으면 새 행 추가)
   b) 공통 문서의 character_detail은 변경하지 않는다 (큰 설계 캐릭터시트를 유지).
      EP 범위별 세부 캐릭터 시트가 있으면 create/polish가 ep_range_table에서 우선 참조한다.
   c) R7/R8 리서치 결과 참조 경로 (보조 참조 섹션 — EP 범위별 배열):
      기존 research_r7/r8 항목이 있으면 **배열에 추가** (덮어쓰지 않음):
      - research_r7:
        - { range: "EP{N}~EP{M}", path: "_workspace/00_research/R7_전문지식_{N}~{M}화.md" }
      - research_r8:
        - { range: "EP{N}~EP{M}", path: "_workspace/00_research/R8_사건상세_{N}~{M}화.md" }
      동일 EP 범위의 기존 항목이 있으면 해당 항목만 덮어쓴다.
   ```

   novel-config.md를 Read한 후 즉시 업데이트를 적용한다 (비대화형).
   업데이트 완료 후 변경 내역을 사용자에게 출력한다 (사전 확인 요청 없음).

5. **팀원 종료 요청**
   - SendMessage(to: "character-architect", message: "작업 완료. 종료하세요.")
   - SendMessage(to: "plot-hook-engineer", message: "작업 완료. 종료하세요.")
   - 5초 대기 (팀원 종료 시간 확보)

6. **TeamDelete("design-small-team")** — 팀 즉시 해산
   ⚠️ 팀원 종료 SendMessage 후 즉시 실행. 사용자 응답 대기 없음.
   TeamDelete 실패 시:
   - 10초 대기 후 1회 재시도
   - 재실패 시 에러를 무시하고 다음 스텝으로 진행
     (다음 TeamCreate 시 Phase 2 안전 점검에서 기존 팀을 삭제)

7. `_workspace/` 디렉토리 보존 (사후 검증용)

8. 사용자에게 결과 요약 + 다음 25화 구간 작은 설계 안내:
   ```
   ## 작은 설계 완료 ({N}~{M}화)

   산출물:
   - {DESIGN_DIR}/{작품가제}_세부캐릭터시트_{N}~{M}화.md
   - {DESIGN_DIR}/{작품가제}_세부플롯훅가이드_{N}~{M}화.md

   다음 구간 작은 설계를 진행하시겠습니까?
   → 다음 범위: {M+1}~{M+25}화
   '작은 설계 {M+1}~{M+25}화' 라고 말씀하시면 바로 시작합니다.
   ```

## R7/R8 리서치 결과 활용 경로

domain-researcher가 생성하는 R7(전문지식)/R8(사건상세) 리서치 결과는 **세부 설계 문서에 녹아드는 방식**으로 활용된다.
다운스트림 스킬(create/polish/rewrite)에서 직접 참조하지 않는다.

```
활용 흐름:
R7 → character-architect의 세부 캐릭터 시트에 반영 (전문 지식 활용 장면 디테일)
R7 → plot-hook-engineer의 세부 플롯에 반영 (기술적 디테일의 에피소드 배치)
R8 → character-architect의 적대자 활동에 반영 (실제 사건 기반 타임라인)
R8 → plot-hook-engineer의 에피소드별 사건에 반영 (정확한 날짜/전조 신호)

최종 산출물 (세부캐릭터시트, 세부플롯훅가이드)에 리서치가 내재화되므로,
create/polish/rewrite는 산출물만 읽으면 리서치 내용이 자동으로 반영된다.

참고 경로 (novel-config.md 보조 참조):
- research_r7: _workspace/00_research/R7_전문지식_{N}~{M}화.md
- research_r8: _workspace/00_research/R8_사건상세_{N}~{M}화.md
→ 필요 시 수동 조회 가능
```

## 에러 핸들링

| 상황 | 전략 |
|------|------|
| 큰 설계 문서 3종 중 일부 누락 | Phase 1에서 감지. 누락 문서 없이는 작은 설계 불가 → 사용자에게 큰 설계 먼저 완료하도록 안내 |
| 큰 설계 문서의 포맷이 다름 | Phase 1에서 핵심 섹션(아크 구조, 핵심 역량 모듈, 적대자 계층) 존재 여부 확인. 핵심 섹션 부재 시 보완 요청 |
| 요청 화수 구간이 아크 구조와 불일치 | 아크 경계를 사용자에게 안내하고 구간 조정 제안 |
| domain-researcher 실패 | 1회 재시도. 재실패 시 큰 설계 문서 + genre-dna 기반으로 진행, 보고서에 "자동 리서치 미반영" 명시 |
| character-architect 실패 | 1회 재시도. 재실패 시 리더가 큰 설계 캐릭터 시트 기반 기본 세부 시트 생성 |
| plot-hook-engineer 실패 | 1회 재시도. 재실패 시 큰 설계 플롯 가이드 + 세부 캐릭터 시트로 기본 세부 플롯 생성 |
| 팀원 간 통신 지연 | 리더가 중간에서 파일을 Read하여 수동으로 정보 전달 |
| 리서치 파일 Read 실패 | 리더가 "(리서치 미완료)" 텍스트를 프롬프트에 임베드. 팀원은 큰 설계 요약 기반으로 진행 |

## 데이터 흐름

```
[사용자] → 작은 설계 요청 (화수 구간)
    ↓
Phase 1: 범위 확인 + 큰 설계 문서 점검
    ↓
Phase 1.5: domain-researcher 서브에이전트 → _workspace/00_research/ (R7, R8)
    ↓ (사용자 대기 없음)
Phase 2: TeamCreate("design-small-team") — 리서치 결과 포함
    ↓
Phase 3: character-architect → plot-hook-engineer
    ↓
Phase 4: 통합 검증 → 산출물 2종
    ↓
다음 아크 안내
```

## 테스트 시나리오

### 정상 흐름
1. 사용자가 "1~50화 작은 설계 해줘" 요청
2. Phase 1에서 큰 설계 문서 3종 확인, 아크 개요 추출
3. Phase 1.5에서 domain-researcher가 R7(전문 지식) + R8(사건 상세) 자동 리서치
4. Phase 2에서 팀 구성 (리서치 결과 포함)
5. Phase 3에서 CA→PHE 순서로 작은 설계 완성
6. Phase 4에서 검증 후 산출물 2종 + 다음 아크 안내

### 에러 흐름
1. Phase 1에서 `{작품가제}_플롯훅가이드.md` 누락 감지
2. 사용자에게 "플롯 훅 가이드가 없습니다. 큰 설계(design-big)를 먼저 완료해주세요." 안내
3. 작은 설계 중단
