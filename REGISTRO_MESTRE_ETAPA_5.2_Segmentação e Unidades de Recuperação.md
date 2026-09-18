# Registro Mestre da Análise — Snoopy-RAG

## Etapa 5.2 — Segmentação e Unidades de Recuperação

**Status:** Especificação metodológica e arquitetural — não constitui implementação.
**Relação com as etapas anteriores:** continuidade direta das Etapas 4 e 5.1.
**Função deste registro:** estabelecer os princípios e requisitos que uma futura arquitetura de segmentação deverá satisfazer, sem ainda escolher bibliotecas, modelos, algoritmos ou formatos específicos.

---

# 5.2 — Segmentação e Unidades de Recuperação

## 5.2.1. Objetivo

A Etapa 5.2 trata da forma como a representação canônica do documento será transformada em unidades menores destinadas à recuperação.

O problema não é simplesmente decidir **quantos caracteres devem existir em cada chunk**. A questão arquitetural é mais profunda:

> **Como transformar um documento estruturalmente complexo em unidades recuperáveis sem destruir o contexto necessário para interpretar aquilo que foi recuperado?**

A segmentação precisa, portanto, ser compreendida como uma transformação derivada da representação canônica, e não como a própria representação do documento.

A sequência definida na Etapa 5.1 permanece como princípio:

> **Documento original → representação canônica preservadora → representações derivadas → segmentação → recuperação**

A segmentação ocorre **depois da preservação**, e não em substituição a ela.

---

# 5.2.2. Contexto herdado da Etapa 5.1

A Etapa 5.1 estabeleceu que o documento original não deve ser reduzido diretamente a uma representação textual destinada à busca.

O princípio central foi:

> **preservar antes de derivar.**

Isso produz uma consequência imediata para o chunking.

No sistema atual, a cadeia pode ser simplificada como:

```text
PDF
 ↓
texto extraído
 ↓
limpeza
 ↓
chunking
 ↓
embeddings
```

Essa arquitetura faz com que a segmentação dependa diretamente daquilo que o extrator conseguiu representar como texto.

Na arquitetura pretendida, a relação deve ser diferente:

```text
PDF
 ↓
representação canônica
 ↓
estrutura documental
 ↓
segmentação estrutural
 ↓
segmentação semântica
 ↓
unidades de recuperação
 ↓
embeddings / recuperação
```

Assim, a unidade recuperável não deve ser entendida como uma simples fatia arbitrária do texto.

---

# 5.2.3. O problema metodológico da segmentação

A segmentação é necessária porque documentos inteiros são unidades excessivamente grandes para muitas operações de recuperação.

Entretanto, dividir um documento produz uma tensão fundamental:

* unidades maiores preservam mais contexto;
* unidades menores permitem maior precisão de recuperação.

Essa tensão não pode ser resolvida simplesmente estabelecendo um tamanho universal.

Uma unidade muito grande pode conter grande quantidade de informação irrelevante junto da informação procurada.

Uma unidade muito pequena pode conter apenas uma frase, tabela ou fragmento cujo significado depende do parágrafo anterior, do título da seção, de uma nota ou de outra parte da página.

Portanto:

> **A qualidade da segmentação não pode ser medida apenas pelo tamanho das unidades produzidas.**

Ela deve ser avaliada também pela capacidade de preservar **unidades de sentido e contexto suficientes para que uma recuperação posterior seja interpretável**.

---

# 5.2.4. O chunk atual e seus limites

A implementação analisada atualmente utiliza uma estratégia heurística baseada na estrutura textual disponível, incluindo parágrafos, títulos identificados e um limite aproximado de tamanho.

Essa estratégia foi adequada ao objetivo inicial de transformar documentos em texto pesquisável.

Entretanto, à luz das Etapas 4 e 5.1, ela apresenta limitações conceituais.

O principal problema não é simplesmente o valor máximo utilizado para o tamanho do chunk.

O problema é que a segmentação atual ocorre sobre uma representação textual já derivada e potencialmente empobrecida do documento.

Isso significa que, se a extração perdeu:

* relações espaciais;
* tabelas;
* imagens;
* legendas;
* hierarquia documental;
* notas;
* referências;
* continuidade entre elementos;

o chunker não possui necessariamente informação suficiente para reconstruir essas relações.

Portanto:

> **Um chunker melhor não consegue recuperar informações que já foram destruídas na etapa anterior.**

Essa é uma das razões pelas quais a Etapa 5.2 depende diretamente da Etapa 5.1.

---

# 5.2.5. Chunk não é unidade metodológica

Uma distinção fundamental deve permanecer explícita em toda a arquitetura.

### Chunk

É uma **unidade computacional de segmentação e recuperação**.

Sua finalidade é tornar o documento manipulável pelo sistema, permitindo operações como:

* indexação;
* geração de embeddings;
* busca;
* recuperação;
* ordenação;
* apresentação de contexto.

### Unidade de registro

É uma unidade definida pelo pesquisador de acordo com os objetivos metodológicos de sua investigação.

Ela pode corresponder, dependendo da pesquisa, a:

* uma frase;
* uma proposição;
* um parágrafo;
* uma fala;
* uma ocorrência;
* uma categoria;
* uma unidade temática;
* ou outra definição metodológica.

### Unidade de contexto

É o conjunto de informação necessário para compreender adequadamente a unidade de registro.

Essas três coisas não devem ser confundidas.

A relação correta é:

```text
DOCUMENTO
   ↓
REPRESENTAÇÃO
   ↓
SEGMENTAÇÃO COMPUTACIONAL
   ↓
UNIDADE DE RECUPERAÇÃO
   ↓
PESQUISADOR
   ↓
UNIDADE DE REGISTRO / UNIDADE DE CONTEXTO
```

O sistema pode **auxiliar o pesquisador a localizar possíveis unidades**, mas não deve afirmar que um chunk recuperado constitui automaticamente uma unidade metodológica.

---

# 5.2.6. A segmentação como processo em camadas

A arquitetura proposta deve separar pelo menos três operações conceitualmente distintas:

```text
Representação canônica
        ↓
Segmentação estrutural
        ↓
Segmentação semântica
        ↓
Unidades de recuperação
```

Essa separação é importante porque cada etapa responde a uma pergunta diferente.

### Segmentação estrutural

Pergunta:

> **Como o documento está organizado?**

Exemplos:

* página;
* seção;
* subseção;
* título;
* parágrafo;
* lista;
* tabela;
* figura;
* legenda;
* nota;
* referência;
* bloco textual.

A segmentação estrutural não precisa decidir ainda qual é a unidade ideal para busca semântica.

Ela deve primeiro representar a organização documental.

### Segmentação semântica

Pergunta:

> **Quais partes desse conteúdo formam unidades de sentido adequadas ao objetivo de recuperação?**

Aqui podem ser consideradas relações como:

* continuidade temática;
* dependência entre parágrafos;
* relação entre título e conteúdo;
* explicação e exemplo;
* pergunta e resposta;
* afirmação e justificativa;
* tabela e texto explicativo;
* figura e legenda;
* continuidade entre páginas.

### Unidade de recuperação

Pergunta:

> **Qual unidade será efetivamente indexada e recuperada pelo sistema?**

Essa unidade pode ser derivada das duas camadas anteriores.

Ela deve possuir conteúdo suficiente para ser útil à recuperação, mas permanecer suficientemente específica para evitar que a busca retorne grandes regiões pouco relevantes do documento.

---

# 5.2.7. A segmentação estrutural deve preceder a semântica

Uma consequência importante dessa arquitetura é que o sistema não deve começar perguntando:

> “Onde cortar o texto?”

Deve começar perguntando:

> “Que elementos constituem este documento e como eles se relacionam?”

Somente depois disso deve decidir como combinar ou dividir esses elementos para fins de recuperação.

Exemplo conceitual:

```text
Página 12
 ├── Título: Avaliação formativa
 ├── Parágrafo 1
 ├── Parágrafo 2
 ├── Tabela 1
 └── Legenda da Tabela 1

Página 13
 ├── Parágrafo 3 — continuação
 ├── Parágrafo 4
 └── Nota de rodapé
```

Uma segmentação baseada exclusivamente em caracteres poderia produzir:

```text
chunk 1 = fim do parágrafo 2 + início da tabela
chunk 2 = restante da tabela + legenda + início do parágrafo 3
```

Computacionalmente isso pode ser perfeitamente válido.

Documentalmente, pode ser péssimo.

A arquitetura proposta deve permitir que o sistema saiba que:

* a tabela é uma tabela;
* a legenda pertence à tabela;
* o parágrafo seguinte continua uma estrutura anterior;
* determinados elementos pertencem à mesma região documental;
* cada elemento possui uma localização de origem.

---

# 5.2.8. O problema da continuidade

A segmentação deve preservar relações de continuidade.

Essa continuidade pode ocorrer em diferentes níveis.

### Continuidade textual

Um parágrafo pode continuar na página seguinte.

### Continuidade estrutural

Uma subseção pode atravessar várias páginas.

### Continuidade semântica

Uma ideia pode ser desenvolvida por vários parágrafos.

### Continuidade multimodal

Um texto pode explicar uma figura ou tabela apresentada separadamente.

### Continuidade documental

Uma nota, referência ou legenda pode depender de outro elemento.

Portanto, o fato de dois elementos estarem separados fisicamente não significa necessariamente que sejam independentes.

Da mesma forma, o fato de dois elementos estarem próximos não significa necessariamente que pertençam à mesma unidade de sentido.

A segmentação deve preservar essas relações como informação disponível para decisões posteriores.

---

# 5.2.9. Títulos e hierarquia documental

Títulos não devem ser tratados automaticamente como conteúdo independente.

Em muitos documentos, o significado de um parágrafo depende da seção em que ele está inserido.

Por exemplo:

```text
3. Avaliação formativa

A avaliação pode cumprir diferentes funções...
```

O segundo elemento pode ser semanticamente incompleto quando separado de seu título.

Uma unidade de recuperação poderia, portanto, possuir:

```text
conteúdo:
A avaliação pode cumprir diferentes funções...

contexto estrutural:
Seção 3 — Avaliação formativa
```

Essa distinção é importante.

O contexto estrutural não precisa ser inserido artificialmente no texto original como se fizesse parte da citação.

Ele pode existir como **metadado da unidade de recuperação**.

---

# 5.2.10. Contexto estrutural como metadado

A arquitetura deve distinguir:

### Conteúdo recuperável

Aquilo que efetivamente pertence ao conteúdo do documento.

### Contexto estrutural

Informações que ajudam a localizar e interpretar o conteúdo.

Por exemplo:

```text
documento: artigo_X
página: 12
seção: 3
subseção: 3.2
bloco: 17
tipo: parágrafo
```

Isso é diferente de transformar tudo em:

```text
[Seção 3 — Avaliação formativa]
[Subseção 3.2 — Regulação]
A avaliação...
```

e posteriormente tratar essa concatenação como se fosse o texto original.

A segunda abordagem pode ser útil como representação derivada para determinadas operações, mas não deve substituir o conteúdo canônico nem adulterar a evidência apresentada ao pesquisador.

---

# 5.2.11. Tamanho da unidade de recuperação

Não deve ser estabelecido, nesta etapa, um tamanho universal de chunk.

O tamanho deve ser entendido como uma variável dependente de:

* estrutura documental;
* densidade informacional;
* finalidade da recuperação;
* tipo de conteúdo;
* dependência contextual;
* comportamento observado nas avaliações;
* características do corpus.

Isso significa que “chunk de X caracteres” não deve ser tratado como princípio arquitetural.

Um limite de tamanho pode existir como **restrição operacional**, mas não como definição semântica da unidade.

Em termos conceituais:

> **Tamanho máximo é uma restrição da unidade; não é o que define seu significado.**

---

# 5.2.12. Chunking semântico: cautela terminológica

O termo **“chunking semântico”** não deve ser usado simplesmente para descrever qualquer algoritmo que produza chunks.

Uma segmentação só merece ser caracterizada como semanticamente orientada quando sua decisão de agrupamento ou separação considera alguma propriedade relacionada ao significado ou à estrutura de sentido do conteúdo.

Uma divisão baseada exclusivamente em:

```text
N caracteres
```

ou:

```text
N tokens
```

é uma segmentação por tamanho.

Uma divisão baseada exclusivamente em:

```text
quebra de parágrafo
```

é uma segmentação estrutural simples.

Essas estratégias podem ser úteis e podem fazer parte de uma arquitetura mais sofisticada, mas não devem ser confundidas conceitualmente com uma segmentação semântica.

Essa distinção é importante para a documentação e para a avaliação experimental.

---

# 5.2.13. Unidades grandes e pequenas: trade-off

A arquitetura deve reconhecer explicitamente o conflito entre **precisão de recuperação** e **preservação de contexto**.

### Unidades excessivamente pequenas

Podem:

* aumentar precisão superficial;
* reduzir ruído local;
* facilitar localização de frases específicas;

mas também podem:

* perder contexto;
* separar afirmações de suas justificativas;
* separar títulos de seus conteúdos;
* fragmentar tabelas;
* dificultar interpretação;
* produzir embeddings pouco representativos.

### Unidades excessivamente grandes

Podem:

* preservar contexto;
* representar melhor uma discussão completa;
* reduzir fragmentação;

mas também podem:

* aumentar ruído;
* reduzir especificidade;
* recuperar conteúdo irrelevante;
* dificultar seleção posterior;
* aumentar custo computacional.

Portanto, não existe uma regra puramente teórica que determine o tamanho ideal.

A arquitetura precisa permitir **avaliação empírica**.

---

# 5.2.14. Segmentação orientada à finalidade

A unidade de recuperação deve ser definida em função daquilo que o sistema pretende recuperar.

Uma unidade destinada a localizar:

> “definições de avaliação formativa”

pode exigir uma granularidade diferente daquela necessária para localizar:

> “a argumentação desenvolvida pelo autor sobre a relação entre avaliação e regulação”.

Consequentemente, uma única segmentação pode não ser ideal para todas as tarefas.

Isso abre uma questão arquitetural importante:

> **O Snoopy deve possuir uma única segmentação permanente ou permitir diferentes representações derivadas para diferentes finalidades de recuperação?**

Nesta etapa, a decisão não precisa ser tecnológica.

O requisito é apenas reconhecer que:

> **a segmentação é uma representação derivada orientada à finalidade e não uma propriedade absoluta do documento.**

---

# 5.2.15. Tabelas

Tabelas exigem tratamento específico.

Uma tabela não deve ser reduzida automaticamente à sequência linear de palavras que aparece durante uma extração textual.

Sua estrutura pode carregar significado:

```text
             Grupo A    Grupo B
Método 1        12         18
Método 2         9         21
```

A relação entre:

* linhas;
* colunas;
* cabeçalhos;
* células;
* unidades;
* notas;

pode ser fundamental para interpretação.

A arquitetura de segmentação deve, portanto, permitir que uma tabela seja representada como uma unidade estrutural própria ou como conjunto estruturado de elementos relacionados.

A forma exata dessa representação permanece uma decisão da etapa técnica.

O requisito metodológico é:

> **a segmentação não deve destruir a estrutura necessária para interpretar uma tabela.**

---

# 5.2.16. Figuras, gráficos e imagens

O mesmo princípio se aplica a elementos visuais.

Uma figura pode possuir:

```text
Figura
+
legenda
+
texto que a referencia
```

Esses elementos não são necessariamente equivalentes, mas possuem relações documentais.

O sistema deve ser capaz de preservar essa relação antes de decidir como esses elementos serão utilizados na recuperação.

Uma imagem não deve desaparecer simplesmente porque não produz texto durante a extração.

Quando o elemento visual for relevante para o conteúdo, a arquitetura deve permitir que ele permaneça representado na camada canônica e seja associado aos seus derivados.

---

# 5.2.17. Notas de rodapé e referências

Notas de rodapé apresentam outro problema de segmentação.

Uma nota pode:

* explicar uma afirmação;
* fornecer uma definição;
* indicar uma fonte;
* acrescentar uma ressalva;
* conter informação metodologicamente relevante.

Portanto, ela não deve ser automaticamente incorporada ao parágrafo principal como se fosse texto contínuo.

Também não deve ser simplesmente descartada.

A arquitetura deve manter a relação:

```text
texto principal
      ↕
nota de rodapé
```

permitindo decidir posteriormente se a nota:

* participa da recuperação;
* aparece como contexto;
* é recuperada separadamente;
* ou é apenas preservada para consulta.

---

# 5.2.18. Cabeçalhos, rodapés e elementos repetitivos

Elementos repetidos em várias páginas também exigem tratamento específico.

Exemplos:

* título abreviado do artigo;
* nome da revista;
* número da página;
* cabeçalho institucional;
* rodapé editorial.

Esses elementos podem aparecer repetidamente no texto extraído e contaminar a segmentação.

Entretanto, removê-los definitivamente durante a primeira transformação também pode eliminar informação útil em determinados casos.

A arquitetura deve preferir:

```text
preservar → classificar → decidir uso
```

em vez de:

```text
detectar → apagar → perder definitivamente
```

Isso segue diretamente o princípio de não destruição estabelecido na Etapa 5.1.

---

# 5.2.19. Caracteres especiais e fórmulas

A segmentação também deve preservar conteúdos cuja representação textual seja sensível.

Isso inclui:

* símbolos científicos;
* fórmulas;
* caracteres matemáticos;
* índices;
* expoentes;
* acentuação;
* caracteres Unicode;
* notações específicas.

A unidade recuperável não deve ser considerada adequada apenas porque “contém texto”.

É necessário verificar se o texto continua representando corretamente o conteúdo documental original.

---

# 5.2.20. Localização exata da unidade

Toda unidade de recuperação deve possuir uma referência rastreável à sua origem.

A cadeia mínima conceitual deve permitir algo como:

```text
unidade de recuperação
      ↓
bloco estrutural
      ↓
página
      ↓
documento
      ↓
arquivo original
```

Quando disponível, a localização espacial pode ser mais precisa:

```text
página
+
bounding box
```

ou equivalente.

Isso permite que o sistema não apenas diga:

> “Esse trecho foi encontrado no artigo X.”

mas também:

> “Esse trecho corresponde a este elemento específico do documento.”

Essa diferença é fundamental para a leitura e para a auditoria.

---

# 5.2.21. Identidade da unidade

A unidade de recuperação precisa possuir uma identidade estável dentro da versão da representação que a originou.

Conceitualmente:

```text
documento
   ↓
versão documental
   ↓
representação canônica
   ↓
elemento estrutural
   ↓
unidade de recuperação
```

Isso será importante posteriormente para:

* reindexação;
* atualização;
* comparação de versões;
* avaliação;
* cache;
* rastreabilidade;
* reprodução de consultas.

Uma unidade não deve ser identificada apenas por sua posição eventual em uma lista.

---

# 5.2.22. Relação entre chunk e documento

A unidade de recuperação deve continuar sendo entendida como **parte do documento**, e não como um documento independente.

Isso parece trivial, mas possui consequências práticas.

A unidade deve manter informações como:

* documento de origem;
* versão;
* página;
* posição;
* estrutura;
* tipo de elemento;
* relação com elementos vizinhos;
* representação derivada da qual foi produzida.

Assim, a recuperação de uma unidade não rompe sua ligação com o documento maior.

---

# 5.2.23. Relação entre unidades vizinhas

Uma unidade recuperada isoladamente pode não ser suficiente para interpretação.

Por isso, a arquitetura deve permitir recuperar ou reconstruir contexto adjacente.

Exemplo:

```text
unidade 41
unidade 42  ← recuperada
unidade 43
```

O sistema pode determinar que a unidade 42 possui dependência contextual relevante das unidades 41 e 43.

Isso é diferente de simplesmente aumentar indefinidamente o tamanho do chunk.

A solução arquitetural pode ser:

```text
unidade principal
+
contexto estrutural
+
contexto adjacente recuperável
```

mantendo a unidade original identificável.

Esse princípio preserva simultaneamente:

* precisão;
* contexto;
* rastreabilidade.

---

# 5.2.24. Chunk primário e contexto recuperável

Uma consequência importante é separar:

### Unidade primária de recuperação

A unidade cuja similaridade justificou sua recuperação.

### Contexto complementar

Elementos adicionados para permitir interpretação adequada.

Por exemplo:

```text
[TRECHO PRINCIPAL]
parágrafo 42

[CONTEXTO]
título da seção
parágrafo anterior
parágrafo seguinte
```

Isso evita que todo o contexto seja incorporado à unidade primária e posteriormente apresentado como se fosse parte dela.

A distinção também melhora a rastreabilidade.

---

# 5.2.25. Sobreposição (overlap)

A utilização de sobreposição entre chunks pode ser útil para preservar continuidade textual.

Entretanto, overlap não deve ser tratado como solução universal.

Uma sobreposição pode:

* preservar contexto;
* reduzir cortes artificiais;
* aumentar chance de recuperar uma ideia completa;

mas também pode:

* duplicar conteúdo;
* aumentar armazenamento;
* produzir resultados repetidos;
* dificultar deduplicação;
* distorcer métricas de recuperação.

Portanto:

> **overlap é uma estratégia possível de segmentação, não um requisito absoluto.**

Sua necessidade deve ser avaliada empiricamente de acordo com a arquitetura escolhida.

---

# 5.2.26. Deduplicação não deve compensar segmentação ruim

Existe uma relação importante entre segmentação e recuperação.

Se várias unidades contêm grande quantidade de conteúdo repetido devido ao overlap, o mecanismo de busca pode recuperar:

```text
chunk 12
chunk 13
chunk 14
```

quando, na prática, os três representam quase a mesma evidência.

A solução não deve ser simplesmente aumentar a agressividade da deduplicação.

É necessário distinguir:

> **duplicação causada pela segmentação**

de:

> **evidências realmente distintas que apresentam conteúdo semanticamente semelhante.**

Essa distinção será importante na Etapa 5.3, dedicada à recuperação.

---

# 5.2.27. Segmentação e embeddings

Embeddings devem ser tratados como uma representação derivada da unidade de recuperação.

Portanto:

```text
representação canônica
       ↓
segmentação
       ↓
unidade de recuperação
       ↓
embedding
```

e não:

```text
PDF
 ↓
embedding
 ↓
tentativa de reconstrução
```

O embedding não deve possuir responsabilidade de representar aquilo que a arquitetura de segmentação não preservou.

Se a unidade estiver mal formada, um embedding melhor não resolve necessariamente o problema.

Isso reforça a sequência:

> **preservar → estruturar → segmentar → representar semanticamente → recuperar.**

---

# 5.2.28. Segmentação e recuperação não são a mesma coisa

Outra distinção importante:

### Segmentação

Decide:

> “Quais unidades existem?”

### Recuperação

Decide:

> “Quais dessas unidades são relevantes para esta consulta?”

Uma boa segmentação não garante uma boa recuperação.

Da mesma forma, um mecanismo de recuperação sofisticado não compensa necessariamente uma segmentação documental inadequada.

As duas etapas precisam ser avaliadas separadamente.

---

# 5.2.29. Segmentação e relevância metodológica

A existência de uma unidade bem formada também não significa que ela seja metodologicamente relevante.

A sequência permanece:

```text
documento
 ↓
segmentação
 ↓
unidade recuperável
 ↓
similaridade com consulta
 ↓
recuperação
 ↓
avaliação humana
 ↓
relevância metodológica
```

A decisão metodológica continua sendo do pesquisador.

O Snoopy pode dizer:

> “esta unidade apresenta alta correspondência com a consulta.”

Não deve transformar isso automaticamente em:

> “esta unidade é uma evidência metodologicamente relevante.”

Essa distinção permanece um dos axiomas centrais do projeto.

---

# 5.2.30. Requisitos arquiteturais da Etapa 5.2

A partir dos problemas identificados, estabelecem-se os seguintes requisitos.

### RSEG1 — Segmentação derivada

A segmentação deve ser derivada da representação canônica, não substituí-la.

### RSEG2 — Não destruição

A segmentação não deve destruir informação existente na representação canônica.

### RSEG3 — Estrutura antes de semântica

A arquitetura deve permitir identificar a estrutura documental antes da definição das unidades destinadas à recuperação.

### RSEG4 — Separação de camadas

Segmentação estrutural, segmentação semântica e unidade de recuperação devem ser conceitualmente distinguíveis.

### RSEG5 — Continuidade

A arquitetura deve preservar relações de continuidade entre unidades.

### RSEG6 — Contexto estrutural

Cada unidade deve poder manter contexto estrutural sem necessariamente incorporá-lo ao conteúdo original.

### RSEG7 — Localização

Cada unidade deve possuir relação rastreável com sua origem documental.

### RSEG8 — Página

A relação com a página de origem deve ser preservada.

### RSEG9 — Localização espacial

Quando disponível na representação canônica, a localização espacial deve poder ser preservada.

### RSEG10 — Elementos complexos

Tabelas, figuras, imagens, legendas, notas e referências não devem ser destruídas simplesmente para produzir chunks textuais.

### RSEG11 — Tabelas

A estrutura necessária para interpretar tabelas deve ser preservada.

### RSEG12 — Relações multimodais

Elementos visuais e textuais relacionados devem permanecer associáveis.

### RSEG13 — Títulos

A hierarquia de títulos deve poder ser utilizada como contexto estrutural.

### RSEG14 — Tamanho

Limites de tamanho devem ser tratados como restrições operacionais, não como definição de significado.

### RSEG15 — Semântica

Uma segmentação baseada apenas em tamanho ou quebra textual não deve ser apresentada como semanticamente orientada.

### RSEG16 — Finalidade

A definição das unidades deve considerar a finalidade da recuperação.

### RSEG17 — Contexto adjacente

Deve ser possível recuperar contexto adicional sem descaracterizar a unidade primária.

### RSEG18 — Overlap controlado

Sobreposição, quando utilizada, deve possuir finalidade definida e ser avaliada quanto aos efeitos sobre duplicação e recuperação.

### RSEG19 — Identidade

Cada unidade deve possuir identidade associável à versão documental e à representação que a originou.

### RSEG20 — Independência metodológica

A unidade de recuperação não deve ser identificada automaticamente como unidade de registro ou unidade de contexto.

### RSEG21 — Independência da relevância

A existência de uma unidade não implica sua relevância metodológica.

### RSEG22 — Independência do embedding

O embedding deve ser derivado da unidade de recuperação e não substituir sua estrutura documental.

### RSEG23 — Avaliação empírica

A qualidade da segmentação deve ser avaliada empiricamente em conjunto com a recuperação.

### RSEG24 — Reprocessamento

A segmentação deve poder ser refeita a partir da representação canônica sem exigir necessariamente nova extração do documento original.

---

# 5.2.31. Arquitetura conceitual resultante

A partir dos requisitos acima, a arquitetura passa a ser representada como:

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
                    │             │
                    │             └── contexto estrutural
                    │
                    ├── localização
                    ├── página
                    ├── bloco
                    ├── relações
                    └── versão
                           │
                           ▼
                      EMBEDDINGS
                           │
                           ▼
                       RETRIEVAL
                           │
                           ▼
                  EVIDÊNCIA RECUPERADA
                           │
                           ▼
                      PESQUISADOR
                           │
                           ▼
            UNIDADE DE REGISTRO / CONTEXTO
```

O ponto fundamental é que a unidade de recuperação possui **duas dimensões simultâneas**:

1. conteúdo suficiente para recuperação;
2. identidade e contexto suficientes para rastreabilidade.

---

# 5.2.32. O que muda em relação ao modelo atual

### Modelo atual

```text
PDF
 ↓
texto
 ↓
limpeza
 ↓
chunk heurístico
 ↓
embedding
```

Nesse modelo, o chunk é produzido sobre uma representação textual que já pode ter perdido características documentais.

### Modelo especificado

```text
PDF
 ↓
representação canônica
 ↓
estrutura documental
 ↓
segmentação estrutural
 ↓
segmentação semântica
 ↓
unidade de recuperação
 ↓
embedding
```

A mudança fundamental não é simplesmente “fazer chunks melhores”.

É mudar o **status arquitetural do chunk**.

No modelo anterior:

> o chunk é praticamente o ponto final da transformação documental.

No modelo novo:

> o chunk é apenas uma representação derivada e descartável/reconstruível para uma finalidade específica.

---

# 5.2.33. Princípio central da Etapa 5.2

A formulação central desta etapa pode ser condensada em:

> **A unidade de recuperação deve ser uma representação derivada do documento, construída a partir de sua estrutura preservada, suficientemente específica para recuperação e suficientemente contextualizada para interpretação, mantendo rastreabilidade até sua origem documental.**

Isso evita dois extremos:

### Extremo 1 — fragmentação

```text
frases isoladas
 ↓
embeddings
 ↓
alta precisão aparente
```

mas com perda de contexto.

### Extremo 2 — blocos gigantes

```text
seções inteiras
 ↓
embeddings
 ↓
grande quantidade de contexto
```

mas com baixa especificidade.

A arquitetura deve buscar uma unidade intermediária cuja adequação seja demonstrada empiricamente.

---

# 5.2.34. Relação com a Etapa 5.3

A Etapa 5.2 não deve resolver antecipadamente problemas que pertencem à recuperação.

Por exemplo, não cabe aqui decidir:

* qual modelo de embedding utilizar;
* qual métrica de similaridade;
* qual threshold;
* qual top-k;
* como fazer reranking;
* como decompor consultas;
* como combinar busca lexical e semântica.

Essas decisões pertencem à **Etapa 5.3 — Motor de Recuperação**.

A Etapa 5.2 deve fornecer a matéria-prima adequada para essas decisões:

> **unidades de recuperação bem definidas, identificáveis, rastreáveis e contextualizáveis.**

---

# 5.2.35. Relação com a Etapa 5.4

A segmentação também fornece a base para a proveniência documental.

A cadeia deverá poder ser reconstruída:

```text
consulta
 ↓
resultado recuperado
 ↓
unidade de recuperação
 ↓
elemento estrutural
 ↓
página
 ↓
documento
 ↓
arquivo original
```

Portanto, a qualidade da proveniência futura depende parcialmente da qualidade da identidade e localização atribuídas nesta etapa.

Se a unidade não souber de onde veio, nenhuma camada posterior poderá recuperar essa informação de forma confiável.

---

# 5.2.36. Relação com a Etapa 5.5

A segmentação também deverá ser versionável.

Uma mudança no método de segmentação pode produzir unidades completamente diferentes a partir do mesmo documento.

Por exemplo:

```text
Documento V1
 + segmentação A
 = chunks A1...A20

Documento V1
 + segmentação B
 = chunks B1...B35
```

Não se deve interpretar isso como alteração no documento original.

É alteração em uma **representação derivada**.

Isso será importante para distinguir:

* versão do documento;
* versão da representação canônica;
* versão da segmentação;
* versão do embedding;
* versão da configuração de recuperação.

Essa distinção será aprofundada na Etapa 5.5.

---

# 5.2.37. Questão da avaliação

A segmentação não deve ser considerada correta apenas porque:

* o sistema funciona;
* as consultas retornam resultados;
* os resultados “parecem bons”.

Será necessário testar empiricamente se diferentes estratégias de segmentação produzem diferenças observáveis na recuperação.

Uma avaliação futura pode comparar, por exemplo:

```text
Segmentação A
        ↓
recuperação
        ↓
qualidade

Segmentação B
        ↓
recuperação
        ↓
qualidade
```

utilizando um corpus controlado e consultas previamente definidas.

Entre os indicadores possíveis estarão:

* precisão;
* cobertura/recall;
* estabilidade;
* redundância;
* qualidade contextual;
* rastreabilidade;
* utilidade para avaliação humana.

A escolha exata dos protocolos de avaliação será tratada posteriormente.

---

# 5.2.38. Testes adversariais específicos para segmentação

A futura validação da segmentação deve incluir documentos que representem situações problemáticas.

Entre elas:

* parágrafos atravessando páginas;
* textos em duas ou mais colunas;
* títulos próximos a quebras de página;
* tabelas extensas;
* tabelas que continuam em outra página;
* figuras com legendas;
* notas de rodapé;
* referências;
* listas;
* documentos escaneados;
* OCR imperfeito;
* fórmulas;
* caracteres especiais;
* cabeçalhos e rodapés;
* documentos com diferentes densidades textuais;
* seções muito curtas;
* seções muito longas;
* documentos com mistura de texto, tabela e imagem.

O objetivo não é apenas verificar se o sistema “não quebra”.

É verificar se a unidade produzida continua representando adequadamente o conteúdo e o contexto necessários à recuperação.

---

# 5.2.39. Decisões que permanecem em aberto

Esta etapa **não escolhe**:

* algoritmo específico de chunking;
* biblioteca;
* modelo de linguagem;
* modelo de embedding;
* tamanho exato das unidades;
* quantidade de overlap;
* formato definitivo de armazenamento;
* mecanismo específico de OCR;
* método específico para tabelas;
* método específico para imagens;
* método específico de reranking;
* banco ou índice específico;
* protocolo final de avaliação.

Essas decisões serão tomadas somente depois que os requisitos forem traduzidos em critérios técnicos.

---

# 5.2.40. Princípios epistemológicos preservados

A Etapa 5.2 mantém os seguintes limites:

1. **Chunk não é unidade de registro.**
2. **Chunk não é unidade de contexto.**
3. **Segmentação não é análise de conteúdo.**
4. **Similaridade não é relevância metodológica.**
5. **Recuperação não é interpretação.**
6. **Contexto computacional não é automaticamente contexto metodológico.**
7. **Embedding não é evidência.**
8. **Unidade recuperada não é evidência metodológica por si mesma.**
9. **Uma representação derivada não substitui o documento original.**
10. **Uma segmentação melhor não recupera informação que foi destruída antes dela.**

---

# 5.2.41. Síntese da Etapa 5.2

O problema da segmentação no Snoopy-RAG não deve ser reduzido à escolha de um tamanho adequado para chunks.

A questão fundamental é definir **como o documento preservado será transformado em unidades computacionais recuperáveis sem romper as estruturas e relações necessárias à interpretação posterior**.

Para isso, a arquitetura deve separar:

```text
DOCUMENTO
   ↓
REPRESENTAÇÃO CANÔNICA
   ↓
ESTRUTURA DOCUMENTAL
   ↓
SEGMENTAÇÃO ESTRUTURAL
   ↓
SEGMENTAÇÃO SEMÂNTICA
   ↓
UNIDADE DE RECUPERAÇÃO
   ↓
EMBEDDING
   ↓
RECUPERAÇÃO
```

A unidade de recuperação deve possuir conteúdo suficiente para representar uma ideia recuperável, mas também manter:

* sua origem;
* sua localização;
* seu contexto estrutural;
* suas relações com outras unidades;
* sua versão;
* sua ligação com o documento original.

A segmentação deve ser tratada como **representação derivada e orientada à finalidade**, não como uma propriedade definitiva do documento.

O sistema não deve confundir:

> **“este é o documento”**

com:

> **“esta é uma forma pela qual o sistema decidiu dividir o documento para recuperá-lo”.**

Essa distinção é a continuação direta do princípio estabelecido na Etapa 5.1.

---

# 5.2.42. Formulação consolidada

> **O Snoopy-RAG deverá derivar suas unidades de recuperação a partir de uma representação canônica e estruturalmente preservadora do documento, distinguindo segmentação estrutural, segmentação semântica e unidades destinadas à recuperação. Essas unidades deverão preservar contexto suficiente para interpretação, manter rastreabilidade até seus elementos de origem e permitir recuperação de contexto adicional sem descaracterizar o conteúdo original. A segmentação computacional não deverá ser confundida com unidade de registro, unidade de contexto ou relevância metodológica, permanecendo essas últimas sob responsabilidade do pesquisador.**

---

# 5.2.43. Princípio final

A Etapa 5.1 estabeleceu:

> **preservar antes de derivar.**

A Etapa 5.2 acrescenta:

> **estruturar antes de segmentar e segmentar antes de indexar.**

Portanto, a cadeia arquitetural consolidada passa a ser:

> **PRESERVAR → ESTRUTURAR → SEGMENTAR → REPRESENTAR → RECUPERAR → INTERPRETAR**

E, sobretudo:

> **O chunk não é o documento. É uma hipótese computacional sobre qual porção do documento deve ser tratada como unidade adequada para uma determinada tarefa de recuperação.**

Essa formulação é importante porque transforma o chunking de uma etapa aparentemente operacional em uma **decisão arquitetural passível de avaliação empírica**.
