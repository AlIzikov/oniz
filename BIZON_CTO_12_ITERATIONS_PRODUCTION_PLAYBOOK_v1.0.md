# BizON --- CTO 12-итерационный Production Playbook

## Полный аудит → исправление → UX/UI → архитектура → код → тесты → Production Gate

**Версия документа:** 1.0\
**Дата:** 20.09.2026\
**Исходный снапшот:** `BizON_v1.2.0-pilot-platform_audit-20260919`\
**Назначение:** дать исполнителю и следующим аудиторам единый,
исполняемый план работы над BizON и требовать доказательства каждого
результата.

------------------------------------------------------------------------

# 0. РЕЖИМ РАБОТЫ CTO

Ты действуешь как автономный CTO и ведущий архитектор BizON.

Но «автономный» означает не «делать всё подряд», а:

> самостоятельно принимать технические решения в пределах утверждённой
> архитектуры, политики безопасности и продуктовой идеологии, выполнять
> независимые проверки и останавливать опасные изменения.

Главное правило:

> **DONE ≠ VERIFIED.**

Наличие модели, API, сервиса или кнопки не является доказательством
готовности.

Готово только тогда, когда доказана полная цепочка:

`Идея → пользовательская задача → данные → модель → сервис → API → UI → действие → результат → тест → доказательство`.

------------------------------------------------------------------------

# 1. ИСХОДНОЕ СОСТОЯНИЕ 19.09.2026

По предоставленному отчёту:

-   План-v2: 155/155 шагов отмечены DONE.
-   Независимая проверка: 225 pass / 0 fail по 22 тест-файлам.
-   TSC: 0 ошибок.
-   ESLint: 0/0.
-   Backend ядро шести волн в основном работает.
-   W2 Store и W4 Project Room получили READY.
-   Governance ядро W6 работает.
-   Живой браузерный аудит выявил примерно 70% READY, около 25%
    PARTIAL/DOC-MISMATCH и остаточные MOCK/риски.
-   Главный разрыв: backend «есть», но некоторые пользовательские
    «двери» отсутствуют.
-   Тестовые данные загрязняют живую ленту.
-   Некоторые ML-названия создают впечатление реального ML, хотя внутри
    используются эвристики.
-   Часть UI-гейтов проверяла наличие строки в исходнике вместо реальной
    UI→API→DB→результат проводки.

Критические выявленные разрывы:

1.  Intro backend существует, но полноценный пользовательский путь из
    профиля не доведён.
2.  UI создания вакансии для компании отсутствует при наличии API.
3.  Complaint UI показывает fake success вместо POST `/api/complaints`.
4.  VC/W3C export API есть, UI-двери нет.
5.  `/api/me/relationship/[userId]` не имеет полноценного UI consumer.
6.  Opportunity `/related` существует без нормальной навигации.
7.  Некоторые ML labels не соответствуют фактической реализации.
8.  Precision@Top3 основан на том же сигнале, которым строится ranking.
9.  FPR eval не оформлен как воспроизводимый сохранённый benchmark.
10. Web-push при настроенных ключах может вернуть успешный результат без
    реальной отправки.
11. В live feed есть QA/E2E тестовые записи.
12. Demo credentials нельзя показывать публично в production.
13. UI содержит семантически неверные элементы для company profile.
14. Outbox имеет инфраструктуру, но не все необходимые producer paths.
15. SEO sitemap указывает на сущности, для которых нет соответствующих
    UI routes.
16. 6/9 гейтов волн слишком сильно зависели от grep.
17. Исполнитель сам ставил DONE без независимой приемки.
18. UI acceptance не всегда требовала сохранённого браузерного
    доказательства.

------------------------------------------------------------------------

# 2. НЕПОДВИЖНЫЕ ПРИНЦИПЫ BIZON

## 2.1. BizON не является клоном LinkedIn

BizON --- профессиональная инфраструктура:

`Person ↔ Company ↔ Project ↔ Skill ↔ Evidence ↔ Trust ↔ Intent ↔ Opportunity ↔ Result`

Главная петля:

`Action → Proof → Skill → Development → Intent → Market → Opportunity → Result → Capital`

AI находится НАД профессиональным графом, а не заменяет его.

------------------------------------------------------------------------

# 3. ПРАВИЛО ИНФОРМАЦИОННОЙ ДИЕТЫ

Каждый элемент UI обязан пройти четыре вопроса:

1.  Зачем он пользователю?
2.  Какое решение он помогает принять?
3.  Какое действие после него возможно?
4.  Почему пользователь должен видеть его именно сейчас?

Если нет хорошего ответа:

`HIDE → COLLAPSE → MOVE → REMOVE`.

Нельзя показывать данные только потому, что они существуют в Prisma.

## Пример

Плохо:

``` tsx
{user.skills.map(...)}
{user.projects.map(...)}
{user.certificates.map(...)}
{user.connections.map(...)}
{user.activities.map(...)}
{user.statistics.map(...)}
```

Хорошо:

``` tsx
<ProfessionalProfile>
  <Identity />
  <CurrentIntent />
  <ProofSummary />
  <KeyCapabilities />
  <RelevantExperience />
  <Recommendations />
  <PrimaryActions />
  <Details />
</ProfessionalProfile>
```

Порядок:

`Кто → что умеет → что доказано → чего хочет → что делать дальше`.

------------------------------------------------------------------------

# 4. 21 ЭКСПЕРТНАЯ ЛИНЗА

Каждое существенное изменение проверяется по:

1.  Идеология.
2.  Product Strategy.
3.  Product Architecture.
4.  System Architecture.
5.  Information Architecture.
6.  UX.
7.  UI/Visual Design.
8.  Behaviour Design.
9.  Data Architecture.
10. Professional Graph.
11. AI Architecture.
12. Agent/Autonomy Architecture.
13. Human--Machine Governance.
14. Security.
15. Privacy/Data Guard.
16. Backend Engineering.
17. Frontend Engineering.
18. Performance/Scale.
19. QA/Red Team.
20. Anti-Complexity.
21. Real User Result.

------------------------------------------------------------------------

# 5. ВАЖНОЕ УТОЧНЕНИЕ ПО «СОВРЕМЕННЫМ ФИЧАМ»

Запрос «внедрить AI, real-time, аналитику и геймификацию» не означает
автоматически добавлять всё это.

BizON уже имеет правило:

-   нет публичного numeric reputation;
-   нет leaderboard;
-   нет XP;
-   нет streaks;
-   нет механики, где человек играет ради баллов вместо
    профессионального результата.

Поэтому **классическая геймификация запрещена**, если она противоречит
этой идеологии.

Вместо неё развиваем:

-   профессиональный прогресс;
-   доказательства;
-   понятные next-best-actions;
-   project outcomes;
-   trust;
-   opportunity conversion;
-   AI-assisted work;
-   collaboration.

------------------------------------------------------------------------

# 6. АРХИТЕКТУРНЫЙ BASELINE

Текущий заявленный стек:

-   Next.js 16 App Router.
-   React 19.
-   TypeScript 5 strict.
-   Tailwind 4.
-   shadcn/ui New York.
-   Prisma.
-   SQLite для текущего pilot.
-   Zustand.
-   TanStack Query.
-   next-themes.
-   z-ai-web-dev-sdk только backend.
-   Socket.IO для realtime mini-services.
-   Bun.
-   Zod.
-   Playwright.
-   ESLint.

Не менять стек ради моды.

Перед добавлением новой зависимости:

`existing code → stdlib → existing package → new package only if justified`.

------------------------------------------------------------------------

# 7. 12 ИТЕРАЦИЙ

12 итераций --- это не обещание, что физически любой объём production
work будет выполнен за 12 часов. Это **12 последовательных
CTO-контуров**, каждый из которых имеет вход, работу, доказательства и
gate.

------------------------------------------------------------------------

# ITERATION 1 --- REPOSITORY TRUTH

## Цель

Установить фактическое состояние репозитория.

## Проверить

``` bash
bun --version
bun install
bunx tsc --noEmit
bun run lint
bun test
git status
git log --oneline -20
git branch -a
```

Составить:

`docs/CTO/01-REPOSITORY-TRUTH.md`

## Найти

``` bash
find src -type f
find src/app/api -type f
find src/lib/services -type f
find src/components -type f
grep -R "TODO\|FIXME\|mock\|fake\|Math.random" src
```

Но grep используется только как discovery, не как acceptance.

## Реестр

Для каждой фичи:

  ID   Feature   Model   Service   API   UI   Test   Status
  ---- --------- ------- --------- ----- ---- ------ --------

## Acceptance

Нельзя начинать крупный refactor до появления Repository Truth.

------------------------------------------------------------------------

# ITERATION 2 --- SECURITY + DATA GUARD

## Проверить

-   authentication;
-   authorization;
-   session invalidation;
-   CSRF;
-   rate limiting;
-   IDOR;
-   privilege escalation;
-   injection;
-   secrets;
-   PII;
-   consent;
-   tenant isolation;
-   audit;
-   AI prompt injection;
-   unsafe tool execution.

## Пример authorization

``` ts
function assertProjectMember(
  project: { ownerId: string },
  userId: string
) {
  if (project.ownerId !== userId) {
    throw new Error('FORBIDDEN');
  }
}
```

Но лучше централизовать:

``` ts
await accessControl.require({
  userId: ctx.user.id,
  resource: 'project',
  resourceId: projectId,
  action: 'read',
});
```

## Нельзя

``` ts
if (session) {
  return db.project.findUnique({ where: { id } });
}
```

Нужно:

``` ts
const project = await db.project.findUnique({ where: { id } });

await accessControl.require({
  userId: ctx.user.id,
  resource: 'project',
  resourceId: project.id,
  action: 'read',
});
```

## Security tests

Создать adversarial cases:

-   user A reads user B private evidence;
-   company A edits company B vacancy;
-   revoked session accesses API;
-   AI tries prohibited action;
-   malformed IDs;
-   rate-limit bypass;
-   duplicate idempotency key;
-   replayed approval;
-   forged governance payload.

------------------------------------------------------------------------

# ITERATION 3 --- CORE DATA / PROFESSIONAL GRAPH

Проверить, что данные не просто существуют, а образуют причинную
цепочку.

## Основная модель

``` text
Person
  ↓
Intent
  ↓
Opportunity / Project
  ↓
Contribution
  ↓
Deliverable
  ↓
Result
  ↓
Evidence
  ↓
Skill
  ↓
Recommendation
  ↓
Trust
```

## Новые поля нельзя добавлять без use-case.

Плохое:

``` prisma
model User {
  aiMagicScore Float?
}
```

Хорошее:

``` prisma
model ProfessionalEvidence {
  id          String   @id @default(cuid())
  projectId   String?
  ownerId     String
  claimId     String?
  visibility  EvidenceVisibility
  status      EvidenceStatus
  problem     String?
  context     String?
  solution    String?
  result      String?
  createdAt   DateTime @default(now())
}
```

## Evidence принцип

`Claim ≠ Proof`.

`Project completed ≠ Client verified`.

------------------------------------------------------------------------

# ITERATION 4 --- UX/UI INFORMATION SURGERY

Это одна из самых важных итераций.

## Основные страницы

Проверить:

-   Dashboard;
-   Profile;
-   Company;
-   Projects;
-   Project Room;
-   Jobs;
-   Opportunity;
-   Search;
-   Network;
-   Messages;
-   News;
-   Store;
-   Settings;
-   AI/Career;
-   Trust Graph.

Для каждой страницы создать таблицу:

  --------------------------------------------------------------------------
  Screen      User intent Must         Primary     Secondary   Hide/remove
                          understand   action      info        
  ----------- ----------- ------------ ----------- ----------- -------------

  --------------------------------------------------------------------------

## Dashboard

Не показывать 20 метрик.

Главный вопрос:

> «Что мне сейчас важно сделать?»

Структура:

``` text
Morning / Daily Brief
        ↓
3 important signals
        ↓
Next Best Action
        ↓
Relevant Opportunities
        ↓
Proof / Trust only when useful
```

## Profile

Первый экран:

``` text
Identity
Current Intent
Capabilities
Proof Summary
Relevant Results
Primary Actions
```

Подробности --- ниже.

## Company

Не показывать человеку:

-   birthday;
-   персональные блоки;
-   нерелевантные user-only поля.

Компания должна отвечать:

`Кто компания → что делает → что подтверждено → кого ищет → какие проекты → какие возможности`.

------------------------------------------------------------------------

# ITERATION 5 --- CLOSE THE DOORS: UI → API

Это P0 текущего аудита.

## D-01 Intro

API уже существует:

``` text
POST /api/intros
```

Нужно добавить UI.

Пример:

``` tsx
<Button onClick={handleRequestIntro}>
  Запросить знакомство
</Button>
```

``` ts
async function handleRequestIntro() {
  await api('/api/intros', {
    method: 'POST',
    body: JSON.stringify({
      targetId: data.user.id,
      purpose: 'project',
      message,
    }),
  });
}
```

Но перед отправкой показать:

``` text
Вы
 ↓
Анна
 ↓
Игорь
```

И объяснить:

> Анна --- ваш профессиональный контакт с Игорем.

После отправки:

`POST → DB → notification → introducer inbox`.

## D-02 Intro Inbox

Проверить:

``` text
GET /api/intros?box=incoming
POST /api/intros/:id/accept
POST /api/intros/:id/decline
```

UI должен реально вызывать API.

## D-03 Complaints

Текущий антипример:

``` tsx
toast({ title: 'Жалоба отправлена' });
```

без API вызова.

Правильно:

``` ts
await api('/api/complaints', {
  method: 'POST',
  body: JSON.stringify({
    targetId,
    category,
    reason,
  }),
});
```

Только после успешного HTTP response:

``` ts
toast({ title: 'Жалоба принята' });
```

## D-04 Vacancy creation

Для company profile:

``` tsx
<Button onClick={() => setView('jobs-create')}>
  Разместить вакансию
</Button>
```

POST:

``` ts
await api('/api/jobs', {
  method: 'POST',
  body: JSON.stringify(payload),
});
```

Проверить:

`company → create → validate → DB → vacancy visible → candidate can apply`.

## D-05 Credentials export

Настройки:

``` text
Профессиональный паспорт
[Экспортировать подтверждения]
```

API:

``` text
GET /api/me/credentials
```

Но UI должен показать:

-   что экспортируется;
-   какие credentials;
-   формат;
-   дату;
-   приватность;
-   результат загрузки.

------------------------------------------------------------------------

# ITERATION 6 --- PROJECTS / EVIDENCE / RESULTS

Project Room должен быть центральным доказательством работы.

## Поток

``` text
Project
 ↓
Role
 ↓
Contribution
 ↓
Deliverable
 ↓
Result
 ↓
Evidence
 ↓
Client confirmation
```

## Поля

``` ts
type ProjectEvidenceInput = {
  problem: string
  context?: string
  solution: string
  result: string
  visibility: 'PUBLIC' | 'PROFESSIONAL' | 'PRIVATE' | 'CONFIDENTIAL'
}
```

## Нельзя

``` ts
project.status = 'COMPLETED'
```

и автоматически считать проект подтверждённым.

Нужно:

``` ts
project.status = 'COMPLETED'
evidence.status = 'UNVERIFIED'
```

После подтверждения клиента:

``` ts
evidence.status = 'VERIFIED'
evidence.confirmedBy = clientId
```

------------------------------------------------------------------------

# ITERATION 7 --- OPPORTUNITY + MATCHMAKER

## Match

Текущая продуктовая формула:

``` ts
score =
  skills * 0.60 +
  trust * 0.25 +
  growth * 0.15
```

Но score должен быть explainable.

``` ts
type MatchExplanation = {
  skills: {
    score: number
    reasons: string[]
  }
  trust: {
    score: number
    reasons: string[]
  }
  growth: {
    score: number
    reasons: string[]
  }
}
```

UI:

``` text
Почему BizON рекомендует это?

✓ 4 совпадающих ключевых навыка
✓ 2 подтверждённых проекта
✓ есть общий профессиональный контакт
✓ ваш текущий intent совпадает

[Посмотреть доказательства]
```

Не показывать пользователю бессмысленный:

``` text
Match: 87%
```

если невозможно объяснить происхождение числа.

------------------------------------------------------------------------

# ITERATION 8 --- AI CORE + HONEST AI

## Правило

AI не создаёт факты BizON.

AI может:

-   искать;
-   суммировать;
-   объяснять;
-   ранжировать на основе существующих данных;
-   готовить черновики;
-   предлагать действия.

AI не может:

-   придумывать доказательства;
-   подтверждать клиента;
-   повышать trust;
-   менять юридические данные;
-   тратить деньги;
-   отправлять high-risk actions без governance.

## Action Registry

Пример:

``` ts
REQUEST_INTRO: {
  key: 'REQUEST_INTRO',
  risk: 'medium',
  reversible: true,
  externalSideEffect: true,
  defaultModes: NO_AUTOPILOT,
  title: 'Запросить знакомство через посредника',
  lumenCost: 0,
}
```

## Policy

``` ts
const decision = await PolicyEngine.evaluateAction({
  actionKey: 'REQUEST_INTRO',
  mode,
  authorityActions,
  userId,
})

if (!decision.allowed) {
  throw new Error(decision.reason)
}

if (decision.requiresApproval) {
  return createDecisionInboxItem(...)
}
```

------------------------------------------------------------------------

# ITERATION 9 --- HUMAN--MACHINE MANAGEMENT

## Autonomy

Использовать уровни:

``` text
0 — Advice
1 — Collaborative
2 — Delegated
3 — Supervised
4 — Bounded Autonomous
5 — Process Autonomy
```

В текущей реализации modes должны оставаться серверно ограниченными.

## Decision Inbox

Каждое значимое действие:

``` text
Что AI хочет сделать?
Почему?
Какие данные использованы?
Какой риск?
Сколько стоит?
Можно ли отменить?
Что произойдёт после подтверждения?
```

Пример:

``` text
AI предлагает:

Отправить Анне сообщение
Причина: найден проект, соответствующий вашему Intent
Риск: HIGH
Стоимость: 0 L
Изменение внешнего мира: ДА

[Разрешить один раз]
[Отклонить]
[Запретить это действие]
```

## Kill switch

Проверка должна находиться вне AI-контура:

``` ts
if (PolicyEngine.isAutonomyKilled()) {
  throw new Error('AI_AUTONOMY_KILLED')
}
```

Kill switch не должен зависеть от самого агента, которого он должен
остановить.

------------------------------------------------------------------------

# ITERATION 10 --- PERFORMANCE / OBSERVABILITY / SCALE

## Проверить

-   RPS;
-   p50;
-   p95;
-   p99;
-   DB latency;
-   cache hit;
-   queue depth;
-   memory;
-   CPU;
-   error rate;
-   circuit breakers.

Уже существует:

``` text
GET /api/metrics
GET /api/metrics?format=json
scripts/grafana-dashboard.json
```

## Не оптимизировать вслепую

Плохо:

``` ts
const cached = ...
```

просто потому что «cache нужен».

Сначала:

`measure → identify bottleneck → optimize → benchmark`.

## DB

Проверить индексы:

``` prisma
@@index([ownerId])
@@index([createdAt])
@@index([projectId, status])
```

И проверять реальные query plans.

------------------------------------------------------------------------

# ITERATION 11 --- QA / RED TEAM / E2E

Каждый критический путь должен иметь E2E.

## Минимальные сценарии

### Registration

``` text
landing
→ register
→ session
→ dashboard
```

### Profile

``` text
search person
→ profile
→ evidence
→ intent
→ intro
```

### Company

``` text
company
→ create vacancy
→ vacancy appears
→ candidate applies
```

### Project

``` text
create project
→ add member
→ contribution
→ evidence
→ client verification
```

### AI

``` text
AI request
→ governance
→ approval if required
→ execution
→ audit
```

### Complaint

``` text
menu
→ complaint dialog
→ category
→ POST
→ persisted complaint
```

## Browser acceptance

Нельзя считать:

``` text
grep "IntroInbox"
```

доказательством UI.

Нужно:

``` text
browser
→ click
→ network request
→ HTTP 2xx
→ DB state changed
→ visible result
```

------------------------------------------------------------------------

# ITERATION 12 --- PRODUCTION RELEASE GATE

Финальный gate должен проверять:

## Code

-   TypeScript 0.
-   ESLint 0.
-   tests pass.
-   no forbidden `any`.
-   no silent catch.
-   no fake success.
-   no dead feature flags.
-   no accidental secrets.

## Product

-   primary user journeys complete.
-   no critical dead-end.
-   no orphan API for critical feature.
-   no misleading UI.

## Security

-   auth.
-   authorization.
-   rate limit.
-   injection.
-   IDOR.
-   AI governance.
-   kill switch.
-   audit.

## Data

-   no QA contamination.
-   no fake truth.
-   no accidental PII exposure.

## UX

-   mobile 390px.
-   touch targets ≥44px.
-   loading.
-   empty.
-   error.
-   success.
-   accessibility.

## Performance

-   p95 measured.
-   critical endpoints benchmarked.
-   no obvious N+1.

## Release

``` text
P0 = 0
P1 critical = 0
fake success = 0
DOC-MISMATCH critical = 0
unverified production-critical feature = 0
```

------------------------------------------------------------------------

# 8. ПРАВИЛО ДЛЯ КАЖДОГО ИЗМЕНЕНИЯ

Каждая задача оформляется:

``` md
## TASK

### Objective
...

### User problem
...

### Current truth
...

### Files
- ...

### Data
- ...

### API
- ...

### UI
- ...

### Security
- ...

### Tests
- ...

### Acceptance
- ...

### Evidence
- screenshot
- HTTP request
- DB assertion
- test output
```

------------------------------------------------------------------------

# 9. ПРИМЕР ПРАВИЛЬНОЙ ЗАДАЧИ ДЛЯ Z.AI

``` text
TASK: D-01 Intro UI

DO NOT MARK DONE UNTIL:
1. чужой профиль показывает путь знакомства;
2. пользователь нажимает «Запросить знакомство»;
3. открывается форма purpose/message;
4. POST /api/intros получает реальные данные;
5. DB IntroductionRequest создаётся;
6. посредник видит request в inbox;
7. посредник может accept/decline;
8. после accept target/requester получает следующий доступ согласно policy;
9. E2E тест проходит;
10. сохранён screenshot;
11. network trace показывает POST /api/intros;
12. нет fake success.

FILES TO INSPECT FIRST:
- src/components/profile/profile-view.tsx
- src/components/trust/intro-inbox.tsx
- src/app/api/intros/route.ts
- src/lib/services/intro-service.ts
- src/lib/ai-governance/action-registry.ts
- src/lib/ai-governance/policy-engine.ts

DO NOT:
- create duplicate intro service;
- bypass governance;
- invent trust score;
- add public reputation number.
```

------------------------------------------------------------------------

# 10. КОНКРЕТНЫЕ КОДОВЫЕ ШАБЛОНЫ

## 10.1 API-first

``` ts
export const POST = withRoute(
  async (req, ctx) => {
    const input = schema.parse(ctx.data)

    await accessControl.require({
      userId: ctx.user.id,
      resource: 'resource',
      resourceId: input.id,
      action: 'write',
    })

    const result = await service.execute(input, ctx.user.id)

    return NextResponse.json(result, { status: 201 })
  },
  {
    rateLimit: {
      key: 'resource-post',
      limit: 30,
      windowMs: 60 * 60 * 1000,
    },
  },
)
```

## 10.2 Реальный UI success

Неправильно:

``` ts
toast({ title: 'Готово' })
```

Правильно:

``` ts
try {
  const result = await api('/api/resource', {
    method: 'POST',
    body: JSON.stringify(payload),
  })

  toast({
    title: 'Готово',
    description: result.message,
  })
} catch (error) {
  toast({
    title: 'Не удалось выполнить действие',
    description: getErrorMessage(error),
    variant: 'destructive',
  })
}
```

## 10.3 Loading / Empty / Error

``` tsx
if (loading) {
  return <ProfileSkeleton />
}

if (error) {
  return <ErrorState onRetry={loadProfile} />
}

if (!data.items.length) {
  return (
    <EmptyState
      title="Пока ничего нет"
      description="Добавьте первый проект, чтобы здесь появились результаты."
      action={<Button onClick={createProject}>Создать проект</Button>}
    />
  )
}
```

------------------------------------------------------------------------

# 11. PROFILE REDESIGN --- КОНКРЕТНАЯ ЦЕЛЕВАЯ СТРУКТУРА

Текущий `ProfileView` содержит много данных и часть legacy-концепций.

Новая структура:

``` tsx
<ProfileHeader />
<CurrentIntentCard />
<ProofSummary />
<KeyCapabilities />
<RelevantProjects />
<Recommendations />
<ProfessionalConnections />
<ProfileDetails />
```

## Header

``` text
Имя
Роль
Локация
Verification status
Primary action
```

## Current Intent

``` text
Сейчас ищет:
Проект / партнёр / клиент / работа

Когда:
Сейчас / месяц / дата

Ограничения:
Удалённо / регион / бюджет / отрасль
```

## Proof

Не:

``` text
Trust 87
```

А:

``` text
8 подтверждённых проектов
5 подтверждённых результатов
3 рекомендации по результатам работы
```

## Skills

Показывать только наиболее релевантные текущей задаче.

Остальные:

``` text
+ 12 навыков
```

## Projects

Показывать результаты, а не только названия.

``` text
Проект
Роль
Что сделал
Результат
Подтверждение
```

------------------------------------------------------------------------

# 12. COMPANY PROFILE

Структура:

``` text
Company Identity
What we do
Verified facts
Current needs
Open opportunities
Projects / results
People
Professional recommendations
Company evidence
```

Не показывать company как человека.

Никаких:

``` text
birthday
personal interests
personal CV
```

------------------------------------------------------------------------

# 13. DASHBOARD

Главный экран не должен быть энциклопедией.

Приоритет:

``` text
1. Что изменилось?
2. Что важно сейчас?
3. Что рекомендовано сделать?
4. Какие возможности?
5. Какие доказательства требуют внимания?
```

Целевой блок:

``` tsx
<DailyBrief />
<NextBestAction />
<RelevantOpportunities limit={3} />
<PendingDecisions />
<ProofRequests />
```

------------------------------------------------------------------------

# 14. PROJECT ROOM

8 существующих табов не должны автоматически означать 8 обязательных
информационных массивов.

Порядок:

``` text
Overview
Team
Roles
Evidence
Stages
Discussion
Results
LOT
```

Но первый экран:

``` text
Project outcome
Current stage
My role
My contribution
Evidence status
Next action
```

------------------------------------------------------------------------

# 15. JOBS / OPPORTUNITIES

Не превращать Jobs в обычную доску вакансий.

Пользователь должен видеть:

``` text
Why this opportunity?
What matches?
What is missing?
What proves fit?
What should I do?
```

Пример:

``` text
Почему подходит

✓ 4/5 ключевых навыков
✓ 2 подтверждённых проекта
✓ совпадает текущий intent
△ нет подтверждённого опыта в X

[Посмотреть]
[Откликнуться]
```

------------------------------------------------------------------------

# 16. SEARCH

Поиск должен вести не к «результатам базы», а к решению задачи.

Пример:

``` text
"Нужен инженер по РЗА для проекта на Сахалине"
```

Результат:

``` text
People
Companies
Projects
Evidence
Opportunities
```

Каждый результат должен объяснять релевантность.

------------------------------------------------------------------------

# 17. NEWS

News Bridge не должен превращать BizON в склад статей.

Каждая карточка:

``` text
Источник
Короткий summary
Почему важно
Связанные opportunities
Original link
```

Не хранить полные внешние статьи без необходимости.

------------------------------------------------------------------------

# 18. BIZON RECOMMENDS / SARAFAN RADIO

Рекомендация должна иметь основание.

Плохо:

``` text
Иван — отличный специалист ⭐⭐⭐⭐⭐
```

Хорошо:

``` text
Рекомендую Анну для автоматизации проекта.

Основание:
Мы работали вместе над X.
Она отвечала за Y.
Результат: Z.
```

------------------------------------------------------------------------

# 19. TRUST

Trust нельзя превращать в публичную цифру.

Вместо:

``` text
Trust 91
```

показывать:

``` text
Проверено:
✓ личность
✓ 6 проектов
✓ 4 результата
✓ 2 подтверждения клиентов
✓ 3 профессиональные рекомендации
```

------------------------------------------------------------------------

# 20. AI ECONOMY / LUMENS

AI action должен иметь:

``` text
actionKey
risk
lumenCost
policy
audit
result
```

Пример:

``` ts
type AiActionResult = {
  actionKey: string
  lumenCost: number
  success: boolean
  auditId: string
}
```

Если AI действие не было оказано:

`refund`.

------------------------------------------------------------------------

# 21. ML HONESTY

Если алгоритм --- heuristic, он должен называться heuristic.

Плохо:

``` text
LSTM Forecast
```

если код:

``` ts
holtForecast(...)
```

Правильно:

``` text
Trend Forecast v0 — Holt heuristic
```

При переходе к реальной модели:

``` text
ADR
+
training/eval dataset
+
baseline comparison
+
metrics
+
rollback
```

------------------------------------------------------------------------

# 22. EVALUATION

Нельзя тестировать ranking тем же самым правилом, которым ranking
строится.

Плохо:

``` ts
const expected = skillOverlap(query, candidate)
const actual = matcher.rank(query, candidate)
```

Хорошо:

``` text
Human-labelled ground truth
        ↓
held-out dataset
        ↓
matcher
        ↓
Precision@3
Recall@k
NDCG
calibration
```

------------------------------------------------------------------------

# 23. OBSERVABILITY

Добавить correlation id:

``` ts
const requestId = req.headers.get('x-request-id') ?? crypto.randomUUID()
```

Логировать:

``` json
{
  "requestId": "...",
  "userId": "...",
  "action": "...",
  "durationMs": 42,
  "status": 200
}
```

Не логировать PII без необходимости.

------------------------------------------------------------------------

# 24. DATABASE DISCIPLINE

Новые изменения схемы:

1.  описать reason;
2.  проверить backward compatibility;
3.  migration;
4.  seed only for dev/test;
5.  production data safety;
6.  rollback plan.

Не использовать destructive schema change как быстрый workaround.

------------------------------------------------------------------------

# 25. TEST PYRAMID

``` text
             E2E
          Integration
       Service / API
      Unit / Pure logic
```

Unit не заменяет E2E.

E2E не заменяет security tests.

------------------------------------------------------------------------

# 26. DEFINITION OF DONE

Фича считается DONE только если:

-   [ ] requirement сформулирован;
-   [ ] user outcome понятен;
-   [ ] architecture checked;
-   [ ] schema checked;
-   [ ] API работает;
-   [ ] UI проводит к API;
-   [ ] success state реальный;
-   [ ] error state;
-   [ ] loading state;
-   [ ] empty state;
-   [ ] authorization;
-   [ ] rate limit;
-   [ ] audit если требуется;
-   [ ] unit;
-   [ ] integration;
-   [ ] E2E;
-   [ ] adversarial;
-   [ ] mobile;
-   [ ] accessibility;
-   [ ] performance;
-   [ ] screenshot/evidence;
-   [ ] независимая приёмка.

------------------------------------------------------------------------

# 27. ЧТО ДОЛЖНО БЫТЬ ПРЕДСТАВЛЕНО ПОСЛЕ КАЖДОЙ ИТЕРАЦИИ

``` text
ITERATION N
Status:
Changed:
Files:
DB:
API:
UI:
Tests:
Security:
Performance:
Screenshots:
Known issues:
Next iteration:
```

------------------------------------------------------------------------

# 28. НЕ РАБОТАТЬ РАДИ РАБОТЫ

Запрещено:

-   переписывать рабочий модуль без причины;
-   добавлять новую библиотеку без Decision Rule;
-   делать новый service при наличии существующего;
-   добавлять AI ради надписи «AI»;
-   добавлять realtime ради realtime;
-   добавлять dashboard ради графиков;
-   добавлять gamification ради engagement;
-   делать новый model без пользовательского результата.

Перед каждым изменением:

> **Если мы это не сделаем, какую реальную проблему пользователя мы
> оставим?**

Если ответ слабый --- задача откладывается или удаляется.

------------------------------------------------------------------------

# 29. ПРАВИЛО REUSE → EXTEND → REFACTOR → REPLACE

Перед созданием нового сервиса:

``` bash
grep -R "similarConcept" src/lib/services
```

Затем:

1.  REUSE;
2.  EXTEND;
3.  REFACTOR;
4.  только потом REPLACE.

------------------------------------------------------------------------

# 30. ПРАВИЛО ANTI-FAKE

Никогда:

``` ts
return { success: true }
```

если действие не было выполнено.

Никогда:

``` ts
toast("Отправлено")
```

до получения результата API.

Никогда:

``` ts
score = 0.91
```

если score не имеет доказуемого источника.

------------------------------------------------------------------------

# 31. ПРАВИЛО «НЕ ПОКАЗЫВАТЬ ВСЁ»

Данные делятся:

``` text
Primary
Secondary
Contextual
Deep detail
Private
System-only
```

Например:

`ipScore` --- system-only/legacy, не public.

`Evidence` --- contextual.

`Intent` --- primary для opportunity matching.

`AuditLog` --- system-only.

------------------------------------------------------------------------

# 32. ПОРЯДОК ПЕРВЫХ P0 ПОСЛЕ АУДИТА

## P0.1

Intro UI.

## P0.2

Intro inbox.

## P0.3

Complaint UI.

## P0.4

Company vacancy creation.

## P0.5

Credential export UI.

После каждого:

`UI → API → DB → E2E → screenshot`.

------------------------------------------------------------------------

# 33. P1

После P0:

1.  relationship UI;
2.  Opportunity related navigation;
3.  reputation estimator API/UI только если есть реальный use-case;
4.  ML naming cleanup;
5.  benchmark ground truth;
6.  FPR evaluation;
7.  web-push honest status;
8.  outbox producers;
9.  sitemap route integrity;
10. clean test data;
11. company-specific copy;
12. remove public demo credentials.

------------------------------------------------------------------------

# 34. P2

-   visual polish;
-   motion;
-   accessibility refinement;
-   loading transitions;
-   responsive refinement;
-   performance micro-optimizations;
-   information density reduction.

------------------------------------------------------------------------

# 35. ПРОЦЕСС АГЕНТОВ

Использовать специализированные роли.

## CTO / Architect

Определяет:

-   architecture;
-   dependencies;
-   priority;
-   acceptance.

## Product Agent

Проверяет:

-   user problem;
-   business value;
-   feature necessity.

## Information Architect

Проверяет:

-   information hierarchy;
-   cognitive load;
-   redundancy.

## UX Agent

Проверяет:

-   flow;
-   states;
-   next action.

## UI Agent

Проверяет:

-   visual hierarchy;
-   responsive;
-   design system.

## Frontend Agent

Пишет React/Next/Tailwind.

## Backend Agent

Пишет API/services.

## Data Agent

Проверяет Prisma/data graph.

## AI Agent

Проверяет AI contracts/evals.

## Governance Agent

Проверяет:

`Action Registry → Policy → Approval → Audit → Kill Switch`.

## Security Agent

Ищет:

-   IDOR;
-   privilege escalation;
-   injection;
-   secrets;
-   privacy violations.

## QA Agent

Пишет:

-   unit;
-   integration;
-   E2E.

## Red Team

Намеренно ломает систему.

## Performance Agent

Измеряет.

## Release Agent

Проверяет Production Gate.

------------------------------------------------------------------------

# 36. КАК АГЕНТЫ ДОЛЖНЫ РАБОТАТЬ

Не параллельно менять один и тот же файл.

Правило:

``` text
Architect
   ↓
Contract
   ↓
Backend/Data
   ↓
Frontend
   ↓
QA
   ↓
Security
   ↓
Independent Acceptance
```

Для независимых задач допускается параллельность.

------------------------------------------------------------------------

# 37. CTO REVIEW КАЖДОГО PR / PATCH

Проверить:

``` text
Why?
What changed?
What did not change?
Does it match architecture?
Does it reduce complexity?
Can it be tested?
Can it be rolled back?
Does it create new authority?
Does it expose new data?
Does AI gain new power?
```

------------------------------------------------------------------------

# 38. КРИТИЧЕСКАЯ ПРОВЕРКА ПРАВ AI

Если новая функция AI может:

-   отправлять;
-   публиковать;
-   изменять;
-   покупать;
-   удалять;
-   подписывать;
-   приглашать;
-   менять права;

то она обязана пройти:

``` text
Action Registry
→ Policy Engine
→ Authority
→ Approval
→ Audit
→ Rollback / kill path
```

------------------------------------------------------------------------

# 39. RELEASE CANDIDATE

Перед RC сформировать:

``` text
docs/CTO/FINAL-RELEASE-REPORT.md
docs/CTO/OPEN-RISKS.md
docs/CTO/SECURITY-REPORT.md
docs/CTO/UX-REPORT.md
docs/CTO/EVIDENCE-MATRIX.md
```

И:

``` text
RELEASE_STATUS = READY | BLOCKED
```

READY разрешён только при выполнении production gates.

------------------------------------------------------------------------

# 40. ФОРМАТ ДОКАЗАТЕЛЬСТВА

Для каждого исправления:

``` text
ID: D-01
Requirement: Intro UI

Before:
src/components/profile/profile-view.tsx:...

Change:
src/components/profile/profile-view.tsx:...

API:
POST /api/intros

DB:
IntroductionRequest created

Test:
tests/intro-ui.spec.ts

Browser:
Screenshot: ...

Result:
PASS
```

------------------------------------------------------------------------

# 41. ЧТО НУЖНО ПРЕДОСТАВИТЬ ДЛЯ СЛЕДУЮЩЕГО АНАЛИЗА

После выполнения очередной итерации вернуть:

1.  обновлённый архив;
2.  `git diff` или commit;
3.  список изменённых файлов;
4.  тестовый вывод;
5.  E2E screenshots;
6.  актуальный `VERSION`;
7.  актуальный `worklog`;
8.  `CTO-EVIDENCE-MATRIX`;
9.  список оставшихся BLOCKED;
10. known risks.

Не писать только:

> «Всё готово».

Нужно писать числа и доказательства.

------------------------------------------------------------------------

# 42. ФОРМАТ СЛЕДУЮЩЕГО АУДИТА

Анализировать:

``` text
Repository Truth
↓
Changed files
↓
Tests
↓
Runtime behaviour
↓
UI screenshots
↓
API traces
↓
DB state
↓
Security
↓
UX
↓
Architecture
↓
Production gate
```

Особенно проверять DOC-MISMATCH.

------------------------------------------------------------------------

# 43. CTO RULE: НЕ ДОВЕРЯТЬ СОБСТВЕННОМУ ОТЧЁТУ

Исполнитель не может быть единственным источником доказательства.

Если агент пишет:

``` text
DONE
```

это означает только:

``` text
implementation claims done
```

Независимый reviewer должен установить:

``` text
VERIFIED
```

------------------------------------------------------------------------

# 44. ПЕРВЫЙ КОНКРЕТНЫЙ БЭКЛОГ

``` text
P0
D-01 Intro profile UI
D-02 Intro inbox
D-03 Complaint dialog/API
D-04 Company vacancy creation
D-05 Credential export

P1
D-06 Relationship UI
D-07 Opportunity related navigation
D-08 ML naming honesty
D-09 Ground truth evaluation
D-10 Sarafan FPR benchmark
D-11 Web push truthful status
D-12 Outbox producers
D-13 Sitemap route integrity

P2
D-14 Test-data cleanup
D-15 Company copy cleanup
D-16 Demo credential removal
D-17 UI density reduction
D-18 accessibility/performance polish
```

------------------------------------------------------------------------

# 45. СТРАТЕГИЧЕСКАЯ ЦЕЛЬ

BizON не должен становиться платформой, где «есть всё».

Он должен стать системой, которая отвечает на профессиональный вопрос:

> **Что мне делать сейчас, с кем, почему этому человеку/компании можно
> доверять, что подтверждает это решение и какой результат я получу?**

------------------------------------------------------------------------

# 46. ФИНАЛЬНАЯ МОДЕЛЬ

``` text
USER
 ↓
INTENT
 ↓
OPPORTUNITY
 ↓
MATCH
 ↓
ACTION
 ↓
PROJECT / WORK
 ↓
RESULT
 ↓
EVIDENCE
 ↓
SKILL
 ↓
TRUST
 ↓
RECOMMENDATION
 ↓
NEW OPPORTUNITY
 ↓
NEW RESULT
```

AI:

``` text
          ┌─────────────────────┐
          │     BIZON AI        │
          │ Explain / Search /  │
          │ Recommend / Assist  │
          └─────────┬───────────┘
                    │
                    ↓
          Professional Graph
```

AI не должен подменять Graph.

------------------------------------------------------------------------

# 47. ФИНАЛЬНОЕ ПРАВИЛО CTO

Каждая строка нового кода должна иметь причину.

Каждый API endpoint должен иметь потребителя или документированную
системную роль.

Каждая Prisma model должна иметь use-case.

Каждый UI блок должен иметь пользователя.

Каждая AI action должна иметь governance.

Каждый публичный факт должен иметь источник.

Каждая рекомендация должна иметь основание.

Каждый статус READY должен иметь доказательство.

Каждая сложность должна оправдываться результатом.

И главный вопрос всей разработки:

> **Если пользователь завтра откроет BizON впервые, станет ли ему
> понятнее, что делать, кому доверять и как получить реальный
> профессиональный результат?**

Если нет --- мы не закончили.

------------------------------------------------------------------------

# 48. COMMAND ДЛЯ ИСПОЛНИТЕЛЯ

Исполнитель должен начать с:

``` text
1. Read VERSION
2. Read docs/PROJECT-RULES.md
3. Read docs/audit/AUDIT-REPORT-2026-09-19.md
4. Read worklog.md
5. Generate Repository Truth
6. Do not modify production code yet
7. Produce P0/P1/P2 evidence matrix
8. Execute D-01 only
9. Run unit + integration + E2E + security
10. Produce evidence
11. Continue D-02 only after D-01 verification
```

Нельзя начинать с:

``` text
"rewrite the whole app"
```

Нельзя начинать с:

``` text
"add modern AI features"
```

Нельзя начинать с:

``` text
"redesign everything"
```

Начинать нужно с:

``` text
PROVE CURRENT TRUTH
→ CLOSE P0 USER DOORS
→ REDUCE INFORMATION NOISE
→ VERIFY
→ THEN EXPAND
```

------------------------------------------------------------------------

# 49. ГОТОВНОСТЬ К СЛЕДУЮЩЕМУ ЦИКЛУ

После выполнения этого playbook следующий аудит должен отвечать не на
вопрос:

> «Сколько функций сделано?»

а на вопросы:

1.  Сколько пользовательских циклов реально закрыто?
2.  Сколько API имеют работающие UI consumers?
3.  Сколько фактов имеют доказательства?
4.  Сколько AI actions проходят governance?
5.  Сколько критических DOC-MISMATCH осталось?
6.  Сколько fake-success осталось?
7.  Сколько P0 осталось?
8.  Сколько UI блоков удалено как ненужные?
9.  Как изменилось время пользователя до результата?
10. Что реально стало лучше для человека?

**Именно эти показатели являются следующим уровнем зрелости BizON.**
