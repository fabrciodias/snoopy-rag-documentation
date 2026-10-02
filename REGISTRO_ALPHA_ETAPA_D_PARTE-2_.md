# Registro Alpha da Etapa D — Consolidação D.5–D.8

**Status:** CONSOLIDADO / PAUSA
**Etapa seguinte:** D.9 — Regras de implementação para o executor

---

# 1. Propósito deste registro

D.5–D.8 completam a transformação da arquitetura integrada da Alpha em um mapa operacional de implementação.

A sequência consolidada é:

```text
D.1 — Baseline e migração
D.2 — Planta do projeto
D.3 — Contratos e modelo
D.4 — Banco e persistência
        ↓
D.5 — API e comunicação
D.6 — Frontend e renderer
D.7 — Ordem de implementação
D.8 — Vertical slice e validação
        ↓
D.9 — Contrato com o executor
```

D.5–D.8 não criam uma nova arquitetura. Elas tornam implementáveis e verificáveis as decisões estabelecidas em A–C e D.1–D.4.

A regra continua sendo:

> **Comprimir o escopo, não os princípios.**

---

# 2. D.5 — Contratos de API e comunicação

## 2.1 Princípio central

A comunicação da Alpha é estruturada em:

```text
HTTP
+
Operation
+
SSE
```

A regra fundamental é:

> **SSE comunica mudanças; a API fornece o estado autoritativo.**

SSE não é fonte de verdade e não deve se transformar em uma segunda API de estado.

---

## 2.2 Categorias de comunicação

### Queries

Consultam estado:

```text
GET
```

### Commands

Solicitam mudanças ou execuções:

```text
POST
PUT
DELETE
```

### Operations

Comandos assíncronos recebem:

```text
operation_id
```

A Operation é a identidade pública da execução assíncrona.

### SSE

Comunica que algum recurso mudou e permite atualização reativa da interface.

---

## 2.3 Contratos principais

| Função                  | Método | Endpoint                                      | Natureza      |
| ----------------------- | ------ | --------------------------------------------- | ------------- |
| Iniciar investigação    | POST   | `/investigations/search`                      | command/async |
| Recuperar investigação  | GET    | `/investigations/{investigation_id}`          | query         |
| Recuperar operação      | GET    | `/operations/{operation_id}`                  | query         |
| Recuperar evidência     | GET    | `/evidence/{evidence_id}`                     | query         |
| Recuperar leitura/fonte | GET    | `/documents/{document_id}/reading` + contexto | query         |
| Listar corpus           | GET    | `/corpus/documents`                           | query         |
| Sincronizar corpus      | POST   | `/corpus/sync`                                | command/async |
| Atualizações            | SSE    | canal de eventos                              | notification  |

---

## 2.4 Iniciar investigação

```http
POST /investigations/search
```

Request conceitual:

```json
{
  "query": "...",
  "filters": {}
}
```

`query` representa a pergunta original do usuário.

Consultas derivadas — lexical, semantic, decomposed query, embeddings etc. — pertencem internamente ao Retrieval e não fazem parte do contrato público da investigação sem necessidade.

Resposta inicial:

```json
{
  "investigation_id": "...",
  "operation_id": "...",
  "status": "PENDING"
}
```

ou estado equivalente já iniciado.

A investigação possui identidade antes de os resultados chegarem.

---

## 2.5 Recuperar Operation

```http
GET /operations/{operation_id}
```

A resposta deve permitir conhecer:

* identidade;
* tipo;
* estado;
* alvo/referência;
* timestamps relevantes;
* erro, quando aplicável.

Estados:

```text
PENDING
PROCESSING
COMPLETED
FAILED
CANCELLED
```

O frontend não depende do SSE para descobrir o estado final.

---

## 2.6 Recuperar Investigation

```http
GET /investigations/{investigation_id}
```

A Investigation é o estado recuperável da experiência de pesquisa.

O contrato deve permitir reconstruir a superfície da investigação, incluindo conceitualmente:

```text
query
filters
status
results
evidence
synthesis
structured_response
```

Isso não obriga toda implementação futura a transportar sempre todo conteúdo pesado em uma única resposta. Otimizações de carregamento são possíveis desde que não destruam a capacidade semântica de reconstrução da investigação.

---

## 2.7 Recuperar Evidence

```http
GET /evidence/{evidence_id}
```

A evidência possui identidade própria e continua recuperável independentemente de o drawer estar aberto.

Informações mínimas:

```text
evidence_id
investigation_id
unit_id
document_id
location
content
context
provenance
```

A proveniência mínima permanece:

```text
Evidence
 ↓
RetrievalUnit
 ↓
Document
 ↓
Location
 ↓
Original Source
```

---

## 2.8 Reading / Source

Reading é uma **superfície da interface**, não uma entidade de domínio.

O contrato conceitual é:

```http
GET /documents/{document_id}/reading
```

com o contexto necessário para localizar a região relevante, como:

```text
document_id
location
evidence_id
```

A responsabilidade é fornecer a representação contextual necessária para leitura da fonte.

O fluxo permanece:

```text
Investigation
 ↓
Evidence
 ↓
Reading
 ↓
Return
 ↓
mesma Investigation
```

A rota do frontend não é autoridade sobre o estado da investigação.

---

## 2.9 Corpus

Listagem:

```http
GET /corpus/documents
```

Sincronização:

```http
POST /corpus/sync
```

A sincronização é assíncrona e retorna uma `operation_id`.

A execução efetiva ocorre através da infraestrutura de operações/queue/worker.

---

## 2.10 Erros

Taxonomia mínima:

```text
400 — requisição inválida
401 — não autenticado
403 — não autorizado
404 — recurso não encontrado
409 — conflito de estado
422 — conteúdo semanticamente inválido
500 — erro interno
503 — serviço temporariamente indisponível
```

Envelope comum:

```json
{
  "error": {
    "code": "...",
    "message": "...",
    "details": null
  }
}
```

O campo `code` permite tratamento programático.

`message` é destinado à interface.

`details` somente contém informação segura e necessária.

Não devem vazar:

* stack traces;
* caminhos locais;
* SQL;
* credenciais;
* chaves de serviço;
* detalhes internos de workers;
* informações equivalentes de infraestrutura.

---

## 2.11 Distinções de falha

A API preserva explicitamente:

```text
HTTP error
≠
Operation FAILED
≠
communication loss
```

Também:

```text
timeout
≠
FAILED
```

e:

```text
SSE disconnect
≠
FAILED
```

Se o frontend perder comunicação depois de receber um `operation_id`, deve consultar o estado real antes de assumir qualquer resultado.

---

## 2.12 SSE

O SSE é um canal de notificação.

Eventos conceituais:

```text
operation.created
operation.processing
operation.completed
operation.failed

document.updated
corpus.updated

investigation.updated
```

O payload deve conter identidade suficiente para localizar a mudança.

Exemplo:

```json
{
  "operation_id": "..."
}
```

ou:

```json
{
  "investigation_id": "..."
}
```

O evento pode carregar informação mínima de estado quando útil, mas isso não transforma seu payload na autoridade definitiva.

Fluxo:

```text
SSE
 ↓
notificação
 ↓
GET / recurso correspondente
 ↓
estado autoritativo
```

---

## 2.13 Reconexão

Se o SSE cair:

```text
SSE disconnected
 ↓
reconnect
 ↓
revalidate
 ↓
GET
 ↓
estado confirmado
```

Não é necessário reconstruir todos os eventos perdidos.

O estado persistido permite recuperar a situação atual.

O frontend nunca deve interpretar a simples perda de SSE como `FAILED` ou `CANCELLED`.

---

## 2.14 Streaming de síntese

Streaming de tokens por SSE não é requisito arquitetural da Alpha.

A síntese pode ocorrer:

```text
Retrieval
 ↓
Evidence
 ↓
Context
 ↓
Synthesis
 ↓
StructuredResponse
 ↓
persist/reconstruct
```

e então:

```text
SSE
 ↓
investigation.updated
```

O frontend recupera a investigação estruturada.

Streaming estruturado pode existir futuramente como otimização, sem alterar o fundamento da arquitetura.

---

## 2.15 Idempotência

Commands assíncronos que possam ser repetidos e gerar efeitos duplicados devem possuir mecanismo de idempotência.

Especialmente:

```text
POST /investigations/search
POST /corpus/sync
```

Conceitualmente:

```http
Idempotency-Key: <client-generated-key>
```

A mesma solicitação com a mesma chave representa a mesma operação lógica.

Isso protege contra cenários como:

```text
POST
 ↓
Operation A criada
 ↓
resposta perdida
 ↓
retry
 ↓
POST novamente
```

Retry explícito de uma operação que terminou realmente em `FAILED` cria uma nova `operation_id`.

---

## 2.16 Autorização

A API segue:

```text
request
 ↓
authentication
 ↓
authorization
 ↓
application
 ↓
data
```

O backend é responsável por autorizar acesso a:

* investigação;
* evidência;
* documento;
* corpus;
* operação.

RLS permanece como defesa adicional, não como substituto da autorização de domínio.

O frontend não decide a autorização.

---

## 2.17 Fronteira de conhecimento do frontend

O frontend não conhece diretamente:

```text
PostgreSQL
pgvector
PostgreSQL FTS
embeddings
reranker
queue
worker
Google Drive credentials
LLM internals
DocumentRepresentation interna
segmentation algorithm
fusion algorithm
```

Conhece os contratos necessários para:

```text
Investigation
Evidence
Document/Reading
Corpus
Operation
StructuredResponse
```

---

# 3. D.6 — Planta do frontend

## 3.1 Tecnologia

A Alpha utiliza:

```text
Vue 3 + TypeScript
```

Comunicação:

```text
HTTP + SSE
```

---

## 3.2 Superfícies

As superfícies principais são:

```text
Investigation
Evidence Drawer
Reading / Source
Corpus
Operations
```

Investigation permanece como superfície central.

Evidence é expansão contextual.

Reading é superfície própria.

Corpus é transversal.

Operations representam execução assíncrona.

---

## 3.3 Estrutura física

```text
frontend/
│
├── app/
│   ├── App.vue
│   ├── router.ts
│   └── layouts/
│
├── features/
│   ├── investigation/
│   │   ├── components/
│   │   └── composables/
│   ├── evidence/
│   │   ├── components/
│   │   └── composables/
│   ├── reading/
│   │   ├── components/
│   │   └── composables/
│   ├── corpus/
│   │   ├── components/
│   │   └── composables/
│   └── operations/
│       ├── components/
│       └── composables/
│
├── components/
│
├── renderer/
│   ├── ResponseRenderer
│   └── blocks/
│
├── infrastructure/
│   ├── api/
│   └── sse/
│
├── state/
│   ├── investigation/
│   ├── corpus/
│   ├── operations/
│   └── ui/
│
└── main.ts
```

A organização é por responsabilidade, não por conveniência tecnológica.

---

## 3.4 App e Router

`app/` realiza a composição global:

* inicialização;
* router;
* layouts;
* providers necessários;
* montagem da aplicação.

`App.vue` não deve concentrar lógica de negócio.

Rotas conceituais:

```text
/                         → Investigation
/investigation/:id        → Investigation
/document/:id             → Reading
/corpus                   → Corpus
```

Evidence Drawer não precisa ser uma rota independente.

O router representa navegação.

```text
URL
 ↓
navegação
```

Enquanto:

```text
investigation_id
 ↓
estado da investigação
```

O estado é recuperado pelo backend.

---

## 3.5 Features

### Investigation

Centraliza:

```text
Query
Results
Synthesis
Evidence Drawer
```

Preserva a investigação original, filtros, resultados, evidências relevantes, síntese, StructuredResponse e estados associados.

### Evidence

Materializa a apresentação/interação da evidência.

O Drawer continua funcionalmente ligado à Investigation.

### Reading

Representa a superfície de leitura da fonte.

Não duplica o estado completo da Investigation.

### Corpus

Representa documentos, estados, sincronização, operações e erros.

Não acessa diretamente infraestrutura de Drive, banco, queue ou worker.

### Operations

Representa operações simultâneas e seus estados.

Não existe um `isLoading` global como autoridade.

---

## 3.6 Components

`components/` contém apenas componentes genuinamente transversais.

Exemplos:

```text
Button
Input
Dialog
Drawer
Tabs
StatusIndicator
LoadingState
ErrorState
```

Componentes específicos de uma feature permanecem dentro da própria feature.

---

## 3.7 Renderer

O Renderer constitui uma fronteira própria:

```text
StructuredResponse
        ↓
Presentation Renderer
        ↓
Vue Components / HTML
```

Ele conhece o contrato da `StructuredResponse`, não a lógica de Retrieval.

Referências de evidência possuem identidade semântica:

```text
evidence_id
```

e podem abrir o Evidence Drawer.

Markdown não é o contrato estrutural primário.

---

## 3.8 Infrastructure

```text
infrastructure/
├── api/
└── sse/
```

A API client centraliza as chamadas.

As features não constroem URLs manualmente espalhadas pelo projeto.

O módulo SSE:

* abre conexão;
* recebe eventos;
* trata reconexão;
* detecta perda de comunicação;
* encaminha notificações.

Não determina estado de domínio.

---

## 3.9 State

Não haverá uma store global monolítica.

Domínios:

```text
INVESTIGATION
CORPUS
OPERATIONS
UI
```

Estrutura:

```text
state/
├── investigation/
├── corpus/
├── operations/
└── ui/
```

O mecanismo físico — Pinia, composables, reactive state ou combinação — não foi congelado.

O que foi congelado é a separação dos domínios.

---

## 3.10 Distinções de estado

Devem permanecer separados:

```text
Domain state
≠
Frontend state
≠
Cache
≠
UI state
```

### Investigation State

Deve sobreviver:

```text
Investigation
 ↓
Evidence Drawer
 ↓
Reading
 ↓
Return
```

Conceitualmente:

```text
investigation_id
query
filters
results
evidence
synthesis
structured_response
status
```

### Corpus State

Representa o estado que a interface está visualizando:

```text
documents
document states
sync operation
errors
```

### Operations State

Cada operação possui identidade própria.

Exemplo:

```text
op-123 → PROCESSING
op-456 → COMPLETED
op-789 → FAILED
```

### UI State

Contém apenas estado visual:

```text
drawer
tabs
menus
focus
theme
visual preferences
```

Não deve ser autoridade sobre estados de domínio.

---

## 3.11 Composition API

Composition API pode organizar lógica reutilizável das features.

Exemplos conceituais:

```text
useInvestigation()
useEvidence()
useReading()
useCorpus()
useOperations()
```

Esses composables não devem virar “mini-services universais” nem absorver responsabilidades do backend.

---

## 3.12 Direção das dependências

```text
Feature
   ↓
State / composables
   ↓
Infrastructure
   ↓
API / SSE
```

E:

```text
Renderer
   ↓
StructuredResponse contract
```

Enquanto:

```text
Components
   ↓
UI primitives
```

Nenhuma feature acessa diretamente banco, pgvector, Drive, worker, queue ou LLM.

---

## 3.13 O que Vue pode e não pode decidir

Vue pode controlar:

* renderização;
* interação;
* estado visual;
* navegação;
* composição;
* cache local;
* atualização visual.

Vue não decide:

* se documento é `ACTIVE`;
* se Operation terminou;
* quem possui acesso;
* quais unidades são relevantes;
* qual evidência é metodologicamente válida;
* como Retrieval funciona;
* qual é a fonte original.

---

# 4. D.7 — Sequência de implementação

D.7 transforma C.10 em uma ordem operacional para execução.

A regra central é:

> **Implementar o mínimo que materializa a arquitetura nova, não o mínimo que faz uma demonstração parecer nova.**

A ordem é arquiteturalmente orientada e pode conter paralelismo quando as dependências permitirem.

---

## Fase 0 — Baseline, schema e segurança

### Objetivo

Estabelecer o estado real do V2 antes das alterações estruturais.

### Inclui

* baseline do repositório;
* inventário real do banco;
* tabelas;
* relações;
* índices;
* RPCs;
* RLS/policies;
* worker;
* sincronização;
* autenticação;
* frontend;
* credenciais expostas.

### Reaproveitar

O V2 como substrato de migração.

### Não fazer

Assumir que o modelo V2 é o modelo V3.

### Conclusão

O executor consegue explicar o estado inicial e classificar o que será preservado, adaptado, substituído, removido ou validado.

---

## Fase 1 — Entidades, estados e Operation

### Objetivo

Materializar o núcleo lógico da V3.

### Entidades

```text
Document
DocumentRepresentation
RetrievalUnit
Investigation
RetrievalResult
Evidence
Operation
StructuredResponse
```

### Preservar

Identidades e dados V2 necessários.

### Não preservar diretamente

`jobs` como modelo definitivo de execução.

### Conclusão

É possível identificar documentos, criar/acompanhar Operations e manter separadas identidade, estado e execução.

---

## Fase 2 — DocumentRepresentation

### Objetivo

Substituir Markdown como representação canônica do documento.

### Fluxo

```text
document
 ↓
processor
 ↓
DocumentRepresentation
```

### Deve preservar

* páginas;
* blocos;
* texto;
* ordem;
* localização;
* estrutura básica.

### Reaproveitar

Extrator/PyMuPDF e partes compatíveis do processamento V2.

### Não preservar

A cadeia:

```text
PDF
 ↓
Markdown
 ↓
chunk
```

como arquitetura documental.

### Conclusão

Existe uma representação documental válida e navegável.

OCR permanece pós-Alpha.

---

## Fase 3 — Segmentation → RetrievalUnit

### Objetivo

Criar as unidades reais de recuperação da V3.

```text
Representation
 ↓
Segmentation
 ↓
RetrievalUnit[]
```

A segmentação é governada pelo Snoopy.

> **Algoritmos externos propõem; o Snoopy governa.**

### Reaproveitar

`chunker.py` apenas como matéria-prima/algoritmo potencialmente adaptável.

### Não preservar

Chunk como conceito arquitetural.

### Conclusão

Cada unidade possui identidade e pode ser rastreada:

```text
unit
 ↓
representation
 ↓
document
 ↓
location
```

---

## Fase 4 — Indexação e publicação → ACTIVE

### Objetivo

Tornar o documento recuperável somente depois de processamento válido.

```text
PROCESSING
 ↓
Representation
 ↓
Validation
 ↓
RetrievalUnits
 ↓
Embedding + FTS
 ↓
Validation
 ↓
ACTIVE
```

### Reaproveitar

* embeddings;
* Gemini;
* pgvector;
* PostgreSQL;
* worker compatível.

### Não preservar

Publicação parcial.

### Conclusão

Somente documentos validamente indexados entram no Retrieval.

---

## Fase 5 — Retrieval híbrido

### Objetivo

Materializar a cadeia de recuperação V3.

```text
query + filters
 ↓
ACTIVE corpus
 ↓
lexical + semantic
 ↓
fusion
 ↓
dedup
 ↓
contextualization
 ↓
reranking
 ↓
selection
 ↓
RetrievalResult[]
```

### Reaproveitar

Partes da busca semântica V2, embeddings, pgvector e `match_chunks()` quando compatíveis.

### Não preservar

`match_chunks()` como se fosse o Retrieval inteiro.

### Conclusão

A investigação produz resultados identificáveis:

```text
result_id
investigation_id
unit_id
rank
score
```

e:

```text
RetrievalResult ≠ Evidence
```

---

## Fase 6 — Evidence + Provenance

### Objetivo

Transformar resultados em evidências navegáveis e rastreáveis.

```text
RetrievalResult
 ↓
Evidence
```

Com:

```text
evidence_id
investigation_id
unit_id
document_id
location
content
context
provenance
```

### Não preservar

`[TRECHO X]` como identidade semântica.

### Conclusão

É possível reconstruir:

```text
Evidence
 ↓
Unit
 ↓
Document
 ↓
Location
 ↓
Source
```

---

## Fase 7 — Synthesis → StructuredResponse

### Objetivo

Separar claramente fonte, evidência, síntese e apresentação.

```text
Evidence
 ↓
Context
 ↓
Synthesis
 ↓
StructuredResponse
```

### Reaproveitar

Lógica de síntese e integração Gemini quando compatíveis.

### Não preservar

LLM → Markdown → UI como caminho estrutural primário.

### Conclusão

A investigação produz uma `StructuredResponse` recuperável/reconstruível e com referências semânticas a evidências.

---

## Fase 8 — Vue 3 + TypeScript + Renderer

### Objetivo

Materializar os contratos no frontend.

### Componentes

* Vue 3;
* TypeScript;
* Router;
* features;
* state;
* API;
* SSE;
* Renderer.

### Reaproveitar

Comportamentos do V2:

* investigação;
* resultados;
* leitura;
* evidência.

### Não preservar

Frontend Vanilla JS híbrido como arquitetura permanente.

### Conclusão

O frontend consegue:

```text
iniciar investigação
 ↓
acompanhar Operation
 ↓
recuperar Investigation
 ↓
renderizar StructuredResponse
 ↓
abrir Evidence Drawer
 ↓
abrir Reading
 ↓
retornar
```

---

## Fase 9 — Investigation → Evidence → Reading

### Objetivo

Integrar as superfícies definidas em B.

```text
Investigation
 ↓
Results
 ↓
Evidence Drawer
 ↓
Evidence
 ↓
Reading / Source
 ↓
Return
 ↓
same Investigation
```

### Conclusão

A navegação não destrói o contexto da investigação.

---

## Fase 10 — SSE + recuperação

### Objetivo

Integrar operações assíncronas sem transformar SSE em autoridade.

```text
Command
 ↓
operation_id
 ↓
SSE event
 ↓
GET / revalidation
 ↓
confirmed state
 ↓
UI
```

### Reaproveitar

Infraestrutura SSE do V2 quando compatível.

### Não preservar

Evento SSE como estado definitivo.

### Conclusão

A interface sobrevive a:

* perda de SSE;
* reconexão;
* operação concluída durante desconexão;
* timeout;
* recuperação por GET.

---

## Fase 11 — Vertical slice

### Objetivo

Provar a arquitetura integrada.

Fluxo obrigatório:

```text
Drive
 ↓
Document
 ↓
DocumentRepresentation
 ↓
RetrievalUnit[]
 ↓
indexes
 ↓
ACTIVE
 ↓
query
 ↓
hybrid retrieval
 ↓
Results
 ↓
Evidence
 ↓
Synthesis
 ↓
StructuredResponse
 ↓
Vue Renderer
 ↓
Investigation
 ↓
Evidence Drawer
 ↓
Reading
 ↓
Return
```

### Conclusão

O fluxo funciona com documento e investigação reais atravessando as fronteiras V3.

Módulos isoladamente funcionais não bastam.

---

## Fase 12 — Testes e fechamento

### Objetivo

Validar invariantes e encerrar a Alpha.

Devem ser verificados, no mínimo:

* documento válido → `ACTIVE`;
* documento inválido → não `ACTIVE`;
* falha → `FAILED`;
* remoção → `REMOVED`;
* publicação parcial não pesquisável;
* unidades identificáveis;
* apenas `ACTIVE` no Retrieval;
* filtros;
* lexical + semantic;
* `RetrievalResult ≠ Evidence`;
* proveniência reconstruível;
* Operations concorrentes independentes;
* retry de operação falhada gera nova Operation;
* timeout não equivale a falha;
* cancelamento só é confirmado pelo backend;
* perda/reconexão de SSE;
* GET como autoridade;
* investigação preservada durante navegação;
* Evidence Drawer funcional;
* Reading funcional;
* Renderer funcional;
* estados visualmente distinguíveis;
* conteúdo não confiável não executado como HTML arbitrário;
* autorização no backend;
* ausência de vazamento de segredos/infraestrutura.

---

# 5. D.8 — Vertical Slice e validação

D.8 é o **teste de realidade** da Alpha.

A pergunta não é:

> “Todos os módulos foram criados?”

A pergunta é:

> **“Um documento real consegue atravessar a arquitetura inteira e terminar como uma investigação real, navegável e recuperável?”**

---

## 5.1 Documento real

O teste começa com documento real proveniente do corpus.

Deve ser possível reconstruir:

```text
drive_file_id
 ↓
document_id
 ↓
operation_id
 ↓
representation_id
```

Dados previamente processados ou mocks que escondam o pipeline não constituem vertical slice completo.

---

## 5.2 Representation

A representação deve preservar:

* páginas;
* blocos;
* texto;
* ordem de leitura;
* localização;
* estrutura básica.

O teste fundamental é:

> **A representação permite localizar posteriormente aquilo que foi recuperado?**

---

## 5.3 Segmentation

Deve produzir `RetrievalUnit[]`.

Cada unidade deve ser rastreável:

```text
unit_id
 ↓
representation_id
 ↓
document_id
 ↓
location
```

Embedding não pode ser confundido com identidade da unidade.

---

## 5.4 Indexação e publicação

Devem existir caminhos válidos e inválidos.

### Válido

```text
processing
→ indexing
→ validation
→ ACTIVE
→ searchable
```

### Inválido

```text
processing
→ failure/inconsistency
→ FAILED ou REJECTED
→ não pesquisável
```

A Alpha não pode depender apenas de testes em que tudo dá certo.

---

## 5.5 Investigation real

A investigação deve preservar:

```text
original_query
filters
investigation_id
operation_id
```

A operação de busca não pode ser apenas uma lista temporária retornada pelo backend.

---

## 5.6 Retrieval híbrido

A investigação deve atravessar a cadeia real:

```text
filters
 ↓
ACTIVE corpus
 ↓
lexical + semantic
 ↓
fusion
 ↓
dedup
 ↓
contextualization
 ↓
reranking
 ↓
selection
 ↓
RetrievalResult[]
```

A D.8 não exige demonstrar cientificamente a superioridade do algoritmo.

Exige demonstrar que a arquitetura híbrida existe e é atravessada pelo fluxo real.

---

## 5.7 Evidence e provenance

Pelo menos um resultado deve chegar a uma Evidence identificável.

Deve ser possível perguntar:

> **“Recebendo apenas o `evidence_id`, consigo descobrir de onde essa evidência veio?”**

O caminho deve ser reconstruível sem depender de o frontend ainda estar aberto na investigação original.

---

## 5.8 Synthesis

A síntese deve consumir:

```text
Evidence
 ↓
Context
```

e continuar sendo uma derivação.

Portanto:

```text
Source
≠
Evidence
≠
Synthesis
```

O LLM não se torna uma nova fonte de verdade.

---

## 5.9 StructuredResponse

A resposta deve ser estruturada semanticamente:

```text
StructuredResponse
 ↓
Presentation Renderer
 ↓
Vue
```

A interface não deve depender de Markdown como linguagem estrutural primária.

Referências de evidência devem utilizar identidade semântica estável.

---

## 5.10 Investigation → Evidence → Reading → Return

O fluxo completo da experiência deve funcionar:

```text
Investigation
 ↓
Results
 ↓
Evidence Drawer
 ↓
Evidence
 ↓
Reading / Source
 ↓
Return
 ↓
same Investigation
```

Ao retornar:

* query permanece;
* resultados permanecem;
* contexto permanece;
* evidência continua identificável.

---

## 5.11 Estados assíncronos

Devem ser demonstráveis:

```text
PENDING
PROCESSING
COMPLETED
FAILED
CANCELLED
```

para Operations, quando aplicável.

E:

```text
PENDING
PROCESSING
ACTIVE
FAILED
REJECTED
REMOVED
```

para Documents.

Situações diferentes não podem ser mascaradas por um único `loading` genérico.

---

## 5.12 Perda de comunicação

Deve ser possível testar:

```text
operation PROCESSING
 ↓
SSE disconnect
 ↓
reconnect
 ↓
GET operation
 ↓
estado real
```

A perda de SSE não pode ser apresentada como falha da Operation.

Esse teste materializa a regra:

> **SSE comunica mudança; GET recupera estado.**

---

## 5.13 Idempotência e retry

A validação deve cobrir repetição acidental de commands assíncronos.

Uma solicitação repetida com a mesma chave de idempotência não deve produzir efeitos duplicados.

Já um retry explícito de uma Operation realmente `FAILED` produz nova:

```text
operation_id
```

---

## 5.14 Segurança

A Alpha deve demonstrar os invariantes fundamentais:

* conteúdo documental/LLM é tratado como dado não confiável;
* não há injeção arbitrária de HTML;
* autorização ocorre no backend;
* RLS é defesa adicional;
* respostas não vazam infraestrutura ou segredos.

Não é uma auditoria de segurança completa.

---

## 5.15 Publicação coerente

A Alpha deve demonstrar que:

```text
Document = PROCESSING
```

não vira:

```text
Document = ACTIVE
```

antes de existir estado recuperável e validamente indexado.

`ACTIVE` significa:

> **o documento está em condição válida para participar do Retrieval.**

---

# 6. Fora do critério de fechamento da Alpha

D.8 não transforma a validação em validação do Snoopy inteiro.

Não constituem requisitos de fechamento:

* OCR completo;
* multimodal avançado;
* tabelas complexas;
* múltiplos perfis de segmentação;
* múltiplos rerankers;
* MMR;
* decomposição sofisticada de consultas;
* avaliação formal de Retrieval;
* versionamento completo;
* snapshots;
* histórico completo;
* grafo formal de proveniência;
* colaboração;
* infraestrutura distribuída;
* Kubernetes;
* microservices;
* event sourcing;
* CQRS;
* offline-first;
* observabilidade avançada;
* disaster recovery;
* WebSockets caso SSE seja suficiente.

Essas capacidades permanecem explicitamente pós-Alpha.

---

# 7. Critério definitivo de fechamento

A Alpha estará arquiteturalmente concluída quando um vertical slice real conseguir executar:

```text
DOCUMENTO
   ↓
REPRESENTAÇÃO
   ↓
SEGMENTAÇÃO
   ↓
UNIDADES
   ↓
INDEXAÇÃO
   ↓
ACTIVE
   ↓
INVESTIGAÇÃO
   ↓
RECUPERAÇÃO HÍBRIDA
   ↓
RESULTADOS
   ↓
EVIDÊNCIA
   ↓
PROVENIÊNCIA
   ↓
SÍNTESE
   ↓
STRUCTURED RESPONSE
   ↓
RENDERER
   ↓
INTERFACE
   ↓
EVIDENCE DRAWER
   ↓
READING / SOURCE
   ↓
RETORNO
```

respeitando simultaneamente:

```text
identidade
estado
proveniência
publicação coerente
autorização
operações assíncronas
recuperação após perda de comunicação
```

---

# 8. Teste de realidade da Alpha

O critério final é:

> **Não declarar a Alpha concluída porque os módulos existem. Declarar concluída quando um documento real puder atravessar o fluxo documental e, posteriormente, ser recuperado em uma investigação real, transformado em evidência, utilizado em uma síntese estruturada, apresentado na interface, aberto na fonte e retornado à investigação original — mantendo identidade, estado, proveniência e recuperação diante de falhas.**

Se esse fluxo funciona, a Alpha materializou a arquitetura.

Se existem muitos módulos, uma interface funcional ou uma demonstração visual, mas o fluxo ainda depende de atalhos estruturais do V2, a Alpha não está concluída.

---

# 9. Estado da Etapa D após D.5–D.8

Com D.1–D.8 consolidadas:

```text
D.1  Baseline / Migração             ✓
D.2  Planta do projeto               ✓
D.3  Contratos / Modelo              ✓
D.4  Banco / Persistência            ✓
D.5  API / Comunicação               ✓
D.6  Frontend / Renderer              ✓
D.7  Sequência de implementação       ✓
D.8  Vertical Slice / Validação      ✓
D.9  Contrato com o executor         → PRÓXIMA
```

D.5–D.8 não deixam decisões arquiteturais fundamentais abertas para o executor.

O próximo passo é D.9, que possui natureza diferente das anteriores: **não é mais uma parte da arquitetura do Snoopy, mas o contrato que determina como o executor deve interpretar, preservar e executar toda a especificação D.**

**D.1–D.8 permanecem fechadas e não devem ser reabertas sem nova evidência arquitetural.**
