# AI-Context: Agents & Workflow PREVC

## 📋 Índice

1. [O que é AI-Context?](#o-que-é-ai-context)
2. [Sistema de Agents](#sistema-de-agents)
3. [Workflow PREVC](#workflow-prevc)
4. [Plans & Task Management](#plans--task-management)
5. [Skills dos Agents](#skills-dos-agents)
6. [Como Usar](#como-usar)
7. [Exemplos Práticos](#exemplos-práticos)
8. [Best Practices](#best-practices)

---

## O que é AI-Context?

**AI-Context** é um sistema de orquestração de agentes especializados que trabalham em conjunto seguindo um workflow estruturado (PREVC) para desenvolvimento de software com qualidade.

### Principais Componentes

1. **Agents** - Agentes especializados em diferentes aspectos do desenvolvimento
2. **Workflow PREVC** - Processo estruturado em 5 fases
3. **Plans** - Planejamento e tracking de tarefas
4. **Context Management** - Documentação e contexto do projeto

### Benefícios

✅ Desenvolvimento estruturado e previsível  
✅ Separação clara de responsabilidades  
✅ Quality gates em cada fase  
✅ Documentação automática do processo  
✅ Handoffs claros entre diferentes especialistas  
✅ Rastreabilidade de decisões  

---

## Sistema de Agents

### Agents Disponíveis

O AI-Context possui **14 agentes especializados** integrados:

| Agent | Descrição | Fases Principais |
|-------|-----------|------------------|
| **code-reviewer** | Revisa código para qualidade, estilo e best practices | R, V |
| **bug-fixer** | Identifica e corrige bugs com soluções direcionadas | E |
| **feature-developer** | Implementa novas features seguindo arquitetura | E |
| **refactoring-specialist** | Melhora estrutura do código e elimina code smells | E |
| **test-writer** | Cria suites de testes abrangentes | V |
| **documentation-writer** | Escreve e mantém documentação | P, C |
| **performance-optimizer** | Identifica e resolve gargalos de performance | V |
| **security-auditor** | Audita código para vulnerabilidades de segurança | R, V |
| **backend-specialist** | Desenvolve lógica server-side e APIs | E |
| **frontend-specialist** | Constrói interfaces e interações de usuário | E |
| **architect-specialist** | Desenha arquitetura de sistema e padrões | P, R |
| **devops-specialist** | Gerencia deployment e pipelines CI/CD | C |
| **database-specialist** | Desenha e otimiza soluções de banco de dados | E |
| **mobile-specialist** | Desenvolve aplicações mobile | E |

### Características dos Agents

Cada agent possui:

- **Especialização** - Foco em uma área específica do desenvolvimento
- **Context Awareness** - Acesso à documentação relevante do projeto
- **Skills** - Conjunto de habilidades e ferramentas específicas
- **Documentação primária** - Documentos que consultam prioritariamente

### Descoberta de Agents

```bash
# Listar todos os agents
mcp_ai-context_agent discover

# Ver detalhes de um agent
mcp_ai-context_agent getInfo --agentType=feature-developer

# Ver documentação de um agent
mcp_ai-context_agent getDocs --agent=code-reviewer
```

---

## Workflow PREVC

### O que é PREVC?

**PREVC** é um workflow estruturado em 5 fases que garante qualidade e consistência no desenvolvimento:

```
P → R → E → V → C
│   │   │   │   │
│   │   │   │   └─ Complete/Confirmation
│   │   │   └───── Validation
│   │   └───────── Execution
│   └───────────── Review
└───────────────── Planning
```

### Fase 1: P - Planning (Planejamento)

**Objetivo**: Planejar a feature/mudança antes de implementar

**Agents Recomendados**:
- `architect-specialist` - Design de arquitetura
- `documentation-writer` - Documentação do plano
- `frontend-specialist` - Design de UI/UX (se aplicável)

**Atividades**:
- Análise de requisitos
- Design de arquitetura
- Definição de escopo
- Criação de ADRs (Architecture Decision Records)
- Planejamento de tarefas

**Documentação Consultada**:
- Architecture
- Glossary
- Documentation Index

**Gates de Saída**:
- ✅ Plano documentado e aprovado (se `require_plan=true`)
- ✅ Arquitetura definida
- ✅ Tarefas quebradas em steps

**Exemplo**:
```markdown
# Plan: Adicionar Filtros Avançados em Conversas

## Objetivo
Implementar filtros avançados para busca de conversas

## Arquitetura
- Adicionar scope `with_advanced_filters` no model Conversation
- Criar service `ConversationFilterService`
- Adicionar componente Vue `AdvancedFilters.vue`

## Tarefas
1. Backend: Criar scope e service
2. Frontend: Componente de filtros
3. Testes: RSpec + Vitest
4. Documentação: Atualizar API.md
```

---

### Fase 2: R - Review (Revisão de Design)

**Objetivo**: Revisar design e arquitetura antes da implementação

**Agents Recomendados**:
- `architect-specialist` - Validação de arquitetura
- `code-reviewer` - Review de padrões
- `security-auditor` - Review de segurança

**Atividades**:
- Review de design
- Validação de padrões
- Security assessment
- Identificação de riscos
- Ajustes no plano

**Documentação Consultada**:
- Architecture
- Security
- Data Flow

**Gates de Saída**:
- ✅ Design aprovado (se `require_approval=true`)
- ✅ Sem vulnerabilidades críticas identificadas
- ✅ Padrões validados

**Exemplo**:
```markdown
# Design Review - Filtros Avançados

## ✅ Aprovações
- Arquitetura: Aprovada
- Segurança: Sem issues críticos
- Performance: N+1 queries prevenidos

## ⚠️ Observações
- Adicionar índice em conversations(status, priority, created_at)
- Limitar número máximo de filtros (max 10)
- Implementar rate limiting no endpoint
```

---

### Fase 3: E - Execution (Execução/Implementação)

**Objetivo**: Implementar a feature seguindo o plano aprovado

**Agents Recomendados**:
- `feature-developer` - Implementação geral
- `backend-specialist` - APIs e lógica servidor
- `frontend-specialist` - UI/UX
- `database-specialist` - Queries e migrações
- `mobile-specialist` - Apps mobile (se aplicável)
- `bug-fixer` - Correções durante dev

**Atividades**:
- Implementação do código
- Criação de migrations
- Desenvolvimento de UI
- Integração de componentes
- Resolução de issues durante dev

**Documentação Consultada**:
- Architecture
- API Reference
- Data Flow
- Getting Started

**Gates de Saída**:
- ✅ Código implementado
- ✅ Builds passando
- ✅ Lint sem erros
- ✅ Funcionalidade básica funcionando

**Exemplo**:
```ruby
# app/services/conversation_filter_service.rb
class ConversationFilterService
  MAX_FILTERS = 10

  def initialize(account:, filters: {})
    @account = account
    @filters = filters.slice(*allowed_filter_keys).first(MAX_FILTERS)
  end

  def perform
    conversations = @account.conversations
    apply_filters(conversations)
  end

  private

  def apply_filters(scope)
    @filters.reduce(scope) do |result, (key, value)|
      apply_filter(result, key, value)
    end
  end
  
  # ...
end
```

---

### Fase 4: V - Validation (Validação)

**Objetivo**: Validar qualidade, testes e performance

**Agents Recomendados**:
- `test-writer` - Criação de testes
- `code-reviewer` - Review de código
- `security-auditor` - Auditoria de segurança
- `performance-optimizer` - Otimização de performance

**Atividades**:
- Criação de testes (unit, integration, e2e)
- Code review
- Security audit
- Performance testing
- Coverage analysis

**Documentação Consultada**:
- Testing
- Security
- API Reference

**Gates de Saída**:
- ✅ Testes passando (coverage adequado)
- ✅ Code review aprovado
- ✅ Security scan sem vulnerabilidades
- ✅ Performance aceitável

**Exemplo**:
```ruby
# spec/services/conversation_filter_service_spec.rb
RSpec.describe ConversationFilterService do
  describe '#perform' do
    let(:account) { create(:account) }
    let(:service) { described_class.new(account: account, filters: filters) }

    context 'with status filter' do
      let(:filters) { { status: 'open' } }
      
      it 'returns only open conversations' do
        open_conv = create(:conversation, account: account, status: :open)
        create(:conversation, account: account, status: :resolved)
        
        expect(service.perform).to eq([open_conv])
      end
    end
    
    context 'with multiple filters' do
      # ...
    end
  end
end
```

---

### Fase 5: C - Complete/Confirmation (Finalização)

**Objetivo**: Finalizar, documentar e preparar para deploy

**Agents Recomendados**:
- `documentation-writer` - Documentação final
- `devops-specialist` - Deploy e CI/CD

**Atividades**:
- Documentação final
- Changelog
- Release notes
- Deploy preparation
- Handoff para produção

**Documentação Consultada**:
- Deployment
- Documentation Index
- Contributing

**Gates de Saída**:
- ✅ Documentação atualizada
- ✅ Changelog escrito
- ✅ Deploy checklist completo
- ✅ PR merged

**Exemplo**:
```markdown
# Release Notes - v3.2.0

## 🎉 New Features

### Filtros Avançados em Conversas
- Filtre conversas por múltiplos critérios (status, priority, assignee, labels)
- Suporte para até 10 filtros simultâneos
- Rate limiting implementado (100 req/min)

## 📚 Documentation
- Atualizado API.md com novos endpoints
- Adicionado exemplo de uso em DEVELOPMENT.md

## 🔒 Security
- Rate limiting implementado
- Input validation em todos os filtros

## 🚀 Deploy Notes
- Nova migration: `add_index_to_conversations`
- Nova variável de env: `CONVERSATION_FILTER_MAX` (default: 10)
```

---

## Gestão de Workflow

### Inicializar Workflow

```bash
# Iniciar workflow para uma feature
mcp_ai-context_workflow-manage init --name=advanced-filters

# Verificar status
mcp_ai-context_workflow-status
```

### Avançar Fases

```bash
# Avançar para próxima fase (P → R)
mcp_ai-context_workflow-advance

# Forçar avanço (bypass gates)
mcp_ai-context_workflow-advance --force

# Completar fase com outputs
mcp_ai-context_workflow-advance --outputs=["plan.md", "architecture.md"]
```

### Handoffs entre Agents

```bash
# Transferir trabalho entre agents
mcp_ai-context_workflow-manage handoff \
  --from=feature-developer \
  --to=test-writer \
  --artifacts=["ConversationFilterService", "advanced_filters_spec.rb"]
```

### Colaboração

```bash
# Iniciar sessão de colaboração
mcp_ai-context_workflow-manage collaborate \
  --topic="Review filtros avançados" \
  --participants=["architect-specialist", "security-auditor"]
```

### Aprovar Plan

```bash
# Aprovar plano (necessário para P→R se require_approval=true)
mcp_ai-context_workflow-manage approvePlan \
  --planSlug=advanced-filters \
  --approver=architect \
  --notes="Arquitetura aprovada com ajustes"
```

### Modo Autônomo

```bash
# Ativar modo autônomo (bypass gates automaticamente)
mcp_ai-context_workflow-manage setAutonomous \
  --enabled=true \
  --reason="Feature simples, não requer aprovações"

# Desativar
mcp_ai-context_workflow-manage setAutonomous \
  --enabled=false
```

---

## Plans & Task Management

### O que são Plans?

**Plans** são documentos estruturados que descrevem:
- Objetivo da tarefa
- Steps necessários
- Fase PREVC atual
- Status de cada step
- Decisões tomadas

### Estrutura de um Plan

```yaml
# .context/plans/advanced-filters.yaml
name: advanced-filters
title: Implementar Filtros Avançados
summary: Adicionar filtros avançados na listagem de conversas
status: in_progress
current_phase: E

phases:
  planning:
    status: completed
    steps:
      - name: Análise de requisitos
        status: completed
        output: requirements.md
      - name: Design de arquitetura
        status: completed
        output: architecture.md

  execution:
    status: in_progress
    steps:
      - name: Implementar service
        status: completed
        output: conversation_filter_service.rb
      - name: Criar migration
        status: in_progress
      - name: Implementar componente Vue
        status: pending

decisions:
  - title: Usar Service Object pattern
    description: Service objects para encapsular lógica de filtros
    phase: planning
    alternatives:
      - Query objects
      - Scopes diretos no model
```

### Criar Plan

```bash
# Criar plan
mcp_ai-context_plan scaffoldPlan \
  --planName=advanced-filters \
  --title="Implementar Filtros Avançados" \
  --summary="Feature de filtros avançados em conversas" \
  --autoFill=true

# Link plan ao workflow
mcp_ai-context_plan link --planSlug=advanced-filters
```

### Atualizar Plan

```bash
# Atualizar status de fase
mcp_ai-context_plan updatePhase \
  --planSlug=advanced-filters \
  --phaseId=execution \
  --status=in_progress

# Atualizar step
mcp_ai-context_plan updateStep \
  --planSlug=advanced-filters \
  --phaseId=execution \
  --stepIndex=1 \
  --status=completed \
  --output=conversation_filter_service.rb \
  --notes="Service implementado com rate limiting"

# Registrar decisão
mcp_ai-context_plan recordDecision \
  --planSlug=advanced-filters \
  --title="Limitar número de filtros" \
  --description="Máximo de 10 filtros para prevenir queries complexas" \
  --phase=execution \
  --alternatives=["Sem limite", "Limite de 5", "Limite dinâmico"]
```

### Visualizar Plans

```bash
# Listar plans linkados
mcp_ai-context_plan getLinked

# Ver detalhes de um plan
mcp_ai-context_plan getDetails --planSlug=advanced-filters

# Ver plans de uma fase
mcp_ai-context_plan getForPhase --phase=E

# Ver status de execução
mcp_ai-context_plan getStatus --planSlug=advanced-filters
```

### Sincronizar Plan

```bash
# Atualizar markdown do tracking
mcp_ai-context_plan syncMarkdown --planSlug=advanced-filters

# Criar commit de fase completa
mcp_ai-context_plan commitPhase \
  --planSlug=advanced-filters \
  --phaseId=execution \
  --coAuthor="feature-developer" \
  --dryRun=false
```

---

## Skills dos Agents

### Backend Specialist

**Skills principais**:
- ✅ Ruby on Rails development
- ✅ API design (RESTful)
- ✅ Service objects & design patterns
- ✅ ActiveRecord & SQL
- ✅ Background jobs (Sidekiq)
- ✅ WebSockets (Action Cable)
- ✅ Authentication & authorization

**Ferramentas**:
- RSpec para testes
- RuboCop para linting
- Rails console para debugging
- Byebug para breakpoints

**Quando usar**:
- Implementar APIs
- Criar services/jobs
- Modificar models
- Trabalhar com banco de dados

---

### Frontend Specialist

**Skills principais**:
- ✅ Vue 3 (Composition API)
- ✅ Tailwind CSS
- ✅ Component design
- ✅ State management (Vuex)
- ✅ API integration
- ✅ Responsive design

**Ferramentas**:
- Vitest para testes
- ESLint para linting
- Vue DevTools para debugging
- Vite para bundling

**Quando usar**:
- Criar componentes Vue
- Implementar UI/UX
- State management
- Integração com APIs

---

### Test Writer

**Skills principais**:
- ✅ RSpec (unit, integration, system)
- ✅ Vitest (unit, component)
- ✅ FactoryBot para fixtures
- ✅ Test-driven development (TDD)
- ✅ Coverage analysis
- ✅ E2E testing

**Ferramentas**:
- RSpec para Ruby
- Vitest para JavaScript
- SimpleCov para coverage
- Faker para dados fake

**Quando usar**:
- Criar suites de testes
- Aumentar coverage
- TDD
- Validar comportamento

---

### Code Reviewer

**Skills principais**:
- ✅ Code quality assessment
- ✅ Design patterns review
- ✅ Best practices enforcement
- ✅ Refactoring suggestions
- ✅ DRY/SOLID principles
- ✅ Code smell detection

**Ferramentas**:
- RuboCop rules
- ESLint rules
- Git diff analysis
- Architecture docs

**Quando usar**:
- Pull request reviews
- Code quality checks
- Refactoring guidance
- Architecture validation

---

### Security Auditor

**Skills principais**:
- ✅ Vulnerability scanning
- ✅ Authentication review
- ✅ Authorization (Pundit)
- ✅ SQL injection prevention
- ✅ XSS prevention
- ✅ CSRF protection
- ✅ Mass assignment protection

**Ferramentas**:
- Brakeman (Rails security)
- Bundle audit (gems)
- npm audit (packages)
- Security best practices

**Quando usar**:
- Security audits
- Before production deploy
- Sensitive feature review
- Compliance checks

---

### Performance Optimizer

**Skills principais**:
- ✅ N+1 query detection
- ✅ Database optimization
- ✅ Caching strategies
- ✅ Background job optimization
- ✅ Memory profiling
- ✅ Query performance

**Ferramentas**:
- Bullet gem (N+1 detection)
- Rails profiler
- Database EXPLAIN
- Redis cache
- New Relic/Scout

**Quando usar**:
- Performance issues
- Slow endpoints
- High memory usage
- Database optimization

---

### Database Specialist

**Skills principais**:
- ✅ Schema design
- ✅ Migration creation
- ✅ Index optimization
- ✅ Query optimization
- ✅ Data modeling
- ✅ PostgreSQL features

**Ferramentas**:
- Rails migrations
- PostgreSQL EXPLAIN
- Database indexes
- ActiveRecord scopes

**Quando usar**:
- Create migrations
- Optimize queries
- Design schema
- Data modeling

---

### DevOps Specialist

**Skills principais**:
- ✅ Docker & containers
- ✅ CI/CD pipelines
- ✅ Deployment automation
- ✅ Environment management
- ✅ Monitoring & logging
- ✅ Infrastructure as Code

**Ferramentas**:
- Docker/docker-compose
- GitHub Actions
- Kubernetes
- Terraform
- Monitoring tools

**Quando usar**:
- Setup CI/CD
- Deploy features
- Infrastructure changes
- Environment setup

---

## Como Usar

### Workflow Típico

#### 1. Início (Planning)

```bash
# Iniciar workflow
workflow-init --name=my-feature

# Orquestrar agent de planejamento
agent orchestrate --task="Plan advanced filters" --phase=P

# Criar plan
plan scaffoldPlan --planName=my-feature --autoFill=true
```

**Agent sugerido**: `architect-specialist`

#### 2. Review

```bash
# Avançar para Review
workflow-advance

# Orquestrar agents de review
agent orchestrate --task="Review design" --phase=R
```

**Agents sugeridos**: `architect-specialist`, `security-auditor`, `code-reviewer`

#### 3. Execution

```bash
# Aprovar e avançar
workflow-manage approvePlan
workflow-advance

# Orquestrar agents de implementação
agent orchestrate --task="Implement feature" --phase=E
```

**Agents sugeridos**: `feature-developer`, `backend-specialist`, `frontend-specialist`

#### 4. Validation

```bash
# Avançar para validação
workflow-advance

# Orquestrar agents de teste
agent orchestrate --task="Test and validate" --phase=V
```

**Agents sugeridos**: `test-writer`, `performance-optimizer`, `security-auditor`

#### 5. Complete

```bash
# Avançar para finalização
workflow-advance

# Orquestrar agents de documentação
agent orchestrate --task="Document and deploy" --phase=C
```

**Agents sugeridos**: `documentation-writer`, `devops-specialist`

---

## Exemplos Práticos

### Exemplo 1: Nova Feature (Happy Path)

```bash
# 1. Planning
workflow-init --name=export-conversations
agent getDocs --agent=architect-specialist
plan scaffoldPlan --planName=export-conversations

# Agent architect-specialist cria:
# - Plan detalhado
# - Architecture decision
# - Task breakdown

# 2. Review
workflow-advance
agent orchestrate --phase=R --task="Review export architecture"

# Agents (architect + security) revisam:
# - Design patterns
# - Security concerns
# - Performance implications

# 3. Execution
workflow-manage approvePlan
workflow-advance
agent orchestrate --phase=E --task="Implement export"

# Agents (backend + frontend) implementam:
# - Service ExportConversationsService
# - Job ExportConversationsJob
# - API endpoint
# - UI component

# 4. Validation
workflow-advance
agent orchestrate --phase=V --task="Test export feature"

# Agents (test-writer + performance) criam:
# - RSpec tests
# - Performance tests
# - Load testing

# 5. Complete
workflow-advance
agent orchestrate --phase=C --task="Document and deploy"

# Agents (documentation + devops) finalizam:
# - Documentation updated
# - Changelog
# - Deploy checklist
```

---

### Exemplo 2: Bug Fix

```bash
# 1. Planning (rápido)
workflow-init --name=fix-attachment-upload
agent getDocs --agent=bug-fixer

# Agent bug-fixer:
# - Reproduz bug
# - Identifica causa raiz
# - Propõe solução

# 2. Review (opcional para bugs simples)
workflow-manage setAutonomous --enabled=true
workflow-advance

# 3. Execution
workflow-advance
agent orchestrate --phase=E --task="Fix attachment upload"

# Agent bug-fixer:
# - Implementa fix
# - Adiciona validação
# - Trata edge cases

# 4. Validation
workflow-advance
agent orchestrate --phase=V --task="Test bug fix"

# Agent test-writer:
# - Adiciona regression test
# - Valida fix

# 5. Complete
workflow-advance
# Deploy direto
```

---

### Exemplo 3: Refactoring

```bash
# 1. Planning
workflow-init --name=refactor-message-processing
agent getDocs --agent=refactoring-specialist

# Agent refactoring-specialist analisa:
# - Code smells
# - Duplicação
# - Complexidade

# 2. Review
workflow-advance
agent orchestrate --phase=R

# Agents (architect + code-reviewer) avaliam:
# - Impacto do refactoring
# - Breaking changes
# - Migration strategy

# 3. Execution
workflow-advance
agent orchestrate --phase=E

# Agent refactoring-specialist:
# - Extrai métodos
# - Elimina duplicação
# - Simplifica lógica

# 4. Validation
workflow-advance
agent orchestrate --phase=V

# Agent test-writer:
# - Garante testes passam
# - Adiciona testes faltantes
# - Valida behavior mantido

# 5. Complete
workflow-advance
# Documentation e deploy
```

---

## Best Practices

### ✅ Do's

1. **Use o workflow PREVC para features complexas**
   - Features > 100 linhas
   - Mudanças de arquitetura
   - Múltiplos componentes afetados

2. **Auto-organize agents por fase**
   - Planning → Architects
   - Review → Reviewers + Security
   - Execution → Developers + Specialists
   - Validation → Testers + Optimizers
   - Complete → Documentation + DevOps

3. **Documente decisões no plan**
   - Use `recordDecision` para ADRs
   - Registre alternativas consideradas
   - Justifique escolhas

4. **Use handoffs explícitos**
   - Transfira contexto entre agents
   - Liste artifacts produzidos
   - Documente estado atual

5. **Mantenha plans atualizados**
   - Update step status conforme avança
   - Sync markdown regularmente
   - Commit por fase completa

6. **Aproveite especialização**
   - Cada agent para sua área
   - Não force agents em tarefas fora do escopo
   - Colabore quando necessário

### ❌ Don'ts

1. **Não use PREVC para mudanças triviais**
   - Typos
   - One-liners
   - Mudanças óbvias
   - Use mode autônomo ou skip workflow

2. **Não pule fases sem justificativa**
   - Cada fase tem propósito
   - Gates existem por razão
   - Use `force` apenas quando necessário

3. **Não misture responsabilidades**
   - Backend specialist não faz UI
   - Test writer não faz deployment
   - Respeite especialização

4. **Não ignore gates**
   - Plan approval → previne retrabalho
   - Design review → previne problemas
   - Tests → previne regressões

5. **Não sobrecarregue plans**
   - Mantenha focused e conciso
   - Break large features em múltiplos plans
   - Um plan = uma feature/epic

6. **Não use agents sem contexto**
   - Garanta agent tem documentação
   - Passe artifacts relevantes
   - Explique objetivo claramente

---

## Orquestração Avançada

### Sequência de Agents

```bash
# Obter sequência recomendada de agents para uma task
agent getSequence \
  --task="Implement advanced filters" \
  --includeReview=true \
  --phases=["P", "R", "E", "V", "C"]

# Retorna:
# Phase P: architect-specialist → documentation-writer
# Phase R: architect-specialist → security-auditor → code-reviewer
# Phase E: feature-developer → backend-specialist → frontend-specialist
# Phase V: test-writer → performance-optimizer → code-reviewer
# Phase C: documentation-writer → devops-specialist
```

### Orquestração por Role

```bash
# Orquestrar agents por role
agent orchestrate --role=developer --task="Implement API"
agent orchestrate --role=qa --task="Test feature"
agent orchestrate --role=reviewer --task="Review code"
```

**Roles disponíveis**:
- `planner` - Planejamento
- `designer` - Design
- `architect` - Arquitetura
- `developer` - Desenvolvimento
- `qa` - Quality Assurance
- `reviewer` - Code review
- `documenter` - Documentação
- `solo-dev` - Full-stack (todas as fases)

---

## Gestão de Documentos de Workflow

### Criar Documentos Estruturados

```bash
# PRD (Product Requirements Document)
workflow-manage createDoc \
  --type=prd \
  --docName="advanced-filters-prd"

# Tech Spec
workflow-manage createDoc \
  --type=tech-spec \
  --docName="filters-technical-spec"

# Architecture Decision Record
workflow-manage createDoc \
  --type=adr \
  --docName="adr-001-service-pattern"

# Test Plan
workflow-manage createDoc \
  --type=test-plan \
  --docName="filters-test-plan"

# Changelog
workflow-manage createDoc \
  --type=changelog \
  --docName="v3.2.0-changelog"
```

### Gates de Workflow

```bash
# Verificar status de gates
workflow-manage getGates

# Retorna:
# {
#   "require_plan": true,      # P → R requer plan
#   "require_approval": true,  # R → E requer approval
#   "autonomous_mode": false   # Gates ativos
# }
```

---

## Troubleshooting

### Workflow Travado

```bash
# Ver status detalhado
workflow-status

# Forçar avanço se necessário
workflow-advance --force

# Ou ativar modo autônomo
workflow-manage setAutonomous --enabled=true
```

### Agent Não Encontrado

```bash
# Verificar agents disponíveis
agent discover

# Ver detalhes do agent
agent getInfo --agentType=AGENT_NAME

# Verificar documentation do agent
agent getDocs --agent=AGENT_NAME
```

### Plan Inconsistente

```bash
# Ver status do plan
plan getStatus --planSlug=my-plan

# Sincronizar com markdown
plan syncMarkdown --planSlug=my-plan

# Re-linkar se necessário
plan link --planSlug=my-plan
```

---

## Integração com Chatwoot

### Estrutura Sugerida

```
.context/
├── docs/              # Documentação (já existe)
├── plans/             # Plans de features
│   ├── advanced-filters.yaml
│   ├── export-conversations.yaml
│   └── audit-logs.yaml
├── workflow/          # Estado do workflow
│   ├── status.yaml
│   └── history/
└── decisions/         # ADRs
    ├── 001-service-pattern.md
    ├── 002-rate-limiting.md
    └── 003-caching-strategy.md
```

### Exemplo: Implementar Feature no Chatwoot

#### Feature: Sistema de Audit Logs

**1. Planning (architect-specialist)**

```bash
workflow-init --name=audit-logs
plan scaffoldPlan --planName=audit-logs --title="Sistema de Audit Logs"
```

Plan criado:
```yaml
name: audit-logs
title: Sistema de Audit Logs
summary: Registrar todas as ações importantes para auditoria

phases:
  planning:
    steps:
      - name: Definir eventos a auditar
        status: pending
      - name: Design de schema
        status: pending
      - name: Design de API
        status: pending
```

**2. Review (security-auditor)**

```bash
workflow-advance
agent getDocs --agent=security-auditor
```

Security considerations:
- PII data handling
- Retention policies
- Access controls
- GDPR compliance

**3. Execution (backend-specialist + database-specialist)**

```bash
workflow-manage approvePlan
workflow-advance
```

Implementation:
- Model `AuditLog`
- Service `AuditLogService`
- Observer pattern para eventos
- API endpoints

**4. Validation (test-writer + performance-optimizer)**

```bash
workflow-advance
```

Tests:
- RSpec para service
- Performance test (bulk inserts)
- API integration tests

**5. Complete (documentation-writer)**

```bash
workflow-advance
```

Documentation:
- Update API.md
- Add to MODELS.md
- Security guidelines

---

## Recursos Adicionais

### Comandos Quick Reference

```bash
# Agents
agent discover                              # Listar todos
agent getInfo --agentType=TYPE             # Detalhes
agent  getDocs --agent=TYPE                 # Documentação
agent orchestrate --task=TASK --phase=P    # Orquestrar

# Workflow
workflow-init --name=NAME                   # Iniciar
workflow-status                             # Status
workflow-advance                            # Avançar fase
workflow-advance --force                    # Forçar
workflow-manage setAutonomous --enabled=T   # Modo autônomo

# Plans
plan scaffoldPlan --planName=NAME          # Criar
plan link --planSlug=SLUG                  # Linkar
plan getDetails --planSlug=SLUG            # Ver
plan updateStep --planSlug=SLUG ...        # Atualizar
plan recordDecision --planSlug=SLUG ...    # Decisão

# Documents
workflow-manage createDoc --type=prd       # Criar doc
workflow-manage getGates                   # Ver gates
```

### PREVC Cheat Sheet

| Fase | Foco | Agents | Output |
|------|------|--------|--------|
| **P** | Planejar | architect, documentation-writer | Plan, ADRs |
| **R** | Revisar | architect, security-auditor, code-reviewer | Design approval |
| **E** | Executar | feature-developer, specialists | Code, migrations |
| **V** | Validar | test-writer, performance-optimizer | Tests, benchmarks |
| **C** | Completar | documentation-writer, devops-specialist | Docs, deployment |

### Quando Usar Cada Agent

| Situação | Agent | Por quê |
|----------|-------|---------|
| Nova feature | feature-developer | Conhece patterns gerais |
| Bug urgente | bug-fixer | Foco em fix rápido |
| API nova | backend-specialist | Expert em APIs |
| UI nova | frontend-specialist | Expert em Vue |
| Performance ruim | performance-optimizer | Profiling e optimization |
| Security issue | security-auditor | Security expertise |
| Código confuso | refactoring-specialist | Clean code |
| Sem testes | test-writer | Test expertise |
| Deploy | devops-specialist | Infrastructure |
| Docs | documentation-writer | Writing skills |

---

## Conclusão

O sistema **AI-Context Agents & Workflow PREVC** oferece:

✅ **Estrutura** - Processo claro e repetível  
✅ **Qualidade** - Gates de qualidade em cada fase  
✅ **Especialização** - Agents especializados para cada tarefa  
✅ **Rastreabilidade** - Plans e decisões documentadas  
✅ **Colaboração** - Handoffs e colaboração entre agents  
✅ **Flexibilidade** - Mode autônomo para tarefas simples  

Use PREVC para features complexas e aproveite a especialização dos agents para entregar software de qualidade de forma estruturada e previsível.

---

**Próximos passos**:
1. Inicialize um workflow com `workflow-init`
2. Explore agents com `agent discover`
3. Crie seu primeiro plan com `plan scaffoldPlan`
4. Siga o workflow PREVC para sua próxima feature! 🚀

---

*Documentação AI-Context versão 1.0 - 02/02/2026*
