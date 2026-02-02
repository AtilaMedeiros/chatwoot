# Documentação do Chatwoot

Documentação completa e detalhada do codebase do Chatwoot.

## 📚 Índice de Documentos

### 1. [ARQUITETURA.md](./ARQUITETURA.md)
**Visão geral completa da arquitetura do Chatwoot**

Conteúdo:
- Estrutura de diretórios e organização do projeto
- Camadas da aplicação (Models, Controllers, Services, Jobs, Frontend)
- Fluxos principais (mensagens incoming/outgoing, atribuição)
- Integrações com canais (WhatsApp, Facebook, Email, etc)
- Real-time & WebSockets (Action Cable)
- Background jobs & queues (Sidekiq)
- Estrutura de testes
- Enterprise Edition
- Performance & otimização
- Segurança e monitoramento

**Para quem**: Desenvolvedores novos no projeto, arquitetos querendo entender o sistema

---

### 2. [MODELS.md](./MODELS.md)
**Guia detalhado dos models e banco de dados**

Conteúdo:
- Hierarquia e relacionamentos entre models
- Documentação completa dos principais models:
  - Account (multi-tenancy)
  - User (usuários e autenticação)
  - Conversation (core do sistema)
  - Contact (contatos/clientes)
  - Message (mensagens)
  - Inbox (canais de comunicação)
  - Team (equipes)
  - Label (tags/etiquetas)
- Atributos, enums, scopes, métodos importantes
- Concerns, validações, callbacks
- Padrões comuns (JSONB, store_accessors, enums)
- Database triggers

**Para quem**: Desenvolvedores backend, engenheiros de dados, quem vai modificar/criar models

---

### 3. [API.md](./API.md)
**Documentação completa das APIs REST**

Conteúdo:
- Visão geral das APIs (Client, Platform, Public, Super Admin)
- Autenticação (tokens, Devise, API keys)
- Endpoints detalhados:
  - Conversas (listar, criar, atualizar, atribuir, labels, snooze)
  - Mensagens (listar, criar, deletar, anexos)
  - Contatos (CRUD, busca, filtros)
  - Inboxes (CRUD, membros, configuração)
  - Times (CRUD, membros)
  - Respostas prontas (canned responses)
  - Labels
  - Webhooks
  - Relatórios (reports)
- Platform API para integrações
- Public API para widgets
- Webhooks (formato, eventos, validação)
- Rate limiting e paginação
- Códigos de erro
- Exemplos de integração (cURL, JavaScript, Python)

**Para quem**: Desenvolvedores de integrações, frontend, mobile, consumidores da API

---

### 4. [DEVELOPMENT.md](./DEVELOPMENT.md)
**Guia prático de desenvolvimento**

Conteúdo:
- Setup inicial completo (Ruby, Node, PostgreSQL, Redis)
- Instalação do ambiente
- Como executar a aplicação (Overmind, manual)
- Estrutura do código (backend e frontend)
- Testes (RSpec, Vitest, factories)
- Linting e formatação (RuboCop, ESLint)
- Migrations (criar, executar, rollback)
- Console Rails (comandos úteis)
- Debug (Byebug, Vue Devtools, logs)
- Git workflow (branches, commits, PRs)
- Enterprise Edition (estrutura, padrões, testes)
- Performance (query optimization, caching, indexes)
- Solução de problemas comuns
- Deploy (Docker, Heroku)
- Recursos e comunidade

**Para quem**: Todo desenvolvedor que vai contribuir com código

---

### 5. [CHEATSHEET.md](./CHEATSHEET.md)
**Referência rápida de comandos, patterns e snippets**

Conteúdo:
- Comandos Comuns (setup, console, migrations, testes, linting, git)
- Models Principais (tabela de referência)
- API Endpoints Essenciais (tabela com métodos e rotas)
- Padrões de Código (templates Ruby/Vue/RSpec/Vitest)
- Database (enums, statuses, content types)
- Queries Úteis (ActiveRecord snippets prontos)
- Scopes Comuns
- Authorization (Pundit)
- Frontend (composables, Vuex, API clients)
- Testing (factories, matchers)
- Debug (byebug, loggers, console)
- Performance (N+1, caching, optimizations)
- Troubleshooting
- Deploy Checklist

**Para quem**: Desenvolvedores que precisam de consulta rápida durante o desenvolvimento

---

### 6. [AI_CONTEXT_WORKFLOW.md](./AI_CONTEXT_WORKFLOW.md)
**Sistema de Agents especializados e Workflow PREVC**

Conteúdo:
- O que é AI-Context e seus componentes
- Sistema de 14 Agents especializados:
  - code-reviewer, bug-fixer, feature-developer
  - refactoring-specialist, test-writer, documentation-writer
  - performance-optimizer, security-auditor
  - backend/frontend/architect/devops/database/mobile specialists
- Workflow PREVC (Planning → Review → Execute → Validate → Complete):
  - Fase P: Planejamento e arquitetura
  - Fase R: Review de design e segurança
  - Fase E: Implementação com specialists
  - Fase V: Validação com testes e performance
  - Fase C: Documentação e deploy
- Plans & Task Management
- Skills detalhados de cada Agent
- Orquestração avançada (roles, sequências, handoffs)
- Gestão de documentos de workflow (PRD, Tech Spec, ADR)
- Exemplos práticos (feature, bug fix, refactoring)
- Best practices e troubleshooting
- Integração com Chatwoot

**Para quem**: Desenvolvedores trabalhando em features complexas, arquitetos, líderes técnicos que querem processo estruturado

---

## 🚀 Quick Start

### Novo no Projeto?

1. **Comece por**: [ARQUITETURA.md](./ARQUITETURA.md)
   - Entenda a estrutura geral do projeto
   - Conheça as principais tecnologias e padrões

2. **Configure o ambiente**: [DEVELOPMENT.md](./DEVELOPMENT.md) - Setup Inicial
   - Instale dependências
   - Configure banco de dados
   - Execute a aplicação

3. **Explore os models**: [MODELS.md](./MODELS.md)
   - Entenda as entidades principais
   - Veja os relacionamentos

4. **Teste a API**: [API.md](./API.md)
   - Faça chamadas de teste
   - Experimente criar conversas e mensagens

5. **Consulte referência rápida**: [CHEATSHEET.md](./CHEATSHEET.md)
   - Comandos e patterns prontos
   - Copie e adapte snippets de código

### Precisa de uma Funcionalidade Específica?

**Trabalhando com Conversas?**
→ [MODELS.md > Conversation](./MODELS.md#conversation) + [API.md > Conversas](./API.md#conversas)

**Criando um Canal Novo?**
→ [ARQUITETURA.md > Integrações com Canais](./ARQUITETURA.md#integrações-com-canais)

**Desenvolvendo Frontend?**
→ [ARQUITETURA.md > Frontend](./ARQUITETURA.md#6-frontend-appjavascript) + [DEVELOPMENT.md > Testes JavaScript](./DEVELOPMENT.md#javascriptvue-vitest)

**Fazendo Integrações?**
→ [API.md](./API.md) completo

**Otimizando Performance?**
→ [DEVELOPMENT.md > Performance](./DEVELOPMENT.md#performance)

**Desenvolvendo Feature Complexa?**
→ [AI_CONTEXT_WORKFLOW.md](./AI_CONTEXT_WORKFLOW.md) - Use workflow PREVC e agents especializados

**Precisa de Comando Rápido?**
→ [CHEATSHEET.md](./CHEATSHEET.md) - Referência rápida sempre à mão

---

## 📖 Padrões e Boas Práticas

### Código Ruby/Rails

✅ **DO**:
- Seguir RuboCop rules (150 chars por linha)
- Usar concerns para código compartilhado
- Services para lógica de negócio complexa
- Validações fortes nos models
- Background jobs para operações demoradas
- Includes/joins para evitar N+1 queries

❌ **DON'T**:
- Lógica de negócio nos controllers
- Queries complexas nos views/controllers
- Pular validações (`.update_column`, `.save(validate: false)`)
- Ignorar warnings do RuboCop
- Múltiplas responsabilidades em um service

### Código Vue.js/JavaScript

✅ **DO**:
- Usar Composition API com `<script setup>`
- Apenas Tailwind CSS (utility classes)
- PropTypes para validação de props
- i18n para todas as strings
- TypeScript para novos componentes
- Testes para componentes complexos

❌ **DON'T**:
- CSS customizado ou scoped
- Inline styles
- Hardcoded strings
- Options API (legado)
- Props sem validação

### Commits

Seguir [Conventional Commits](https://www.conventionalcommits.org/):

```
feat(scope): add new feature
fix(scope): fix bug
refactor(scope): refactor code
docs(scope): update documentation
test(scope): add tests
```

### Testes

- ✅ Escrever testes para lógica de negócio crítica
- ✅ Usar factories para criar dados de teste
- ✅ Testar edge cases e error paths
- ✅ Manter testes rápidos e isolados
- ❌ Não testar código trivial (getters/setters simples)
- ❌ Não acoplar testes à implementação interna

---

## 🏗️ Arquitetura em Resumo

```
┌─────────────────────────────────────────────────────┐
│                   Frontend (Vue.js)                  │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │  Dashboard   │  │   Widget     │  │   Survey   │ │
│  └──────────────┘  └──────────────┘  └────────────┘ │
└────────────────────────┬────────────────────────────┘
                         │ HTTP/WebSocket
┌────────────────────────┴────────────────────────────┐
│              Backend (Ruby on Rails)                 │
│  ┌──────────────────────────────────────────────┐   │
│  │             Controllers (API)                 │   │
│  └─────────────────────┬────────────────────────┘   │
│  ┌─────────────────────┴────────────────────────┐   │
│  │       Services / Jobs / Listeners            │   │
│  └─────────────────────┬────────────────────────┘   │
│  ┌─────────────────────┴────────────────────────┐   │
│  │         Models (ActiveRecord)                │   │
│  └─────────────────────┬────────────────────────┘   │
└────────────────────────┴────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
   PostgreSQL         Redis           Sidekiq
  (Data Store)     (Cache/Pub-Sub)  (Background Jobs)
```

### Fluxo de Mensagem

```
Webhook → Job → Service → Model → Event → Listener → 
  Job → Notification/Webhook/etc → WebSocket Broadcast
```

---

## 🔑 Conceitos-Chave

### Multi-tenancy
- Tudo é escoped por `Account`
- Isolamento completo de dados entre contas
- Configurações específicas por conta

### Event-Driven Architecture
- Events disparados em ações importantes
- Listeners processam events de forma assíncrona
- Desacoplamento entre componentes

### Channel Abstraction
- Inbox + Channel (polimórfico)
- Suporte a múltiplos canais de forma uniforme
- Fácil adicionar novos canais

### Real-time Communication
- Action Cable para WebSockets
- Broadcast de updates para usuários conectados
- Typing indicators, presence

---

## 🛠️ Ferramentas Recomendadas

### Desenvolvimento
- **IDE**: VS Code, RubyMine
- **Git**: Git CLI, GitHub CLI
- **API Testing**: Postman, Insomnia, curl
- **Database**: TablePlus, pgAdmin
- **Email Testing**: MailCatcher
- **Tunneling**: ngrok (para webhooks locais)

### Extensions VS Code
- Ruby LSP
- Vetur ou Volar (Vue)
- ESLint
- Tailwind CSS IntelliSense
- GitLens
- PostgreSQL

---

## 📚 Recursos Adicionais

### Documentação Externa
- [Chatwoot Official Docs](https://www.chatwoot.com/docs)
- [Rails Guides](https://guides.rubyonrails.org)
- [Vue.js Guide](https://vuejs.org/guide/)
- [Tailwind CSS](https://tailwindcss.com/docs)

### Comunidade
- [GitHub](https://github.com/chatwoot/chatwoot)
- [Discord](https://discord.gg/cJXdrwS)
- [Discussions](https://github.com/chatwoot/chatwoot/discussions)

### Tutoriais e Artigos
- [Chatwoot Blog](https://www.chatwoot.com/blog)
- [Contributing Guide](https://github.com/chatwoot/chatwoot/blob/develop/CONTRIBUTING.md)

---

## 🤝 Como Contribuir

1. Fork o repositório
2. Crie uma branch (`git checkout -b feature/amazing-feature`)
3. Faça suas alterações
4. Commit (`git commit -m 'feat: add amazing feature'`)
5. Push (`git push origin feature/amazing-feature`)
6. Abra um Pull Request

Consulte [DEVELOPMENT.md > Git Workflow](./DEVELOPMENT.md#git-workflow) para detalhes.

---

## 📝 Manutenção desta Documentação

Esta documentação foi gerada em **02/02/2026** e reflete o estado atual do codebase.

### Quando Atualizar

- ✏️ Ao adicionar novas features significativas
- ✏️ Ao mudar arquitetura ou padrões
- ✏️ Ao adicionar novos models ou endpoints
- ✏️ Ao modificar fluxos principais

### Como Atualizar

1. Edite o documento relevante em `.context/docs/`
2. Mantenha o formato e estrutura consistentes
3. Adicione exemplos quando apropriado
4. Atualize este README se necessário

---

## ❓ FAQ

**P: Por onde começo se sou novo no projeto?**
R: Leia [ARQUITETURA.md](./ARQUITETURA.md) primeiro, depois siga o Quick Start acima.

**P: Como adiciono um novo canal?**
R: Veja [ARQUITETURA.md > Integrações com Canais](./ARQUITETURA.md#integrações-com-canais).

**P: Onde fica a lógica de negócio?**
R: Em `app/services/`. Veja [ARQUITETURA.md > Services](./ARQUITETURA.md#3-services-appservices).

**P: Como testo minhas mudanças?**
R: Veja [DEVELOPMENT.md > Testes](./DEVELOPMENT.md#testes).

**P: A API está documentada?**
R: Sim! Veja [API.md](./API.md) para documentação completa.

**P: Como funciona o Enterprise Edition?**
R: Veja [ARQUITETURA.md > Enterprise Edition](./ARQUITETURA.md#enterprise-edition-notes) e [DEVELOPMENT.md > Enterprise Edition](./DEVELOPMENT.md#enterprise-edition).

---

**Última atualização**: 02/02/2026

**Versão do Chatwoot**: 3.x (develop branch)

**Mantenedores**: Time de desenvolvimento Chatwoot

---

*Esta documentação foi criada para facilitar onboarding e colaboração no projeto Chatwoot. Contribuições e melhorias são bem-vindas!* 🚀
