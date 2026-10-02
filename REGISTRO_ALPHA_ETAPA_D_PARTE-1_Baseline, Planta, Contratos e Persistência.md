# REGISTRO ALPHA — ETAPA D

## Parte I — Baseline, Planta, Contratos e Persistência

**Projeto:** Snoopy-RAG
**Etapa:** D — Mapa de Implementação
**Parte:** I
**Escopo consolidado:** D.1, D.2, D.3 e D.4
**Status:** CONSOLIDADO / FECHADO
**Próxima etapa:** D.5 — Contratos de API e Comunicação

---

# 1. Objetivo deste registro

Este registro consolida a primeira parte da discussão da Etapa D, transformando as decisões tomadas em D.1, D.2, D.3 e D.4 em uma referência única para a continuidade do Mapa de Implementação.

A Etapa D parte das decisões arquiteturais consolidadas nas etapas anteriores e tem como finalidade preparar a implementação da V3 Alpha.

A implementação ainda não é realizada neste registro.

O objetivo é estabelecer, de maneira suficientemente concreta, **de onde o sistema parte, como será organizado, quais entidades compõem seu núcleo e como seu estado será persistido**.

A regra geral permanece:

> **O V2 é substrato de migração, não arquitetura a ser preservada integralmente.**

---

# 2. D.1 — Baseline e estratégia de migração

## 2.1 Baseline

O V2 existente constitui o substrato material da migração para a V3 Alpha.

O baseline inclui, entre outros elementos:

* Node/Express;
* frontend Vanilla JavaScript;
* Python para processamento e busca;
* PostgreSQL/Supabase;
* pgvector;
* Google Drive;
* processamento assíncrono;
* SSE;
* autenticação e autorização;
* estruturas documentais e de acervo existentes.

O banco real do V2 é tratado como **estado existente a ser mapeado**, não como modelo-alvo da V3.

---

## 2.2 Regra fundamental de migração

A migração deve ser orientada por responsabilidade e significado, e não por simples correspondência nominal entre estruturas.

Assim:

```text
V2 chunks        ≠ V3 RetrievalUnit
V2 jobs          ≠ V3 Operation
V2 match_chunks  ≠ V3 Retrieval
V2 content       ≠ V3 DocumentRepresentation
V2 folders       ≠ entidade Corpus obrigatória
V2 server.js     ≠ arquitetura V3
```

Cada estrutura do V2 deve ser classificada conforme sua função na V3:

* preservar;
* adaptar;
* substituir;
* remover;
* validar antes de decidir.

---

## 2.3 Estratégia progressiva

A migração seguirá a sequência arquitetural definida em C.10:

```text
Baseline
→ entidades/estados/Operation
→ DocumentRepresentation
→ Segmentation/RetrievalUnit
→ Indexação/Publicação/ACTIVE
→ Retrieval híbrido
→ Evidence/Proveniência
→ Synthesis/StructuredResponse
→ Vue + TypeScript + Renderer
→ Investigation/Evidence/Reading
→ SSE/recuperação
→ vertical slice
→ testes/fechamento
```

O código V2 poderá permanecer temporariamente quando sua responsabilidade for compatível com a V3 ou quando estiver funcionando como ponte explícita de migração.

Entretanto, nenhuma estrutura V2 deve continuar controlando uma responsabilidade que conflite com uma fronteira V3 já estabelecida.

---

# 3. D.2 — Planta do projeto

## 3.1 Princípio

O projeto será organizado por **responsabilidades arquiteturais**, e não simplesmente por tecnologia.

A tecnologia materializa as responsabilidades; ela não define, sozinha, as fronteiras do sistema.

A estrutura conceitual estabelecida é:

```text
snoopy-rag/
│
├── backend/
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   ├── api/
│   └── main.*
│
├── workers/
│   ├── document/
│   ├── retrieval/
│   ├── synthesis/
│   └── main.*
│
├── frontend/
│   ├── app/
│   ├── features/
│   ├── components/
│   ├── renderer/
│   ├── infrastructure/
│   └── main.*
│
├── shared/
│   ├── contracts/
│   └── representations/
│
├── migrations/
├── tests/
└── scripts/
```

Essa árvore é uma **planta conceitual**.

Ela não implica a criação automática de múltiplos serviços, processos ou abstrações para cada diretório.

---

## 3.2 Backend

O backend é dividido conceitualmente em:

```text
Domain
Application
Infrastructure
API
```

### Domain

Contém os conceitos e regras fundamentais do Snoopy.

Não depende diretamente de:

* PostgreSQL;
* Supabase;
* Google Drive;
* Vue;
* pgvector;
* LLM;
* SSE.

### Application

Orquestra os casos de uso do sistema.

Entre eles:

* processamento documental;
* segmentação;
* publicação;
* busca/investigação;
* construção de evidências;
* síntese;
* sincronização do corpus;
* recuperação de investigação;
* recuperação de evidência;
* operações assíncronas.

### Infrastructure

Contém as implementações concretas dos mecanismos externos:

* PostgreSQL/Supabase;
* Drive;
* embeddings;
* reranker;
* LLM;
* queue;
* busca;
* SSE;
* armazenamento de artefatos.

### API

Materializa os contratos externos HTTP/SSE.

A API não deve absorver a lógica interna do Retrieval, processamento ou investigação.

---

# 4. Workers

Os workers executam operações assíncronas.

A arquitetura permanece:

> **backend único + workers assíncronos**

e não uma arquitetura de microserviços.

A divisão conceitual:

```text
workers/
├── document/
├── retrieval/
├── synthesis/
└── main.*
```

não significa que cada pasta deva se tornar um serviço independente.

O worker executa operações e utiliza as capacidades da aplicação.

---

# 5. Frontend

O framework definido para a Alpha é:

> **Vue 3 + TypeScript**

A organização conceitual é:

```text
frontend/
├── app/
├── features/
├── components/
├── renderer/
├── infrastructure/
└── main.*
```

As principais features são:

```text
Investigation
Evidence
Reading
Corpus
Operations
```

O frontend possui responsabilidade sobre:

* interface;
* estado visual;
* estado local;
* comunicação HTTP/SSE;
* apresentação da StructuredResponse;
* interação do usuário.

O frontend não é autoridade sobre o estado persistente do sistema.

---

# 6. Renderer

O Renderer é uma fronteira própria do frontend:

```text
StructuredResponse
        ↓
Presentation Renderer
        ↓
Vue components / HTML
```

A `StructuredResponse` não será substituída estruturalmente por Markdown.

Markdown pode existir como representação derivada para exportação, cópia, interoperabilidade ou outros usos, mas não é a representação semântica primária da resposta.

---

# 7. Direção das dependências

A direção fundamental é:

```text
Frontend
   ↓ HTTP/SSE
API
   ↓
Application
   ↓
Domain
```

A infraestrutura fornece implementações concretas às abstrações necessárias.

O domínio não deve conhecer detalhes da infraestrutura.

Durante a migração, adapters temporários podem ser utilizados para conectar V3 a componentes V2, desde que sua existência seja explícita e não transforme novamente o V2 na arquitetura oculta da V3.

---

# 8. D.3 — Contratos e modelo concreto

D.3 estabeleceu oito entidades principais:

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

Essas entidades possuem responsabilidades diferentes e não devem ser colapsadas.

---

# 9. Document

`Document` representa a identidade e o estado documental do item pertencente ao corpus.

Sua identidade interna é:

```text
document_id
```

A identidade externa da origem é:

```text
drive_file_id
```

A existência de `drive_file_id` não elimina a necessidade de `document_id`.

O documento mantém, conceitualmente:

* identidade;
* origem;
* relação com o acervo;
* estado;
* metadados documentais;
* timestamps;
* informações necessárias à disponibilidade e processamento.

Estados documentais:

```text
PENDING
PROCESSING
ACTIVE
FAILED
REJECTED
REMOVED
```

Somente documentos `ACTIVE` participam do Retrieval.

---

# 10. DocumentRepresentation

`DocumentRepresentation` representa a estrutura documental preservada pelo Snoopy.

Sua finalidade mínima é permitir recuperar:

```text
documento
→ página
→ bloco
→ ordem
→ localização
→ conteúdo
```

A representação deve preservar, no mínimo:

* páginas;
* blocos;
* texto;
* ordem de leitura;
* localização;
* estrutura básica.

Ela não é:

* Markdown;
* chunk;
* embedding;
* saída do LLM.

A relação é:

```text
Document
    ↓
DocumentRepresentation
```

O processamento documental produz a representação.

A publicação ocorre somente após sua validação.

---

# 11. RetrievalUnit

`RetrievalUnit` substitui conceitualmente o papel dos `chunks` do V2.

Ela é a unidade sobre a qual o mecanismo de recuperação opera.

Deve possuir:

* `unit_id`;
* `document_id`;
* `representation_id`;
* conteúdo;
* localização;
* metadados estruturais;
* metadados contextuais;
* referências aos índices derivados quando necessárias.

A identidade da unidade é independente de:

* embedding;
* contexto adicional;
* score;
* resultado de recuperação.

---

## 11.1 Contexto não altera identidade

A contextualização pode ampliar a informação utilizada pelo Retrieval, mas não deve criar silenciosamente uma nova unidade.

Assim:

```text
unit A
+
contexto
```

continua sendo:

```text
unit A
```

Essa regra preserva a rastreabilidade da unidade até a representação documental original.

---

# 12. Investigation

`Investigation` representa o contexto de uma investigação realizada pelo usuário.

Deve preservar a pergunta original:

```text
original_query
```

A pergunta original é distinta de:

* consulta lexical;
* consulta semântica;
* embedding query;
* consultas derivadas;
* decomposições internas.

Também pertence à investigação o conjunto de filtros estruturados utilizados.

---

# 13. RetrievalResult

`RetrievalResult` representa um resultado produzido pelo mecanismo de recuperação.

Conceitualmente:

```text
result_id
investigation_id
unit_id
rank
retrieval_score
retrieval_metadata
```

A relação é:

```text
Investigation
    ↓
RetrievalResult
    ↓
RetrievalUnit
```

`RetrievalResult` não é automaticamente `Evidence`.

O score de recuperação não significa:

* importância metodológica;
* verdade;
* qualidade da evidência;
* relevância epistemológica.

Ele descreve o processo de recuperação.

---

# 14. Evidence

`Evidence` representa um elemento recuperado, contextualizado e navegável dentro da investigação.

Deve permitir, no mínimo:

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

A existência de um `evidence_id` estável substitui a identificação textual informal do V2, como:

```text
[TRECHO 1]
[TRECHO 2]
```

A evidência possui identidade semântica própria.

---

# 15. Proveniência mínima

A Alpha deve permitir reconstruir:

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

Também devem existir identificadores suficientes para recuperar o contexto investigativo e operacional quando necessário:

```text
investigation_id
operation_id
result_id
unit_id
document_id
location
```

A Alpha não implementará ainda um grafo formal completo de proveniência.

---

# 16. Operation

`Operation` generaliza o conceito de `job` do V2.

Não é apenas uma troca de nome.

`Operation` representa uma execução assíncrona identificável.

Estados:

```text
PENDING
PROCESSING
COMPLETED
FAILED
CANCELLED
```

Exemplos:

```text
DOCUMENT_PROCESSING
INVESTIGATION_SEARCH
CORPUS_SYNC
```

Cada nova tentativa deve possuir uma nova `operation_id`.

A operação não substitui o estado da entidade sobre a qual opera.

Assim:

```text
Document.status
```

e:

```text
Operation.status
```

são estados diferentes.

---

# 17. StructuredResponse

`StructuredResponse` representa semanticamente o resultado da investigação.

Ela pode conter, conceitualmente:

* texto;
* hierarquia;
* referências a evidências;
* referências;
* tabelas;
* relações de navegação;
* outros elementos semânticos necessários à apresentação.

A cadeia é:

```text
Synthesis
    ↓
StructuredResponse
    ↓
Presentation Renderer
    ↓
Vue
```

`StructuredResponse` não é:

* Markdown;
* HTML;
* texto simples;
* fonte documental.

A Alpha exige que a resposta seja recuperável ou reconstruível a partir da identidade da investigação independentemente do SSE.

Isso não implica necessariamente uma tabela própria para `StructuredResponse`.

---

# 18. D.4 — Banco e persistência

D.4 estabeleceu como as entidades de D.3 serão materializadas no PostgreSQL/Supabase.

Princípio fundamental:

> **PostgreSQL/Supabase é a fonte de verdade do estado lógico do Snoopy; índices e artefatos derivados permanecem separados.**

---

# 19. Fonte original

Google Drive continua sendo a fonte documental original.

A relação fundamental é:

```text
drive_file_id
      ↓
document_id
```

O documento interno não deve depender da identidade externa como sua identidade lógica.

---

# 20. Estado lógico

O PostgreSQL/Supabase mantém o estado lógico de:

```text
Document
DocumentRepresentation
RetrievalUnit
Investigation
RetrievalResult
Evidence
Operation
```

Além disso, mantém as relações, estados, metadados e mecanismos de segurança necessários.

---

# 21. Índices derivados

A `RetrievalUnit` alimenta os mecanismos de recuperação:

```text
RetrievalUnit
   ├── PostgreSQL FTS
   └── pgvector
```

O embedding é um artefato derivado.

A unidade de recuperação continua sendo identificada por `unit_id`.

A perda ou reconstrução de um índice não deve destruir a identidade fundamental da unidade.

A Alpha não introduzirá Elasticsearch/OpenSearch.

---

# 22. Artefatos físicos

Artefatos derivados podem existir em armazenamento físico separado, inclusive:

* PDFs temporários;
* arquivos intermediários;
* outros artefatos de processamento.

A fonte original permanece no Google Drive.

A existência de um artefato físico não o transforma em fonte de verdade do estado documental.

---

# 23. Folders e Corpus

O V2 possui `folders`.

Na V3, essa estrutura continua relevante como:

* contexto/acervo;
* relação documental;
* fronteira de acesso;
* contexto da recuperação.

Entretanto, `Corpus` permanece um conceito lógico.

Não foi decidida nem exigida uma entidade persistente independente chamada `Corpus`.

---

# 24. Migração das estruturas V2

O mapeamento conceitual é:

```text
documents
    ↓
Document
    +
DocumentRepresentation

chunks
    ↓
RetrievalUnit
    +
índices derivados

jobs
    ↓
Operation
    +
Queue

match_chunks()
    ↓
componente do Retrieval

search_history
    ↓
estrutura auxiliar, quando necessária
```

A migração não deve ser executada como simples renomeação de tabelas.

---

# 25. Publicação coerente

O fluxo documental persistente deve respeitar:

```text
Document
   ↓
PROCESSING
   ↓
DocumentRepresentation
   ↓
validação
   ↓
RetrievalUnits
   ↓
embeddings + índice lexical
   ↓
validação da publicação
   ↓
ACTIVE
```

Somente depois da produção e validação coerente dos elementos necessários o documento poderá se tornar `ACTIVE`.

Um documento parcialmente processado não deve ser publicado como pesquisável.

Portanto:

```text
Representation ✓
Units ✓
Embedding ✗
      ↓
NÃO ACTIVE
```

A publicação é uma mudança de estado lógico e deve ser protegida contra publicação parcial.

---

# 26. Concorrência e idempotência

O processamento deve ser idempotente.

A mesma identidade externa não deve gerar arbitrariamente múltiplos documentos lógicos.

Operações concorrentes sobre o mesmo documento devem ser controladas para evitar:

* processamento concorrente incoerente;
* sobrescrita indevida;
* índices inconsistentes;
* publicação parcial.

O mecanismo físico de locking/claim ainda pertence à implementação.

O requisito arquitetural, entretanto, está fechado.

---

# 27. Segurança e autorização

A autorização permanece responsabilidade do backend.

RLS é mecanismo de defesa adicional:

```text
Frontend
   ↓
Backend authorization
   ↓
PostgreSQL
   ↓
RLS
```

Não é permitido tratar um `folder_id`, `document_id` ou outro identificador recebido pelo cliente como autorização suficiente.

As regras de isolamento existentes no V2 devem ser mapeadas e preservadas/adaptadas durante a migração.

---

# 28. Dependências consolidadas

Ao final de D.1–D.4, a arquitetura física e lógica pode ser resumida como:

```text
                    GOOGLE DRIVE
                         │
                         ▼
                     Document
                         │
                         ▼
              DocumentRepresentation
                         │
                         ▼
                  RetrievalUnit
                    │         │
                    ▼         ▼
                 FTS       pgvector
                    │         │
                    └────┬────┘
                         ▼
                     Retrieval
                         │
                         ▼
                   Investigation
                         │
                         ▼
                  RetrievalResult
                         │
                         ▼
                      Evidence
                         │
                         ▼
                     Synthesis
                         │
                         ▼
                 StructuredResponse
                         │
                         ▼
                    Renderer
                         │
                         ▼
                  Vue 3 + TypeScript
```

Transversalmente:

```text
Operation
   ↓
Queue
   ↓
Worker
```

e:

```text
Backend authorization
+
RLS
```

protegem o acesso ao estado persistente.

---

# 29. Limites preservados

D.1–D.4 não introduzem no escopo da Alpha:

* versionamento completo;
* snapshots;
* histórico documental completo;
* grafo formal de proveniência;
* múltiplos perfis de segmentação;
* múltiplos rerankers;
* MMR;
* avaliação formal completa do Retrieval;
* OCR;
* multimodalidade avançada;
* infraestrutura distribuída;
* microserviços;
* Kubernetes;
* Kafka;
* CQRS;
* event sourcing;
* API Gateway;
* observabilidade avançada;
* disaster recovery completo;
* offline-first.

Esses itens continuam classificados como pós-Alpha conforme as decisões anteriores.

---

# 30. Invariantes que não devem ser violados

As seguintes distinções estão consolidadas:

```text
Document ≠ DocumentRepresentation
Document ≠ RetrievalUnit
RetrievalUnit ≠ embedding
RetrievalResult ≠ Evidence
Evidence ≠ Synthesis
Synthesis ≠ Source
Operation ≠ Entity State
StructuredResponse ≠ Markdown
StructuredResponse ≠ HTML
Original Query ≠ Derived Retrieval Query
SSE Event ≠ Authoritative State
Corpus ≠ necessariamente entidade persistente
```

Essas distinções devem ser tratadas como restrições arquiteturais para as etapas seguintes.

---

# 31. Estado após a Parte I da Etapa D

D.1–D.4 estabeleceram:

```text
D.1
DE ONDE PARTIMOS
        ↓
D.2
COMO O PROJETO SERÁ ORGANIZADO
        ↓
D.3
O QUE OS DADOS SIGNIFICAM
        ↓
D.4
COMO O ESTADO SERÁ PERSISTIDO
```

A primeira parte da Etapa D está, portanto, **consolidada e fechada**.

Não há necessidade de reabrir D.1–D.4 durante a continuidade normal da Etapa D.

Caso uma etapa posterior encontre uma necessidade que afete diretamente uma decisão já consolidada, a alteração deverá ser tratada explicitamente como **nova decisão arquitetural**, acompanhada de sua justificativa, e não introduzida silenciosamente.

---

# 32. Próxima etapa

A próxima etapa será:

## D.5 — Contratos de API e Comunicação

D.5 deverá transformar as fronteiras já definidas em contratos externos concretos, incluindo:

* endpoints;
* requests;
* responses;
* `operation_id`;
* erros;
* autorização;
* SSE;
* eventos;
* reconexão;
* recuperação de estado;
* relação entre comunicação e estado persistente.

O princípio já consolidado para essa etapa é:

> **HTTP e SSE são mecanismos de comunicação; o estado persistente do backend continua sendo a autoridade.**

A perda de comunicação não deve ser interpretada automaticamente como falha da operação.

---

## Encerramento

**D.1–D.4: CONSOLIDADOS.**

A primeira parte da Etapa D encerra-se com a arquitetura suficientemente definida para avançar da **estrutura interna e persistência** para os **contratos de comunicação**.

**Próximo ponto de retomada: D.5.**
