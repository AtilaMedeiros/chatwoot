# Guia de APIs do Chatwoot

## Visão Geral

Chatwoot expõe múltiplas APIs REST para diferentes propósitos:

1. **Client API** (`/api/v1/accounts/{account_id}/*`): API para dashboard web
2. **Platform API** (`/platform/api/v1/*`): API para integrações externas
3. **Public API** (`/public/api/v1/*`): API pública (widgets, webhooks)
4. **Super Admin API** (`/super_admin/*`): Administração super admin

## Autenticação

### Client API

Autenticação via Devise (cookies de sessão) ou API access token.

**Header**:
```
api_access_token: YOUR_ACCESS_TOKEN
```

**Obter Token**:
```
POST /api/v1/profile
Body: { email, password }
Response: { user, access_token }
```

### Platform API

Autenticação via Platform App API key.

**Header**:
```
api_access_token: YOUR_PLATFORM_API_KEY
```

**Criar Platform App**:
```
POST /platform/api/v1/platform_apps
Headers: { api_access_token: USER_ACCESS_TOKEN }
Body: { name: "My App" }
Response: { id, name, access_token }
```

### Public API

APIs públicas geralmente não requerem autenticação ou usam tokens específicos do contexto (inbox, contact).

---

## Client API (`/api/v1/accounts/{account_id}`)

API usada pelo dashboard web, requer autenticação de usuário.

### Conversas

#### Listar Conversas

```http
GET /api/v1/accounts/{account_id}/conversations

Query Parameters:
  status: open | resolved | pending | snoozed (default: open, pode passar vários)
  assignee_type: me | unassigned | all (default: all)
  inbox_id: integer (filtrar por inbox)
  team_id: integer (filtrar por time)
  labels: array (filtrar por labels)
  page: integer (paginação, default: 1)
  sort_by: last_activity_at_asc | last_activity_at_desc | created_at_asc | created_at_desc

Response: {
  data: {
    meta: {
      mine_count: integer,
      unassigned_count: integer,
      all_count: integer,
      ...
    },
    payload: [
      {
        id: integer,
        display_id: integer,
        inbox_id: integer,
        contact_id: integer,
        status: string,
        priority: string,
        assignee: { ... },
        messages: [ ... ],
        account: { ... },
        inbox: { ... },
        ...
      }
    ]
  }
}
```

#### Detalhes de uma Conversa

```http
GET /api/v1/accounts/{account_id}/conversations/{id}

Response: {
  id: integer,
  display_id: integer,
  status: string,
  priority: string,
  messages: [...],
  contact: {...},
  inbox: {...},
  assignee: {...},
  labels: [...],
  ...
}
```

#### Criar Conversa

```http
POST /api/v1/accounts/{account_id}/conversations

Body: {
  source_id: string,
  inbox_id: integer,
  contact_id: integer,
  additional_attributes: object (opcional),
  custom_attributes: object (opcional),
  status: string (opcional, default: open),
  assignee_id: integer (opcional),
  team_id: integer (opcional),
  message: {
    content: string,
    content_type: string (opcional, default: text),
    content_attributes: object (opcional)
  }
}

Response: {
  id: integer,
  display_id: integer,
  ...
}
```

#### Atualizar Status

```http
POST /api/v1/accounts/{account_id}/conversations/{id}/toggle_status

Response: { status: string }
```

#### Atribuir Agente

```http
POST /api/v1/accounts/{account_id}/conversations/{id}/assignments

Body: {
  assignee_id: integer
}

Response: { conversation: {...}, assignee: {...} }
```

#### Atribuir Time

```http
POST /api/v1/accounts/{account_id}/conversations/{id}/assignments

Body: {
  team_id: integer
}

Response: { conversation: {...} }
```

#### Adicionar/Remover Labels

```http
POST /api/v1/accounts/{account_id}/conversations/{id}/labels

Body: {
  labels: ["label1", "label2"]
}

Response: { labels: [...] }
```

#### Snooze

```http
POST /api/v1/accounts/{account_id}/conversations/{id}/toggle_status

Body: {
  status: "snoozed",
  snoozed_until: "2024-01-01T10:00:00Z"
}

Response: { status: "snoozed", snoozed_until: "..." }
```

#### Mute/Unmute

```http
POST /api/v1/accounts/{account_id}/conversations/{id}/mute

Response: { muted: boolean }
```

#### Transcript

```http
POST /api/v1/accounts/{account_id}/conversations/{id}/transcript

Body: {
  email: string
}

Response: { message: "Transcript sent successfully" }
```

---

### Mensações

#### Listar Mensagens de uma Conversa

```http
GET /api/v1/accounts/{account_id}/conversations/{conversation_id}/messages

Query Parameters:
  before: integer (message_id para paginação)

Response: {
  meta: {
    contact_last_seen_at: datetime,
    agent_last_seen_at: datetime
  },
  payload: [
    {
      id: integer,
      content: string,
      message_type: string,
      content_type: string,
      content_attributes: object,
      created_at: datetime,
      private: boolean,
      sender: {
        id: integer,
        name: string,
        type: string
      },
      attachments: [...]
    }
  ]
}
```

#### Criar Mensagem

```http
POST /api/v1/accounts/{account_id}/conversations/{conversation_id}/messages

Body: {
  content: string,
  message_type: string (default: outgoing),
  private: boolean (default: false),
  content_type: string (default: text),
  content_attributes: object (opcional),
  template_params: object (opcional, para templates),
  cc_emails: string (opcional, para emails),
  bcc_emails: string (opcional, para emails)
}

Multipart (com arquivos):
  content: string
  attachments[]: files
  message_type: string
  private: boolean

Response: {
  id: integer,
  content: string,
  ...
}
```

#### Deletar Mensagem

```http
DELETE /api/v1/accounts/{account_id}/conversations/{conversation_id}/messages/{id}

Response: 200 OK
```

---

### Contatos

#### Listar Contatos

```http
GET /api/v1/accounts/{account_id}/contacts

Query Parameters:
  sort: name | email | phone_number | created_at | last_activity_at
  page: integer

Response: {
  payload: [
    {
      id: integer,
      name: string,
      email: string,
      phone_number: string,
      thumbnail: string,
      custom_attributes: object,
      additional_attributes: object,
      identifier: string,
      last_activity_at: datetime,
      created_at: datetime
    }
  ]
}
```

#### Buscar Contatos

```http
GET /api/v1/accounts/{account_id}/contacts/search

Query Parameters:
  q: string (nome, email ou telefone)
  sort: name | -name | email | -email (default: name)
  page: integer

Response: { payload: [...] }
```

#### Criar Contato

```http
POST /api/v1/accounts/{account_id}/contacts

Body: {
  name: string (requerido),
  email: string (opcional),
  phone_number: string (opcional),
  identifier: string (opcional),
  custom_attributes: object (opcional),
  inbox_id: integer (requerido)
}

Response: {
  id: integer,
  name: string,
  ...
}
```

#### Atualizar Contato

```http
PATCH /api/v1/accounts/{account_id}/contacts/{id}

Body: {
  name: string,
  email: string,
  phone_number: string,
  custom_attributes: object
}

Response: {
  id: integer,
  name: string,
  ...
}
```

#### Deletar Contato

```http
DELETE /api/v1/accounts/{account_id}/contacts/{id}

Response: 200 OK
```

#### Filtrar Contatos

```http
POST /api/v1/accounts/{account_id}/contacts/filter

Body: {
  payload: [
    {
      attribute_key: string,
      attribute_model: string,
      filter_operator: string,
      values: array,
      query_operator: "and" | "or"
    }
  ]
}

Operadores:
  - equal_to
  - not_equal_to
  - contains
  - does_not_contain
  - is_present
  - is_not_present
  - is_greater_than
  - is_less_than
  - days_before

Response: {
  payload: [...],
  count: integer
}
```

---

### Inboxes

#### Listar Inboxes

```http
GET /api/v1/accounts/{account_id}/inboxes

Response: {
  payload: [
    {
      id: integer,
      name: string,
      channel_type: string,
      avatar_url: string,
      channel: {
        id: integer,
        type: string,
        ...
      },
      ...
    }
  ]
}
```

#### Detalhes de um Inbox

```http
GET /api/v1/accounts/{account_id}/inboxes/{id}

Response: {
  id: integer,
  name: string,
  channel_type: string,
  ...
}
```

#### Criar Inbox

Varia por tipo de canal. Exemplo para Web Widget:

```http
POST /api/v1/accounts/{account_id}/inboxes

Body: {
  name: string,
  channel: {
    type: "web_widget",
    website_url: string,
    widget_color: string,
    welcome_title: string,
    welcome_tagline: string,
    ...
  }
}

Response: {
  id: integer,
  name: string,
  ...
}
```

#### Atualizar Inbox

```http
PATCH /api/v1/accounts/{account_id}/inboxes/{id}

Body: {
  name: string,
  enable_auto_assignment: boolean,
  greeting_enabled: boolean,
  greeting_message: string,
  ...
}

Response: {
  id: integer,
  ...
}
```

#### Listar Agentes de um Inbox

```http
GET /api/v1/accounts/{account_id}/inbox_members/{inbox_id}

Response: {
  payload: [
    {
      id: integer,
      name: string,
      email: string,
      role: string,
      ...
    }
  ]
}
```

#### Adicionar Agente ao Inbox

```http
POST /api/v1/accounts/{account_id}/inbox_members

Body: {
  inbox_id: integer,
  user_ids: [integer, ...]
}

Response: {
  payload: [...]
}
```

---

### Times

#### Listar Times

```http
GET /api/v1/accounts/{account_id}/teams

Response: {
  payload: [
    {
      id: integer,
      name: string,
      description: string,
      allow_auto_assign: boolean,
      ...
    }
  ]
}
```

#### Criar Time

```http
POST /api/v1/accounts/{account_id}/teams

Body: {
  name: string,
  description: string,
  allow_auto_assign: boolean
}

Response: {
  id: integer,
  ...
}
```

#### Adicionar Membros ao Time

```http
POST /api/v1/accounts/{account_id}/teams/{id}/team_members

Body: {
  user_ids: [integer, ...]
}

Response: {
  users: [...]
}
```

---

### Respostas Prontas (Canned Responses)

#### Listar

```http
GET /api/v1/accounts/{account_id}/canned_responses

Response: {
  payload: [
    {
      id: integer,
      short_code: string,
      content: string
    }
  ]
}
```

#### Criar

```http
POST /api/v1/accounts/{account_id}/canned_responses

Body: {
  short_code: string,
  content: string
}

Response: {
  id: integer,
  ...
}
```

---

### Labels

#### Listar Labels

```http
GET /api/v1/accounts/{account_id}/labels

Response: {
  payload: [
    {
      id: integer,
      title: string,
      description: string,
      color: string,
      show_on_sidebar: boolean
    }
  ]
}
```

#### Criar Label

```http
POST /api/v1/accounts/{account_id}/labels

Body: {
  title: string,
  description: string,
  color: string,
  show_on_sidebar: boolean
}

Response: {
  id: integer,
  ...
}
```

---

### Webhooks

#### Listar Webhooks

```http
GET /api/v1/accounts/{account_id}/webhooks

Response: {
  payload: [
    {
      id: integer,
      url: string,
      webhook_type: string,
      subscriptions: [...]
    }
  ]
}
```

#### Criar Webhook

```http
POST /api/v1/accounts/{account_id}/webhooks

Body: {
  url: string,
  subscriptions: [
    "conversation_created",
    "message_created",
    ...
  ]
}

Response: {
  id: integer,
  ...
}
```

---

### Reports

#### Relatório de Conversas

```http
GET /api/v1/accounts/{account_id}/reports/conversations

Query Parameters:
  metric: conversations_count | incoming_messages_count | outgoing_messages_count | 
          avg_first_response_time | avg_resolution_time | resolutions_count
  type: account | agent | inbox | label | team
  id: integer (quando type != account)
  since: datetime
  until: datetime
  group_by: day | week | month | year

Response: {
  data: [
    {
      timestamp: integer,
      value: number
    }
  ]
}
```

#### Relatório de Agentes

```http
GET /api/v1/accounts/{account_id}/reports/agents

Query Parameters:
  since: datetime
  until: datetime

Response: [
  {
    id: integer,
    name: string,
    email: string,
    thumbnail: string,
    availability: string,
    metric: {
      conversations_count: integer,
      incoming_messages_count: integer,
      outgoing_messages_count: integer,
      avg_first_response_time: number,
      avg_resolution_time: number,
      resolutions_count: integer
    }
  }
]
```

---

## Platform API (`/platform/api/v1`)

API para integrações externas, requer Platform App API key.

### Conversas

Endpoints similares à Client API mas com escopo de Platform App:

```http
GET /platform/api/v1/accounts/{account_id}/conversations
POST /platform/api/v1/accounts/{account_id}/conversations
GET /platform/api/v1/accounts/{account_id}/conversations/{id}
POST /platform/api/v1/accounts/{account_id}/conversations/{id}/toggle_status
```

### Contatos

```http
GET /platform/api/v1/accounts/{account_id}/contacts
POST /platform/api/v1/accounts/{account_id}/contacts
GET /platform/api/v1/accounts/{account_id}/contacts/{id}
PATCH /platform/api/v1/accounts/{account_id}/contacts/{id}
```

### Mensagens

```http
GET /platform/api/v1/accounts/{account_id}/conversations/{conversation_id}/messages
POST /platform/api/v1/accounts/{account_id}/conversations/{conversation_id}/messages
```

---

## Public API (`/public/api/v1`)

API pública usada por widgets e integrações.

### Contatos (Widget)

#### Criar/Atualizar Contato

```http
POST /public/api/v1/inboxes/{inbox_identifier}/contacts

Headers:
  X-Auth-Token: CONTACT_TOKEN (opcional, para update)

Body: {
  name: string,
  email: string,
  phone_number: string,
  custom_attributes: object
}

Response: {
  id: integer,
  pubsub_token: string,
  contact_inboxes: [...]
}
```

### Conversas (Widget)

#### Listar Conversas do Contato

```http
GET /public/api/v1/inboxes/{inbox_identifier}/contacts/{contact_identifier}/conversations

Headers:
  X-Auth-Token: CONTACT_TOKEN

Response: {
  payload: [...]
}
```

#### Criar Conversa

```http
POST /public/api/v1/inboxes/{inbox_identifier}/contacts/{contact_identifier}/conversations

Headers:
  X-Auth-Token: CONTACT_TOKEN

Body: {
  custom_attributes: object
}

Response: {
  id: integer,
  ...
}
```

### Mensagens (Widget)

#### Listar Mensagens

```http
GET /public/api/v1/inboxes/{inbox_identifier}/contacts/{contact_identifier}/conversations/{conversation_id}/messages

Headers:
  X-Auth-Token: CONTACT_TOKEN

Response: {
  messages: [...]
}
```

#### Criar Mensagem

```http
POST /public/api/v1/inboxes/{inbox_identifier}/contacts/{contact_identifier}/conversations/{conversation_id}/messages

Headers:
  X-Auth-Token: CONTACT_TOKEN

Body: {
  content: string,
  content_attributes: object (opcional)
}

Multipart (com arquivos):
  content: string
  attachments[]: files

Response: {
  id: integer,
  ...
}
```

#### Marcar como Lida

```http
POST /public/api/v1/inboxes/{inbox_identifier}/contacts/{contact_identifier}/conversations/{conversation_id}/update_last_seen

Headers:
  X-Auth-Token: CONTACT_TOKEN

Response: 200 OK
```

---

## Webhooks

Chatwoot envia webhooks para URLs configuradas quando eventos ocorrem.

### Formato do Payload

```json
{
  "id": "uuid",
  "event": "conversation_created",
  "account": {
    "id": 1,
    "name": "ACME Inc"
  },
  "inbox": {
    "id": 1,
    "name": "Website"
  },
  "conversation": {
    "id": 123,
    "display_id": 45,
    "status": "open",
    "priority": null,
    "additional_attributes": {},
    "custom_attributes": {},
    "inbox_id": 1,
    "contact": {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com"
    },
    "messages": [...]
  },
  "sender": {
    "id": 1,
    "name": "John Doe",
    "type": "contact"
  },
  "message": {
    "id": 1,
    "content": "Hello",
    "message_type": "incoming",
    "content_type": "text",
    "created_at": "2024-01-01T00:00:00Z"
  },
  "event_info": {
    "triggered_at": "2024-01-01T00:00:00Z"
  }
}
```

### Eventos Disponíveis

- `conversation_created`
- `conversation_updated`
- `conversation_opened`
- `conversation_resolved`
- `conversation_status_changed`
- `message_created`
- `message_updated`
- `contact_created`
- `contact_updated`

### Validação de Webhook

Webhooks incluem header de assinatura HMAC:

```
X-Chatwoot-Signature: HMAC_SHA256_SIGNATURE
```

Para validar:

```ruby
def valid_signature?(payload, signature, secret)
  computed = OpenSSL::HMAC.hexdigest('SHA256', secret, payload)
  ActiveSupport::SecurityUtils.secure_compare(computed, signature)
end
```

---

## Rate Limiting

Rate limits aplicados por IP ou API key:

- **Client API**: 100 requests/minuto por usuário
- **Platform API**: 100 requests/minuto por API key
- **Public API**: 20 requests/minuto por IP

Headers de resposta:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1609459200
```

---

## Paginação

APIs paginadas retornam meta informações:

```json
{
  "meta": {
    "current_page": 1,
    "next_page": 2,
    "prev_page": null,
    "total_pages": 10,
    "total_count": 95
  },
  "payload": [...]
}
```

Query parameter:

```
?page=2
```

---

## Códigos de Erro

```
200 OK                    # Sucesso
201 Created               # Recurso criado
204 No Content            # Sucesso sem corpo de resposta
400 Bad Request           # Request inválido
401 Unauthorized          # Autenticação necessária
403 Forbidden             # Sem permissão
404 Not Found             # Recurso não encontrado
422 Unprocessable Entity # Validação falhou
429 Too Many Requests     # Rate limit excedido
500 Internal Server Error # Erro no servidor
```

Formato de erro:

```json
{
  "error": "Error message",
  "message": "Detailed error description",
  "attributes": {
    "field_name": ["error1", "error2"]
  }
}
```

---

## Exemplos de Integração

### cURL

```bash
# Listar conversas
curl -X GET \
  'https://app.chatwoot.com/api/v1/accounts/1/conversations?status=open' \
  -H 'api_access_token: YOUR_TOKEN'

# Criar mensagem
curl -X POST \
  'https://app.chatwoot.com/api/v1/accounts/1/conversations/123/messages' \
  -H 'api_access_token: YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "content": "Hello from API!",
    "message_type": "outgoing"
  }'
```

### JavaScript

```javascript
const BASE_URL = 'https://app.chatwoot.com';
const API_TOKEN = 'your_token';
const ACCOUNT_ID = 1;

const headers = {
  'api_access_token': API_TOKEN,
  'Content-Type': 'application/json'
};

// Buscar conversas
async function getConversations() {
  const response = await fetch(
    `${BASE_URL}/api/v1/accounts/${ACCOUNT_ID}/conversations?status=open`,
    { headers }
  );
  return response.json();
}

// Enviar mensagem
async function sendMessage(conversationId, content) {
  const response = await fetch(
    `${BASE_URL}/api/v1/accounts/${ACCOUNT_ID}/conversations/${conversationId}/messages`,
    {
      method: 'POST',
      headers,
      body: JSON.stringify({
        content,
        message_type: 'outgoing'
      })
    }
  );
  return response.json();
}
```

### Python

```python
import requests

BASE_URL = 'https://app.chatwoot.com'
API_TOKEN = 'your_token'
ACCOUNT_ID = 1

headers = {
    'api_access_token': API_TOKEN,
    'Content-Type': 'application/json'
}

# Buscar conversas
def get_conversations():
    url = f'{BASE_URL}/api/v1/accounts/{ACCOUNT_ID}/conversations'
    params = {'status': 'open'}
    response = requests.get(url, headers=headers, params=params)
    return response.json()

# Enviar mensagem
def send_message(conversation_id, content):
    url = f'{BASE_URL}/api/v1/accounts/{ACCOUNT_ID}/conversations/{conversation_id}/messages'
    data = {
        'content': content,
        'message_type': 'outgoing'
    }
    response = requests.post(url, headers=headers, json=data)
    return response.json()
```

---

## Conclusão

As APIs do Chatwoot são bem estruturadas e consistentes:
- REST completo com verbos HTTP apropriados
- Autenticação via tokens
- Paginação e filtragem robustas
- Webhooks para eventos em tempo real
- Rate limiting para proteção
- Documentação OpenAPI disponível em `/swagger`
