---
name: design-big
description: "웹소설의 큰 설계(전체 소설)를 수행하는 오케스트레이터. 부트스트랩+캐릭터 시트+플롯 훅 가이드를 생성한다. 자동 리서치(domain-researcher 서브에이전트)로 전문 분야 자료를 수집한 후 설계를 진행한다. '큰 설계', '전체 소설 설계', '소설 설계', '부트스트랩부터 캐릭터 플롯까지' 요청에 이 스킬을 사용할 것. 통합 설계(큰+작은 둘 다)나 모호한 요청은 design 라우터를, 작은 설계만 필요하면 design-small을, 단일 영역만 필요하면 bootstrap/character/plot-hook을 사용하라."
---

# Novel Design Big — 큰 설계 오케스트레이터

웹소설의 전체 소설 설계를 수행한다. AI가 장르 DNA 프레임워크로 뼈대를 세우고, domain-researcher 서브에이전트가 자동 리서치로 전문 분야의 살을 붙이는 구조.

## 실행 모드: 에이전트 팀

## 에이전트 구성

| 팀원 | 에이전트 파일 | 역할 | 스킬 | 출력 |
|------|-------------|------|------|------|
| concept-builder | `${CLAUDE_PLUGIN_ROOT}/agents/concept-builder.md` | 부트스트랩 설계 | bootstrap | 부트스트랩 문서 |
| character-architect | `${CLAUDE_PLUGIN_ROOT}/agents/character-architect.md` | 캐릭터 설계 | character | 캐릭터 시트 |
| plot-hook-engineer | `${CLAUDE_PLUGIN_ROOT}/agents/plot-hook-engineer.md` | 플롯/훅 설계 | plot-hook | 플롯 훅 가이드 |

**domain-researcher는 팀원이 아닌 서브에이전트**로, Phase 1.5에서 팀 구성(TeamCreate) 전에 실행된다.

## 공유 레퍼런스

- **genre-dna-framework.md 위치**: `${CLAUDE_PLUGIN_ROOT}/skills/design/references/genre-dna-framework.md` (라우터 하위 — big/small 공용)

## 워크플로우

### Phase 0: 기존 설계 확인 (선택)

> 프로젝트 루트에 기존 큰 설계 산출물(`{작품가제}_부트스트랩.md`, `{작품가제}_캐릭터시트.md`, `{작품가제}_플롯훅가이드.md`)이 존재할 때만 실행. 없으면 Phase 1로 직행.

1. 프로젝트 루트에서 기존 큰 설계 산출물을 Glob으로 확인
2. 기존 산출물이 발견되면 사용자에게 모드 선택 요청:

```
기존 큰 설계 문서가 발견되었습니다:
- {발견된 파일 목록}

다음 중 선택해주세요:
1. **전체 재설계** — 기존 문서를 _workspace/_backup/에 백업 후 처음부터 다시 설계
2. **부분 수정** — 특정 영역만 재실행 (예: 부트스트랩만, 캐릭터만, 플롯만)
3. **자료 보강 후 재설계** — 새 리서치 자료를 추가한 뒤 기존 설계를 업그레이드
```

3. 모드별 처리:
   - **전체 재설계**: 기존 산출물을 `_workspace/_backup/{timestamp}/`에 복사 → Phase 1부터 진행
   - **부분 수정**: 수정 대상 에이전트만 재실행. 나머지 산출물은 유지. Phase 2에서 해당 에이전트만 팀에 포함.
   - **자료 보강 후 재설계**: 기존 `_workspace/00_concept_analysis.md`에서 컨셉 복원 → Phase 1 스킵 → Phase 1.5부터 진행

### Phase 1: 컨셉 분석 및 방향 수립

1. **제안서 자동 연결** (강화):
   - 프로젝트 루트에서 `*_제안서.md` 패턴을 Glob으로 탐색한다
   - 발견되면 Read하여 제안서의 **컨셉, 장르, 플랫폼, 차별화 포인트, 로그라인**을 자동 추출한다
   - 추출된 플랫폼은 반드시 아래 허용 플랫폼 집합으로 검증한다:
     - 문피아, 네이버시리즈, 카카오페이지, 리디, 조아라, 노벨피아
   - `Munpia`, `munpia`, `문 피아`, `카카페`, `KakaoPage`, `리디북스`, `NovelPia` 등 허용 별칭은 canonical name으로 정규화한다
   - 허용되지 않는 플랫폼 또는 모호한 별칭이면 자동 반영하지 않고, 사용자에게 제안서 수정 또는 플랫폼 재선택을 요청한다
   - 추출된 정보로 Phase 1의 컨셉 분석을 사전 채운다 (사용자에게 확인 요청)
   - 제안서의 `_workspace/00_research/` 디렉토리도 확인하여 기존 R1/R2/R5 리서치가 있으면 Phase 1.5에서 재활용한다
   - 제안서가 없으면: 사용자 입력에서 직접 분석 (기존 동작)
   ```
   제안서 '{파일명}'을(를) 발견했습니다.
   다음 컨셉으로 큰 설계를 진행합니다:
   - 장르: {제안서에서 추출}
   - 컨셉: {제안서에서 추출}
   - 플랫폼: {제안서에서 추출}
   - 차별화: {제안서에서 추출}

   수정하실 부분이 있으면 말씀해주세요. 없으면 바로 진행합니다.
   ```
2. 사용자 입력 분석 — 소설 컨셉, 직업, 분위기, 차별화 방향 파악
   - 타겟 플랫폼이 명시되지 않았으면 한국 플랫폼 6개 중 하나를 확인한다
   - 타겟 플랫폼이 비지원 값이면 자동 치환하지 않고 재선택을 요청한다
3. `${CLAUDE_PLUGIN_ROOT}/skills/design/references/genre-dna-framework.md`를 읽어 장르 DNA 프레임워크 확인
4. 프로젝트 루트의 참고 문서 확인 (존재 시)
5. **컨셉 방향 요약**을 사용자에게 제시하여 확인:
   - 주인공 직업/전문 분야
   - 서사 기점 (시대, 계기)
   - 핵심 차별화 포인트
   - 타겟 플랫폼 (문피아, 네이버시리즈, 카카오페이지, 리디, 조아라, 노벨피아 중 1개)
   - 예상 톤 & 무드
   - 유사 작품군 2~3개 (리더가 genre-dna 차별화 전략 + 업계 상식으로 제안)
6. **`{작품가제}` 확정** — 사용자에게 작품 가제를 확인한다. 이 가제가 이후 모든 파일명의 접두사로 사용된다.
7. Phase 1 결과를 `_workspace/00_concept_analysis.md`에 저장 — 새 대화 시작 시에도 컨셉을 복원할 수 있도록 한다. 저장 내용: 작품가제, 전문 분야, 서사 기점, 차별화 포인트, 유사 작품군, 톤 & 무드.
8. 사용자 확인 후 → Phase 1.5로 진행

### Phase 1.5: 자동 리서치 (서브에이전트)

> domain-researcher 서브에이전트를 호출하여 전문 분야 리서치를 자동 수행한다. 사용자 대기 없이 즉시 진행한다.

**서브에이전트: domain-researcher**
- subagent_type: `general-purpose`
- 리서치 항목:
  - **R3 업계/직업 구조**: 해당 전문 분야의 조직 구조, 커리어 패스, 권력 계층
  - **R4 사건 연표**: 해당 분야/시대의 주요 사건, 전환점, 업계 변화
  - **R5 기존작 분석**: 유사 장르/소재의 기존 웹소설 분석, 차별화 가능점
  - **R6 갈등 사례**: 해당 분야의 실제 갈등/사건 사례, 빌런 모티프

- 프롬프트:
```
당신은 domain-researcher 서브에이전트입니다.
다음 리서치를 수행하세요:

1. R3 업계/직업 구조:
   - 전문 분야: {전문 분야}
   - 분석 항목: 조직 구조, 직급 체계, 커리어 패스, 권력 관계, 핵심 역량

2. R4 사건 연표:
   - 시대: {서사 기점 시대}~현재
   - 분석 항목: 업계 주요 사건, 사회적 전환점, 기술 변화, 위기와 기회

3. R5 기존작 분석:
   - 장르: {장르}
   - 분석 항목: 유사 작품 리스트, 성공 요인, 독자 반응, 차별화 빈 공간

4. R6 갈등 사례:
   - 분야: {전문 분야}
   - 분석 항목: 실제 갈등/비리/사건, 빌런 모티프로 활용 가능한 패턴

출력:
- _workspace/00_research/R3_업계구조.md
- _workspace/00_research/R4_사건연표.md
- _workspace/00_research/R5_기존작분석.md
- _workspace/00_research/R6_갈등사례.md
```

- 리서치 결과는 `_workspace/00_research/`에 저장
- 리서치 완료 후 즉시 Phase 2로 진행 (사용자 대기 없음)

### Phase 2: 팀 구성

리더는 TeamCreate 전에 `_workspace/00_research/` 내 리서치 결과 파일의 존재 여부를 Glob으로 확인한다.

```
TeamCreate(
  team_name: "design-big-team",
  members: [
    {
      name: "concept-builder",
      agent_type: "general-purpose",
      prompt: "당신은 concept-builder 에이전트입니다.
        ${CLAUDE_PLUGIN_ROOT}/agents/concept-builder.md를 읽고 역할을 숙지하세요.
        ${CLAUDE_PLUGIN_ROOT}/skills/bootstrap/SKILL.md를 읽고 작업 절차와 출력 템플릿을 따르세요.
        ${CLAUDE_PLUGIN_ROOT}/skills/design/references/genre-dna-framework.md를 읽고 장르 DNA 프레임워크를 숙지하세요.
        프로젝트 루트의 참고 문서도 읽으세요 (존재 시).

        ★ 자동 리서치 결과 (존재하는 파일만 Read):
        - _workspace/00_research/R3_업계구조.md → 반영: 세계관 > 업계 조직도, 주인공 배경 > 커리어 패스
        - _workspace/00_research/R4_사건연표.md → 반영: 핵심 역량 모듈(미래 지식 연표) 구체화
        - _workspace/00_research/R5_기존작분석.md → 반영: 기존 작품 대비 포지셔닝, 셀링포인트 차별화
        - _workspace/00_research/R6_갈등사례.md → 반영: 세계관 > 사회적 맥락, 핵심 역량 모듈 > 서사적 기능
        (파일이 없으면 genre-dna 기반으로 진행하되, 보고서에 '리서치 자료 미반영: [카테고리명]' 명시)

        사용자의 소설 컨셉: {사용자 입력 요약}
        부트스트랩 문서를 작성하여 _workspace/01_concept-builder_bootstrap.md에 저장하세요.
        작성 완료 후 다음 정보를 SendMessage하세요:
        - character-architect에게: (1)주인공 핵심 설정 (2)전문 분야 특성 (3)서사 기점 (4)세계관의 사회 구조
        - plot-hook-engineer에게: (1)핵심 역량 모듈 요약 (2)유료 전환 전략 (3)50화 단위 아크 골격 (4)스케일 확대 로드맵"
    },
    {
      name: "character-architect",
      agent_type: "general-purpose",
      prompt: "당신은 character-architect 에이전트입니다.
        ${CLAUDE_PLUGIN_ROOT}/agents/character-architect.md를 읽고 역할을 숙지하세요.
        ${CLAUDE_PLUGIN_ROOT}/skills/character/SKILL.md를 읽고 작업 절차와 출력 템플릿을 따르세요.
        ${CLAUDE_PLUGIN_ROOT}/skills/design/references/genre-dna-framework.md를 읽고 장르 DNA 프레임워크를 숙지하세요.

        ★ 자동 리서치 결과 (존재하는 파일만 Read):
        - _workspace/00_research/R3_업계구조.md → 반영: 빌런 > 직급/소속/권한, 조력자 > 업계 내 위치
        - _workspace/00_research/R6_갈등사례.md → 반영: 빌런 > 동기와 행동 패턴, 갈등 유형의 현실적 근거
        (파일이 없으면 genre-dna 캐릭터 프레임워크 기반으로 진행하되, 보고서에 미반영 카테고리 명시)

        concept-builder로부터 SendMessage를 수신하면,
        _workspace/01_concept-builder_bootstrap.md를 Read하여 전체 부트스트랩을 숙지한 뒤
        큰 설계 캐릭터 시트를 작성하세요.
        출력: _workspace/02_character-architect_sheet.md
        작성 완료 후 plot-hook-engineer에게 다음을 SendMessage하세요:
        (1)주인공 핵심 동기 (2)적대자 계층 전체 (3)VIP 조력자 리스트와 등장 시점 (4)로맨스 라인 설정"
    },
    {
      name: "plot-hook-engineer",
      agent_type: "general-purpose",
      prompt: "당신은 plot-hook-engineer 에이전트입니다.
        ${CLAUDE_PLUGIN_ROOT}/agents/plot-hook-engineer.md를 읽고 역할을 숙지하세요.
        ${CLAUDE_PLUGIN_ROOT}/skills/plot-hook/SKILL.md를 읽고 작업 절차와 출력 템플릿을 따르세요.
        ${CLAUDE_PLUGIN_ROOT}/skills/design/references/genre-dna-framework.md를 읽고 장르 DNA 프레임워크를 숙지하세요.

        ★ 자동 리서치 결과 (존재하는 파일만 Read):
        - _workspace/00_research/R4_사건연표.md → 반영: 핵심 역량 모듈 활용 타임라인 > 실제 사건 매핑
        - _workspace/00_research/R6_갈등사례.md → 반영: 아크별 갈등 구조, 빌런 대결의 현실적 패턴
        (파일이 없으면 genre-dna 서사 공식 기반으로 진행하되, 보고서에 미반영 카테고리 명시)

        concept-builder와 character-architect 양쪽에서 SendMessage를 모두 수신하면,
        _workspace/01_concept-builder_bootstrap.md와 _workspace/02_character-architect_sheet.md를
        Read하여 전체 내용을 숙지한 뒤 큰 설계 플롯/훅 가이드를 작성하세요.
        출력: _workspace/03_plot-hook-engineer_guide.md"
    }
  ]
)
```

작업 등록:
```
TaskCreate(tasks: [
  { title: "부트스트랩 문서 작성", assignee: "concept-builder" },
  { title: "캐릭터 시트 작성 (큰 설계)", assignee: "character-architect",
    depends_on: ["부트스트랩 문서 작성"] },
  { title: "플롯/훅 가이드 작성 (큰 설계)", assignee: "plot-hook-engineer",
    depends_on: ["부트스트랩 문서 작성", "캐릭터 시트 작성 (큰 설계)"] }
])
```

### Phase 3: 큰 설계 수행

**실행 방식:** 파이프라인 + 부분 병렬

1. concept-builder가 부트스트랩 문서 작성
2. concept-builder → character-architect, plot-hook-engineer에게 SendMessage (핵심 설정 공유)
3. character-architect가 캐릭터 시트 작성 (concept-builder의 설정 기반)
4. character-architect → plot-hook-engineer에게 SendMessage (적대자 계층, VIP 리스트)
5. plot-hook-engineer가 플롯/훅 가이드 작성 (부트스트랩 + 캐릭터 시트 기반)

**팀원 간 통신 규칙:**
- concept-builder는 부트스트랩 완성 시 양쪽 팀원에게 핵심 설정 SendMessage
- character-architect는 캐릭터 시트 완성 시 plot-hook-engineer에게 SendMessage
- 설정 모순 발견 시 해당 팀원에게 직접 SendMessage로 조정 요청
- 각 팀원은 파일 저장 완료 시 리더에게 알림

**산출물 저장:**

| 팀원 | 출력 경로 |
|------|----------|
| concept-builder | `_workspace/01_concept-builder_bootstrap.md` |
| character-architect | `_workspace/02_character-architect_sheet.md` |
| plot-hook-engineer | `_workspace/03_plot-hook-engineer_guide.md` |

**리더 모니터링:**
- TaskGet으로 전체 진행률 확인
- 팀원 유휴 시 자동 알림 수신
- 특정 팀원이 막히면 SendMessage로 개입

### Phase 4: 통합 검증 및 정리

1. 모든 팀원 작업 완료 대기 (TaskGet으로 상태 확인)
2. 각 팀원의 산출물을 Read로 수집
3. **일관성 검증 체크리스트** (부트스트랩 문서가 source of truth):
   - [ ] 주인공 직업/나이/서사 기점이 부트스트랩↔캐릭터 시트에서 일치하는가
   - [ ] 주인공의 과거/배경 설정이 부트스트랩↔캐릭터 시트에서 일치하는가
   - [ ] 핵심 역량 모듈(부트스트랩)의 항목이 플롯 가이드의 활용 타임라인에 반영되었는가
   - [ ] 캐릭터 시트의 적대자 이름/아크가 플롯 가이드의 아크별 적대자와 일치하는가
   - [ ] VIP 조력자의 등장 시점(캐릭터 시트)이 플롯 가이드의 타임라인과 맞는가
   - [ ] 로맨스 라인 첫 만남 시점이 캐릭터 시트↔플롯 가이드에서 일치하는가
   - [ ] 유료 전환 전략(부트스트랩)이 플롯 가이드의 25화/50화 배치와 정합하는가
   - [ ] 리서치 자료의 핵심 정보가 부트스트랩 핵심 역량 모듈에 반영되었는가
   - [ ] 적대자의 직급/행동이 업계 구조 리서치와 정합하는가
   - [ ] 실제 갈등 사례에서 영감을 받은 아크가 최소 1개 이상인가
   - 모순 발견 시: 부트스트랩 기준으로 다른 문서를 수정하고, 수정 내역을 결과 보고에 포함

4. 최종 산출물을 `{DESIGN_DIR}`에 복사 (novel-config.md의 경로와 일치시킴):

| 중간 산출물 | 최종 경로 |
|-----------|----------|
| `_workspace/01_*_bootstrap.md` | `{DESIGN_DIR}/{작품가제}_부트스트랩.md` |
| `_workspace/02_*_sheet.md` | `{DESIGN_DIR}/{작품가제}_캐릭터시트.md` |
| `_workspace/03_*_guide.md` | `{DESIGN_DIR}/{작품가제}_플롯훅가이드.md` |

   > **경로 일관성 원칙**: novel-config.md의 설정문서 매핑 경로와 실제 파일 위치가 반드시 일치해야 한다.
   > Phase 5에서 config에 `design/{작품가제}_*.md`로 기록하므로, 여기서도 `design/` 하위에 저장한다.
   > `{DESIGN_DIR}` 디렉토리가 없으면 생성한다 (기본값: `design/`).

5. 팀원들에게 종료 요청 (SendMessage)
6. **TeamDelete("design-big-team")** — 팀 해산
   > 이후 design-small 실행 시 팀 충돌 방지. 반드시 실행한다.
7. `_workspace/` 디렉토리 보존 (사후 검증용)
8. 사용자에게 결과 요약 보고

### Phase 5: novel-config.md 초안 자동 생성

큰 설계 완료 후, 창작/윤문/재작성 스킬이 사용할 `novel-config.md` 초안을 자동 생성한다.
사용자가 수동으로 작성할 필요 없이, 설계 산출물의 경로와 구조를 분석하여 초안을 만든다.

1. 플롯훅가이드에서 아크 구조(1막/2막/3막)를 추출하여 EP 범위 테이블을 자동 구성한다
2. 부트스트랩에서 보존 가드레일 후보를 추출한다 (세계관 규칙, 핵심 설정)
3. 캐릭터시트에서 대화 DNA 섹션 존재 여부를 확인한다
4. `${CLAUDE_PLUGIN_ROOT}/skills/polish/references/project-config-template.md`를 참조하여 형식을 맞춘다
5. **target_platform 검증 게이트**:
   - Phase 1에서 확정된 플랫폼이 한국 플랫폼 canonical name 6개 중 하나인지 재검증한다
   - 비지원 값이면 `novel-config.md`를 생성하지 않고 사용자 수정을 요청한다

생성 경로: `{프로젝트 루트}/novel-config.md`

```markdown
# novel-config.md (자동 생성 초안 — 검토 후 수정 가능)

## 프로젝트 기본 정보
project:
  name: "{작품가제}"
  target_platform: "{Phase 1에서 확인된 플랫폼}"
  target_genre: "{Phase 1에서 확인된 장르}"
  episode_dir: "episode/"
  work_dir: "revision/"
  design_dir: "design/"

## 설정문서 매핑
### 공통 문서
| 문서 키 | 경로 | 용도 |
|---------|------|------|
| character_core | {DESIGN_DIR}/{작품가제}_캐릭터시트.md | 캐릭터 핵심 정의 |
| character_detail | {DESIGN_DIR}/{작품가제}_캐릭터시트.md | 보이스표, 비언어 태그 |
| dialogue_dna | {DESIGN_DIR}/{작품가제}_캐릭터시트.md#dialogue-dna | Dialogue DNA (대사 고유성) — 캐릭터시트 내 섹션 |
| bootstrap | {DESIGN_DIR}/{작품가제}_부트스트랩.md | 세계관, 매크로 수치 |
| writing_rules | CLAUDE.md | 집필 규칙 |

### EP 범위별 설정문서
| EP 범위 | 레이블 | 플롯 가이드 경로 | 세부 플롯 가이드 (선택) | 세부 캐릭터 시트 (선택) |
|---------|--------|----------------|----------------------|----------------------|
| {아크1 범위} | {아크1 레이블} | {DESIGN_DIR}/{작품가제}_플롯훅가이드.md | | |
| {아크2 범위} | {아크2 레이블} | {DESIGN_DIR}/{작품가제}_플롯훅가이드.md | | |
| {아크3 범위} | {아크3 레이블} | {DESIGN_DIR}/{작품가제}_플롯훅가이드.md | | |

## 보존 가드레일
{부트스트랩에서 추출한 핵심 보존 항목}

## 수치 교차검증 정본 우선순위
1. plot_by_ep — EP별 확정 수치
2. bootstrap — 매크로 수치
3. verification — 검증 완료 수치
4. 직전 에피소드 — 서사 연속성
```

5. 사용자에게 초안 검토를 요청한다:
```
novel-config.md 초안을 생성했습니다.
보존 가드레일과 EP 범위 테이블을 검토하고, 필요시 커스텀 축을 추가해주세요.
```

### 큰 설계 완료 시 작은 설계 안내

```
## 큰 설계가 완료되었습니다.

### 생성된 문서
- design/{작품가제}_부트스트랩.md
- design/{작품가제}_캐릭터시트.md
- design/{작품가제}_플롯훅가이드.md
- novel-config.md (초안 — 검토 후 수정 가능)

### 다음 단계: 작은 설계 (25화 단위 세부 설계)

⚠️ **작은 설계를 건너뛰고 바로 `/create`를 실행하면 EP별 플롯 비트가 없어 에피소드 품질이 크게 저하됩니다.**
큰 설계의 플롯 훅 가이드는 아크 단위 개요만 포함하므로, episode-architect가 EP별 설계도를 추출하기 어렵습니다.

작은 설계를 진행하시려면 `design-small` 스킬을 사용하세요.
작은 설계에서도 domain-researcher가 자동으로 해당 아크의 세부 리서치를 수행합니다.
작은 설계 완료 시 novel-config.md의 EP 범위 테이블에 세부 플롯 가이드 경로가 자동 추가됩니다.
```

## 에러 핸들링

| 상황 | 전략 |
|------|------|
| domain-researcher 실패 | 1회 재시도. 재실패 시 genre-dna 기반으로 팀 구성 진행, 보고서에 "자동 리서치 미반영" 명시 |
| concept-builder 실패 | 핵심 설정이므로 반드시 재시도. 재실패 시 리더가 직접 부트스트랩 초안 작성 |
| character-architect 실패 | 1회 재시도. 재실패 시 리더가 genre-dna 기반 기본 캐릭터 시트 생성 |
| plot-hook-engineer 실패 | 1회 재시도. 재실패 시 부트스트랩+캐릭터 시트만으로 기본 플롯 가이드 생성 |
| 설정 모순 발견 | 부트스트랩 문서를 기준(source of truth)으로 다른 문서 수정 |
| 팀원 간 통신 지연 | 리더가 중간에서 파일을 Read하여 수동으로 정보 전달 |
| 리서치 결과 품질 부족 | genre-dna 프레임워크 기반으로 보강, 보고서에 명시 |
| 제안서/입력의 플랫폼이 비지원 값 | 자동 매핑하지 않고 한국 플랫폼 6개 중 재선택 요청 |

## 데이터 흐름

```
[사용자] → 소설 컨셉
    ↓
Phase 0: 기존 설계 확인 (선택)
    ↓
Phase 1: 컨셉 분석 → _workspace/00_concept_analysis.md
    ↓ (제안서 존재 시 자동 로드)
Phase 1.5: domain-researcher 서브에이전트 → _workspace/00_research/ (R3~R6)
    ↓ (사용자 대기 없음)
Phase 2: TeamCreate("design-big-team") — 리서치 결과 포함
    ↓
Phase 3: concept-builder → character-architect → plot-hook-engineer
    ↓
Phase 4: 통합 검증 → 산출물 3종
    ↓
작은 설계 안내
```

## 테스트 시나리오

### 정상 흐름
1. 사용자가 "회귀한 법의학 전문의" 컨셉 제공
2. Phase 1에서 컨셉 분석, 작품가제 확정 ("법의학전문의")
3. Phase 1.5에서 domain-researcher가 R3~R6 자동 리서치 수행
4. Phase 2에서 팀 구성 (리서치 결과 프롬프트에 포함)
5. Phase 3에서 CB→CA→PHE 순서로 큰 설계 완성
6. Phase 4에서 일관성 검증 후 산출물 3종 + 작은 설계 안내

### 제안서 연결 흐름
1. 프로젝트 루트에 `법의학전문의_제안서.md` 존재
2. Phase 1에서 제안서를 로드하여 컨셉 자동 파악
3. 사용자 확인 후 Phase 1.5로 즉시 진행
4. 이후 정상 흐름과 동일

### 에러 흐름
1. Phase 1.5에서 domain-researcher 실패
2. 리더가 1회 재시도 → 재실패
3. genre-dna 프레임워크 기반으로 Phase 2 진행, 보고서에 "자동 리서치 미반영" 명시
4. Phase 3에서 plot-hook-engineer 에러로 중지
5. 리더가 유휴 알림 수신 → SendMessage로 상태 확인 → 1회 재시도
6. 재시도 실패 시 리더가 부트스트랩 + 캐릭터 시트 기반 기본 플롯 가이드 직접 작성
7. 최종 보고서에 "plot-hook-engineer 자동 생성 — 수동 검토 권장" 명시
