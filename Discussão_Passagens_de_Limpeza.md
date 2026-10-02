Calma aí, não cresça muito as etapas não. Vamos ser precisos.
Serão 4 passadas:

# A sequência de limpeza que agora proponho

Com o que descobrimos, eu faria **cinco passadas**, nesta ordem:

### Passada 1 — Higienização estrutural

Sem mudar comportamento:

-  identificar artefatos; 
-  identificar legado; 
-  identificar duplicações; 
-  identificar arquivos não utilizados; 
-  identificar arquivos temporários; 
-  identificar entrypoints; 
-  identificar dependências entre V2 e V3. 

Aqui entra, por exemplo, `diff-main.txt`.

### Passada 2 — Backend

Transformar:

```
```

```
main.py ≈ 962 linhas
```

em uma API realmente modular, conforme a documentação.

Primeiro rotas/dependências/composição; depois serviços.

### Passada 3 — Modelo e contratos

Unificar as fronteiras:

```
```

```
Document
DocumentRepresentation
RetrievalUnit
Operation
Investigation
RetrievalResult
Evidence
StructuredResponse
```

e corrigir contratos inconsistentes como o `location`.

### Passada 4 — Retrieval/document pipeline

Depois da estrutura:

-  representação; 
-  segmentação; 
-  publicação; 
-  FTS; 
-  vetor; 
-  RRF; 
-  contexto; 
-  evidência; 
-  proveniência. 

Aqui entra a correção do RRF que encontramos anteriormente.

### Passada 5 — Frontend

Só então:

```
```

```
1. separar main.js
2. consolidar state
3. consolidar features
4. consolidar renderer
5. definir contratos
6. migrar para Vue 3 + TypeScript
7. reproduzir a experiência V2

```

Vamos evitar cair nos erros que fizeram os outros dois chats anteriores a você terem atingido o limite de mensagens 

Perfeito. **Cinco passadas, exatamente nessa ordem.** Eu tinha expandido demais o processo.

E, sim: vamos trabalhar com uma regra importante para não repetir o problema dos outros chats: **cada passada tem um escopo fechado**. Não vamos antecipar a Passada 4 enquanto estamos na 1, nem transformar uma descoberta em dez novas etapas.

### Nosso contrato de trabalho

**Passada 1 — Higienização estrutural**

* artefatos
* legado
* duplicações
* não utilizados
* temporários
* entrypoints
* dependências V2 ↔ V3

**Passada 2 — Backend**

* modularização de `main.py`
* rotas/dependências/composição
* serviços
* sem ainda reabrir toda a arquitetura conceitual

**Passada 3 — Modelo e contratos**

* entidades
* fronteiras
* contratos entre camadas
* inconsistências como `location`

**Passada 4 — Retrieval/document pipeline**

* representação
* segmentação
* publicação
* FTS/vetor
* RRF
* contexto
* evidência
* proveniência

**Passada 5 — Frontend**

* `main.js`
* state
* features
* renderer
* contratos
* Vue 3 + TypeScript
* preservação da experiência V2

E uma distinção fundamental:

> **Analisar ≠ alterar.**

Durante cada passada, podemos descobrir que algo deve ser removido ou reestruturado. **Isso vira uma decisão registrada. Não vira uma alteração automática no GitHub.**

No final de cada passada, idealmente teremos algo como:

```text
ACHADOS
├── confirmado
├── provável
└── precisa verificar

DECISÕES
├── manter
├── migrar
├── substituir
└── remover

DEPENDÊNCIAS
└── o que precisa permanecer até determinada etapa
```

Só quando tu disser explicitamente algo como **“agora pode aplicar a limpeza da Passada 1”**, eu passo de análise para alteração.

E concordo contigo sobre o principal: **não vamos transformar o Snoopy numa reforma infinita da reforma da reforma.** Cinco passadas, escopo definido, profundidade onde ela é necessária e sem abrir novas frentes artificialmente.


Bora. **Passada 1 — Higienização estrutural.** Sem alterar nada no GitHub.

Vou fechar essa passada com o levantamento objetivo do que temos hoje e das decisões que ela produz.

## 1. O que pertence à V3 Alpha

O núcleo novo está claramente concentrado em:

```text
backend/
frontend/
supabase/
requirements.txt
```

E a configuração de ambiente/projeto:

```text
.env.example
.gitignore
LICENSE
```

não entra na discussão de legado.

Esses elementos ficam.

---

## 2. `src/` — legado V2 confirmado

```text
src/
├── chunker.py
├── cleaner.py
├── extractor.py
├── pipeline.py
├── search.py
├── tagger.py
└── worker.py
```

Nenhum componente da Alpha depende dele.

Ele continua existindo em `main`, portanto não há risco de perder o motor V2.

**Decisão da Passada 1:**

> `src/` → **REMOVER de `dev` posteriormente.**

Não há necessidade de carregá-lo para as próximas passadas.

---

## 3. `ui/` — não é para remover

```text
ui/
├── api.js
├── auth.js
├── index.html
├── main.js
├── style.css
└── ui.js
```

Ele não é utilizado diretamente pelo novo frontend Alpha, mas possui justamente aquilo que precisamos preservar: **a implementação funcional da experiência V2**.

Então:

> `ui/` → **MANTER durante todo o processo de migração.**

Depois da Passada 5, quando a interface V3 tiver absorvido o que precisa dela, aí podemos decidir seu destino.

---

## 4. `server.js` — transição, não Alpha

O `server.js` atual é uma ponte entre o mundo antigo e a Alpha:

```text
Express
 ├── serve ui/
 └── /api/v3/*
          ↓
    FastAPI :3000
```

Mas a execução canônica que definimos é:

```bash
uvicorn backend.api.main:app --reload --port 3000
```

Logo ele **não faz parte do núcleo de execução da V3**.

Porém, diferentemente de `src/`, ele teve função explícita durante a transição.

**Decisão:**

> `server.js` → **LEGADO DE TRANSIÇÃO; candidato à remoção.**

Não precisamos dele para a Passada 2.

---

## 5. `package.json` e `package-lock.json`

São configurações do ambiente Node/V2.

O problema não é simplesmente terem "Node". O problema é que eles ainda descrevem o projeto como:

```text
npm start
    ↓
server.js
```

quando o Alpha é:

```text
uvicorn
    ↓
backend.api.main:app
```

Além disso, o futuro frontend será Vue 3 + TypeScript, então haverá naturalmente um novo `package.json`.

**Decisão:**

> `package.json` atual → **LEGADO; será substituído.**

> `package-lock.json` → **LEGADO; será substituído junto da nova configuração frontend.**

Não devemos apagar agora e ficar com um buraco até a Passada 5. Mas sabemos que eles **não devem sobreviver à configuração final da V3**.

---

## 6. `README.MD`

O README atual ainda carrega a identidade da V2.

Isso é um problema de organização do projeto, não de código.

A `dev` precisa eventualmente dizer claramente:

```text
Snoopy-RAG V3 Alpha
```

e explicar:

```text
uvicorn backend.api.main:app --reload --port 3000
```

em vez de apresentar o ambiente V2 como se fosse o atual.

**Decisão:**

> `README.MD` → **SUBSTITUIR por documentação da V3 Alpha.**

Isso pode ser feito na limpeza estrutural, sem esperar as outras passadas.

---

## 7. `README_V1.md`

Esse é histórico.

Não é necessário para executar a V3, mas também não tem o mesmo caráter de `src/`: ele pode ter valor documental.

Então não vou classificá-lo como "apague imediatamente".

**Decisão:**

> `README_V1.md` → **LEGADO DOCUMENTAL; avaliar remoção da `dev`.**

A história da V1 não precisa necessariamente estar na árvore de trabalho da V3.

---

## 8. `diff-main.txt`

Aqui é diferente.

Isso é claramente um artefato produzido durante o desenvolvimento:

```text
diff-main.txt
```

Não representa componente, documentação oficial, configuração nem histórico necessário para execução.

**Decisão:**

> `diff-main.txt` → **REMOVER.**

Esse é o caso mais inequívoco de lixo estrutural encontrado.

---

# Resultado da Passada 1

A fotografia final fica:

```text
snoopy-rag/
│
├── .env.example             KEEP
├── .gitignore               KEEP
├── LICENSE                  KEEP
├── README.MD                REWRITE
│
├── backend/                 KEEP — V3
├── frontend/                KEEP — V3
├── requirements.txt         KEEP — V3
├── supabase/                KEEP — V3
│
├── ui/                      KEEP — referência V2
│
├── src/                     REMOVE — V2
├── server.js                REMOVE — transição
├── package.json             REPLACE
├── package-lock.json        REPLACE
├── README_V1.md             REMOVE/ARQUIVAR
└── diff-main.txt             REMOVE
```

### E o que **não** fazemos agora

Não mexemos em:

```text
backend/
frontend/
supabase/
```

para "melhorar" código.

Isso já seria Passada 2, 3, 4 ou 5.

Não mexemos em:

```text
ui/
```

porque ela é nossa referência funcional.

Não começamos a quebrar `main.py`.

Não começamos a corrigir RRF.

Não começamos a discutir Vue.

**Passada 1 terminou aqui.**

O resultado dela é simplesmente: **sabemos o que é V3, o que é legado V2, o que é transição e o que é artefato.**

A próxima é a **Passada 2 — Backend**, e aí sim entramos no `main.py` e na organização interna do backend.


## Passada 2 — Backend

Fiz a leitura do backend atual da `dev`. A conclusão principal é importante:

> **A arquitetura do backend já está estruturalmente separada em camadas. O problema principal não é falta de serviços; é que `main.py` ainda concentra responsabilidades demais e faz a composição diretamente no módulo de entrada.**

Ou seja: **não precisamos reconstruir o backend. Precisamos desacoplar o que já existe.**

### 1. O que temos hoje

A estrutura é:

```text
backend/
├── api/
│   ├── main.py
│   └── translation.py
│
├── application/
│   ├── document_processor.py
│   ├── drive_sync_service.py
│   ├── operation_service.py
│   ├── publication_service.py
│   ├── retrieval_service.py
│   ├── segmentation_service.py
│   └── synthesis_service.py
│
├── domain/
│   ├── entities.py
│   └── repositories.py
│
├── infrastructure/
│   ├── database.py
│   ├── drive_provider.py
│   ├── gemini_provider.py
│   └── supabase_repositories.py
│
└── config.py
```

Isso já nos dá uma separação bastante clara:

```text
API
 ↓
Application
 ↓
Domain ←→ Infrastructure
```

Portanto, **não vamos inventar uma arquitetura nova**.

---

# 2. O problema real: `main.py`

O `main.py` está fazendo praticamente tudo.

Ele atualmente concentra:

```text
FastAPI initialization
        +
infraestrutura
        +
injeção/composição dos serviços
        +
autenticação
        +
autorização
        +
modelos de request
        +
folders
        +
documents
        +
Drive sync
        +
operations
        +
history
        +
investigation
        +
static/frontend serving
```

Isso explica as ~962 linhas.

O problema não é simplesmente "arquivo grande".

O problema é que o **entrypoint conhece detalhes demais**.

Por exemplo, ele instancia diretamente:

```text
SupabaseOperationRepository
SupabaseDocumentRepository
SupabaseRetrievalUnitRepository
SupabaseInvestigationRepository
SupabaseRetrievalResultRepository
SupabaseEvidenceRepository

GeminiEmbeddingProvider
GoogleDriveProvider

OperationService
RetrievalService
SynthesisService
DocumentProcessor
SegmentationService
PublicationService
DriveSyncService
```

E ainda contém as próprias regras de autenticação/autorização.

Então hoje temos algo próximo de:

```text
main.py
 ├── sabe como construir infraestrutura
 ├── sabe como autenticar
 ├── sabe como autorizar
 ├── sabe como acessar Supabase
 ├── define requests
 ├── define endpoints
 └── coordena aplicação
```

Isso é o que a Passada 2 precisa corrigir.

---

# 3. O primeiro corte: composição

Eu não começaria quebrando endpoint por endpoint.

Primeiro devemos retirar de `main.py` a **composição da aplicação**.

Algo conceitualmente assim:

```text
backend/
├── api/
│   ├── main.py
│   ├── dependencies.py
│   └── routers/
│
├── application/
├── domain/
├── infrastructure/
└── config.py
```

E:

```text
main.py
   │
   ├── cria FastAPI
   ├── registra routers
   └── configura aplicação
```

Enquanto outra parte passa a cuidar de:

```text
repositories
providers
services
```

Isso reduz drasticamente o acoplamento do entrypoint.

---

# 4. Segundo corte: autenticação/autorização

Hoje:

```python
def get_authenticated_user_id(...)
```

e

```python
def ensure_folder_access(...)
```

estão em `main.py`.

Isso é claramente responsabilidade de API/dependência, não do módulo que simplesmente inicializa a aplicação.

Eu separaria conceitualmente:

```text
api/
├── dependencies.py
└── authorization.py
```

ou, se quisermos evitar criar arquivos demais:

```text
api/
└── dependencies.py
```

contendo:

```text
get_authenticated_user_id()
ensure_folder_access()
```

A razão é simples:

> Os endpoints precisam dessas regras; o `main.py` não.

---

# 5. Terceiro corte: endpoints

O próximo problema é que todos os endpoints estão no mesmo arquivo.

Pelo que encontramos, já existem grupos naturais:

```text
Folders
Documents
Drive / Sync
Operations
History
Investigation
Translation
```

E `translation.py` já demonstra que o projeto começou a fazer isso.

Então a direção natural é:

```text
api/
├── main.py
├── dependencies.py
├── translation.py
└── routers/
    ├── folders.py
    ├── documents.py
    ├── operations.py
    ├── investigations.py
    └── sync.py
```

Não estou dizendo que precisamos obrigatoriamente desses nomes exatos. O importante é a **fronteira por responsabilidade da API**.

---

# 6. Mas há uma coisa que NÃO devemos fazer

Não quero transformar isto em:

```text
main.py
100 linhas

folders.py
150 linhas

documents.py
180 linhas

sync.py
200 linhas

...
```

e chamar isso de modularização.

Porque aí apenas deslocamos o problema.

A regra deve ser:

> **Router recebe requisição → valida/autentica → chama aplicação → transforma resultado em resposta.**

O router não deve começar a implementar lógica de negócio.

Por exemplo:

```text
❌ Router
   ↓
Supabase diretamente
   ↓
regra de negócio
   ↓
Service
```

Queremos:

```text
✓ Router
   ↓
Application Service
   ↓
Domain
   ↓
Infrastructure
```

Quando uma operação ainda não possui um serviço adequado, **aí sim** identificamos isso como parte da Passada 2.

---

# 7. Um ponto positivo importante

Os serviços existentes já mostram uma boa divisão.

Temos, por exemplo:

```text
DriveSyncService
PublicationService
RetrievalService
SynthesisService
OperationService
DocumentProcessor
SegmentationService
```

Isso é bom porque o `main.py` não contém o processamento documental inteiro.

A cadeia já está aproximadamente:

```text
DriveSyncService
       ↓
PublicationService
       ↓
DocumentProcessor
       ↓
SegmentationService
       ↓
EmbeddingProvider
       ↓
Repository
```

E:

```text
RetrievalService
       ↓
Investigation
       ↓
RetrievalResult
       ↓
SynthesisService
       ↓
Evidence
       ↓
StructuredResponse
```

Portanto, **não devemos fragmentar esses serviços só porque estamos fazendo uma limpeza**.

---

# 8. O problema secundário: Infrastructure exposta demais

`main.py` instancia diretamente infraestrutura:

```text
Supabase...
Gemini...
GoogleDrive...
```

Isso cria:

```text
API
 ↓
Infrastructure
```

diretamente.

Embora os serviços usem abstrações do domínio em vários pontos, a composição está toda exposta ao entrypoint.

A solução não é criar uma "Dependency Injection Framework™ 47 camadas". 😅

Basta uma composição explícita e simples:

```text
application factory / container
        ↓
services
        ↓
repositories/providers
```

Algo pequeno, compreensível e Pythonico.

---

# 9. O que encontrei nos repositórios

A persistência também está relativamente bem encapsulada.

Existe:

```text
OperationRepository
DocumentRepository
RetrievalUnitRepository
InvestigationRepository
RetrievalResultRepository
EvidenceRepository
```

com implementações em:

```text
Supabase...
```

Isso confirma que **não precisamos resolver persistência agora**.

Especialmente porque a Passada 3 é justamente onde vamos mexer nas fronteiras:

```text
Document
DocumentRepresentation
RetrievalUnit
Operation
Investigation
RetrievalResult
Evidence
StructuredResponse
```

Então não vou antecipar isso aqui.

---

# 10. Resultado da Passada 2

A fotografia que fica é:

### 🟢 Manter

```text
application/
domain/
infrastructure/
config.py
translation.py
```

A arquitetura geral é válida.

### 🟡 Refatorar

```text
backend/api/main.py
```

Principalmente:

```text
composição
autenticação
autorização
modelos de request
rotas
```

### 🟢 Serviços

Não há motivo para uma reescrita geral dos serviços.

Eles precisam ser **revisados à medida que o acoplamento for removido**, não destruídos e reconstruídos.

---

## Estrutura-alvo desta passada

Sem ainda mexer nos contratos da Passada 3, eu colocaria o backend aproximadamente nesta forma:

```text
backend/
│
├── api/
│   ├── main.py
│   ├── dependencies.py
│   ├── translation.py
│   └── routers/
│       ├── folders.py
│       ├── documents.py
│       ├── operations.py
│       ├── sync.py
│       └── investigations.py
│
├── application/
│   ├── document_processor.py
│   ├── drive_sync_service.py
│   ├── operation_service.py
│   ├── publication_service.py
│   ├── retrieval_service.py
│   ├── segmentation_service.py
│   └── synthesis_service.py
│
├── domain/
│   ├── entities.py
│   └── repositories.py
│
├── infrastructure/
│   ├── database.py
│   ├── drive_provider.py
│   ├── gemini_provider.py
│   └── supabase_repositories.py
│
└── config.py
```

**Isso é o resultado da análise, não uma autorização para aplicar a mudança.**

E há uma decisão importante para não misturarmos as passadas:

> **A Passada 2 termina na modularização da API/backend. Não vamos corrigir `RetrievalUnit`, `location`, RRF, representação documental ou persistência agora.**

Esses problemas ficam deliberadamente estacionados para as Passadas 3 e 4.


## Passada 3 — Modelo e contratos

Aqui apareceu uma coisa mais importante do que eu esperava: **o problema não é apenas padronizar nomes. Algumas entidades previstas pela arquitetura ainda não existem como contratos de domínio completos.**

A V3 já tem boa parte das peças, mas algumas fronteiras estão sendo atravessadas por `dict`, `metadata` e strings.

---

# 1. O mapa atual

Hoje `entities.py` define:

```text
DocumentStatus
OperationStatus

DocumentBlock
DocumentPage
DocumentRepresentation
RetrievalUnit
Investigation
RetrievalResult
Evidence
StructuredResponse
Operation
```

Mas repare no primeiro problema:

```text
Document
```

**não existe como entidade de domínio.**

Existe `DocumentRepository`, existe a tabela `documents`, existem documentos sendo manipulados nos serviços, mas não existe um:

```python
class Document(...)
```

Isso é uma lacuna real.

A arquitetura que definimos conceitualmente é:

```text
Document
   ↓
DocumentRepresentation
   ↓
RetrievalUnit
```

Mas o código atualmente começa em:

```text
document_id: str
title: str
folder_id: str
user_id: str
drive_file_id: str
document_hash: str
```

espalhados como parâmetros.

### Decisão

Na Passada 3:

> **Criar uma fronteira explícita para `Document`.**

Não necessariamente significa criar uma entidade gigantesca. Significa que o sistema precisa ter uma representação coerente do documento como entidade.

---

# 2. `DocumentRepresentation` está conceitualmente correta

Esta é uma das partes mais sólidas:

```python
class DocumentRepresentation(BaseModel):
    representation_id: str
    document_id: str
    pages: List[DocumentPage]
    metadata: Dict[str, Any]
```

Temos:

```text
Document
   ↓
Representation
```

e a representação possui sua própria identidade.

Isso é importante porque permite distinguir:

```text
o documento
≠
uma representação específica daquele documento
```

**Não vamos desfazer isso.**

A melhoria da estrutura interna da representação pertence à Passada 4.

---

# 3. `RetrievalUnit` está correta como conceito, mas seu contrato está frouxo

Hoje:

```python
class RetrievalUnit(BaseModel):
    unit_id: Optional[int]
    document_id: str
    representation_id: str
    unit_index: int
    content: str
    location: Dict[str, Any]
    section: str
    metadata: Dict[str, Any]
```

O problema mais evidente é:

```text
location: Dict[str, Any]
metadata: Dict[str, Any]
```

Isso permite praticamente qualquer coisa.

E já estamos usando `location` para algo importante:

```json
{
  "start_page": 1,
  "end_page": 3,
  "start_block": 2,
  "end_block": 7
}
```

Isso não deveria ser um dicionário arbitrário para sempre.

Mas **não devemos resolver a estrutura detalhada de localização agora**, porque isso encosta diretamente na Passada 4, onde vamos trabalhar representação, segmentação e proveniência.

### Decisão

Passada 3:

> preservar `location` como fronteira conceitual, mas preparar um contrato explícito.

Passada 4:

> definir sua estrutura documental precisa.

---

# 4. `RetrievalResult` está carregando coisa demais dentro de `metadata`

Esse é provavelmente o problema contratual mais claro.

Hoje:

```python
class RetrievalResult(BaseModel):
    result_id: str
    investigation_id: str
    unit_id: int
    rank: int
    retrieval_score: float
    metadata: Dict[str, Any]
```

E o `RetrievalService` coloca dentro desse `metadata`:

```text
document_id
content
location
raw_semantic_score
raw_lexical_score
```

Ou seja:

```text
RetrievalResult
    └── metadata
         ├── document_id
         ├── content
         ├── location
         ├── semantic_score
         └── lexical_score
```

Isso é um sinal de contrato ainda provisório.

O resultado já possui:

```text
unit_id
```

Portanto, o `content`, `document_id` e `location` não deveriam precisar viajar escondidos em `metadata` para que o próximo serviço descubra o que foi recuperado.

Isso também explica por que `SynthesisService` precisa fazer:

```python
result.metadata.get("document_id")
result.metadata.get("location")
result.metadata.get("content")
```

### Decisão

> **`RetrievalResult` precisa ganhar um contrato explícito para os dados necessários à materialização da evidência.**

Mas a forma exata disso deve ser decidida junto com a Passada 4, porque envolve o que exatamente o retrieval retorna.

---

# 5. `Evidence` está conceitualmente muito melhor

Aqui a arquitetura está boa:

```python
class Evidence(BaseModel):
    evidence_id
    investigation_id
    unit_id
    document_id
    location
    content
    context
    provenance
```

Temos:

```text
RetrievalResult
       ↓
Evidence
```

e não:

```text
RetrievalResult == Evidence
```

Isso é exatamente a distinção que queremos.

A `Evidence` ganha identidade própria e registra sua proveniência.

**Essa fronteira deve permanecer.**

---

# 6. Mas `Evidence.provenance` ainda é um `dict`

Hoje:

```python
provenance: Dict[str, Any]
```

e o serviço produz:

```json
{
  "investigation_id": "...",
  "result_id": "...",
  "unit_id": "...",
  "document_id": "...",
  "location": {...}
}
```

Isso funciona como Alpha, mas o problema é que a proveniência é **estruturalmente importante demais para permanecer eternamente como bagagem arbitrária**.

Porém, novamente:

> Não vamos modelar completamente a proveniência na Passada 3.

A Passada 4 é justamente onde isso será fechado junto com rastreabilidade documental.

Então aqui fica:

**identificado → não resolvido ainda.**

---

# 7. `Investigation` tem uma pequena inconsistência importante

Hoje:

```python
class Investigation(BaseModel):
    investigation_id
    user_id
    original_query
    filters
    status
    structured_response
    created_at
```

O `folder_id` não existe diretamente.

Ele está escondido em:

```python
filters = {
    "folder_id": folder_id,
}
```

Isso é ruim porque o acervo é uma fronteira estrutural da investigação.

Hoje:

```text
Investigation
   └── filters
        └── folder_id
```

Quando conceitualmente queremos:

```text
Investigation
   ├── investigation_id
   ├── user_id
   ├── folder_id
   ├── original_query
   └── ...
```

`filters` deveria representar **filtros da investigação**, não esconder sua relação estrutural com o acervo.

### Decisão

> **`folder_id` deve ser um atributo explícito de `Investigation`.**

`filters` continua existindo para filtros propriamente ditos.

---

# 8. `Operation` também precisa de uma fronteira melhor

Hoje:

```python
class Operation(BaseModel):
    operation_id
    operation_type: str
    target_id: Optional[str]
    status: OperationStatus
    ...
```

Temos:

```text
target_id
```

mas não:

```text
target_type
```

Então:

```text
target_id = "abc"
```

pode significar documento, acervo, investigação etc.

No Alpha atual sabemos que `DRIVE_SYNC` usa o `folder_id`, mas isso está implícito no contexto.

Para a V3, a operação precisa deixar claro:

```text
Operation
 ├── operation_id
 ├── operation_type
 ├── target_type
 ├── target_id
 └── status
```

Não necessariamente precisamos de uma abstração exagerada; basta eliminar a ambiguidade.

### Decisão

> **Revisar `Operation.target_id` para explicitar a natureza do alvo.**

---

# 9. Estados estão excessivamente stringificados

Temos:

```python
OperationStatus(Enum)
```

mas:

```python
Investigation.status: str
```

e o código usa:

```python
"PROCESSING"
"COMPLETED"
"FAILED"
```

enquanto documentos usam:

```python
DocumentStatus(Enum)
```

Isso é inconsistente.

O modelo deveria ser:

```text
DocumentStatus       → enum
OperationStatus      → enum
InvestigationStatus  → enum
```

Isso evita estados arbitrários como:

```text
"complete"
"COMPLETED"
"finished"
"done"
```

### Decisão

> Criar `InvestigationStatus` e tipar `Investigation.status`.

O mesmo princípio vale para tipos de operação, mas aqui precisamos primeiro definir quais tipos realmente existem.

---

# 10. `StructuredResponse` está sem persistência própria

Temos:

```python
class StructuredResponse:
    response_id
    content
    sections
    evidence_refs
    references
```

Mas não existe:

```text
StructuredResponseRepository
```

A resposta é armazenada dentro de:

```text
investigations.structured_response
```

através de:

```python
investigation_repo.save_structured_response(...)
```

Isso não é necessariamente errado.

Aliás, **pode ser exatamente a decisão correta**: a resposta ser um estado materializado da Investigation, e não uma entidade persistida independentemente.

O problema é que isso precisa ser uma decisão arquitetural explícita.

Hoje parece mais uma consequência da implementação.

### Decisão

> **Não criar automaticamente um `StructuredResponseRepository`.**

Na Passada 3 devemos formalizar:

```text
StructuredResponse
    pertence à Investigation
```

caso essa seja realmente a relação pretendida.

Isso evita criar entidade/tabela só porque o nome existe no modelo.

---

# 11. O contrato mais problemático: `Repository`

Os repositórios estão razoavelmente alinhados com as entidades, mas ainda existem sinais de transição.

Por exemplo:

```python
RetrievalUnitRepository.save_batch(
    units,
    embeddings,
    user_id,
    folder_id
)
```

O próprio `RetrievalUnit` já contém:

```text
document_id
```

mas não contém:

```text
folder_id
user_id
```

Então o repositório precisa receber dados externos para completar a persistência.

Isso indica que temos duas possibilidades:

### A

`RetrievalUnit` representa somente a unidade documental, e contexto de autorização/persistência fica fora dela.

### B

`RetrievalUnit` deveria carregar a relação com acervo/owner.

Não devemos decidir isso superficialmente.

Pela arquitetura que estabelecemos, eu **não colocaria `user_id` dentro da RetrievalUnit só para satisfazer o banco**. Identidade documental e autorização são conceitos diferentes.

Então:

> `user_id` e `folder_id` no repositório precisam ser tratados como **contexto de persistência/autorização**, não automaticamente como atributos da unidade.

Isso é importante.

---

# 12. O maior problema transversal: `metadata` como válvula de escape

A fotografia geral é:

```text
DocumentRepresentation
    metadata: Dict

RetrievalUnit
    metadata: Dict
    location: Dict

RetrievalResult
    metadata: Dict

Evidence
    provenance: Dict
```

Quatro pontos diferentes estão usando dicionários para transportar informação estrutural.

Isso é típico de uma Alpha — e **não é necessariamente um defeito neste estágio**.

Mas agora sabemos onde está a dívida:

```text
dict
 ↓
contrato explícito
```

A Passada 3 deve fechar as **fronteiras semânticas**.

A Passada 4 vai fechar os **conteúdos dessas estruturas**, principalmente localização e proveniência.

---

# Resultado da Passada 3

Eu fecharia a passada com este quadro:

| Elemento                 | Estado atual                             | Decisão                 |
| ------------------------ | ---------------------------------------- | ----------------------- |
| `Document`               | ❌ ausente como entidade                  | **criar contrato**      |
| `DocumentRepresentation` | 🟢 bom                                   | manter                  |
| `RetrievalUnit`          | 🟢 conceito correto / contrato frouxo    | revisar                 |
| `Operation`              | 🟡 `target_id` ambíguo                   | explicitar alvo         |
| `Investigation`          | 🟡 `folder_id` escondido em `filters`    | tornar explícito        |
| `RetrievalResult`        | 🟡 dados essenciais em `metadata`        | revisar contrato        |
| `Evidence`               | 🟢 conceito correto                      | manter fronteira        |
| `StructuredResponse`     | 🟡 persistência acoplada à Investigation | formalizar relação      |
| Status                   | 🟡 mistura enum/string                   | tipar                   |
| `location`               | 🟡 `Dict` provisório                     | estruturar na Passada 4 |
| `provenance`             | 🟡 `Dict` provisório                     | estruturar na Passada 4 |
| Repositories             | 🟢 arquitetura correta                   | alinhar aos contratos   |

### O fluxo conceitual que emerge

```text
Document
   │
   ▼
DocumentRepresentation
   │
   ▼
RetrievalUnit
   │
   ▼
RetrievalResult
   │
   ▼
Evidence
   │
   ▼
StructuredResponse
   │
   ▼
Investigation
```

Com `Operation` atravessando os processos de execução, **sem virar pai dessas entidades**.

Essa é a principal conclusão da Passada 3: **as entidades fundamentais estão quase todas presentes, mas algumas relações importantes ainda estão escondidas em `metadata`, `filters` ou parâmetros soltos.** A Passada 4 pode agora trabalhar em cima dessas fronteiras sem precisar redefinir a arquitetura inteira.


Sim. Depois de seguir o fluxo real da `dev`, a **Passada 4 revela que a arquitetura conceitual está correta, mas a implementação ainda tem alguns pontos importantes de consistência**. E há um problema bem sério no ciclo de atualização de documentos.

## 1. Representação: a direção está certa, mas ainda não é realmente canônica

O `DocumentProcessor` faz algo conceitualmente correto:

```text
PDF
 ↓
PyMuPDF
 ↓
DocumentPage
 ↓
DocumentBlock
 ↓
DocumentRepresentation
```

Ele deixou de fazer o achatamento destrutivo PDF → Markdown da V2. Isso é exatamente a direção que o Snoopy precisava.

Mas há uma diferença entre **“preservar a estrutura”** e **“preservar a representação documental”**.

Hoje `DocumentBlock` guarda apenas:

```python
block_index
text
block_type
```

Embora o `get_text("blocks")` forneça:

```text
x0, y0, x1, y1, texto, block_number, block_type
```

as coordenadas são descartadas.

Então hoje o sistema consegue dizer:

> “esta unidade veio da página 7, blocos 3–5”

mas não consegue dizer:

> “este texto estava nesta região específica da página”.

Isso é especialmente relevante para:

* colunas;
* tabelas;
* legendas;
* cabeçalhos/rodapés;
* relações espaciais;
* leitura de documentos acadêmicos diagramados.

**Conclusão:** a representação é uma boa base Alpha, mas ainda não cumpre integralmente a ideia de representação canônica que a documentação do projeto estabelece.

---

# 2. Segmentação: o contrato está correto, o algoritmo ainda é provisório

O fluxo:

```text
DocumentRepresentation
        ↓
SegmentationService
        ↓
RetrievalUnit
```

está correto.

E há uma coisa boa aqui: a unidade não perde sua origem.

Cada `RetrievalUnit` recebe:

```text
document_id
representation_id
unit_index
location
content
```

Isso mantém o caminho:

```text
RetrievalUnit
    ↓
DocumentRepresentation
    ↓
Document
```

### Mas a segmentação atual é um “balde de 250 palavras”

Ela simplesmente acumula blocos até chegar a aproximadamente 250 palavras.

Consequências:

* pode cortar uma unidade em lugares semanticamente ruins;
* não reconhece títulos;
* não reconhece seções;
* não reconhece parágrafos como unidades semânticas;
* não conhece tabelas;
* não conhece colunas;
* `section` é sempre `"Geral"`.

Então este comentário do próprio código:

> “garantindo que cada fragmento mantenha sua rastreabilidade espacial exata”

é **mais forte do que aquilo que o código realmente garante**.

Ele mantém rastreabilidade **por intervalo de página/bloco**, não rastreabilidade espacial exata.

### E encontrei um pequeno bug de localização

No fragmento residual final, o código faz:

```python
end_page = representation.pages[-1].page_number
end_block = representation.pages[-1].blocks[-1].block_index
```

Ou seja, se a última `RetrievalUnit` começa na página 20 e termina no bloco 2 da página 20, mas o documento possui páginas até a 30, ela poderá ser registrada como terminando na página 30.

Isso não afeta o texto recuperado, mas **corrompe a proveniência declarada da unidade**.

É um bug real de `location`, não apenas uma questão arquitetural.

---

# 3. Publicação: há uma boa garantia, mas a transação lógica ainda é frágil

O `PublicationService` possui uma sequência muito boa:

```text
PROCESSING
   ↓
PDF → Representation
   ↓
persist Representation
   ↓
Representation → RetrievalUnits
   ↓
Embeddings
   ↓
persist Units + embeddings
   ↓
ACTIVE
```

E só marca o documento como `ACTIVE` depois de todas as etapas.

Isso é correto.

Também há uma verificação importante:

```text
quantidade de embeddings == quantidade de RetrievalUnits
```

Excelente para evitar desalinhamento silencioso.

### Porém, o ciclo de atualização quebra uma propriedade importante

O problema está na combinação:

```text
DriveSyncService
        ↓
delete_by_document()
        ↓
download
        ↓
create_or_update()
        ↓
PublicationService
```

O `DriveSyncService` apaga as unidades antigas **antes mesmo de terminar o download do novo PDF**.

Portanto:

```text
Documento ACTIVE
+ unidades válidas
        ↓
delete unidades
        ↓
download falha
        ↓
Documento continua ACTIVE
+ zero RetrievalUnits
```

Isso é uma inconsistência grave.

O sistema pode terminar com:

```text
documents.status = ACTIVE
chunks = ∅
```

Ou seja, o documento aparenta estar publicado, mas não pode ser recuperado.

Isso é mais importante que simplesmente “refatorar o código”: é uma propriedade de consistência do pipeline.

---

# 4. Outro problema: atualização não é versionamento

Há outro ponto ainda mais importante.

A representação é salva assim:

```text
documents.representation = JSONB
```

Quando o documento muda, essa representação é substituída.

Ao mesmo tempo, cada `RetrievalUnit` guarda:

```text
representation_id
```

Isso parece excelente à primeira vista.

Mas existe uma contradição:

```text
RetrievalUnit
    ↓
representation_id = R1
```

e depois:

```text
Document
    ↓
representation = R2
```

A representação `R1` não possui uma entidade persistente própria. Ela foi apenas substituída dentro de `documents`.

Então o sistema pode saber:

> “essa unidade veio da representação R1”

mas não necessariamente consegue recuperar **R1** posteriormente.

Isso significa que `representation_id` hoje funciona mais como **identificador de uma versão lógica** do que como verdadeira referência persistente a uma representação histórica.

Para o Snoopy isso é importante porque a documentação estabelece uma preocupação explícita com:

```text
verificabilidade
reprodutibilidade
rastreabilidade
proveniência
```

Uma referência histórica que aponta para uma versão que não está mais armazenada não fecha completamente esse circuito.

---

# 5. FTS + vetor: a arquitetura híbrida está correta

Aqui a estrutura está boa:

```text
                 ┌── Vector
Query ───────────┤
                 └── FTS
                       ↓
                     RRF
```

O PostgreSQL faz:

### Semântico

```sql
1 - (embedding <=> query_embedding)
```

### Lexical

```sql
ts_rank(
    c.fts,
    websearch_to_tsquery('portuguese', p_query_text)
)
```

E ambos respeitam:

```text
document ACTIVE
folder_id
user_id / acervo público
```

Isso está alinhado com a ideia do V3.

A função ainda devolve a união dos candidatos:

```sql
WHERE
    semantic_search.id IS NOT NULL
    OR lexical_search.id IS NOT NULL
```

Isso é correto para recuperação híbrida.

---

# 6. Mas aqui está o bug do RRF que já tínhamos encontrado

O SQL devolve algo como:

```text
A: semantic = 0.91, lexical = 0
B: semantic = 0.90, lexical = 0.42
```

O problema é que o Python faz:

```python
semantic_sorted = sorted(raw_results, ...)
lexical_sorted = sorted(raw_results, ...)
```

Ele não sabe se um resultado **realmente participou** de cada modalidade.

Então A recebe:

```text
semantic_rank = 1
lexical_rank = 2
```

mesmo não tendo aparecido na busca lexical.

Da mesma maneira, B recebe uma posição semântica mesmo que, em outro cenário, ele só tivesse vindo do FTS.

Depois:

```python
1 / (k + semantic_rank)
+
1 / (k + lexical_rank)
```

Todo candidato passa a parecer participante das duas buscas.

### Portanto:

**O RRF atual não é RRF verdadeiro.**

Ele é uma aproximação baseada no conjunto unido de candidatos.

O correto é preservar a informação:

```text
candidato A
  semantic: rank 1
  lexical:  ausente

candidato B
  semantic: rank 2
  lexical:  rank 1
```

e só calcular a parcela correspondente à modalidade em que o candidato realmente apareceu.

Isso entra diretamente na correção da Passada 4.

---

# 7. Há um problema adicional no desenho do RRF

O SQL atualmente faz:

```text
LIMIT p_match_count * 2
```

para cada modalidade.

Isso é perfeitamente razoável para um Alpha.

O problema não é o `2`.

O problema é que o SQL transforma a ausência em:

```python
COALESCE(..., 0.0)
```

e entrega ao Python:

```text
semantic_score = 0
lexical_score = 0
```

A partir daí, perdeu-se a distinção entre:

```text
não participou da busca
```

e

```text
participou e obteve score 0
```

Essa distinção precisa sobreviver até a fusão.

**Esse é o ponto exato onde o contrato da recuperação precisa ser corrigido.**

---

# 8. Contextualização: aqui temos uma lacuna real

Esse foi um dos achados mais interessantes.

A arquitetura planejada fala em:

```text
recuperação
→ fusão
→ contextualização
→ evidência
```

Mas o código atual não possui realmente uma etapa de contextualização.

O `SynthesisService` faz:

```python
for evidence in evidences:
    block = evidence.content
```

Ou seja, o contexto enviado ao LLM é essencialmente:

```text
chunk recuperado
chunk recuperado
chunk recuperado
...
```

A única informação adicional construída é:

```text
Documento ID: X
```

na propriedade `Evidence.context`.

Não há, por exemplo:

```text
unidade recuperada
+ unidade anterior
+ unidade posterior
+ seção
+ título
+ página
+ metadados documentais
```

Portanto:

```text
RetrievalUnit → Evidence
```

está funcionando.

Mas:

```text
RetrievalUnit
→ contextualização documental
→ Evidence
```

**ainda não existe como etapa própria.**

Isso é importante porque um chunk pode ser suficiente para recuperar um conceito, mas insuficiente para compreender sua função dentro do documento.

---

# 9. Evidence está conceitualmente muito melhor que RetrievalResult

Aqui o projeto acertou.

Hoje:

```text
RetrievalResult
```

é o produto da recuperação.

Depois:

```text
RetrievalResult
        ↓
Evidence
```

E a Evidence possui:

```text
investigation_id
unit_id
document_id
location
content
context
provenance
```

Essa separação deve ser preservada.

Especialmente porque o próprio `SynthesisService` instrui o LLM:

> não trate similaridade de recuperação como prova de relevância metodológica.

Isso está muito alinhado com a filosofia do Snoopy:

```text
recuperação propõe
        ↓
evidência materializa
        ↓
pesquisador interpreta
```

---

# 10. Porém, a proveniência ainda não fecha o circuito histórico

A `Evidence` atualmente registra:

```text
investigation_id
result_id
unit_id
document_id
location
```

Isso é bom para rastrear:

```text
Evidence
 ↓
RetrievalResult
 ↓
RetrievalUnit
 ↓
Document
```

Mas falta um elemento importante para a rastreabilidade histórica:

```text
representation_id
```

O `RetrievalUnit` possui esse ID, mas a Evidence não o materializa diretamente.

E, como vimos, a representação também não possui armazenamento versionado independente.

Isso produz:

```text
Evidence
   ↓
unit_id
   ↓
representation_id
   ↓
???
```

O `???` é justamente o problema: a representação histórica não está garantidamente disponível.

---

# 11. Há também uma questão temporal

Imagine:

```text
Documento D
Representação R1
Unidade U1
Evidence E1
```

Depois o PDF é atualizado:

```text
Documento D
Representação R2
Unidade U2
```

A Evidence `E1` continua contendo o texto original.

Isso é bom.

Mas:

```text
E1 → document_id D
```

agora aponta para um documento cuja representação atual é `R2`.

Então temos uma situação potencialmente ambígua:

> “E1 veio do documento D”

é verdade.

Mas:

> “E1 representa o estado atual do documento D”

não necessariamente é verdade.

Para um sistema de pesquisa acadêmica, essa diferença é importante.

---

# 12. A síntese tem uma boa fronteira de segurança

Aqui eu **não mexeria conceitualmente**.

O `SynthesisService`:

* recebe evidências;
* trata o conteúdo recuperado como dados não confiáveis;
* não permite que instruções dentro dos documentos controlem o modelo;
* exige JSON;
* valida `evidence_refs`;
* persiste a resposta.

Isso é uma boa separação:

```text
Retrieval
   ↓
Evidence
   ↓
LLM
   ↓
StructuredResponse
```

E o LLM não é tratado como fonte primária.

Isso deve permanecer.

---

# 13. O pipeline real da Alpha, portanto, fica assim

Hoje, de fato:

```text
PDF
 ↓
DocumentProcessor
 ↓
DocumentRepresentation
 ↓
SegmentationService
 ↓
RetrievalUnit
 ↓
Embedding + FTS
 ↓
Supabase hybrid search
 ↓
RetrievalService
 ↓
RetrievalResult
 ↓
SynthesisService
 ↓
Evidence
 ↓
LLM
 ↓
StructuredResponse
```

A arquitetura conceitual pretendida era:

```text
PDF
 ↓
Document
 ↓
DocumentRepresentation
 ↓
Segmentation
 ↓
RetrievalUnit
 ↓
FTS + Vector
 ↓
RRF
 ↓
Contextualization
 ↓
Evidence
 ↓
StructuredResponse
```

A diferença fundamental está em três pontos:

```text
1. representação ainda perde informação;
2. RRF está incorreto;
3. contextualização ainda não existe de fato.
```

E há um quarto problema transversal:

```text
4. publicação/atualização não é atomicamente segura.
```

---

# Diagnóstico da Passada 4

| Área                      | Estado real          | Diagnóstico                            |
| ------------------------- | -------------------- | -------------------------------------- |
| PDF → Representation      | funcional            | **Alpha, mas incompleto**              |
| Preservação de layout     | parcial              | **insuficiente**                       |
| Segmentação               | funcional            | **heurística/provisória**              |
| `location`                | funcional            | **tem bug no fragmento final**         |
| Publication               | funcional            | **boa sequência, mas não atômica**     |
| Atualização de documentos | problemática         | **pode deixar ACTIVE sem chunks**      |
| Versionamento             | ausente              | **lacuna importante**                  |
| FTS                       | funcional            | **correto**                            |
| Vector                    | funcional            | **correto**                            |
| Hybrid retrieval          | funcional            | **correto como união de candidatos**   |
| RRF                       | incorreto            | **bug confirmado**                     |
| Contextualização          | praticamente ausente | **lacuna arquitetural**                |
| RetrievalResult           | funcional            | **contrato ainda depende de metadata** |
| Evidence                  | bom conceito         | **preservar**                          |
| Proveniência              | parcial              | **precisa fechar o vínculo histórico** |
| Synthesis                 | funcional            | **boa fronteira**                      |

## O que a Passada 4 nos deixa como correções

Sem tocar no repositório ainda, a lista que fica registrada é:

1. **corrigir o `location` residual da segmentação;**
2. **preservar mais informação da representação documental;**
3. **definir melhor o contrato de localização;**
4. **corrigir o RRF para distinguir ausência de participação de score zero;**
5. **tirar a informação essencial de `RetrievalResult.metadata`;**
6. **introduzir contextualização documental real;**
7. **tornar a proveniência historicamente resolvível;**
8. **rever a publicação para que uma falha de atualização não destrua um documento anteriormente válido;**
9. **separar representação atual de representação histórica/versionada, conforme o grau de permanência que o V3 exigir.**

E uma distinção importante: **nem tudo isso precisa ser resolvido no mesmo nível agora**. O Alpha não precisa virar um sistema de versionamento documental completo antes de funcionar. Mas essas são as fronteiras que não devemos mascarar com `dict`, `metadata` ou heurísticas.

**Passada 4, portanto, está essencialmente mapeada.** A estrutura conceitual do pipeline é boa; os problemas estão nas garantias que conectam cada etapa.


Agora sim: **a Passada 5 fecha a análise do frontend**. E aqui apareceu uma coisa interessante: a estrutura de pastas da `frontend/` parece modular, mas o código ainda funciona essencialmente como uma aplicação DOM monolítica que foi parcialmente desmontada.

## 1. O problema central: a modularização começou, mas não terminou

A estrutura atual diz:

```text
frontend/
├── app/
├── components/
├── features/
├── infrastructure/
├── renderer/
└── state/
```

Isso é uma boa direção.

Mas o comportamento real ainda é:

```text
main.js
   ↓
DOM
   ↓
estado global
   ↓
controllers
   ↓
renderer
```

com várias responsabilidades atravessando essas camadas.

O `frontend/app/main.js` tem aproximadamente **30 KB / ~1.500 linhas**, apesar de já existir:

* `state/`
* `features/`
* `components/`
* `renderer/`
* `infrastructure/`

Isso indica que a primeira tentativa de modularização extraiu arquivos, mas **não extraiu completamente as responsabilidades**.

---

# 2. `main.js` ainda é o verdadeiro controlador da aplicação

Ele ainda controla diretamente:

```text
tema
acervos
Google Picker
sincronização
investigação
histórico
autenticação
logout
desconexão de acervo
sidebar
mobile
evidência
reading
settings
boot
```

Isso significa que:

```text
id="h1f5qp"
main.js
```

ainda conhece praticamente toda a aplicação.

O problema não é simplesmente o tamanho.

O problema é que ele conhece **as regras de coordenação** de quase todas as features.

Por exemplo, para sincronização ele conhece:

```text
folderId
googleToken
operation
polling
estado visual
mensagens
botões
```

Para autenticação:

```text
session
user
access_token
provider_token
folders
history
UI
```

Para investigação:

```text
query
investigation controller
DOM
mobile
```

Isso é exatamente o que precisamos desmontar.

---

# 3. O `state/` existe, mas ainda não governa a aplicação

Temos:

```javascript
appState
investigationState
```

Isso é bom.

Mas `main.js` ainda altera diretamente:

```javascript
appState.folderId
appState.folderName
appState.user
appState.session
appState.googleToken
```

E o controller também faz:

```javascript
investigationState.selectedEvidence =
    evidence;
```

embora o estado tenha:

```javascript
selectedEvidenceId
```

Ou seja, já existe uma **inconsistência concreta entre o contrato declarado do estado e o uso real**.

O estado deveria ser a fonte única da verdade.

Hoje ele é parcialmente:

> estado central

e parcialmente:

> depósito de variáveis globais compartilhadas.

Isso precisa ser corrigido antes ou durante a migração para Vue.

---

# 4. O `renderer` ainda não é realmente um renderer

`renderer/render.js` parece uma camada de apresentação, mas ele trabalha diretamente com:

```javascript
document.getElementById(...)
document.querySelector(...)
classList
innerHTML
textContent
addEventListener
```

Isso é DOM imperativo.

Então temos:

```text
controller
   ↓
renderer
   ↓
DOM
```

e o próprio `main.js` também faz:

```text
main.js
   ↓
DOM
```

Portanto existem **dois donos da UI**.

Isso é ruim para a futura migração porque Vue precisa assumir:

```text
state → template → DOM
```

e não:

```text
state
   ↘
    main.js → DOM
   ↘
    renderer → DOM
```

---

# 5. Os componentes ainda não são componentes de verdade

Temos:

```text
EvidencePanel.js
ResultList.js
SearchBar.js
```

Mas eles são módulos JS que criam/manipulam DOM.

Por exemplo, `EvidencePanel` não é um componente declarativo.

E `SearchBar.js` nem aparece como parte relevante do fluxo atual do `main.js`.

Então a arquitetura atual é uma espécie de:

> “componentização manual em JavaScript”

e não uma arquitetura de componentes.

Isso reforça que **Vue 3 não deve ser colocado por cima dessa estrutura simplesmente adicionando Vue**.

O correto será usar a migração para substituir essa camada DOM imperativa.

---

# 6. `ResultList.js` é um candidato forte a sobra

Esse era um dos arquivos que já havia levantado suspeita.

Depois de olhar o fluxo:

```text
controller.js
    ↓
renderResult()
    ↓
renderSources()
renderEvidenceList()
```

o próprio controller constrói a lista de resultados.

Portanto `ResultList.js` parece ter sido uma tentativa anterior de extração que não foi integrada ao fluxo atual.

Não vou declarar ainda como “pode deletar”, porque nossa regra é aguardar o relatório final.

Mas a classificação correta agora é:

```text
ResultList.js
→ candidato forte a artefato/transicional
→ verificar dependências finais
→ não usar na arquitetura futura sem necessidade
```

O mesmo raciocínio vale para `SearchBar.js`.

---

# 7. A feature de investigação está melhor encaminhada

`controller.js` já representa uma separação mais significativa:

```text
investigation/
└── controller.js
```

Ele:

* executa a investigação;
* controla `busy`;
* chama API;
* enriquece evidências;
* coordena rendering;
* seleciona evidências;
* abre leitura.

Mas ainda é uma mistura de:

```text
application logic
+
presentation logic
+
DOM manipulation
```

Por exemplo:

```javascript
document.createElement(...)
document.createTextNode(...)
element.addEventListener(...)
```

está dentro do controller.

Então o controller não é realmente um controller de aplicação; ele é simultaneamente:

```text
use case coordinator
+
view controller
+
renderer
```

A migração para Vue resolve isso naturalmente:

```text
Investigation service/composable
        ↓
investigation state
        ↓
Vue components
```

---

# 8. `api.js` é provavelmente a parte mais aproveitável

Essa camada está relativamente bem separada.

Temos:

```text
frontend/infrastructure/api.js
```

e ela centraliza:

```text
request()
getFreshAccessToken()
fetchFolders()
createFolder()
disconnectFolder()
investigate()
startSync()
getOperation()
waitForOperation()
fetchDocument()
fetchHistory()
translateText()
```

Isso é bom.

Principalmente porque o frontend não está espalhando `fetch()` por todos os componentes.

Para Vue/TypeScript, isso pode virar algo como:

```text
api/
├── client.ts
├── investigations.ts
├── documents.ts
├── folders.ts
├── operations.ts
└── auth.ts
```

ou uma organização equivalente.

Não precisamos decidir os nomes agora.

O princípio é:

> **preservar a fronteira API; substituir sua implementação JS por contratos TypeScript.**

---

# 9. Autenticação também possui uma fronteira razoável

`infrastructure/auth.js` concentra:

* Supabase client;
* sessão;
* OAuth;
* Google provider token;
* Google Picker.

Isso é uma boa separação.

Mas existe uma duplicação:

```text
auth.js
        ↘
         appState
        ↗
main.js
```

`main.js` também manipula diretamente:

```text
session
user
userToken
googleToken
```

Então a futura autenticação deve ter uma única fonte de verdade.

---

# 10. A grande vantagem: a V2 continua preservada

Aqui a decisão arquitetural fica bastante clara.

A `ui/` V2 ainda contém a experiência funcional original:

```text
pesquisa
histórico
acervos
Google Drive
sincronização
evidências
modo leitura
PDF original
tema
mobile
configurações
```

E a V3 já começou a reproduzir essa experiência.

Portanto o objetivo da migração não deve ser:

```text
V2 → jogar fora → inventar frontend novo
```

mas:

```text
V2 UX
   ↓
frontend V3
   ↓
Vue 3 + TypeScript
```

A experiência é a referência.

A arquitetura é que muda.

---

# 11. E há uma questão importante no `package.json`

Aqui está uma das maiores dívidas estruturais.

O `package.json` atual ainda é:

```json
{
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  }
}
```

e contém:

```text
express
ws
dotenv
supabase JS
```

Mas a aplicação Alpha real é:

```text
FastAPI
+
frontend estático
```

Então o projeto atualmente não possui um verdadeiro **ambiente de build frontend**.

Não há:

```text
Vite
Vue
TypeScript
```

nem scripts correspondentes.

Portanto a migração para Vue 3 + TypeScript não é apenas:

> renomear `.js` para `.ts`.

Ela exige definir o runtime/build do frontend.

Isso também explica por que **não devemos simplesmente apagar `package-lock.json` agora**: primeiro definimos o novo ambiente, depois geramos o lock correspondente.

---

# 12. O que a Passada 5 nos dá como arquitetura-alvo

Sem implementar ainda:

```text
frontend/
├── app/
│   ├── App.vue
│   └── main.ts
│
├── components/
│   ├── layout/
│   ├── investigation/
│   ├── evidence/
│   ├── reading/
│   ├── folders/
│   └── settings/
│
├── features/
│   ├── investigation/
│   ├── reading/
│   ├── authentication/
│   ├── folders/
│   └── synchronization/
│
├── services/
│   └── api/
│
├── stores/
│   ├── application.ts
│   └── investigation.ts
│
└── types/
    ├── investigation.ts
    ├── evidence.ts
    ├── document.ts
    └── operation.ts
```

**Isso é um exemplo de direção, não uma árvore que devemos copiar cegamente.**

O ponto principal é:

```text
Vue
 ↓
componentes
 ↓
estado/composables
 ↓
services
 ↓
FastAPI
```

e não:

```text
Vue
 ↓
DOM manual
 ↓
main.js gigante
 ↓
fetch espalhado
```

---

# 13. Diagnóstico final da Passada 5

| Área                      | Estado                      | Diagnóstico                          |
| ------------------------- | --------------------------- | ------------------------------------ |
| Organização de diretórios | boa                         | **manter conceito**                  |
| `main.js`                 | ~1.500 linhas               | **principal dívida**                 |
| State                     | existe                      | **incompleto/inconsistente**         |
| Controller                | existe                      | **mistura responsabilidades**        |
| Renderer                  | existe                      | **DOM imperativo**                   |
| Components                | existem                     | **não são componentes declarativos** |
| API layer                 | boa                         | **preservar fronteira**              |
| Auth layer                | razoável                    | **centralizar estado**               |
| `ResultList.js`           | suspeito                    | **candidato a sobra**                |
| `SearchBar.js`            | pouco integrado             | **revisar**                          |
| V2 `ui/`                  | funcional/referência        | **preservar até migração**           |
| Vue                       | ausente                     | **migração necessária**              |
| TypeScript                | ausente                     | **migração necessária**              |
| Build frontend            | inexistente                 | **precisa ser definido**             |
| UX                        | já reproduzida parcialmente | **não redesenhar do zero**           |

---

# E agora chegamos ao ponto que queríamos desde o começo

As **cinco passadas estão concluídas**.

Temos agora uma visão fechada:

```text
PASSADA 1
estrutura
        ↓
PASSADA 2
backend
        ↓
PASSADA 3
contratos
        ↓
PASSADA 4
pipeline
        ↓
PASSADA 5
frontend
```

E, principalmente, conseguimos separar três coisas que antes estavam misturadas:

### O que já está conceitualmente correto

* arquitetura em camadas do backend;
* separação Document → Representation → RetrievalUnit;
* recuperação híbrida;
* separação RetrievalResult → Evidence;
* síntese baseada em evidências;
* preservação da experiência V2;
* ideia de frontend separado do backend.

### O que precisa ser refatorado

* `main.py`;
* contratos de domínio;
* `location`/proveniência;
* RRF;
* contextualização;
* publicação/atualização;
* estado e renderer do frontend;
* API/build do frontend.

### O que é candidato a remoção/substituição

* `src/`;
* `server.js`;
* `diff-main.txt`;
* configuração Node antiga;
* possíveis artefatos como `ResultList.js`/`SearchBar.js`;
* eventualmente `ui/`, **mas somente depois da migração V3**;
* documentação/arquivos históricos conforme o relatório final.

**E nada disso foi alterado no repositório.**

O próximo passo, portanto, **não é uma sexta passada**. É consolidar tudo isso no **relatório final das cinco passadas**, com uma lista fechada de `manter / refatorar / substituir / remover`, dependências e ordem de execução. Só depois disso você decide se autoriza alguma alteração efetiva.

Sim — e, na verdade, **eu acho que vale fazer essa análise**, mas eu não chamaria de uma sexta passada. Ela é um aprofundamento da **Passada 5**, porque até aqui eu analisei principalmente a arquitetura do frontend e só tratei a V2 como referência funcional. Não fiz uma crítica sistemática da **interface e da experiência de uso**.

E concordo contigo numa coisa importante: **não precisamos jogar fora a UI/UX da V2 só porque vamos trocar o framework**. O framework é implementação; a experiência é outra camada.

Vou tratar assim:

```text
V2
│
├── UI — o que o usuário vê e manipula
│
├── UX — como ele entende e executa a tarefa
│
└── fluxos — pesquisa → evidência → leitura → fonte
             ↓
       preservar o que funciona
             ↓
       corrigir o que precisa
             ↓
       Vue 3 + TypeScript
             ↓
           V3
```

E vou olhar especialmente para:

* **arquitetura da tela**: sidebar, área principal, painel de evidências, modo leitura;
* **hierarquia visual**;
* **fluxo de investigação**;
* **descoberta de acervos e seleção de contexto**;
* **apresentação da síntese versus evidências**;
* **transição entre evidência e documento original**;
* **modo de leitura**;
* **histórico**;
* **sincronização do Drive e feedback de operações**;
* **login/configurações**;
* **responsividade/mobile**;
* **tema e consistência visual**;
* pontos em que a V2/V3 ainda produz **fricção, ambiguidade ou carga cognitiva desnecessária**.

E uma distinção que acho particularmente importante para o Snoopy:

> A interface não deve parecer a de um “chatbot que responde perguntas”. Ela precisa comunicar que o pesquisador está **investigando um acervo documental**, e que a síntese é uma camada derivada das evidências.

Isso muda bastante a avaliação de UX.

Também não vou transformar isso numa discussão “React vs Vue”. Para o Snoopy, a questão é: **qual arquitetura de interface queremos preservar?** Depois Vue 3 implementa essa arquitetura. Sua preferência por React também não precisa ser descartada — Vue e React resolvem problemas semelhantes; a escolha aqui pode ser tratada pragmaticamente, sem transformar framework em religião. 😄

Então eu faria essa análise **agora, dentro da Passada 5**, antes do relatório final. E, principalmente, compararia a `ui/` V2 com o que a `frontend/` V3 já começou a reproduzir, para separar:

**1. o que a V2 acertou → preservar;
2. o que a V3 já melhorou → preservar;
3. o que ambas fazem de forma problemática → corrigir;
4. o que é puramente implementação → descartar na migração para Vue.**

Vou considerar isso como o fechamento da Passada 5, sem criar uma sexta etapa.


Agora sim. E depois de olhar a V2 especificamente como **produto/interface**, acho que dá para chegar a uma conclusão bem mais forte do que simplesmente “vamos reaproveitar a UI”.

**Sim: a V2 deve ser a base visual e de experiência da V3.** Não vejo motivo para redesenhar o Snoopy do zero. Mas também não recomendo uma cópia 1:1: há algumas decisões de UX que vale corrigir enquanto fazemos a migração para Vue 3.

## 1. Primeiro: a V2 tem uma identidade de produto bastante clara

A interface não parece um chatbot genérico.

Ela se organiza em torno de:

```text
┌──────────────┬───────────────────────────────┐
│              │                               │
│   ACERVO     │       INVESTIGAÇÃO            │
│              │                               │
│ histórico    │       síntese                 │
│ configurações│                               │
│ sincronização│       referências              │
│              │                               │
│              ├───────────────────────────────┤
│              │ evidências / trechos          │
└──────────────┴───────────────────────────────┘
```

Isso é muito coerente com o propósito do Snoopy.

O usuário não entra para conversar com uma IA. Ele:

> **seleciona um corpus → formula uma investigação → recebe uma síntese → examina evidências → volta ao documento.**

Essa sequência é provavelmente a coisa mais importante que a V3 deve preservar.

---

# 2. A home da V2 funciona justamente porque é simples

A tela inicial é praticamente:

> **LPP-Acervo**
> Memória Institucional e Recuperação Semântica Documental
>
> `[ Pesquisar ]`

Isso é bom.

Não existe uma tela inicial lotada de:

* cards;
* métricas;
* “insights”;
* explicações sobre IA;
* botões demais;
* dashboard artificial.

Para uma ferramenta acadêmica, essa sobriedade faz sentido.

### Eu manteria

A ideia de:

```text
Acervo selecionado
        ↓
Pesquisar
```

é uma boa entrada cognitiva.

### Mas há uma melhoria importante na V3

O usuário precisa entender **em qual acervo está pesquisando**.

Hoje isso está na sidebar, mas a relação entre:

> “Acervo Atual”

e

> “esta pesquisa”

poderia ficar mais explícita na tela de investigação.

Não precisa virar um banner gigante.

Algo discreto como:

```text
Minha investigação

Acervo: GEPAFOR
```

já resolveria.

Isso é particularmente importante porque o acervo não é simplesmente uma preferência visual: **ele determina o universo documental da investigação**.

---

# 3. A sidebar é uma das melhores decisões da V2

Ela concentra:

* identidade;
* nova pesquisa;
* acervo;
* sincronização;
* histórico;
* tema;
* autenticação;
* configurações.

Isso cria uma separação bastante intuitiva:

```text
SIDEBAR = contexto da aplicação
ÁREA CENTRAL = trabalho intelectual
```

Eu preservaria isso quase integralmente.

E o botão:

> **Nova Pesquisa**

é particularmente importante.

Ele fornece uma ação explícita para sair de uma investigação e começar outra, sem depender de “voltar” ou limpar manualmente o estado.

### Porém...

Há uma pequena questão conceitual:

**Acervo** e **Histórico** são atualmente elementos da mesma hierarquia visual.

Mas eles têm funções diferentes:

* Acervo = **contexto da pesquisa**
* Histórico = **recuperação de pesquisas anteriores**

Na V3 eu manteria ambos na sidebar, mas visualmente deixaria o primeiro mais claramente contextual e o segundo mais claramente navegacional.

---

# 4. O fluxo de investigação da V2 é o coração que devemos preservar

O fluxo é:

```text
Pesquisar
   ↓
carregando
   ↓
Síntese
   ↓
Referências do Acervo
   ↓
Trecho Original
   ↓
Ler no Contexto
   ↓
Documento
   ↓
PDF original
```

Isso é muito bom para o tipo de sistema que estamos construindo.

Porque cria uma cadeia cognitiva:

> **pergunta → resposta → evidência → contexto → fonte**

E isso conversa diretamente com aquela arquitetura metodológica que definimos.

Não é apenas uma decisão estética.

É uma decisão epistemológica da interface.

---

# 5. “Síntese” é melhor que “Resposta”

Eu manteria isso.

A V2 chama a área principal de:

> **Síntese:**

Isso é muito mais adequado ao Snoopy do que:

> “Resposta da IA”

ou

> “Resposta”.

Porque a interface já comunica implicitamente:

> isto é uma construção derivada da investigação documental.

E isso evita um dos problemas mais comuns de interfaces de RAG: fazer o usuário esquecer que o LLM está produzindo uma **síntese sobre fontes**, não uma fonte.

---

# 6. O painel “Trecho Original” é provavelmente o elemento de UX mais importante

A V2 tem:

> **Trecho Original**

e apresenta o conteúdo recuperado.

Isso é excelente porque a evidência não fica escondida atrás de uma interação secundária.

O pesquisador consegue ir:

```text
Síntese
   ↓
Referência
   ↓
Trecho Original
```

sem perder o contexto.

Eu **não substituiria isso por um chat com citações inline**, por exemplo.

Isso faria o Snoopy parecer muito mais um chatbot e menos uma ferramenta de pesquisa.

---

# 7. Mas aqui existe uma oportunidade importante para a V3

Atualmente a evidência é apresentada como:

```text
Evidência

[texto do trecho]

Documento · Página X
```

Isso funciona.

Mas a V3 poderia fazer uma distinção visual mais clara entre:

### conteúdo documental

> texto efetivamente recuperado do documento

e

### metadados da evidência

```text
Documento
Página
Localização
```

e eventualmente:

### ações

```text
Ler no contexto
Abrir fonte
```

Algo como:

```text
┌──────────────────────────────────────┐
│ Trecho Original                      │
│                                      │
│ “...texto documental...”             │
│                                      │
│ ───────────────────────────────────  │
│ Artigo X                             │
│ p. 14                                │
│                                      │
│ [Ler no contexto]                    │
└──────────────────────────────────────┘
```

Não muda a experiência.

Só deixa a hierarquia informacional mais explícita.

---

# 8. “Ler no Contexto” é uma excelente ideia — e merece mais destaque

Esse botão é, para mim, uma das decisões mais acertadas da V2.

Porque ele resolve uma diferença fundamental:

```text
Trecho recuperado ≠ documento inteiro
```

O pesquisador pode achar o trecho relevante, mas querer saber:

> “Tá, mas o que veio antes e depois?”

O modo de leitura responde justamente isso.

E isso conversa diretamente com o problema que encontramos na Passada 4:

> **o backend ainda não possui uma contextualização documental realmente robusta.**

Ou seja:

**a UX já pede uma capacidade que o backend ainda precisa amadurecer.**

Isso é uma informação arquitetural muito importante.

---

# 9. O modo de leitura deve ser preservado

A V2 possui uma tela separada:

```text
← Voltar

              Título do Documento

                              [PDF]
────────────────────────────────────────

              documento
              em leitura
```

Isso é conceitualmente diferente de simplesmente abrir um modal.

O usuário entra em outro **modo cognitivo**:

```text
Investigação
      ↓
Leitura
```

E isso é adequado.

Eu manteria esse conceito no Vue.

---

# 10. Mas o modo de leitura pode ficar muito melhor na V3

A V2 ainda trata a leitura como uma representação relativamente linear do documento.

A arquitetura que estamos construindo permite algo mais interessante:

```text
documento
   ↓
representação
   ↓
páginas
   ↓
blocos
   ↓
unidades
```

Então futuramente o leitor pode saber:

```text
Página 14

[conteúdo]

← anterior        próxima →
```

e, quando veio de uma evidência:

```text
Você está lendo o contexto
da evidência encontrada na página 14.
```

Isso transforma o modo de leitura em uma verdadeira **ponte entre recuperação e documento**, em vez de simplesmente uma tela de texto.

Mas isso depende da Passada 4 — especialmente da representação/localização.

---

# 11. O histórico da V2 é simples, e eu não sofisticaria demais

Ele funciona como:

```text
Histórico Recente

◷ pesquisa anterior
◷ outra pesquisa
◷ outra pesquisa
```

Isso é suficiente para a Alpha.

Eu **não colocaria ainda**:

* filtros complexos;
* pastas de pesquisas;
* tags;
* dashboards;
* favoritos;
* análise estatística.

O Snoopy não precisa virar um gerenciador de projetos de pesquisa.

O histórico deve responder apenas:

> “Quero voltar àquela investigação que fiz antes.”

---

# 12. A sincronização é onde a UX precisa amadurecer bastante

A V2 já teve uma boa intuição com o accordion:

```text
Sincronizando acervo...      ▼
```

e, quando aberto:

```text
arquivo A     PROCESSANDO
arquivo B     CONCLUÍDO
arquivo C     ...
```

Isso é melhor do que ocupar permanentemente a tela com uma barra de progresso.

Mas existe uma questão conceitual importante:

**sincronização é uma operação de infraestrutura, não a atividade principal do pesquisador.**

Portanto ela deveria ficar em segundo plano.

A UX ideal seria algo como:

```text
Acervo: GEPAFOR

✓ sincronizado
```

e, quando houver operação:

```text
↻ Sincronizando...
```

clicável para ver detalhes.

Isso evita transformar a sidebar em um painel de DevOps. 😂

---

# 13. A interface de acervo também está conceitualmente correta

A V2 faz uma coisa importante:

```text
Acervo Atual
[ GEPAFOR (Público) ▼ ]
```

Isso deixa explícito que a investigação acontece **dentro de um corpus**.

Eu preservaria.

Inclusive, isso reforça uma característica que diferencia o Snoopy de uma busca web comum:

> o pesquisador controla o conjunto documental sobre o qual está perguntando.

---

# 14. O login está corretamente subordinado ao trabalho

A V2 não transforma autenticação na primeira experiência do usuário.

O usuário pode entrar no ambiente e ver o acervo público.

Só quando precisa de recursos privados aparece a necessidade de autenticação.

Isso é uma boa decisão de UX.

Reduz a barreira inicial:

```text
abrir
 ↓
conhecer o sistema
 ↓
pesquisar
 ↓
autenticar quando necessário
```

em vez de:

```text
abrir
 ↓
login obrigatório
 ↓
talvez descobrir o que o sistema faz
```

Eu preservaria essa lógica se ela continuar compatível com os requisitos de segurança.

---

# 15. A identidade visual também merece ser preservada

A V2 tem uma identidade bem específica:

* fundo escuro;
* verde/sálvia como cor primária;
* Inter;
* bordas discretas;
* pouco contraste ornamental;
* cantos moderadamente arredondados;
* ícones Lucide;
* pouca ornamentação.

É uma espécie de:

> **ferramenta acadêmica contemporânea + terminal/arquivo institucional**

Isso combina bastante com o Snoopy.

Não vejo necessidade de transformar a V3 numa interface “SaaS startup” com:

```text
✨ AI Research Copilot
🚀 Ask anything
```

etc.

Inclusive, a estética atual ajuda a comunicar que **o documento é o objeto principal**, não a IA.

---

# 16. A V2 também acerta ao não exagerar nas cores

A paleta:

```text
background
panel
hover
text
muted
border
primary
```

é pequena.

Isso é bom.

A V3 deveria transformar isso em um sistema de design formal:

```text
tokens/
├── colors
├── typography
├── spacing
├── radius
├── shadows
└── transitions
```

não porque precisamos de uma mega design system, mas porque agora teremos Vue e componentes reutilizáveis.

---

# 17. Responsividade: a ideia é boa, mas precisa ser tratada como primeira classe

A V2 possui:

* sidebar colapsável;
* menu mobile;
* overlay;
* painel de evidências como espécie de drawer;
* botão de voltar;
* adaptações de leitura.

Isso mostra que a responsividade **já foi pensada**.

Mas o comportamento mobile não deve ser simplesmente:

> “desktop espremido”.

No mobile a hierarquia deveria ser:

```text
Pesquisa
   ↓
Síntese
   ↓
Evidência
   ↓
Leitura
```

um estado por vez.

A V2 já caminha nessa direção com o painel de evidência e o modo de leitura.

Portanto novamente:

**reaproveitar conceito, reimplementar mecanismo.**

---

# 18. Um detalhe importante: o painel de evidência é melhor entendido como “drawer”

Na desktop:

```text
Síntese │ Evidências
```

No mobile:

```text
Síntese
   ↓
[ Evidência ]
```

Isso significa que não deveríamos pensar no componente Vue como:

> `EvidencePanel`

apenas.

Ele é conceitualmente um:

> **Evidence Inspector / Evidence Drawer**

que muda de apresentação conforme o viewport.

Isso vai facilitar muito a arquitetura responsiva.

---

# 19. A V2 tem algumas pequenas inconsistências de interação

Aqui entra a parte crítica.

Não são motivos para jogar a UX fora, mas devem ser corrigidos na V3.

### Exemplo: navegação

A V2 usa simultaneamente:

* troca de views;
* classes CSS;
* `history`;
* hashes;
* `popstate`.

Isso funciona, mas é uma navegação manualmente coordenada.

Vue Router pode tornar isso muito mais explícito.

Algo conceitualmente como:

```text
/
  home

/investigation/:id
  investigation

/investigation/:id/evidence/:evidenceId
  evidence state

/document/:id
  reading
```

Não estou dizendo que precisamos necessariamente transformar cada coisa em uma URL dessa forma.

Mas a ideia é:

> **estado de navegação deve ser explícito, não distribuído entre DOM + hash + listeners.**

---

# 20. Outro problema: a V2 mistura estado persistente e estado efêmero

Exemplos:

### Persistente

```text
tema
acervo selecionado
sessão
histórico
```

### Efêmero

```text
painel de evidência aberto
evidência selecionada
loading
sincronização em andamento
modal aberto
```

A V3 deve separar esses conceitos.

Algo como:

```text
Application state
├── session
├── current folder
└── preferences

Investigation state
├── query
├── response
├── evidence
└── selected evidence

UI state
├── sidebar
├── modal
├── drawer
└── reading mode
```

Isso conversa diretamente com o que encontramos na análise estrutural do frontend.

---

# 21. E isso leva à decisão sobre Vue 3

Depois de olhar a interface como produto, eu estou **mais confortável ainda com Vue 3**.

Não porque Vue seja “melhor que React”.

Mas porque o problema do Snoopy é essencialmente:

```text
estado relativamente rico
+
componentes de interface
+
interfaces de leitura
+
painéis/drawers/modais
+
fluxos de aplicação
```

Vue 3 lida muito bem com isso.

E principalmente:

```text
HTML existente
        ↓
componentes Vue
        ↓
estado reativo
```

é uma transição conceitualmente tranquila.

Sua preferência por React é perfeitamente razoável — sobretudo porque você encontra React muito mais frequentemente. Mas **não há nada na UI do Snoopy que exija React especificamente**.

---

# 22. O que eu preservaria literalmente como princípio

### Preservar

**Estrutura**

```text
Sidebar
   +
Workspace
   +
Evidence Inspector
   +
Reading Mode
```

**Fluxo**

```text
Acervo
 ↓
Investigação
 ↓
Síntese
 ↓
Referências
 ↓
Evidência
 ↓
Contexto
 ↓
Documento original
```

**Conceitos**

* Síntese;
* Trecho Original;
* Ler no Contexto;
* Referências do Acervo;
* Histórico;
* Acervo Atual;
* sincronização discreta;
* leitura separada;
* acesso ao PDF original.

**Identidade**

* paleta;
* tipografia;
* densidade;
* sobriedade;
* iconografia;
* linguagem.

---

# 23. O que eu mudaria na V3

Principalmente:

```text
V2
DOM imperativo
   ↓
Vue 3
```

mas também:

### Estado

Centralizar e tipar.

### Navegação

Tornar explícita.

### Evidência

Separar melhor conteúdo, metadados e ações.

### Leitura

Usar a representação documental para contextualização real.

### Acervo

Deixar o contexto da pesquisa mais evidente.

### Sincronização

Tirar do centro da experiência.

### Mobile

Tratar cada modo como uma experiência própria.

### Acessibilidade

Aproveitar a reconstrução para corrigir:

* foco;
* teclado;
* labels;
* estados de loading;
* `aria`;
* contraste;
* botões somente com ícone.

### Feedback

Substituir `alert()` por componentes de feedback da aplicação.

---

# 24. Minha conclusão sobre a V2

Eu **não faria um redesign**.

Faria uma:

> **migração arquitetural com refinamento de UX.**

A diferença é enorme.

Não:

```text
V2 → jogar fora → criar V3
```

Nem:

```text
V2 → copiar HTML para Vue
```

Mas:

```text
                    ┌─ UI visual
V2 ────────────────┤
                    ├─ fluxos
                    ├─ hierarquia
                    ├─ conceitos
                    └─ linguagem
                           ↓
                    especificação UX/UI
                           ↓
                    Vue 3 + TypeScript
                           ↓
                         V3
```

E eu colocaria a seguinte regra para a implementação:

> **O Vue substitui a tecnologia de apresentação; não substitui a experiência de pesquisa que já foi construída.**

Isso fecha muito bem a Passada 5.

E, olhando agora para as cinco passadas em conjunto, a arquitetura fica bastante coerente: **backend, domínio, pipeline e frontend não precisam ser reinventados; precisam ser consolidados, corrigidos nos pontos problemáticos e finalmente transformados numa V3 coerente.**

Nenhuma alteração foi feita no repositório.
