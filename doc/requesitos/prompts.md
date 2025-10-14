MEU ROADMAP PARA referencia:
```md
# Create a markdown roadmap file for the DS6 project
from datetime import date

content = """# DS6 — Roadmap de Implementação
> **Projeto:** DistriSchool (DS6) · **Data:** 13/10/2025 · **Owner:** Gabriel Aderaldo (PM/Tech)  
> **Arquitetura alvo:** Backend Spring Boot (MVC) + EDD (Kafka) · Single-tenant por instância (MVP) → (opcional) multi-tenant lógico;  
> Frontend **Flutter Web** (Micro-frontends) + **MVVM (ChangeNotifier)**; **BFF por domínio** com **DDD** (domínio + casos de uso); Consumo **REST + WebSocket**;  
> DevOps: **Kubernetes** + **GitHub Actions** (workflow reutilizável), **PostgreSQL + Flyway**, Observabilidade (**OTel/Prometheus/Grafana**), LGPD.

---

## 1) Princípios norteadores (não negociáveis)
- **Contrato antes do código**: SRS v1.0 como base; RFs testáveis (BDD); RNFs numéricos.
- **Isolamento por escola** no MVP (**single-tenant por instância**): namespace, DB, tópicos, segredos.
- **Dados por serviço** (sem DB compartilhado); **Outbox + Idempotência** para eventos; **Schema versioning** (`*.v1`).
- **Front “burro”, VM inteligente**: Pages/Widgets (Atomic) sem regra; ViewModel orquestra (Commands) e integra BFF (REST/WS).
- **CI/CD reprodutível**: workflow reutilizável; deploy declarativo (manifests k8s); health via Actuator.
- **Observabilidade de verdade**: logs JSON, traces distribuídos, métricas com `tenant_id` e `correlation_id`.
- **LGPD by design**: minimização de PII, retenção/anonimização, trilhas de auditoria.

---

## 2) Estado atual (13/10/2025)
- **Repos criados**: `ds6-auth-service`, `ds6-user-service`, `ds6-student-service`, `ds6-presence-service`, `ds6-grade-exams-service`, `ds6-class-schedule-service`, `ds6-admin-service`, `library`, `.github`.
- **Pipeline**: workflow reutilizável definido; padrão de manifests k8s com `image: imagem-placeholder` + probes Actuator; segredo de org `KUBE_CONFIG_DATA`.
- **Decisão arquitetural**: **Single-tenant (MVP)**, **MVC + EDD**, **BFF DDD**, **Flutter Web MFE + MVVM** (ChangeNotifier), **offline-first** no front.
- **Pendências chave**: fechar RNFs p99, regras exatas de **notas/atrasos**, catálogo de **eventos v1** consolidado no `library`.

---

## 3) Macro cronograma (Q4/2025 → Q3/2026)
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

## 4) Fases detalhadas

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

## 6) Versionamento & compatibilidade
- **SemVer** nos serviços/BFFs; imagens `ghcr.io/org/svc:1.2.3`.
- **Eventos**: `*.v1` **congelado** até F2; mudanças **aditivas** ok; breaking ⇒ `v2`.
- **Contratos**: contract tests em CI (back e front/WS); OpenAPI publicado.

---

## 7) Métricas & metas (SLOs principais)
- **Login**: p95 < **2s**; erro < **0,5%**.
- **Presença POST**: p95 < **1,5s**; WS push < **5s** (p95).
- **Disponibilidade**: **99%** (Auth/Presence/Grades).
- **Provisionamento**: nova escola ≤ **15min** (F2).
- **Custo/tenant**: redução **20%** até F3.

---

## 8) Riscos & mitigação (Top-8)
1. Vazamento cross-tenant → Isolamento por instância; testes anti-vazamento; revisão de queries; auditoria.
2. Incompatibilidade de contratos → Contract tests; versionamento; `v1` congelado.
3. Falhas WS/realtime → Backpressure; fila de retry; alerts p95/p99.
4. Importações lentas → Pipes&Filters v2; paralelismo controlado; idempotência.
5. LGPD incompleta → DPO no loop; retenção/anonimização; DPIA se necessário.
6. Custos de infra → Requests/Limits; autoscaling; sizing por workload.
7. Déficit de UX MFE → Design tokens; lints de UI; revisão de acessibilidade.
8. Rotina de DR não testada → Drills semestrais; RPO/RTO medidos.

---

## 9) Dependency map (ordem que evita dor)
1. **F0** (library, CI/CD, k8s) → 2. **Auth/Users** → 3. **Students** → 4. **Presence (WS)** → 5. **BFF/Front** básico → 6. **Export/Import** → 7. **Grades** → 8. **Reports** → 9. **Control Plane** → 10. **R&D multi-tenant**.

---

## 10) Checklists de saída por fase
- **F0**: SRS assinado; `library@1.0.0`; 100% repos com CI/CD + k8s/; smoke verde.
- **F1**: p95 login <2s; WS up; offline fila criada; 99% uptime/semana.
- **F1.1**: Escola #1 ativa; DR drill ok; export/import ok.
- **F1.2**: 3 escolas on; Grades + reports; LGPD inicial; eventos v1 congelados.
- **F2**: create-school ≤15min; contratos gateando; quotas/limites.
- **F3**: p95 páginas <2s; custo/tenant ↓20%; POC multi-tenant validada.
- **F4**: design tokens; lints; analytics leve; dívida ≤10% WIP.

---

## 11) ADRs a registrar (resumo)
- ADR-001: **Single-tenant por instância (MVP)**; pronta para migração lógica.
- ADR-002: **MVC + EDD** (REST + eventos Kafka) como estilo de integração.
- ADR-003: **Front Flutter Web MFE + MVVM** (Pages/Widgets “dumb”; VM orquestra).
- ADR-004: **BFF por domínio (DDD)**; contratos REST/WS; Outbox.
- ADR-005: **Observabilidade & SLOs** (correlação, métricas, alerts).
- ADR-006: **Control Plane** (IaC) p/ provisionar escolas.
- ADR-007: **Versionamento de eventos** (freeze v1; aditivo vs breaking).

---

## 12) Próximos passos imediatos (sem depender de ninguém)
1) Publicar `library@1.0.0` (envelope + Problem+JSON + Schemas v1).  
2) Propagar `k8s/` + `main-pipeline.yml` em todos os serviços; checar probes.  
3) Implementar **Auth+Users** e **Students** com Outbox + eventos; **BFFs** e **MFEs** iniciais.  
4) Ligar dashboards e alertas (Auth/Presence); definir `X-Correlation-Id`.  
5) Fechar **RNFs p99** e **regras de notas/atrasos**.

---

_Atualizado em 13/10/2025 (America/Fortaleza)._"""
```

```md
Você é um redator técnico sênior. Gere a seção **“0) Capa & Histórico”** do SRS do projeto **DS6 — DistriSchool**.

Contexto base (não altere):
- Arquitetura: Backend Spring Boot (MVC) + **Event-Driven (EDD)** com Kafka; **single-tenant por instância** no MVP.
- Frontend: **Micro-frontends** em **Flutter Web** com **MVVM (ChangeNotifier)**; **BFF** por domínio com **DDD** (domínio + use cases); consumo via **REST + WebSocket**.
- DevOps: Kubernetes + GitHub Actions (workflow reutilizável), PostgreSQL + Flyway, observabilidade (OTel/Prometheus/Grafana), LGPD.
- Repositórios existentes: ds6-* services + library + .github (pipeline).

Instruções:
1) Produza: Título, Versão, Data, Autores, Aprovadores (com linhas para assinatura).
2) Tabela “Histórico de versões” (Data, Versão, Mudanças, Autor).
3) **Resumo executivo** (1 parágrafo, objetivo e sem marketing).
4) Cite o **template do cliente** como base estrutural e, quando fizer sentido, 1–2 referências adicionais.
5) Onde faltar informação, marque **[PENDÊNCIA]**.

Formato:
- Listas e tabelas em Markdown.
- Citações no fim de parágrafo entre parênteses, ex.: (1; Coulouris, cap. 3; Valente, cap. 13).
Fontes possíveis: (1–9) do cliente/bibliografia.

```

```md
Gere a seção **“1) Introdução”** do SRS.

Inclua obrigatoriamente:
1. Finalidade do documento (contrato técnico; público-alvo).
2. Escopo do produto (MVP, limites, o que entra e o que fica fora).
3. Objetivos de negócio mensuráveis (com números e prazos) — use [PENDÊNCIA] se não houver baseline.
4. Definições/Acrônimos (JWT, WS, EDD, MVVM, BFF, Single-tenant etc.).
5. Referências (liste as 1–9).
6. Suposições & Dependências (infra, serviços de e-mail, etc.).
7. Fora de escopo (MVP).

Regras:
- Use o contexto base (MVC+EDD, MFE Flutter+MVVM, BFF DDD, single-tenant).
- Dê pelo menos **1 citação** por sub-seção (ex.: (1; 8) para requisitos; (4; 5) para estilos arquiteturais).
- Marque lacunas como **[PENDÊNCIA]**.
```

```md
Gere a seção **“2) Descrição Geral”**.

Conteúdo mínimo:
2.1 Perspectiva (sistema distribuído; microsserviços; REST + eventos; single-tenant no MVP; futuro multi-tenant lógico via tenant_id).
2.2 Funções principais (Auth, Users, Students, Schedule, Presence, Grades, Notifications, Reports, Admin).
2.3 Atores (Admin, Secretaria, Professor, Aluno, Responsável, Sistemas externos).
2.4 Ambiente de Operação (Spring Boot, Kafka, PostgreSQL+Flyway, Docker, **Kubernetes**; Flutter Web MFE; BFF por domínio).
2.5 Restrições do cliente (login ≤2s p95; 99% uptime; BCrypt; i18n; backup diário; LGPD).
2.6 Documentação do usuário (help/FAQ).
2.7 Suposições de produto.

Regras:
- Use **listas** e **tabelas** quando útil.
- Cite pedidos do cliente (2–3) e fundamentos (4–5).
- Aponte **[PENDÊNCIA]** se algo não estiver fechado.

```

```md
Gere a seção **“3) Requisitos Funcionais”** organizada por domínio/serviço.

Para cada sub-seção (Auth & Users; Students; Schedule; Presence; Grades; Notifications/Reports/Admin) forneça:
- Tabela RF: **ID, Descrição, Prioridade, Critérios de Aceitação (BDD Given/When/Then), Eventos publicados/consumidos, Endpoints REST/WS**.
- **Eventos**: nome padronizado `ds6.<tenant>.<context>.<event>.v1`.
- **Endpoints**: base `/api/v1` no backend e `/bff/<domínio>` no BFF quando aplicável.
- Importações (CSV/Excel) modeladas como **Pipes & Filters** assíncronas (quando for o caso).
- **Offline-first** onde fizer sentido (Presence, etc.).

Regras:
- Reflita o contexto base (MVC+EDD, BFF, MFE).
- 1+ citação por item relevante (ex.: BDD/Histórias (6), EDD/pub-sub (4–5), requisitos do cliente (2)).
- Use **[PENDÊNCIA]** para fórmulas de notas ou regras não fechadas.


```

```md
Gere a seção **“4) Requisitos Não Funcionais”** em tabela.

Categorias (exemplos): Desempenho, Disponibilidade, Escalabilidade, Segurança/LGPD, Observabilidade, Backup/DR, I18n, Confiabilidade, Manutenibilidade.
- Para cada RNF: **ID, Categoria, Descrição objetiva com metas numéricas** (p95/p99, SLO, RPO/RTO, limites por payload, etc.).
- Aponte serviços/rotas críticas (Auth, Presence) e metas específicas.
- Incluir logging/metrics/tracing com `tenant_id` e `correlation_id`.

Regras:
- Cite pedidos do cliente (2–3) e fundamentos (4–5; 8).
- Marque **[PENDÊNCIA]** se métricas precisarem validação.

```

```md
Gere a seção **“5) Casos de Uso”**.

Para cada UC (Auth Login; Cadastrar Estudante; Importar Grade; Registrar Presença (realtime/offline); Lançar Nota):
- Atores.
- Pré-condições / Pós-condições.
- Fluxo Principal e Alternativos.
- Regras de Negócio (inclua erros e limites).
- **Cenários BDD (Given/When/Then)** prontos para automação.
- Referencie endpoints REST/WS e eventos disparados.

Regras:
- 1+ citação por UC (6 para BDD; 4–5 para padrões distribuídos).
- Use **[PENDÊNCIA]** nas partes indefinidas (ex.: fórmula de média).
```

```md
Gere a seção **“6) Arquitetura”**.

Inclua:
6.1 Estilos adotados: Cliente-Servidor 3 camadas (externo), Microsserviços (interno), **Publish–Subscribe (EDD)**, Filas (1→1), Single-tenant por instância (MVP).
6.2 Backend (serviços ds6-* e library): dados por serviço; Outbox; Idempotência; **envelope de evento** (traga JSON completo).
6.3 Tenancy: Single-tenant no MVP; futuro multi-tenant lógico (com `tenant_id`).
6.4 Frontend: **Micro-frontends Flutter Web** + **MVVM (ChangeNotifier)**; **Pages/Widgets “dumb”** (Atomic Design); **ViewModel** como único orquestrador.
6.5 BFF por domínio (DDD + EDD): camadas domain/application/adapters/infrastructure; REST + WS; versionamento.
6.6 Protocolos & Contratos: REST (RFC 7807), WS (JSON Schema).
6.7 Diagramas (descreva em texto estilo C4 e 2–3 diagramas de sequência).

Regras:
- Cada decisão deve ter **justificativa + citação** (Coulouris 4; Valente 5; Dissertação 9).

```

```md
Gere a seção **“7) Dados & Persistência”**.

Inclua:
- Estratégia: **PostgreSQL por escola** (single-tenant), nomes de DB, usuários/secrets por instância.
- Migrações: **Flyway** por serviço.
- Índices recomendados (matrícula; composições para conflitos de agenda).
- Políticas de retenção, arquivamento, anonimização (LGPD) — use [PENDÊNCIA] onde não houver decisão.
- Backup/DR: frequência, testes de restauração; RPO/RTO.

Regras:
- Cite (2–3) para client asks; (4) para boas práticas distribuídas; (8) para LGPD/requisitos.
```


```md
Gere a seção **“8) Segurança & LGPD”**.

Conteúdo:
- Autenticação (JWT), Autorização (RBAC por roles), **BCrypt** (cost alvo).
- Criptografia em trânsito; segregação de dados por escola; princípio do menor privilégio.
- LGPD: consentimento, base legal, retenção/anonimização, atendimento a titulares, logging sem PII.
- Gestão de segredos (K8s Secrets), rotação, acesso auditável.
- Matriz de acessos (papéis x permissões).

Regras:
- Referencie (2) e (8) e fundamentos de SD (4).
- Use checklists e tabelas. Marque **[PENDÊNCIA]** onde faltar política.

```

```md
Gere a seção **“9) CI/CD & Deploy”**.

Inclua:
- Workflow reutilizável `.github/.github/workflows/reusable-java-k8s-pipeline.yml`.
- Por serviço: `Dockerfile`, `k8s/deployment.yaml` (imagem placeholder + probes Actuator + envFrom), `service.yaml`, `configmap.yaml`.
- Segredos por serviço: `kubectl create secret generic <svc>-secret ...`.
- Org secret: `KUBE_CONFIG_DATA` (kubeconfig Base64).
- Pipeline: build/test → build/push GHCR → patch imagem → `kubectl apply` → readiness ok → smoke.
- Rollback: `kubectl rollout undo` ou re-deploy da tag anterior.
- Checklist de pré-requisitos.

Regras:
- Cite (2–3) e princípios DevOps em Valente (5).
```


```md
Gere a seção **“10) Observabilidade”**.

Inclua:
- Health (Actuator liveness/readiness).
- Logs estruturados (JSON), Traces (OpenTelemetry), Métricas (Prometheus).
- Padrões: `tenant_id`, `correlation_id` em tudo.
- Dashboards (Grafana) e alertas (SLI/SLO p95/p99).
- Error budget e políticas de escalonamento.

Regras:
- Cite (3) complementos de operação; (4) falhas/monitoramento; (5) engenharia.

```

```md
Gere a seção **“11) Catálogo de Eventos (v1)”**.

Produza tabela com:
- Nome do evento, Tópico (padrão: `ds6.<tenant>.<context>.<event>.v1`), Produtor, Consumidores, Semântica, **Payload (JSON Schema resumido)**, Idempotência, Versionamento.

Eventos mínimos:
- `auth.user.logged.v1`
- `student.created.v1`, `student.updated.v1`
- `class.assignment.changed.v1`, `schedule.updated.v1`
- `attendance.recorded.v1`
- `grade.posted.v1`

Regras:
- Cite (4) pub/sub; (5) serviços; use **JSON** em bloco.


```

```md
Gere a seção **“12) Front/BFF — RFs e Contratos”**.

Para cada BFF (Auth, Students, Presence, Grades):
- **REST**: endpoints, parâmetros, exemplos de request/response (OpenAPI sketch).
- **WebSocket**: canais, mensagens (JSON), versionamento.
- **Commands (VM)**: liste por caso de uso (MVVM).
- **Critérios BDD**: Given/When/Then.
- **Offline-first**: filas/retry/reconciliação.

Regras:
- Aderente a **Flutter Web MVVM (ChangeNotifier)** com **Pages/Widgets “dumb”**; BFF como orquestrador.
- Cite dissertação (9) para split/composição/comunicação; (6) para BDD; (4–5) para EDD.
```

```md
Gere a seção **“13) Rastreabilidade”**.

Inclua uma **matriz** que relacione:
RF/RNF ↔ Serviço (repo) ↔ MFE ↔ BFF ↔ Endpoint/Evento ↔ Testes (unit/contract/e2e) ↔ ADR.

Regras:
- Tabela grande em Markdown; use IDs estáveis.
- Cite (8) para gestão de requisitos.

```

```md
Gere a seção **“14) ADRs (sumário)”**.

Liste ADRs com: ID, Título, Status, Contexto, Decisão, Consequências, Referências.
ADRs mínimas:
- MFE Split: Vertical + client-side composition (Shell).
- Integração: REST + WS; Pub/Sub (EDD); Filas 1→1.
- Tenancy: Single-tenant por instância (MVP).
- Dados: DB por serviço; Outbox; Idempotência.
- Observabilidade: correlação (tenant_id, correlation_id).

Regras:
- Cada ADR com justificativa e citação (4–5; 9).


```

```md
Gere a seção **“16) Anexos”**.

Inclua templates prontos:
- **Envelope de evento** (JSON completo).
- **Problem+JSON (RFC 7807)** de erros (modelo).
- **Checklist DoR/DoD** (história INVEST, BDD, contratos, observabilidade).
- **Runbooks**: bootstrap de secrets, rollback.
- **Glossário** (termos-chave do projeto).

Regras:
- Tudo em Markdown com blocos de código quando couber.
- Cite (6) INVEST/BDD; (7) TDD; (5) arquitetura; (4) eventos.
```