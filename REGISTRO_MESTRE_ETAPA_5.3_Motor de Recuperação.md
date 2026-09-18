


# Registro Mestre da Análise — Snoopy-RAG

## Etapa 5.3 — Motor de Recuperação

**Status:** Especificação metodológica e arquitetural — não constitui implementação.
**Relação com as etapas anteriores:** continuidade direta das Etapas 4, 5.1 e 5.2.
**Função deste registro:** estabelecer os princípios, requisitos e propriedades que o futuro motor de recuperação deverá satisfazer, sem ainda escolher bibliotecas, modelos, algoritmos ou infraestrutura específicos.

---

# 5.3 — Motor de Recuperação

## 5.3.1. Objetivo

A Etapa 5.3 trata da transformação de uma pergunta de pesquisa em um conjunto de evidências documentais recuperadas do corpus.

O problema não é simplesmente:

> **“Qual trecho possui o embedding mais parecido com a pergunta?”**

A questão arquitetural é:

> **Como transformar uma consulta em uma recuperação de evidências suficientemente relevante, estável, rastreável e avaliável, sem confundir similaridade semântica com relevância metodológica?**

O motor de recuperação deve ser entendido como uma camada intermediária entre:

```text
PESQUISA DO USUÁRIO
        ↓
      CONSULTA
        ↓
    RECUPERAÇÃO
        ↓
EVIDÊNCIAS DOCUMENTAIS
        ↓
    PESQUISADOR
```

Ele não deve ser tratado como o componente responsável por decidir o significado metodológico definitivo da evidência.

---

# 5.3.2. Contexto herdado das etapas anteriores

As etapas anteriores estabeleceram uma sequência arquitetural:

```text
Documento original
        ↓
Representação canônica
        ↓
Estrutura documental
        ↓
Segmentação
        ↓
Unidades de recuperação
        ↓
Recuperação
        ↓
Evidência
        ↓
Pesquisador
```

A Etapa 5.3 atua especificamente a partir das **unidades de recuperação** definidas na Etapa 5.2.

Portanto, o motor de recuperação não deve precisar reconstruir o documento original nem inferir novamente sua estrutura.

Sua responsabilidade é selecionar, entre as unidades disponíveis, aquelas que apresentam maior potencial de responder à consulta.

---

# 5.3.3. O problema central: similaridade não é relevância

O mecanismo vetorial atual trabalha com uma noção de proximidade entre embeddings.

Isso é útil porque permite localizar unidades semanticamente próximas de uma consulta.

Entretanto:

> **similaridade semântica não é equivalente a relevância metodológica.**

Um trecho pode ser semanticamente muito parecido com uma pergunta e ainda assim ser inadequado.

Por exemplo, uma consulta sobre:

> “avaliação formativa e regulação da aprendizagem”

pode recuperar:

* uma definição;
* uma crítica;
* um exemplo;
* uma discussão histórica;
* uma comparação;
* uma ocorrência incidental dos termos.

Todos podem apresentar alta similaridade.

Mas somente alguns podem responder efetivamente à questão que o pesquisador está investigando.

Portanto, o mecanismo de recuperação deve ser entendido como:

> **mecanismo de localização de candidatos relevantes**

e não:

> **mecanismo automático de determinação da relevância metodológica.**

---

# 5.3.4. O papel do embedding

O embedding é uma representação matemática utilizada para comparar semanticamente uma consulta e unidades documentais.

Sua função é permitir uma operação do tipo:

```text
consulta
   ↓
representação vetorial
   ↓
comparação
   ↓
unidades semanticamente próximas
```

O embedding não deve ser tratado como:

* evidência;
* interpretação;
* classificação metodológica;
* unidade de registro;
* garantia de relevância;
* garantia de recuperação completa.

Ele é um **instrumento de representação para recuperação**.

Consequentemente:

> **a qualidade do motor de recuperação não pode ser avaliada apenas pela qualidade do modelo de embedding.**

---

# 5.3.5. A recuperação começa na consulta

O sistema atual possui uma etapa de decomposição da pergunta em microconsultas.

Conceitualmente:

```text
pergunta original
        ↓
decomposição
        ↓
microconsultas
        ↓
embeddings
        ↓
buscas
        ↓
resultados
```

Essa estratégia pode ser útil para perguntas compostas.

Uma pergunta como:

> “Como Autor A define X e como Autor B relaciona Y a Z?”

pode conter múltiplos núcleos de informação.

Uma única representação vetorial pode não representar adequadamente todos eles.

A decomposição pode produzir:

```text
Q1 = Autor A + X

Q2 = Autor B + Y

Q3 = relação Y + Z
```

Isso aumenta a capacidade de procurar diferentes núcleos conceituais.

Entretanto, também cria um novo problema:

> **a própria decomposição passa a fazer parte do processo de recuperação.**

Se a decomposição mudar, os resultados podem mudar.

Portanto, ela deve ser tratada como uma etapa explícita e versionável do perfil de recuperação.

---

# 5.3.6. Decomposição não deve destruir a pergunta original

A consulta original precisa permanecer preservada.

A arquitetura deve manter:

```text
consulta original
        │
        ├── consulta derivada 1
        ├── consulta derivada 2
        └── consulta derivada 3
```

e não apenas:

```text
consulta derivada 1
consulta derivada 2
consulta derivada 3
```

Isso é necessário para:

* rastreabilidade;
* auditoria;
* comparação entre execuções;
* avaliação da decomposição;
* reprodução da busca.

O sistema deve ser capaz de responder:

> **“Quais consultas efetivamente foram utilizadas para procurar as evidências apresentadas?”**

---

# 5.3.7. A decomposição como hipótese de busca

A decomposição deve ser compreendida como uma **transformação operacional da pergunta**, e não como interpretação metodológica definitiva.

O sistema pode interpretar uma pergunta de maneira diferente daquela pretendida pelo pesquisador.

Por isso:

```text
pergunta original
      ↓
interpretação computacional
      ↓
consultas derivadas
      ↓
recuperação
```

deve permanecer auditável.

O pesquisador precisa poder distinguir:

> “eu perguntei isso”

de:

> “o sistema procurou isso”.

Essa diferença será particularmente importante quando a recuperação produzir resultados inesperados.

---

# 5.3.8. Estratégia de consulta

A arquitetura deve permitir que uma pergunta possa ser processada por uma ou mais estratégias de consulta.

Conceitualmente, podem existir:

```text
consulta original
```

ou:

```text
consulta original
   ↓
decomposição
   ↓
consultas especializadas
```

ou ainda combinações de diferentes representações da mesma pergunta.

A etapa atual não determina qual estratégia será escolhida.

O requisito é:

> **a estratégia efetivamente utilizada deve ser identificável e versionável.**

---

# 5.3.9. Busca por unidade de recuperação

A recuperação deve operar sobre as unidades definidas na Etapa 5.2.

Portanto:

```text
consulta
   ↓
representação da consulta
   ↓
comparação
   ↓
unidades de recuperação
```

Cada resultado deve permanecer associado à unidade original.

Não deve existir uma etapa em que o sistema transforme o resultado em um texto sem identidade documental e continue trabalhando apenas com esse texto.

---

# 5.3.10. Threshold de similaridade

O uso de um limite mínimo de similaridade pode impedir que resultados muito distantes da consulta sejam recuperados.

Conceitualmente:

```text
similaridade >= limiar
        ↓
candidato
```

Entretanto, o valor do threshold não possui significado absoluto.

Um valor de similaridade depende de:

* representação vetorial;
* modelo utilizado;
* dimensão;
* corpus;
* distribuição dos embeddings;
* natureza das consultas;
* unidade de recuperação;
* métrica utilizada.

Portanto:

> **um threshold não deve ser tratado como constante universal de relevância.**

Ele deve ser considerado uma configuração do perfil de recuperação e avaliado empiricamente.

---

# 5.3.11. Top-K

A quantidade de resultados recuperados também deve ser explicitamente configurável.

O top-K responde à pergunta:

> **Quantas unidades candidatas devem ser consideradas antes das etapas posteriores de seleção e síntese?**

Um K muito pequeno pode:

* perder evidências relevantes;
* aumentar o risco de omissão;
* favorecer resultados superficiais.

Um K muito grande pode:

* aumentar ruído;
* aumentar custo;
* sobrecarregar a síntese;
* dificultar a avaliação;
* aumentar duplicação.

Portanto:

> **top-K é uma propriedade do experimento de recuperação, não uma constante metodológica universal.**

---

# 5.3.12. Threshold e top-K possuem funções diferentes

É importante não tratar threshold e top-K como mecanismos equivalentes.

### Threshold

Define uma condição mínima de similaridade.

### Top-K

Define quantos resultados podem ser selecionados entre os candidatos.

Conceitualmente:

```text
corpus
  ↓
candidatos
  ↓
threshold
  ↓
resultados elegíveis
  ↓
ranking
  ↓
top-K
```

A ordem exata dessas operações dependerá da arquitetura técnica futura.

O requisito é que seu comportamento seja explícito e documentado.

---

# 5.3.13. Ranking

A recuperação não termina necessariamente na obtenção de candidatos.

É necessário estabelecer como os candidatos serão ordenados.

O ranking pode considerar diferentes propriedades, dependendo da arquitetura futura.

O princípio importante é:

> **o critério de ordenação deve ser conhecido.**

Não basta dizer:

> “o sistema encontrou os dez melhores resultados.”

É necessário poder determinar:

* segundo qual métrica;
* com qual configuração;
* a partir de qual consulta;
* sobre qual versão das unidades;
* utilizando qual representação.

---

# 5.3.14. Deduplicação

Quando uma pergunta é decomposta em múltiplas consultas, uma mesma unidade pode aparecer várias vezes.

Por exemplo:

```text
Q1 → unidade A
Q2 → unidade A
Q3 → unidade B
```

A arquitetura precisa evitar que a mesma evidência ocupe artificialmente várias posições apenas porque foi encontrada por consultas diferentes.

Entretanto, deduplicação não deve ser confundida com remoção indiscriminada de conteúdo semelhante.

É necessário distinguir:

```text
mesma unidade documental
```

de:

```text
unidades documentais diferentes contendo informação semelhante
```

e também de:

```text
unidades vizinhas que representam partes complementares da mesma discussão.
```

---

# 5.3.15. A identidade da unidade deve ser preferida ao texto como chave

A deduplicação idealmente deve operar sobre a identidade da unidade recuperada quando isso for possível.

Isso é conceitualmente superior a considerar dois textos iguais somente quando suas strings são iguais.

Por exemplo:

```text
Unidade A
página 12
bloco 8

Unidade B
página 13
bloco 9
```

podem conter texto muito semelhante e ainda assim serem duas ocorrências documentais distintas.

Da mesma forma, a mesma unidade pode aparecer em várias microconsultas e deve continuar sendo uma única unidade documental.

---

# 5.3.16. Contexto complementar

A unidade recuperada pode não ser suficiente para interpretação.

Nesse caso, o motor deve poder recuperar contexto adicional.

A arquitetura conceitual é:

```text
consulta
   ↓
unidade principal
   ↓
contexto estrutural/adjacente
```

O ponto importante é que o contexto adicional não deve substituir a unidade que motivou a recuperação.

Deve ser possível distinguir:

```text
resultado principal
+
contexto complementar
```

Isso preserva a capacidade de responder:

> **“Por que este trecho foi recuperado?”**

---

# 5.3.17. Recuperação de contexto não é expansão indiscriminada

Adicionar contexto pode melhorar a compreensão, mas também pode introduzir ruído.

Não se deve simplesmente recuperar:

```text
+ 5 chunks antes
+ 5 chunks depois
```

sem considerar estrutura e finalidade.

O contexto deve respeitar, quando possível:

* limites estruturais;
* continuidade;
* relação entre unidades;
* página;
* seção;
* tipo de elemento.

A expansão contextual deve ser uma decisão arquitetural explícita.

---

# 5.3.18. Recuperação e estrutura documental

A recuperação deve utilizar a estrutura preservada na Etapa 5.1 e produzida na Etapa 5.2.

Isso significa que o sistema deve ser capaz, conceitualmente, de recuperar não apenas:

> “um texto parecido com a pergunta”

mas:

> “uma unidade documental identificada, pertencente a determinada seção, página e estrutura do documento”.

Essa informação será fundamental para a apresentação da evidência ao pesquisador.

---

# 5.3.19. Filtros de corpus

A recuperação deve respeitar os limites do corpus selecionado.

No Snoopy, isso é especialmente importante porque existem diferentes contextos de acervo.

Conceitualmente:

```text
usuário
   ↓
acervo/pasta autorizada
   ↓
unidades elegíveis
   ↓
recuperação
```

O mecanismo não deve buscar unidades fora do escopo documental autorizado apenas porque apresentam alta similaridade.

Portanto:

> **o escopo documental é parte da definição da consulta.**

---

# 5.3.20. Filtro de pasta não é detalhe de implementação

O filtro de corpus possui significado metodológico.

Se o pesquisador escolhe:

```text
Corpus A
```

e o sistema recupera uma evidência de:

```text
Corpus B
```

o resultado não representa a investigação originalmente definida.

Portanto:

> **a fronteira do corpus deve ser preservada e registrada como parte da execução da recuperação.**

Isso também é necessário para reprodução posterior.

---

# 5.3.21. Corpus como parte do perfil de recuperação

Uma recuperação não deve ser descrita apenas como:

```text
pergunta X
```

mas como algo mais próximo de:

```text
pergunta X
+
corpus Y
+
versão do corpus
+
configuração Z
+
pipeline de recuperação W
```

O mesmo texto consultado em dois corpora diferentes constitui dois experimentos de recuperação diferentes.

---

# 5.3.22. Ordenação determinística

A recuperação precisa considerar o problema de empates.

Imagine:

```text
A → 0,812
B → 0,801
C → 0,801
D → 0,799
```

Se C e B possuírem exatamente a mesma similaridade, a ordem entre eles precisa ser determinada por algum critério estável.

Caso contrário, duas execuções aparentemente idênticas podem produzir:

```text
B, C, D
```

em uma execução e:

```text
C, B, D
```

em outra.

Isso pode alterar:

* top-K;
* contexto enviado à síntese;
* evidências apresentadas;
* resposta final.

Portanto:

> **a ordenação precisa possuir um mecanismo explícito de desempate quando a estabilidade da recuperação for requisito.**

---

# 5.3.23. Estabilidade da recuperação

A arquitetura deve distinguir:

### Determinismo

Mesma entrada e mesma configuração produzem o mesmo resultado.

### Estabilidade

Pequenas variações controladas produzem resultados suficientemente consistentes.

### Reprodutibilidade

Outra execução posterior consegue reproduzir as condições necessárias para obter resultados comparáveis.

Esses conceitos não são equivalentes.

Um mecanismo pode ser tecnicamente determinístico em uma execução e ainda assim não ser reproduzível se:

* o corpus mudou;
* o modelo mudou;
* a configuração mudou;
* as unidades mudaram;
* o índice mudou;
* a consulta derivada mudou.

---

# 5.3.24. Variabilidade introduzida pelo LLM

A decomposição da consulta utiliza um modelo de linguagem.

Mesmo com temperatura baixa, ela constitui uma etapa potencialmente variável do pipeline.

Isso significa que:

```text
mesma pergunta
```

pode eventualmente gerar:

```text
Q1 + Q2
```

em uma execução e:

```text
Q1 + Q2 + Q3
```

em outra.

Consequentemente, a recuperação pode mudar sem qualquer alteração no corpus.

Esse comportamento precisa ser tratado como parte da arquitetura de recuperação, e não como um detalhe irrelevante.

---

# 5.3.25. O perfil de recuperação

Para permitir avaliação e reprodução, cada execução relevante deve ser conceitualmente associável a um **perfil de recuperação**.

Esse perfil deve identificar, no mínimo, propriedades como:

```text
consulta original
consultas derivadas
corpus
versão do corpus
representação das unidades
estratégia de segmentação
modelo de embedding
dimensão vetorial
métrica de similaridade
threshold
top-K
estratégia de ranking
deduplicação
estratégia de contexto
versão do pipeline
```

A lista exata de campos será definida posteriormente.

O princípio é:

> **uma recuperação precisa ser descrita por mais do que sua pergunta.**

---

# 5.3.26. Versionamento do embedding

O modelo utilizado para produzir os embeddings é parte da identidade da representação vetorial.

Se o modelo mudar:

```text
documento
 ↓
embedding A
```

pode se tornar:

```text
documento
 ↓
embedding B
```

Mesmo que o documento não tenha mudado.

Portanto, não se deve considerar embeddings como propriedades permanentes do documento.

Eles são derivados de:

```text
unidade
+
modelo
+
configuração
```

Consequentemente, a versão do modelo de embedding deve integrar o perfil de recuperação.

---

# 5.3.27. Dimensão vetorial

A dimensão do vetor também deve ser tratada como propriedade da representação.

Uma mudança de dimensão pode implicar mudança de representação e de infraestrutura de indexação.

Portanto:

```text
modelo
+
dimensão
+
representação
```

devem permanecer coerentes.

A arquitetura não deve permitir que diferentes versões de embeddings sejam tratadas como se fossem necessariamente equivalentes.

---

# 5.3.28. Atualização do corpus

Quando um documento é:

* adicionado;
* alterado;
* removido;
* substituído;

a recuperação pode mudar.

Isso não é necessariamente uma falha.

É consequência da natureza do corpus.

Por isso, a arquitetura precisa permitir distinguir:

```text
mudança na pergunta
```

de:

```text
mudança no corpus
```

e de:

```text
mudança no mecanismo de recuperação.
```

Essas três alterações podem produzir resultados diferentes e possuem significados distintos.

---

# 5.3.29. Invalidação de representações derivadas

Quando uma unidade documental muda, suas representações derivadas podem deixar de corresponder ao documento atual.

Isso inclui:

* segmentação;
* embeddings;
* índices;
* resultados armazenados;
* cache.

A arquitetura deve permitir identificar quando uma representação derivada deixou de ser válida.

Esse problema será aprofundado na Etapa 5.5.

---

# 5.3.30. Cache

O cache pode melhorar desempenho, mas introduz uma variável adicional na análise do comportamento do sistema.

Uma consulta aparentemente repetida pode:

```text
consulta
 ↓
cache
 ↓
resultado anterior
```

em vez de executar novamente:

```text
consulta
 ↓
decomposição
 ↓
embedding
 ↓
recuperação
```

Isso é útil operacionalmente, mas pode mascarar alterações no sistema durante testes.

Portanto:

> **o estado do cache deve ser distinguível durante avaliação e diagnóstico.**

Idealmente, deve ser possível saber se determinado resultado foi:

* calculado novamente;
* recuperado do cache;
* parcialmente reutilizado.

---

# 5.3.31. Busca vetorial e busca híbrida

A arquitetura atual é predominantemente baseada em representação vetorial.

Entretanto, alguns tipos de consulta podem depender fortemente de correspondência lexical exata.

Exemplos conceituais:

* nome de autor;
* título;
* sigla;
* termo técnico;
* expressão específica;
* identificador;
* fórmula;
* palavra rara.

Uma busca puramente semântica pode não ser ideal em todos esses casos.

Por isso, a arquitetura deve manter aberta a possibilidade de combinar diferentes mecanismos de recuperação.

Isso não significa determinar agora que a busca híbrida será implementada.

Significa reconhecer como requisito:

> **o motor deve poder acomodar diferentes estratégias de recuperação quando a avaliação demonstrar que uma única forma de busca é insuficiente.**

---

# 5.3.32. Recuperação lexical e semântica possuem funções diferentes

Uma busca lexical tende a responder melhor à pergunta:

> “Onde aparece exatamente este termo?”

Uma busca semântica tende a responder:

> “Onde existe conteúdo conceitualmente semelhante?”

Essas propriedades podem ser complementares.

A arquitetura futura poderá, portanto, avaliar diferentes combinações sem assumir antecipadamente qual será superior.

---

# 5.3.33. Reranking

A recuperação inicial pode produzir um conjunto de candidatos.

Uma etapa posterior pode reordená-los segundo critérios mais sofisticados.

Conceitualmente:

```text
consulta
   ↓
recuperação inicial
   ↓
candidatos
   ↓
reranking
   ↓
resultados finais
```

O reranking deve ser entendido como uma possível camada de seleção, não como requisito de uma tecnologia específica.

Se utilizado, deverá ser:

* identificável;
* versionável;
* avaliável;
* separável da recuperação inicial.

---

# 5.3.34. Recuperação em múltiplos estágios

A arquitetura, portanto, deve admitir que a recuperação possa ser composta por múltiplas etapas:

```text
CONSULTA
   ↓
DECOMPOSIÇÃO
   ↓
GERAÇÃO DE CANDIDATOS
   ↓
FILTRAGEM
   ↓
RANKING
   ↓
DEDUPLICAÇÃO
   ↓
RERANKING
   ↓
CONTEXTO
   ↓
EVIDÊNCIAS
```

Nem todas essas etapas precisam existir na implementação final.

O princípio é que elas devem ser **conceitualmente separáveis**, permitindo identificar onde determinada decisão ocorreu.

---

# 5.3.35. Recuperação de evidência versus síntese

O motor de recuperação termina, conceitualmente, na seleção das evidências.

A geração da resposta textual é outra etapa.

Portanto:

```text
RECUPERAÇÃO
       ↓
EVIDÊNCIAS
       ↓
SÍNTESE
```

e não:

```text
PERGUNTA
   ↓
LLM
   ↓
“evidências”
```

A síntese não deve ser usada para esconder deficiências na recuperação.

Se o motor recuperou evidências ruins, uma resposta aparentemente convincente não transforma essas evidências em boas.

---

# 5.3.36. A recuperação deve ser avaliável sem o LLM de síntese

Esse é um requisito importante.

Deve ser possível avaliar:

```text
consulta
   ↓
recuperação
   ↓
resultados
```

sem depender da qualidade da resposta final produzida pelo LLM.

Isso permite distinguir:

### Falha de recuperação

A evidência correta não foi recuperada.

### Falha de seleção

A evidência estava entre os candidatos, mas não foi selecionada.

### Falha de síntese

A evidência correta foi fornecida ao LLM, mas a resposta foi inadequada.

Esses problemas possuem causas diferentes e precisam ser diagnosticados separadamente.

---

# 5.3.37. Relevância em camadas

Uma unidade pode ser:

1. semanticamente semelhante à consulta;
2. recuperada;
3. contextualmente adequada;
4. documentalmente relevante;
5. metodologicamente relevante.

Essas propriedades não são equivalentes.

Podemos representá-las como:

```text
similaridade
     ↓
recuperabilidade
     ↓
adequação contextual
     ↓
relevância documental
     ↓
relevância metodológica
```

O Snoopy atua principalmente nas primeiras camadas.

A última permanece sob responsabilidade do pesquisador.

---

# 5.3.38. Recuperação e auditabilidade

Cada resultado relevante deve poder responder:

> **Por que ele apareceu?**

A resposta deve ser reconstruível a partir de informações como:

* consulta original;
* consulta derivada;
* corpus;
* unidade recuperada;
* representação utilizada;
* configuração;
* ranking;
* localização documental.

O objetivo não é necessariamente produzir uma explicação causal completa do comportamento matemático do modelo.

É permitir uma **trilha operacional de recuperação**.

---

# 5.3.39. A cadeia de rastreabilidade da recuperação

A arquitetura deverá permitir reconstruir:

```text
PERGUNTA ORIGINAL
       ↓
CONSULTAS DERIVADAS
       ↓
CONFIGURAÇÃO DE RECUPERAÇÃO
       ↓
CANDIDATOS
       ↓
UNIDADES SELECIONADAS
       ↓
BLOCOS
       ↓
PÁGINAS
       ↓
DOCUMENTOS
       ↓
ARQUIVOS ORIGINAIS
```

Essa cadeia conecta diretamente a Etapa 5.3 à Etapa 5.4.

---

# 5.3.40. O que significa “reproduzir uma busca”

A reprodução de uma busca não deve ser definida como:

> “o LLM precisa gerar exatamente a mesma resposta”.

O alvo mais importante é:

> **sob o mesmo corpus, configuração e versões relevantes, o sistema deve conseguir recuperar evidências equivalentes ou suficientemente estáveis.**

A síntese textual pode variar sem que isso necessariamente represente falha da recuperação.

---

# 5.3.41. Reprodutibilidade em camadas

A arquitetura deve reconhecer pelo menos:

### Camada 1 — Corpus

Qual conjunto de documentos estava disponível?

### Camada 2 — Representação

Como os documentos e unidades estavam representados?

### Camada 3 — Consulta

Qual pergunta foi feita e como foi decomposta?

### Camada 4 — Recuperação

Quais configurações foram utilizadas?

### Camada 5 — Resultado

Quais unidades foram recuperadas?

### Camada 6 — Síntese

Como o LLM transformou essas unidades em resposta?

O objetivo principal do motor de recuperação é tornar as camadas 1–5 suficientemente rastreáveis e avaliáveis.

---

# 5.3.42. Requisitos arquiteturais da Etapa 5.3

A partir dos problemas identificados, estabelecem-se os seguintes requisitos.

### RRET1 — Recuperação como camada independente

O mecanismo de recuperação deve ser conceitualmente separado da síntese por LLM.

### RRET2 — Consulta original preservada

A pergunta original deve permanecer associada à execução.

### RRET3 — Consultas derivadas rastreáveis

Quando houver decomposição, as consultas derivadas devem ser preservadas.

### RRET4 — Estratégia de consulta explícita

A estratégia utilizada para transformar a pergunta em consultas deve ser identificável.

### RRET5 — Unidade de recuperação

A busca deve operar sobre unidades de recuperação identificáveis.

### RRET6 — Similaridade ≠ relevância

A similaridade semântica não deve ser tratada como garantia de relevância metodológica.

### RRET7 — Corpus explícito

A recuperação deve respeitar o corpus selecionado.

### RRET8 — Fronteira documental

Resultados fora do escopo do corpus não devem participar da recuperação.

### RRET9 — Threshold configurável

O limiar de similaridade deve ser uma configuração explícita.

### RRET10 — Top-K configurável

A quantidade de resultados deve ser configurável.

### RRET11 — Ranking explícito

O critério de ordenação dos resultados deve ser identificável.

### RRET12 — Desempate estável

Quando necessário, empates devem possuir critério de ordenação estável.

### RRET13 — Deduplicação

Resultados repetidos provenientes de diferentes consultas devem poder ser deduplicados sem destruir evidências distintas.

### RRET14 — Identidade documental

A deduplicação deve preferencialmente utilizar a identidade da unidade e não apenas seu conteúdo textual.

### RRET15 — Contexto complementar

O sistema deve permitir recuperar contexto adicional sem descaracterizar a unidade principal.

### RRET16 — Contexto estrutural

A estrutura documental deve permanecer associada aos resultados.

### RRET17 — Localização

Toda evidência recuperada deve manter relação com sua localização documental.

### RRET18 — Versionamento de embeddings

O modelo e a configuração utilizados na representação vetorial devem ser identificáveis.

### RRET19 — Dimensão vetorial

A dimensão da representação deve ser identificável e coerente com sua versão.

### RRET20 — Perfil de recuperação

Cada execução relevante deve poder ser associada às configurações necessárias para sua reconstrução.

### RRET21 — Corpus versionável

A recuperação deve poder ser associada a uma versão ou estado identificável do corpus.

### RRET22 — Pipeline versionável

Mudanças no processo de recuperação devem ser identificáveis.

### RRET23 — Cache identificável

Resultados obtidos por cache devem poder ser distinguidos de resultados recalculados durante avaliação.

### RRET24 — Recuperação avaliável isoladamente

A qualidade da recuperação deve poder ser avaliada sem depender da síntese final do LLM.

### RRET25 — Busca alternativa

A arquitetura deve permitir avaliar diferentes estratégias de recuperação quando necessário, incluindo estratégias semânticas, lexicais ou combinadas.

### RRET26 — Reranking desacoplado

Caso exista uma etapa de reranking, ela deve ser separável e versionável.

### RRET27 — Estabilidade

A arquitetura deve permitir medir a estabilidade dos resultados sob condições controladas.

### RRET28 — Reprodutibilidade

As condições relevantes para reproduzir uma recuperação devem ser registráveis.

### RRET29 — Proveniência operacional

Deve ser possível reconstruir a cadeia consulta → resultado → unidade → documento → origem.

### RRET30 — Separação metodológica

O motor de recuperação não deve atribuir automaticamente relevância metodológica às unidades encontradas.

---

# 5.3.43. Perfil conceitual de uma execução

Uma execução futura pode ser conceitualmente representada como:

```text
RECUPERAÇÃO
│
├── Consulta original
│
├── Corpus
│   ├── identificação
│   └── versão/estado
│
├── Consulta derivada
│   ├── estratégia
│   └── versão
│
├── Representação
│   ├── unidade de recuperação
│   ├── versão da segmentação
│   └── versão do embedding
│
├── Busca
│   ├── métrica
│   ├── threshold
│   ├── top-K
│   └── filtros
│
├── Ranking
│   └── versão da estratégia
│
├── Deduplicação
│   └── estratégia
│
├── Contexto
│   └── estratégia de expansão
│
└── Resultados
    ├── unidade
    ├── localização
    └── documento
```

Esse registro não determina como isso será armazenado.

Ele define apenas que essas relações devem existir de maneira reconstruível.

---

# 5.3.44. Arquitetura conceitual resultante

A arquitetura da recuperação pode ser representada como:

```text
                         PERGUNTA
                            │
                            ▼
                 CONSULTA ORIGINAL PRESERVADA
                            │
                            ▼
                  DECOMPOSIÇÃO / TRANSFORMAÇÃO
                            │
                            ▼
                    CONSULTAS DERIVADAS
                            │
                            ▼
                   REPRESENTAÇÃO DA CONSULTA
                            │
                            ▼
                 FILTRO DE CORPUS AUTORIZADO
                            │
                            ▼
                  GERAÇÃO DE CANDIDATOS
                            │
                            ▼
                      RANKING INICIAL
                            │
                            ▼
                       DEDUPLICAÇÃO
                            │
                            ▼
                       RERANKING*
                            │
                            ▼
                   SELEÇÃO DE RESULTADOS
                            │
                            ▼
                  CONTEXTO COMPLEMENTAR
                            │
                            ▼
                 EVIDÊNCIAS RECUPERADAS
                            │
                            ▼
                     RASTREABILIDADE
                            │
                            ▼
                       PESQUISADOR
```

* quando aplicável.

Em paralelo, cada resultado mantém:

```text
unidade
 ↓
bloco
 ↓
página
 ↓
documento
 ↓
arquivo original
```

---

# 5.3.45. Relação com a síntese

A arquitetura deve produzir uma fronteira clara:

```text
              MOTOR DE RECUPERAÇÃO
                       │
                       ▼
              EVIDÊNCIAS SELECIONADAS
                       │
                       ▼
                MOTOR DE SÍNTESE
                       │
                       ▼
                 RESPOSTA TEXTUAL
```

Essa separação é fundamental para avaliação.

Se uma resposta estiver errada, devemos conseguir perguntar:

> A evidência correta não foi recuperada?

ou:

> A evidência correta foi recuperada, mas a síntese falhou?

Sem essa separação, o sistema se torna difícil de avaliar metodologicamente.

---

# 5.3.46. Relação com a Etapa 5.2

A Etapa 5.2 definiu que:

> **o chunk não é o documento.**

A Etapa 5.3 acrescenta:

> **a unidade recuperada não é necessariamente a evidência metodológica.**

A relação completa passa a ser:

```text
DOCUMENTO
   ↓
REPRESENTAÇÃO CANÔNICA
   ↓
ESTRUTURA
   ↓
SEGMENTAÇÃO
   ↓
UNIDADE DE RECUPERAÇÃO
   ↓
RECUPERAÇÃO
   ↓
EVIDÊNCIA DOCUMENTAL
   ↓
AVALIAÇÃO DO PESQUISADOR
   ↓
RELEVÂNCIA METODOLÓGICA
```

Essa cadeia impede que o sistema pule diretamente de:

```text
embedding
```

para:

```text
conclusão metodológica.
```

---

# 5.3.47. Relação com a Etapa 5.4

A Etapa 5.4 deverá aprofundar a proveniência.

A 5.3 define o que precisa ser rastreável.

A 5.4 deverá definir como a arquitetura deve representar essa cadeia de proveniência.

A fronteira conceitual é:

### 5.3

> **Quais resultados foram recuperados e sob quais condições?**

### 5.4

> **Como demonstrar documentalmente de onde veio cada resultado?**

Assim, a 5.3 prepara a estrutura necessária para a rastreabilidade.

---

# 5.3.48. Relação com a Etapa 5.5

A recuperação depende de representações que mudam ao longo do tempo.

Podem mudar:

* documentos;
* versões;
* chunks;
* embeddings;
* modelos;
* configurações;
* índices;
* estratégias de ranking;
* decomposição.

Portanto, a Etapa 5.5 deverá tratar da persistência dessas versões e da consistência entre elas.

A 5.3 define:

> **o que precisa ser identificável.**

A 5.5 definirá:

> **como manter essas identidades e relações de forma consistente.**

---

# 5.3.49. Avaliação empírica da recuperação

A qualidade do motor não deve ser inferida apenas pela aparência das respostas.

Será necessário construir uma avaliação específica de recuperação.

Uma avaliação conceitual pode conter:

```text
corpus controlado
       ↓
consultas controladas
       ↓
resultados recuperados
       ↓
julgamento humano de relevância
       ↓
métricas
```

Entre as propriedades que podem ser avaliadas:

* precisão;
* recall/cobertura;
* posição da evidência relevante;
* estabilidade;
* redundância;
* adequação contextual;
* rastreabilidade.

---

# 5.3.50. Recall e o problema da evidência ausente

Uma recuperação pode parecer boa porque os resultados recuperados são relevantes.

Mas isso não responde:

> **quantas evidências relevantes ficaram de fora?**

Esse é o problema da cobertura.

Por exemplo:

```text
10 resultados recuperados
8 relevantes
```

parece excelente.

Mas se existiam:

```text
30 evidências relevantes
```

a recuperação encontrou apenas uma fração delas.

Portanto:

> **precisão sem cobertura pode esconder um problema grave de seleção.**

---

# 5.3.51. Precisão e o problema do ruído

O problema inverso também existe.

Se o sistema recuperar:

```text
100 unidades
```

e:

```text
10 são relevantes
```

pode haver boa cobertura, mas baixa precisão.

Isso gera:

* ruído;
* custo;
* dificuldade de leitura;
* maior carga para o LLM;
* maior carga para o pesquisador.

Portanto, a avaliação deve considerar simultaneamente:

```text
PRECISÃO
+
COBERTURA
+
CONTEXTO
+
ESTABILIDADE
```

---

# 5.3.52. Estabilidade como dimensão própria

Mesmo uma recuperação com boa precisão e recall pode apresentar resultados instáveis.

Por exemplo:

```text
Execução 1:
A, B, C, D, E

Execução 2:
A, B, D, C, E

Execução 3:
A, C, B, D, E
```

Dependendo da finalidade, isso pode ser irrelevante ou importante.

Por isso, a estabilidade deve ser medida separadamente.

O objetivo não é necessariamente exigir determinismo absoluto em todos os níveis.

O objetivo é conhecer:

> **quanto o resultado da recuperação varia sob condições consideradas equivalentes.**

---

# 5.3.53. Avaliação de diferentes segmentações

A Etapa 5.2 estabeleceu que a segmentação é uma variável arquitetural.

Portanto, a Etapa 5.3 deve permitir comparar:

```text
Segmentação A
      ↓
Recuperação A
      ↓
Resultado A
```

com:

```text
Segmentação B
      ↓
Recuperação B
      ↓
Resultado B
```

sem confundir a diferença com uma alteração do corpus.

Isso permite investigar empiricamente se determinada estratégia de segmentação realmente melhora a recuperação.

---

# 5.3.54. Avaliação de diferentes perfis de recuperação

O mesmo corpus pode ser submetido a diferentes configurações:

```text
Perfil A
threshold = X
top-K = Y
estratégia = A
```

versus:

```text
Perfil B
threshold = X'
top-K = Y'
estratégia = B
```

A comparação deve permitir determinar quais alterações realmente produzem melhora.

Isso evita o processo de:

> “mudamos a configuração e parece melhor”.

A arquitetura deve permitir:

> “sob o mesmo corpus e conjunto de consultas, essa configuração produziu determinada alteração mensurável.”

---

# 5.3.55. Testes adversariais de recuperação

A recuperação deve ser testada com consultas que explorem diferentes dificuldades.

Por exemplo:

* termos raros;
* nomes de autores;
* conceitos abstratos;
* conceitos com sinônimos;
* perguntas compostas;
* perguntas comparativas;
* perguntas sobre relações entre conceitos;
* consultas muito curtas;
* consultas muito longas;
* termos presentes em muitos documentos;
* termos presentes em poucos documentos;
* conceitos implicitamente descritos;
* perguntas cuja resposta depende de contexto.

O objetivo é verificar se o sistema funciona apenas em consultas fáceis ou se mantém utilidade em situações reais de pesquisa.

---

# 5.3.56. Limitações epistemológicas

A Etapa 5.3 deve preservar explicitamente os seguintes limites:

1. **Embedding não é evidência.**
2. **Similaridade não é relevância.**
3. **Recuperação não é análise.**
4. **Ranking não é julgamento metodológico.**
5. **Top-K não representa necessariamente todas as evidências relevantes.**
6. **Threshold não representa um limite universal de relevância.**
7. **Consulta derivada não é equivalente à intenção original do pesquisador.**
8. **Contexto complementar não é parte automática da unidade principal.**
9. **Uma boa resposta não prova que a recuperação foi boa.**
10. **Uma recuperação ruim não pode ser corrigida simplesmente por uma síntese mais convincente.**
11. **Reprodutibilidade da recuperação não implica reprodutibilidade literal da síntese.**
12. **A seleção final da evidência metodológica permanece sob responsabilidade do pesquisador.**

---

# 5.3.57. Princípio central da Etapa 5.3

A formulação central desta etapa pode ser condensada em:

> **O motor de recuperação deve transformar consultas em conjuntos de unidades documentais candidatas de maneira configurável, rastreável, avaliável e suficientemente estável, preservando a relação entre consulta, corpus, configuração, unidade recuperada e origem documental, sem confundir similaridade semântica com relevância metodológica.**

---

# 5.3.58. Arquitetura consolidada das Etapas 5.1–5.3

Com as três etapas já especificadas, a arquitetura conceitual começa a assumir uma forma mais completa:

```text
                    DOCUMENTO ORIGINAL
                           │
                           ▼
              REPRESENTAÇÃO CANÔNICA
                           │
                           ▼
                ESTRUTURA DOCUMENTAL
                           │
                           ▼
              SEGMENTAÇÃO ESTRUTURAL
                           │
                           ▼
               SEGMENTAÇÃO SEMÂNTICA
                           │
                           ▼
               UNIDADES DE RECUPERAÇÃO
                           │
                    ┌──────┴──────┐
                    │             │
                    ▼             ▼
                CONTEÚDO      METADADOS
                    │             │
                    └──────┬──────┘
                           │
                           ▼
                       EMBEDDING
                           │
                           ▼
                      INDEXAÇÃO
                           │
                           │
PERGUNTA ──→ DECOMPOSIÇÃO ─┘
   │
   ▼
CONSULTAS DERIVADAS
   │
   ▼
RECUPERAÇÃO
   │
   ├── filtros
   ├── candidatos
   ├── ranking
   ├── deduplicação
   ├── reranking*
   └── contexto
   │
   ▼
EVIDÊNCIAS RECUPERADAS
   │
   ▼
SÍNTESE
   │
   ▼
PESQUISADOR
   │
   ▼
INTERPRETAÇÃO METODOLÓGICA
```

* quando aplicável.

---

# 5.3.59. Mudança fundamental em relação ao modelo atual

O modelo atual pode ser resumido como:

```text
pergunta
 ↓
LLM decompõe
 ↓
embedding
 ↓
match_chunks
 ↓
deduplicação textual
 ↓
primeiros resultados
 ↓
LLM
```

A arquitetura especificada passa a ser conceitualmente:

```text
pergunta original
       ↓
consulta(s) derivada(s)
       ↓
perfil de recuperação
       ↓
corpus explicitamente definido
       ↓
unidades de recuperação identificadas
       ↓
geração de candidatos
       ↓
filtragem
       ↓
ranking
       ↓
deduplicação
       ↓
seleção contextual
       ↓
evidências rastreáveis
       ↓
síntese
```

A diferença fundamental não é simplesmente adicionar etapas.

É tornar explícitas as decisões que anteriormente estavam implícitas.

---

# 5.3.60. Do “buscar chunks” para “recuperar evidências”

Essa mudança de linguagem é importante.

O sistema atual pode ser descrito operacionalmente como:

> “buscar chunks semelhantes à pergunta”.

A arquitetura proposta é melhor descrita como:

> **“recuperar unidades documentais potencialmente relevantes para uma consulta, mantendo sua identidade, contexto e origem.”**

Isso muda também o que precisa ser medido.

Não basta perguntar:

> “o vetor encontrou algo parecido?”

É necessário perguntar:

> “a recuperação localizou as unidades documentais que permitiam responder adequadamente à consulta?”

E posteriormente:

> “essas unidades eram relevantes para a investigação do pesquisador?”

A primeira pergunta pertence ao sistema.

A segunda já começa a envolver julgamento humano.

---

# 5.3.61. Decisões que permanecem em aberto

Esta etapa **não escolhe**:

* modelo específico de embedding;
* dimensão definitiva;
* banco vetorial específico;
* algoritmo específico de índice;
* métrica definitiva;
* threshold definitivo;
* top-K definitivo;
* modelo específico para decomposição;
* temperatura definitiva;
* algoritmo específico de reranking;
* mecanismo definitivo de busca híbrida;
* estratégia definitiva de deduplicação;
* formato definitivo de armazenamento;
* protocolo final de avaliação.

Essas decisões pertencem à etapa seguinte de alternativas técnicas e implementação.

---

# 5.3.62. O que a Etapa 5.3 efetivamente especifica

A etapa não diz:

> “use tecnologia X”.

Ela diz que qualquer solução futura deverá ser capaz de:

```text
preservar a pergunta
       ↓
registrar sua transformação
       ↓
definir o corpus
       ↓
recuperar unidades identificáveis
       ↓
controlar os parâmetros de busca
       ↓
ordenar os resultados
       ↓
evitar duplicação indevida
       ↓
recuperar contexto quando necessário
       ↓
preservar localização
       ↓
permitir avaliação
       ↓
permitir reconstrução da execução
```

Essa é exatamente a diferença entre **especificação arquitetural** e **escolha tecnológica**.

---

# 5.3.63. Síntese da Etapa 5.3

O motor de recuperação constitui a camada que conecta a pergunta do pesquisador às evidências documentais disponíveis no corpus.

Sua função não é interpretar metodologicamente os documentos, mas **localizar unidades documentais potencialmente relevantes de maneira controlável, rastreável e avaliável**.

A recuperação deve partir das unidades definidas na Etapa 5.2 e preservar sua relação com a representação canônica definida na Etapa 5.1.

A consulta original deve permanecer preservada mesmo quando for decomposta em múltiplas consultas derivadas. O corpus, a configuração, a representação vetorial, os parâmetros de busca, o ranking e as demais transformações relevantes devem fazer parte do perfil da execução.

A recuperação também deve ser avaliada independentemente da síntese do LLM. Isso permite distinguir:

```text
falha de recuperação
```

de:

```text
falha de seleção
```

e de:

```text
falha de síntese.
```

O princípio fundamental permanece:

> **Grounding reduz o problema da invenção; não elimina o problema da seleção.**

Uma síntese perfeitamente ancorada em dez trechos ainda pode ser metodologicamente insuficiente se o décimo primeiro trecho relevante jamais tiver sido recuperado.

Por isso, o objetivo do motor não deve ser simplesmente produzir **algum contexto plausível**, mas construir uma recuperação que possa ser examinada, comparada e empiricamente validada.

---

# 5.3.64. Formulação consolidada

> **O Snoopy-RAG deverá possuir um mecanismo de recuperação capaz de transformar consultas originais e, quando necessário, consultas derivadas em conjuntos de unidades documentais potencialmente relevantes, respeitando o corpus definido, mantendo rastreabilidade das transformações e das configurações utilizadas e permitindo controle sobre filtragem, similaridade, ranking, seleção, deduplicação e contexto. A recuperação deverá ser avaliável independentemente da síntese por modelo de linguagem e não deverá confundir similaridade semântica, posição no ranking ou recuperação computacional com relevância metodológica.**

---

# 5.3.65. Princípio final

A Etapa 5.1 estabeleceu:

> **PRESERVAR ANTES DE DERIVAR.**

A Etapa 5.2 acrescentou:

> **ESTRUTURAR ANTES DE SEGMENTAR E SEGMENTAR ANTES DE INDEXAR.**

A Etapa 5.3 acrescenta:

> **RECUPERAR ANTES DE SINTETIZAR E AVALIAR A RECUPERAÇÃO ANTES DE CONFIAR NA SÍNTESE.**

A cadeia conceitual agora é:

> **PRESERVAR → ESTRUTURAR → SEGMENTAR → REPRESENTAR → RECUPERAR → RASTREAR → SINTETIZAR → INTERPRETAR**

E a distinção fundamental passa a ser:

> **O embedding localiza similaridade; o motor de recuperação seleciona candidatos; a evidência permanece documental; e a relevância metodológica continua sendo uma decisão do pesquisador.**

Esse é o ponto em que o Snoopy deixa definitivamente de poder ser pensado apenas como **“um RAG que responde perguntas sobre PDFs”**. Ele passa a ser especificado como uma infraestrutura de **recuperação documental auditável**, na qual a síntese é uma etapa posterior e subordinada à qualidade daquilo que foi efetivamente recuperado.
