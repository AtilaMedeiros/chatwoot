# Chatwoot Agents

Este diretório contém configurações e definições de agents especializados para desenvolvimento no Chatwoot.

## Estrutura

```
.context/agents/
├── README.md                    # Este arquivo
├── manifest.yaml                # Configuração de todos os agents
├── chatwoot-specialist.yaml     # Agent especialista em Chatwoot
├── conversation-specialist.yaml # Agent especialista em conversas
└── channel-specialist.yaml      # Agent especialista em canais
```

**Nota**: Skills estão em `.context/skills/` (diretório separado) - veja [../skills/README.md](../skills/README.md)

## Agents Built-in do AI-Context

O AI-Context já inclui 14 agents especializados:

| Agent | Descrição |
|-------|-----------|
| code-reviewer | Review de código |
| bug-fixer | Correção de bugs |
| feature-developer | Desenvolvimento de features |
| refactoring-specialist | Refactoring |
| test-writer | Criação de testes |
| documentation-writer | Documentação |
| performance-optimizer | Otimização |
| security-auditor | Segurança |
| backend-specialist | Backend |
| frontend-specialist | Frontend |
| architect-specialist | Arquitetura |
| devops-specialist | DevOps |
| database-specialist | Database |
| mobile-specialist | Mobile |

## Agents Customizados para Chatwoot

Além dos built-in, criamos agents especializados no domínio do Chatwoot:

### 1. Chatwoot Specialist
Expert geral em toda a arquitetura do Chatwoot
- Models, Controllers, Services
- Channel integrations
- WebSockets e real-time
- Background jobs

### 2. Conversation Specialist
Foco no core do sistema - conversas
- Model Conversation
- Message handling
- Status workflows
- Assignment logic

### 3. Channel Specialist
Especialista em integrações de canais
- WhatsApp, Facebook, Email
- Webhooks
- API Channel
- Twilio SMS

## Como Usar Agents

### Via CLI (MCP ai-context)

```bash
# Descobrir agents disponíveis
mcp_ai-context_agent discover

# Ver informações de um agent
mcp_ai-context_agent getInfo --agentType=chatwoot-specialist

# Orquestrar agent para uma tarefa
mcp_ai-context_agent orchestrate \
  --task="Implementar filtros avançados em conversas" \
  --phase=E

# Ver documentação de um agent
mcp_ai-context_agent getDocs --agent=conversation-specialist
```

### Via Workflow PREVC

```bash
# Fase Planning (P)
agent orchestrate --phase=P --role=architect

# Fase Review (R)
agent orchestrate --phase=R --role=reviewer

# Fase Execution (E)
agent orchestrate --phase=E --role=developer

# Fase Validation (V)
agent orchestrate --phase=V --role=qa

# Fase Complete (C)
agent orchestrate --phase=C --role=documenter
```

## Skills

Skills são capacidades específicas que agents podem ter. Ver `skills/` para detalhes.

Cada skill define:
- **Tools**: Ferramentas disponíveis (RSpec, ESLint, etc)
- **Patterns**: Padrões de código conhecidos
- **Best Practices**: Práticas recomendadas
- **Context**: Documentação relevante

## Criando Agents Customizados

Para criar um agent customizado para seu contexto:

1. Criar arquivo YAML em `.context/agents/`
2. Definir metadata (nome, descrição, role)
3. Associar skills relevantes
4. Definir documentação primária
5. Registrar no `manifest.yaml`

Ver exemplos nos arquivos `.yaml` deste diretório.

## Handoffs entre Agents

Agents podem transferir trabalho entre si:

```bash
# Backend → Frontend
workflow-manage handoff \
  --from=backend-specialist \
  --to=frontend-specialist \
  --artifacts=["API endpoint criado", "ConversationController#index"]

# Developer → Test Writer
workflow-manage handoff \
  --from=feature-developer \
  --to=test-writer \
  --artifacts=["ConversationFilterService implementado"]
```

## Colaboração entre Agents

Múltiplos agents podem colaborar:

```bash
# Sessão de design review
workflow-manage collaborate \
  --topic="Review arquitetura de filtros" \
  --participants=["architect-specialist", "security-auditor", "performance-optimizer"]
```

## Best Practices

✅ **DO**:
- Use agents especializados para sua área
- Passe artifacts e contexto nos handoffs
- Documente decisões com `recordDecision`
- Siga o workflow PREVC para features complexas

❌ **DON'T**:
- Não force agents em tarefas fora do escopo
- Não pule fases sem justificativa
- Não misture responsabilidades
- Não use agents sem contexto adequado

## Referências

- [AI_CONTEXT_WORKFLOW.md](../docs/AI_CONTEXT_WORKFLOW.md) - Workflow PREVC completo
- [manifest.yaml](./manifest.yaml) - Configuração de agents
- [skills/](./skills/) - Skills disponíveis
