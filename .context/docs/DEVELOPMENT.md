# Guia de Desenvolvimento Chatwoot

## Setup Inicial

### Pré-requisitos

- **Ruby**: Versão especificada em `.ruby-version` (gerenciar via rbenv)
- **Node.js**: v18+ (recomendado via nvm)
- **PostgreSQL**: 12+
- **Redis**: 6+
- **pnpm**: Para gerenciamento de pacotes frontend

### Instalação do Ambiente

#### 1. Ruby com rbenv

```bash
# Instalar rbenv
curl -fsSL https://github.com/rbenv/rbenv-installer/raw/main/bin/rbenv-installer | bash

# Adicionar ao shell
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
source ~/.bashrc

# Instalar Ruby
rbenv install $(cat .ruby-version)
```

#### 2. Dependências do Sistema

**Ubuntu/Debian**:
```bash
sudo apt-get update
sudo apt-get install -y \
  postgresql postgresql-contrib \
  redis-server \
  imagemagick \
  libpq-dev \
  build-essential
```

**macOS**:
```bash
brew install postgresql redis imagemagick
brew services start postgresql
brew services start redis
```

#### 3. Instalar Dependências

```bash
# Bundler e gems Ruby
gem install bundler
bundle install

# Pacotes Node.js
npm install -g pnpm
pnpm install
```

#### 4. Configurar Banco de Dados

```bash
# Editar config/database.yml com credenciais do PostgreSQL

# Criar bancos de dados
bundle exec rails db:create

# Executar migrations
bundle exec rails db:migrate

# Seed inicial (opcional)
bundle exec rails db:seed
```

#### 5. Configurar Variáveis de Ambiente

Criar arquivo `.env` na raiz do projeto:

```bash
# Environment
RAILS_ENV=development
NODE_ENV=development

# Database
DATABASE_URL=postgresql://username:password@localhost/chatwoot_development

# Redis
REDIS_URL=redis://localhost:6379

# Rails
SECRET_KEY_BASE=generate_with_rails_secret

# Frontend
FRONTEND_URL=http://localhost:3000

# Storage (local development)
ACTIVE_STORAGE_SERVICE=local

# Optional: Channels
FACEBOOK_VERIFY_TOKEN=your_token
FACEBOOK_APP_SECRET=your_secret
FACEBOOK_APP_ID=your_app_id
```

Gerar `SECRET_KEY_BASE`:
```bash
bundle exec rails secret
```

---

## Desenvolvimento

### Executar Aplicação

#### Desenvolvimento com Overmind (Recomendado)

```bash
# Instalar overmind
brew install tmux overmind  # macOS
# ou
sudo apt-get install tmux && go install github.com/DarthSim/overmind/v2@latest  # Linux

# Executar todos os processos
overmind start -f Procfile.dev
```

`Procfile.dev` inicia:
- Rails server (porta 3000)
- Vite dev server (HMR para frontend)
- Sidekiq (background jobs)
- Webpack dev server (legado)

#### Desenvolvimento Manual

Terminal 1 - Rails:
```bash
bundle exec rails server
```

Terminal 2 - Vite (Frontend):
```bash
pnpm dev
```

Terminal 3 - Sidekiq (Background Jobs):
```bash
bundle exec sidekiq -C config/sidekiq.yml
```

### URLs de Acesso

- **Dashboard**: http://localhost:3000
- **API**: http://localhost:3000/api/v1
- **Sidekiq Dashboard**: http://localhost:3000/sidekiq (autenticado)

### Criar Conta e Usuário Inicial

1. Acessar http://localhost:3000
2. Clicar em "Create new account"
3. Preencher dados da conta e usuário
4. Confirmar email (em development, ver logs do Rails ou MailCatcher)

---

## Estrutura do Código

### Backend (Rails)

```
app/
├── models/           # Models ActiveRecord
├── controllers/      # Controllers (API e Views)
├── services/         # Lógica de negócio
├── jobs/             # Background jobs (Sidekiq)
├── listeners/        # Event listeners
├── policies/         # Authorization (Pundit)
├── presenters/       # Transformação de dados
├── builders/         # Construção de objetos complexos
├── finders/          # Queries complexas
└── channels/         # WebSocket channels
```

### Frontend (Vue.js)

```
app/javascript/dashboard/
├── api/              # API clients
├── components/       # Componentes reutilizáveis
├── components-next/  # Novos componentes (Composition API)
├── composables/      # Composition API composables
├── routes/           # Vue Router
├── store/            # Vuex store
├── views/            # Páginas/Views
├── helper/           # Funções auxiliares
└── i18n/             # Traduções
```

---

## Testes

### Setup de Testes

```bash
# Criar banco de testes
RAILS_ENV=test bundle exec rails db:create db:migrate
```

### Executar Testes

#### Ruby/Rails (RSpec)

```bash
# Todos os testes
bundle exec rspec

# Arquivo específico
bundle exec rspec spec/models/conversation_spec.rb

# Linha específica
bundle exec rspec spec/models/conversation_spec.rb:42

# Com coverage
COVERAGE=true bundle exec rspec

# Testes paralelos (mais rápido)
bundle exec parallel_rspec spec/
```

#### JavaScript/Vue (Vitest)

```bash
# Todos os testes
pnpm test

# Watch mode (re-executa ao salvar)
pnpm test:watch

# Arquivo específico
pnpm test src/components/Button.spec.js

# Coverage
pnpm test:coverage
```

### Escrever Testes

#### RSpec (Ruby)

**Model Spec**:
```ruby
# spec/models/conversation_spec.rb
require 'rails_helper'

RSpec.describe Conversation, type: :model do
  describe 'associations' do
    it { is_expected.to belong_to(:account) }
    it { is_expected.to belong_to(:inbox) }
    it { is_expected.to have_many(:messages) }
  end

  describe 'validations' do
    it { is_expected.to validate_presence_of(:account_id) }
  end

  describe '#toggle_status' do
    let(:conversation) { create(:conversation, status: :open) }

    it 'changes status from open to resolved' do
      conversation.toggle_status
      expect(conversation.reload.status).to eq('resolved')
    end
  end
end
```

**Controller Spec**:
```ruby
# spec/controllers/api/v1/accounts/conversations_controller_spec.rb
require 'rails_helper'

RSpec.describe Api::V1::Accounts::ConversationsController, type: :controller do
  let(:account) { create(:account) }
  let(:user) { create(:user, account: account) }
  let(:inbox) { create(:inbox, account: account) }

  before { sign_in user }

  describe 'GET #index' do
    it 'returns conversations' do
      create_list(:conversation, 3, account: account, inbox: inbox)
      
      get :index, params: { account_id: account.id }
      
      expect(response).to have_http_status(:success)
      expect(JSON.parse(response.body)['data']['payload'].length).to eq(3)
    end
  end
end
```

**Service Spec**:
```ruby
# spec/services/conversations/assignment_service_spec.rb
require 'rails_helper'

RSpec.describe Conversations::AssignmentService do
  subject(:service) { described_class.new(conversation: conversation, assignee: agent) }

  let(:conversation) { create(:conversation) }
  let(:agent) { create(:user) }

  describe '#perform' do
    it 'assigns agent to conversation' do
      expect { service.perform }
        .to change { conversation.reload.assignee }
        .from(nil).to(agent)
    end

    it 'creates activity message' do
      expect { service.perform }
        .to change { conversation.messages.activity.count }
        .by(1)
    end
  end
end
```

#### Vitest (JavaScript/Vue)

**Component Spec**:
```javascript
// app/javascript/dashboard/components/Button.spec.js
import { mount } from '@vue/test-utils'
import { describe, it, expect } from 'vitest'
import Button from './Button.vue'

describe('Button', () => {
  it('renders button text', () => {
    const wrapper = mount(Button, {
      slots: {
        default: 'Click me'
      }
    })
    
    expect(wrapper.text()).toContain('Click me')
  })

  it('emits click event', async () => {
    const wrapper = mount(Button)
    
    await wrapper.trigger('click')
    
    expect(wrapper.emitted('click')).toBeTruthy()
  })

  it('applies variant class', () => {
    const wrapper = mount(Button, {
      props: {
        variant: 'primary'
      }
    })
    
    expect(wrapper.classes()).toContain('bg-woot-500')
  })
})
```

**Composable Spec**:
```javascript
// app/javascript/dashboard/composables/useConversation.spec.js
import { describe, it, expect, vi } from 'vitest'
import { useConversation } from './useConversation'
import { ref } from 'vue'

describe('useConversation', () => {
  it('toggles conversation status', async () => {
    const conversation = ref({ id: 1, status: 'open' })
    const { toggleStatus } = useConversation(conversation)
    
    await toggleStatus()
    
    expect(conversation.value.status).toBe('resolved')
  })
})
```

### Factories (FactoryBot)

```ruby
# spec/factories/conversations.rb
FactoryBot.define do
  factory :conversation do
    association :account
    association :inbox
    association :contact
    
    status { :open }
    
    trait :resolved do
      status { :resolved }
    end
    
    trait :with_messages do
      after(:create) do |conversation|
        create_list(:message, 3, conversation: conversation)
      end
    end
  end
end

# Uso
conversation = create(:conversation)
resolved_conversation = create(:conversation, :resolved)
conversation_with_messages = create(:conversation, :with_messages)
```

---

## Linting e Formatação

### Ruby

```bash
# Lint
bundle exec rubocop

# Auto-fix
bundle exec rubocop -a

# Arquivo específico
bundle exec rubocop app/models/conversation.rb
```

**Configuração**: `.rubocop.yml`

### JavaScript/Vue

```bash
# Lint
pnpm eslint

# Auto-fix
pnpm eslint:fix

# Arquivo específico
pnpm eslint app/javascript/dashboard/components/Button.vue --fix
```

**Configuração**: `.eslintrc.js`, `.prettierrc`

---

## Migrations

### Criar Migration

```bash
# Adicionar coluna
bundle exec rails g migration AddPriorityToConversations priority:integer

# Criar tabela
bundle exec rails g migration CreateLabels title:string description:text color:string

# Adicionar índice
bundle exec rails g migration AddIndexToConversations
```

### Editar Migration

```ruby
# db/migrate/20240101000000_add_priority_to_conversations.rb
class AddPriorityToConversations < ActiveRecord::Migration[7.0]
  def change
    add_column :conversations, :priority, :integer
    add_index :conversations, :priority
  end
end
```

### Executar Migration

```bash
# Development
bundle exec rails db:migrate

# Test
RAILS_ENV=test bundle exec rails db:migrate

# Rollback (desfazer última)
bundle exec rails db:rollback

# Rollback N migrations
bundle exec rails db:rollback STEP=3

# Re-executar última migration
bundle exec rails db:migrate:redo
```

---

## Console Rails

```bash
# Abrir console
bundle exec rails console

# Ou modo sandbox (rollback ao sair)
bundle exec rails console --sandbox
```

### Comandos Úteis

```ruby
# Encontrar registros
account = Account.first
conversation = Conversation.find(123)
user = User.find_by(email: 'user@example.com')

# Criar registros
account = Account.create!(name: 'ACME Inc')

# Atualizar
conversation.update!(status: :resolved)

# Deletar
conversation.destroy

# Queries
Conversation.where(status: :open).count
Conversation.assigned_to(user).limit(10)

# Reload
conversation.reload

# Associations
conversation.messages.last
user.assigned_conversations

# JSON attributes
conversation.custom_attributes
conversation.custom_attributes['priority'] = 'high'
conversation.save

# Testar services
service = Conversations::AssignmentService.new(conversation: conversation, assignee: user)
service.perform

# Executar jobs
ConversationReplyEmailWorker.perform_async(conversation.id, 'hi@example.com')

# Sidekiq
Sidekiq::Queue.new.size
Sidekiq::Queue.new.clear
```

---

## Debug

### Byebug (Ruby)

Adicionar `byebug` no código:

```ruby
def my_method
  byebug  # Breakpoint aqui
  # código continua
end
```

Comandos no byebug:
```
next (n)      # Próxima linha
step (s)      # Entrar no método
continue (c)  # Continuar execução
list (l)      # Mostrar código ao redor
var local     # Mostrar variáveis locais
var instance  # Mostrar @variables
pp variable   # Pretty-print variável
exit          # Sair do debug
```

### Vue Devtools

Instalar extensão do navegador:
- Chrome: [Vue.js devtools](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd)
- Firefox: [Vue.js devtools](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/)

### Logs

```bash
# Tail logs em desenvolvimento
tail -f log/development.log

# Grep logs
grep "ERROR" log/development.log

# Sidekiq logs
tail -f log/sidekiq.log
```

---

## Git Workflow

### Branches

```bash
# Criar branch de feature
git checkout -b feature/conversation-priority

# Criar branch de bugfix
git checkout -b fix/message-notification
```

### Commits

Seguir Conventional Commits:

```bash
git commit -m "feat(conversations): add priority field"
git commit -m "fix(messages): resolve notification bug"
git commit -m "refactor(services): extract assignment logic"
git commit -m "docs(api): update conversation endpoints"
git commit -m "test(models): add conversation priority specs"
```

Tipos:
- `feat`: Nova funcionalidade
- `fix`: Correção de bug
- `refactor`: Refatoração de código
- `docs`: Documentação
- `test`: Testes
- `chore`: Tarefas de manutenção
- `style`: Formatação de código

### Pull Requests

1. Push da branch
2. Abrir PR no GitHub
3. Preencher template (se existir)
4. Aguardar CI passar
5. Solicitar review
6. Fazer ajustes se necessário
7. Merge após aprovação

---

## Enterprise Edition

### Estrutura

Código enterprise sobrepõe/estende OSS:

```
enterprise/
├── app/
│   ├── controllers/
│   │   └── enterprise/
│   │       └── api/
│   ├── models/
│   │   └── concerns/
│   │       └── enterprise/
│   └── services/
│       └── enterprise/
└── config/
    └── routes/
        └── enterprise.rb
```

### Padrões

#### Estender Model

```ruby
# enterprise/app/models/concerns/enterprise/conversation.rb
module Enterprise::Conversation
  extend ActiveSupport::Concern

  included do
    belongs_to :sla_policy, optional: true
  end

  def sla_breached?
    # Lógica Enterprise
  end
end

# app/models/conversation.rb
class Conversation < ApplicationRecord
  # ...
end

Conversation.include_mod_with('Enterprise::Conversation')
```

#### Sobrescrever Método

```ruby
# enterprise/app/models/concerns/enterprise/conversation.rb
module Enterprise::Conversation
  def toggle_status
    # Lógica customizada Enterprise
    log_sla_event
    super  # Chama método OSS
  end
end

Conversation.prepend_mod_with('Enterprise::Conversation')
```

#### Adicionar Rotas

```ruby
# enterprise/config/routes/enterprise.rb
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      namespace :accounts do
        resources :sla_policies
      end
    end
  end
end
```

### Testar Enterprise

```bash
# Testes enterprise
bundle exec rspec spec/enterprise/

# Contexto OSS + Enterprise
ENABLE_ENTERPRISE=true bundle exec rspec
```

---

## Performance

### Query Optimization

#### N+1 Queries

**Problema**:
```ruby
# Executa 1 query + N queries para cada conversa
conversations = Conversation.all
conversations.each do |conv|
  puts conv.contact.name  # N queries
end
```

**Solução**:
```ruby
# Executa apenas 2 queries
conversations = Conversation.includes(:contact)
conversations.each do |conv|
  puts conv.contact.name  # Cached
end
```

#### Eager Loading

```ruby
# Carregar associações
Conversation.includes(:messages, :contact, :assignee)

# Carregar associações aninhadas
Conversation.includes(:messages, contact: :contact_inboxes)

# Left join (retorna mesmo sem associação)
Conversation.left_joins(:assignee)

# Inner join
Conversation.joins(:assignee)
```

#### Select

```ruby
# Selecionar apenas campos necessários
Conversation.select(:id, :status, :display_id)

# Evita carregar todos os campos
```

#### Pluck

```ruby
# Retorna array de IDs (mais eficiente)
Conversation.pluck(:id)

# Array de arrays
Conversation.pluck(:id, :status)
```

#### Find Each

```ruby
# Para processar muitos registros (batch)
Conversation.find_each(batch_size: 1000) do |conv|
  # Processa conv
end
```

### Caching

#### Fragment Cache

```erb
<%# views/api/v1/conversations/show.json.jbuilder %>
<% cache conversation do %>
  json.id conversation.id
  json.status conversation.status
<% end %>
```

#### Rails Cache

```ruby
# Cache por 1 hora
Rails.cache.fetch("conversation_#{id}/stats", expires_in: 1.hour) do
  expensive_calculation
end

# Limpar cache
Rails.cache.delete("conversation_#{id}/stats")
```

#### Counter Cache

```ruby
# Migration
add_column :conversations, :messages_count, :integer, default: 0

# Model
class Message < ApplicationRecord
  belongs_to :conversation, counter_cache: true
end

# Uso
conversation.messages_count  # Não executa query
```

### Database Indexes

```ruby
# Migration
add_index :conversations, :status
add_index :conversations, [:account_id, :inbox_id]
add_index :conversations, :created_at

# Partial index (apenas status open)
add_index :conversations, :assignee_id, where: "status = 0"
```

### Background Jobs

Mover processamento pesado para jobs:

```ruby
# Antes (síncrono, lento)
def create
  conversation = Conversation.create!(params)
  SendNotificationService.new(conversation).perform
  WebhookService.new(conversation).deliver
  render json: conversation
end

# Depois (assíncrono, rápido)
def create
  conversation = Conversation.create!(params)
  NotificationJob.perform_later(conversation.id)
  WebhookJob.perform_later(conversation.id)
  render json: conversation
end
```

---

## Solução de Problemas

### Reset Banco de Dados

```bash
bundle exec rails db:drop db:create db:migrate db:seed
```

### Limpar Assets

```bash
# Limpar cache de assets
rm -rf tmp/cache public/assets public/packs

# Recompilar
bundle exec rails assets:precompile
```

### Reinstalar Gems

```bash
rm -rf vendor/bundle
bundle install
```

### Reinstalar Node Modules

```bash
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

### Sidekiq Travado

```bash
# Ver jobs na fila
bundle exec rails console
Sidekiq::Queue.new.size

# Limpar fila
Sidekiq::Queue.new.clear

# Reiniciar Sidekiq
pkill -9 sidekiq
bundle exec sidekiq -C config/sidekiq.yml
```

### Port Already in Use

```bash
# Encontrar processo usando porta 3000
lsof -ti:3000

# Matar processo
kill -9 $(lsof -ti:3000)
```

---

## Deploy

### Preparação

```bash
# Precompile assets
RAILS_ENV=production bundle exec rails assets:precompile

# Executar migrations
RAILS_ENV=production bundle exec rails db:migrate
```

### Docker

```bash
# Build image
docker build -t chatwoot:latest .

# Run container
docker run -d \
  -p 3000:3000 \
  -e SECRET_KEY_BASE=your_secret \
  -e DATABASE_URL=postgresql://... \
  -e REDIS_URL=redis://... \
  chatwoot:latest
```

### Heroku

```bash
# Criar app
heroku create my-chatwoot

# Adicionar addons
heroku addons:create heroku-postgresql:standard-0
heroku addons:create heroku-redis:premium-0

# Deploy
git push heroku main

# Migrations
heroku run rails db:migrate
```

---

## Recursos

### Documentação

- [Chatwoot Docs](https://www.chatwoot.com/docs)
- [Rails Guides](https://guides.rubyonrails.org)
- [Vue.js Docs](https://vuejs.org/guide/)
- [Tailwind CSS](https://tailwindcss.com/docs)

### Comunidade

- [GitHub Discussions](https://github.com/chatwoot/chatwoot/discussions)
- [Discord](https://discord.gg/cJXdrwS)
- [Forum](https://github.com/chatwoot/chatwoot/discussions)

### Ferramentas

- [Overmind](https://github.com/DarthSim/overmind) - Process manager
- [MailCatcher](https://mailcatcher.me/) - Email testing
- [Ngrok](https://ngrok.com/) - Tunneling para webhooks locais

---

## Conclusão

Este guia cobre os principais aspectos do desenvolvimento no Chatwoot. Para dúvidas específicas, consulte a documentação oficial ou a comunidade no Discord/Forum.

Happy coding! 🚀
