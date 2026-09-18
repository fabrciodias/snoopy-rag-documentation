# Registro Mestre da Análise — Snoopy-RAG

## Etapa 6.2.4 — Persistência da Representação

**Status:** Registro Mestre da Etapa 6.2.4 — consolidado
**Natureza:** Investigação e definição de diretrizes tecnológicas para persistência
**Escopo:** Persistência da representação canônica e de seus derivados
**Relação:** Etapa 6.2 — Investigação das alternativas tecnológicas
**Próxima etapa:** 6.2.5 — Segmentação

---

## 1. Finalidade da Etapa

A Etapa 6.2.4 teve como finalidade investigar como as representações produzidas pelo Snoopy-RAG devem ser persistidas de forma adequada, sustentável e coerente com os requisitos estabelecidos nas etapas anteriores.

A investigação partiu da distinção entre:

* fonte documental original;
* estado lógico do sistema;
* representação canônica;
* derivados da representação;
* artefatos físicos;
* estados de processamento e publicação.

O objetivo não foi definir ainda o esquema definitivo de banco de dados, a estrutura final de diretórios ou mecanismos específicos de implementação, mas estabelecer **qual papel cada meio de armazenamento deve desempenhar na arquitetura do Snoopy V3**.

---

## 2. Princípio Fundamental

A persistência do Snoopy não deve ser entendida como a escolha de um único local onde "tudo" será armazenado.

Diferentes tipos de informação possuem diferentes necessidades de persistência.

Assim, a arquitetura deverá distinguir:

> **estado lógico governável pelo sistema**

de

> **artefatos físicos derivados produzidos pelo processamento.**

Essa separação permite utilizar cada meio de armazenamento de acordo com sua função, evitando tanto a transformação do banco de dados em depósito indiscriminado de arquivos quanto a perda do controle lógico sobre os artefatos armazenados.

---

## 3. O Google Drive como Fonte Documental Original

O Google Drive permanece como **fonte documental original** do Snoopy.

O PDF original não deverá ser duplicado no armazenamento próprio do Snoopy apenas para manter uma cópia permanente.

Durante o processamento, o Snoopy pode materializar temporariamente o arquivo para utilização pelo pipeline, mas essa materialização operacional não constitui necessariamente uma nova cópia documental persistente.

O sistema deverá manter as referências necessárias à fonte, incluindo sua identificação e informações necessárias para relacionar os derivados ao documento de origem.

Consequentemente:

```text
Google Drive
    ↓
documento original
    ↓
processamento do Snoopy
```

O armazenamento próprio do Snoopy não substitui a fonte documental original.

---

## 4. Separação entre Estado Lógico e Artefatos Físicos

A persistência deverá ser conceitualmente dividida em duas camadas principais.

### 4.1. Estado lógico

O PostgreSQL/Supabase deverá funcionar como camada governante das informações estruturais e relacionais do sistema.

Entre os elementos que pertencem conceitualmente a essa camada estão:

* documentos;
* versões documentais;
* metadados;
* estados;
* relações;
* unidades de recuperação;
* embeddings;
* provenance;
* jobs;
* configurações;
* referências aos artefatos físicos.

Essa camada responde essencialmente:

> O que existe?
> A que pertence?
> Qual é sua versão?
> Qual é seu estado?
> De onde veio?
> Qual derivação o produziu?
> Qual artefato físico corresponde a ele?

### 4.2. Artefatos físicos

Artefatos derivados que sejam fisicamente volumosos, binários ou pouco adequados ao armazenamento relacional poderão ser mantidos em um Storage separado do banco.

Exemplos possíveis:

* imagens extraídas;
* páginas renderizadas;
* recortes utilizados e considerados persistentes;
* representações canônicas grandes;
* outros artefatos físicos derivados.

O Storage não constitui uma segunda fonte documental. Ele funciona como **repositório físico de artefatos derivados do processamento do Snoopy**.

---

## 5. Granularidade da Persistência

A persistência não deverá adotar uma granularidade extrema.

Não se pretende transformar cada elemento mínimo do documento — como cada palavra, bbox ou célula — em uma entidade relacional independente.

Também não se pretende concentrar indiscriminadamente toda a representação em um único objeto monolítico.

A tendência arquitetural consolidada é distinguir entre:

### Entidades com identidade própria

Conceitualmente, devem possuir identidade persistente própria elementos como:

* documento;
* versão documental;
* representação;
* segmentação;
* unidade de recuperação;
* embedding;
* job;
* artefato.

Essas entidades participam diretamente das relações de dependência, estado, versionamento e rastreabilidade do sistema.

### Elementos internos

Elementos como:

* parágrafos;
* blocos;
* células;
* bboxes;
* relações internas;
* elementos de página;

podem permanecer encapsulados dentro das representações estruturadas quando não houver necessidade de tratá-los como entidades independentes.

A definição exata dessa granularidade, incluindo o modelo de tabelas e estruturas de dados, pertence à Etapa 6.3.

---

## 6. Identidade Física dos Artefatos

Os artefatos físicos não deverão depender de nomes humanos de arquivos como sua identidade principal.

A organização física deverá poder utilizar identificadores estáveis do modelo documental, especialmente identificadores associados a documentos e versões.

Conceitualmente:

```text
storage/
└── documents/
    └── {document_id}/
        └── versions/
            └── {version_id}/
                ├── ...
```

Essa organização é preferível a uma estrutura rigidamente baseada em usuários, pois a autorização pertence ao modelo lógico do sistema.

O caminho físico não precisa reproduzir integralmente a estrutura de permissões.

A distinção consolidada é:

> **segregação lógica → banco de dados**
> **segregação física → estrutura de armazenamento e identificadores**
> **autorização → aplicação/backend**

A estrutura definitiva do filesystem será definida na Etapa 6.3.

---

## 7. Versionamento e Genealogia

A investigação confirmou que diferentes tipos de versão não devem ser confundidos.

Devem ser conceitualmente distinguidos:

```text
versão documental
≠
versão da representação
≠
versão da segmentação
≠
versão do embedding
```

Uma alteração no documento pode produzir uma nova versão documental e, consequentemente, novas representações e derivados.

Uma alteração na segmentação pode ocorrer sem alteração do documento ou da representação canônica.

Uma mudança no modelo de embedding pode produzir novos embeddings sem exigir nova extração do documento.

Assim, a persistência deverá manter a **genealogia das derivações**.

Conceitualmente:

```text
Documento V1
    ↓
Representação C1
    ├── Segmentação S1
    │      └── Embeddings E1
    │
    └── Segmentação S2
           └── Embeddings E2
```

e:

```text
Documento V2
    ↓
Representação C2
    └── Segmentação S3
           └── Embeddings E3
```

Cada derivado deverá poder ser relacionado ao estado documental sobre o qual foi produzido.

A implementação concreta dessa genealogia será definida na Etapa 6.3.

---

## 8. Hash, Identidade e Integridade

O hash deverá ser entendido como mecanismo de identificação e verificação de conteúdo, e não como substituto do conceito de versão documental.

Assim:

```text
hash
→ identidade/integridade do conteúdo do artefato

version_id
→ identidade do estado documental

versão da derivação
→ identidade do processo/representação derivada
```

Esses conceitos deverão permanecer distintos.

---

## 9. Persistência versus Artefatos Temporários

Nem tudo que é produzido durante o processamento deverá ser persistido permanentemente.

O pipeline poderá produzir artefatos intermediários necessários apenas para determinada etapa.

Exemplos:

* PDF materializado temporariamente a partir do Drive;
* imagens temporárias utilizadas somente pelo OCR;
* crops utilizados em processamento intermediário;
* resultados intermediários que não participem da representação publicada;
* caches.

Esses elementos poderão ser descartados após o uso.

Em contrapartida, deverão possuir persistência os estados e derivados necessários para que o Snoopy mantenha:

* a representação documental;
* sua rastreabilidade;
* suas unidades recuperáveis;
* seus embeddings;
* seus estados;
* sua capacidade de reutilização;
* sua capacidade de reconstrução ou auditoria, quando necessária.

O critério não é "guardar tudo", mas:

> **preservar aquilo cuja existência é necessária para o estado documental, a reutilização, a rastreabilidade ou a recuperação do sistema.**

---

## 10. Publicação e Consistência

A existência física de um artefato não deverá significar automaticamente que ele está publicado ou vigente no Snoopy.

Essa distinção é necessária porque o armazenamento lógico e o armazenamento físico podem sofrer falhas em momentos diferentes.

Conceitualmente:

```text
geração
   ↓
armazenamento
   ↓
validação
   ↓
registro
   ↓
publicação
```

O estado lógico deverá determinar se determinado artefato ou representação é considerado válido e disponível para utilização pelo restante do sistema.

Isso evita estados em que, por exemplo:

* o banco aponta para um artefato inexistente;
* um artefato incompleto é tratado como publicado;
* uma representação antiga permanece sendo utilizada após uma nova versão documental;
* um processamento parcialmente concluído é confundido com processamento válido.

O mecanismo concreto de publicação e reconciliação será definido em 6.3.

---

## 11. Storage Local

Para a versão V3 atualmente projetada, o armazenamento físico dos artefatos derivados será **local ao servidor do Snoopy**.

A escolha foi feita considerando:

* escala atual do projeto;
* volume esperado de dados;
* existência do servidor;
* ausência de necessidade atual de armazenamento distribuído;
* simplicidade operacional;
* ausência de necessidade de duplicar os PDFs originais.

Não há necessidade, neste momento, de introduzir um serviço externo de object storage exclusivamente por razões de capacidade.

O Storage local não é considerado uma solução provisória arquiteturalmente inadequada. Ele é a solução proporcional à escala atual.

---

## 12. Backup

O Storage local deverá possuir uma estratégia de backup.

A função do backup é proteger principalmente os artefatos derivados que seriam inconvenientes ou custosos de reconstruir.

O Google Drive continua oferecendo a fonte original dos documentos, permitindo, em muitos casos, reconstrução dos derivados a partir do documento original.

Entretanto, a possibilidade de reconstrução não elimina a necessidade de backup, pois reconstrução pode envolver:

* tempo;
* processamento;
* mudanças de ferramentas;
* mudanças de versões;
* alterações de configuração;
* perda de estados intermediários ou históricos.

Assim, a estratégia consolidada é:

```text
Storage local
      ↓
backup periódico
```

A implementação concreta, periodicidade, destino e política de retenção do backup pertencem à Etapa 6.3/6.5.

---

## 13. Escalabilidade Futura

A arquitetura não deverá impedir futura mudança da camada física de armazenamento.

Caso o Snoopy cresça significativamente, poderão ser consideradas alternativas como:

* expansão do armazenamento local;
* armazenamento dedicado;
* NAS ou solução equivalente;
* object storage externo;
* outras estratégias de armazenamento.

Essas alternativas não fazem parte da solução atual.

A decisão presente é:

> **Storage local na V3, com backup, mantendo aberta a possibilidade de migração ou expansão futura conforme necessidade real.**

---

## 14. O que foi descartado como necessidade atual

A investigação não identificou justificativa para:

* armazenar permanentemente os PDFs originais no Snoopy;
* introduzir object storage externo apenas por princípio;
* projetar armazenamento para escalas de petabytes;
* criar infraestrutura distribuída antecipadamente;
* duplicar no filesystem a lógica de autorização do banco;
* persistir indiscriminadamente todos os artefatos intermediários;
* utilizar um sistema especializado de versionamento de dados como camada principal do Snoopy.

Essas decisões refletem a escala e o contexto atual do projeto.

---

## 15. Estado da Investigação Tecnológica

A investigação das alternativas mostrou que a persistência pode ser composta por diferentes tecnologias, mas a necessidade atual não exige uma arquitetura de armazenamento externo.

O modelo que melhor se ajusta aos requisitos consolidados é:

```text
Google Drive
     │
     │ fonte original
     ▼
   Snoopy
     │
     ├───────────────┐
     │               │
     ▼               ▼
PostgreSQL       Storage local
     │               │
estado lógico    artefatos físicos
relações         derivados
versões          assets
provenance       representações grandes
embeddings
jobs
     │
     └───────────────┐
                     ▼
                  Backup
```

O PostgreSQL governa o significado e o estado dos dados.

O Storage local mantém os artefatos físicos que não devem necessariamente residir no banco.

O Google Drive permanece como fonte original.

O backup protege a camada física derivada.

---

## 16. Relação com a Arquitetura Geral

A persistência deverá sustentar a cadeia:

```text
documento
   ↓
versão documental
   ↓
representação canônica
   ↓
estrutura
   ↓
segmentação
   ↓
unidade de recuperação
   ↓
embedding
   ↓
recuperação
```

Cada estado derivado deverá permanecer relacionado ao estado que o originou.

A persistência, portanto, não é apenas infraestrutura de armazenamento: ela é parte da capacidade do Snoopy de preservar a coerência entre diferentes estados e representações do corpus.

---

## 17. Consequências para a Etapa 6.3

A Etapa 6.3 deverá transformar essas conclusões em um projeto técnico concreto.

Entre as decisões que deverão ser tomadas estão:

* modelo definitivo das entidades persistentes;
* estrutura das relações entre documentos e versões;
* representação física da representação canônica;
* critérios para decidir o que permanece no PostgreSQL e o que vai para o Storage;
* estrutura de diretórios;
* identificação dos artefatos;
* modelo concreto de genealogia;
* estratégia de publicação;
* tratamento de falhas parciais;
* mecanismo de limpeza de artefatos temporários;
* estratégia de backup;
* política de retenção;
* implementação concreta da persistência local.

Essas decisões não fazem parte da presente etapa.

---

## 18. Princípios Consolidados da Etapa 6.2.4

A investigação permite consolidar os seguintes princípios:

1. **O Google Drive permanece como fonte documental original.**
2. **O Snoopy não precisa duplicar permanentemente os PDFs originais.**
3. **O PostgreSQL constitui a camada governante do estado lógico.**
4. **Artefatos físicos derivados podem ser armazenados separadamente do banco.**
5. **O Storage da V3 será local ao servidor.**
6. **O Storage local possuirá backup periódico.**
7. **A identidade dos artefatos deve ser independente de nomes humanos de arquivos.**
8. **A identidade física deve se relacionar à identidade documental lógica.**
9. **Autorização pertence à aplicação, não ao filesystem.**
10. **Versão documental, versão de representação e versões de derivações devem permanecer distintas.**
11. **Hash não substitui versionamento.**
12. **A genealogia dos derivados deve ser preservada.**
13. **Artefatos temporários não devem ser persistidos indiscriminadamente.**
14. **Existência física não equivale a publicação lógica.**
15. **Estados parcialmente processados não devem ser apresentados como estados válidos.**
16. **A solução de armazenamento deve ser proporcional à escala do Snoopy.**
17. **A arquitetura deve permitir futura migração ou expansão do armazenamento sem depender dela na V3.**

---

## 19. Princípio Central da Etapa

A persistência do Snoopy-RAG deverá separar **a fonte documental, o estado lógico e os artefatos físicos derivados**, mantendo entre eles relações identificáveis e rastreáveis.

O Google Drive permanece como fonte original; o PostgreSQL governa identidade, estado, relações, versões e provenance; e o armazenamento local mantém os artefatos físicos derivados que necessitem persistência.

A arquitetura deverá preservar somente aquilo que possui função documental, computacional, operacional ou de rastreabilidade relevante, descartando intermediários que possam ser reconstruídos ou que não possuam valor após seu uso.

O armazenamento local deverá possuir backup e permanecer arquiteturalmente substituível ou expansível caso o crescimento futuro do Snoopy torne isso necessário.

> **O Snoopy não deve armazenar tudo indiscriminadamente; deve preservar de maneira governável aquilo que precisa sobreviver ao processamento.**

**6.2.4 fica, portanto, consolidada.** A próxima etapa é a **6.2.5 — Segmentação**, onde vamos investigar as alternativas tecnológicas para transformar a estrutura documental preservada em unidades de recuperação adequadas, sem confundir chunk técnico com unidade metodológica.
