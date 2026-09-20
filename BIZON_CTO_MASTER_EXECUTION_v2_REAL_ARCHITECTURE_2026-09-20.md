# BizON CTO MASTER EXECUTION v2

## Реальный план доведения текущей версии v1.2.0 до Production через полезность информации, профессиональный граф, доказательства и управляемый AI

**Дата:** 20.09.2026\
**Базовая версия:** BizON v1.2.0-pilot-platform, снапшот 19.09.2026\
**Роль документа:** единый execution document для CTO, архитекторов,
UX/UI, backend/frontend, data, AI, security, QA, SRE и z.ai.

------------------------------------------------------------------------

# 0. Главный вывод CTO

Предыдущий документ был слишком похож на checklist релиза.

Это недостаточно для BizON.

Текущая версия уже содержит значительный объём технического ядра:
155/155 шагов Plan-v2 заявлены закрытыми, 225/225 тестов прошли, есть
Governance, Store, Project Room, Opportunity, News, observability и
другие подсистемы. Однако внешний аудит 19.09 показывает примерно 70%
READY и около 25% PARTIAL/DOC-MISMATCH, а главный разрыв описан как
**«ядро без дверей»** --- готовые API не всегда превращены в реальные
пользовательские пути. fileciteturn4file0

Но для CTO есть более глубокая проблема.

## Главная продуктовая проблема

BizON сейчас рискует стать системой, в которой **данных и функций
больше, чем решений, которые пользователь может принять благодаря этим
данным**.

Это противоречит North Star BizON:

> система должна не просто показывать контент, а повышать вероятность
> следующего полезного профессионального результата.
> fileciteturn4file2

Поэтому следующий этап нельзя определять как:

> «доделать ещё API и UI».

Правильная цель:

> **превратить существующее техническое ядро в систему, которая на
> каждом ключевом экране отвечает пользователю на вопрос: "что я теперь
> могу сделать и почему эта информация важна именно мне?"**

------------------------------------------------------------------------

# 1. ЧТО МЫ СЧИТАЕМ ТЕКУЩЕЙ ПРАВДОЙ

Последний отчёт фиксирует:

-   Plan-v2 --- 155/155;
-   225 pass / 0 fail по заявленным 22 тест-файлам;
-   Governance: Action Registry + Policy Engine + Governance Service +
    hash-chain AuditLog;
-   `/api/metrics` с RPS/p50/p95/p99;
-   Grafana dashboard;
-   15 adversarial auth cases;
-   Virtual Cashier и Cashier Ports;
-   реальный login smoke для 5 аккаунтов;
-   браузерный аудит физлица и юрлица на 390px.
    fileciteturn4file0turn4file1

Одновременно аудит обнаружил:

-   UI/API gaps;
-   неполные пользовательские consumer paths;
-   fake-success в отдельных местах;
-   тестовые данные;
-   misleading ML terminology;
-   слабую независимость evaluation;
-   push-status risk;
-   outbox producer gaps;
-   sitemap/UI integrity issues;
-   слишком сильную зависимость старых gates от grep;
-   необходимость второй независимой приёмки. fileciteturn4file0

Это означает:

> **сейчас нельзя начинать большой rewrite.**

Нужно сначала превратить уже созданное ядро в согласованную систему.

------------------------------------------------------------------------

# 2. АРХИТЕКТУРНАЯ МОДЕЛЬ BIZON

BizON должен работать как:

``` text
PERSON / COMPANY
        ↓
CURRENT INTENT
        ↓
NEED / OPPORTUNITY
        ↓
MATCH
        ↓
ACTION
        ↓
PROJECT / WORK
        ↓
CONTRIBUTION
        ↓
RESULT
        ↓
EVIDENCE
        ↓
SKILL
        ↓
TRUST
        ↓
RECOMMENDATION / RELATIONSHIP
        ↓
NEW OPPORTUNITY
        ↓
NEW RESULT
```

News подключается:

``` text
NEWS
 ↓
COMPANY
 ↓
PROJECT / NEED
 ↓
OPPORTUNITY
 ↓
SKILL
 ↓
PERSON / TEAM
```

Эта связь прямо предусмотрена master specification.
fileciteturn4file3turn4file4

------------------------------------------------------------------------

# 3. НОВАЯ CTO-ФОРМУЛА КАЖДОЙ СТРАНИЦЫ

Для каждого экрана использовать:

``` text
USER
 ↓
INTENT
 ↓
QUESTION
 ↓
MINIMUM INFORMATION
 ↓
PROOF
 ↓
ACTION
 ↓
RESULT
```

Не:

``` text
DATABASE
 ↓
ALL AVAILABLE FIELDS
 ↓
UI
```

------------------------------------------------------------------------

# 4. ПРАВИЛО «ИНФОРМАЦИЯ ДОЛЖНА ЗАРАБОТАТЬ»

Каждый UI block обязан иметь:

``` ts
type InformationPurpose = {
  userQuestion: string
  decisionEnabled: string
  actionEnabled?: string
  evidenceSource?: string
  visibility: 'primary' | 'secondary' | 'contextual' | 'private'
}
```

Если нельзя заполнить:

``` text
userQuestion
decisionEnabled
```

элемент не должен находиться на первом экране.

------------------------------------------------------------------------

# 5. 20+ ЭКСПЕРТНЫХ РОЛЕЙ --- НЕ ДЛЯ КРАСИВОГО СПИСКА

Ниже --- конкретно, **что каждая роль должна изменить в BizON**.

------------------------------------------------------------------------

## ROLE 1 --- CHIEF PRODUCT ARCHITECT

### Задача

Свести продукт к Professional Value Loop.

### Изменение

Каждая функция получает:

``` text
REQ
→ user problem
→ graph object
→ action
→ result
→ KPI
```

### Проверка

Ни одна feature не принимается, если она заканчивается на:

``` text
"пользователь увидел информацию"
```

Нужен результат или осмысленное следующее действие.

------------------------------------------------------------------------

# ROLE 2 --- SYSTEM ARCHITECT

### Задача

Проверить зависимости.

Главный граф:

``` text
Identity
→ Intent
→ Opportunity
→ Project
→ Evidence
→ Skill
→ Trust
→ Recommendation
```

### Что сделать

Создать:

`docs/architecture/BIZON-CANONICAL-GRAPH.md`

Для каждой сущности:

-   owner;
-   source of truth;
-   lifecycle;
-   permissions;
-   API;
-   consumers;
-   audit requirements.

### Gate

Нет двух разных источников правды для одного понятия.

------------------------------------------------------------------------

# ROLE 3 --- REPOSITORY TRUTH AUDITOR

### Задача

Не доверять документации.

Для каждой feature:

``` text
DOC
vs
CODE
vs
RUNTIME
vs
DATABASE
```

### Выход

``` text
READY
PARTIAL
MOCK
DOC-MISMATCH
MISSING
DEPRECATED
DANGEROUS
```

### Gate

Ни один P0 не может быть READY только по grep.

------------------------------------------------------------------------

# ROLE 4 --- INFORMATION ARCHITECT

Это **главная роль для твоего текущего беспокойства**.

## Задача

Сделать BizON не энциклопедией данных, а системой решений.

Создать:

`docs/ux/INFORMATION-ARCHITECTURE-MATRIX.md`

Для каждого блока:

  Screen   Block   Question   Decision   Action   Evidence   Priority
  -------- ------- ---------- ---------- -------- ---------- ----------

### Пример

``` text
Profile → Skills
Question:
"Подходит ли человек для моей задачи?"

Decision:
"Есть ли нужный capability?"

Action:
"Open relevant proof"

Evidence:
ProjectContribution + SkillEvidence
```

Если Skills просто перечисляются --- это недостаточно.

------------------------------------------------------------------------

# ROLE 5 --- UX ARCHITECT

## Profile flow

Первый экран:

``` text
WHO
WHAT THEY DO
WHAT THEY WANT NOW
WHAT PROVES IT
WHAT I CAN DO NEXT
```

### Target component

``` tsx
<ProfessionalHero />
<CurrentIntent />
<ProofSummary />
<RelevantCapabilities />
<RelevantProjects />
<Recommendations />
<PrimaryActions />
```

Не показывать 20 блоков сразу.

------------------------------------------------------------------------

# ROLE 6 --- UI / VISUAL DESIGNER

Master specification задаёт направление:

-   light;
-   minimal;
-   premium;
-   calm;
-   international;
-   lighthouse of truth;
-   без generic SaaS dashboard;
-   без dashboard overload;
-   без dark-only AI UI. fileciteturn4file6

### Задача

Не «перекрасить».

Нужно создать visual hierarchy:

``` text
PRIMARY
↓
SUPPORTING
↓
PROOF
↓
DETAIL
```

### Gate

На первом экране максимум 1 primary decision и 1--3 supporting signals.

------------------------------------------------------------------------

# ROLE 7 --- BEHAVIOUR DESIGNER

Каждый экран должен иметь:

``` text
Trigger
→ Understanding
→ Decision
→ Action
→ Feedback
```

Пример:

``` text
"You may need this engineer"
→
"4 matching capabilities"
→
"proof exists"
→
"request introduction"
→
"intro pending"
```

------------------------------------------------------------------------

# ROLE 8 --- PERSON / PROFESSIONAL PASSPORT ARCHITECT

Master specification определяет Professional Passport как:

`NAME — ROLE — WHAT I DO`

затем:

-   proof;
-   projects;
-   skills;
-   recommendations;
-   career;
-   intent;
-   contact;
-   QR. fileciteturn4file12

### Новая логика

Не:

``` text
"профиль пользователя"
```

а:

``` text
"ответ на вопрос: могу ли я работать с этим человеком?"
```

------------------------------------------------------------------------

# ROLE 9 --- COMPANY / LEGAL ENTITY ARCHITECT

Компания --- отдельный продукт, а не UserProfile с другим аватаром.

Public:

``` text
Identity
→ What we create
→ Products
→ Projects
→ People
→ Opportunities
→ News
→ Trust / Evidence
→ Contacts
```

Internal:

``` text
Control Center
→ Hiring
→ Projects
→ Team
→ AI
→ Analytics
→ Governance
```

Это прямо закреплено в master specification. fileciteturn4file13

------------------------------------------------------------------------

# ROLE 10 --- HR ARCHITECT

HR должен видеть не «резюме».

Рабочая задача:

``` text
Need
→ required capability
→ evidence
→ candidate
→ relationship path
→ availability
→ hiring action
```

### Candidate card

``` text
Почему кандидат здесь?
✓ skill evidence
✓ relevant project
✓ verified result
✓ intent match
✓ relationship path

[Открыть доказательства]
[Пригласить]
```

------------------------------------------------------------------------

# ROLE 11 --- PROJECT / EVIDENCE ARCHITECT

Projects --- proof surfaces, не карточки CV. Это принцип master
specification. fileciteturn4file2

### Canonical chain

``` text
Project
→ Role
→ Contribution
→ Deliverable
→ Result
→ Evidence
→ Confirmation
```

### Код

``` ts
type EvidenceInput = {
  problem: string
  context?: string
  solution: string
  result: string
  visibility: EvidenceVisibility
}
```

### Нельзя

``` ts
project.status = 'COMPLETED'
evidence.status = 'VERIFIED'
```

автоматически.

Должно быть:

``` ts
project.status = 'COMPLETED'
evidence.status = 'UNVERIFIED'
```

и отдельное authenticated confirmation.

------------------------------------------------------------------------

# ROLE 12 --- SKILL GRAPH ARCHITECT

Навык нельзя считать доказанным потому, что пользователь его написал.

``` text
Skill Claim
↓
Evidence
↓
Project
↓
Contribution
↓
Result
↓
Recency
↓
Confidence
```

### UI

Вместо:

``` text
SCADA — Expert
```

показывать:

``` text
SCADA

Подтверждено:
3 проекта
2 результата
1 клиентское подтверждение

Последнее применение:
2026
```

------------------------------------------------------------------------

# ROLE 13 --- TRUST / SARAFAN RADIO ARCHITECT

Trust --- не public number.

Master principle:

> Trust is evidence-backed, not a public numeric score.
> fileciteturn4file2

### Recommendation

Не:

``` text
★★★★★
```

А:

``` text
Я работал с Анной в проекте X.
Она отвечала за Y.
Результат — Z.
```

### Sarafan recommendation schema

``` ts
type RecommendationInput = {
  recipientId: string
  relationshipType: string
  projectId?: string
  contribution?: string
  result?: string
  evidenceId?: string
  context: string
}
```

------------------------------------------------------------------------

# ROLE 14 --- 5 HANDSHAKES ARCHITECT

Пять рукопожатий должны отвечать на:

> «Как безопасно выйти на нужного человека?»

Путь:

``` text
Me
→ Contact A
→ Contact B
→ Contact C
→ Target
```

Но:

> discovery ≠ permission to expose private contact data.

Это прямо закреплено в master specification. fileciteturn4file2

### UI

Показывать:

``` text
Вы
 ↓
Анна
 ↓
Игорь

Общий профессиональный контекст:
Проект X
```

Не показывать private contact без consent.

------------------------------------------------------------------------

# ROLE 15 --- OPPORTUNITY ARCHITECT

Opportunity шире Jobs.

Типы:

``` text
JOB
PROJECT
PARTNERSHIP
TEAM
CLIENT
MENTOR
INVESTOR
SUPPLIER
```

### Opportunity card

``` text
What
Why now
Why you
Proof
Missing
Next action
```

------------------------------------------------------------------------

# ROLE 16 --- MATCHING / SEARCH ARCHITECT

Формула продукта:

``` ts
score =
  skills * 0.60 +
  trust * 0.25 +
  growth * 0.15
```

Но число не является продуктом.

Продуктом является explanation:

``` ts
type MatchExplanation = {
  matchedSkills: string[]
  evidence: EvidenceRef[]
  relationshipPath?: RelationshipRef
  intentReasons: string[]
  gaps: string[]
}
```

### UI

``` text
Почему показано?

✓ 4 релевантных навыка
✓ 2 подтверждённых проекта
✓ текущий intent совпадает
△ нет подтверждения X

[Посмотреть доказательства]
```

------------------------------------------------------------------------

# ROLE 17 --- NEWS / DISCOVER ARCHITECT

News Bridge:

``` text
source
→ article
→ summary
→ company
→ topic
→ skill
→ opportunity
```

Не хранить полный внешний материал без необходимости. Это прямой принцип
master specification. fileciteturn4file2

### Главное изменение

News не должна быть просто лентой.

Она должна отвечать:

> «Почему эта новость важна именно для моей профессиональной ситуации?»

------------------------------------------------------------------------

# ROLE 18 --- AI ARCHITECT

AI применяется там, где:

-   есть ambiguity;
-   semantics;
-   synthesis;
-   contextual reasoning;
-   natural language.

Не применять LLM там, где deterministic code достаточно. Это
обязательный принцип. fileciteturn4file2

### AI response contract

``` ts
type AIResult<T> = {
  answer: T
  sources: EvidenceRef[]
  confidence?: number
  assumptions: string[]
  generatedAt: string
  model: string
}
```

AI не должен выдавать утверждение без источника, если утверждение
относится к professional truth.

------------------------------------------------------------------------

# ROLE 19 --- AI AGENT / AUTONOMY ARCHITECT

Уже существующий Governance слой нужно не расширять бесконтрольно, а
связать с UI.

``` text
AI
 ↓
Action Registry
 ↓
Policy
 ↓
Authority
 ↓
Approval
 ↓
Execution
 ↓
Audit
```

Autonomy:

``` text
0 Advice
1 Collaborative
2 Delegated
3 Supervised
4 Bounded Autonomous
5 Process Autonomous
```

Но public production должен начинаться с controlled/manual modes.

Предыдущий аудит прямо рекомендует не запускать uncontrolled autopilot.
fileciteturn4file10

------------------------------------------------------------------------

# ROLE 20 --- HUMAN--MACHINE GOVERNANCE

Для каждого AI action:

``` ts
type GovernedAction = {
  actionKey: string
  risk: 'low' | 'medium' | 'high' | 'critical'
  reversible: boolean
  externalSideEffect: boolean
  requiresApproval: boolean
  lumenCost: number
}
```

### Decision Inbox

Пользователь видит:

``` text
Что?
Почему?
На каких данных?
Риск?
Стоимость?
Что изменится?
Можно ли отменить?
```

------------------------------------------------------------------------

# ROLE 21 --- SECURITY / DATA GUARD

Проверять не только endpoint security.

Проверять **field-level access**.

Пример:

``` ts
type ProfessionalProjection = {
  public: PublicProfile
  professional?: ProfessionalData
  private?: PrivateData
  hr?: HRData
}
```

Agent получает только необходимый projection.

Master specification использует принцип:

> Deep Knowledge, Controlled Access. fileciteturn4file13

------------------------------------------------------------------------

# ROLE 22 --- DATA / PRISMA ARCHITECT

Не добавлять model потому, что «будет удобно».

Каждая model:

``` text
owner
lifecycle
source
consumers
permissions
indexes
audit
retention
```

### Example

``` prisma
model ProfessionalEvidence {
  id         String @id @default(cuid())
  ownerId    String
  projectId  String?
  status     EvidenceStatus
  visibility EvidenceVisibility

  problem    String?
  context    String?
  solution   String?
  result     String?

  createdAt  DateTime @default(now())

  @@index([ownerId, status])
  @@index([projectId])
}
```

------------------------------------------------------------------------

# ROLE 23 --- BACKEND ARCHITECT

Canonical service:

``` text
Route
→ Validation
→ Authorization
→ Service
→ Transaction
→ Event
→ Audit
→ Response
```

Нельзя:

``` text
route → Prisma directly
```

если действие требует domain logic/governance.

------------------------------------------------------------------------

# ROLE 24 --- FRONTEND ARCHITECT

Frontend должен отражать domain, а не database.

Плохая архитектура:

``` tsx
<UserEverything />
```

Хорошая:

``` tsx
<ProfessionalHero />
<CurrentIntent />
<ProofSummary />
<RelevantProjects />
<Recommendations />
```

Каждый component должен иметь понятный user purpose.

------------------------------------------------------------------------

# ROLE 25 --- QA / E2E

Каждый критический путь:

``` text
click
→ request
→ server
→ database
→ event
→ visible result
```

Не принимать:

``` text
grep found component
```

как proof.

------------------------------------------------------------------------

# ROLE 26 --- RED TEAM

Минимум:

-   IDOR;
-   cross-company access;
-   private evidence leakage;
-   fake approval;
-   replay;
-   duplicate payment;
-   governance bypass;
-   AI prompt injection;
-   tool abuse;
-   audit tampering.

------------------------------------------------------------------------

# ROLE 27 --- PERFORMANCE / SRE

Уже есть metrics endpoint. fileciteturn4file0

Следующий уровень:

``` text
route
→ trace
→ DB query
→ external service
→ latency
```

Не оптимизировать без measurement.

------------------------------------------------------------------------

# ROLE 28 --- ECONOMICS / GROWTH

Primary KPI:

> **Verified Professional Value Created**

А не:

-   page views;
-   likes;
-   streak;
-   session length.

Master specification также перечисляет Evidence Coverage, Opportunity
Conversion, time-to-result, vacancy integrity и другие outcome-oriented
metrics. fileciteturn4file10

------------------------------------------------------------------------

# ROLE 29 --- ANTI-COMPLEXITY EDITOR

Эта роль получает право сказать:

> Удалить.

Каждый экран:

``` text
Keep
Move
Collapse
Remove
```

### Пример

Если профиль показывает:

``` text
15 metrics
8 counters
12 badges
27 skills
17 interests
```

а пользователь не может понять:

> «Что делать дальше?»

то задача не «улучшить CSS».

Задача:

> **удалить информационный шум.**

------------------------------------------------------------------------

# ROLE 30 --- RELEASE ARCHITECT

Final state:

``` text
CODE
+
PRODUCT
+
SECURITY
+
UX
+
DATA
+
AI
+
OBSERVABILITY
+
RECOVERY
=
PRODUCTION
```

------------------------------------------------------------------------

# 6. ГЛАВНАЯ РАБОТА СЕЙЧАС: ИНФОРМАЦИОННАЯ ХИРУРГИЯ

Не начинать с P2 visual polish.

Сначала разобрать следующие экраны:

1.  Home;
2.  Person Profile;
3.  Company Profile;
4.  Project;
5.  Opportunity;
6.  Job;
7.  Search;
8.  News;
9.  Discover;
10. Network;
11. AI;
12. Decision Inbox;
13. Work;
14. Store;
15. Settings.

Для каждого создать:

`SCREEN-TRUTH.md`

------------------------------------------------------------------------

# 7. НОВАЯ МАТРИЦА ЭКРАНА

Пример:

  ---------------------------------------------------------------------------------------
  Element        User question Decision        Action      Source           Keep?
  -------------- ------------- --------------- ----------- ---------------- -------------
  Name           Кто?          identity        open        User             YES
                                               profile                      

  Skills         Что умеет?    capability      inspect     Skill/Evidence   YES
                                               proof                        

  87 score       Насколько     unclear         none        heuristic        REMOVE
                 хорош?                                                     

  Projects       Что сделал?   fit             inspect     Project          YES
                                               project                      

  Intent         Чего хочет?   opportunity fit invite      Intent           YES

  Activity count Активен?      weak            none        Activity         HIDE

  Certificates   Есть ли       qualification   verify      Credential       CONDITIONAL
                 credential?                                                

  Connections    Насколько     weak            none        Relation         HIDE
  count          известен?                                                  
  ---------------------------------------------------------------------------------------

------------------------------------------------------------------------

# 8. HOME --- НОВАЯ АРХИТЕКТУРА

Главная не должна быть dashboard wall.

Цель:

> **показать 3--5 вещей, которые имеют значение сейчас.**

``` tsx
<Home>
  <ContextHeader />
  <NextBestAction />
  <RelevantOpportunities />
  <ImportantChanges />
  <PendingDecisions />
</Home>
```

------------------------------------------------------------------------

# 9. PERSON PROFILE --- НОВАЯ АРХИТЕКТУРА

``` tsx
<Profile>
  <Hero />
  <CurrentIntent />
  <ProofSummary />
  <RelevantCapabilities />
  <RelevantProjects />
  <Recommendations />
  <RelationshipPath />
  <Details />
</Profile>
```

## ProofSummary

``` tsx
<ProofSummary
  verifiedProjects={6}
  verifiedResults={4}
  clientConfirmations={2}
/>
```

Но значения должны приходить из БД, а не быть mocked.

------------------------------------------------------------------------

# 10. COMPANY PROFILE

``` tsx
<CompanyProfile>
  <CompanyIdentity />
  <WhatWeCreate />
  <CompanyTruth />
  <CurrentNeeds />
  <Projects />
  <OpenOpportunities />
  <People />
  <Recommendations />
  <News />
</CompanyProfile>
```

## CompanyTruth

``` text
CLAIM
VERIFIED
SOURCE
FRESHNESS
UNRESOLVED
```

Master specification прямо требует разделять claims и verified facts.
fileciteturn4file8

------------------------------------------------------------------------

# 11. OPPORTUNITY

Карточка должна отвечать:

``` text
WHAT
WHY NOW
WHY YOU
WHAT PROVES FIT
WHAT IS MISSING
WHAT NEXT
```

------------------------------------------------------------------------

# 12. PROJECT

Первый экран:

``` text
Outcome
Current stage
My role
My contribution
Evidence status
Next action
```

Остальные подробности --- drill-down.

------------------------------------------------------------------------

# 13. SEARCH

Natural language:

``` text
"Нужен инженер по РЗА для проекта на Сахалине"
```

Результат не просто:

``` text
People: 143
```

а:

``` text
3 highly relevant people

Why:
✓ 4 skills
✓ 2 similar projects
✓ one relationship path

[Open proof]
```

------------------------------------------------------------------------

# 14. NEWS

Карточка:

``` text
Headline
Source
Why it matters to you
Company
Potential opportunity
Original source
```

------------------------------------------------------------------------

# 15. ПЕРВЫЕ КОНКРЕТНЫЕ КОДОВЫЕ ИЗМЕНЕНИЯ

## TASK IA-001 --- создать InformationPurpose

``` ts
export type InformationPurpose = {
  id: string
  question: string
  decision: string
  action?: string
  source: string
  priority: 'primary' | 'secondary' | 'contextual'
}
```

Не обязательно хранить это в БД; на первом этапе это architecture
metadata.

------------------------------------------------------------------------

## TASK UX-001 --- убрать unexplained metrics

До:

``` tsx
<Metric label="Trust" value={trustScore} />
```

После:

``` tsx
<ProofSummary
  verifiedProjects={verifiedProjects}
  verifiedResults={verifiedResults}
  clientConfirmations={clientConfirmations}
/>
```

------------------------------------------------------------------------

## TASK MATCH-001 --- заменить magic score UI

До:

``` tsx
<Badge>{matchScore}%</Badge>
```

После:

``` tsx
<MatchExplanation
  matchedSkills={match.matchedSkills}
  evidence={match.evidence}
  intentReasons={match.intentReasons}
  gaps={match.gaps}
/>
```

------------------------------------------------------------------------

## TASK AI-001 --- AI source contract

``` ts
export type AIClaim = {
  text: string
  sourceIds: string[]
  confidence?: number
  generatedAt: string
}
```

Если sourceIds пуст:

``` ts
throw new Error('UNSUPPORTED_PROFESSIONAL_CLAIM')
```

для тех AI workflows, которые утверждают professional truth.

------------------------------------------------------------------------

# 16. P0 CURRENT RELEASE TASKS

В первую очередь закрыть существующие gaps:

### D-01

Intro UI.

### D-02

Intro inbox.

### D-03

Complaint UI.

### D-04

Company vacancy creation.

### D-05

Credential export.

Это прямо следует из внешнего аудита текущей версии.
fileciteturn4file0

Но делать их не изолированно.

Каждая задача должна пройти:

``` text
UX purpose
→ UI
→ API
→ DB
→ permission
→ event
→ audit
→ E2E
→ screenshot
```

------------------------------------------------------------------------

# 17. P1 CURRENT RELEASE TASKS

После P0:

1.  relationship UI;
2.  opportunity related navigation;
3.  ML terminology correction;
4.  independent benchmark;
5.  FPR evaluation;
6.  truthful web-push status;
7.  outbox producer completeness;
8.  sitemap/UI route integrity;
9.  QA-data isolation;
10. demo credential removal.

------------------------------------------------------------------------

# 18. ЧТО НЕ ДЕЛАТЬ СЕЙЧАС

Не делать:

-   full Digital Twin;
-   GPT-1000 infrastructure;
-   10B-user infrastructure;
-   50 AI agents;
-   full token economy;
-   tTRUST;
-   AR/VR;
-   giant AI dashboard;
-   leaderboard;
-   XP;
-   streaks;
-   новый framework ради framework;
-   rewrite всего frontend.

Master specification также прямо относит многие из этих идей к
future/source-only, а масштабирование должно происходить по измеренной
необходимости. fileciteturn4file15turn4file18

------------------------------------------------------------------------

# 19. 12 ИТЕРАЦИЙ --- НОВАЯ ПОСЛЕДОВАТЕЛЬНОСТЬ

## I1 --- Truth

`repo → docs → runtime → DB`

Output:

``` text
REPO_TRUTH
TRACEABILITY
CONFLICT_REGISTER
```

## I2 --- Information Architecture

15 ключевых экранов.

Output:

``` text
SCREEN-TRUTH
INFORMATION-MATRIX
REMOVE/HIDE list
```

## I3 --- P0 UI Doors

D-01...D-05.

## I4 --- Professional Graph

Person / Company / Project / Evidence / Skill / Intent.

## I5 --- Opportunity

Intent / Search / Match / Explanation.

## I6 --- Trust

Evidence / recommendations / 5 handshakes.

## I7 --- Work

Project / Contribution / Deliverable / Result.

## I8 --- AI

AI Core / explainability / sources.

## I9 --- Governance

Action Registry / Policy / Decision Inbox / Audit / Kill.

## I10 --- Security/Data Guard

field-level permissions / tenant isolation / privacy.

## I11 --- QA/SRE

E2E / red team / metrics / recovery.

## I12 --- Release

Production gate / closed pilot.

------------------------------------------------------------------------

# 20. КОНТРОЛЬНАЯ ТОЧКА ПОСЛЕ КАЖДОЙ ИТЕРАЦИИ

Нельзя писать:

``` text
DONE
```

Нужно:

``` md
## ITERATION RESULT

Changed:
...

User outcome:
...

Files:
...

API:
...

DB:
...

Tests:
...

Security:
...

Screenshots:
...

Before:
...

After:
...

Remaining:
...

Independent verification:
PASS / FAIL
```

------------------------------------------------------------------------

# 21. ЧТО ЗНАЧИТ «СДЕЛАНО ПРАВИЛЬНО»

Feature считается VERIFIED только когда:

### Product

-   пользователь понимает зачем она;
-   есть следующий meaningful action;
-   она связана с professional outcome.

### Information

-   каждый показанный блок имеет purpose;
-   нет дублирования;
-   нет irrelevant information;
-   sensitive information скрыта.

### Architecture

-   используется canonical graph;
-   нет duplicate source of truth;
-   reuse существующего кода проверен.

### Backend

-   validation;
-   authorization;
-   service;
-   transaction;
-   audit/event если требуется.

### Frontend

-   loading;
-   empty;
-   error;
-   success;
-   mobile;
-   keyboard/accessibility.

### AI

-   source;
-   policy;
-   cost;
-   audit;
-   no fabricated truth.

### Security

-   IDOR test;
-   permission test;
-   tenant isolation;
-   privacy test.

### QA

-   unit;
-   integration;
-   E2E;
-   adversarial.

### Production

-   metrics;
-   rollback;
-   backup/restore;
-   migration tested.

------------------------------------------------------------------------

# 22. KPI НОВОГО ЭТАПА

Не:

``` text
number of screens
number of APIs
number of models
```

А:

## Primary

**Verified Professional Value Created**

## Secondary

-   Evidence Coverage;
-   Opportunity Conversion;
-   time-to-result;
-   project verification rate;
-   recommendation quality;
-   match engagement;
-   vacancy integrity;
-   company response rate;
-   AI approval/rejection rate;
-   AI cost per successful outcome.

Эти outcome-oriented KPI соответствуют Master Specification.
fileciteturn4file10

------------------------------------------------------------------------

# 23. CTO ACCEPTANCE QUESTIONS

Перед тем как принять любую новую страницу:

1.  Кто её пользователь?
2.  В каком состоянии он её открывает?
3.  Какой вопрос у него в голове?
4.  Что он должен понять за 5 секунд?
5.  Какая информация необходима?
6.  Что можно убрать?
7.  Какой факт доказывает показанное?
8.  Какое действие следует?
9.  Что произойдёт после действия?
10. Можно ли объяснить AI-рекомендацию?
11. Кто имеет право видеть данные?
12. Как проверить, что действие реально произошло?
13. Как отменить действие?
14. Что произойдёт при ошибке?
15. Что будет на 390px?
16. Какой E2E тест доказывает работу?
17. Какой red-team сценарий может её сломать?
18. Что измеряем после запуска?
19. Какой реальный профессиональный результат улучшится?
20. Можно ли удалить половину этого экрана?

Если на последний вопрос ответ «да» --- сначала удаляем.

------------------------------------------------------------------------

# 24. ПЕРВЫЙ HANDOFF ДЛЯ Z.AI

Z.ai получает этот документ и обязан сначала вернуть:

``` text
1. REPO_TRUTH_REPORT.md
2. BIZON_TRACEABILITY_MATRIX.md
3. BIZON_CONFLICT_REGISTER.md
4. BIZON_SCREEN_INFORMATION_MATRIX.md
5. BIZON_UI_DEAD_DOORS.md
6. BIZON_IMPLEMENTATION_PLAN.md
```

**До этих шести документов production code не переписывать.**

После этого:

``` text
D-01
→ verify
→ D-02
→ verify
→ D-03
→ verify
→ D-04
→ verify
→ D-05
→ verify
```

------------------------------------------------------------------------

# 25. ФОРМАТ ЗАДАЧИ ДЛЯ Z.AI

``` text
TASK ID: UX-001

TITLE:
Reduce Profile Information Noise

USER PROBLEM:
User sees too much information and cannot understand
why this person is relevant.

CURRENT:
[exact file:line]

REMOVE:
- public numeric trust
- irrelevant activity counters
- duplicated statistics

KEEP:
- identity
- current intent
- relevant skills
- verified proof
- projects
- recommendations
- primary action

NEW UI:
<ProfileHero />
<CurrentIntent />
<ProofSummary />
<RelevantCapabilities />
<RelevantProjects />
<Recommendations />
<PrimaryActions />

DATA:
Only real DB-derived values.

API:
Reuse existing profile endpoints where possible.

SECURITY:
Private evidence must not leak.

TEST:
- desktop
- 390px
- empty
- loading
- error
- E2E
- authorization

ACCEPTANCE:
A new user can answer:
1. Who is this?
2. What can they do?
3. What proves it?
4. What do they want now?
5. What can I do next?

EVIDENCE:
Screenshot + network trace + test output + file:line.
```

------------------------------------------------------------------------

# 26. ФИНАЛЬНЫЙ CTO ВЕРДИКТ ПО ТЕКУЩЕМУ НАПРАВЛЕНИЮ

BizON сейчас не нуждается прежде всего в ещё сотнях функций.

Он нуждается в **конвертации существующего технического капитала в
профессиональный результат пользователя**.

У нас уже есть значительная инфраструктура:

-   governance;
-   projects;
-   evidence;
-   opportunity;
-   store;
-   news;
-   observability;
-   matching;
-   authentication;
-   services;
-   APIs. fileciteturn4file0

Теперь необходимо сделать три вещи одновременно:

## 1. CLOSE THE DOORS

Все существующие полезные backend capabilities должны получить реальные
пользовательские пути.

## 2. REDUCE THE NOISE

Информация должна появляться не потому, что её можно показать, а потому
что она помогает принять профессиональное решение.

## 3. CONNECT THE GRAPH

``` text
WHO
→
WHAT
→
PROOF
→
INTENT
→
OPPORTUNITY
→
ACTION
→
RESULT
```

------------------------------------------------------------------------

# 27. ФИНАЛЬНЫЙ ПРИНЦИП

> **BizON не должен показывать пользователю всё, что он знает.**
>
> **BizON должен показывать ровно то, что помогает человеку сделать
> следующий правильный профессиональный шаг --- и дать возможность
> открыть доказательство, если пользователь хочет проверить основание.**

Это и есть переход от:

**«социальной сети с большим количеством функций»**

к:

**Professional Intelligence Platform.**

------------------------------------------------------------------------

# 28. STATUS

Этот документ является execution direction, но не заменяет Repository
Truth.

Первый обязательный шаг:

``` text
REPOSITORY TRUTH
+
SCREEN INFORMATION AUDIT
+
P0 UI DOORS
```

Только после их подтверждения разрешается следующий слой архитектурных
изменений.

------------------------------------------------------------------------

# 29. ZERO-LOSS REFERENCES

Этот документ сохраняет и использует текущие формулировки Master
Specification:

-   North Star: real action → proof → skills → intelligence → intent →
    opportunity → relationship → result → new proof.
    fileciteturn4file2
-   Evidence over claims / Results over vanity / Intelligence ≠
    Authority / Human Root of Trust. fileciteturn4file2
-   Person / Company / HR как отдельные продукты. fileciteturn4file13
-   Professional Passport. fileciteturn4file12
-   Job/Opportunity intent flow. fileciteturn4file7
-   Company Truth. fileciteturn4file8
-   Governance and controlled autonomy. fileciteturn4file10
-   Lost Thought Map / zero-loss preservation. fileciteturn4file15
