# ADR-001: Service Object Pattern for Business Logic

**Status**: Aceito  
**Data**: 2026-02-02  
**Autores**: architect-specialist, chatwoot-specialist  
**Reviewers**: code-reviewer, backend-specialist  
**Phase**: Planning

## Contexto

O Chatwoot possui lógica de negócio complexa que precisa ser organizada de forma testável, reutilizável e manutenível. Exemplos incluem:

- Processamento de mensagens incoming de diferentes canais
- Atribuição automática de conversas
- Roteamento de conversas para times
- Processamento de webhooks
- Geração de relatórios

Atualmente, parte dessa lógica está espalhada entre:
- Models (callbacks, métodos de instância)
- Controllers (diretamente na ação)
- Jobs (misturando orquestração com lógica)

**Problema**: Código difícil de testar, duplicação, responsabilidades pouco claras.

## Decisão

**Adotar Service Objects como padrão para encapsular lógica de negócio complexa.**

Service Objects são classes Ruby simples (POROs - Plain Old Ruby Objects) que:
- Têm uma responsabilidade única e bem definida
- Recebem dependências via construtor (initialize)
- Expõem um método principal `perform` ou `call`
- Retornam resultado ou levantam exception

### Estrutura Padrão

```ruby
# app/services/my_service.rb
class MyService
  def initialize(param1:, param2:)
    @param1 = param1
    @param2 = param2
  end

  def perform
    # Lógica de negócio
    validate!
    process
    result
  end

  private

  def validate!
    raise ValidationError unless valid?
  end

  def process
    # Implementação
  end

  def valid?
    # Validações
  end
end

# Uso:
service = MyService.new(param1: value1, param2: value2)
result = service.perform
```

### Quando Usar

✅ **Use Service Objects para**:
- Lógica que envolve múltiplos models
- Operações complexas (> 5 linhas)
- Integração com APIs externas
- Processamento que requer transações
- Lógica que não pertence claramente a um model

❌ **Não use para**:
- Queries simples (use scopes)
- Operações de um único model sem lógica extra
- Callbacks simples

## Alternativas Consideradas

### Opção 1: Fat Models

**Descrição**: Colocar toda lógica nos models

**Prós**:
- Tudo relacionado ao model está junto
- Menos arquivos
- Rails way tradicional

**Contras**:
- Models ficam enormes e difíceis de manter
- Difícil testar lógica isoladamente
- Viola Single Responsibility Principle
- Callbacks podem ter efeitos colaterais inesperados

### Opção 2: Concerns

**Descrição**: Extrair lógica para concerns

**Prós**:
- Organiza código relacionado
- Reutilizável entre models

**Contras**:
- Ainda mistura responsabilidades
- Dificulta entender fluxo completo
- Concerns viram "dumping ground"

### Opção 3: Implementação direta em Controllers

**Descrição**: Lógica de negócio nos controllers

**Prós**:
- "Simples" (tudo no mesmo lugar)
- Menos arquivos

**Contras**:
- Controllers ficam enormes (fat controllers)
- Impossível reutilizar lógica
- Muito difícil de testar
- Viola MVC

### Opção 4: Query Objects

**Descrição**: Usar Query Objects para consultas complexas

**Prós**:
- Ótimo para queries complexas
- Testável
- Reutilizável

**Contras**:
- Limitado a queries (read operations)
- Não resolve o problema geral de lógica de negócio

## Decisão Escolhida

**Service Objects** (Opção principal) porque:

1. **Responsabilidade única**: Cada service faz uma coisa  
2. **Testabilidade**: Fácil de testar isoladamente  
3. **Reutilização**: Pode ser chamado de controllers, jobs, outros services  
4. **Clareza**: Nome descritivo deixa claro o que faz  
5. **Manutenibilidade**: Mudanças ficam localizadas  
6. **Transações**: Fácil envolver em transações quando necessário  

**Complementado por**:
- Query Objects para queries complexas (read-only)
- Concerns para código realmente compartilhado entre models
- Scopes para queries simples

## Consequências

### Positivas

✅ **Organização**:
- Código mais organizado e fácil de encontrar
- Responsabilidades claras

✅ **Testabilidade**:
- Services são POROs, fáceis de testar
- Não precisam carregar todo Rails para testes
- Mocks e stubs mais simples

✅ **Reutilização**:
- Services podem ser chamados de qualquer lugar
- Evita duplicação de lógica

✅ **Manutenibilidade**:
- Mudanças localizadas em um service
- Fácil entender o que o código faz

✅ **Composição**:
- Services podem chamar outros services
- Orquestração clara

### Negativas

❌ **Mais arquivos**:
- Mais arquivos no projeto
- Precisa navegação para entender fluxo completo

❌ **Overhead inicial**:
- Criar classe para lógica simples pode parecer overkill
- Time precisa aprender padrão

❌ **Padronização necessária**:
- Precisa estabelecer convenções (naming, estrutura)
- Sem convenções, vira caos

### Riscos e Mitigação

⚠️ **Risco**: Services muito grandes ou com múltiplas responsabilidades  
✅ **Mitigação**: Revisar periodicamente, quebrar services grandes

⚠️ **Risco**: Proliferação excessiva de services pequenos  
✅ **Mitigação**: Use bom senso, nem tudo precisa ser service

⚠️ **Risco**: Inconsistência de naming  
✅ **Mitigação**: Convenção: `VerbNounService` (ex: CreateConversationService)

## Implementação

### Convenções de Naming

- **Verbo + Noun + Service**: `CreateConversationService`, `SendMessageService`
- Localização: `app/services/`
- Organização por contexto: `app/services/conversations/`, `app/services/messages/`

### Estrutura

```ruby
class VerbNounService
  def initialize(required_param:, optional: nil)
    @required_param = required_param
    @optional = optional
  end

  def perform
    validate!
    execute
  end

  private

  def validate!
    # Validations
  end

  def execute
    # Implementation
  end
end
```

### Testing

```ruby
RSpec.describe MyService do
  describe '#perform' do
    let(:service) { described_class.new(param: value) }

    it 'does something' do
      result = service.perform
      expect(result).to eq(expected)
    end

    context 'when invalid' do
      it 'raises error' do
        expect { service.perform }.to raise_error(ValidationError)
      end
    end
  end
end
```

## Exemplos no Chatwoot

### Existentes (seguem este padrão)

- `Conversations::FilterService`
- `Messages::MessageBuilder`
- `Integrations::Slack::SendOnSlackService`

### A serem criados

- `ConversationFilterService` (ADR-001 context)
- `AutoAssignmentService`
- `ReportGeneratorService`

## Referências

- [Service Objects in Rails](https://www.toptal.com/ruby-on-rails/rails-service-objects-tutorial)
- [PORO Service Objects](https://www.rubyguides.com/2019/02/ruby-service-objects/)
- Chatwoot: `app/services/` (exemplos existentes)
- [ARQUITETURA.md](../docs/ARQUITETURA.md#services)

## Notas de Review

**code-reviewer**: Aprovado. Já seguimos parcialmente, formalizar é importante.  
**backend-specialist**: Concordo. Sugiro documentar naming conventions claramente.  
**chatwoot-specialist**: Alinhado com patterns atuais do Chatwoot.

## Status History

- 2026-02-02: Proposta criada
- 2026-02-02: Review completo, consenso alcançado
- 2026-02-02: **Aceito** como padrão oficial

---

**Supersedes**: N/A  
**Superseded by**: N/A  
**Related**: ADR-002 (Rate Limiting usa services)
