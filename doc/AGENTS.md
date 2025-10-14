# Repository Guidelines

## Estrutura de Projeto e Módulos
Cada domínio mora em um diretório `ds6-*-service`, com código produtivo em `src/main/java/com/ds6`, recursos em `src/main/resources`, testes em `src/test/java` e manifestos Kubernetes no subdiretório `k8s/`; use `.github/README.md` e `GEMINI.MD` para alinhar contratos e padrões antes de criar novas rotas ou tópicos. (.github/README.md; GEMINI.MD)

## Build, Test e Desenvolvimento
No diretório do serviço, rode `./gradlew clean build` para compilar com Java 17 e Flyway, `./gradlew test` para a suíte JUnit 5 e `./gradlew bootRun` para desenvolvimento local com Actuator; para validar contêineres, use `docker build -t ds6-admin-service:dev .` e `docker run -p 8080:8080 ds6-admin-service:dev`. (ds6-admin-service/build.gradle; ds6-admin-service/Dockerfile)

## Estilo de Código e Convenções
Padronize pacotes como `com.ds6.<contexto>`, classes em PascalCase, métodos/campos em camelCase e sufixos `Controller`, `Service` e `Repository` conforme a camada; mantenha adapters REST separados do domínio, use Lombok apenas para reduzir boilerplate e normalize erros HTTP com `ProblemDetail` (RFC 7807). (.github/README.md; ds6-admin-service/build.gradle)

## Diretrizes de Testes
Escreva testes unitários em `src/test/java`, com classes `AlgoServiceTest` e métodos `should<Comportamento>`; execute `./gradlew test --info` antes de abrir PR e priorize cenários de validação, persistência e contratos REST. Meta inicial de cobertura: ≥80% para módulos core, revisitada na etapa F0. (ds6-admin-service/build.gradle; .github/README.md)

## Commits e Pull Requests
Adote Conventional Commits (`feat`, `fix`, `docs`, `refactor`, `test`, `build`) com escopo opcional e descrição imperativa; PRs devem linkar requisito/issue, listar testes executados, anexar diffs de OpenAPI ou eventos alterados e indicar plano de rollback ou feature flag quando o impacto for amplo. (.github/README.md)

## Segurança e Configuração
Gerencie secrets via Kubernetes, propagando `tenant_id` e `correlation_id` em logs/traces; atualizações em `k8s/` exigem readiness/liveness consistentes e devem passar pelo workflow reutilizável com `KUBE_CONFIG_DATA` atualizado após qualquer rotação de credenciais. (GEMINI.MD; .github/README.md)
