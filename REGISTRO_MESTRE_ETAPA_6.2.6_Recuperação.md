# Registro Mestre da Análise — Snoopy-RAG

## Etapa 6.2.6 — Recuperação

**Status:** Registro Mestre da Etapa 6.2.6 — consolidado
**Natureza:** Investigação de alternativas tecnológicas
**Escopo:** mecanismos tecnológicos para recuperação de unidades documentais, a partir da representação e segmentação definidas nas etapas anteriores.
**Relação com etapas anteriores:** a Etapa 6.2.6 parte da unidade de recuperação estabelecida na Etapa 6.2.5 e investiga como localizá-la, combiná-la, contextualizá-la, reranqueá-la e selecioná-la.

---

## 1. Finalidade da Etapa

A Etapa 6.2.6 teve como objetivo investigar alternativas tecnológicas capazes de realizar o mecanismo de recuperação necessário ao Snoopy-RAG V3.

A investigação não teve como finalidade escolher definitivamente as tecnologias do sistema. Essa escolha pertence à Etapa 6.3 — Projeto Técnico da Solução Escolhida.

O objetivo foi identificar quais capacidades tecnológicas estão disponíveis, quais alternativas podem fornecê-las, quais limitações apresentam e como podem ser compostas para atender aos requisitos metodológicos e arquiteturais definidos anteriormente.

A investigação parte da compreensão de que o mecanismo de recuperação não deve ser reduzido a uma única operação de busca vetorial. O Snoopy necessita combinar diferentes formas de recuperação e seleção, mantendo sob sua própria governança a identidade das unidades, as relações documentais, a configuração do processo e a proveniência dos resultados.

---

# 2. Princípio fundamental

A investigação consolidou a seguinte distinção:

> **O Snoopy não deve procurar uma tecnologia que seja, sozinha, o seu mecanismo de recuperação completo. Deve possuir um mecanismo de recuperação composto, no qual tecnologias especializadas fornecem capacidades específicas enquanto o Snoopy governa sua composição e seus resultados.**

Assim, deve-se distinguir:

* **capacidade tecnológica:** aquilo que uma biblioteca, modelo ou mecanismo consegue realizar;
* **composição:** forma como diferentes capacidades são encadeadas;
* **governança do Snoopy:** definição das identidades, regras, configurações, relações, estados, provenance e publicação dos resultados.

Esse princípio mantém continuidade com a conclusão da Etapa 6.2.5:

> **algoritmos externos fornecem capacidades; o Snoopy governa a derivação e o uso dessas capacidades.**

---

# 3. Capacidades necessárias ao mecanismo de recuperação

A investigação identificou como capacidades relevantes:

1. filtragem por metadados estruturados;
2. recuperação lexical;
3. recuperação semântica;
4. recuperação híbrida;
5. combinação/fusão de resultados;
6. deduplicação por identidade documental;
7. contextualização dos candidatos;
8. reranking;
9. seleção final;
10. diversificação quando necessária;
11. preservação da provenance de todo o processo;
12. registro das configurações utilizadas;
13. possibilidade de decomposição ou transformação operacional da consulta;
14. possibilidade de utilização de diferentes estratégias conforme o tipo de consulta.

Essas capacidades não precisam ser fornecidas pela mesma tecnologia.

---

# 4. Filtragem estruturada

Uma conclusão importante da investigação foi a necessidade de separar **restrições estruturadas** de **sinais de relevância textual**.

Por exemplo, uma consulta como:

> “trabalhos de Linda Allal publicados em francês”

pode envolver:

```text
author = Linda Allal
language = fr
```

Essas condições não deveriam ser tratadas simplesmente como sinais semânticos produzidos por embeddings.

A filtragem estruturada responde à pergunta:

> **“Quais documentos ou unidades podem participar da recuperação?”**

Enquanto os mecanismos de recuperação respondem:

> **“Quais dos candidatos permitidos são potencialmente relevantes para a consulta?”**

Isso é especialmente importante para autoria.

A busca lexical pode localizar ocorrências do nome de um autor, mas a autoria propriamente dita deve ser tratada como **metadado estruturado**, quando essa informação estiver disponível e for considerada confiável.

Portanto:

> **metadados estruturados devem funcionar como restrições do universo de recuperação, e não simplesmente como mais um sinal de similaridade.**

---

# 5. Recuperação lexical

A investigação confirmou que a busca lexical possui papel próprio no Snoopy e não deve ser substituída pela busca semântica.

Ela é particularmente importante para consultas que dependem de:

* termos exatos;
* nomes de autores;
* títulos;
* expressões específicas;
* ocorrências literais;
* siglas;
* nomenclaturas;
* palavras ou frases cuja forma textual possui importância própria.

Uma possibilidade relevante é o **PostgreSQL Full-Text Search**, permitindo manter a recuperação lexical dentro da mesma infraestrutura PostgreSQL que já governa o estado lógico do Snoopy.

Também foram consideradas engines especializadas de busca, como Elasticsearch e OpenSearch, que oferecem mecanismos maduros de busca textual e integração com recuperação vetorial.

A investigação, entretanto, não encontrou razão suficiente para assumir previamente uma engine externa especializada como necessária.

---

# 6. Recuperação semântica

A recuperação semântica permanece necessária porque muitas necessidades de pesquisa não podem ser reduzidas à coincidência lexical.

O embedding permite representar a consulta e as unidades documentais em um espaço vetorial e localizar unidades semanticamente próximas.

Entretanto:

> **similaridade semântica não equivale a relevância metodológica.**

O mecanismo semântico deve ser entendido como mecanismo de geração de candidatos, e não como autoridade sobre aquilo que é metodologicamente relevante.

O `pgvector` permanece uma alternativa tecnologicamente forte para essa função, especialmente porque permite realizar busca vetorial diretamente no PostgreSQL e oferece mecanismos de busca aproximada, incluindo HNSW e IVFFlat.

Isso mantém a recuperação próxima da camada que governa os demais estados documentais e reduz a necessidade de manter uma segunda infraestrutura persistente apenas para indexação.

---

# 7. Recuperação híbrida

A investigação mostrou que lexical e semântica não devem ser entendidos como alternativas mutuamente exclusivas.

Uma arquitetura híbrida permite combinar:

```text
consulta
   ├── recuperação lexical
   └── recuperação semântica
             ↓
          candidatos
```

A combinação pode ocorrer dentro de uma engine especializada ou ser governada pelo próprio Snoopy.

Uma possibilidade particularmente relevante é:

```text
PostgreSQL Full-Text Search
          +
       pgvector
          ↓
       Snoopy
          ↓
        fusion
```

Entre os mecanismos possíveis de fusão, o **Reciprocal Rank Fusion (RRF)** mostrou-se conceitualmente interessante porque combina rankings diferentes sem exigir que seus scores tenham a mesma escala.

Assim, não é necessário presumir que um score lexical e um score vetorial possam ser combinados diretamente como se fossem grandezas equivalentes.

A investigação não determina que RRF seja obrigatoriamente utilizado; ele permanece como alternativa tecnológica relevante para a Etapa 6.3.

---

# 8. PostgreSQL como núcleo da recuperação

Uma das alternativas arquiteturais mais fortes identificadas foi utilizar o próprio PostgreSQL como núcleo de recuperação.

Nesse modelo:

```text
                 PostgreSQL
                     │
        ┌────────────┼────────────┐
        │            │            │
   metadados       FTS        pgvector
        │            │            │
        └────────────┼────────────┘
                     ↓
              Snoopy Retrieval
```

Essa alternativa apresenta uma vantagem arquitetural importante: metadados, filtros, unidades documentais, embeddings e estados lógicos permanecem próximos uns dos outros.

Isso reduz a necessidade de sincronizar uma segunda infraestrutura de busca com o estado documental governado pelo PostgreSQL.

O PostgreSQL + pgvector, portanto, **não deve ser considerado uma solução provisória que necessariamente precisará ser substituída no V3**.

É uma candidata real para a arquitetura final.

---

# 9. Engines especializadas de busca

Elasticsearch e OpenSearch apresentam capacidades fortes para:

* busca lexical;
* busca vetorial;
* hybrid search;
* fusão de rankings;
* ranking;
* mecanismos de recuperação em múltiplos estágios.

Do ponto de vista puramente funcional, são alternativas tecnicamente capazes.

Entretanto, sua adoção introduziria uma segunda infraestrutura persistente de busca:

```text
PostgreSQL
    ↕
Search Engine
```

Isso criaria uma nova relação de consistência entre o estado governado pelo PostgreSQL e o índice mantido pela engine.

Essa complexidade não constitui uma razão para descartar essas tecnologias, mas estabelece uma condição para sua adoção:

> **a capacidade adicional fornecida pela engine especializada deverá justificar a complexidade de manter uma segunda camada de estado e sincronização.**

A decisão sobre isso pertence à Etapa 6.3.

---

# 10. Reranking

A investigação identificou o reranking como uma capacidade distinta da recuperação inicial.

O padrão conceitual é:

```text
recuperação inicial
        ↓
candidatos
        ↓
contextualização
        ↓
reranking
        ↓
seleção
```

O reranker não deve substituir a recuperação inicial. Ele opera sobre um conjunto limitado de candidatos, permitindo utilizar um mecanismo mais sofisticado e potencialmente mais custoso.

Foram consideradas duas famílias principais:

### Reranker local

Modelos como os da família BGE Reranker demonstram a possibilidade de executar reranking localmente, inclusive em cenários multilíngues.

Vantagens potenciais:

* autonomia;
* privacidade;
* controle sobre o modelo;
* versionamento;
* ausência de custo por chamada;
* possibilidade de execução no próprio ambiente do Snoopy;
* possibilidade futura de especialização.

Limitações:

* consumo computacional;
* necessidade de administrar o modelo;
* necessidade de avaliar qualidade;
* necessidade de dimensionar adequadamente hardware e latência.

### Reranker externo

Serviços como Cohere Rerank fornecem reranking como serviço e oferecem modelos multilíngues capazes de trabalhar sobre candidatos recuperados por mecanismos diferentes.

Vantagens potenciais:

* qualidade e capacidade prontas para uso;
* menor responsabilidade operacional local;
* modelos especializados disponibilizados pelo fornecedor.

Limitações:

* custo;
* latência;
* dependência externa;
* transferência de dados;
* menor controle sobre mudanças de modelo e comportamento.

---

# 11. Preferência por reranking local

A investigação não estabeleceu que um reranker local seja necessariamente superior a um serviço externo.

Entretanto, considerando os objetivos arquiteturais do Snoopy, estabeleceu-se uma **preferência de investigação** por uma solução local que consiga atender adequadamente aos critérios de qualidade e custo computacional.

A lógica é:

> **primeiro investigar se um reranker local suficientemente competente pode assumir a função; somente caso existam limitações relevantes avaliar se o ganho de um serviço externo justifica a dependência adicional.**

Isso evita transformar uma tecnologia externa em requisito simplesmente porque ela apresenta bons resultados gerais.

A escolha concreta do modelo permanece para a Etapa 6.3.

---

# 12. Possibilidade de reranking híbrido

A investigação também confirmou que é tecnicamente possível combinar diferentes rerankers.

Por exemplo:

```text
                    candidatos
                        │
               ┌────────┴────────┐
               │                 │
         reranker local    reranker externo
               │                 │
               └────────┬────────┘
                        ↓
                   combinação
                        ↓
                  ranking final
```

Entretanto, essa possibilidade não deve ser transformada em requisito arquitetural inicial.

Um ensemble de rerankers somente seria justificável se os modelos produzissem sinais suficientemente complementares para compensar:

* custo computacional;
* custo financeiro;
* complexidade;
* latência;
* manutenção.

A investigação também identificou uma forma potencialmente mais interessante de hibridização: combinar diferentes **tipos de sinal**.

Por exemplo:

```text
ranking lexical
+
ranking semântico
+
score do reranker
+
restrições estruturadas
+
contexto documental
```

Nesse modelo, nem todos os sinais possuem a mesma função.

Alguns são **restrições**:

> “pode ou não participar?”

Outros são **sinais de relevância**:

> “entre os candidatos permitidos, quão relevante parece ser esta unidade?”

Essa distinção deverá ser preservada na arquitetura.

---

# 13. Contextualização antes do reranking

A investigação consolidou como hipótese arquitetural a possibilidade de contextualizar os candidatos **antes** do reranking.

O fluxo seria:

```text
consulta
   ↓
recuperação inicial
   ↓
pequeno conjunto de candidatos
   ↓
contextualização
   ↓
reranking
   ↓
seleção
```

A contextualização não deve ser aplicada indiscriminadamente ao corpus inteiro.

Ela deve ocorrer sobre um conjunto limitado de candidatos já recuperados.

O objetivo é permitir que o reranker avalie uma unidade não apenas pelo seu texto isolado, mas também considerando informações documentais relevantes, como:

* título;
* seção;
* hierarquia;
* elementos relacionados;
* contexto estrutural;
* relações com tabelas, figuras ou legendas;
* contexto documental necessário à interpretação.

A contextualização, portanto, não precisa necessariamente ser uma geração textual por LLM. Pode utilizar relações estruturais já presentes na representação documental.

A possibilidade de utilizar geração por modelo de linguagem permanece como alternativa tecnológica.

A escolha concreta da forma de contextualização pertence à Etapa 6.3.

---

# 14. Decomposição de consultas

A investigação considerou a decomposição de consultas como capacidade operacional possível.

Há pelo menos três estratégias:

### Busca direta

```text
Q → retrieval
```

### Decomposição

```text
Q
↓
Q1 + Q2 + Q3
↓
retrieval
↓
fusão
```

### Decomposição condicional

```text
Q
↓
decisão operacional
├── consulta simples → busca direta
└── consulta complexa → decomposição
```

A terceira alternativa é particularmente compatível com a natureza do Snoopy, pois evita aplicar decomposição indiscriminadamente.

Uma consulta simples, como a procura por uma definição ou por um autor, pode não exigir múltiplas consultas.

Consultas complexas, envolvendo múltiplos conceitos ou relações, podem se beneficiar da decomposição.

Quando um LLM for utilizado, sua função deve ser entendida como **transformação operacional da consulta**, e não como interpretação metodológica do problema de pesquisa.

---

# 15. Deduplicação

A deduplicação deve ocorrer principalmente pela **identidade da unidade de recuperação**, e não simplesmente pela igualdade textual.

Uma unidade pode ser recuperada por múltiplas vias:

```text
U123
 ← lexical
 ← semantic
 ← Q1
 ← Q2
```

Esses sinais não devem desaparecer simplesmente porque a unidade foi deduplicada.

A unidade continua sendo uma única unidade documental, mas sua provenance de recuperação deve registrar os caminhos pelos quais foi encontrada.

Também deve ser distinguida:

* duplicação de identidade;
* redundância semântica;
* seleção final.

Duas unidades diferentes podem expressar conteúdo semelhante sem serem a mesma unidade documental.

Portanto:

> **deduplicação de identidade e controle de redundância semântica são problemas diferentes.**

---

# 16. Seleção e diversificação

Ranking e seleção não são a mesma operação.

O ranking produz uma ordenação.

A seleção decide quais resultados efetivamente serão apresentados ou encaminhados à síntese.

A seleção pode considerar:

* posição no ranking;
* limiar;
* quantidade máxima;
* redundância;
* diversidade;
* cobertura de diferentes aspectos da consulta.

Mecanismos como MMR (Maximal Marginal Relevance) constituem alternativas tecnológicas possíveis para reduzir redundância.

Entretanto, diversidade não deve substituir relevância.

A seleção continua sendo uma operação computacional de recuperação e apresentação, e não uma decisão metodológica do pesquisador.

---

# 17. Arquitetura composta de recuperação

A investigação convergiu para o seguinte modelo funcional:

```text
                    CONSULTA
                       │
             interpretação operacional
                       │
          ┌────────────┴────────────┐
          │                         │
    restrições estruturadas     necessidade textual
          │                         │
          ▼                         ▼
       filtros               lexical + semântico
          │                         │
          └────────────┬────────────┘
                       ▼
                    fusion
                       ▼
                  candidatos
                       ▼
               deduplicação
                       ▼
              contextualização
                       ▼
                  reranking
                       ▼
              seleção/diversificação
                       ▼
                   evidência
```

Esse fluxo não representa ainda a implementação definitiva.

Ele representa a **composição funcional que as tecnologias escolhidas na Etapa 6.3 deverão ser capazes de realizar**.

---

# 18. Alternativas arquiteturais que permanecem para a Etapa 6.3

A investigação deixou três alternativas principais como candidatas reais:

### A — PostgreSQL-centric

```text
PostgreSQL
 ├── metadados
 ├── filtros
 ├── Full-Text Search
 └── pgvector
          ↓
       Snoopy
```

### B — Search-engine-centric

```text
PostgreSQL
     ↕
Elasticsearch/OpenSearch
     ↓
retrieval/ranking
```

Essa alternativa possui alta capacidade tecnológica, mas introduz maior complexidade de infraestrutura e consistência.

### C — PostgreSQL + componentes especializados

```text
PostgreSQL
   ↓
FTS + pgvector
   ↓
hybrid retrieval
   ↓
contextualização
   ↓
reranker local*
   ↓
Snoopy selection
```

`*` Reranker local é uma preferência de investigação, não uma escolha definitiva.

A alternativa C pode coexistir com a A, pois o pipeline composto não exige abandonar o PostgreSQL.

---

# 19. Papel do Snoopy

A investigação consolidou que o Snoopy deve permanecer responsável por:

* governar a consulta;
* aplicar restrições estruturadas;
* definir as estratégias de recuperação utilizadas;
* identificar unidades;
* combinar resultados;
* preservar sinais de recuperação;
* deduplicar por identidade;
* contextualizar candidatos;
* controlar o reranking;
* realizar seleção;
* manter provenance;
* registrar configurações;
* versionar estratégias;
* controlar estados;
* preservar as relações entre consulta, unidade e documento.

Componentes externos ou especializados podem fornecer:

* busca lexical;
* busca vetorial;
* fusão;
* algoritmos de reranking;
* algoritmos de expansão/decomposição;
* mecanismos de diversidade;
* outros mecanismos especializados.

A regra transversal é:

> **a tecnologia fornece capacidade; o Snoopy fornece governança.**

---

# 20. Proveniência da recuperação

O mecanismo de recuperação deverá ser capaz de preservar uma cadeia semelhante a:

```text
consulta original
      ↓
transformação/decomposição
      ↓
perfil de recuperação
      ↓
recuperação lexical/semântica
      ↓
candidatos
      ↓
deduplicação
      ↓
contextualização
      ↓
reranking
      ↓
seleção
      ↓
unidade de evidência
      ↓
elemento documental
      ↓
página
      ↓
documento
      ↓
fonte original
```

Essa cadeia não é apenas uma preocupação de auditoria.

Ela permite compreender **como uma evidência chegou ao resultado apresentado**, preservando a distinção entre o documento original e as operações realizadas pelo sistema sobre ele.

---

# 21. Relação com a Etapa 6.3

A Etapa 6.2.6 não escolhe definitivamente:

* PostgreSQL ou Elasticsearch/OpenSearch;
* algoritmo de fusion;
* modelo específico de reranking;
* local ou externo como decisão final;
* modelo específico de decomposição;
* mecanismo específico de diversificação;
* configuração de thresholds;
* valores de top-K;
* parâmetros finais de ranking.

Essas escolhas serão feitas na Etapa 6.3 a partir dos critérios técnicos estabelecidos na Etapa 6.1 e das alternativas investigadas nesta etapa.

A Etapa 6.3 deverá, portanto, transformar:

> **capacidade necessária → alternativa tecnológica → composição concreta → arquitetura técnica.**

---

# 22. Estado epistemológico

As conclusões desta etapa devem ser entendidas como **consolidação de alternativas tecnológicas**, e não como demonstração empírica de superioridade.

Não foi objetivo desta etapa provar que:

* um determinado reranker é melhor;
* hybrid search é quantitativamente superior;
* uma engine é mais rápida;
* uma determinada configuração produz maior Recall;
* uma estratégia é mais estável.

Essas questões pertencem à implementação e validação posteriores.

A documentação da arquitetura poderá declarar capacidades projetadas; os testes da Etapa 6.5 deverão demonstrar o comportamento efetivamente observado.

---

# 23. Princípio consolidado da Etapa 6.2.6

> **O mecanismo de recuperação do Snoopy deverá ser composto por capacidades distintas de filtragem estruturada, recuperação lexical e semântica, combinação de candidatos, contextualização, reranking e seleção. Essas capacidades não precisam ser fornecidas por uma única tecnologia. O Snoopy deverá governar sua composição, identidade, configuração, estados e provenance, enquanto componentes especializados poderão fornecer mecanismos específicos.**
>
> **A recuperação lexical e a recuperação semântica devem ser tratadas como capacidades complementares, permitindo uma estratégia híbrida quando apropriado. Restrições estruturadas, como autoria e idioma, devem permanecer distintas dos sinais de relevância textual.**
>
> **O reranking constitui uma etapa distinta da recuperação inicial e poderá receber candidatos previamente contextualizados. A execução local de um reranker deve ser investigada como alternativa prioritária, buscando preservar autonomia, controle, privacidade e reprodutibilidade; serviços externos permanecem como alternativas caso apresentem vantagem qualitativa ou funcional suficiente para justificar sua dependência.**
>
> **A possibilidade de combinar múltiplos sinais ou múltiplos rerankers existe, mas não constitui requisito arquitetural inicial.**
>
> **A arquitetura concreta do mecanismo de recuperação será definida na Etapa 6.3, a partir das alternativas tecnológicas identificadas e dos critérios técnicos estabelecidos anteriormente.**

---

## 24. Encerramento

A Etapa 6.2.6 encerra a investigação das alternativas tecnológicas fundamentais para a recuperação.

O resultado não é uma escolha definitiva de ferramentas, mas um espaço tecnológico suficientemente delimitado para permitir o projeto da solução concreta.

A investigação estabeleceu que o Snoopy V3 não deve ser concebido como um simples mecanismo de busca vetorial nem como uma dependência de uma única plataforma de recuperação.

Ele deverá ser concebido como um **pipeline de recuperação composto e governado pelo próprio Snoopy**, no qual filtros estruturados, recuperação lexical, recuperação semântica, fusão, contextualização, reranking e seleção desempenham funções distintas e rastreáveis.

A questão que permanece para a Etapa 6.3 deixa de ser:

> “Que tecnologias existem para fazer busca?”

e passa a ser:

> **“Qual composição tecnológica atende melhor aos requisitos do Snoopy V3, dadas suas necessidades documentais, metodológicas, operacionais e de infraestrutura?”**
