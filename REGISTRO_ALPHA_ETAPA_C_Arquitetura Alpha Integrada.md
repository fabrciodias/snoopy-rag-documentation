# REGISTRO ALPHA DA ETAPA C

## Arquitetura Alpha Integrada

**Projeto:** Snoopy-RAG
**Versão:** V3 Alpha
**Etapa:** C — Arquitetura Alpha Integrada
**Status:** CONSOLIDADA / ENCERRADA
**Base:** Alpha A + Alpha B + Resgate do Fundamental + análise do V2

---

# 1. Objetivo da Etapa C

A Etapa C transforma as decisões consolidadas nas etapas anteriores em um **mapa técnico de implementação da V3 Alpha**.

A pergunta central desta etapa não é mais o que o Snoopy deve fazer, mas:

> **Como as capacidades já definidas serão materializadas como sistema, quais responsabilidades pertencem a cada componente, como os dados atravessam as fronteiras e em que ordem a Alpha deve ser implementada?**

C deve permitir que a implementação seja realizada sem que o implementador precise inventar silenciosamente decisões arquiteturais fundamentais.

A Alpha não pretende representar a arquitetura definitiva do Snoopy. Seu objetivo é materializar o **núcleo arquitetural que caracteriza efetivamente a V3**, comprimindo o escopo sem comprimir os princípios.

---

# 2. Princípios arquiteturais

A implementação da Alpha deve preservar as seguintes distinções fundamentais:

* documento ≠ representação documental;
* representação documental ≠ unidade de recuperação;
* unidade de recuperação ≠ embedding;
* recuperação ≠ análise;
* resultado de recuperação ≠ evidência;
* evidência ≠ fonte original;
* síntese ≠ fonte primária;
* `drive_file_id` ≠ `document_id`;
* `document_id` ≠ `unit_id`;
* `unit_id` ≠ `operation_id`;
* estado persistente ≠ cache;
* estado de domínio ≠ estado visual;
* StructuredResponse ≠ HTML;
* StructuredResponse ≠ Markdown;
* SSE ≠ fonte de verdade;
* frontend ≠ autoridade sobre estado persistente;
* framework frontend ≠ arquitetura do domínio.

O princípio geral é:

> **Algoritmos externos propõem; o Snoopy governa.**

Isso se aplica especialmente à representação, segmentação e recuperação.

---

# 3. Arquitetura integrada

A arquitetura Alpha é organizada nas seguintes fronteiras funcionais:

```text
SNOOPY V3 ALPHA

SOURCE / CORPUS
Google Drive
documentos
estados
       ↓
DOCUMENT PROCESSING
representação
estrutura
segmentação
publicação
       ↓
PERSISTENCE / INDEX
estado lógico
unidades
metadados
embeddings
índices
operações
       ↓
RETRIEVAL
filtros
lexical
semântico
fusão
deduplicação
contextualização
reranking
seleção
       ↓
INVESTIGATION / EVIDENCE
investigação
resultados
evidências
contexto
proveniência
síntese
       ↓
STRUCTURED RESPONSE
       ↓
PRESENTATION RENDERER
       ↓
VUE 3 + TYPESCRIPT
```

Essas fronteiras representam **responsabilidades internas da aplicação**. Não implicam microserviços nem infraestrutura distribuída.

Segurança e autorização atravessam todas as fronteiras.

Operações assíncronas, estados e comunicação constituem capacidades transversais.

---

# 4. Separação dos dois fluxos fundamentais

A Alpha mantém dois fluxos distintos que se encontram nas unidades recuperáveis e nas evidências.

## 4.1 Fluxo documental

```text
Google Drive
    ↓
Document
    ↓
Operation
    ↓
DocumentRepresentation
    ↓
validação estrutural
    ↓
SegmentationService
    ↓
RetrievalUnit[]
    ↓
índice lexical + vetorial
    ↓
validação de publicação
    ↓
ACTIVE
```

O fluxo documental é responsável por transformar a fonte documental em conteúdo que possa participar da recuperação.

## 4.2 Fluxo investigativo

```text
Usuário
    ↓
query original + filtros
    ↓
Investigation
    ↓
Retrieval
    ↓
RetrievalResult[]
    ↓
Evidence[]
    ↓
contexto
    ↓
Synthesis
    ↓
StructuredResponse
    ↓
PresentationRenderer
    ↓
Interface
    ↓
Evidence Drawer
    ↓
Reading / Source
    ↓
retorno à Investigation
```

O sistema documental não se torna proprietário da investigação, e a investigação não passa a ser proprietária da representação documental.

---

# 5. Componentes e responsabilidades

## 5.1 Backend — domínio e aplicação

| Componente                | Responsabilidade                                                  |
| ------------------------- | ----------------------------------------------------------------- |
| **Document Service**      | Identidade, existência e estado dos documentos                    |
| **Document Processor**    | Transformação da fonte em `DocumentRepresentation`                |
| **Segmentation Service**  | Segmentação governada em `RetrievalUnit[]`                        |
| **Publication Service**   | Validação e publicação coerente do documento                      |
| **Corpus Service**        | Estado e operações do corpus                                      |
| **Retrieval Service**     | Cadeia completa de recuperação                                    |
| **Evidence Service**      | Transformação de resultados selecionados em evidências navegáveis |
| **Investigation Service** | Contexto da investigação e coordenação de seus resultados         |
| **Synthesis Service**     | Síntese baseada em evidências e contexto                          |
| **Response Builder**      | Construção do `StructuredResponse`                                |
| **Operation Service**     | Identificação e acompanhamento de operações assíncronas           |

### Distinções obrigatórias

**Document Service ≠ Document Processor**

O primeiro administra o documento como entidade e seu estado. O segundo executa o processamento necessário para produzir a representação documental.

**Segmentation Service ≠ Document Processor**

O processor produz a representação documental. A segmentação transforma essa representação em unidades de recuperação.

**Retrieval Service ≠ Investigation Service**

O Retrieval recupera unidades. A Investigation mantém o contexto da investigação na qual essa recuperação ocorre.

**RetrievalResult ≠ Evidence**

Uma unidade recuperada pode ser um resultado sem ser automaticamente tratada como evidência.

**SynthesisService ≠ Retrieval**

A síntese recebe evidências/contexto e produz conteúdo derivado. Ela não redefine a recuperação.

---

# 6. Infraestrutura

A infraestrutura fornece as capacidades externas utilizadas pela aplicação:

```text
Infrastructure
├── DriveAdapter
├── PostgreSQL repositories
├── pgvector
├── PostgreSQL FTS
├── EmbeddingProvider
├── RerankerProvider
├── LLMProvider
├── Queue
├── SSE Publisher
└── Storage
```

Responsabilidades:

| Componente            | Responsabilidade                            |
| --------------------- | ------------------------------------------- |
| **DriveAdapter**      | Acesso à fonte original                     |
| **Repositories**      | Persistência e recuperação do estado lógico |
| **pgvector**          | Busca vetorial                              |
| **PostgreSQL FTS**    | Busca lexical                               |
| **EmbeddingProvider** | Geração de embeddings                       |
| **RerankerProvider**  | Reranking dos candidatos                    |
| **LLMProvider**       | Síntese/modelo de linguagem                 |
| **Queue**             | Execução assíncrona                         |
| **SSE Publisher**     | Comunicação de mudanças ao frontend         |
| **Storage**           | Artefatos derivados quando necessários      |

A aplicação deve depender de capacidades, não de detalhes concretos de infraestrutura quando houver uma fronteira real.

Não se deve criar abstrações apenas por antecipação. A abstração deve corresponder a uma responsabilidade ou dependência externa efetiva.

---

# 7. Modelo mínimo de dados

A Alpha trabalha conceitualmente com as seguintes entidades principais:

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

## 7.1 Identidades

```text
drive_file_id       → identidade externa no Google Drive
document_id         → identidade interna do documento
representation_id   → identidade da representação
unit_id             → identidade da unidade de recuperação
investigation_id    → identidade da investigação
result_id           → identidade do resultado
evidence_id         → identidade da evidência
operation_id        → identidade da operação
response_id         → identidade da resposta estruturada
```

Essas identidades não devem ser confundidas.

## 7.2 Corpus

`Corpus` representa o universo lógico de documentos disponível para determinada operação/contexto.

Na Alpha, isso não exige necessariamente uma entidade persistente independente. O corpus não deve ser transformado em tabela própria apenas por existir como conceito arquitetural.

## 7.3 Source

`Source` representa a relação de origem documental.

O Google Drive constitui a fonte original na Alpha, mas isso não exige uma entidade `Source` independente.

O documento deve manter informação suficiente para relacioná-lo à sua fonte original.

---

# 8. DocumentRepresentation

`DocumentRepresentation` é a representação documental canônica utilizada pelo Snoopy para preservar a estrutura mínima necessária à recuperação, evidência e localização.

Ela deve preservar, no mínimo:

* identidade da representação;
* relação com o documento;
* páginas;
* blocos;
* texto;
* ordem de leitura;
* localização;
* estrutura básica;
* metadados necessários à reconstrução.

Conceitualmente:

```text
DocumentRepresentation
├── pages
│   ├── page
│   │   ├── block
│   │   ├── block
│   │   └── ...
│   └── ...
└── metadata
```

A representação documental **não é**:

* Markdown;
* chunk;
* embedding;
* HTML;
* saída de LLM.

Markdown pode existir como representação derivada.

## Fora da Alpha

Não bloqueiam a Alpha:

* OCR;
* multimodalidade completa;
* reconstrução tipográfica;
* tabelas avançadas;
* equações;
* reconstrução espacial sofisticada.

---

# 9. RetrievalUnit e segmentação

A segmentação transforma a representação documental em unidades identificáveis de recuperação:

```text
DocumentRepresentation
        ↓
SegmentationService
        ↓
RetrievalUnit[]
```

Cada unidade deve possuir identidade e relação suficiente com sua origem:

```text
unit_id
document_id
representation_id
location
content
context
metadata
```

A unidade não deve depender da existência de um embedding específico para possuir identidade.

O algoritmo de segmentação pode reutilizar componentes do V2, mas o `chunker.py` não deve simplesmente ser renomeado para representar a arquitetura V3.

A segmentação é governada pelo Snoopy.

---

# 10. Estados documentais

Os estados mínimos são:

```text
PENDING
PROCESSING
ACTIVE
FAILED
REJECTED
REMOVED
```

Transições principais:

```text
PENDING
   ├→ PROCESSING
   └→ REJECTED

PROCESSING
   ├→ ACTIVE
   ├→ FAILED
   └→ REJECTED

ACTIVE
   ├→ PROCESSING
   └→ REMOVED

FAILED
   └→ PROCESSING

REMOVED
   └→ PENDING
```

Somente documentos `ACTIVE` participam da recuperação.

A publicação de um documento deve ocorrer somente depois que o processamento e os índices necessários tiverem sido validados.

---

# 11. Operações assíncronas

`Operation` generaliza a noção de execução assíncrona.

Estados:

```text
PENDING
PROCESSING
COMPLETED
FAILED
CANCELLED
```

Estados terminais:

```text
COMPLETED
FAILED
CANCELLED
```

Uma operação possui, no mínimo:

```text
operation_id
operation_type
target/reference
status
created_at
started_at
finished_at
error (quando aplicável)
```

Retry gera nova `operation_id`. A operação anterior permanece registrada como operação concluída/terminal.

Timeout ou perda de comunicação não significa automaticamente que a operação falhou ou foi cancelada.

---

# 12. Fluxo documental completo

A arquitetura documental da Alpha é:

```text
Google Drive
      ↓
DriveAdapter
      ↓
Document
      ↓
Operation
      ↓
DocumentProcessor
      ↓
DocumentRepresentation
      ↓
validação
      ↓
SegmentationService
      ↓
RetrievalUnit[]
      ↓
EmbeddingProvider
      ↓
pgvector

RetrievalUnit[]
      ↓
PostgreSQL FTS

      ↓
validação de publicação
      ↓
PublicationService
      ↓
ACTIVE
```

A publicação deve ser coerente e atômica do ponto de vista lógico.

Não deve ocorrer:

```text
Document publicado
       ↓
processamento incompleto
       ↓
documento parcialmente pesquisável
```

A regra é:

```text
processar
   ↓
validar
   ↓
indexar
   ↓
publicar
   ↓
ACTIVE
```

Falhas não devem resultar na publicação de estado documental incompleto.

---

# 13. Persistência

A Alpha distingue:

```text
estado lógico persistente
        ≠
índice derivado
        ≠
artefato físico derivado
        ≠
fonte original
```

O Google Drive permanece como fonte documental original.

PostgreSQL/Supabase constitui a persistência lógica principal.

`pgvector` e PostgreSQL FTS são estruturas derivadas utilizadas para recuperação.

Arquivos físicos derivados podem ser armazenados temporariamente quando necessários ao processamento.

A representação documental deve permanecer recuperável/utilizável pelo sistema, mas a forma física exata de armazenamento não é uma decisão arquitetural adicional desta etapa.

Não há, na Alpha:

* versionamento completo;
* snapshots;
* histórico completo de documentos;
* grafo formal de proveniência.

---

# 14. Retrieval híbrido

A fronteira do Retrieval recebe uma requisição conceitual:

```text
RetrievalRequest
├── investigation_id
├── query
└── filters
```

A cadeia é:

```text
RetrievalRequest
      ↓
filtros estruturados
      ↓
corpus elegível
      ↓
┌──────────────┬──────────────┐
│              │              │
lexical     semantic          │
│              │              │
└──────┬───────┘              │
       ↓
     fusion
       ↓
     dedup
       ↓
contextualização
       ↓
   reranking
       ↓
    seleção
       ↓
RetrievalResult[]
```

O Retrieval:

* não interpreta metodologicamente o conteúdo;
* não determina verdade;
* não determina importância metodológica;
* não transforma automaticamente todo resultado em evidência;
* não produz a fonte original.

---

# 15. RetrievalResult

O resultado da recuperação pertence à investigação:

```text
RetrievalResult
├── result_id
├── investigation_id
├── unit_id
├── rank
├── retrieval_score
└── retrieval metadata
```

Scores de recuperação não representam:

* importância metodológica;
* qualidade da evidência;
* verdade;
* relevância epistemológica definitiva.

A unidade selecionada pode então ser resolvida pelo `EvidenceService`.

---

# 16. Evidence

A evidência possui identidade própria:

```text
Evidence
├── evidence_id
├── investigation_id
├── unit_id
├── document_id
├── location
├── content
├── context
└── provenance
```

A proveniência mínima permite reconstruir:

```text
evidence
   ↓
unit
   ↓
document
   ↓
location
   ↓
original source
```

Quando necessário, a reconstrução operacional também relaciona:

```text
investigation_id
operation_id
result_id
```

A evidência é navegável e pode ser reaberta independentemente de sua posição original na lista de resultados.

A Alpha não implementa um grafo formal de proveniência.

---

# 17. Investigação e consulta

A consulta original é preservada na `Investigation`.

```text
Investigation
├── investigation_id
├── original_query
├── filters
├── results
├── evidences
└── synthesis/response
```

Consultas derivadas para busca lexical, semântica ou outras operações internas não substituem a consulta original.

Filtros estruturados permanecem distintos da relevância textual.

---

# 18. Synthesis

A síntese ocorre depois de Retrieval e Evidence:

```text
Retrieval
   ↓
Evidence
   ↓
Context
   ↓
Synthesis
```

A síntese é conteúdo derivado.

Ela não é fonte primária nem substitui as evidências ou os documentos originais.

O LLM é acessado por meio de `LLMProvider`.

---

# 19. StructuredResponse

A resposta da investigação é representada semanticamente por `StructuredResponse`.

Conceitualmente:

```text
StructuredResponse
├── content
├── sections
├── evidence_refs
├── references
└── navigation
```

A estrutura exata do contrato pode ser refinada na implementação sem alterar sua função arquitetural.

A regra fundamental é:

```text
Evidence
   ↓
Synthesis
   ↓
StructuredResponse
   ↓
PresentationRenderer
   ↓
Vue components
```

`StructuredResponse` não é HTML nem Markdown.

Markdown pode existir como formato derivado para exportação, cópia ou interoperabilidade.

O `StructuredResponse` deve ser recuperável ou reconstruível a partir da identidade da investigação, sem depender da continuidade de uma conexão SSE. Isso não exige uma tabela persistente exclusiva para a resposta.

---

# 20. Backend ↔ Frontend

A fronteira é:

```text
Vue 3 + TypeScript
        ↕
     HTTP / SSE
        ↕
Backend
```

O backend é autoridade sobre:

* estado persistente;
* autorização;
* operações;
* documentos;
* investigações;
* evidências;
* corpus.

O frontend é responsável por:

* apresentação;
* estado visual;
* estado local;
* interação;
* consumo dos contratos.

O frontend não deve conhecer:

* estrutura interna do PostgreSQL;
* pgvector;
* implementação do reranker;
* embeddings;
* filas;
* workers;
* credenciais do Drive;
* detalhes internos do LLM;
* representação documental interna quando não necessária ao contrato.

---

# 21. Comunicação

A comunicação da Alpha utiliza quatro categorias principais:

1. consultas;
2. comandos;
3. operações assíncronas identificadas por `operation_id`;
4. SSE para notificações de mudança.

O contrato conceitual de pesquisa é:

```text
POST /investigations/search

{
  query,
  filters
}

→

{
  investigation_id,
  operation_id,
  status
}
```

Os endpoints exatos permanecem uma decisão de implementação, mas os contratos devem preservar:

* identidade;
* entrada;
* saída;
* estado;
* erros;
* recuperabilidade.

Contratos mínimos:

| Operação           | Entrada                           | Saída                               |
| ------------------ | --------------------------------- | ----------------------------------- |
| Search             | query + filters                   | `investigation_id` + `operation_id` |
| Get Investigation  | `investigation_id`                | investigação/resultados/evidências  |
| Get Operation      | `operation_id`                    | estado da operação                  |
| Get Evidence       | `evidence_id`                     | evidência + proveniência            |
| Get Source/Reading | documento + localização/evidência | contexto documental                 |
| List Corpus        | filtros                           | documentos/estados                  |
| Sync Corpus        | parâmetros                        | `operation_id`                      |
| SSE                | conexão                           | notificações de mudança             |

---

# 22. SSE e recuperação

SSE é um **canal de atualização**, não a fonte de verdade.

A sequência é:

```text
persistir estado
      ↓
publicar atualização
      ↓
SSE
```

Se o SSE for interrompido:

```text
SSE desconectado
      ↓
reconectar
      ↓
GET / revalidação
      ↓
estado atual
```

A perda da conexão não deve ser interpretada como falha da operação.

Não haverá:

* streaming obrigatório de toda a resposta estruturada;
* WebSockets quando SSE for suficiente;
* sistema offline-first;
* filas no frontend;
* mecanismo complexo de sincronização cliente-servidor.

---

# 23. Estados do frontend

O frontend separa quatro domínios de estado:

```text
INVESTIGATION
CORPUS
OPERATIONS
UI
```

### Investigation

Mantém:

* query;
* filtros;
* resultados;
* evidências;
* evidência explorada;
* contexto;
* relação com fonte;
* retorno.

### Corpus

Mantém o estado apresentado do acervo e dos documentos.

### Operations

Mantém operações identificadas independentemente:

```text
search #123 → PROCESSING
sync #456   → PROCESSING
```

Não existe um `isLoading` global como autoridade.

### UI

Mantém:

* drawer;
* foco;
* menus;
* tabs;
* tema;
* detalhes visuais.

Cache não é tratado como estado de domínio.

---

# 24. Interface

O frontend da Alpha utiliza:

**Vue 3 + TypeScript**

A escolha é operacional e não constitui uma decisão definitiva sobre o futuro do Snoopy.

A arquitetura deve manter as fronteiras de:

* domínio;
* infraestrutura;
* contratos;
* representação;
* apresentação

independentes do framework.

A estrutura funcional mínima é:

```text
Frontend
├── Investigation
│   ├── Query
│   ├── Filters
│   ├── Results
│   ├── Synthesis
│   └── EvidenceDrawer
│
├── Reading
├── Corpus
├── Operations
├── Renderer
└── Infrastructure
    ├── HTTP
    └── SSE
```

Vue Router pode ser utilizado para navegação.

O roteamento não é a autoridade sobre o estado da investigação.

---

# 25. Acessibilidade e resiliência

A acessibilidade deve ser incorporada desde a construção dos componentes.

Critérios mínimos da Alpha:

* HTML semântico;
* elementos nativos quando apropriados;
* navegação razoável por teclado;
* estados não dependentes exclusivamente de cor;
* foco adequado ao abrir/fechar Evidence Drawer;
* retorno funcional à investigação.

Não há exigência de certificação WCAG completa na Alpha.

A interface deve distinguir:

* esperando;
* processando;
* concluído;
* falha;
* cancelamento;
* perda de comunicação;
* reconexão.

Não devem existir percentuais falsos de progresso.

Não haverá retry automático indiscriminado.

Retries automáticos são reservados a operações claramente idempotentes e seguras; o padrão para operações ambíguas é retry manual.

---

# 26. Segurança

Todo conteúdo externo ou produzido por processamento/modelo é tratado como **dado não confiável**.

Isso inclui:

* documentos;
* texto recuperado;
* conteúdo de evidência;
* conteúdo gerado por LLM;
* Markdown;
* metadados provenientes da fonte.

Não deve haver injeção arbitrária de HTML.

Quando Markdown for utilizado:

```text
Markdown
   ↓
parser
   ↓
HTML
   ↓
sanitização
   ↓
DOM
```

A autorização é responsabilidade do backend.

RLS pode atuar como defesa em profundidade, mas não substitui autorização de aplicação.

A service role do Supabase não deve ser tratada como mecanismo de segurança por si só.

Erros apresentados ao usuário não devem expor:

* stack traces;
* credenciais;
* caminhos internos;
* segredos;
* detalhes internos desnecessários.

---

# 27. Stack da Alpha

A arquitetura tecnológica consolidada é:

| Área                     | Tecnologia                                      |
| ------------------------ | ----------------------------------------------- |
| Frontend                 | **Vue 3 + TypeScript**                          |
| Roteamento               | Vue Router                                      |
| Comunicação              | HTTP/REST + SSE                                 |
| Persistência             | PostgreSQL / Supabase                           |
| Busca lexical            | PostgreSQL Full-Text Search                     |
| Busca vetorial           | pgvector                                        |
| Processamento assíncrono | Supabase Queues / PGMQ + Python workers         |
| Fonte documental         | Google Drive                                    |
| Embeddings               | `EmbeddingProvider`                             |
| Reranking                | `RerankerProvider`                              |
| Síntese                  | `LLMProvider`                                   |
| Apresentação             | StructuredResponse → PresentationRenderer → Vue |

A implementação permanece como **backend único com workers assíncronos**, não como arquitetura de microserviços.

Não são necessários para a Alpha:

* Elasticsearch/OpenSearch;
* Kubernetes;
* Kafka;
* microservices;
* API Gateway;
* CQRS;
* event sourcing;
* infraestrutura distribuída;
* multi-região.

---

# 28. Relação com o V2

O V2 é tratado como **substrato de migração**, não como arquitetura a ser preservada integralmente.

Mapeamento principal:

| V2                 | V3 Alpha                             |
| ------------------ | ------------------------------------ |
| `server.js`        | Backend/Application + Infrastructure |
| `ui/*.js`          | Vue 3 + TypeScript                   |
| `pipeline.py`      | Document Processor + Publication     |
| `extractor.py`     | Document Representation              |
| `cleaner.py`       | processamento/normalização           |
| `tagger.py`        | extração de metadados                |
| `chunker.py`       | Segmentation Service — adaptado      |
| `search.py`        | Retrieval + Synthesis separados      |
| `jobs`             | Operations + Queue                   |
| `worker.py`        | Python Worker                        |
| `chunks`           | RetrievalUnits — modelo revisado     |
| `match_chunks()`   | parte do Retrieval                   |
| `[TRECHO X]`       | Evidence IDs / StructuredResponse    |
| `documents`        | Document                             |
| `drive_file_id`    | identidade externa preservada        |
| `raw_pdfs/`        | artefato temporário                  |
| `reading-view`     | Reading/Source                       |
| `evidence-section` | Evidence Drawer                      |
| `folder_id`        | fronteira/contexto do Corpus         |

O reaproveitamento é permitido quando a implementação existente for compatível com as fronteiras V3.

Quando houver conflito, a fronteira V3 prevalece.

---

# 29. Ordem de implementação

Existem três **pré-condições operacionais** anteriores à implementação:

1. estabelecer o baseline do V2;
2. mapear o schema real do banco V2, incluindo tabelas, RPCs, índices e políticas relevantes;
3. rotacionar credenciais que tenham sido expostas durante o desenvolvimento.

Essas ações não constituem etapas da arquitetura V3.

A sequência arquitetural é:

```text
1. Entidades + estados + Operation
                    ↓
2. DocumentRepresentation
                    ↓
3. Segmentation → RetrievalUnit
                    ↓
4. Indexação + publicação atômica → ACTIVE
                    ↓
5. Retrieval híbrido
                    ↓
6. Evidence + provenance
                    ↓
7. Synthesis → StructuredResponse
                    ↓
8. Vue 3 + TypeScript + Renderer
                    ↓
9. Investigation → Evidence → Reading
                    ↓
10. SSE + recuperação/revalidação
                    ↓
11. Vertical slice completo
                    ↓
12. Testes de falha + fechamento da Alpha
```

Frontend e outras partes podem avançar em paralelo quando seus contratos já estiverem suficientemente definidos.

---

# 30. Vertical Slice da Alpha

O primeiro vertical slice completo deve materializar:

```text
Documento no Google Drive
        ↓
Document
        ↓
Operation
        ↓
DocumentRepresentation
        ↓
Segmentation
        ↓
RetrievalUnit
        ↓
Embedding + FTS
        ↓
ACTIVE
        ↓
Investigation
        ↓
Retrieval híbrido
        ↓
RetrievalResult
        ↓
Evidence
        ↓
Context
        ↓
Synthesis
        ↓
StructuredResponse
        ↓
Renderer
        ↓
Interface
        ↓
Evidence Drawer
        ↓
Reading / Source
        ↓
Retorno à Investigation
```

Esse fluxo constitui o núcleo funcional da Alpha.

A Alpha não precisa conter todas as funcionalidades futuras do Snoopy para ser considerada uma evolução arquitetural real.

---

# 31. Critério de conclusão da Alpha

A Alpha será considerada concluída quando o sistema demonstrar, de ponta a ponta:

## Corpus

* documento identificado a partir do Drive;
* operação de processamento identificável;
* `DocumentRepresentation` produzida;
* `RetrievalUnit`s governadas;
* índice lexical e vetorial;
* publicação somente após processamento válido;
* ausência de publicação parcial.

## Investigação

* consulta original preservada;
* filtros distintos da consulta;
* `Investigation` identificável;
* `Operation` associada;
* recuperação lexical e semântica;
* resultados identificáveis;
* resultado distinto de evidência.

## Evidência

* `evidence_id`;
* unidade e documento identificáveis;
* localização;
* contexto;
* proveniência mínima;
* reabertura independente da posição original.

## Síntese

* baseada em evidências/contexto;
* tratada como conteúdo derivado;
* `StructuredResponse`;
* referências semanticamente identificáveis.

## Interface

* Investigation;
* Results;
* Evidence Drawer;
* Reading/Source;
* retorno à investigação;
* renderer estruturado;
* estados assíncronos;
* falha;
* cancelamento;
* perda de comunicação;
* reconexão/revalidação.

## Segurança

* conteúdo tratado como dado não confiável;
* ausência de HTML arbitrário;
* autorização no backend;
* ausência de vazamento de credenciais ou detalhes internos.

---

# 32. Limites explícitos da Alpha

As seguintes capacidades permanecem **pós-Alpha** e não devem bloquear sua conclusão:

* OCR;
* representação digital avançada;
* multimodalidade completa;
* tratamento avançado de tabelas;
* múltiplos perfis de segmentação;
* múltiplos segmentadores;
* múltiplos rerankers;
* MMR;
* decomposição sofisticada de consultas;
* avaliação formal de retrieval;
* versionamento completo;
* snapshots;
* histórico completo;
* proveniência formal/completa;
* múltiplas investigações avançadas;
* colaboração;
* infraestrutura distribuída;
* observabilidade avançada;
* disaster recovery;
* offline-first;
* GraphQL;
* WebSockets quando SSE for suficiente;
* event sourcing;
* CQRS;
* microservices;
* API Gateway;
* arquitetura distribuída complexa.

Esses itens não são decisões rejeitadas definitivamente; são capacidades deliberadamente deslocadas para uma etapa posterior.

---

# 33. Critério arquitetural central

A Alpha deve cumprir o seguinte princípio:

> **Implementar o mínimo que materializa a arquitetura nova, e não o mínimo que faz uma demonstração parecer nova.**

Consequentemente:

> **Comprimir o escopo, não os princípios.**

Uma interface nova sobre o mesmo modelo documental, o mesmo chunking, o mesmo fluxo de publicação parcial e o mesmo acoplamento do V2 não constitui a V3 Alpha definida nesta etapa.

A Alpha precisa demonstrar que as novas fronteiras existem de fato no sistema.

---

# 34. Decisão consolidada da Etapa C

> **A implementação da Alpha será conduzida como uma migração arquitetural progressiva do V2 para o núcleo V3, priorizando primeiro as entidades, estados e fluxo documental; em seguida, segmentação, publicação/indexação, recuperação, evidência e síntese; e finalmente a materialização desses contratos no frontend Vue 3 + TypeScript. O V2 será utilizado como substrato de reaproveitamento, mas suas estruturas internas não serão preservadas quando conflitarem com as fronteiras V3.**

> **O critério de conclusão da Alpha é um vertical slice funcional completo — documento → representação → segmentação → indexação → ACTIVE → investigação → recuperação híbrida → evidência → síntese → StructuredResponse → renderer → interface → fonte/leitura → retorno — com estados assíncronos, proveniência mínima, publicação coerente e recuperação após perda de comunicação.**

> **A Alpha não será bloqueada por capacidades explicitamente classificadas como pós-Alpha.**

---

# 35. Encerramento da Etapa C

Com C.1–C.10 consolidadas, a Etapa C deixa de ser uma etapa de decisão arquitetural e passa a constituir a **especificação de implementação da V3 Alpha**.

A arquitetura resultante integra:

```text
REPRESENTAÇÃO DOCUMENTAL
        ↓
SEGMENTAÇÃO GOVERNADA
        ↓
RECUPERAÇÃO HÍBRIDA
        ↓
EVIDÊNCIA
        ↓
PROVENIÊNCIA MÍNIMA
        ↓
SÍNTESE
        ↓
STRUCTURED RESPONSE
        ↓
PRESENTATION RENDERER
```

sobre as capacidades transversais de:

```text
IDENTIDADE
ESTADOS
PERSISTÊNCIA
PROCESSAMENTO
OPERAÇÕES
COMUNICAÇÃO
SEGURANÇA
```

e materializa a experiência:

```text
INVESTIGATION
      ↓
RESULTS
      ↓
EVIDENCE
      ↓
CONTEXT
      ↓
SOURCE / READING
      ↓
RETURN
```

**Status da Etapa C: CONSOLIDADA E ENCERRADA.**

**Próxima finalidade do registro:** servir como mapa técnico para a implementação da V3 Alpha, sem reabrir decisões já consolidadas e sem permitir que o implementador substitua silenciosamente as fronteiras arquiteturais definidas.
