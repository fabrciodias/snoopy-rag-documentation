# Registro Mestre da Análise — Snoopy-RAG

## Etapa 6.2.7 — Proveniência e Versionamento

**Status:** Registro Mestre da Etapa 6.2.7 — consolidado
**Natureza:** Investigação e consolidação de alternativas tecnológicas e conceituais
**Etapa seguinte:** 6.2.8 — Operação, Segurança e Resiliência
**Projeto técnico:** Etapa 6.3
**Implementação:** Etapa 6.4
**Testes e validação:** Etapa 6.5

---

## 1. Finalidade da Etapa 6.2.7

A Etapa 6.2.7 investigou alternativas tecnológicas e conceituais capazes de permitir ao Snoopy preservar e reconstruir, de maneira verificável, a história de seus documentos, estados, representações, unidades de recuperação, embeddings e resultados de investigação.

A questão central não é simplesmente como versionar arquivos, mas como representar a história de um objeto do Snoopy de modo que seja possível reconstruir:

* sua origem;
* seu estado;
* as transformações pelas quais passou;
* as condições em que foi produzido;
* as dependências utilizadas;
* e sua participação em uma determinada recuperação.

Versionamento e proveniência são, portanto, capacidades relacionadas, mas distintas.

**Versionamento** responde principalmente qual estado de uma entidade está sendo considerado.

**Proveniência** responde como esse estado ou objeto chegou a existir e de quais entidades, atividades e condições depende.

---

## 2. Princípio fundamental

A proveniência não deve ser reduzida ao conteúdo de um objeto.

Conteúdo igual não implica objeto igual.

Duas unidades podem possuir exatamente o mesmo texto e, ainda assim, pertencer a:

* documentos diferentes;
* versões diferentes;
* páginas diferentes;
* posições diferentes;
* contextos diferentes;
* perfis de segmentação diferentes;
* ou execuções diferentes.

Consequentemente, a proveniência deve operar sobre identidades persistentes e relações, e não sobre o conteúdo textual isoladamente.

---

## 3. Dimensões da proveniência

A investigação distinguiu três dimensões principais.

### 3.1 Proveniência documental

Responde:

> De onde veio a informação?

A cadeia pode ser representada conceitualmente como:

```text
Unidade
  ↓
Elemento
  ↓
Página
  ↓
Versão documental
  ↓
Documento
  ↓
Fonte original
```

### 3.2 Proveniência de produção e execução

Responde:

> Como o Snoopy produziu essa representação?

Exemplo:

```text
Unidade
  ↓
Segmentation Run
  ↓
Perfil
  ↓
Configuração
  ↓
Pipeline
  ↓
Modelos/dependências utilizados
```

### 3.3 Proveniência da recuperação

Responde:

> Como essa unidade chegou a ser apresentada como evidência para esta investigação?

Exemplo:

```text
Consulta original
  ↓
Consultas derivadas
  ↓
Recuperação lexical/semântica
  ↓
Fusão
  ↓
Contextualização
  ↓
Reranking
  ↓
Seleção
  ↓
Evidência
```

Essas dimensões não devem ser confundidas, embora possam convergir na reconstrução de um resultado.

---

## 4. Identidade, estado, conteúdo e proveniência

A investigação consolidou a separação entre quatro conceitos.

### 4.1 Identidade lógica

Identifica a entidade no modelo do Snoopy.

```text
document_id
unit_id
corpus_id
```

### 4.2 Estado ou versão

Identifica um estado historicamente distinguível daquela entidade.

```text
document_id
  ├── version_1
  ├── version_2
  └── version_3
```

### 4.3 Conteúdo concreto

Pode ser identificado ou verificado por um digest criptográfico.

```text
content_digest
```

### 4.4 Proveniência

Registra as relações que explicam a origem e a produção daquele estado ou objeto.

Esses conceitos não devem ser colapsados em um único identificador.

---

## 5. Hashes e identidade de conteúdo

Hashes criptográficos possuem papel importante na arquitetura, especialmente para:

* integridade;
* detecção de conteúdos idênticos;
* identificação e verificação de artefatos;
* apoio à identificação de estados de conteúdo;
* eventual content-addressability.

Entretanto:

> **Hash identifica ou verifica conteúdo; não substitui identidade lógica, versionamento ou proveniência.**

Duas unidades documentalmente distintas podem possuir o mesmo conteúdo textual sem serem a mesma unidade lógica.

Assim, a arquitetura conceitualmente combina:

```text
ID lógico
+
versão/estado
+
content digest
+
relações de proveniência
+
execução
+
configuração
```

---

## 6. Derivação não precisa necessariamente ser uma entidade independente

Foi investigada a possibilidade de representar cada derivação como uma entidade explícita:

```text
Objeto A
  ↓
Derivation D1
  ↓
Objeto B
```

e a alternativa de representar a proveniência por meio das relações persistentes do próprio objeto:

```text
Objeto B
  ├── origem
  ├── atividade geradora
  ├── configuração
  ├── versão
  └── demais dependências
```

A investigação concluiu que uma entidade independente `Derivation` não é obrigatória.

O critério fundamental é:

> **A representação escolhida permite reconstruir integralmente a cadeia de proveniência necessária para cada objeto, incluindo origem, atividade geradora, configuração relevante, versão, temporalidade e dependências?**

Se as relações existentes forem suficientes para responder essa pergunta, uma entidade explícita de derivação não é necessária apenas por razões de elegância conceitual.

Essa decisão não deve ser transformada em dogma. Caso uma derivação futura possua complexidade que não possa ser representada adequadamente pelas relações existentes, uma entidade explícita poderá ser introduzida no projeto técnico.

---

## 7. Proveniência deve utilizar relações semanticamente distintas

A investigação descartou a ideia de uma relação genérica baseada simplesmente em `source_id`.

Relações como:

```text
derived_from
generated_by
belongs_to
retrieved_by
selected_by
contextualized_by
```

possuem significados diferentes.

Portanto, a proveniência deve preservar relações semanticamente tipadas.

Essa exigência é conceitual e não determina a utilização de um banco de dados de grafos.

---

## 8. Entity, State, Activity e Derivation

O modelo conceitual consolidado distingue:

### Entity

Uma entidade lógica persistente:

```text
Documento D1
Unidade U182
Corpus C7
```

### State

Um estado historicamente distinguível:

```text
D1-V3
C7-S12
```

### Activity / Run

Uma execução concreta que produz ou transforma entidades:

```text
ExtractionRun
StructuralAnalysisRun
SegmentationRun
EmbeddingRun
RetrievalRun
```

### Derivation

A relação que indica que uma entidade ou estado foi produzido a partir de outro.

Esse modelo permite representar tanto:

```text
A → B
```

quanto:

```text
A
├── B
└── C
```

sem reduzir toda a história do sistema a uma sequência linear.

---

## 9. Versionamento linear e proveniência em grafo

Uma das principais conclusões da investigação é que versionamento e proveniência não precisam possuir a mesma topologia.

Para a evolução de um documento, uma sequência linear é adequada:

```text
Documento D1
  ├── V1
  ├── V2
  └── V3
```

Porém, derivados podem ramificar:

```text
D1-V3
   ↓
  R1
 /  \
SP1 SP2
```

SP1 e SP2 não são necessariamente versões sucessivas uma da outra.

São derivações alternativas produzidas a partir da mesma origem.

Da mesma maneira:

```text
U182
├── Embedding E1
└── Embedding E2
```

E2 não precisa ser tratado como uma “versão posterior” de E1. Pode ser outra representação da mesma unidade.

Portanto:

> **A cadeia de evolução de uma entidade pode ser linear; a proveniência geral deve admitir ramificações e derivações alternativas.**

---

## 10. O modelo lógico é um grafo, mas isso não determina a tecnologia

A existência de uma estrutura de proveniência em grafo não implica a adoção de um graph database.

As relações podem ser representadas em uma arquitetura relacional por meio de:

* identificadores;
* chaves estrangeiras;
* relações tipadas;
* estados;
* atividades;
* registros de execução;
* estruturas de lineage.

Consequentemente, a investigação não encontrou justificativa suficiente para introduzir um banco de grafos dedicado apenas para representar proveniência.

A decisão tecnológica concreta permanece na Etapa 6.3.

---

## 11. Representação de estados históricos

Estados históricos relevantes não devem ser silenciosamente sobrescritos.

Se:

```text
V1
```

é sucedido por:

```text
V2
```

o estado V1 deve continuar historicamente distinguível quando sua preservação for necessária à reconstrução, auditoria ou comparação.

Isso não implica que todo dado operacional do sistema precise ser imutável permanentemente.

O princípio é:

> **Estados históricos relevantes para proveniência e reconstrução devem permanecer distinguíveis e não ser silenciosamente sobrescritos.**

---

## 12. Event sourcing

Event sourcing foi investigado como possível estratégia de persistência histórica.

A ideia de registrar eventos como:

```text
DocumentCreated
RepresentationCreated
SegmentationCreated
EmbeddingCreated
DocumentUpdated
VersionActivated
DocumentRemoved
```

é compatível com as necessidades de reconstrução histórica.

Entretanto, a investigação distinguiu:

* **event sourcing completo**, em que o estado é reconstruído a partir dos eventos;
* **histórico/event log/audit trail**, em que o estado atual continua persistido normalmente e acontecimentos relevantes também são registrados.

A segunda abordagem foi considerada mais adequada ao contexto atual do Snoopy.

Portanto:

> **Event sourcing completo não é requisito arquitetural identificado para o Snoopy. Um histórico ou registro de eventos relevantes pode ser utilizado como capacidade complementar de proveniência e auditoria.**

---

## 13. Temporalidade

A investigação identificou que diferentes dimensões temporais podem possuir significados distintos.

Entre elas:

### Valid time

Quando determinado estado é considerado válido no domínio documental.

### Transaction/system time

Quando o Snoopy registrou determinado estado.

### Discovery time

Quando o Snoopy tomou conhecimento de uma alteração na fonte externa.

Exemplo:

```text
V1 válido até 10/01
V2 válido desde 10/01
Snoopy descobriu V2 em 12/01
```

Portanto:

```text
valid time
≠
transaction time
≠
discovery time
```

A existência dessas dimensões não implica a adoção de bitemporalidade ou temporalidade universal em toda a arquitetura.

O princípio é preservar as dimensões temporais que possuam significado para a entidade ou processo correspondente.

---

## 14. Relação entre Google Drive e estado interno

O Google Drive permanece como fonte documental externa.

O estado do Drive e o estado do Snoopy são entidades conceitualmente distintas.

O Snoopy precisa ser capaz de representar que determinado estado interno corresponde ao estado observado da fonte externa em determinado momento.

Conceitualmente:

```text
Drive File
   ↓
estado observado da fonte
   ↓
Document Version
   ↓
representações derivadas
```

Isso permite distinguir:

* estado da fonte;
* momento em que o Snoopy o descobriu;
* estado interno produzido;
* derivados produzidos a partir daquele estado.

---

## 15. Profile, Run e resultado

A investigação consolidou a distinção entre estratégia, execução e resultado.

### Profile

Representa uma estratégia reutilizável.

```text
Segmentation Profile SP3
```

### Run

Representa uma execução concreta:

```text
Segmentation Run R42
```

### Resultado

Representa aquilo que foi produzido pela execução:

```text
Unidades U1, U2, U3...
```

Uma mesma estratégia pode possuir várias execuções:

```text
SP3
├── Run41
├── Run42
└── Run43
```

Cada Run deve permitir reconstruir as condições efetivamente utilizadas.

---

## 16. Snapshot da configuração

A existência de uma configuração reutilizável não elimina a necessidade de registrar as condições concretas de uma execução.

Conceitualmente:

```text
Profile
   ↓
Run
   ↓
Configuration Snapshot
```

O snapshot permite registrar os parâmetros efetivamente utilizados sem transformar toda alteração transitória em uma nova entidade conceitual permanente.

Princípio:

> **Entidades conceituais reutilizáveis podem possuir versionamento próprio; execuções concretas devem preservar o estado efetivamente utilizado.**

---

## 17. Dependências externas

O Snoopy poderá depender de:

* bibliotecas;
* modelos;
* mecanismos de OCR;
* componentes de estruturação;
* modelos de embedding;
* rerankers;
* LLMs;
* outros componentes tecnológicos.

O Snoopy precisa registrar suficientemente **qual dependência foi utilizada e qual versão ou identificação era aplicável à execução**.

Entretanto:

> **O Snoopy precisa identificar a dependência utilizada; não precisa administrar o histórico interno de versões dessa dependência.**

A responsabilidade pelo versionamento interno de um modelo ou biblioteca permanece com seu respectivo fornecedor ou ecossistema.

---

## 18. Corpus como estado identificável

O corpus não deve ser tratado apenas como uma pasta ou conjunto mutável de documentos.

Uma investigação precisa ser relacionada ao estado documental do corpus utilizado.

Conceitualmente:

```text
Corpus C7
├── Snapshot S1
├── Snapshot S2
└── Snapshot S3
```

Cada snapshot pode representar um conjunto específico de estados documentais:

```text
C7-S3
 ├── D1-V3
 ├── D2-V1
 └── D5-V4
```

Uma recuperação pode então referenciar o snapshot correspondente:

```text
Retrieval Run R91
   ↓
Corpus Snapshot C7-S3
```

Isso permite reconstruir o universo documental sobre o qual a recuperação ocorreu.

---

## 19. Proveniência da recuperação

A recuperação possui sua própria cadeia histórica.

Uma unidade recuperada pode possuir a cadeia documental:

```text
U182
 ↓
elemento
 ↓
representação
 ↓
document version
 ↓
document
```

Enquanto a recuperação que a selecionou possui:

```text
Retrieval Run
 ↓
Corpus Snapshot
 ↓
Query
 ↓
Derived Queries
 ↓
Retrieval Profile
 ↓
Candidate Generation
 ↓
Contextualization
 ↓
Reranking
 ↓
Selection
 ↓
U182
```

Essas duas cadeias devem permanecer conceitualmente distintas.

A recuperação referencia a unidade e registra sua própria história, em vez de duplicar a proveniência documental completa da unidade.

---

## 20. Proveniência abaixo do nível documental

A proveniência precisa poder chegar aos níveis necessários para reconstruir a origem da evidência.

Conceitualmente:

```text
Document Version
 ↓
Page
 ↓
Element
 ↓
Fragment / Cell
 ↓
Retrieval Unit
```

Isso é particularmente importante para:

* parágrafos;
* tabelas;
* células;
* figuras;
* legendas;
* fragmentos;
* elementos espaciais;
* demais estruturas preservadas pela representação canônica.

Assim, uma unidade pode ser rastreada, quando necessário, até sua localização documental concreta.

---

## 21. Equivalência não deve destruir proveniência

Duas entidades podem possuir conteúdo equivalente:

```text
U182
≈
U205
```

ou:

```text
hash(U182) = hash(U205)
```

Isso não significa que devam ser automaticamente colapsadas em uma única entidade.

A investigação consolidou:

```text
mesmo conteúdo
≠
mesma proveniência
≠
mesma execução
≠
mesma entidade lógica
```

Equivalência ou deduplicação podem ser representadas separadamente da identidade e da proveniência.

---

## 22. Content-addressability

Content-addressability foi considerada uma capacidade potencialmente útil para:

* integridade;
* artefatos imutáveis;
* identificação de conteúdo;
* detecção de duplicação;
* reconstrução histórica.

Entretanto, não deve substituir a identidade lógica do Snoopy.

A investigação também não encontrou justificativa para adotar uma infraestrutura específica, como OCI, apenas para obter essa capacidade.

---

## 23. W3C PROV

O W3C PROV foi identificado como uma referência semântica particularmente relevante para o modelo de proveniência.

Sua estrutura fornece conceitos para:

* entidades;
* atividades;
* agentes;
* derivação;
* geração;
* uso;
* invalidação;
* especialização;
* entidades alternativas.

Sua principal utilidade para o Snoopy está na possibilidade de fornecer uma linguagem conceitual formal para descrever relações de proveniência.

O Snoopy não precisa necessariamente implementar todo o modelo PROV.

A decisão concreta sobre quais conceitos incorporar pertence à Etapa 6.3.

---

## 24. OpenLineage

OpenLineage foi identificado como referência particularmente interessante para aspectos relacionados a:

* Jobs;
* Runs;
* Datasets;
* parâmetros de execução;
* informações de versão;
* facets;
* lineage operacional.

Sua função potencial é complementar a referência do PROV:

```text
PROV
→ semântica geral de proveniência

OpenLineage
→ referência para execução e lineage operacional
```

Nenhum dos dois foi considerado um substituto do modelo interno de proveniência do Snoopy.

---

## 25. PostgreSQL como camada de governança persistente

Considerando a decisão estabelecida na Etapa 6.2.4 de utilizar PostgreSQL/Supabase como camada governante do estado lógico, a investigação identificou o PostgreSQL como candidato natural para persistir:

* entidades;
* estados;
* versões;
* relações;
* lineage;
* corpus snapshots;
* runs;
* configurações;
* referências a artefatos;
* hashes;
* informações temporais.

Isso evita introduzir uma segunda fonte de verdade apenas para armazenar proveniência.

---

## 26. Arquitetura tecnológica composta

Assim como nas etapas anteriores, nenhuma tecnologia isolada demonstrou resolver satisfatoriamente todo o problema.

A direção tecnológica consolidada é de composição:

```text
                         SNOOPY
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
       ESTADO            LINHAGEM          EXECUÇÃO
          │                 │                 │
          ▼                 ▼                 ▼
    PostgreSQL        modelo inspirado     Run/Event
                      em PROV
          │
          └───────────────┬──────────────────┘
                          ▼
                    PROVENIÊNCIA
```

Complementada por:

```text
IDs + hashes
```

para identidade lógica e integridade/verificação de conteúdo.

O princípio arquitetural é:

> **A tecnologia fornece a capacidade; o Snoopy fornece a governança.**

Cada componente deve possuir uma responsabilidade concreta e justificável.

---

## 27. Exportação da proveniência

A investigação distinguiu explicitamente:

**persistência da proveniência**

de:

**exportação da proveniência**.

A primeira é necessária.

A segunda é opcional.

Uma exportação pode ser útil para:

* auditoria externa;
* interoperabilidade;
* arquivamento;
* compartilhamento de uma investigação;
* documentação independente;
* reprodução por terceiros.

Entretanto, exportar também pode:

* criar uma segunda fonte de verdade;
* introduzir problemas adicionais de versão;
* exigir definição de formato;
* aumentar complexidade;
* expor informações internas;
* criar uma falsa impressão de que o arquivo exportado, sozinho, garante reprodução.

Portanto:

> **A exportação não deve ser requisito arquitetural primário da proveniência. Deve ser tratada como uma projeção externa opcional do estado interno, justificada por uma necessidade concreta.**

A fonte de verdade permanece no estado persistente governado pelo Snoopy.

---

## 28. O que foi descartado ou não se mostrou necessário

A investigação não encontrou justificativa suficiente para:

* utilizar um graph database exclusivamente para provenance;
* adotar event sourcing completo;
* transformar todo estado em evento;
* utilizar hashes como identidade universal;
* tratar toda alteração como uma nova versão;
* tratar toda derivação como uma entidade independente;
* delegar a proveniência integralmente a PROV ou OpenLineage;
* criar um sistema externo de provenance como segunda fonte de verdade;
* exportar provenance por padrão;
* adotar content-addressability como identidade lógica universal;
* introduzir uma tecnologia apenas porque ela oferece um mecanismo teoricamente mais sofisticado.

Essas conclusões não significam que essas tecnologias sejam inadequadas em qualquer cenário, apenas que não demonstraram necessidade suficiente para constituir o núcleo do Snoopy nas condições investigadas.

---

## 29. Modelo conceitual consolidado

A estrutura geral resultante da investigação pode ser representada como:

```text
                         ENTITY
                            │
                  ┌─────────┴─────────┐
                  ▼                   ▼
                STATE              DERIVATION
                  │                   │
                  │                   ▼
                  │                ACTIVITY
                  │                   │
                  └─────────┬─────────┘
                            ▼
                       PROVENANCE
```

Aplicada ao Snoopy:

```text
Documento
   │
   ├── Versão documental
   │       │
   │       └── Representação
   │               │
   │               └── Estrutura
   │                       │
   │                       └── Unidades
   │                               │
   │                               └── Embeddings
   │
   └── Estados/derivações alternativas
```

E, paralelamente:

```text
Corpus Snapshot
      +
Query
      +
Retrieval Profile
      +
Run
      +
Candidate Generation
      +
Contextualization
      +
Reranking
      +
Selection
      ↓
Evidence
```

As duas estruturas se conectam por relações de proveniência.

---

## 30. Princípios consolidados da Etapa 6.2.7

A investigação estabelece os seguintes princípios:

1. **Identidade lógica não deve ser confundida com conteúdo.**

2. **Versão identifica um estado historicamente distinguível de uma entidade.**

3. **Hash identifica ou verifica conteúdo; não substitui identidade ou proveniência.**

4. **Proveniência deve ser representada por relações semanticamente distintas.**

5. **Uma entidade universal de `Derivation` não é obrigatória quando as relações existentes forem suficientes para reconstruir a cadeia necessária.**

6. **A proveniência deve admitir ramificações e derivações alternativas.**

7. **O versionamento de um documento pode ser linear, enquanto a proveniência geral pode possuir estrutura de grafo.**

8. **Um modelo lógico em grafo não implica um banco de grafos.**

9. **Estados históricos relevantes não devem ser silenciosamente sobrescritos.**

10. **Event sourcing completo não é requisito arquitetural identificado.**

11. **Temporalidade deve ser preservada conforme o significado de cada entidade e processo.**

12. **Profile, Run e resultado são conceitos distintos.**

13. **Execuções concretas devem preservar as condições efetivamente utilizadas.**

14. **Dependências externas devem ser identificáveis em relação à execução que as utilizou.**

15. **O corpus deve possuir estados identificáveis para permitir reconstrução do universo documental da investigação.**

16. **A proveniência documental e a proveniência da recuperação devem permanecer conceitualmente distintas.**

17. **A proveniência deve alcançar o nível documental necessário à localização da evidência.**

18. **Equivalência de conteúdo não deve destruir distinções de identidade e proveniência.**

19. **W3C PROV é uma referência semântica útil; OpenLineage é uma referência útil para execução e lineage operacional.**

20. **PostgreSQL é candidato natural à governança persistente do modelo, dentro da arquitetura estabelecida na 6.2.4.**

21. **A exportação de provenance é opcional e deve ser tratada como projeção externa, não como fonte de verdade.**

22. **A arquitetura deve ser composta, mas governada pelo Snoopy.**

---

## 31. Limites epistemológicos da Etapa 6.2.7

Esta etapa não definiu:

* schema definitivo;
* tabelas definitivas;
* nomes definitivos de entidades;
* formato definitivo dos IDs;
* algoritmo definitivo de hashing;
* formato da representação canônica;
* estrutura definitiva de lineage;
* política definitiva de temporalidade;
* política definitiva de retenção;
* implementação de audit/event log;
* mecanismo definitivo de consulta de provenance;
* formato de exportação;
* adoção definitiva de PROV;
* adoção definitiva de OpenLineage.

Esses elementos pertencem ao **projeto técnico da Etapa 6.3**, salvo questões operacionais que pertençam à 6.2.8.

---

## 32. Relação com a Etapa 6.2.8

A investigação também estabelece uma fronteira importante para a próxima etapa.

A Etapa 6.2.7 concentrou-se em:

> **como representar identidade, estado, versão, derivação, execução, temporalidade e lineage.**

A Etapa 6.2.8 deverá concentrar-se em:

* autenticação;
* autorização;
* isolamento;
* segurança;
* concorrência operacional;
* idempotência;
* filas;
* segredos;
* observabilidade;
* limites de recursos;
* recuperação de falhas;
* operação;
* resiliência;
* políticas operacionais.

Questões de consistência podem aparecer em ambas, mas sob perspectivas diferentes:

```text
6.2.7
→ como representar e preservar a história e as relações?

6.2.8
→ como operar essa estrutura de forma segura, confiável e resiliente?
```

---

## 33. Princípio central da Etapa 6.2.7

> **O Snoopy deve possuir um modelo interno de proveniência baseado em identidades persistentes, estados versionados, relações tipadas de derivação e execução, informações temporais e identificação das dependências utilizadas. A proveniência deve ser reconstruível como uma rede de relações, sem exigir necessariamente uma entidade independente para cada derivação. Tecnologias e padrões externos podem fornecer mecanismos, semântica e referências de interoperabilidade, mas a fonte de verdade e a governança da proveniência devem permanecer no próprio Snoopy.**

---

## 34. Conclusão da Etapa 6.2.7

A investigação da Etapa 6.2.7 demonstrou que o problema de proveniência e versionamento do Snoopy não deve ser reduzido ao versionamento de arquivos nem resolvido pela introdução de uma ferramenta externa especializada.

O modelo mais coerente é aquele em que identidade lógica, estado, conteúdo, derivação, execução e temporalidade são dimensões distintas, porém relacionadas.

O documento pode possuir uma evolução linear:

```text
V1 → V2 → V3
```

enquanto suas representações, segmentações, embeddings e experimentos podem ramificar:

```text
        R1
       /  \
     S1    S2
```

A proveniência geral, portanto, deve admitir uma estrutura de grafo, ainda que sua persistência possa permanecer relacional.

A ausência de uma entidade explícita de derivação não constitui perda de provenance quando as relações persistentes forem suficientes para reconstruir a cadeia necessária. Da mesma forma, hashes podem garantir integridade e identificar conteúdo sem substituir as identidades lógicas e históricas do sistema.

W3C PROV fornece uma referência semântica para o modelo; OpenLineage oferece referências úteis para execução e lineage operacional; hashes oferecem mecanismos de integridade e identificação de conteúdo; e PostgreSQL constitui o candidato natural para persistência e governança dessas informações dentro da arquitetura já estabelecida.

Nenhuma dessas tecnologias deve se tornar uma fonte de verdade externa ao Snoopy.

A composição tecnológica permanece, portanto, como direção preferencial:

```text
capacidades especializadas
          ↓
       composição
          ↓
       governança
          ↓
         Snoopy
```

A conclusão fundamental da etapa é:

> **A cadeia de evolução de uma entidade pode ser linear; a proveniência do sistema não precisa ser. O Snoopy deve preservar a história de suas entidades e estados como uma rede reconstruível de relações, distinguindo evolução, derivação, alternativas, conteúdo e execução.**

Com isso, a investigação tecnológica fundamental da **Etapa 6.2.7 — Proveniência e Versionamento** é considerada encerrada e pronta para transição à **Etapa 6.2.8** e, posteriormente, ao projeto técnico da **Etapa 6.3**.
