# 🚀 Getting Started com AI-Context

Guia rápido para começar a usar o sistema completo de documentação, agents e workflow PREVC.

## 📚 O que você tem agora?

Estrutura completa do AI-Context criada:

```
✅ 7 documentos de referência (docs/)
✅ 14 agents built-in + 3 custom (agents/)
✅ 8 skills detalhados (skills/)
✅ Sistema de plans (plans/)
✅ Architecture Decision Records (decisions/)
✅ Workflow PREVC configurado
```

## 🎯 Primeiros Passos

### 1. Explore a Documentação (5 min)

```bash
# Índice de tudo
cat .context/docs/README.md

# Arquitetura do Chatwoot
cat .context/docs/ARQUITETURA.md

# Comandos e patterns prontos (⭐ favorito!)
cat .context/docs/CHEATSHEET.md
```

### 2. Conheça os Agents (3 min)

```bash
# Ver todos os agents disponíveis
cat .context/agents/README.md

# Agent especialista em Chatwoot
cat .context/agents/chatwoot-specialist.yaml

# Skills de Ruby on Rails
cat .context/skills/ruby-rails.yaml
```

### 3. Entenda o Workflow PREVC (10 min)

```bash
# Guia completo do workflow
cat .context/docs/AI_CONTEXT_WORKFLOW.md

# Estrutura de um plan
cat .context/plans/example-plan.yaml

# Exemplo de ADR
cat .context/decisions/001-service-object-pattern.md
```

## 💡 Use Cases Comuns

### Use Case 1: "Preciso implementar uma feature rápida"

**Sem workflow complexo** (feature simples):

```bash
# 1. Consulte o cheatsheet
vim .context/docs/CHEATSHEET.md

# 2. Veja patterns similares em ARQUITETURA.md
vim .context/docs/ARQUITETURA.md

# 3. Implemente seguindo os patterns
# 4. Teste
bundle exec rspec
pnpm test

# 5. Commit
git commit -m "feat(conversations): add simple filter"
```

---

### Use Case 2: "Feature complexa, preciso de estrutura"

**Com workflow PREVC completo**:

```bash
# === FASE P: PLANNING ===
# 1. Iniciar workflow
workflow-init --name=advanced-filters

# 2. Criar plan
plan scaffoldPlan \
  --planName=advanced-filters \
  --title="Advanced Conversation Filters" \
  --autoFill=true

# 3. Agent arquiteto planeja
agent getDocs --agent=architect-specialist

# 4. Documentar arquitetura
# (Editar .context/plans/advanced-filters.yaml)

# === FASE R: REVIEW ===
# 5. Avançar para review
workflow-advance

# 6. Agents revisam
agent orchestrate --phase=R --task="Review design"

# 7. Criar ADR se decisão importante
workflow-manage createDoc --type=adr --docName="adr-005-filter-strategy"

# 8. Aprovar plan
workflow-manage approvePlan --planSlug=advanced-filters

# === FASE E: EXECUTION ===
# 9. Avançar para execution
workflow-advance

# 10. Implementar seguindo plan
# - Backend: service + controller
# - Frontend: component
# - Tests: RSpec + Vitest

# 11. Atualizar status de steps
plan updateStep \
  --planSlug=advanced-filters \
  --phaseId=execution \
  --stepIndex=1 \
  --status=completed

# === FASE V: VALIDATION ===
# 12. Avançar para validation
workflow-advance

# 13. Rodar testes
bundle exec rspec
pnpm test

# === FASE C: COMPLETE ===
# 14. Avançar para complete
workflow-advance

# 15. Atualizar documentação
vim .context/docs/API.md

# 16. Commit e PR
git commit -m "feat(filters): implement advanced conversation filters"
```

---

### Use Case 3: "Preciso consultar como fazer X"

**Consulta rápida de patterns**:

```bash
# Backend patterns (Services, Jobs, Models)
vim .context/skills/ruby-rails.yaml

# Frontend patterns (Vue, Composables, Vuex)
vim .context/skills/vue-frontend.yaml

# Testing patterns (RSpec, Vitest)
vim .context/skills/testing.yaml

# Comando específico
grep -r "factory" .context/docs/CHEATSHEET.md
```

---

### Use Case 4: "Qual agent devo usar?"

**Escolher agent por tarefa**:

```bash
# Ver mapping de tasks → agents
cat .context/agents/manifest.yaml | grep -A 20 "taskMapping:"

# Ver agent específico
vim .context/agents/conversation-specialist.yaml
```

**Quick Reference**:

| Tarefa | Agent(s) |
|--------|----------|
| Implementar feature | feature-developer, backend-specialist |
| Review de código | code-reviewer |
| Corrigir bug | bug-fixer |
| Escrever testes | test-writer |
| Otimizar performance | performance-optimizer |
| Audit segurança | security-auditor |
| Deploy | devops-specialist |
| Documentação | documentation-writer |
| Conversas | conversation-specialist |
| Canais | channel-specialist |

---

### Use Case 5: "Tomar decisão arquitetural"

**Documentar decisão importante**:

```bash
# 1. Copiar template
cp .context/decisions/template.md .context/decisions/005-my-decision.md

# 2. Preencher ADR
vim .context/decisions/005-my-decision.md

# 3. Discutir com time
# 4. Marcar como "Aceito"
# 5. Commit
git add .context/decisions/005-my-decision.md
git commit -m "docs: add ADR-005 for my decision"
```

---

## 🎓 Aprendizado Progressivo

### Nível 1: Iniciante (Semana 1)

📖 **Foco**: Documentação e comandos básicos

```bash
✅ Ler README.md
✅ Consultar CHEATSHEET.md diariamente
✅ Entender ARQUITETURA.md
✅ Conhecer MODELS.md básico
```

**Resultado**: Consegue desenvolver seguindo patterns existentes.

---

### Nível 2: Intermediário (Semana 2-3)

🤖 **Foco**: Agents e Skills

```bash
✅ Entender agents disponíveis
✅ Consultar skills por área
✅ Usar plans para features médias
✅ Registrar decisões em plans
```

**Resultado**: Consegue estruturar features complexas.

---

### Nível 3: Avançado (Mês 1+)

🔄 **Foco**: Workflow PREVC completo

```bash
✅ Usar workflow PREVC para features grandes
✅ Orquestrar múltiplos agents
✅ Criar ADRs para decisões importantes
✅ Handoffs entre agents
✅ Modo autônomo quando apropriado
```

**Resultado**: Master do sistema AI-Context!

---

## 🛠️ Setup do Ambiente

### VS Code (Recomendado)

**Extensões úteis**:
```json
{
  "recommendations": [
    "redhat.vscode-yaml",           // YAML syntax
    "yzhang.markdown-all-in-one",   // Markdown
    "github.copilot"                // AI assistance
  ]
}
```

**Snippets** (`.vscode/chatwoot-context.code-snippets`):

```json
{
  "AI Context Cheatsheet": {
    "prefix": "cheat",
    "body": [
      "# Quick reference:",
      "vim .context/docs/CHEATSHEET.md"
    ]
  },
  "AI Context Agent": {
    "prefix": "agent",
    "body": [
      "vim .context/agents/${1:chatwoot-specialist}.yaml"
    ]
  },
  "AI Context Skill": {
    "prefix": "skill",
    "body": [
      "vim .context/skills/${1:ruby-rails}.yaml"
    ]
  }
}
```

**Keybindings** (`.vscode/keybindings.json`):

```json
[
  {
    "key": "ctrl+shift+c",
    "command": "workbench.action.terminal.sendSequence",
    "args": { "text": "vim .context/docs/CHEATSHEET.md\n" }
  }
]
```

---

### Shell Aliases

Adicione ao seu `.bashrc` ou `.zshrc`:

```bash
# AI-Context aliases
alias ctx-cheat='vim .context/docs/CHEATSHEET.md'
alias ctx-arch='vim .context/docs/ARQUITETURA.md'
alias ctx-models='vim .context/docs/MODELS.md'
alias ctx-api='vim .context/docs/API.md'
alias ctx-agents='cat .context/agents/README.md'
alias ctx-plan='vim .context/plans/'
alias ctx-adr='vim .context/decisions/'

# Workflow aliases
alias ctx-init='workflow-init --name='
alias ctx-status='workflow-status'
alias ctx-advance='workflow-advance'

# Plan aliases
alias ctx-plan-new='plan scaffoldPlan --planName='
alias ctx-plan-show='plan getDetails --planSlug='
```

---

## 📌 Dicas Pro

### 1. Mantenha CHEATSHEET.md Sempre Aberto

É sua referência mais rápida para comandos, patterns e snippets.

```bash
# Em um terminal/pane separado:
watch -n 300 cat .context/docs/CHEATSHEET.md  # Refresh a cada 5min
```

### 2. Use grep para Busca Rápida

```bash
# Buscar pattern em toda documentação
grep -r "service object" .context/docs/

# Buscar comando específico
grep -r "bundle exec rspec" .context/

# Buscar agent capabilities
grep -r "capabilities:" .context/agents/
```

### 3. Customize Seu Fluxo

Cada pessoa/time pode adaptar o workflow:

- **Solo dev**: Mode autônomo, menos gates
- **Team lead**: Workflow completo, ADRs rigorosos
- **Pair programming**: Plans leves, foco em execution

### 4. Evolua Incrementalmente

Não precisa usar tudo de uma vez:

**Dia 1-7**: Só documentação  
**Dia 8-14**: + Agents/skills  
**Dia 15-30**: + Plans simples  
**Mês 2+**: Workflow PREVC completo  

---

## 🆘 Troubleshooting

### "Muita informação, por onde começar?"

👉 Comece pelo **CHEATSHEET.md** - tem só o essencial.

### "Não sei qual agent usar"

👉 Veja **agents/manifest.yaml** seção `taskMapping`.

### "Workflow muito complexo para tarefa simples"

👉 Use **mode autônomo** ou **skip workflow** completamente.

```bash
workflow-manage setAutonomous --enabled=true --reason="Simple change"
```

### "Preciso de pattern específico rapidamente"

👉 Use grep:

```bash
grep -r "pattern-name" .context/skills/
```

---

## 🎉 Próximos Passos

Agora você tem tudo configurado! 

**Sugestão de cronograma**:

**Hoje**:
1. ✅ Leia este arquivo (você está aqui!)
2. ✅ Explore CHEATSHEET.md
3. ✅ Veja um exemplo de agent (chatwoot-specialist.yaml)

**Esta semana**:
1. Use CHEATSHEET.md no seu desenvolvimento
2. Consulte MODELS.md e API.md quando necessário
3. Veja skills quando implementar algo novo

**Próxima semana**:
1. Experimente criar um plan para uma feature
2. Tente usar um agent especializado
3. Documente uma decisão (ADR)

**Mês 1**:
1. Use workflow PREVC para feature complexa
2. Orquestre múltiplos agents
3. Domine o sistema! 🚀

---

## 📚 Recursos

| Recurso | Onde Encontrar | Quando Usar |
|---------|----------------|-------------|
| **Quick Reference** | CHEATSHEET.md | Todo dia |
| **Architecture** | ARQUITETURA.md | Entender sistema |
| **Models** | MODELS.md | Trabalhar com DB |
| **APIs** | API.md | Integrations |
| **Development** | DEVELOPMENT.md | Setup, workflows |
| **Agents Guide** | AI_CONTEXT_WORKFLOW.md | Features complexas |
| **Plans** | plans/ | Tracking features |
| **Decisions** | decisions/ | ADRs importantes |

---

## ❓ Perguntas Frequentes

**P: Preciso usar workflow PREVC para tudo?**  
R: Não! Use para features complexas. Para mudanças simples, apenas consulte docs e implemente.

**P: Posso criar meus próprios agents?**  
R: Sim! Copie um existente como template e customize.

**P: Como mantenho documentação atualizada?**  
R: Atualize docs/ quando mudar código. Faça parte do seu PR checklist.

**P: ADRs são obrigatórios?**  
R: Apenas para decisões arquiteturais significativas. Use bom senso.

**P: Qual a diferença entre plan e ADR?**  
R: Plan = tracking de feature (tasks, status). ADR = documentar decisão importante.

---

**🎊 Parabéns! Você está pronto para usar o AI-Context!**

**Need help?** Consulte qualquer arquivo em `.context/` - toda informação está lá!

Happy coding! 🚀

---

*Getting Started Guide - v1.0 - 02/02/2026*
