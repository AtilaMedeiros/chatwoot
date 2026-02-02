# Skills (Habilidades)

Este diretório contém definições de skills que agents podem possuir.

## O que são Skills?

**Skills** são capacidades específicas que agents têm. Cada skill define:

- **Tools**: Ferramentas disponíveis (RSpec, ESLint, etc)
- **Patterns**: Padrões de código conhecidos
- **Best Practices**: Práticas recomendadas
- **Context**: Documentação relevante
- **Commands**: Comandos úteis

## Skills Disponíveis

| Skill | Descrição | Proficiency Levels |
|-------|-----------|-------------------|
| **ruby-rails** | Desenvolvimento Ruby on Rails | beginner → advanced → expert |
| **vue-frontend** | Desenvolvimento Vue.js | beginner → advanced → expert |
| **api-development** | Desenvolvimento de APIs REST | beginner → advanced → expert |
| **testing** | Testes automatizados (RSpec, Vitest) | beginner → advanced → expert |
| **database-design** | Design e otimização de banco de dados | beginner → advanced → expert |
| **performance** | Otimização de performance | beginner → advanced → expert |
| **security** | Segurança e auditoria | beginner → advanced → expert |
| **devops** | DevOps, CI/CD, Deploy | beginner → advanced → expert |

## Estrutura de uma Skill

```yaml
name: skill-name
version: "1.0"
category: backend | frontend | fullstack | infrastructure

proficiencyLevels:
  beginner:
    description: "Basic understanding"
    capabilities: [...]
  
  advanced:
    description: "Proficient"
    capabilities: [...]
  
  expert:
    description: "Deep expertise"
    capabilities: [...]

tools:
  - name: tool-name
    purpose: "..."
    commands: [...]

patterns:
  - name: pattern-name
    description: "..."
    example: |
      code example

bestPractices:
  - "Practice 1"
  - "Practice 2"

documentation:
  - path: "../docs/FILE.md"
    sections: [...]
```

## Como Skills são Usadas

### 1. Agents têm Skills

Agents declaram suas skills com proficiency levels:

```yaml
# chatwoot-specialist.yaml
skills:
  - id: ruby-rails
    proficiency: expert
  
  - id: vue-frontend
    proficiency: expert
  
  - id: testing
    proficiency: advanced
```

### 2. Proficiency Determina Capabilities

- **Beginner**: Conhecimentos básicos, patterns simples
- **Advanced**: Conhecimentos profundos, patterns complexos
- **Expert**: Deep expertise, architecture decisions

### 3. Skills Definem Context

Skills indicam qual documentação e ferramentas o agent deve usar.

## Criando Nova Skill

1. Criar arquivo YAML em `.context/skills/`
2. Definir proficiency levels e capabilities
3. Listar tools e commands
4. Documentar patterns e best practices
5. Referenciar em agents que a possuem

## Exemplos de Uso

### Agent com Ruby Rails Expert

```yaml
agent: backend-specialist
skills:
  - id: ruby-rails
    proficiency: expert
    focus: ["Services", "Jobs", "Models"]
```

**Capabilities**:
- Design complex services
- Optimize database queries
- Implement background jobs
- Design domain models
- Handle transactions

### Agent com Vue Frontend Advanced

```yaml
agent: frontend-specialist
skills:
  - id: vue-frontend
    proficiency: advanced
    focus: ["Components", "State Management"]
```

**Capabilities**:
- Build complex components
- Manage Vuex state
- Implement composables
- Handle API integration
- Write component tests

## Referências

- [manifest.yaml](../manifest.yaml) - Agent configuration
- [chatwoot-specialist.yaml](../chatwoot-specialist.yaml) - Example agent with skills
- Individual skill files - Detailed skill definitions
