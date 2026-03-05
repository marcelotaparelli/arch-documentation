# arch-documentation

## 1. Estilo de Arquitetura

**Escolha e documente o padrão base**

- [ ] Definir o estilo: Monolito Modular, Microserviços, Event-Driven, ou Hexagonal (Ports & Adapters)
- [ ] Aplicar **Clean Architecture**: Domain → Application → Interface Adapters → Infrastructure. O domínio nunca depende de frameworks
- [ ] Usar **DDD (Domain-Driven Design)**: mapear Bounded Contexts, linguagem ubíqua por domínio, sem banco compartilhado entre contextos
- [ ] Documentar **ADRs (Architecture Decision Records)**: arquivo markdown versionado no repo com contexto, decisão e consequências de cada escolha arquitetural
- [ ] Definir comunicação entre serviços: **síncrona** (REST/gRPC) para queries, **assíncrona** (Kafka, RabbitMQ) para eventos e operações tolerantes a latência
- [ ] Criar **C4 Model**: diagramas de Contexto, Contêineres e Componentes — versionados junto ao código

---

## 2. Autenticação & Autorização

**Identidade, sessões e permissões**

- [ ] Usar **OAuth 2.0 + OpenID Connect (OIDC)** com Authorization Code + PKCE. Nunca implementar OAuth do zero — usar Keycloak, Auth0, Cognito ou similar
- [ ] **JWT com rotação de tokens**: access token com TTL curto (15 min), refresh token com TTL longo (7–30 dias) em cookie `HttpOnly; Secure; SameSite=Strict`. Implementar rotação de família de tokens para detectar reuso
- [ ] **MFA obrigatório** para ações sensíveis: TOTP (Authenticator), SMS e/ou WebAuthn/Passkeys
- [ ] **RBAC ou ABAC**: Role-Based para sistemas simples, Attribute-Based (OPA/Open Policy Agent, Casbin) para políticas complexas e dinâmicas
- [ ] **Princípio do menor privilégio**: usuários, serviços e processos com permissões mínimas necessárias
- [ ] **Proteção contra brute force**: rate limiting por IP e por conta, lockout progressivo, CAPTCHA para IPs suspeitos
- [ ] **Hash seguro de senhas**: Argon2id ou bcrypt (cost ≥ 12). Nunca MD5/SHA-1. Validar contra lista de senhas vazadas (HaveIBeenPwned API)
- [ ] **Proteção de sessão**: invalidar tokens no logout, blacklist de tokens revogados, detectar session hijacking por fingerprint de device/IP

---

## 3. Sistema de Logs

**Observabilidade, auditoria e troubleshooting**

- [ ] **Logs estruturados em JSON**: campos obrigatórios: `timestamp` (ISO 8601), `level`, `service`, `traceId`, `userId`, `requestId`, `message`, `duration_ms`
- [ ] **Níveis corretos**: ERROR (falha crítica), WARN (anomalia sem falha), INFO (evento de negócio), DEBUG (detalhe técnico, desligado em produção), TRACE (apenas dev)
- [ ] **Correlation ID / Trace ID propagado**: UUID único por request, propagado via headers (`traceparent` W3C) em todos os serviços e presente em todos os logs
- [ ] **Log de auditoria imutável**: registrar toda ação sensível — quem fez, o quê, quando, de onde (IP), antes e depois. Armazenar em storage append-only separado
- [ ] **Nunca logar dados sensíveis**: mascarar CPF, cartão, senha, token, PII. Implementar log scrubbing automático no pipeline
- [ ] **Centralização**: enviar logs para stack centralizada (ELK Stack, Loki+Grafana, Datadog, CloudWatch). Retenção mínima de 90 dias, 1 ano para auditoria
- [ ] **Alertas em tempo real**: notificação imediata para ERRORs em produção, anomalias de volume, falhas de autenticação em sequência

---

## 4. Observabilidade (além dos logs)

**Os 3 pilares: Logs, Métricas e Traces**

- [ ] **Distributed Tracing**: instrumentar com OpenTelemetry (padrão agnóstico). Visualizar no Jaeger, Zipkin ou Tempo
- [ ] **Métricas**: expor métricas no padrão Prometheus — latência (p50, p95, p99), taxa de erros, throughput, saturação de recursos
- [ ] **Health checks**: endpoint `/health/live` (app está de pé?) e `/health/ready` (app está pronto para receber tráfego?)
- [ ] **Dashboards operacionais**: Grafana com painéis por serviço — latência, erros, CPU/memória, conexões de banco
- [ ] **SLOs e SLAs definidos**: ex: 99.9% de disponibilidade, p99 < 500ms. Error budget documentado e monitorado
- [ ] **Alertas com runbook**: cada alerta deve ter um runbook associado explicando como investigar e resolver

---

## 5. Segurança da Aplicação (AppSec)

**OWASP Top 10 e além**

- [ ] **Proteção contra Injection** (SQL, NoSQL, Command): usar ORM/query builders com prepared statements. Nunca concatenar input do usuário em queries
- [ ] **Validação e sanitização de input**: validar no servidor (nunca confiar no client). Usar schemas de validação (Zod, Joi, Yup, class-validator)
- [ ] **Headers de segurança HTTP**: `Content-Security-Policy`, `Strict-Transport-Security`, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`. Usar Helmet.js ou similar
- [ ] **CORS configurado corretamente**: whitelist explícita de origens. Nunca `Access-Control-Allow-Origin: *` em APIs autenticadas
- [ ] **Proteção CSRF**: tokens CSRF para formulários, validar `Origin`/`Referer` header para APIs
- [ ] **Rate limiting global e por endpoint**: proteção contra DDoS e abuso de API. Implementar no API Gateway ou middleware (Redis-backed)
- [ ] **Dependency scanning**: Snyk, Dependabot ou OWASP Dependency-Check no CI/CD. Nenhuma vulnerabilidade crítica em produção
- [ ] **SAST (Static Analysis)**: SonarQube, CodeQL ou Semgrep rodando no pipeline para detectar vulnerabilidades no código
- [ ] **Secrets nunca no código**: usar variáveis de ambiente + secret manager (AWS Secrets Manager, HashiCorp Vault, Azure Key Vault). Implementar rotação automática de secrets
- [ ] **TLS 1.2+ obrigatório em todo tráfego**: HTTPS everywhere, certificados gerenciados automaticamente (Let's Encrypt, ACM)

---

## 6. Banco de Dados

**Integridade, performance e resiliência**

- [ ] **Migrations versionadas**: Flyway, Liquibase ou Alembic. Nunca alterar banco manualmente em produção
- [ ] **Índices nas colunas certas**: analisar query plans (EXPLAIN ANALYZE). Índices em FKs, colunas de filtro frequente e ordenação
- [ ] **Backups automáticos testados**: backup diário com retenção configurada. Testar restore periodicamente (disaster recovery drill)
- [ ] **Connection pooling**: PgBouncer, HikariCP ou equivalente. Nunca abrir uma conexão por request
- [ ] **Read replicas para queries pesadas**: separar workload de leitura do de escrita
- [ ] **Dados sensíveis criptografados at rest**: campos como CPF, dados financeiros, saúde — criptografia em nível de coluna ou disco
- [ ] **Soft delete com auditoria**: nunca deletar dados permanentemente sem política clara. Campos `deleted_at`, `updated_at`, `created_at` em toda entidade
- [ ] **Transações e idempotência**: operações críticas em transações. APIs de mutação idempotentes com chave de idempotência

---

## 7. API Design

**Contratos claros, versionados e seguros**

- [ ] **Versionamento de API**: `/v1/`, `/v2/` na URL ou via header `Accept: application/vnd.api+json;version=2`. Nunca quebrar contratos sem deprecation period
- [ ] **OpenAPI/Swagger atualizado**: documentação gerada a partir do código (contract-first ou code-first). Compartilhada com frontend e parceiros
- [ ] **Paginação consistente**: cursor-based (mais escalável) ou offset-based. Campos padronizados: `data`, `meta.total`, `meta.cursor`
- [ ] **Respostas de erro padronizadas**: formato RFC 7807 (Problem Details): `type`, `title`, `status`, `detail`, `instance`. Nunca expor stack trace em produção
- [ ] **Idempotency keys**: endpoints de criação e pagamento aceitam `Idempotency-Key` header para evitar duplicações em retry
- [ ] **Timeout e circuit breaker**: definir timeout em todas as chamadas externas. Circuit breaker (Resilience4j, Polly) para isolar falhas de dependências

---

## 8. CI/CD e DevOps

**Entrega contínua com qualidade e segurança**

- [ ] **Pipeline completo**: Lint → Testes Unitários → Testes de Integração → SAST → Build → Deploy para staging → Smoke tests → Deploy para produção
- [ ] **Cobertura de testes mínima**: 80% de cobertura como gate no CI. Testes de unidade, integração e E2E (Cypress, Playwright)
- [ ] **Infraestrutura como Código (IaC)**: Terraform ou Pulumi para toda infraestrutura. Nunca criar recursos manualmente em produção
- [ ] **Ambientes isolados**: Development, Staging (espelho da produção) e Production. Dados reais nunca em staging
- [ ] **Feature flags**: LaunchDarkly, Unleash ou similar para deploy desacoplado de release. Permite rollback instantâneo sem redeploy
- [ ] **Rollback automatizado**: deploy com canary release (5% → 25% → 100%) ou blue/green. Rollback automático em caso de aumento de erros
- [ ] **Container security**: imagens Docker com base mínima (distroless), scan de vulnerabilidades (Trivy, Snyk Container), nunca rodar como root

---

## 9. Resiliência e Escalabilidade

**O sistema sobrevive a falhas e cresce com a demanda**

- [ ] **Design for failure**: assumir que tudo vai falhar. Implementar retry com exponential backoff e jitter em todas as chamadas externas
- [ ] **Idempotência em mensageria**: consumidores de eventos devem ser idempotentes — processar a mesma mensagem duas vezes deve ser seguro
- [ ] **Dead Letter Queue (DLQ)**: mensagens que falharam após N tentativas vão para DLQ com alerta. Nunca perder mensagens silenciosamente
- [ ] **Graceful shutdown**: ao receber SIGTERM, a aplicação para de aceitar novas requisições, termina as em andamento e fecha conexões de banco/fila
- [ ] **Auto-scaling configurado**: Horizontal Pod Autoscaler (Kubernetes) ou equivalente baseado em CPU, memória e métricas customizadas
- [ ] **Cache estratégico**: Redis ou Memcached para dados quentes. Cache-aside pattern. TTL definido. Invalidação explícita quando necessário
- [ ] **CDN para assets estáticos**: CloudFront, Fastly ou Cloudflare. Nunca servir imagens e JS direto do servidor de aplicação

---

## 10. Compliance e Privacidade

**LGPD/GDPR e regulatórios**

- [ ] **Mapeamento de dados pessoais**: documentar quais dados PII são coletados, onde ficam, por quanto tempo e com qual base legal
- [ ] **Direitos do titular implementados**: endpoints para exportar dados do usuário, deletar conta (right to erasure), e portabilidade de dados
- [ ] **Consentimento granular**: registrar quando e como o usuário consentiu. Guardar log de consentimento imutável
- [ ] **Data minimization**: coletar apenas o necessário. Revisar periodicamente campos que podem ser removidos
- [ ] **Política de retenção**: dados com TTL definido e deleção automatizada após o prazo. Backups também devem seguir a política
- [ ] **Pen testing periódico**: teste de penetração por equipe externa ao menos anualmente ou antes de grandes lançamentos

---

## 11. Qualidade de Código e Governança

**Sustentabilidade técnica a longo prazo**

- [ ] **Code review obrigatório**: PRs com ao menos 1 aprovação. Checklist de review: segurança, performance, testes, legibilidade, ADR atualizado?
- [ ] **Linting e formatação automáticos**: ESLint, Prettier, Black, Ruff, Checkstyle — rodando no pre-commit e no CI
- [ ] **Sem secrets no git**: pre-commit hook com detect-secrets ou gitleaks. Rotacionar imediatamente se vazar
- [ ] **Documentação próxima ao código**: README por serviço com: propósito, como rodar localmente, variáveis de ambiente, dependências
- [ ] **Dívida técnica gerenciada**: backlog de tech debt visível, priorizado e alocado em cada sprint (regra: 20% do tempo)
- [ ] **Onboarding documentado**: um novo dev deve conseguir rodar o projeto do zero em menos de 30 minutos seguindo o README

---

> **Regra de ouro do arquiteto:** a melhor arquitetura é a mais simples que resolve o problema atual e deixa espaço para evoluir. Complexidade prematura é o inimigo.
