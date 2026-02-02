# Architecture Decision Records (ADRs)

Este diretório contém registros de decisões arquiteturais importantes tomadas durante o desenvolvimento do Chatwoot.

## O que é um ADR?

Um **Architecture Decision Record (ADR)** é um documento que captura uma decisão arquitetural importante, incluindo:

- **Contexto**: Por que a decisão foi necessária
- **Decisão**: O que foi decidido
- **Alternativas**: Opções consideradas
- **Consequências**: Impactos da decisão
- **Status**: Proposta, aceita, rejeitada, substituída

## Estrutura

```
.context/decisions/
├── README.md                      # Este arquivo
├── 001-service-object-pattern.md  # Exemplo: Pattern de services
├── 002-rate-limiting-strategy.md  # Exemplo: Rate limiting
├── 003-caching-strategy.md        # Exemplo: Caching
└── template.md                    # Template para novos ADRs
```

## Quando Criar um ADR?

Crie um ADR quando:

✅ A decisão impacta a arquitetura geral  
✅ É difícil de reverter  
✅ Afeta múltiplos times ou componentes  
✅ Estabelece um padrão a ser seguido  
✅ Há trade-offs significativos  
✅ A decisão será questionada no futuro  

❌ **Não** crie ADR para:
- Decisões triviais
- Mudanças temporárias
- Detalhes de implementação local

## Como Criar um ADR

1. Copie `template.md`
2. Numere sequencialmente (001, 002, etc)
3. Use nome descritivo no arquivo
4. Preencha todas as seções
5. Discuta com o time
6. Marque como "Aceito" quando aprovado

## Template Rápido

```markdown
# ADR-XXX: [Título Curto]

**Status**: Proposta | Aceita | Rejeitada | Substituída por ADR-YYY  
**Data**: YYYY-MM-DD  
**Autores**: Nome(s)  
**Reviewers**: Nome(s)

## Contexto

[Por que esta decisão é necessária?]

## Decisão

[O que decidimos fazer?]

## Alternativas Consideradas

### Opção 1: [Nome]
**Prós**:
- ...

**Contras**:
- ...

### Opção 2: [Nome]
...

## Decisão Escolhida

[Opção X] porque [razões]

## Consequências

**Positivas**:
- ...

**Negativas**:
- ...

**Riscos**:
- ...

## Notas
```

## ADRs Existentes

| # | Título | Status | Data |
|---|--------|--------|------|
| 001 | Service Object Pattern | Aceito | 2026-02-02 |
| 002 | Rate Limiting Strategy | Aceito | 2026-02-02 |
| 003 | Caching Strategy | Proposta | 2026-02-02 |

## Integração com Workflow

ADRs são criados durante as fases **Planning** e **Review** do workflow PREVC:

```bash
# Durante o planning
plan recordDecision \
  --planSlug=my-feature \
  --title="Use Service Object Pattern" \
  --description="Encapsulate complex logic in services" \
  --phase=planning \
  --alternatives=["Model methods", "Concerns", "Query objects"]

# Cria ADR automaticamente
workflow-manage createDoc \
  --type=adr \
  --docName="adr-004-service-pattern"
```

## Best Practices

✅ **DO**:
- Numere sequencialmente
- Seja conciso mas completo
- Liste todas as alternativas consideradas
- Documente consequências (boas e ruins)
- Atualize status quando necessário
- Referencie ADRs relacionados

❌ **DON'T**:
- Não edite ADRs aceitos (crie novo suapersedendo)
- Não seja vago nas razões
- Não esqueça consequências negativas
- Não crie ADR sem discussão

## Ciclo de Vida

```
Proposta → [Discussão] → Aceita ──────┐
                ↓                       ↓
            Rejeitada          [Tempo] Substituída
                                        ↓
                                  Nova Proposta
```

## Referências

- [ADR Guidelines](https://adr.github.io/)
- [AI_CONTEXT_WORKFLOW.md](../docs/AI_CONTEXT_WORKFLOW.md)
- [ARQUITETURA.md](../docs/ARQUITETURA.md)
