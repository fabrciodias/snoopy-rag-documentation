# Registro Mestre da Análise — Snoopy-RAG

## Etapa 6.2.5 — Segmentação

**Status:** Registro Mestre da Etapa 6.2.5 — consolidado
**Natureza:** Investigação tecnológica e consolidação arquitetural
**Escopo:** Segmentação estrutural, segmentação semântica, subdivisão, tamanho, overlap, contextualização, múltiplos perfis e alternativas tecnológicas
**Relação com a Etapa 6.3:** Este registro estabelece capacidades, responsabilidades e alternativas tecnológicas identificadas. Não constitui ainda a escolha definitiva da stack ou o projeto técnico final da V3.

---

## 1. Finalidade da Etapa 6.2.5

A Etapa 6.2.5 investiga como os requisitos estabelecidos nas Etapas 5.2 e 6.1 podem ser realizados tecnologicamente, preservando os princípios definidos para a arquitetura do Snoopy-RAG V3.

A questão central não é simplesmente determinar qual biblioteca produz os melhores chunks, mas identificar:

* quais capacidades tecnológicas são necessárias;
* quais estratégias de segmentação são possíveis;
* quais componentes externos podem fornecer essas capacidades;
* quais responsabilidades devem permanecer sob controle do Snoopy;
* como múltiplos perfis de segmentação podem coexistir;
* e como preservar identidade, proveniência, versionamento e reprocessabilidade ao utilizar componentes especializados.

A investigação partiu da distinção fundamental entre:

> **algoritmo de segmentação** e **governança da segmentação**.

Uma biblioteca pode fornecer um algoritmo ou capacidade de segmentação sem, por isso, tornar-se responsável pela definição metodológica ou arquitetural das unidades do Snoopy.

---

# 2. Princípio central consolidado

A investigação confirmou a pertinência da **Strategy D — motor de segmentação próprio do Snoopy**.

Entretanto, “motor próprio” não significa que o Snoopy precise implementar internamente todos os algoritmos de segmentação existentes.

O sentido arquitetural adotado é:

> **O Snoopy deve possuir um motor de segmentação próprio no sentido de governar a definição, composição, versionamento, proveniência e publicação das unidades de recuperação, mas não necessariamente implementar internamente todos os algoritmos utilizados para encontrar fronteiras ou realizar subdivisões.**

Consequentemente:

> **Algoritmos externos podem propor fronteiras, agrupamentos ou subdivisões; o Snoopy decide como essas propostas se tornam derivações identificáveis do corpus.**

Princípio operacional:

> **O algoritmo fornece capacidade; o Snoopy fornece governança.**

---

# 3. Capacidades tecnológicas necessárias

A investigação consolidou as seguintes capacidades como relevantes para a segmentação do Snoopy:

| Capacidade            | Necessidade                                                                                                                     |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Elementos estruturais | Trabalhar sobre elementos documentais identificáveis, evitando texto previamente achatado                                       |
| Agrupamento           | Combinar elementos estrutural ou semanticamente compatíveis preservando ordem                                                   |
| Subdivisão            | Dividir elementos ou conjuntos excessivamente grandes quando necessário                                                         |
| Estrutura             | Respeitar títulos, seções, tabelas, figuras, captions e outros elementos                                                        |
| Semântica             | Identificar fronteiras ou agrupamentos semanticamente plausíveis                                                                |
| Restrições de tamanho | Aplicar limites de tokens/caracteres sem permitir que esses limites definam isoladamente a unidade                              |
| Especialização        | Possibilitar tratamento específico de tabelas, figuras, equações, listas e estruturas semelhantes                               |
| Hierarquia            | Representar relações pai/filho ou outras relações estruturais                                                                   |
| Contextualização      | Produzir representações contextualizadas para embedding, geração ou visualização sem alterar a identidade documental da unidade |
| Proveniência          | Manter ligação entre unidade, elementos de origem e documento                                                                   |
| Extensibilidade       | Permitir múltiplos perfis e estratégias                                                                                         |
| Governança            | Controlar identidade, versão, configuração, publicação e invalidação                                                            |

Essas capacidades não precisam ser fornecidas por uma única tecnologia.

---

# 4. Estrutura conceitual da segmentação

A arquitetura de segmentação consolidada não deve ser entendida como uma operação única.

A composição mais adequada é:

```text
representação documental
        ↓
estrutura documental
        ↓
segmentação estrutural
        ↓
refinamento semântico
        ↓
restrições de tamanho
        ↓
contextualização
        ↓
unidade de recuperação
        ↓
representação para indexação
```

Essa sequência deve ser entendida como uma **ordem de responsabilidade**, e não necessariamente como uma sequência de bibliotecas.

O princípio é:

> **estrutura → semântica → restrição → contexto**

e não:

> **tamanho → texto → chunk**.

---

# 5. Segmentação estrutural

A segmentação estrutural constitui a base da estratégia.

Ela deve operar sobre uma representação documental que preserve elementos como:

* títulos;
* subtítulos;
* parágrafos;
* listas;
* tabelas;
* células;
* figuras;
* captions;
* equações;
* notas;
* referências;
* elementos de layout;
* relações hierárquicas;
* ordem de leitura;
* proveniência espacial.

A estrutura documental deve estabelecer as fronteiras iniciais dentro das quais outras formas de segmentação podem operar.

A principal consequência é que um segmentador semântico não deve receber, como regra geral, um documento previamente reduzido a uma sequência indiferenciada de caracteres.

---

# 6. Segmentação semântica

A segmentação semântica possui papel de **refinamento**, não de substituição da estrutura documental.

Ela é especialmente útil quando:

* uma região estruturalmente coerente é grande demais;
* uma seção contém múltiplas unidades temáticas;
* as fronteiras estruturais existentes são insuficientes;
* ou é desejável produzir unidades mais específicas para determinada finalidade de recuperação.

A segmentação semântica pode utilizar algoritmos externos, inclusive algoritmos baseados em embeddings e detecção de pontos de quebra.

Entretanto:

> **a fronteira semântica produzida por um algoritmo deve ser considerada uma proposta de segmentação, e não automaticamente uma decisão arquitetural do Snoopy.**

O Snoopy deve permanecer capaz de registrar:

* qual algoritmo foi utilizado;
* qual configuração foi utilizada;
* sobre qual representação ele operou;
* quais fronteiras foram propostas;
* qual perfil aceitou essas fronteiras;
* e qual unidade resultou da derivação.

---

# 7. Relação entre estrutura, semântica e tamanho

A investigação rejeitou a ideia de que segmentação deva ser governada primariamente por tamanho.

O tamanho é uma **restrição operacional**.

Portanto, a regra conceitual consolidada é:

1. preservar o elemento quando ele é coerente e compatível com as restrições;
2. agrupar elementos estrutural ou semanticamente compatíveis;
3. manter separados elementos incompatíveis;
4. subdividir elementos quando seu tamanho ou natureza exigir;
5. contextualizar a unidade sem alterar sua identidade;
6. utilizar overlap apenas quando necessário para reduzir perda causada por subdivisões forçadas.

Consequentemente:

> **limites de tokens ou caracteres não devem definir sozinhos o significado de uma unidade de recuperação.**

Uma unidade de 800 tokens não é metodologicamente superior a uma de 400 ou 1200 tokens simplesmente por causa do tamanho.

O tamanho deve atender às necessidades dos componentes posteriores, especialmente embedding e recuperação.

---

# 8. Overlap

O overlap foi tratado como mecanismo auxiliar, não como fundamento da segmentação.

A utilização indiscriminada de overlap pode:

* aumentar redundância;
* duplicar evidências;
* poluir unidades semanticamente coerentes;
* dificultar interpretação dos resultados;
* aumentar custo computacional.

Assim, o princípio consolidado é:

> **overlap deve ser seletivo e justificado pela situação de segmentação.**

O valor padrão pode ser zero.

Overlap pode tornar-se apropriado principalmente quando um elemento estruturalmente indivisível precisa ser subdividido por uma restrição operacional e existe risco significativo de perda de contexto entre fragmentos.

O uso de overlap deve ser tratado como parâmetro identificável do perfil de segmentação.

---

# 9. Elementos especiais

A segmentação não deve tratar todos os tipos documentais como se fossem equivalentes.

Diferentes elementos possuem diferentes propriedades de segmentação.

Exemplos:

* títulos tendem a permanecer indivisíveis;
* captions possuem relação semântica com seus objetos;
* equações devem preservar sua identidade;
* tabelas possuem estrutura própria;
* células possuem relações internas e posicionais;
* figuras devem permanecer vinculadas às respectivas captions;
* listas possuem continuidade estrutural;
* notas e referências podem possuir regras específicas.

Portanto:

> **a estratégia de segmentação deve ser sensível ao tipo estrutural do elemento.**

Isso não significa necessariamente criar um algoritmo completamente diferente para cada tipo, mas exige que o motor consiga aplicar políticas diferentes quando a natureza documental justificar.

---

# 10. Tabelas

Tabelas constituem um caso especial.

A unidade de recuperação pode envolver:

* a tabela completa;
* fragmentos da tabela;
* subconjuntos logicamente coerentes;
* células ou grupos de células;
* contexto do cabeçalho;
* relação entre tabela e caption.

Quando uma tabela precisa ser subdividida, os fragmentos devem preservar a identidade da tabela original e sua proveniência.

Cabeçalhos repetidos na representação textual de um fragmento não devem ser confundidos com duplicação da tabela original.

A distinção é:

```text
tabela original
      ↓
fragmentos derivados
      ↓
contexto de cabeçalho
      ↓
serialização textual
```

O cabeçalho pode ser repetido na serialização para tornar o fragmento interpretável, enquanto a representação canônica continua contendo uma única tabela original.

---

# 11. Unidade primária e contexto

A investigação consolidou uma distinção importante:

> **unidade de recuperação não é necessariamente igual à representação que será enviada ao embedding ou ao modelo de linguagem.**

A unidade primária deve possuir:

* identidade;
* conteúdo ou referência ao conteúdo;
* elementos de origem;
* posição;
* proveniência;
* perfil;
* versão;
* relações.

O contexto pode ser representado separadamente.

Ele pode incluir:

* título da seção;
* seção pai;
* elemento anterior;
* elemento posterior;
* caption;
* cabeçalho de tabela;
* relações documentais;
* contexto estrutural.

Isso permite:

```text
unidade
   │
   ├── conteúdo primário
   │
   ├── provenance
   │
   └── relações de contexto
            ↓
      contextualização
            ↓
     representação textual
            ↓
         embedding
```

Essa abordagem evita contaminar a identidade da unidade com uma determinada forma de serialização.

---

# 12. Contextualização como derivação

A contextualização deve ser considerada uma transformação derivada.

Portanto:

```text
unidade ≠ contextualização
```

Uma unidade pode possuir várias representações contextualizadas para finalidades diferentes.

Por exemplo:

```text
unit-123
   ├── serialization-general
   ├── serialization-embedding
   ├── serialization-reading
   └── serialization-generation
```

A mudança da contextualização não implica necessariamente uma nova segmentação.

Isso é particularmente importante para preservar reprocessabilidade.

---

# 13. Múltiplos perfis de segmentação

A investigação confirmou que o Snoopy deve poder trabalhar com **múltiplos perfis de segmentação**.

Um perfil representa uma forma identificável de derivar unidades de recuperação de determinada representação documental.

Exemplos conceituais:

```text
documento
   │
   ├── profile-general
   │      └── unidades gerais
   │
   ├── profile-fine
   │      └── unidades mais específicas
   │
   └── profile-broad
          └── unidades mais amplas
```

Os perfis podem utilizar diferentes estratégias.

Mais importante:

> **um mesmo perfil pode ser híbrido.**

Por exemplo:

```text
profile-general
    ↓
estrutura
    ↓
semântica
    ↓
limite de tokens
    ↓
contextualização
```

Portanto, “múltiplos perfis” e “segmentação híbrida” são conceitos compatíveis.

---

# 14. Identidades distintas

A investigação identificou três identidades que não devem ser confundidas:

### 14.1 Identidade documental

Representa:

> “este elemento pertence a esta versão deste documento”.

### 14.2 Identidade da unidade de segmentação

Representa:

> “este perfil definiu estes elementos ou fragmentos como uma unidade de recuperação”.

### 14.3 Representação de indexação

Representa:

> “esta unidade foi serializada/contextualizada desta maneira e transformada em uma representação matemática para recuperação”.

Consequentemente:

```text
documento
    ↓
elemento documental
    ↓
unidade de segmentação
    ↓
serialização
    ↓
embedding/index
```

Essas camadas possuem diferentes identidades e diferentes ciclos de vida.

---

# 15. Compartilhamento entre perfis

Dois perfis podem operar sobre os mesmos elementos documentais sem produzir necessariamente a mesma unidade final.

Por exemplo:

```text
elementos:
A B C D

profile A:
[A+B] [C+D]

profile B:
[A] [B+C] [D]
```

Não é necessário duplicar o conteúdo original para representar essas duas segmentações.

O compartilhamento deve ocorrer principalmente na camada de:

* elementos;
* fragmentos;
* relações documentais;
* proveniência.

As unidades finais podem permanecer independentes.

Assim:

> **o compartilhamento de unidades finais entre perfis pode ser uma otimização, mas não deve constituir requisito arquitetural.**

Isso evita que uma decisão de otimização imponha restrições à independência experimental dos perfis.

---

# 16. Fragmentação de elementos

Um elemento original pode, em determinados casos, precisar ser subdividido.

A relação conceitual é:

```text
elemento E123
       ↓
 ┌─────┼─────┐
 ↓     ↓     ↓
E123.1 E123.2 E123.3
```

Os fragmentos são derivados.

O elemento original continua existindo como referência documental.

Isso permite que:

* o conteúdo original seja preservado;
* múltiplos perfis reutilizem o mesmo elemento;
* fragmentações diferentes coexistam;
* a proveniência seja reconstruída;
* e a segmentação seja refeita sem nova extração.

---

# 17. Quando uma nova segmentação existe

Uma nova segmentação deve ser considerada quando houver alteração significativa na:

* composição das unidades;
* fronteiras das unidades;
* identidade das unidades;
* finalidade do perfil;
* estratégia de segmentação;
* ou regras que determinam a constituição das unidades.

Em contraste, alterações exclusivamente em:

* serialização;
* contextualização;
* embedding;
* modelo de representação matemática;
* índice;

não constituem necessariamente uma nova segmentação.

Isso permite distinguir:

```text
segmentação
     ↓
serialização
     ↓
embedding
     ↓
índice
```

como diferentes camadas de derivação.

---

# 18. Perfil, execução e resultado

A investigação também consolidou a necessidade de distinguir:

### Strategy

A estratégia conceitual de segmentação.

### Profile

Uma configuração identificável e versionada de determinada estratégia.

### Run

Uma execução concreta do perfil sobre determinada versão documental.

### Unit

O resultado persistente dessa execução.

Conceitualmente:

```text
Strategy
    ↓
Profile
    ↓
Run
    ↓
Units
    ↓
Serialization
    ↓
Embedding
```

Isso evita tratar qualquer combinação experimental de parâmetros como uma nova entidade arquitetural permanente.

Um experimento pode ser uma execução de um perfil sem necessariamente constituir um novo perfil persistente.

---

# 19. Responsabilidade do Snoopy

Independentemente da tecnologia escolhida, as seguintes responsabilidades devem permanecer sob controle do Snoopy:

* identidade das unidades;
* identidade dos perfis;
* versionamento;
* configuração;
* lineage;
* proveniência;
* relações entre elementos e unidades;
* políticas de agrupamento;
* políticas de subdivisão;
* tratamento dos elementos especiais;
* validação dos resultados;
* publicação;
* invalidação;
* reprocessamento;
* associação com a versão documental;
* distinção entre unidade, contexto e serialização.

Essas responsabilidades não devem ser delegadas implicitamente a uma biblioteca.

---

# 20. Alternativas tecnológicas investigadas

Foram analisadas quatro alternativas principais, além da possibilidade de implementação nativa do próprio Snoopy.

---

## 20.1 Docling

O Docling apresenta forte aderência às necessidades estruturais e híbridas identificadas.

Entre suas capacidades relevantes estão:

* representação documental estruturada;
* elementos documentais;
* hierarquia;
* tabelas;
* figuras;
* provenance;
* chunking hierárquico;
* chunking híbrido;
* refinamento orientado a tokens;
* contextualização;
* serialização;
* extensibilidade.

Seu `HybridChunker` combina uma abordagem estrutural com refinamento orientado por tokens, incluindo divisão de unidades excessivamente grandes e combinação de unidades compatíveis. A arquitetura de chunking também separa a unidade do processo de contextualização/serialização.

Isso o torna um candidato particularmente forte como **provedor de capacidades estruturais e de segmentação híbrida**.

### Limitação

O Docling possui seu próprio modelo documental e seus próprios objetos de chunking.

O Snoopy não deve assumir:

> `Docling chunk = Snoopy unit`.

O Docling pode fornecer informação e capacidade; a identidade arquitetural deve permanecer no Snoopy.

Também deve ser considerada a distinção entre a licença do código do projeto e as licenças específicas de modelos utilizados por determinados componentes.

**Papel potencial:** provedor estrutural e de segmentação híbrida.

---

## 20.2 Unstructured

O Unstructured apresenta forte alinhamento com a abordagem element-centric.

Suas capacidades relevantes incluem:

* particionamento em elementos;
* agrupamento de elementos;
* preservação de fronteiras de seção;
* tratamento de tabelas;
* subdivisão de elementos excessivamente grandes;
* limites rígidos e preferenciais;
* controle de overlap;
* referência aos elementos originais;
* preservação de metadados e localização.

A abordagem demonstra uma regra tecnologicamente coerente com a arquitetura definida:

> elementos inteiros devem ser preservados sempre que possível; subdivisão deve ocorrer quando necessária.

O `orig_elements` também demonstra uma forma prática de manter relação entre chunks e elementos de origem.

### Limitação

O modelo continua sendo o modelo do próprio componente.

O Unstructured não deve tornar-se automaticamente:

* modelo canônico do Snoopy;
* autoridade dos perfis;
* sistema de versionamento;
* sistema de provenance;
* ou governança das unidades.

**Papel potencial:** provedor alternativo de particionamento e segmentação estrutural/element-centric.

---

## 20.3 LlamaIndex

O LlamaIndex apresenta capacidades especialmente interessantes para:

* segmentação semântica;
* hierarquia;
* relações pai/filho;
* múltiplas granularidades;
* expansão contextual;
* integração com mecanismos de recuperação.

Seu `SemanticSplitterNodeParser` fornece uma abordagem baseada em similaridade para identificação de fronteiras semânticas.

Seu modelo hierárquico demonstra a viabilidade tecnológica de trabalhar com unidades menores relacionadas a unidades maiores.

### Limitação

Sua orientação é mais voltada à representação de nós para RAG e recuperação do que à representação documental rica.

Por isso, não deve ocupar o primeiro estágio da arquitetura.

Seu papel mais coerente é:

```text
estrutura documental do Snoopy
        ↓
região estrutural coerente
        ↓
LlamaIndex
        ↓
propostas de fronteira semântica
        ↓
Snoopy
```

**Papel potencial:** provedor de refinamento semântico e/ou hierarquia de recuperação.

---

## 20.4 LangChain

O LangChain possui uma função mais específica.

Seu `RecursiveCharacterTextSplitter` é adequado para:

* subdivisão de texto;
* respeito progressivo a separadores;
* imposição de limites;
* situações de overflow.

Isso o torna uma opção útil quando um elemento já identificado pelo Snoopy precisa ser dividido em fragmentos menores.

### Limitação

Não fornece, por si só, a arquitetura documental necessária.

Não deve ser responsável por:

* estrutura documental;
* tabelas;
* perfis;
* identidade;
* provenance completa;
* governança;
* publicação.

**Papel potencial:** mecanismo auxiliar/fallback para subdivisão.

---

# 21. Comparação tecnológica consolidada

| Função                  | Docling                    | Unstructured               | LlamaIndex            | LangChain          | Snoopy                |
| ----------------------- | -------------------------- | -------------------------- | --------------------- | ------------------ | --------------------- |
| Estrutura documental    | Muito forte                | Forte                      | Fraco/médio           | Fraco              | **Obrigatório**       |
| Elementos estruturais   | Muito forte                | Muito forte                | Médio                 | Fraco              | **Obrigatório**       |
| Agrupamento estrutural  | Forte                      | Forte                      | Médio                 | Fraco              | **Governado**         |
| Subdivisão              | Forte                      | Forte                      | Forte                 | Muito forte        | **Governada**         |
| Limite operacional      | Forte                      | Forte                      | Forte                 | Forte              | **Parâmetro**         |
| Semântica               | Médio/modular              | Médio                      | Muito forte           | Fraco              | **Orquestração**      |
| Hierarquia              | Forte                      | Razoável/forte             | Muito forte           | Fraco              | **Obrigatório**       |
| Tabelas                 | Muito forte                | Forte/muito forte          | Fraco                 | Fraco              | **Política própria**  |
| Contextualização        | Muito forte                | Forte                      | Muito forte           | Médio              | **Derivação própria** |
| Proveniência            | Forte                      | Forte                      | Dependente da entrada | Limitada           | **Autoridade**        |
| Múltiplos perfis        | Não é sua responsabilidade | Não é sua responsabilidade | Possível              | Possível           | **Obrigatório**       |
| Versionamento           | Não                        | Não                        | Não                   | Não                | **Obrigatório**       |
| Identidade das unidades | Externa ao Snoopy          | Externa ao Snoopy          | Node                  | Document/chunk IDs | **Snoopy**            |
| Publicação/invalidação  | Não                        | Não                        | Não                   | Não                | **Snoopy**            |

A tabela não constitui ranking absoluto das tecnologias.

Ela representa a **adequação de cada componente a determinadas responsabilidades**.

---

# 22. Composição tecnológica

A investigação demonstra que a composição é tecnicamente plausível e arquiteturalmente coerente.

Uma composição conceitual possível é:

```text
                    SNOOPY SEGMENTATION ENGINE
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
        structural          semantic          overflow
         provider            provider          provider
              │                │                │
       Docling /          LlamaIndex       LangChain /
       Unstructured        / custom         Docling
              │                │                │
              └────────────────┼────────────────┘
                               ▼
                       Snoopy validation
                               │
                       Snoopy Unit Model
                               │
                ┌──────────────┼──────────────┐
                ▼              ▼              ▼
             Profile A      Profile B      Profile C
                │              │              │
                └──────────────┼──────────────┘
                               ▼
                    contextual serialization
                               │
                           embedding
```

Essa arquitetura não significa que todas essas ferramentas deverão ser utilizadas simultaneamente.

O princípio arquitetural é:

> **a arquitetura deve permitir composição; a escolha concreta dos componentes será feita na Etapa 6.3.**

---

# 23. Implementação nativa de responsabilidades simples

A investigação também demonstrou que determinadas funções não precisam ser delegadas a frameworks externos.

É perfeitamente plausível que o Snoopy implemente internamente:

* registro de perfis;
* identidade;
* lineage;
* regras de compatibilidade;
* agrupamento básico;
* controle de limites;
* tratamento de relações;
* validação;
* publicação;
* versionamento;
* invalidação;
* serialização específica do Snoopy.

Isso não constitui “reinventar toda a tecnologia de segmentação”.

A diferença está entre implementar algoritmos especializados e implementar a **política arquitetural do sistema**.

A questão essencial não é possuir um algoritmo sofisticado para cada operação, mas definir corretamente conceitos como:

```text
compatible()
within_constraints()
should_split()
preserve_relation()
publish()
invalidate()
```

Essas decisões pertencem ao Snoopy.

---

# 24. Critério tecnológico fundamental

A investigação estabeleceu que a escolha tecnológica da Etapa 6.3 deve privilegiar não simplesmente a qualidade aparente do chunk produzido, mas:

1. preservação da estrutura;
2. possibilidade de trabalhar sobre elementos;
3. capacidade de subdivisão controlada;
4. capacidade de refinamento semântico;
5. preservação de relações;
6. proveniência;
7. possibilidade de composição;
8. reprocessabilidade;
9. independência do modelo de dados do fornecedor;
10. compatibilidade com múltiplos perfis;
11. operação local quando necessária;
12. licenciamento e dependências;
13. custo computacional;
14. capacidade de teste e validação.

O critério mais importante permanece:

> **a tecnologia deve ampliar a capacidade do Snoopy sem se tornar a autoridade sobre aquilo que o Snoopy considera uma unidade documental ou de recuperação.**

---

# 25. O que não foi decidido nesta etapa

A Etapa 6.2.5 não determina definitivamente:

* Docling como biblioteca obrigatória;
* Unstructured como biblioteca obrigatória;
* LlamaIndex como biblioteca obrigatória;
* LangChain como biblioteca obrigatória;
* utilização simultânea de todos os componentes;
* algoritmo semântico definitivo;
* modelo de embedding;
* limites definitivos de tokens;
* valores definitivos de overlap;
* formato físico definitivo das unidades;
* esquema definitivo de banco de dados;
* estratégia definitiva de indexação;
* implementação concreta do motor de segmentação.

Essas decisões pertencem à Etapa 6.3.

---

# 26. Relação com as etapas anteriores

A Etapa 6.2.5 depende diretamente das decisões estabelecidas em:

### Etapa 5.1 — Representação Canônica

A segmentação deve operar sobre uma representação documental preservada, e não substituir essa representação.

### Etapa 5.2 — Segmentação

A unidade de recuperação é uma derivação do documento, e não uma unidade metodológica.

### Etapa 5.3 — Recuperação

A segmentação deve produzir unidades identificáveis e adequadas à recuperação, sem confundir similaridade computacional com relevância metodológica.

### Etapa 5.4 — Proveniência

Toda unidade deve permanecer rastreável até sua origem documental.

### Etapa 5.5 — Persistência, Versionamento e Consistência

Perfis, unidades e derivados devem possuir estados e dependências identificáveis.

### Etapa 5.6 — Segurança, Isolamento e Operação

A segmentação deve respeitar as fronteiras de corpus, usuário, ambiente e processamento.

---

# 27. Conclusão consolidada da Etapa 6.2.5

A investigação tecnológica indica que a segmentação do Snoopy-RAG V3 deverá ser implementada como uma **arquitetura composta e governada pelo próprio Snoopy**, e não delegada integralmente a uma biblioteca externa.

A segmentação deverá partir da estrutura documental preservada, utilizando elementos identificáveis como base para agrupamento e composição. O refinamento semântico poderá ser aplicado dentro de regiões estruturalmente coerentes, enquanto limites de tokens ou caracteres deverão funcionar como restrições operacionais subordinadas à estrutura e ao significado. Overlap deverá ser tratado como mecanismo auxiliar e seletivo, não como fundamento geral da segmentação.

O Snoopy deverá possuir múltiplos perfis de segmentação, capazes de produzir diferentes unidades sobre os mesmos elementos documentais sem exigir duplicação do conteúdo original. Perfis, execuções e unidades deverão possuir identidades distintas e permitir versionamento e rastreamento de lineage.

Docling apresenta forte aderência como provedor de capacidades estruturais e de segmentação híbrida; Unstructured constitui alternativa relevante para processamento element-centric e segmentação estrutural; LlamaIndex oferece capacidades especialmente adequadas ao refinamento semântico e à hierarquia; e LangChain constitui alternativa útil para subdivisão auxiliar. Nenhum desses componentes, entretanto, deve assumir a autoridade sobre a identidade, a governança, a proveniência, o versionamento ou a publicação das unidades do Snoopy.

A conclusão arquitetural pode ser resumida em:

```text
preservar
    ↓
estruturar
    ↓
segmentar estruturalmente
    ↓
refinar semanticamente
    ↓
aplicar restrições
    ↓
contextualizar
    ↓
governar unidades no Snoopy
    ↓
serializar
    ↓
representar para recuperação
```

E o princípio central da etapa é:

> **O Snoopy deve possuir um motor de segmentação próprio no sentido de governar a derivação das unidades, podendo delegar algoritmos especializados a componentes externos. Algoritmos externos propõem; o Snoopy governa.**

---

# 28. Consequência para a Etapa 6.3

A Etapa 6.2.5 atingiu seu critério de parada.

A investigação já identificou:

* as capacidades tecnológicas necessárias;
* a ordem conceitual de composição;
* o papel de estrutura, semântica, tamanho e overlap;
* a distinção entre unidade e contexto;
* a necessidade de múltiplos perfis;
* a distinção entre elemento, unidade, serialização e embedding;
* a forma de compartilhar conteúdo entre perfis;
* a responsabilidade que deve permanecer no Snoopy;
* alternativas tecnológicas de naturezas distintas;
* os pontos fortes e limitações dessas alternativas;
* e a possibilidade de uma arquitetura composta.

Portanto, **não há necessidade de continuar catalogando ferramentas de chunking antes da Etapa 6.3**.

A partir daqui, a pergunta muda de:

> “Que tecnologias podem fazer isso?”

para:

> **“Qual combinação concreta de tecnologias, estruturas e componentes implementará essa arquitetura no Snoopy V3?”**

Essa é a questão própria da Etapa 6.3.
