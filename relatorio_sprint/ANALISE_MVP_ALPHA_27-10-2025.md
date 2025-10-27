# Análise de Progresso: Início da Fase F1 - MVP Alpha (27/10/2025)

Este documento detalha o estado atual dos microserviços em relação ao que foi planejado para a fase **F1 – MVP Alpha (27/Out – 29/Nov/2025)**, conforme o `DS6_Roadmap.md`.

---

## 1. Backend

O objetivo para o backend nesta fase é ter o núcleo operacional dos serviços de autenticação, usuários, alunos e presença, incluindo a publicação de eventos e padrões de resiliência.

### 1.1 `ds6-auth-service` & `ds6-user-service`
**Planejado:**
- Login, verificação, recovery e gestão de papéis.
- Publicação do evento `auth.user.logged.v1`.

**Análise:**
- **Status:** Parcialmente Concluído.
- **Feito:**
    - O `ds6-user-service` implementa o registro (`/auth/register`) e o login (`/auth/login`).
    - A autenticação parece ser baseada em JWT (`JwtServiceImpl`).
    - A gestão de papéis (`Role`) está presente no modelo `User`.
- **Faltando:**
    - **Publicação de Eventos:** Não foi encontrada implementação de Kafka ou a publicação do evento `auth.user.logged.v1`.
    - **Funcionalidades:** Não há endpoints para recuperação de senha (recovery) ou verificação de conta.
    - **Separação de Serviços:** As responsabilidades de autenticação e usuários estão centralizadas no `ds6-user-service`, enquanto o roadmap sugere dois serviços distintos. O `ds6-auth-service` existe, mas parece estar vazio.

### 1.2 `ds6-student-service`
**Planejado:**
- CRUD completo e busca de alunos.
- Funcionalidade de exportação.
- Publicação do evento `student.created.v1`.

**Análise:**
- **Status:** Concluído.
- **Feito:**
    - **CRUD e Busca:** O `StudentController` possui endpoints para criar, listar, buscar por ID, e buscar por nome ou matrícula.
    - **Exportação:** O endpoint `GET /students/export/csv` está implementado.
    - **Publicação de Eventos:** O `build.gradle` inclui a dependência `spring-kafka`, e o `StudentServiceImpl` injeta um `KafkaTemplate` para publicar o evento `student.created` após a criação de um aluno.

### 1.3 `ds6-presence-service`
**Planejado:**
- Registro simples de presença.
- Canal de comunicação via WebSocket (WS).
- Publicação do evento `attendance.recorded.v1`.

**Análise:**
- **Status:** Em Andamento.
- **Feito:**
    - O `AttendanceController` possui um endpoint `POST /attendances` para registrar a presença, o que atende ao requisito de "registro simples".
- **Faltando:**
    - **WebSocket:** Não foi encontrada nenhuma configuração ou dependência relacionada a WebSockets no projeto. A comunicação em tempo real não está implementada.
    - **Publicação de Eventos:** Assim como no `user-service`, não há implementação de Kafka para a publicação do evento `attendance.recorded.v1`.

### 1.4 Padrões de Arquitetura
**Planejado:**
- Implementação do **Outbox Pattern** para garantir a entrega de eventos.
- Uso do padrão **Problem+JSON (RFC 7807)** para erros de API.

**Análise:**
- **Status:** Não Iniciado.
- **Faltando:**
    - **Outbox Pattern:** Nenhum dos serviços analisados apresenta uma implementação visível do padrão Outbox (seja via tabelas de outbox, Debezium ou outra ferramenta). A publicação de eventos no `student-service` é feita diretamente no código, o que pode levar a inconsistências em caso de falha.
    - **Problem+JSON:** Não foram encontrados `ControllerAdvice` ou tratamentos de exceção que padronizem as respostas de erro para o formato RFC 7807. As exceções atuais, como `ResourceNotFoundException`, retornam o status HTTP correspondente, mas sem o corpo padronizado.

---

## 2. BFF & Frontend

**Planejado:**
- Criação dos BFFs iniciais (`ds6-auth-bff`, `ds6-students-bff`, `ds6-presence-bff`).
- Criação do Shell Flutter e dos Micro-Frontends (MFEs) para Alunos e Presença.

**Análise:**
- **Status:** Não Iniciado.
- **Faltando:**
    - Não foram encontrados repositórios ou pastas correspondentes aos BFFs ou aos projetos de frontend em Flutter. Esta parte do planejamento para a F1 parece não ter sido iniciada.

---

## 3. Funcionalidades Adiantadas

Alguns serviços que estavam planejados para fases futuras já possuem uma implementação base:

- **`ds6-grade-exams-service` (Planejado para F1.2):** Já possui endpoints para criação e consulta de notas.
- **`ds6-class-schedule-service` (Não detalhado no MVP Alpha):** Já possui CRUD para turmas e gerenciamento de alunos.
- **`ds6-admin-service` (Não detalhado no MVP Alpha):** Já possui CRUD para professores.

---

## 4. Conclusão e Próximos Passos

**Resumo do Status para 27/10/2025:**
- **Backend:** O núcleo CRUD dos serviços está parcialmente de pé, com o `ds6-student-service` sendo o mais completo e aderente ao plano. Os principais GAPs são a falta de comunicação via eventos (exceto no `student-service`), a ausência de WebSockets e a não implementação dos padrões de arquitetura (Outbox, Problem+JSON).
- **Frontend/BFF:** Nenhum trabalho parece ter sido iniciado.

**Recomendações Imediatas:**
1.  **Priorizar Eventos e Padrões:** Implementar a publicação de eventos com o padrão Outbox nos serviços de `user` e `presence` para garantir a consistência da arquitetura. Adotar o `Problem+JSON` para padronizar os erros.
2.  **Iniciar Frontend:** Começar o desenvolvimento do Shell Flutter e dos BFFs, pois são um bloqueio para a validação ponta-a-ponta das funcionalidades.
3.  **Implementar WebSocket:** Adicionar a camada de WebSocket no `ds6-presence-service` para cumprir o requisito de tempo real.
4.  **Revisar Separação Auth/User:** Decidir se a responsabilidade de autenticação será movida para o `ds6-auth-service` ou se o serviço será descontinuado em favor de um `user-service` unificado.
