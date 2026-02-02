# Guia de Models do Chatwoot

## Hierarquia e Relacionamentos

### Diagrama Conceitual

```
Account (Organização)
  ├── Users (via account_users)
  ├── Inboxes (Canais de comunicação)
  │   └── Channel (polimórfico: WebWidget, WhatsApp, Email, etc)
  ├── Contacts (Clientes)
  │   └── ContactInboxes (Contato por inbox)
  ├── Conversations (Conversas)
  │   ├── Messages (Mensagens)
  │   │   └── Attachments (Anexos)
  │   ├── ConversationParticipants (Participantes)
  │   └── Mentions (Menções)
  ├── Teams (Equipes)
  │   └── TeamMembers (Membros)
  ├── Labels (Etiquetas)
  ├── CannedResponses (Respostas prontas)
  ├── Macros (Automações)
  ├── AutomationRules (Regras de automação)
  └── Webhooks (Webhooks)
```

## Models Principais

### Account

**Propósito**: Representa uma organização/conta no sistema (multi-tenancy).

**Atributos Principais**:
```ruby
# Básicos
name: string                    # Nome da conta
domain: string                  # Domínio customizado
support_email: string           # Email de suporte
status: enum                    # active, suspended
locale: enum                    # Idioma padrão (en, pt_BR, etc)

# Configurações (JSONB)
settings: {
  auto_resolve_after: integer          # Minutos para auto-resolver
  auto_resolve_message: string         # Mensagem ao resolver
  auto_resolve_ignore_waiting: boolean # Ignorar conversas em espera
  audio_transcriptions: boolean        # Habilitar transcrição de áudio
  auto_resolve_label: string          # Label para auto-resolve
  captain_models: {                   # Modelos de IA configurados
    editor: string,
    assistant: string,
    copilot: string,
    label_suggestion: string
  },
  captain_features: {                 # Features de IA habilitadas
    editor: boolean,
    assistant: boolean,
    copilot: boolean
  }
}

# Feature Flags (bit flags)
feature_flags: bigint           # Flags de funcionalidades

# Atributos customizados
custom_attributes: jsonb        # Atributos customizados
```

**Relacionamentos**:
- `has_many :users, through: :account_users`
- `has_many :inboxes`
- `has_many :contacts`
- `has_many :conversations`
- `has_many :teams`
- `has_many :labels`
- `has_many :webhooks`

**Métodos Importantes**:
```ruby
def agents
  # Retorna apenas usuários com role de agent
end

def auto_resolve_after
  # Configuração de auto-resolve em minutos
end

def captain_enabled?
  # Verifica se Captain AI está habilitado
end
```

**Scopes**:
```ruby
Account.with_auto_resolve  # Contas com auto-resolve configurado
Account.active             # Contas ativas
Account.suspended          # Contas suspensas
```

---

### User

**Propósito**: Representa um usuário do sistema (agente ou administrador).

**Atributos Principais**:
```ruby
name: string                    # Nome completo
email: string                   # Email (único e índice)
phone_number: string            # Telefone opcional
availability: enum              # online, offline, busy
confirmed_at: datetime          # Confirmação de email (Devise)
custom_attributes: jsonb        # Atributos customizados
ui_settings: jsonb              # Preferências de UI
```

**Roles** (via AccountUser):
- `administrator`: Acesso total
- `agent`: Acesso limitado a conversas

**Relacionamentos**:
- `has_many :accounts, through: :account_users`
- `has_many :assigned_conversations, class_name: 'Conversation', foreign_key: 'assignee_id'`
- `has_many :teams, through: :team_members`
- `has_many :messages, foreign_key: 'sender_id'`
- `has_many :notifications`

**Métodos Importantes**:
```ruby
def account_user(account)
  # Retorna AccountUser para a conta específica
end

def administrator?(account)
  # Verifica se é admin da conta
end

def agent?(account)
  # Verifica se é agent da conta
end

def active_account_user
  # AccountUser da conta ativa
end
```

---

### Conversation

**Propósito**: Núcleo do sistema, representa uma conversa entre contato e agentes.

**Atributos Principais**:
```ruby
# IDs e referências
display_id: integer             # ID amigável (gerado via trigger)
uuid: uuid                      # UUID único
identifier: string              # Identificador opcional

# Status e prioridade
status: enum                    # open, resolved, pending, snoozed
priority: enum                  # low, medium, high, urgent
snoozed_until: datetime         # Quando deve sair de snoozed

# Timestamps importantes
created_at: datetime
last_activity_at: datetime      # Última atividade
first_reply_created_at: datetime # Primeira resposta do agente
waiting_since: datetime         # Aguardando resposta desde
agent_last_seen_at: datetime    # Última visualização do agente
assignee_last_seen_at: datetime # Última visualização do assignee
contact_last_seen_at: datetime  # Última visualização do contato

# Atributos flexíveis
additional_attributes: jsonb    # Atributos adicionais
custom_attributes: jsonb        # Atributos customizados
cached_label_list: text         # Cache de labels (CSV)
```

**Enums**:
```ruby
enum status: {
  open: 0,      # Conversa aberta
  resolved: 1,  # Resolvida
  pending: 2,   # Pendente (aguardando)
  snoozed: 3    # Adiada
}

enum priority: {
  low: 0,
  medium: 1,
  high: 2,
  urgent: 3
}
```

**Relacionamentos**:
- `belongs_to :account`
- `belongs_to :inbox`
- `belongs_to :contact`
- `belongs_to :contact_inbox`
- `belongs_to :assignee, class_name: 'User'` (opcional)
- `belongs_to :assignee_agent_bot, class_name: 'AgentBot'` (opcional)
- `belongs_to :team` (opcional)
- `has_many :messages`
- `has_many :attachments, through: :messages`
- `has_many :mentions`
- `has_many :conversation_participants`

**Concerns**:
- `Labelable`: Adiciona suporte a tags/labels
- `AssignmentHandler`: Lógica de atribuição
- `AutoAssignmentHandler`: Auto-atribuição
- `ActivityMessageHandler`: Messages de atividade
- `ConversationMuteHelpers`: Silenciar conversas

**Scopes**:
```ruby
Conversation.unassigned                    # Sem agente atribuído
Conversation.assigned                      # Com agente atribuído
Conversation.assigned_to(agent)           # Atribuídas a agente específico
Conversation.unattended                    # Não respondidas ou aguardando
Conversation.resolvable_not_waiting(mins) # Podem ser auto-resolvidas
Conversation.open                          # Status open
Conversation.resolved                      # Status resolved
Conversation.pending                       # Status pending
```

**Métodos Importantes**:
```ruby
def toggle_status
  # Alterna entre open e resolved
end

def toggle_priority(priority = nil)
  # Define ou remove prioridade
end

def unread_messages
  # Mensagens não lidas pelo agente
end

def unread_incoming_messages
  # Últimas 10 mensagens incoming não lidas
end

def can_reply?
  # Verifica se pode responder (window policy)
end

def notifiable_assignee_change?
  # Verifica se mudança de assignee deve notificar
end

def assigned_entity
  # Retorna assignee (User ou AgentBot)
end

def recent_messages
  # Últimas 5 mensagens de chat
end
```

**Validações**:
- `validates :account_id, presence: true`
- `validates :inbox_id, presence: true`
- `validates :contact_id, presence: true`
- `validates :uuid, uniqueness: true`
- `validates :additional_attributes, jsonb_attributes_length: true`
- `validates :custom_attributes, jsonb_attributes_length: true`

**Callbacks**:
```ruby
before_validation :validate_additional_attributes
before_validation :reset_agent_bot_when_assignee_present
before_save :ensure_snooze_until_reset
before_create :determine_conversation_status
before_create :ensure_waiting_since
after_update_commit :execute_after_update_commit_callbacks
after_create_commit :notify_conversation_creation
after_create_commit :load_attributes_created_by_db_triggers
```

**Database Triggers**:
```ruby
# Trigger para gerar display_id sequencial por account
trigger.before(:insert).for_each(:row) do
  "NEW.display_id := nextval('conv_dpid_seq_' || NEW.account_id);"
end
```

---

### Contact

**Propósito**: Representa um contato/cliente no sistema.

**Atributos Principais**:
```ruby
name: string                    # Nome do contato
email: string                   # Email
phone_number: string            # Telefone
identifier: string              # Identificador único
custom_attributes: jsonb        # Atributos customizados
additional_attributes: jsonb    # Atributos adicionais
last_activity_at: datetime      # Última atividade
blocked: boolean                # Bloqueado ou not
```

**Relacionamentos**:
- `belongs_to :account`
- `has_many :conversations`
- `has_many :contact_inboxes`
- `has_many :inboxes, through: :contact_inboxes`
- `has_many :messages`
- `has_one_attached :avatar`

**Métodos Importantes**:
```ruby
def get_source_id(inbox_id)
  # Retorna source_id para inbox específico
end

def push_event_data
  # Dados para eventos push
end

def webhook_data
  # Dados para webhooks
end
```

**Scopes**:
```ruby
Contact.resolved_contacts      # Contatos com conversas resolvidas
Contact.order_on_last_activity # Ordenados por última atividade
```

---

### Message

**Propósito**: Representa uma mensagem em uma conversa.

**Atributos Principais**:
```ruby
content: text                   # Conteúdo da mensagem
message_type: enum              # incoming, outgoing, activity, template
content_type: enum              # text, input_select, cards, form, article
content_attributes: jsonb       # Atributos do conteúdo (botões, cards, etc)
status: enum                    # sent, delivered, read, failed
source_id: string               # ID externo (do canal)
sender_type: string             # User, Contact, AgentBot
sender_id: integer              # ID do sender
private: boolean                # Nota privada ou mensagem pública
external_source_ids: jsonb      # IDs de fontes externas (para threading)
```

**Enums**:
```ruby
enum message_type: {
  incoming: 0,    # Mensagem do contato
  outgoing: 1,    # Mensagem do agente
  activity: 2,    # Atividade do sistema
  template: 3     # Template de mensagem
}

enum content_type: {
  text: 0,
  input_select: 1,
  cards: 2,
  form: 3,
  article: 4,
  incoming_email: 5,
  input_email: 6,
  input_csat: 7,
  sticker: 8,
  location: 9,
  story_mention: 10,
  integrations: 11
}

enum status: {
  sent: 0,
  delivered: 1,
  read: 2,
  failed: 3
}
```

**Relacionamentos**:
- `belongs_to :conversation`
- `belongs_to :account`
- `belongs_to :inbox`
- `belongs_to :sender, polymorphic: true`
- `has_many :attachments`
- `has_many_attached :files` (Active Storage)

**Scopes**:
```ruby
Message.incoming               # Mensagens do contato
Message.outgoing               # Mensagens do agente
Message.chat                   # Mensagens de chat (não activity)
Message.today                  # Mensagens de hoje
Message.unread_since(datetime) # Não lidas desde
```

**Métodos Importantes**:
```ruby
def email?
  # Verifica se é mensagem de email
end

def attachments_in_content
  # Extrai URLs de anexos do conteúdo
end

def webhook_data
  # Dados para webhooks
end
```

---

### Inbox

**Propósito**: Representa um canal de comunicação (WebWidget, WhatsApp, Email, etc).

**Atributos Principais**:
```ruby
name: string                    # Nome do inbox
channel_type: string            # Tipo de canal (polimórfico)
enable_auto_assignment: boolean # Auto-atribuição habilitada
enable_email_collect: boolean   # Coletar email do contato
greeting_enabled: boolean       # Mensagem de boas-vindas
greeting_message: text          # Texto da mensagem de boas-vindas
working_hours_enabled: boolean  # Respeitar horário de funcionamento
out_of_office_message: text    # Mensagem fora de horário
timezone: string                # Timezone do inbox
allow_messages_after_resolved: boolean # Permitir mensagens após resolver
```

**Relacionamentos**:
- `belongs_to :account`
- `belongs_to :channel, polymorphic: true`
- `has_many :conversations`
- `has_many :contacts, through: :contact_inboxes`
- `has_many :inbox_members`
- `has_many :members, through: :inbox_members, source: :user`
- `has_many :agent_bot_inboxes`
- `has_many :working_hours`
- `has_one_attached :avatar`

**Channel Types**:
- `Channel::WebWidget`: Widget web
- `Channel::Email`: Inbox de email
- `Channel::Whatsapp`: WhatsApp
- `Channel::FacebookPage`: Facebook/Messenger
- `Channel::Instagram`: Instagram
- `Channel::TwitterProfile`: Twitter
- `Channel::Telegram`: Telegram
- `Channel::Sms`: SMS (Twilio, etc)
- `Channel::Line`: Line
- `Channel::Api`: Canal via API

**Métodos Importantes**:
```ruby
def inbox_type
  # Retorna tipo amigável do canal
end

def webhook_url
  # URL do webhook para o canal
end

def callback_webhook_url
  # URL de callback
end

def active_bot?
  # Verifica se tem bot ativo
end

def web_widget?
  # Verifica se é web widget
end

def facebook?
  # Verifica se é Facebook
end
```

---

###Team

**Propósito**: Agrupa agentes em equipes.

**Atributos Principais**:
```ruby
name: string                    # Nome da equipe
description: text               # Descrição
allow_auto_assign: boolean      # Permite auto-atribuição
```

**Relacionamentos**:
- `belongs_to :account`
- `has_many :team_members`
- `has_many :users, through: :team_members`
- `has_many :conversations`
- `has_many :inbox_teams`
- `has_many :inboxes, through: :inbox_teams`

---

### Label

**Propósito**: Tags/etiquetas para organizar conversas e contatos.

**Atributos Principais**:
```ruby
title: string                   # Título do label
description: text               # Descrição
color: string                   # Cor (hex)
show_on_sidebar: boolean        # Exibir na sidebar
```

**Relacionamentos**:
- `belongs_to :account`
- Usado em conversas e contatos via ActsAsTaggableOn

---

### CannedResponse

**Propósito**: Respostas prontas para uso rápido.

**Atributos Principais**:
```ruby
short_code: string              # Atalho para usar (ex: /welcome)
content: text                   # Conteúdo da resposta
```

**Relacionamentos**:
- `belongs_to :account`

---

### Webhook

**Propósito**: Configuração de webhooks para eventos.

**Atributos Principais**:
```ruby
url: string                     # URL do webhook
webhook_type: enum              # account, inbox
subscriptions: array            # Eventos subscritos
```

**Eventos Disponíveis**:
- `conversation_created`
- `conversation_updated`
- `conversation_resolved`
- `message_created`
- `message_updated`
- `contact_created`
- `contact_updated`

---

## Padrões Comuns

### JSONB Attributes

Muitos models usam JSONB para flexibilidade:

```ruby
# custom_attributes (valores definidos pelo usuário)
{
  "company": "ACME Inc",
  "plan": "enterprise",
  "custom_field": "value"
}

# additional_attributes (valores do sistema)
{
  "browser": "Chrome",
  "browser_version": "120.0",
  "platform": "Windows",
  "referer": "https://example.com",
  "initiated_at": {
    "timestamp": "2024-01-01T00:00:00Z"
  }
}
```

### Store Accessors

Para acesso conveniente a chaves JSONB:

```ruby
store_accessor :settings, :auto_resolve_after, :auto_resolve_message
```

### Concerns

Models compartilham funcionalidades via concerns:

- `Labelable`: Suporte a labels/tags
- `Avatarable`: Suporte a avatares
- `Reportable`: Geração de relatórios
- `Featurable`: Feature flags
- `AssignmentHandler`: Lógica de atribuição

### Enums

Para campos com valores fixos e queries semânticas:

```ruby
enum status: { open: 0, resolved: 1, pending: 2 }

# Uso
conversation.open!
conversation.open?
Conversation.open
```

### Validações

Validações consistentes em todos os models:

```ruby
validates :account_id, presence: true
validates :email, format: { with: URI::MailTo::EMAIL_REGEXP }
validates :custom_attributes, jsonb_attributes_length: true
```

### Callbacks

Lifecycle hooks para lógica adicional:

```ruby
before_validation :normalize_email
before_create :generate_uuid
after_create_commit :send_notifications
after_update_commit :dispatch_events
```

### Scopes

Queries reutilizáveis e nomeadas:

```ruby
scope :unassigned, -> { where(assignee_id: nil) }
scope :assigned_to, ->(agent) { where(assignee_id: agent.id) }
scope :status, ->(status) { where(status: status) }
```

## Conclusão

Os models do Chatwoot seguem um design consistente e bem estruturado:
- Multi-tenancy via Account
- Relacionamentos claros e bem definidos
- JSONB para flexibilidade
- Concerns para reusabilidade
- Callbacks e validações robustas
- Scopes para queries semânticas
