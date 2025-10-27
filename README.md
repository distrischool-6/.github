# DS6 — DistriSchool (Organização)
> **Guia oficial da organização DS6 no GitHub** · README de topo com visão, arquitetura, repositórios, CI/CD e **roadmap**.

<p align="center">
  <em>Back-end Spring Boot (MVC) + EDD · Front-end Flutter Web (Micro-Frontends + MVVM) · BFF por domínio com DDD · Kubernetes · PostgreSQL · Kafka · GitHub Actions</em>
</p>

---

## 📌 TL;DR
O **DS6 — DistriSchool** é uma plataforma de **gestão escolar distribuída** com:
- **Microsserviços Spring Boot** (REST + **eventos Kafka**) e **dados por serviço** (Outbox + Idempotência).
- **Single-tenant por instância** no MVP (um namespace + DB + tópicos por escola), já preparada para evolução futura.
- **Frontend Flutter Web** em **Micro-Frontends (Vertical Split)** com **MVVM (ChangeNotifier)** e **BFF por domínio** (**DDD**).
- **DevOps**: Kubernetes, GitHub Actions (workflow reutilizável), PostgreSQL + Flyway, observabilidade (OTel/Prometheus/Grafana), **LGPD by design**.

➡️ **Roadmap completo**: veja **[DS6_Roadmap.md](./DS6_Roadmap.md)** (este repositório).

---

## 🧭 O que é este projeto
A missão do DS6 é digitalizar processos de escolas privadas (matrícula, turmas, presença, notas, comunicação e relatórios) garantindo **desacoplamento**, **segurança de dados**, **observabilidade** e **escala**, com uma experiência web moderna e **offline-first** quando necessário.

**Arquitetura-alvo (resumo):**
- **Backend**: Spring Boot 3.x; **REST** para request/response; **Kafka (pub/sub)** para integração; **PostgreSQL** por serviço; **Outbox**; **idempotência**; **RFC 7807** para erros.
- **Frontend**: **Flutter Web** com **Micro-Frontends (Vertical Split)**, **MVVM** (Pages/Widgets “dumb”; VM orquestra); **BFF por domínio** com **DDD** (use cases).
- **DevOps**: K8s; imagens no **GHCR**; secrets via **Kubernetes Secrets**; CI/CD central reutilizável; **liveness/readiness Actuator**; logs/traces/métricas com `tenant_id` e `correlation_id`.

---

## 🗂️ Repositórios (mapa)
> Substitua `<ORG_NAME>` pelo nome da sua organização no GitHub.

- **Back-end (serviços)**  
  - `https://github.com/<ORG_NAME>/ds6-auth-service` — Autenticação (JWT), verificação, recuperação; evento `auth.user.logged.v1`.
  - `https://github.com/<ORG_NAME>/ds6-user-service` — Usuários/roles.
  - `https://github.com/<ORG_NAME>/ds6-student-service` — Estudantes CRUD, busca; `student.created.v1`.
  - `https://github.com/<ORG_NAME>/ds6-class-schedule-service` — Turmas/disciplinas/horários; conflitos; importação.
  - `https://github.com/<ORG_NAME>/ds6-presence-service` — Registro de presença; WS; `attendance.recorded.v1`.
  - `https://github.com/<ORG_NAME>/ds6-grade-exams-service` — Notas/avaliações; `grade.posted.v1`.
  - `https://github.com/<ORG_NAME>/ds6-admin-service` — Administração/feature flags.
- **BFFs (por domínio)**  
  - `https://github.com/<ORG_NAME>/ds6-auth-bff`, `ds6-students-bff`, `ds6-presence-bff`, `ds6-grades-bff` (REST + WS, OpenAPI/JSON Schema).
- **Frontend (Flutter Web, MFEs)**  
  - `https://github.com/<ORG_NAME>/ds6-shell-web` — Orquestrador/roteador (Application Shell).
  - `https://github.com/<ORG_NAME>/ds6-students-web`, `ds6-presence-web`, `ds6-grades-web`, `ds6-auth-web` (MVVM/ChangeNotifier, Pages/Widgets “dumb”).

---

## 🏗️ Padrões de CI/CD & Deploy (Kubernetes)
- **Workflow reutilizável**: `.github/.github/workflows/reusable-java-k8s-pipeline.yml` (na raiz da org).
- Em cada serviço/BFF, criar `.github/workflows/main-pipeline.yml` chamando o workflow reutilizável e passando `service-name`.
- **Manifests por serviço** (em `k8s/`): `deployment.yaml` (com `image: imagem-placeholder`), `service.yaml`, `configmap.yaml` + probes Actuator.
- **Secrets**: por serviço `kubectl create secret generic <svc>-secret ...`; e **org secret** `KUBE_CONFIG_DATA` (kubeconfig em Base64).
- Pipeline: build/test → build/push **GHCR** → patch de imagem → `kubectl apply` → readiness OK → smoke.

> Exemplo de workflow por serviço:
```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [ "main" ]
jobs:
  call-reusable-workflow:
    uses: ./.github/.github/workflows/reusable-java-k8s-pipeline.yml
    permissions:
      contents: read
      packages: write
    with:
      service-name: "ds6-student-service"
    secrets:
      inherit
```

---

## 🧩 Convenções de Front (Flutter Web)
- **Micro-Frontends (Vertical Split)**; composição **client-side** via `ds6-shell-web` (um MFE por rota).
- **MVVM (ChangeNotifier)**: **Pages/Widgets “dumb”** (Atomic Design); **ViewModel** como único orquestrador (Commands → REST/WS).
- **Offline-first**: fila local + retry + reconciliação por eventos; indicadores de conectividade na VM.

---

## 🗺️ Roadmap (resumo)
> O arquivo completo está em **[DS6_Roadmap.md](./DS6_Roadmap.md)**. Abaixo, o macro cronograma.

| Fase | Período | Objetivo | Entregas Chave | Exit Criteria |
|---|---|---|---|---|
| **F0 – Fundações** | **13–24/Out/2025** | Padrões & trilhos | SRS v1.0; library; CI/CD; k8s/; secrets | SRS assinado; 100% repos com pipeline verde |
| **F1 – MVP Alpha** | **27/Out – 29/Nov/2025** | Núcleo | Auth/Users; Students; Presence básico; BFFs; Shell+MFE; offline “skeleton” | Pilot interno; login p95<2s; uptime 99%/sem |
| **F1.1 – MVP Beta** | **01–20/Dez/2025** | Piloto Escola #1 | Export/Import (Pipes&Filters v1); Notifications; dashboards; DR drill | Escola #1 2 semanas; RPO≤15m RTO≤1h |
| **F1.2 – GA** | **05–31/Jan/2026** | Go-live (3 escolas) | Grades; reports; LGPD inicial; restore test | 3 escolas on; NPS>7; SLOs batidos |
| **F2 – Escala** | **Fev–Mar/2026** | Control Plane | IaC para “create-school”; quotas; contrato eventos v1 **freeze** | Provisionamento ≤15min; zero cross-tenant |
| **F3 – Evolução** | **Abr–Jun/2026** | Performance & custo | Caching; WS resiliente; POC multi-tenant (lab) | p95 páginas <2s; custo/tenant ↓20% |
| **F4 – Produto 2.0** | **Jul–Set/2026** | Experiência & governança | Design tokens; notif. avançadas; governança MFE | dívida técnica ≤10% WIP |

---

## 🔒 Segurança & LGPD (essencial)
- **JWT** + **RBAC**; **BCrypt** (cost ≥12); TLS 1.3.
- **Segregação por escola** (MVP): namespace + DB + tópicos + segredos.
- **LGPD**: minimização de PII; retenção/anonimização; atendimento a titular; auditoria.
- **Observabilidade**: `tenant_id` e `correlation_id` em logs/traces/métricas.

---

## 🧪 Qualidade & Contratos
- **Glossário da API**: A documentação de referência para todos os endpoints está em **[doc/API_Endpoints.md](./doc/API_Endpoints.md)**.
- **OpenAPI** (REST) e **JSON Schema** (WS/eventos) são definidos em um repositório central de contratos.
- **Contract tests** em CI (quebras exigem bump de versão).
- **TDD/BDD** nas histórias críticas (Given/When/Then).

---

## 🤝 Contribuição
- Use **ADRs** para decisões arquiteturais significativas.
- Commits convencionais (feat/fix/chore/docs/refactor/test/build).
- PRs com testes, descrição clara e link para RF/RNF/issue.

---

## 📑 Licença & Contato
- Licença: [PENDENTE].
- Contato técnico: `@gabrieladeraldo` / Squad DS6.

---

> **Referências** (resumo): Template & requisitos do cliente; *Sistemas Distribuídos* (Coulouris); *Engenharia de Software Moderna* (Valente); *Histórias de Usuário* (2022); *TDD* (Kent Beck); Dissertação (GAM Micro-Frontends).

---

# AVISO IMPORTANTE SOBRE COMMITS

**COMMITs usando `MELHORIA` no começo só vão entrar na versão da segunda entrega.**