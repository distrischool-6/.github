# Glossário de Endpoints da API DistriSchool (DS6)

Este documento serve como referência para os endpoints da API da plataforma DistriSchool.

---

##  Authentication Service (`/auth`)

Serviço responsável pelo registro e login de usuários.

### `POST /auth/register`
- **Descrição**: Registra um novo usuário no sistema.
- **Corpo da Requisição (JSON)**:
  ```json
  {
    "email": "user@example.com",
    "password": "mypassword123",
    "role": "USER"
  }
  ```

### `POST /auth/login`
- **Descrição**: Autentica um usuário e retorna um token de acesso.
- **Corpo da Requisição (JSON)**:
  ```json
  {
    "email": "user@example.com",
    "password": "mypassword123"
  }
  ```

---

## Student Service (`/students`)

Gerencia todas as operações relacionadas aos estudantes.

### `POST /students/create`
- **Descrição**: Cria um novo registro de estudante.
- **Corpo da Requisição (JSON)**:
  ```json
  {
    "name": "João Silva",
    "birthDate": "2000-05-15",
    "grade": "10º Ano",
    "classNumber": "10A",
    "address": "Rua das Flores, 123, Fortaleza",
    "phone": "85999999999"
  }
  ```

### `GET /students/all`
- **Descrição**: Retorna uma lista com todos os estudantes cadastrados.

### `GET /students/{id}`
- **Descrição**: Busca um estudante específico pelo seu ID.
- **Parâmetro de URL**: `id` (ID do estudante).

### `GET /students/search`
- **Descrição**: Procura por estudantes utilizando nome ou número de matrícula.
- **Parâmetros de Query**: `name` (string) ou `registration` (string).

### `POST /students/update/{id}`
- **Descrição**: Atualiza os dados de um estudante existente.
- **Parâmetro de URL**: `id` (ID do estudante).
- **Corpo da Requisição (JSON)**:
  ```json
  {
    "name": "João Silva Santos",
    "grade": "11º Ano",
    "address": "Avenida da Liberdade, 456, Porto",
    "phone": "+351987654321"
  }
  ```

### `GET /students/export/csv`
- **Descrição**: Exporta a lista de estudantes em formato CSV.

---

## Admin Service (`/teachers`)

Gerencia as operações relacionadas aos professores.

### `POST /teachers`
- **Descrição**: Cria um novo registro de professor.
- **Corpo da Requisição (JSON)**:
  ```json
  {
    "name": "João Silva",
    "discipline": "Matemática",
    "phone": "(11) 99999-9999"
  }
  ```

### `GET /teachers`
- **Descrição**: Retorna uma lista com todos os professores.

### `GET /teachers/{id}`
- **Descrição**: Busca um professor específico pelo seu ID.
- **Parâmetro de URL**: `id` (ID do professor).

### `PUT /teachers/{id}`
- **Descrição**: Atualiza os dados de um professor.
- **Parâmetro de URL**: `id` (ID do professor).
- **Corpo da Requisição (JSON)**:
  ```json
  {
    "name": "João Silva Santos",
    "discipline": "Matemática Avançada",
    "phone": "(11) 88888-8888"
  }
  ```

### `DELETE /teachers/{id}`
- **Descrição**: Remove o registro de um professor.
- **Parâmetro de URL**: `id` (ID do professor).

---

## Presence Service (`/attendances`)

Gerencia os registros de presença dos alunos.

### `POST /attendances`
- **Descrição**: Cria um novo registro de presença.
- **Corpo da Requisição (JSON)**:
  ```json
  {
    "studentId": "uuid-do-aluno",
    "classId": "uuid-da-turma",
    "date": "2023-10-27",
    "status": "PRESENT"
  }
  ```

### `GET /attendances`
- **Descrição**: Busca registros de presença. Pode ser filtrado por aluno, turma ou ambos.
- **Parâmetros de Query**: `studentId` (opcional), `classId` (opcional).

---

## Grade & Exams Service (`/grades`)

Gerencia as notas e avaliações dos alunos.

### `POST /grades`
- **Descrição**: Lança uma nova nota para um aluno.
- **Corpo da Requisição (JSON)**:
  ```json
  {
    "studentId": "uuid-do-aluno",
    "classId": "uuid-da-turma",
    "description": "Prova 1 - Teste A/B",
    "value": 8.5,
    "date": "2023-10-27"
  }
  ```

### `GET /grades`
- **Descrição**: Busca as notas. Pode ser filtrado por aluno, turma ou ambos.
- **Parâmetros de Query**: `studentId` (opcional), `classId` (opcional).

---

## Class Schedule Service (`/classes`)

Gerencia as turmas e a alocação de alunos.

### `POST /classes`
- **Descrição**: Cria uma nova turma.
- **Corpo da Requisição (JSON)**:
  ```json
  {
    "name": "Turma de Teste A/B",
    "shift": "Manhã",
    "teacherId": "uuid-do-professor"
  }
  ```

### `GET /classes`
- **Descrição**: Retorna uma lista com todas as turmas.

### `GET /classes/{id}`
- **Descrição**: Busca uma turma específica pelo seu ID.
- **Parâmetro de URL**: `id` (ID da turma).

### `DELETE /classes/{id}`
- **Descrição**: Remove uma turma.
- **Parâmetro de URL**: `id` (ID da turma).

### `POST /classes/{id}/students`
- **Descrição**: Adiciona um aluno a uma turma.
- **Parâmetro de URL**: `id` (ID da turma).
- **Corpo da Requisição (JSON)**:
  ```json
  {
    "studentId": "uuid-do-aluno"
  }
  ```

### `DELETE /classes/{classId}/students/{studentId}`
- **Descrição**: Remove um aluno de uma turma.
- **Parâmetros de URL**: `classId` (ID da turma), `studentId` (ID do aluno).
