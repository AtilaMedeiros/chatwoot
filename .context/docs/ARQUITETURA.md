# Arquitetura do Chatwoot

## Visão Geral

Chatwoot é uma plataforma de atendimento ao cliente open-source construída com Ruby on Rails (backend) e Vue.js (frontend). A aplicação segue o padrão MVC (Model-View-Controller) com uma arquitetura modular que inclui:

- **Backend**: Ruby on Rails 7.x com PostgreSQL
- **Frontend**: Vue 3 (Composition API) com Tailwind CSS
- **Real-time**: Action Cable (WebSockets)
- **Background Jobs**: Sidekiq
- **Cache**: Redis
- **Storage**: Active Storage com suporte a múltiplos providers

## Estrutura de Diretórios

```
chatwoot/
├── app/
│   ├── actions/          # Actions para operações de negócio específicas
│   ├── assets/           # Assets (imagens, fontes)
│   ├── builders/         # Builders para construção de objetos complexos
│   ├── channels/         # Action Cable channels
│   ├── controllers/      # Controllers Rails
│   ├── dashboards/       # Dashboards (visualizações administrativas)
│   ├── dispatchers/      # Event dispatchers para processamento assíncrono
│   ├── drops/            # Liquid drops para templates
│   ├── fields/           # Campos customizados
│   ├── finders/          # Finders para queries complexas
│   ├── helpers/          # View helpers
│   ├── javascript/       # Frontend Vue.js
│   ├── jobs/             # Background jobs (Sidekiq)
│   ├── listeners/        # Event listeners
│   ├── mailboxes/        # Action Mailbox mailboxes
│   ├── mailers/          # Mailers para emails
│   ├── models/           # Models ActiveRecord
│   ├── policies/         # Authorization policies (Pundit)
│   ├── presenters/       # Presenters para transformação de dados
│   ├── services/         # Services para lógica de negócio
│   └── views/            # Views Rails (APIs e emails)
├── config/               # Configurações da aplicação
├── db/                   # Migrações e seeds do banco
├── enterprise/           # Código Enterprise Edition
├── lib/                  # Bibliotecas customizadas
├── public/               # Assets públicos estáticos
└── spec/                 # Specs (testes com RSpec)
```

## Camadas da Aplicação

### 1. Models (app/models/)

Os models representam as entidades do domínio e encapsulam a lógica de dados.

#### Principais Models:

- **Account**: Representa uma conta/organização no Chatwoot
  - Relacionamentos: users, inboxes, conversations, contacts, teams
  - Features: multi-tenancy, feature flags, configurações customizadas
  - Suporta settings via store_accessor (auto_resolve, captain_models, etc)

- **Conversation**: Núcleo do sistema, representa uma conversa
  - Status: open, resolved, pending, snoozed
  - Priority: low, medium, high, urgent
  - Relacionamentos: account, inbox, assignee, contact, messages, team
  - Concerns: Labelable, AssignmentHandler, ActivityMessageHandler
  - Triggers de banco para display_id e UUID

- **Contact**: Representa um contato/cliente
  - Relacionamentos: account, conversations, contact_inboxes
  - Atributos customizados via JSONB
  - Suporte a múltiplos identificadores (email, phone, etc)

- **Inbox**: Representa um canal de comunicação
  - Tipos: WebWidget, Email, Facebook, WhatsApp, Twitter, Telegram, etc
  - Relacionamentos: account, conversations, channel (polimórfico)
  - Configurações específicas por tipo de canal

- **Message**: Representa uma mensagem em uma conversa
  - Tipos: incoming, outgoing, activity, template
  - Content types: text, input_select, cards, form, article
  - Relacionamentos: conversation, sender, attachments
  - Suporte a arquivos via Active Storage

- **User**: Representa um usuário do sistema
  - Roles via account_users: administrator, agent
  - Relacionamentos: accounts, assigned_conversations, teams
  - Autenticação via Devise

- **Team**: Agrupa agentes por equipes
  - Relacionamentos: account, team_members, conversations, inboxes

#### Padrões nos Models:

- **Concerns**: Módulos reutilizáveis (Labelable, Avatarable, etc)
- **Enums**: Para campos com valores fixos (status, priority, role)
- **Scopes**: Queries nomeadas e reutilizáveis
- **Validations**: Validações de dados consistentes
- **Callbacks**: before_validation, after_commit para lógica adicional
- **JSONB**: Uso extensivo para atributos flexíveis (custom_attributes, additional_attributes)
- **Triggers**: Triggers de banco de dados para campos gerados (display_id)

### 2. Controllers (app/controllers/)

Organização hierárquica de controllers:

```
controllers/
├── api/
│   └── v1/              # API REST v1
│       ├── accounts/      # Recursos escoped por account
│       │   ├── conversations_controller.rb
│       │   ├── contacts_controller.rb
│       │   ├── inboxes_controller.rb
│       │   ├── messages_controller.rb
│       │   ├── teams_controller.rb
│       │   └── ...
│       └── widget/        # API pública do widget
├── platform/             # Platform API (integrações externas)
├── public/               # APIs públicas (webhooks, etc)
├── super_admin/          # Painel super admin
└── webhooks/             # Webhooks de canais (Twilio, Slack, etc)
```

#### Padrões nos Controllers:

- **Herança**: Api::V1::Accounts::BaseController < Api::BaseController
- **Authentication**: via tokens (JWT, API keys)
- **Authorization**: via Pundit policies
- **Serialization**: ActiveModel::Serializers ou Jbuilder
- **Pagination**: via Kaminari
- **Rate Limiting**: via Rack::Attack
- **Filtering**: via params customizados

### 3. Services (app/services/)

Services encapsulam lógica de negócio complexa.

#### Principais Services:

- **Conversations/**
  - `AssignmentService`: Atribuição de conversas a agentes
  - `FilterService`: Filtragem avançada de conversas
  - `MessageWindowService`: Controle de janela de mensagens (política de resposta)
  - `TransferService`: Transferência de conversas entre agentes/inboxes
  - `ResolveService`: Resolução automática de conversas

- **Messages/**
  - `MessageBuilder`: Construção de mensagens
  - `TemplateBuilder`: Construção de mensagens via templates
  - `Instagram/MessageBuilder`: Builders específicos por canal

- **Contacts/**
  - `ContactBuilder`: Criação/atualização de contatos
  - `ContactMergeAction`: Merge de contatos duplicados
  - `FilterService`: Filtragem de contatos

- **Integrations/**
  - Services para integrações com plataformas externas
  - Webhooks handlers

#### Padrão de Services:

```ruby
class ServiceName
  def initialize(params)
    @param1 = params[:param1]
    @param2 = params[:param2]
  end

  def perform
    # Lógica principal
    result
  end

  private

  def helper_method
    # Métodos auxiliares
  end
end
```

### 4. Jobs (app/jobs/)

Background jobs executados via Sidekiq.

#### Principais Jobs:

- **ConversationReplyEmailWorker**: Envio de emails de resposta
- **EventDispatcherJob**: Despacho de eventos
- **AutoResolveConversationsJob**: Resolução automática de conversas
- **AgentBotJob**: Processamento de bot
- **ReportingEventsJob**: Geração de eventos para reporting
- **WebhookJob**: Envio de webhooks

#### Padrões em Jobs:

```ruby
class JobName < ApplicationJob
  queue_as :default

  def perform(*args)
    # Lógica do job
  end
end
```

### 5. Listeners (app/listeners/)

Event listeners para processamento assíncrono de eventos.

#### Principais Listeners:

- **ConversationListener**: Eventos de conversas
- **MessageListener**: Eventos de mensagens
- **NotificationListener**: Criação de notificações
- **ReportingListener**: Eventos de reporting
- **WebhookListener**: Envio de webhooks

#### Padrão de Listeners:

```ruby
class ListenerName < BaseListener
  def conversation_created(event)
    # Lógica quando conversa é criada
  end

  def conversation_updated(event)
    # Lógica quando conversa é atualizada
  end
end
```

### 6. Frontend (app/javascript/)

Frontend Vue.js 3 com Composition API.

```
javascript/
├── dashboard/         # Aplicação principal do dashboard
│   ├── api/            # Clients API
│   ├── assets/         # Assets (imagens, SVGs)
│   ├── components/     # Componentes Vue reutilizáveis
│   ├── components-next/ # Novos componentes (migrando)
│   ├── composables/    # Composition API composables
│   ├── helper/         # Funções auxiliares
│   ├── i18n/           # Internacionalização
│   ├── mixins/         # Mixins (legado)
│   ├── modules/        # Módulos Vuex
│   ├── routes/         # Rotas Vue Router
│   ├── store/          # Vuex store
│   └── views/          # Views/páginas
├── sdk/               # SDK do widget
├── survey/            # App de surveys CSAT
└── widget/            # Widget de chat
```

#### Padrões no Frontend:

- **Composition API**: Usar `<script setup>` no topo dos componentes
- **Tailwind CSS**: Apenas utility classes, sem CSS customizado
- **TypeScript**: Componentes novos devem usar TypeScript
- **i18n**: Todas as strings devem estar em arquivos de tradução
- **State Management**: Vuex (migrando para Pinia)
- **API Calls**: Via axios com interceptors
- **WebSockets**: Via Action Cable para real-time

### 7. Policies (app/policies/)

Authorization via Pundit.

#### Principais Policies:

- **ConversationPolicy**: Autorização para conversas
- **ContactPolicy**: Autorização para contatos
- **InboxPolicy**: Autorização para inboxes
- **UserPolicy**: Autorização para usuários

#### Padrão de Policies:

```ruby
class PolicyName < ApplicationPolicy
  def index?
    # Pode listar?
  end

  def show?
    # Pode visualizar?
  end

  def create?
    # Pode criar?
  end

  def update?
    # Pode atualizar?
  end

  def destroy?
    # Pode deletar?
  end
end
```

## Fluxos Principais

### 1. Fluxo de Mensagem Incoming

```
1. Webhook recebido (webhooks_controller.rb)
   ↓
2. Job enfileirado (ChannelWebhookJob)
   ↓
3. Processa webhook via service (Whatsapp::IncomingMessageService)
   ↓
4. Cria/atualiza conversa (ContactInboxBuilder, ConversationBuilder)
   ↓
5. Cria mensagem (Messages::MessageBuilder)
   ↓
6. Dispara eventos (CONVERSATION_CREATED, MESSAGE_CREATED)
   ↓
7. Listeners processam eventos:
   - NotificationListener → cria notificações
   - WebhookListener → envia webhooks
   - ReportingListener → registra eventos
   ↓
8. Broadcast via WebSocket (ConversationChannel)
```

### 2. Fluxo de Mensagem Outgoing

```
1. API request (messages_controller#create)
   ↓
2. Authorization (ConversationPolicy)
   ↓
3. Cria mensagem (Messages::MessageBuilder)
   ↓
4. Envia via canal (Whatsapp::SendOnWhatsappService)
   ↓
5. Atualiza status da mensagem
   ↓
6. Dispara eventos (MESSAGE_CREATED)
   ↓
7. Broadcast via WebSocket
```

### 3. Fluxo de Atribuição de Conversa

```
1. API request (conversations_controller#assign_agent)
   ↓
2. Service (Conversations::AssignmentService)
   ↓
3. Atualiza conversa (conversation.update)
   ↓
4. Cria activity message
   ↓
5. Dispara evento (CONVERSATION_UPDATED)
   ↓
6. NotificationListener → notifica agente atribuído
```

## Integrações com Canais

Chatwoot suporta múltiplos canais de comunicação através de um sistema de channels polimórfico.

### Channels Suportados:

- **WebWidget**: Widget de chat incorporável
- **Email**: Inbox de email (IMAP/SMTP)
- **WhatsApp**: Via APIs oficiais (Cloud API, Business API)
- **Facebook**: Messenger e comentários
- **Instagram**: Direct messages e comentários
- **Twitter**: DMs e menções
- **Telegram**: Bot API
- **SMS**: Twilio, Bandwidth, etc
- **Line**: Line messaging
- **API**: Canal customizável via API

### Estrutura de Channel:

```ruby
# Modelo polimórfico
class Inbox
  belongs_to :channel, polymorphic: true
end

# Exemplo: Channel::Whatsapp
class Channel::Whatsapp < ApplicationRecord
  include Channelable
  
  validates :phone_number, presence: true
  validates :provider, presence: true
end
```

## Real-time & WebSockets

Comunicação em tempo real via Action Cable.

### Principais Channels:

- **ConversationChannel**: Updates de conversas
- **NotificationChannel**: Notificações
- **RoomChannel**: Presença/typing indicators
- **CampaignChannel**: Status de campanhas

### Padrão:

```ruby
class ConversationChannel < ApplicationCable::Channel
  def subscribed
    stream_for account
  end

  def unsubscribed
    stop_all_streams
  end
end
```

## Background Jobs & Queue

Sidekiq para processamento assíncrono com múltiplas filas:

- **default**: Jobs gerais
- **mailers**: Envio de emails
- **webhooks**: Envio de webhooks
- **integrations**: Integrações externas
- **scheduled**: Jobs agendados

## Testes

### Estrutura:

```
spec/
├── controllers/      # Testes de controllers
├── models/           # Testes de models
├── services/         # Testes de services
├── jobs/             # Testes de jobs
├── factories/        # Factories para fixtures
├── support/          # Helpers e configurações
└── enterprise/       # Testes EE
```

### Padrões:

- **RSpec**: Framework de testes
- **FactoryBot**: Factories para criação de dados
- **Faker**: Dados fake
- **Shoulda Matchers**: Matchers para validações
- **Webmock**: Mock de requests HTTP
- **SimpleCov**: Coverage

### Executar testes:

```bash
# Todos os testes
bundle exec rspec

# Arquivo específico
bundle exec rspec spec/models/conversation_spec.rb

# Linha específica
bundle exec rspec spec/models/conversation_spec.rb:42
```

## Enterprise Edition

Código enterprise sob `enterprise/` que estende/sobrescreve funcionalidades OSS.

### Estrutura:

```
enterprise/
├── app/
│   ├── controllers/    # Overrides e novos controllers
│   ├── models/         # Extensions e concerns
│   ├── services/       # Services enterprise
│   └── views/          # Views customizadas
└── config/
    └── routes/         # Rotas adicionais
```

### Padrões:

- **prepend_mod_with**: Para sobrescrever métodos OSS
- **include_mod_with**: Para adicionar funcionalidades
- Manter compatibilidade com OSS
- Extensions via concerns

## Configuração e Deploy

### Variáveis de Ambiente Principais:

- `DATABASE_URL`: URL do PostgreSQL
- `REDIS_URL`: URL do Redis
- `SECRET_KEY_BASE`: Secret key do Rails
- `FRONTEND_URL`: URL do frontend
- `ACTIVE_STORAGE_SERVICE`: Serviço de storage (local, s3, etc)
- Canal-specific: `WHATSAPP_*`, `FACEBOOK_*`, etc

### Deploy:

- Suportado via: Docker, Heroku, Caprover, Kubernetes
- Assets pré-compilados via `vite build`
- Migrations automáticas via `rails db:migrate`
- Background jobs via Sidekiq separado

## Performance & Otimização

### Estratégias:

- **Database Indexes**: Índices estratégicos nos models
- **Counter Caches**: Para contadores (messages_count, etc)
- **Fragment Caching**: Cache de fragmentos de views
- **Query Optimization**: N+1 queries resolvidas com includes/joins
- **Background Processing**: Jobs assíncronos para operações pesadas
- **Redis Caching**: Cache de dados frequentemente acessados
- **CDN**: Para assets estáticos

## Padrões de Código

### Ruby/Rails:

- Seguir Ruby Style Guide
- RuboCop configurado (150 chars por linha)
- Concerns para código compartilhado
- Services para lógica de negócio
- Validações fortes nos models
- Strong parameters nos controllers

### Vue/JavaScript:

- ESLint + Airbnb config
- Composition API com `<script setup>`
- Apenas Tailwind CSS (sem CSS customizado)
- TypeScript para novos componentes
- i18n obrigatório (sem strings hardcoded)
- PropTypes para validação

### Geral:

- MVP focus: menos código, happy path primeiro
- Código legível > código clever
- Testes para código crítico
- Documentação inline onde necessário
- Commits seguindo Conventional Commits

## Segurança

### Medidas:

- **Authentication**: Devise + JWT tokens
- **Authorization**: Pundit policies
- **CSRF Protection**: Rails built-in
- **SQL Injection**: ActiveRecord escaping
- **XSS**: Rails auto-escaping
- **Rate Limiting**: Rack::Attack
- **API Keys**: Encriptados no banco
- **Content Security Policy**: Headers configurados
- **CORS**: Whitelist de origins

## Monitoramento

### Ferramentas suportadas:

- **APM**: New Relic, Scout APM, Elastic APM
- **Error Tracking**: Sentry, Bugsnag
- **Logging**: Rails logger, JSON structured logs
- **Metrics**: Statsd, Prometheus

## Conclusão

Chatwoot é uma aplicação Rails moderna e bem estruturada que segue boas práticas de desenvolvimento. A arquitetura modular facilita extensão e manutenção, enquanto o código limpo e bem testado garante qualidade e confiabilidade.
