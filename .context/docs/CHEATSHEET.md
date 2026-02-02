# Cheat Sheet - Referência Rápida Chatwoot

## 🚀 Comandos Comuns

### Setup e Execução

```bash
# Instalar dependências
bundle install && pnpm install

# Criar e migrar banco
bundle exec rails db:create db:migrate

# Executar aplicação (recomendado)
overmind start -f Procfile.dev

# Ou separadamente:
bundle exec rails server          # Rails (porta 3000)
pnpm dev                          # Vite frontend
bundle exec sidekiq -C config/sidekiq.yml  # Background Jobs
```

### Console Rails

```bash
# Abrir console
bundle exec rails console

# Sandbox (rollback ao sair)
bundle exec rails console --sandbox

# Comandos úteis
account = Account.first
user = User.find_by(email: 'user@example.com')
Conversation.where(status: :open).count
```

### Migrations

```bash
# Criar migration
bundle exec rails g migration AddFieldToModel field:type

# Executar migrations
bundle exec rails db:migrate

# Rollback última migration
bundle exec rails db:rollback

# Redo (rollback + migrate)
bundle exec rails db:migrate:redo
```

### Testes

```bash
# Ruby/Rails
bundle exec rspec                              # Todos
bundle exec rspec spec/models/                # Diretório
bundle exec rspec spec/models/account_spec.rb # Arquivo
bundle exec rspec spec/models/account_spec.rb:42  # Linha

# JavaScript
pnpm test                  # Todos
pnpm test:watch           # Watch mode
pnpm test Button.spec.js  # Arquivo específico
```

### Linting

```bash
# Ruby
bundle exec rubocop        # Lint
bundle exec rubocop -a     # Auto-fix

# JavaScript
pnpm eslint               # Lint
pnpm eslint:fix           # Auto-fix
```

### Git

```bash
# Branch de feature
git checkout -b feature/my-feature

# Commit (Conventional Commits)
git commit -m "feat(scope): description"
git commit -m "fix(scope): description"

# Push e PR
git push origin feature/my-feature
```

---

## 📦 Models Principais

| Model | Propósito | Relações Principais |
|-------|-----------|---------------------|
| **Account** | Organização (multi-tenancy) | has_many :users, :inboxes, :conversations |
| **User** | Usuário/agente | has_many :accounts, :assigned_conversations |
| **Conversation** | Conversa (core) | belongs_to :account, :inbox, :contact, :assignee |
| **Contact** | Contato/cliente | has_many :conversations |
| **Message** | Mensagem | belongs_to :conversation, :sender |
| **Inbox** | Canal de comunicação | belongs_to :channel (polymorphic) |
| **Team** | Equipe de agentes | has_many :users, :conversations |

---

## 🔌 API Endpoints Essenciais

### Base URL
```
Client API: /api/v1/accounts/{account_id}
Platform API: /platform/api/v1/accounts/{account_id}
Public API: /public/api/v1
```

### Conversas

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/conversations` | Listar conversas |
| GET | `/conversations/{id}` | Detalhes |
| POST | `/conversations` | Criar |
| POST | `/conversations/{id}/toggle_status` | Mudar status |
| POST | `/conversations/{id}/assignments` | Atribuir |
| POST | `/conversations/{id}/labels` | Adicionar labels |

### Mensagens

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/conversations/{id}/messages` | Listar |
| POST | `/conversations/{id}/messages` | Criar |
| DELETE | `/conversations/{id}/messages/{msg_id}` | Deletar |

### Contatos

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/contacts` | Listar |
| GET | `/contacts/search` | Buscar |
| POST | `/contacts` | Criar |
| PATCH | `/contacts/{id}` | Atualizar |

### Autenticação

**Header**:
```
api_access_token: YOUR_TOKEN
```

**Obter Token**:
```bash
curl -X POST http://localhost:3000/api/v1/profile \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"password"}'
```

---

## 🎨 Padrões de Código

### Ruby Service

```ruby
class MyService
  def initialize(param1:, param2:)
    @param1 = param1
    @param2 = param2
  end

  def perform
    # Lógica principal
    result
  end

  private

  def helper_method
    # Métodos auxiliares privados
  end
end

# Uso
service = MyService.new(param1: value1, param2: value2)
service.perform
```

### Ruby Job

```ruby
class MyJob < ApplicationJob
  queue_as :default

  def perform(*args)
    # Lógica do job
  end
end

# Enqueue
MyJob.perform_later(arg1, arg2)
```

### Vue Component (Composition API)

```vue
<script setup>
import { ref, computed } from 'vue';

const props = defineProps({
  title: {
    type: String,
    required: true
  }
});

const emit = defineEmits(['click']);

const count = ref(0);
const doubleCount = computed(() => count.value * 2);

const handleClick = () => {
  count.value++;
  emit('click', count.value);
};
</script>

<template>
  <div>
    <h1>{{ title }}</h1>
    <button @click="handleClick">
      Count: {{ count }} ({{ doubleCount }})
    </button>
  </div>
</template>
```

### RSpec Test

```ruby
require 'rails_helper'

RSpec.describe MyClass do
  describe '#my_method' do
    let(:instance) { described_class.new(param: 'value') }
    
    it 'does something' do
      result = instance.my_method
      
      expect(result).to eq('expected')
    end
    
    context 'when condition' do
      it 'does something else' do
        # ...
      end
    end
  end
end
```

### Vitest Test

```javascript
import { describe, it, expect } from 'vitest';
import { mount } from '@vue/test-utils';
import MyComponent from './MyComponent.vue';

describe('MyComponent', () => {
  it('renders correctly', () => {
    const wrapper = mount(MyComponent, {
      props: { title: 'Test' }
    });
    
    expect(wrapper.text()).toContain('Test');
  });
});
```

---

## 💾 Database

### Conversation Statuses

```ruby
enum status: {
  open: 0,      # Aberta
  resolved: 1,  # Resolvida
  pending: 2,   # Pendente (aguardando)
  snoozed: 3    # Adiada
}
```

### Conversation Priorities

```ruby
enum priority: {
  low: 0,
  medium: 1,
  high: 2,
  urgent: 3
}
```

### Message Types

```ruby
enum message_type: {
  incoming: 0,  # Do contato
  outgoing: 1,  # Do agente
  activity: 2,  # Sistema
  template: 3   # Template
}
```

### Message Content Types

```ruby
enum content_type: {
  text: 0,
  input_select: 1,
  cards: 2,
  form: 3,
  article: 4,
  incoming_email: 5,
  # ...
}
```

---

## 🔍 Queries Úteis

### Conversas

```ruby
# Abertas
Conversation.open

# Não atribuídas
Conversation.unassigned

# Atribuídas a um agente
Conversation.assigned_to(agent)

# Por inbox
Conversation.where(inbox_id: inbox.id)

# Com eager loading
Conversation.includes(:messages, :contact, :assignee)

# Contagem por status
Conversation.group(:status).count
```

### Mensagens

```ruby
# Incoming
Message.incoming

# Outgoing
Message.outgoing

# Chat (não activity)
Message.chat

# Hoje
Message.today

# Com anexos
Message.joins(:attachments).distinct
```

### Usuários

```ruby
# Agentes de uma conta
account.agents

# Online
User.where(availability: :online)

# Com conversas atribuídas
User.joins(:assigned_conversations).distinct
```

---

## 🎯 Scopes Comuns

### Account

```ruby
Account.active
Account.suspended
Account.with_auto_resolve
```

### Conversation

```ruby
Conversation.open
Conversation.resolved
Conversation.pending
Conversation.snoozed
Conversation.unassigned
Conversation.assigned
Conversation.unattended
```

### Message

```ruby
Message.incoming
Message.outgoing
Message.chat
Message.today
Message.created_since(time)
```

---

## 🔐 Authorization (Pundit)

### Policy

```ruby
class ConversationPolicy < ApplicationPolicy
  def index?
    user.present?
  end

  def show?
    record.account_id == user.account_id
  end

  def create?
    user.administrator?(record.account)
  end

  def update?
    user.administrator?(record.account) || record.assignee_id == user.id
  end
end
```

### Controller Usage

```ruby
class ConversationsController < ApplicationController
  def show
    @conversation = Conversation.find(params[:id])
    authorize @conversation  # Chama ConversationPolicy#show?
  end
end
```

---

## 🎨 Frontend (Vue)

### Composables

```javascript
// useConversation.js
import { ref, computed } from 'vue';

export function useConversation(conversation) {
  const isOpen = computed(() => conversation.value.status === 'open');
  
  const toggleStatus = async () => {
    // Lógica
  };
  
  return {
    isOpen,
    toggleStatus
  };
}

// Uso
import { useConversation } from '@/composables/useConversation';

const { isOpen, toggleStatus } = useConversation(conversation);
```

### Vuex Store

```javascript
// store/modules/conversations.js
export default {
  namespaced: true,
  
  state: {
    records: [],
    uiFlags: { isFetching: false }
  },
  
  getters: {
    getConversations: state => state.records,
    getUIFlags: state => state.uiFlags
  },
  
  mutations: {
    SET_CONVERSATIONS(state, conversations) {
      state.records = conversations;
    }
  },
  
  actions: {
    async fetchConversations({ commit }) {
      // API call
      const { data } = await ConversationAPI.get();
      commit('SET_CONVERSATIONS', data);
    }
  }
};
```

### API Client

```javascript
// api/conversations.js
import ApiClient from './ApiClient';

class ConversationAPI extends ApiClient {
  constructor() {
    super('conversations', { accountScoped: true });
  }

  toggleStatus(conversationId) {
    return axios.post(`${this.url}/${conversationId}/toggle_status`);
  }
}

export default new ConversationAPI();
```

---

## 🧪 Testing

### Factory (FactoryBot)

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
  end
end

# Uso
create(:conversation)
create(:conversation, :resolved)
build(:conversation)
```

### RSpec Matchers

```ruby
# Model spec
it { is_expected.to belong_to(:account) }
it { is_expected.to have_many(:messages) }
it { is_expected.to validate_presence_of(:account_id) }
it { is_expected.to define_enum_for(:status) }

# Expectation matchers
expect(conversation).to be_open
expect(conversation.status).to eq('open')
expect { service.perform }.to change { Conversation.count }.by(1)
```

---

## 🐛 Debug

### Byebug

```ruby
def my_method
  byebug  # Breakpoint
  # ...
end

# Comandos no byebug:
# n (next) - próxima linha
# s (step) - entrar no método
# c (continue) - continuar
# var local - variáveis locais
# pp variable - pretty print
```

### Rails Logger

```ruby
Rails.logger.info "Message: #{message.inspect}"
Rails.logger.debug "Processing conversation #{conversation.id}"
Rails.logger.error "Error: #{error.message}"
```

### Console Debugging

```ruby
# No console
reload!  # Recarregar código
_  # Último retorno
ap object  # Pretty print (awesome_print)
```

---

## 📊 Performance

### N+1 Prevention

```ruby
# ❌ Bad (N+1)
conversations = Conversation.all
conversations.each { |c| puts c.contact.name }

# ✅ Good
conversations = Conversation.includes(:contact)
conversations.each { |c| puts c.contact.name }
```

### Query Optimization

```ruby
# Select apenas campos necessários
Conversation.select(:id, :status, :display_id)

# Pluck para arrays simples
Conversation.pluck(:id)

# Find each para grandes volumes
Conversation.find_each(batch_size: 1000) do |conv|
  # Processa
end
```

### Caching

```ruby
# Rails cache
Rails.cache.fetch("key", expires_in: 1.hour) do
  expensive_operation
end

# Counter cache
class Message < ApplicationRecord
  belongs_to :conversation, counter_cache: true
end
conversation.messages_count  # Não executa query
```

---

## 🔧 Troubleshooting

### Port em Uso

```bash
# Encontrar processo
lsof -ti:3000

# Matar processo
kill -9 $(lsof -ti:3000)
```

### Reset Banco

```bash
bundle exec rails db:drop db:create db:migrate db:seed
```

### Limpar Cache

```bash
rm -rf tmp/cache
bundle exec rails tmp:clear
```

### Sidekiq Travado

```ruby
# No console
Sidekiq::Queue.new.size  # Ver tamanho da fila
Sidekiq::Queue.new.clear # Limpar fila
```

---

## 📚 Variáveis de Ambiente Essenciais

```bash
# Core
DATABASE_URL=postgresql://user:pass@localhost/chatwoot_dev
REDIS_URL=redis://localhost:6379
SECRET_KEY_BASE=your_secret_key
RAILS_ENV=development
NODE_ENV=development

# Frontend
FRONTEND_URL=http://localhost:3000

# Storage
ACTIVE_STORAGE_SERVICE=local

# Email (opcional)
SMTP_ADDRESS=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=your_email
SMTP_PASSWORD=your_password
```

---

## 🚀 Deploy Checklist

```bash
# 1. Precompile assets
RAILS_ENV=production bundle exec rails assets:precompile

# 2. Run migrations
RAILS_ENV=production bundle exec rails db:migrate

# 3. Restart server
# (depende do método de deploy)

# 4. Restart sidekiq
# (depende do método de deploy)
```

---

## 🔗 Links Rápidos

- [GitHub Repo](https://github.com/chatwoot/chatwoot)
- [Official Docs](https://www.chatwoot.com/docs)
- [API Docs](https://www.chatwoot.com/developers/api/)
- [Discord](https://discord.gg/cJXdrwS)
- [Forum](https://github.com/chatwoot/chatwoot/discussions)

---

## 🎯 Comandos Favoritos

```bash
# Meu combo favorito para começar o dia:
git pull origin develop
bundle install && pnpm install
bundle exec rails db:migrate
overmind start -f Procfile.dev

# Para testar uma feature:
bundle exec rspec spec/my_feature/
pnpm test my_feature.spec.js
bundle exec rubocop app/my_feature/
pnpm eslint app/javascript/my_feature/

# Antes de commitar:
bundle exec rspec
pnpm test
bundle exec rubocop -a
pnpm eslint:fix
```

---

**💡 Dica**: Salve este arquivo em seus favoritos para consulta rápida!

**📌 Bookmarklet**: Adicione aos seus favoritos do navegador para acesso rápido à documentação local.

---

*Última atualização: 02/02/2026*
