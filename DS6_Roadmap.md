# DS6 — Roadmap de Implementação
> **Projeto:** DistriSchool (DS6) · **Data:** 13/10/2025 · **Owner:** Gabriel Aderaldo (PM/Tech)  
> **Arquitetura alvo:** Backend Spring Boot (MVC) + EDD (Kafka) · Single-tenant por instância (MVP) → (opcional) multi-tenant lógico;  
> Frontend **Flutter Web** (Micro-frontends) + **MVVM (ChangeNotifier)**; **BFF por domínio** com **DDD** (domínio + casos de uso); Consumo **REST + WebSocket**;  
> DevOps: **Kubernetes** + **GitHub Actions** (workflow reutilizável), **PostgreSQL + Flyway**, Observabilidade (**OTel/Prometheus/Grafana**), LGPD.

---

## 1 Princípios norteadores (não negociáveis)
- **Contrato antes do código**: SRS v1.0 como base; RFs testáveis (BDD); RNFs numéricos.
- **Isolamento por escola** no MVP (**single-tenant por instância**): namespace, DB, tópicos, segredos.
- **Dados por serviço** (sem DB compartilhado); **Outbox + Idempotência** para eventos; **Schema versioning** (`*.v1`).
- **Front “burro”, VM inteligente**: Pages/Widgets (Atomic) sem regra; ViewModel orquestra (Commands) e integra BFF (REST/WS).
- **CI/CD reprodutível**: workflow reutilizável; deploy declarativo (manifests k8s); health via Actuator.
- **Observabilidade de verdade**: logs JSON, traces distribuídos, métricas com `tenant_id` e `correlation_id`.
- **LGPD by design**: minimização de PII, retenção/anonimização, trilhas de auditoria.

---

## 2 Estado atual (13/10/2025)
- **Repos criados**: `ds6-auth-service`, `ds6-user-service`, `ds6-student-service`, `ds6-presence-service`, `ds6-grade-exams-service`, `ds6-class-schedule-service`, `ds6-admin-service`, `library`, `.github`.
- **Pipeline**: workflow reutilizável definido; padrão de manifests k8s com `image: imagem-placeholder` + probes Actuator; segredo de org `KUBE_CONFIG_DATA`.
- **Decisão arquitetural**: **Single-tenant (MVP)**, **MVC + EDD**, **BFF DDD**, **Flutter Web MFE + MVVM** (ChangeNotifier), **offline-first** no front.
- **Pendências chave**: fechar RNFs p99, regras exatas de **notas/atrasos**, catálogo de **eventos v1** consolidado no `library`.

---

## 3 Macro cronograma (Q4/2025 → Q3/2026)
| Fase | Período | Objetivo | Entregas Chave | Exit Criteria |
|---|---|---|---|---|
| **F0 – Fundações** | **13–24/Out/2025** | Padrões & trilhos | SRS v1.0; `library` (envelope + Problem+JSON); CI/CD herdável; templates k8s em todos os repos; secrets bootstrap | SRS assinado; 100% repos com pipeline verde; smoke `/actuator/health` ok |
| **F1 – MVP Alpha** | **27/Out – 29/Nov/2025** | Núcleo operacional | Auth+Users (login p95<2s); Students (CRUD+busca); Presence básico (POST + WS canal); BFFs iniciais; Shell Flutter + MFE-Students/Presence; offline fila “skeleton” | Pilot interno (10 usuários) com 99% uptime/semana; 0 vazamento de tenant |
| **F1.1 – MVP Beta (Escola #1)** | **01–20/Dez/2025** | Piloto real | Exportações CSV/PDF; Notifications (e-mail básico); Import CSV (Pipes&Filters v1); Observabilidade base (dashboards); backup/restore test | Escola #1 ativa 2 semanas; erro crítico ≤ 1/semana; RPO≤15min, RTO≤1h (simulado) |
| **F1.2 – GA (3 escolas)** | **05–31/Jan/2026** | Go-live controlado | Grades (lançamento + WS); Reports básicos; LGPD (retenção/anonimização inicial); runbooks SRE | 3 escolas on; NPS>7; incidentes P1=0; SLOs batidos |
| **F2 – Escala & Automação** | **Fev–Mar/2026** | “Control plane” de instâncias | IaC para “criar escola” (namespace+DB+topics+secrets); quotas/limites; freeze `eventos v1` + contratos; Import P&F v2 (idempotência fina) | Provisionamento <15min/escola; 0 erro de isolamento; contratos validados em CI |
| **F3 – Evolução Arquitetural** | **Abr–Jun/2026** | Robustez & custo | Hardening de performance; caching; WS resiliente; (Opcional) POC **multi-tenant lógico** em ambiente de lab | p95 páginas <2s; custo/tenant ↓20%; POC sem vazamento comprovado |
| **F4 – Produto 2.0** | **Jul–Set/2026** | Capas finais | Analytics leve; notificações avançadas; UX/Design tokens unificados; governança MFE (GAM) | adoção > X escolas; backlog de tech-debt ≤ 10% do WIP |

> Datas em **America/Fortaleza** (UTC−03). Ajuste conforme feriados.

---

## 4 Fases detalhadas

### F0 — Fundações (13–24/Out/2025)
**Escopo**
- Fechar **SRS v1.0** (RF/RNF, Arquitetura, Catálogo de Eventos v1, ADRs sumário).
- Publicar **`library`**: envelope de evento, JSON Schemas (`*.v1`), `Problem+JSON`, erros padrão.
- Replicar **CI/CD** padrão e **k8s/** (deployment/service/configmap) para todos os `ds6-*`.
- Confirmar **segredos**: `KUBE_CONFIG_DATA` e `<svc>-secret` por instância (dev/staging/prod).
- Observabilidade “starter”: logs JSON, OTel agente, métricas padrão.

**Entregáveis**
- SRS.md v1.0; `library@1.0.0` no GH Packages; dashboards iniciais; runbooks bootstrap/rollback.
**Saída/Exit**
- Build+test+deploy verde em todos os repos; smoke `/actuator/health` ok; ADRs publicadas.

---

### F1 — MVP Alpha (27/Out – 29/Nov/2025)
**Backend**
- `ds6-auth-service` (login, verificação, recovery) + `ds6-user-service` (roles). Evento `auth.user.logged.v1`.
- `ds6-student-service` (CRUD+busca; índices; exportação); evento `student.created.v1`.
- `ds6-presence-service` (registro simples; WS canal); evento `attendance.recorded.v1`.
- Outbox Pattern + idempotência nos produtores; Problem+JSON em todos os erros.

**BFF & Front**
- BFFs: `ds6-auth-bff`, `ds6-students-bff`, `ds6-presence-bff` (REST + WS; OpenAPI/Schema).
- Shell Flutter + **MFE-Students** e **MFE-Presence** (MVVM/ChangeNotifier; Commands; offline queue “skeleton”).

**DevOps**
- Namespaces por escola (dev/staging); K8s Secrets por serviço; backup diário por DB; monitor p95 login.

**Exit**
- Pilot interno (10 usuários), login p95<2s; p99 registrados; 99% uptime/semana; sem cross-tenant.

---

### F1.1 — MVP Beta (01–20/Dez/2025)
**Incrementos**
- Export CSV/PDF; Notifications (e-mail mínimo).
- Import CSV **Pipes & Filters v1** (validação -> normalização -> commit) com progress WS.
- Observabilidade: dashboards por domínio; alertas SLO (Auth, Presence).  
**Exit**
- Escola #1 ao vivo 2 semanas; RPO≤15min, RTO≤1h (drill); incidentes P1=0.

---

### F1.2 — GA (05–31/Jan/2026)
**Incrementos**
- `ds6-grade-exams-service` + `ds6-grades-bff` + **MFE-Grades**; evento `grade.posted.v1`.
- Reports operacionais; LGPD (retenção/anonimização inicial); testes de restauração.
**Exit**
- 3 escolas on; NPS>7; SLOs batidos; catálogo de eventos **congelado** em `v1`.

---

### F2 — Escala & Automação (Fev–Mar/2026)
**Control Plane**
- Terraform/Helm: **create-school** (namespace, DB, usuários/secrets, topics Kafka, rotas, quotas).
- Catálogo de instâncias (owner, SLA, custo).
**Qualidade Contratual**
- Contract tests: bloquear breaking changes sem bump (`*.v1`).  
- Import **P&F v2** com **idempotência** fina e reprocessamento.
**Exit**
- Provisionar escola ≤15min; 0 incidente de isolamento; CI barra contratos inválidos.

---

### F3 — Evolução Arquitetural (Abr–Jun/2026)
**Performance & Custo**
- Caching estratégico (BFF/serviços), compressão, N+1 killers, WS resiliente (backpressure).
- Métricas de custo/tenant; otimização de recursos/k8s (requests/limits).
**R&D (Opcional)**
- POC **multi-tenant lógico** (schema-per-tenant + RLS) em ambiente de lab; suíte de testes anti-vazamento.
**Exit**
- p95 páginas <2s; custo/tenant ↓20%; POC validada sem vazamento.

---

### F4 — Produto 2.0 (Jul–Set/2026)
**Experiência & Governança**
- Design tokens unificados; UX consistente nos MFEs.
- Notificações avançadas (segmentação/opt-out); Analytics leve.
- **GAM**/Governança MFE (linters de arquitetura; CI de contratos do front).
**Exit**
- Adoção > meta; dívida técnica ≤10% WIP; playbooks SRE estáveis.

---

## 5) Trilha por “swimlane”

### 5.1 Backend (serviços)
- **Sempre**: dados por serviço; Outbox; idempotência; RFC 7807; OpenAPI.
- MVP: Auth, Users, Students, Presence → + Grades, Schedule, Admin.
- Pós-MVP: Notifications, Reports; otimizações; POC multi-tenant lógico.

### 5.2 BFF & Front (Flutter Web)
- **Shell** (roteamento global + EventBus); **MFE** por domínio (Vertical Split).
- **MVVM/ChangeNotifier**; Pages/Widgets “dumb”; Commands → VM → REST/WS.
- **Offline-first**: fila + retry + reconciliação por eventos.
- Pós-MVP: design tokens, lints de arquitetura, contratos WS versionados.

### 5.3 DevOps & Segurança
- CI/CD herdável; GHCR; rollout/rollback; secretação K8s; `KUBE_CONFIG_DATA`.
- Observabilidade: logs/traces/métricas; dashboards; SLO/alerts.
- LGPD: retenção/anonimização; matrizes de acesso; rotação de segredos.

### 5.4 Dados
- PostgreSQL por escola; índices (matrícula, composições de horário); backup diário; DR drills.
- [PENDÊNCIA] política final de retenção/anonimização; criptografia em repouso (se aplicável ao provedor).

---

## 6 Versionamento & compatibilidade
- **SemVer** nos serviços/BFFs; imagens `ghcr.io/org/svc:1.2.3`.
- **Eventos**: `*.v1` **congelado** até F2; mudanças **aditivas** ok; breaking ⇒ `v2`.
- **Contratos**: contract tests em CI (back e front/WS); OpenAPI publicado.

---

## 7 Métricas & metas (SLOs principais)
- **Login**: p95 < **2s**; erro < **0,5%**.
- **Presença POST**: p95 < **1,5s**; WS push < **5s** (p95).
- **Disponibilidade**: **99%** (Auth/Presence/Grades).
- **Provisionamento**: nova escola ≤ **15min** (F2).
- **Custo/tenant**: redução **20%** até F3.

---

## 8 Riscos & mitigação (Top-8)
1. Vazamento cross-tenant → Isolamento por instância; testes anti-vazamento; revisão de queries; auditoria.
2. Incompatibilidade de contratos → Contract tests; versionamento; `v1` congelado.
3. Falhas WS/realtime → Backpressure; fila de retry; alerts p95/p99.
4. Importações lentas → Pipes&Filters v2; paralelismo controlado; idempotência.
5. LGPD incompleta → DPO no loop; retenção/anonimização; DPIA se necessário.
6. Custos de infra → Requests/Limits; autoscaling; sizing por workload.
7. Déficit de UX MFE → Design tokens; lints de UI; revisão de acessibilidade.
8. Rotina de DR não testada → Drills semestrais; RPO/RTO medidos.

---

## 9 Dependency map (ordem que evita dor)
1. **F0** (library, CI/CD, k8s) → 2. **Auth/Users** → 3. **Students** → 4. **Presence (WS)** → 5. **BFF/Front** básico → 6. **Export/Import** → 7. **Grades** → 8. **Reports** → 9. **Control Plane** → 10. **R&D multi-tenant**.

---

## 10 Checklists de saída por fase
- **F0**: SRS assinado; `library@1.0.0`; 100% repos com CI/CD + k8s/; smoke verde.
- **F1**: p95 login <2s; WS up; offline fila criada; 99% uptime/semana.
- **F1.1**: Escola #1 ativa; DR drill ok; export/import ok.
- **F1.2**: 3 escolas on; Grades + reports; LGPD inicial; eventos v1 congelados.
- **F2**: create-school ≤15min; contratos gateando; quotas/limites.
- **F3**: p95 páginas <2s; custo/tenant ↓20%; POC multi-tenant validada.
- **F4**: design tokens; lints; analytics leve; dívida ≤10% WIP.

---

## 11 ADRs a registrar (resumo)
- ADR-001: **Single-tenant por instância (MVP)**; pronta para migração lógica.
- ADR-002: **MVC + EDD** (REST + eventos Kafka) como estilo de integração.
- ADR-003: **Front Flutter Web MFE + MVVM** (Pages/Widgets “dumb”; VM orquestra).
- ADR-004: **BFF por domínio (DDD)**; contratos REST/WS; Outbox.
- ADR-005: **Observabilidade & SLOs** (correlação, métricas, alerts).
- ADR-006: **Control Plane** (IaC) p/ provisionar escolas.
- ADR-007: **Versionamento de eventos** (freeze v1; aditivo vs breaking).

---

## 12 Próximos passos imediatos (sem depender de ninguém)
1) Publicar `library@1.0.0` (envelope + Problem+JSON + Schemas v1).  
2) Propagar `k8s/` + `main-pipeline.yml` em todos os serviços; checar probes.  
3) Implementar **Auth+Users** e **Students** com Outbox + eventos; **BFFs** e **MFEs** iniciais.  
4) Ligar dashboards e alertas (Auth/Presence); definir `X-Correlation-Id`.  
5) Fechar **RNFs p99** e **regras de notas/atrasos**.

---

_Atualizado em 13/10/2025 (America/Fortaleza)._