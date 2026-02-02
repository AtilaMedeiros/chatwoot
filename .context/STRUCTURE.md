# 📁 Estrutura Completa do .context/

## Visão Geral

```
.context/
├── docs/                           # 📚 Documentação do projeto
│   ├── README.md                   # Índice de navegação
│   ├── ARQUITETURA.md             # Arquitetura completa
│   ├── MODELS.md                  # Models e banco de dados
│   ├── API.md                     # APIs REST
│   ├── DEVELOPMENT.md             # Guia de desenvolvimento
│   ├── CHEATSHEET.md              # Referência rápida
│   └── AI_CONTEXT_WORKFLOW.md     # Agents & Workflow PREVC
│
├── agents/                         # 🤖 Definições de agents
│   ├── README.md                   # Guia de agents
│   ├── manifest.yaml              # Configuração central
│   ├── chatwoot-specialist.yaml   # Agent domain expert
│   ├── conversation-specialist.yaml  # Agent feature expert
│   └── channel-specialist.yaml    # Agent integration expert
│
├── skills/                         # 🎯 Skills (separado de agents)
│   ├── README.md                   # Guia de skills
│   ├── ruby-rails.yaml            # Skill backend
│   ├── vue-frontend.yaml          # Skill frontend
│   ├── api-development.yaml       # Skill APIs
│   └── testing.yaml               # Skill QA
│
├── plans/                          # 📋 Plans de features/tasks
│   ├── example-plan.yaml          # Template/exemplo completo
│   ├── advanced-filters.yaml      # Plan: Filtros avançados
│   └── audit-logs.yaml            # Plan: Audit logs
│
├── decisions/                      # 🏛️ Architecture Decision Records
│   ├── README.md                   # Guia de ADRs
│   ├── template.md                # Template para novos ADRs
│   ├── 001-service-object-pattern.md    # ADR: Services
│   ├── 002-rate-limiting-strategy.md    # ADR: Rate limiting
│   └── 003-caching-strategy.md          # ADR: Caching
│
└── workflow/                       # 🔄 Estado do workflow PREVC
    ├── status.yaml                # Status atual do workflow
    └── history/                   # Histórico de execuções
        ├── 2026-02-01-feature-x.yaml
        └── 2026-02-02-bugfix-y.yaml
```

## Sumário dos Componentes

### 📚 docs/ - Documentação

7 documentos completos cobrindo toda a arquitetura, APIs, models, desenvolvimento e workflow.

**Uso**: Consulta para entender o projeto, desenvolver features, integrar APIs.

### 🤖 agents/ - Agents Especializados

**Built-in** (14 agents do AI-Context):
- code-reviewer, bug-fixer, feature-developer
- refactoring-specialist, test-writer, documentation-writer
- performance-optimizer, security-auditor
- backend/frontend/architect/devops/database/mobile specialists

**Custom** (3 agents para Chatwoot):
- chatwoot-specialist (domain expert)
- conversation-specialist (feature expert)
- channel-specialist (integration expert)

**Uso**: Orquestrar agents especializados em cada fase do workflow PREVC.

### 🎯 skills/ - Habilidades

Skills definem capacidades que agents possuem:
- **ruby-rails**: Backend development
- **vue-frontend**: Frontend development
- **testing**: QA & automated tests
- **api-development**: REST APIs
- **database-design**: Schema & queries
- **performance**: Optimization
- **security**: Security audit
- **devops**: CI/CD & deployment

Cada skill tem:
- Proficiency levels (beginner → advanced → expert)
- Tools disponíveis
- Patterns de código
- Best practices
- Commands úteis

**Uso**: Consultar capabilities, tools, patterns e best practices por área.

### 📋 plans/ - Planejamento de Features

Plans são documentos YAML estruturados para gerenciar features/tasks:

**Contém**:
- Summary & goals
- PREVC phases (P→R→E→V→C)
- Steps de cada fase
- Status tracking
- Decisions (ADRs inline)
- Technical specs
- Testing strategy
- Rollout plan

**Uso**: 
```bash
# Criar plan
plan scaffoldPlan --planName=my-feature

# Atualizar step
plan updateStep --planSlug=my-feature --stepIndex=1 --status=completed

# Registrar decisão
plan recordDecision --planSlug=my-feature --title="Decision"
```

### 🏛️ decisions/ - Architecture Decision Records

ADRs documentam decisões arquiteturais importantes:

**Estrutura**:
- Contexto (por quê?)
- Decisão (o quê?)
- Alternativas consideradas
- Consequências (impactos)
- Status (proposta/aceita/rejeitada/substituída)

**Uso**: Documentar e revisar decisões arquiteturais significativas.

### 🔄 workflow/ - Estado do Workflow

Tracking do workflow PREVC:
- Fase atual (P/R/E/V/C)
- Plans linkados
- Gates (require_plan, require_approval)
- Modo autônomo
- Histórico de execuções

**Uso**: Gerenciado automaticamente pelo sistema AI-Context.

## Como Usar a Estrutura

### 1. Consulta Rápida (Desenvolvimento Diário)

```bash
# Referência rápida
vim .context/docs/CHEATSHEET.md

# Consultar agent/skill
vim .context/agents/chatwoot-specialist.yaml
vim .context/skills/ruby-rails.yaml
```

### 2. Feature Complexa (Workflow Completo)

```bash
# 1. Iniciar workflow
workflow-init --name=my-feature

# 2. Planning (P)
agent orchestrate --phase=P --role=architect
plan scaffoldPlan --planName=my-feature

# 3. Review (R)
agent orchestrate --phase=R --role=reviewer
workflow-manage createDoc --type=adr --docName="adr-004-decision"

# 4. Execution (E)
workflow-manage approvePlan
workflow-advance
agent orchestrate --phase=E --role=developer

# 5. Validation (V)
workflow-advance
agent orchestrate --phase=V --role=qa

# 6. Complete (C)
workflow-advance
agent orchestrate --phase=C --role=documenter
```

### 3. Consultar Documentação

```bash
# Arquitetura geral
.context/docs/ARQUITETURA.md

# Models e banco
.context/docs/MODELS.md

# APIs
.context/docs/API.md

# Development workflow
.context/docs/DEVELOPMENT.md

# Agents e workflow
.context/docs/AI_CONTEXT_WORKFLOW.md
```

### 4. Decisões Arquiteturais

```bash
# Revisar ADRs existentes
ls .context/decisions/*.md

# Ver decisão específica
vim .context/decisions/001-service-object-pattern.md

# Criar nova decisão
cp .context/decisions/template.md .context/decisions/004-new-decision.md
```

## Integração com Desenvolvimento

### IDE/Editor

Adicione aos snippets:

```json
{
  "Check Agent Skills": {
    "prefix": "agent-skills",
    "body": "vim .context/skills/${1:ruby-rails}.yaml"
  },
  "Check Cheatsheet": {
    "prefix": "cheat",
    "body": "vim .context/docs/CHEATSHEET.md"
  }
}
```

### Git

Adicione ao `.gitignore` se necessário:

```gitignore
.context/workflow/status.yaml  # Se for estado local
.context/workflow/history/     # Se for estado local
```

**Recomendação**: Commitar agents, skills, plans, decisions (são parte do projeto).

### VS Code

Adicione ao `settings.json`:

```json
{
  "files.associations": {
    ".context/**/*.yaml": "yaml",
    ".context/**/*.md": "markdown"
  },
  "search.exclude": {
    ".context/workflow/history/**": true
  }
}
```

## Manutenção

### Atualizar Documentação

```bash
# Atualizar docs quando código mudar
vim .context/docs/ARQUITETURA.md
vim .context/docs/MODELS.md
vim .context/docs/API.md
```

### Atualizar Agents/Skills

```bash
# Quando novos patterns surgem
vim .context/skills/ruby-rails.yaml

# Quando novo agent é necessário
cp .context/agents/chatwoot-specialist.yaml .context/agents/new-specialist.yaml
```

### Limpar Workflow History

```bash
# Arquivar workflows antigos (opcional)
mv .context/workflow/history/2025-*.yaml .context/workflow/archive/
```

## Referências Rápidas

| Preciso de... | Vou em... |
|---------------|-----------|
| Comando rápido | `.context/docs/CHEATSHEET.md` |
| Entender arquitetura | `.context/docs/ARQUITETURA.md` |
| Ver API endpoint | `.context/docs/API.md` |
| Consultar model | `.context/docs/MODELS.md` |
| Setup ambiente | `.context/docs/DEVELOPMENT.md` |
| Usar agents | `.context/docs/AI_CONTEXT_WORKFLOW.md` |
| Skills de agent | `.context/skills/` |
| Ver decisões | `.context/decisions/` |
| Plan de feature | `.context/plans/example-plan.yaml` |

---

**Estrutura completa criada**: 02/02/2026  
**Total**: 7 docs + 3 agents + 8 skills + examples + ADRs
