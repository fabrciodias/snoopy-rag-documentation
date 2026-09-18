# Snoopy-RAG — Etapa 4: Requisitos Metodológicos e Arquiteturais

**Status:** Especificação de análise — não constitui implementação.

## 1. Ponto de partida

A análise das etapas anteriores permite caracterizar o Snoopy-RAG não como um instrumento de análise de conteúdo propriamente dito, mas como uma **infraestrutura computacional de recuperação, contextualização e rastreabilidade documental** destinada a apoiar processos de pesquisa.

O sistema não determina, por si só, categorias analíticas, unidades de registro, regras de codificação ou interpretações teóricas. Sua função principal é localizar material documental, organizá-lo em unidades recuperáveis e apresentá-lo ao pesquisador de maneira contextualizada e rastreável.

Essa distinção é fundamental:

> **Snoopy-RAG não realiza a análise metodológica do corpus; ele constrói uma infraestrutura para que o pesquisador possa localizar e explorar o corpus.**

Nesse sentido, a principal questão metodológica identificada não está inicialmente na geração textual pelo modelo de linguagem, mas em uma etapa anterior: **a transformação do documento original em uma representação utilizada pelo sistema.**

Se essa transformação eliminar ou distorcer informação relevante, o problema ocorre antes mesmo da recuperação semântica e pode afetar a própria composição do material que chega ao pesquisador.

---

# 2. Princípio arquitetural central

O sistema deverá adotar como princípio central a existência de uma **representação canônica do documento anterior às transformações destinadas especificamente à recuperação.**

A arquitetura conceitual deve ser:

```text
Documento original
        ↓
Representação canônica preservadora
        ↓
Representações derivadas
        ├── texto pesquisável
        ├── segmentação estrutural
        ├── chunks de recuperação
        ├── embeddings
        ├── visualização/leitura
        └── síntese contextual
```

A representação canônica não deve ser confundida com:

* Markdown;
* texto puro;
* chunk;
* embedding;
* resposta gerada pelo LLM.

Cada uma dessas representações possui uma finalidade específica e pode ser derivada da representação canônica.

O princípio é importante porque permite **reprocessar as etapas posteriores sem precisar necessariamente reextrair o documento original.**

---

# 3. Requisitos da representação documental

## R1 — Minimização de transformações destrutivas

A transformação do documento deve minimizar a perda de informação.

Não se deve buscar a inexistência absoluta de transformação, pois qualquer representação computacional constitui uma transformação do documento original.

O objetivo é:

> **preservar o máximo possível das características documentais relevantes para recuperação, contextualização e avaliação humana.**

---

## R2 — Preservação da estrutura documental

Sempre que tecnicamente possível, a representação deve preservar informações como:

* página de origem;
* ordem de leitura;
* blocos de texto;
* títulos e subtítulos;
* parágrafos;
* listas;
* tabelas;
* legendas;
* referências;
* notas;
* elementos gráficos relevantes;
* imagens;
* informações provenientes de OCR;
* relações espaciais relevantes entre elementos.

A estrutura deve ser tratada como informação potencialmente relevante, e não apenas como ruído de apresentação.

---

## R3 — Preservação da origem

Cada elemento derivado deve manter vínculo com sua origem no documento.

Idealmente, uma unidade recuperada deverá permitir determinar:

```text
unidade recuperada
      ↓
bloco/segmento
      ↓
página
      ↓
documento
      ↓
fonte original
```

Quando a localização espacial puder ser preservada, informações como coordenadas ou bounding boxes podem ser mantidas.

O objetivo não é apenas saber **qual documento** contém determinada informação, mas permitir retornar ao local em que ela aparece.

---

# 4. Representação canônica ≠ representação de recuperação

A representação utilizada para armazenamento documental não deve necessariamente ser a mesma utilizada para busca.

Essa separação é fundamental.

Uma representação pode ser excelente para:

* leitura humana;
* preservação estrutural;
* reconstrução visual;

mas não ser ideal para:

* busca lexical;
* embeddings;
* recuperação semântica.

Por isso, o sistema deve trabalhar conceitualmente com:

```text
Representação canônica
        ↓
        ├── representação textual
        ├── representação estrutural
        ├── representação para recuperação
        └── representação para interface
```

Isso evita que uma decisão tomada exclusivamente para melhorar o desempenho do mecanismo de busca acabe definindo aquilo que será considerado como o próprio conteúdo documental.

---

# 5. Reprocessamento sem reextração

A representação canônica deverá funcionar como ponto estável para o processamento posterior.

Assim, mudanças em:

* tamanho de chunk;
* estratégia de segmentação;
* modelo de embedding;
* parâmetros de recuperação;
* estrutura de metadados;
* estratégia de busca;

não deveriam exigir necessariamente uma nova extração do PDF.

Conceitualmente:

```text
PDF
 ↓
CANONICAL DOCUMENT
 ↓
 ├── chunker A
 ├── chunker B
 ├── embedding A
 ├── embedding B
 └── representação para leitura
```

Isso reduz acoplamento entre extração e recuperação.

---

# 6. Requisitos de segmentação e chunking

## R6 — Não chamar heurística de "semântica"

A segmentação atual baseada predominantemente em parágrafos e limite de caracteres não deve ser denominada **semantic chunking** sem uma fundamentação adicional.

Uma segmentação baseada em:

* parágrafos;
* tamanho máximo;
* títulos identificados por heurística;

é uma estratégia estrutural/heurística.

O termo "semântico" deve ser reservado para uma estratégia que efetivamente utilize critérios semânticos, estruturais ou linguísticos capazes de justificar essa classificação.

---

## R7 — Continuidade documental

A segmentação não pode eliminar conteúdo devido ao funcionamento interno do algoritmo.

O sistema deverá garantir que:

* nenhum trecho seja sobrescrito;
* mudanças de seção não descartem conteúdo anterior;
* o conteúdo permaneça associado à seção correta;
* a ordem original seja preservada;
* fragmentos não sejam perdidos silenciosamente.

Esse requisito é especialmente importante porque erros de segmentação podem produzir perda de informação sem necessariamente gerar uma falha técnica explícita.

---

## R8 — Contexto estrutural como metadado

Informações como:

```text
[SEÇÃO: Metodologia]
```

não devem necessariamente ser inseridas artificialmente no conteúdo textual do chunk.

Quando possível, o contexto estrutural deverá existir como **metadado estruturado**, por exemplo:

```text
section = "Metodologia"
page_start = 8
page_end = 10
```

Isso mantém separadas:

* informação original;
* informação estrutural inferida;
* informação adicionada pelo sistema.

---

# 7. Chunk ≠ unidade de registro

Uma distinção metodológica central é:

> **chunk de recuperação não é automaticamente unidade de registro.**

O chunk existe para atender a uma finalidade computacional: permitir recuperação eficiente e contextualização adequada.

A unidade de registro, por outro lado, depende da metodologia de pesquisa.

A arquitetura conceitual deve, portanto, separar:

```text
Documento
   ↓
Segmentação estrutural
   ↓
Segmentação semântica
   ↓
Unidade de recuperação
   ↓
Material apresentado ao pesquisador
   ↓
Unidade metodológica definida pelo pesquisador
```

O Snoopy pode fornecer suporte às etapas anteriores, mas não deve presumir qual fragmento constitui uma unidade metodológica válida.

---

# 8. Recuperação semântica

## R11 — Similaridade não é relevância metodológica

Uma alta similaridade vetorial entre uma consulta e um trecho não significa necessariamente que esse trecho seja metodologicamente relevante para a pesquisa.

A similaridade representa uma relação calculada pelo mecanismo de representação vetorial.

A relevância depende do problema de pesquisa e dos critérios do pesquisador.

Portanto:

```text
similaridade ≠ relevância
```

O sistema deve evitar apresentar o resultado da recuperação como se constituísse automaticamente uma seleção metodologicamente válida.

---

# 9. Rastreabilidade da recuperação

A recuperação deve ser conceitualmente rastreável.

Idealmente:

```text
Pergunta original
       ↓
Consultas derivadas
       ↓
Configuração de recuperação
       ↓
Modelo de embedding
       ↓
Unidades recuperadas
       ↓
Localização no documento
       ↓
Documento
       ↓
Fonte original
```

Essa cadeia permite compreender não apenas **o que foi recuperado**, mas **como aquilo chegou ao resultado.**

---

# 10. Avaliação da recuperação

A qualidade da recuperação deve ser avaliada separadamente da qualidade da síntese produzida pelo LLM.

São problemas diferentes.

### Recuperação

Pergunta:

> O sistema encontrou material relevante?

### Síntese

Pergunta:

> O modelo representou corretamente o material recuperado?

Uma síntese linguisticamente excelente pode ser produzida a partir de uma seleção ruim.

Consequentemente:

> **Uma boa resposta não demonstra, por si só, uma boa recuperação.**

---

# 11. Síntese por modelo de linguagem

A síntese produzida pelo LLM deve ser considerada uma **representação derivada do corpus**, e não evidência primária.

O modelo pode:

* condensar;
* reorganizar;
* relacionar;
* traduzir;
* contextualizar;
* formular uma resposta.

Entretanto, essas operações não transformam a resposta do modelo em documento científico original.

A evidência continua sendo o material documental recuperado.

---

# 12. Grounding não resolve seleção incompleta

O uso de evidências recuperadas para fundamentar a resposta reduz o problema de invenção de conteúdo.

Porém, isso não resolve necessariamente o problema da seleção.

A distinção pode ser expressa da seguinte maneira:

> **Grounding reduz o problema da invenção; não elimina o problema da seleção.**

Se um trecho relevante nunca for recuperado, o LLM poderá produzir uma resposta perfeitamente fundamentada **dentro do conjunto de evidências que recebeu**, mas ainda assim incompleta em relação ao corpus.

Esse problema é especialmente importante em pesquisa acadêmica.

---

# 13. Marcadores de evidência

Marcadores como:

```text
[TRECHO 1]
[TRECHO 2]
[TRECHO 3]
```

possuem utilidade operacional para estabelecer relação entre uma síntese e os fragmentos utilizados.

Entretanto, esses marcadores não devem ser tratados automaticamente como:

* unidade de registro;
* unidade de contexto;
* categoria analítica;
* evidência metodológica;
* citação bibliográfica formal.

Sua função primária é de **localização operacional da evidência recuperada.**

---

# 14. Rastreabilidade documental ≠ proveniência metodológica

É necessário distinguir dois conceitos.

### Rastreabilidade documental

Permite responder:

> "De onde veio este trecho?"

Por exemplo:

```text
Resposta
 → trecho
 → chunk
 → documento
 → página
 → fonte
```

### Proveniência metodológica

Permite responder questões adicionais:

* Por que esse documento entrou no corpus?
* Por que outro foi excluído?
* Qual era a versão do corpus?
* Qual critério definiu sua inclusão?
* Por que determinado trecho foi considerado relevante?
* Quais trechos foram descartados?
* Qual regra de codificação foi utilizada?
* Qual decisão metodológica levou à interpretação?

O Snoopy pode oferecer forte suporte à primeira dimensão sem necessariamente resolver a segunda.

Portanto:

> **Rastreabilidade documental não equivale a proveniência metodológica.**

---

# 15. Reprodutibilidade

A reprodutibilidade do sistema deve ser pensada em camadas.

## R19 — Reprodutibilidade do corpus

Registrar:

* documentos presentes;
* identificadores;
* versões;
* hashes;
* origem.

---

## R20 — Reprodutibilidade da configuração

Registrar, quando pertinente:

* versão da pipeline;
* representação documental;
* modelo de embedding;
* parâmetros de recuperação;
* threshold;
* top-K;
* estratégia de decomposição da consulta;
* versões relevantes dos componentes.

---

## R21 — Reprodutibilidade da recuperação

O objetivo é permitir verificar se determinada consulta, sob determinada configuração e corpus, produz resultados suficientemente estáveis ou explicar eventuais diferenças.

---

## R22 — Reprodutibilidade da análise

A análise metodológica permanece sob responsabilidade do pesquisador.

O Snoopy pode registrar e fornecer material para essa análise, mas não deve apresentar a recuperação automática como substituta da interpretação científica.

---

## R23 — Determinismo técnico ≠ reprodutibilidade científica

Uma pipeline determinística não implica automaticamente que a pesquisa seja cientificamente reprodutível.

É necessário distinguir:

```text
determinismo computacional
        ↓
reprodutibilidade técnica
        ↓
reprodutibilidade da recuperação
        ↓
transparência metodológica
        ↓
reprodutibilidade científica
```

São níveis relacionados, mas não equivalentes.

---

# 16. Dimensões de avaliação do Snoopy

A avaliação do sistema deverá separar pelo menos quatro dimensões.

## 16.1 Integridade documental

Pergunta:

> O sistema preservou adequadamente o conteúdo e a estrutura relevante dos documentos?

---

## 16.2 Qualidade da recuperação

Pergunta:

> O sistema recuperou os trechos relevantes?

Pode envolver avaliações como:

* precisão;
* recall;
* cobertura;
* estabilidade;
* relevância julgada por humanos.

---

## 16.3 Rastreabilidade

Pergunta:

> É possível retornar do resultado à localização correspondente no documento original?

---

## 16.4 Utilidade metodológica

Pergunta:

> O sistema efetivamente auxilia o pesquisador em seu processo de investigação?

Essa dimensão não pode ser reduzida à precisão matemática do mecanismo de busca.

---

# 17. Avaliação adversarial dos documentos

A nova arquitetura deverá ser testada não apenas com PDFs simples.

É necessário utilizar documentos capazes de expor fragilidades da transformação.

Exemplos:

* documentos em duas ou mais colunas;
* PDFs digitalizados;
* documentos com OCR;
* tabelas;
* notas de rodapé;
* imagens;
* legendas;
* referências bibliográficas complexas;
* cabeçalhos e rodapés;
* diferentes tamanhos de fonte;
* caracteres especiais;
* documentos longos;
* estruturas acadêmicas complexas;
* elementos posicionados espacialmente;
* páginas com combinação de texto, tabela e imagem.

O objetivo não é apenas verificar se o sistema "consegue ler o PDF".

O objetivo é verificar:

> **o que foi perdido, alterado, deslocado ou reinterpretado durante a transformação.**

---

# 18. Princípio para decisões de transformação

Toda transformação aplicada ao documento deve ser avaliada por cinco perguntas:

### 1. Qual é a finalidade da transformação?

Exemplo:

> melhorar recuperação semântica.

### 2. Que informação ela pode perder?

Exemplo:

> relações espaciais, tabelas, ordem de leitura, imagens.

### 3. Essa perda é aceitável para a finalidade?

Se não for, a transformação não deve ser utilizada dessa maneira.

### 4. A informação perdida pode ser recuperada por outra representação?

Se sim, essa representação deve permanecer disponível e vinculada à origem.

### 5. A origem continua rastreável?

Se não, a transformação compromete a confiabilidade documental do sistema.

---

# 19. Limites epistemológicos

A arquitetura deve assumir explicitamente alguns limites.

### Recuperação não é análise.

O fato de um trecho ter sido recuperado não significa que ele tenha sido analisado.

### Similaridade não é relevância.

O cálculo vetorial fornece um critério computacional, não um julgamento metodológico.

### Chunk não é unidade de registro.

O tamanho e a organização do fragmento são decisões computacionais.

### Evidência recuperada não é automaticamente evidência metodológica.

Sua importância depende da questão e do desenho da pesquisa.

### Rastreabilidade não é proveniência completa.

Saber de onde veio o trecho não explica todas as decisões que determinaram sua utilização na pesquisa.

### Síntese do LLM não é fonte primária.

Ela é uma representação derivada do material recuperado.

---

# 20. Arquitetura conceitual-alvo

A arquitetura metodologicamente mais coerente para o Snoopy passa a ser representada por:

```text
                    ┌────────────────────┐
                    │  DOCUMENTO ORIGINAL│
                    │       (PDF)        │
                    └─────────┬──────────┘
                              │
                              ▼
                 ┌─────────────────────────┐
                 │ REPRESENTAÇÃO CANÔNICA   │
                 │ PRESERVADORA             │
                 └────────────┬────────────┘
                              │
             ┌────────────────┼─────────────────┐
             │                │                 │
             ▼                ▼                 ▼
       Estrutura          Texto              Elementos
       documental        pesquisável        multimodais
             │                │                 │
             └────────────────┼─────────────────┘
                              ▼
                 ┌─────────────────────────┐
                 │ SEGMENTAÇÃO ESTRUTURAL  │
                 └────────────┬────────────┘
                              ▼
                 ┌─────────────────────────┐
                 │ SEGMENTAÇÃO SEMÂNTICA   │
                 └────────────┬────────────┘
                              ▼
                 ┌─────────────────────────┐
                 │ UNIDADES DE RECUPERAÇÃO │
                 └────────────┬────────────┘
                              ▼
                 ┌─────────────────────────┐
                 │ EMBEDDINGS / RETRIEVAL  │
                 └────────────┬────────────┘
                              ▼
                 ┌─────────────────────────┐
                 │ EVIDÊNCIAS RECUPERADAS  │
                 └────────────┬────────────┘
                              ▼
                 ┌─────────────────────────┐
                 │ SÍNTESE / INTERFACE      │
                 └─────────────────────────┘

             ↘ todo elemento deve manter ↙
                rastreabilidade à origem
```

A unidade metodológica da pesquisa permanece **fora dessa cadeia automática**, sendo definida pelo pesquisador conforme o desenho metodológico adotado.

---

# 21. Consequência arquitetural principal

A mudança mais importante decorrente desta análise é abandonar a ideia de:

```text
PDF
 ↓
Markdown
 ↓
chunks
 ↓
embeddings
```

como se o Markdown fosse simplesmente uma representação intermediária descartável.

A arquitetura passa a ser:

```text
PDF
 ↓
REPRESENTAÇÃO CANÔNICA
 ↓
REPRESENTAÇÕES DERIVADAS
 ↓
RECUPERAÇÃO
 ↓
SÍNTESE
```

A diferença é conceitualmente grande.

No primeiro modelo, a representação intermediária pode acabar se tornando, na prática, **aquilo que o Snoopy considera ser o documento**.

No segundo, o sistema reconhece explicitamente que existe uma diferença entre:

> **o documento e aquilo que o sistema fez com o documento.**

---

# 22. Requisito geral consolidado

Pode-se condensar os requisitos desta etapa na seguinte formulação:

> **O Snoopy-RAG deverá minimizar transformações destrutivas dos documentos, preservar características documentais relevantes para recuperação e avaliação humana, manter rastreabilidade entre representações derivadas e suas origens e separar explicitamente representação documental, segmentação computacional, recuperação semântica, síntese por modelo de linguagem e interpretação metodológica.**

---

# 23. Princípios axiomáticos do sistema

A análise permite estabelecer cinco princípios que deverão orientar as decisões arquiteturais posteriores:

### Axioma 1

**Recuperação ≠ análise.**

### Axioma 2

**Similaridade semântica ≠ relevância metodológica.**

### Axioma 3

**Trecho recuperado ≠ unidade de registro/contexto.**

### Axioma 4

**Rastreabilidade da fonte ≠ proveniência metodológica completa.**

### Axioma 5

**Síntese do LLM = representação derivada, não evidência primária.**

Esses princípios devem funcionar como restrições conceituais para evitar que futuras funcionalidades do sistema confundam conveniência computacional com decisão metodológica.

---

# 24. Questões deixadas para a Etapa 5

A Etapa 4 estabelece **o que a arquitetura precisa preservar e respeitar**. Ela não determina ainda a implementação concreta.

A próxima etapa deverá responder:

1. **Qual representação canônica é mais adequada?**
2. Como preservar texto, estrutura, tabelas, imagens e localização espacial?
3. Como tratar PDFs nativamente digitais e PDFs escaneados?
4. Quando utilizar OCR?
5. Como incorporar informação multimodal?
6. Onde armazenar a representação canônica?
7. Como controlar tamanho e custo de armazenamento?
8. Como versionar a representação?
9. Como identificar alterações no documento original?
10. Como derivar o texto de recuperação sem destruir a representação canônica?
11. Como redesenhar a segmentação estrutural?
12. O que deverá ser considerado uma unidade de recuperação?
13. Como implementar uma segmentação realmente semântica?
14. Como preservar a rastreabilidade página → bloco → unidade?
15. Como avaliar empiricamente a nova pipeline?
16. Como comparar a nova arquitetura com a atual?

---

# 25. Encerramento da Etapa 4

A principal conclusão desta etapa é que o problema central não deve ser tratado como um simples defeito do `extractor.py`.

O problema é **arquitetural**.

A pipeline atual foi construída historicamente com uma lógica adequada ao objetivo inicial de transformar PDFs em material textual pesquisável. Com a evolução do Snoopy para uma infraestrutura de exploração documental e apoio à pesquisa, essa premissa tornou-se insuficiente.

O próximo passo, portanto, não deve ser simplesmente:

> "melhorar o extractor".

Deve ser:

> **redesenhar a relação entre documento original, representação documental, segmentação, recuperação e evidência.**

O objetivo não é fazer o PDF "virar Markdown melhor".

O objetivo é fazer com que o Snoopy **transforme o documento o mínimo necessário, preserve aquilo que importa e só depois produza as representações específicas necessárias para busca, leitura e síntese.**

**É aqui que começa, de fato, o projeto do novo motor documental do Snoopy.**

