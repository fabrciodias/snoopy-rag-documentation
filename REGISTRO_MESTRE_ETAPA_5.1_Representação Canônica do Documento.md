# Registro Mestre da Análise — Snoopy-RAG

## Etapa 5.1 — Representação Canônica do Documento

**Status:** Registro mestre interno de análise
**Natureza:** Especificação metodológica e arquitetural consolidada
**Etapa:** 5 — Gaps e Redesenho Arquitetural
**Subetapa:** 5.1 — Representação Canônica
**Precede:** 5.2 — Segmentação Documental
**Base conceitual:** Etapas 1–4 da análise do Snoopy-RAG

---

## 1. Objetivo deste registro

Este documento registra, de forma detalhada, o raciocínio desenvolvido na Etapa 5.1 da análise do Snoopy-RAG.

Seu objetivo não é documentar uma implementação específica, escolher bibliotecas ou definir tecnologias definitivas. O objetivo é preservar o problema identificado, as distinções conceituais estabelecidas, os requisitos derivados e o princípio arquitetural que deverá orientar a investigação técnica posterior.

A Etapa 5.1 inaugura uma mudança fundamental na arquitetura conceitual do Snoopy-RAG:

> **O documento original deve deixar de ser tratado apenas como matéria-prima para produzir texto pesquisável e passar a ser tratado como uma entidade documental cuja representação deve preservar, tanto quanto possível, as características relevantes para recuperação, contextualização, rastreabilidade e avaliação humana.**

A questão central desta subetapa é:

> **“Qual é a representação mais fiel do documento que ainda seja prática para o Snoopy armazenar, processar e reutilizar?”**

---

# 2. Contexto que levou à Etapa 5.1

Nas etapas anteriores, foi reconstruída a evolução do Snoopy-RAG e identificado que o sistema nasceu de um problema relativamente simples:

> localizar rapidamente informação relevante dentro de um conjunto de PDFs de pesquisa.

A primeira arquitetura era adequada para esse objetivo. O documento era transformado em texto, posteriormente segmentado e utilizado para recuperação.

Com a evolução do projeto, entretanto, o Snoopy passou a assumir funções mais amplas:

* organização de acervos;
* pesquisa semântica;
* recuperação de evidências;
* contextualização;
* rastreabilidade;
* leitura documental;
* suporte à pesquisa;
* utilização por múltiplos usuários;
* possibilidade de avaliação da própria recuperação.

Essa evolução alterou a natureza do problema.

Um sistema cuja finalidade é simplesmente produzir texto pesquisável pode aceitar determinadas perdas documentais em troca de simplicidade.

Uma infraestrutura de exploração documental para pesquisa não pode fazer isso sem avaliar metodologicamente as consequências dessas perdas.

---

# 3. O problema central identificado

O problema mais importante identificado na arquitetura atual não é simplesmente um “bug no extrator”.

O problema é mais profundo:

> **O Snoopy transforma o documento antes de estabelecer quais características do documento precisam ser preservadas.**

O fluxo atual pode ser representado aproximadamente como:

```text
PDF
 ↓
extração de texto
 ↓
limpeza
 ↓
Markdown/texto
 ↓
chunking
 ↓
embeddings
 ↓
recuperação
```

Nesse processo, cada transformação pode descartar informações presentes no documento original.

O sistema passa, portanto, a pesquisar não diretamente o documento, mas uma representação já transformada dele.

Isso produz uma questão metodológica:

> **Se uma característica relevante do documento for perdida durante a transformação, o sistema poderá deixar de recuperar determinado material antes mesmo de o pesquisador ter a oportunidade de avaliá-lo.**

---

# 4. Por que a perda de informação é um problema metodológico

A princípio, seria possível argumentar:

> “O PDF original continua disponível. Se o Snoopy perder alguma coisa, o pesquisador pode abrir o PDF.”

Essa solução é insuficiente.

O problema não ocorre apenas na etapa final de leitura.

O problema ocorre anteriormente, durante a **seleção do material que chega à atenção do pesquisador**.

Considere, por exemplo:

```text
PDF original
   ↓
tabela perdida durante a extração
   ↓
informação ausente da representação pesquisável
   ↓
consulta não encontra o conteúdo
   ↓
documento aparece como pouco relevante
   ↓
pesquisador pode não selecioná-lo
```

O pesquisador pode abrir o PDF posteriormente, mas talvez nunca o faça.

Assim, uma perda de informação durante a transformação pode afetar a própria exposição do pesquisador ao corpus.

---

# 5. Consequência: transformação documental pode afetar seleção

Essa constatação é fundamental.

A transformação:

```text
PDF → texto
```

não é metodologicamente neutra quando o texto transformado é utilizado para determinar quais documentos ou passagens serão recuperados.

A cadeia pode ser:

```text
documento original
      ↓
representação computacional
      ↓
recuperação
      ↓
material apresentado ao pesquisador
      ↓
decisão humana
```

Logo, problemas na representação podem introduzir uma forma de **viés de seleção computacional** antes da análise metodológica propriamente dita.

Isso não significa afirmar que o Snoopy automaticamente produz viés metodológico em qualquer circunstância.

Significa que:

> **A arquitetura de representação do documento pode influenciar quais partes do corpus são tornadas visíveis ao pesquisador e, portanto, merece tratamento metodológico explícito.**

---

# 6. O problema específico do PDF

O PDF não é simplesmente um arquivo de texto.

Ele pode conter simultaneamente:

* texto;
* páginas;
* blocos;
* ordem espacial;
* títulos;
* subtítulos;
* parágrafos;
* listas;
* tabelas;
* imagens;
* gráficos;
* diagramas;
* legendas;
* notas de rodapé;
* referências;
* cabeçalhos;
* rodapés;
* elementos posicionados espacialmente;
* caracteres especiais;
* fontes e formatações;
* documentos digitalizados;
* conteúdo obtido por OCR.

Uma transformação que preserve somente uma sequência linear de caracteres não necessariamente preserva a estrutura documental.

---

# 7. O que ocorre no pipeline atual

A extração atualmente utilizada trabalha com blocos textuais do PDF e transforma esses blocos em uma sequência textual.

Entre as operações identificadas estão:

* extração de blocos;
* filtragem de elementos;
* remoção de quebras;
* normalização de espaços;
* transformação em texto;
* limpeza;
* inferência heurística de títulos;
* segmentação posterior.

Essa abordagem é suficiente para produzir um corpus textual pesquisável em muitos documentos, mas possui limitações importantes para uma infraestrutura documental.

Entre as perdas ou riscos identificados estão:

* perda de coordenadas;
* perda da informação espacial;
* perda de elementos que não são blocos textuais;
* perda de imagens;
* perda de relações entre texto e imagem;
* dificuldade com tabelas;
* dificuldade com layouts complexos;
* dificuldade com múltiplas colunas;
* perda de continuidade documental;
* dificuldades com documentos digitalizados;
* possíveis problemas com OCR;
* perda de informações estruturais;
* transformação de diferentes elementos documentais em uma sequência textual homogênea.

A conclusão não é que toda transformação textual seja inadequada.

A conclusão é que:

> **O texto pesquisável deve ser entendido como uma representação derivada do documento, e não como o próprio documento.**

---

# 8. Transformação destrutiva

Toda transformação de um documento envolve alguma forma de alteração.

Portanto, o objetivo não deve ser:

> “não transformar o documento.”

Isso seria impraticável.

O objetivo deve ser:

> **minimizar transformações destrutivas.**

Uma transformação é destrutiva quando elimina informação que pode ser relevante para uma finalidade posterior e não mantém uma forma adequada de recuperar essa informação.

Exemplo:

```text
PDF
 ↓
texto linear
 ↓
posição dos elementos perdida
```

Se a posição for relevante para compreender uma tabela, gráfico, coluna ou relação visual, essa transformação pode ser destrutiva.

---

# 9. Princípio da preservação antes da derivação

A consequência arquitetural central é:

> **Primeiro preservar; depois derivar.**

A arquitetura desejada passa a ser conceitualmente:

```text
DOCUMENTO ORIGINAL
        ↓
REPRESENTAÇÃO CANÔNICA PRESERVADORA
        ↓
REPRESENTAÇÕES DERIVADAS
        ├── texto pesquisável
        ├── estrutura
        ├── segmentação
        ├── elementos multimodais
        ├── embeddings
        ├── leitura
        └── síntese
```

Isso substitui a lógica:

```text
PDF → texto → chunks → embeddings
```

por:

```text
PDF
 ↓
representação canônica
 ↓
representações derivadas
 ↓
recuperação
```

---

# 10. O conceito de representação canônica

A representação canônica é a representação persistente do documento que deve funcionar como referência para as demais transformações.

Ela não precisa ser uma cópia binária do PDF.

Também não precisa reproduzir visualmente o PDF de maneira perfeita.

Sua finalidade é outra:

> **preservar de maneira estruturada as características documentais relevantes que poderão ser necessárias posteriormente.**

Ela deve ser suficientemente rica para permitir a criação de representações derivadas sem que o sistema precise necessariamente retornar ao arquivo original e repetir toda a extração.

---

# 11. O que a representação canônica não é

É importante estabelecer explicitamente o que **não** deve ser confundido com representação canônica.

### Não é Markdown

Markdown pode ser uma excelente representação derivada para leitura ou processamento, mas não preserva necessariamente todas as características documentais.

### Não é texto puro

Texto linear elimina informações estruturais e espaciais.

### Não é chunk

Chunk é uma unidade computacional derivada para determinada finalidade de recuperação.

### Não é embedding

Embedding é uma representação vetorial destinada a operações de similaridade ou recuperação.

### Não é resposta de LLM

Uma resposta gerada é uma representação derivada de outras informações.

Portanto:

> **A representação canônica existe antes dessas representações e não deve depender delas para preservar o documento.**

---

# 12. Características que devem ser preservadas

A representação canônica deve buscar preservar, quando presentes e relevantes:

### Identidade documental

* origem;
* identificadores;
* metadados disponíveis;
* relação com o arquivo original;
* identidade da versão.

### Estrutura espacial e física

* páginas;
* ordem de leitura;
* blocos;
* posição;
* coordenadas;
* bounding boxes, quando disponíveis.

### Estrutura lógica

* títulos;
* subtítulos;
* seções;
* subseções;
* parágrafos;
* listas;
* itens numerados;
* citações;
* referências.

### Elementos complexos

* tabelas;
* imagens;
* gráficos;
* diagramas;
* figuras;
* legendas;
* notas de rodapé;
* cabeçalhos;
* rodapés.

### Conteúdo não diretamente textual

* imagens;
* elementos visuais;
* documentos digitalizados;
* informação obtida por OCR;
* relações entre elementos.

### Características linguísticas e textuais

* caracteres especiais;
* acentuação;
* símbolos;
* fórmulas;
* quebras relevantes;
* continuidade entre páginas.

---

# 13. Página como unidade de localização

A página possui importância particular.

Não porque toda análise deva ser feita por página, mas porque ela fornece uma referência documental relativamente estável.

A representação deve permitir algo conceitualmente semelhante a:

```text
Documento
 └── Página 12
      ├── bloco 1
      ├── bloco 2
      ├── tabela
      └── legenda
```

Isso possibilita que uma unidade recuperada seja relacionada ao local em que aparece no documento.

A página não deve ser confundida com chunk ou unidade metodológica.

Ela é uma dimensão documental de localização.

---

# 14. Ordem de leitura

A ordem de leitura precisa ser tratada explicitamente.

Um PDF pode apresentar visualmente:

```text
COLUNA A | COLUNA B
```

mas uma extração inadequada pode produzir:

```text
A1
B1
A2
B2
```

quando a ordem esperada seria:

```text
A1
A2
B1
B2
```

ou outra organização determinada pelo documento.

A representação canônica deve conservar informação suficiente para reconstruir a ordem de leitura considerada apropriada.

Não se deve simplesmente assumir que a ordem física dos elementos equivale sempre à ordem textual.

---

# 15. Blocos documentais

Os blocos são importantes porque permitem preservar uma unidade intermediária entre:

```text
página
```

e:

```text
texto individual
```

Um bloco pode representar:

* parágrafo;
* título;
* legenda;
* nota;
* referência;
* elemento de tabela;
* outro conteúdo identificado.

Isso permite manter relações estruturais e espaciais que desapareceriam em uma simples sequência textual.

---

# 16. Tabelas

Tabelas constituem um caso particularmente importante.

Transformar:

```text
Tabela
┌────────┬────────┐
│ A      │ B      │
├────────┼────────┤
│ 10     │ 20     │
└────────┴────────┘
```

em:

```text
A B 10 20
```

pode preservar palavras e números, mas destruir a relação estrutural entre linhas e colunas.

Para pesquisa, isso pode ser uma perda significativa.

Portanto, a representação canônica deve buscar preservar a tabela como elemento estruturado ou, quando isso não for possível, manter informação suficiente para reconstruir sua estrutura.

A forma concreta de implementação permanece em aberto nesta etapa.

---

# 17. Imagens, figuras e gráficos

O mesmo princípio vale para elementos visuais.

Uma imagem não deve simplesmente desaparecer porque não possui texto extraível.

A arquitetura deve poder registrar:

```text
imagem
 ↓
posição
 ↓
página
 ↓
relação com legenda
 ↓
relação com texto circundante
```

Isso não significa que toda imagem necessariamente precise ser transformada em texto.

Significa que sua existência e sua relação documental devem ser preservadas quando forem relevantes.

---

# 18. OCR e documentos digitalizados

Documentos digitalizados introduzem outro caso importante.

Um PDF pode conter essencialmente imagens de páginas:

```text
PDF
 ↓
imagem da página
```

Nesse caso, uma extração textual convencional pode produzir pouco ou nenhum texto.

O sistema precisa distinguir:

```text
documento sem texto
```

de:

```text
documento cujo texto está presente visualmente,
mas não está disponível como camada textual.
```

O OCR pode produzir uma representação textual derivada, mas não deve necessariamente substituir a representação visual original.

Portanto:

```text
página digitalizada
      ↓
representação preservada
      +
texto obtido por OCR
```

é conceitualmente mais seguro do que:

```text
página digitalizada
      ↓
OCR
      ↓
texto
      ↓
descartar original
```

---

# 19. Elementos multimodais

A evolução dos modelos de representação permite considerar também conteúdo multimodal.

Isso abre possibilidades para:

* imagens;
* tabelas;
* páginas;
* elementos visuais;
* conteúdo textual;
* relações entre modalidades.

Entretanto, é importante manter uma distinção:

> **Uma representação vetorial multimodal não é substituta da representação documental canônica.**

Um embedding pode representar semanticamente determinado conteúdo, mas não é adequado como fonte primária para reconstruir fielmente o documento.

Portanto:

```text
representação canônica
        ↓
representação multimodal
```

e não:

```text
PDF
 ↓
embedding
 ↓
"documento preservado"
```

---

# 20. Representação canônica e armazenamento

Existe uma tensão real entre fidelidade e custo.

Quanto mais informações forem preservadas, maior pode ser:

* o tamanho;
* o custo de armazenamento;
* o custo de processamento;
* a complexidade;
* a dificuldade de manutenção.

Por isso, a pergunta central não é:

> “Como preservar absolutamente tudo?”

mas:

> **“Qual é a representação mais fiel do documento que ainda seja prática para o Snoopy armazenar, processar e reutilizar?”**

Esse equilíbrio deve orientar a arquitetura.

---

# 21. Evitar redundância excessiva

Preservar informação não significa armazenar todas as versões intermediárias.

Uma arquitetura ingênua poderia produzir:

```text
PDF
+
texto bruto
+
Markdown
+
HTML
+
JSON
+
OCR
+
imagens
+
tabelas
+
chunks
+
embeddings
+
outras derivações
```

Isso pode criar redundância significativa.

O objetivo deve ser estabelecer uma **representação canônica suficientemente rica** e, a partir dela, gerar somente as representações derivadas necessárias.

Assim:

```text
CANÔNICO
   ├── derivação A
   ├── derivação B
   ├── derivação C
   └── derivação D
```

em vez de uma cadeia em que cada transformação destrói a anterior.

---

# 22. Representação canônica para máquina e humano

Outra decisão conceitual importante é abandonar a ideia de que a representação persistente precisa ser exclusivamente “machine-only”.

Se a representação canônica for suficientemente estruturada e preservadora, ela deve ser útil tanto para:

* processamento computacional;
* reconstrução de contexto;
* auditoria;
* inspeção humana;
* desenvolvimento futuro;
* reprocessamento.

Isso não significa que ela precise ser uma interface amigável.

Significa que não deve ser deliberadamente construída de forma que somente o algoritmo consiga utilizá-la.

---

# 23. Separação entre preservação e recuperação

A representação utilizada para recuperação pode ser diferente da representação canônica.

Por exemplo:

```text
CANÔNICO
   ↓
texto pesquisável
   ↓
segmentação
   ↓
embedding
```

O texto pesquisável pode ser otimizado para:

* busca;
* normalização;
* recuperação;
* processamento linguístico.

O embedding pode ser otimizado para:

* representação semântica;
* similaridade.

Nenhum deles precisa carregar sozinho todas as características documentais.

A representação canônica funciona como referência preservadora.

---

# 24. Possibilidade de reprocessamento

Uma das maiores vantagens dessa arquitetura é permitir que decisões futuras sejam revistas.

Por exemplo, se posteriormente concluirmos que a segmentação utilizada não é adequada:

```text
PDF
 ↓
CANÔNICO PERSISTENTE
 ↓
nova segmentação
 ↓
novas unidades
```

não será necessário necessariamente:

```text
PDF
 ↓
nova extração
 ↓
nova limpeza
 ↓
nova estruturação
 ↓
nova segmentação
```

Isso separa o problema de **preservar o documento** do problema de **decidir como recuperá-lo**.

Essa separação é fundamental para a evolução do Snoopy.

---

# 25. Preservação e rastreabilidade

A representação canônica também constitui a base da proveniência definida posteriormente na Etapa 5.4.

Uma unidade recuperada deve poder apontar conceitualmente para:

```text
unidade
 ↓
elemento documental
 ↓
bloco
 ↓
página
 ↓
versão
 ↓
documento
 ↓
fonte original
```

Sem uma representação suficientemente estruturada, essa cadeia se torna muito mais difícil de preservar.

Portanto, a decisão tomada em 5.1 não é isolada.

Ela condiciona diretamente:

* segmentação;
* recuperação;
* proveniência;
* versionamento;
* leitura;
* avaliação.

---

# 26. Preservação não significa reprodução visual perfeita

É necessário estabelecer outro limite.

O objetivo não é necessariamente reconstruir uma cópia visual idêntica do PDF dentro do Snoopy.

Isso poderia gerar complexidade desnecessária.

O objetivo é preservar **as características documentais relevantes para as finalidades do sistema**.

Assim:

```text
fidelidade documental
≠
reprodução visual perfeita
```

A representação canônica pode ser diferente do PDF em sua forma e ainda assim preservar adequadamente:

* conteúdo;
* estrutura;
* localização;
* relações;
* contexto;
* elementos relevantes.

---

# 27. Critério para decidir o que preservar

Para cada transformação ou perda potencial, deve-se perguntar:

### 1. Qual é a finalidade da transformação?

Por que determinada informação está sendo removida ou simplificada?

### 2. Qual informação será perdida?

A perda precisa ser conhecida, não presumida.

### 3. Essa perda é aceitável para a finalidade?

Uma simplificação pode ser aceitável para uma representação específica.

### 4. A informação continua recuperável de outra representação?

Se o dado for necessário posteriormente, existe outra fonte preservada?

### 5. A origem continua rastreável?

Mesmo depois da transformação, é possível voltar ao elemento original?

A regra geral derivada é:

> **Se uma transformação não possui finalidade explícita, produz perda relevante, não oferece recuperação alternativa e rompe a rastreabilidade, ela não deve ser adotada como transformação destrutiva da representação persistente.**

---

# 28. Princípio da reversibilidade prática

Não é necessário exigir reversibilidade matemática completa.

O objetivo é uma forma de **reversibilidade prática**:

> uma representação derivada deve poder ser relacionada novamente à representação documental preservada que lhe deu origem.

Por exemplo:

```text
chunk
 ↓
elementos canônicos
 ↓
página
 ↓
documento
```

Isso é mais importante para o Snoopy do que exigir que toda derivação seja literalmente invertível.

---

# 29. Relação com a recuperação

A recuperação deve operar sobre representações derivadas adequadas ao seu propósito.

Portanto:

```text
CANÔNICO
    ↓
estrutura
    ↓
segmentação
    ↓
unidades de recuperação
    ↓
embeddings / outras representações
```

A recuperação não deve modificar a representação canônica.

Uma consulta não deve produzir uma nova versão do documento.

O processo de recuperação apenas seleciona representações derivadas relacionadas ao documento preservado.

---

# 30. Relação com a síntese

A síntese por LLM também deve ser entendida como uma representação derivada.

A sequência conceitual é:

```text
documento original
 ↓
representação canônica
 ↓
unidade recuperada
 ↓
contexto
 ↓
síntese
```

A síntese não substitui o documento nem a unidade recuperada.

Isso mantém o princípio estabelecido anteriormente:

> **Síntese do LLM é representação derivada, não evidência primária.**

---

# 31. Relação com a análise metodológica

A representação canônica também não transforma automaticamente o Snoopy em uma ferramenta de análise de conteúdo.

Mesmo que o documento esteja perfeitamente representado:

```text
documento
 ↓
representação
 ↓
segmentação
 ↓
recuperação
```

a decisão metodológica permanece com o pesquisador.

A representação canônica apenas melhora a qualidade da infraestrutura sobre a qual essa decisão pode ocorrer.

Portanto:

> **Preservação documental não é análise metodológica.**

---

# 32. Requisitos derivados — Representação Canônica

### RCAN1 — Preservação documental

O sistema deve preservar uma representação suficientemente fiel das características documentais relevantes para suas finalidades de recuperação, contextualização, rastreabilidade e avaliação humana.

### RCAN2 — Minimização de transformações destrutivas

Transformações aplicadas ao documento devem minimizar perdas de informação potencialmente relevantes.

### RCAN3 — Identidade documental

A representação deve manter relação identificável com o documento original e sua fonte.

### RCAN4 — Estrutura de páginas

A representação deve preservar a relação entre conteúdo e página.

### RCAN5 — Ordem de leitura

A representação deve preservar ou permitir reconstruir a ordem de leitura relevante do documento.

### RCAN6 — Blocos documentais

A representação deve preservar unidades estruturais intermediárias, quando disponíveis e relevantes.

### RCAN7 — Estrutura lógica

Títulos, seções, parágrafos, listas, referências, notas e outras estruturas relevantes devem ser preservados quando identificáveis.

### RCAN8 — Elementos complexos

Tabelas, figuras, imagens, gráficos, legendas e outros elementos documentais relevantes não devem ser descartados simplesmente por não serem texto linear.

### RCAN9 — Relações espaciais

Informações espaciais relevantes devem ser preservadas sempre que disponíveis, incluindo localização e, quando apropriado, bounding boxes.

### RCAN10 — Documentos digitalizados

O sistema deve distinguir conteúdo textual nativo de conteúdo obtido por OCR e preservar a relação entre ambos.

### RCAN11 — Conteúdo multimodal

Elementos não textuais relevantes devem permanecer representáveis mesmo quando a recuperação posterior utilize representações vetoriais ou textuais.

### RCAN12 — Separação entre canônico e derivado

A representação canônica não deve ser confundida com Markdown, texto puro, chunk, embedding ou resposta de modelo.

### RCAN13 — Derivação não destrutiva

Representações utilizadas para recuperação devem ser derivadas da representação canônica sem substituí-la.

### RCAN14 — Reprocessamento

A representação canônica deve permitir a criação de novas representações derivadas sem exigir necessariamente nova extração do documento original.

### RCAN15 — Persistência

A representação canônica deve possuir persistência suficiente para funcionar como referência estável das representações derivadas.

### RCAN16 — Rastreabilidade

Toda representação derivada relevante deve poder ser relacionada à representação canônica e, posteriormente, ao documento original.

### RCAN17 — Praticidade

A representação canônica deve equilibrar fidelidade documental, custo de armazenamento, complexidade operacional e possibilidade de reutilização.

### RCAN18 — Não redundância indiscriminada

A arquitetura não deve exigir a persistência de todas as representações intermediárias quando uma representação canônica suficientemente rica permitir sua reconstrução.

### RCAN19 — Utilidade dual

A representação persistida deve ser suficientemente estruturada para atender às necessidades de processamento computacional e inspeção/reconstrução humana.

### RCAN20 — Finalidade explícita

Toda transformação potencialmente destrutiva deve possuir finalidade definida e justificável.

### RCAN21 — Recuperabilidade

Informações consideradas relevantes não devem ser eliminadas quando não houver outra representação capaz de recuperá-las posteriormente.

### RCAN22 — Separação da análise

A representação documental não deve incorporar decisões metodológicas que pertençam ao pesquisador.

---

# 33. Arquitetura conceitual resultante

A partir dos problemas e requisitos identificados, a arquitetura deixa de ser:

```text
PDF
 ↓
Markdown
 ↓
chunks
 ↓
embeddings
 ↓
resposta
```

e passa a ser conceitualmente:

```text
                    DOCUMENTO ORIGINAL
                           │
                           ▼
              REPRESENTAÇÃO CANÔNICA
                    PRESERVADORA
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          estrutura     texto         elementos
          documental   pesquisável    multimodais
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                    REPRESENTAÇÕES
                       DERIVADAS
                           │
                           ▼
                      SEGMENTAÇÃO
                           │
                           ▼
                 UNIDADES DE RECUPERAÇÃO
                           │
                           ▼
                  EMBEDDINGS / RETRIEVAL
```

Todas as representações derivadas devem manter relação rastreável com sua origem.

---

# 34. Princípio arquitetural central

A Etapa 5.1 estabelece o seguinte princípio:

> **O documento original deve ser preservado por meio de uma representação canônica suficientemente fiel e persistente, a partir da qual possam ser derivadas representações específicas para estruturação, segmentação, recuperação, leitura e síntese sem que essas derivações substituam ou destruam a representação documental de referência.**

---

# 35. Consequências para as próximas subetapas

A decisão tomada em 5.1 estabelece restrições para as etapas seguintes.

### Para 5.2 — Segmentação

A segmentação deverá operar sobre uma representação que preserve estrutura documental.

Portanto:

```text
canônico
 ↓
estrutura
 ↓
segmentação
```

e não:

```text
texto já achatado
 ↓
tentativa de reconstrução da estrutura
 ↓
segmentação
```

### Para 5.3 — Recuperação

As unidades recuperáveis deverão possuir relação com elementos documentais preservados.

### Para 5.4 — Proveniência

A origem de uma unidade deverá poder ser rastreada até sua localização documental.

### Para 5.5 — Persistência e versionamento

A representação canônica deverá possuir identidade e versão próprias, permitindo determinar quais representações derivadas continuam válidas.

### Para 5.6 — Segurança e operações

O armazenamento e processamento dessa representação deverão ocorrer dentro de uma arquitetura capaz de proteger os diferentes acervos e controlar os processos que a modificam.

---

# 36. O que esta etapa deliberadamente NÃO decide

A Etapa 5.1 não determina:

* uma biblioteca específica;
* um formato final específico;
* um banco específico;
* um modelo específico;
* uma estratégia específica de OCR;
* uma estratégia específica de multimodalidade;
* um mecanismo específico de armazenamento;
* uma arquitetura específica de indexação;
* um algoritmo específico de reconstrução;
* um modelo específico de chunking.

Essas questões pertencem à investigação técnica posterior.

O papel desta etapa é definir **as propriedades que qualquer solução técnica deverá satisfazer**.

---

# 37. Distinção entre requisito e tecnologia

A formulação correta é:

> “Precisamos preservar relações espaciais relevantes.”

e não:

> “Precisamos usar determinada biblioteca porque ela preserva relações espaciais.”

A primeira é uma exigência arquitetural.

A segunda já é uma hipótese de implementação.

A Etapa 6 deverá investigar quais tecnologias podem satisfazer a primeira.

Essa separação é deliberada para evitar uma arquitetura orientada pela ferramenta.

---

# 38. Limites epistemológicos

A representação canônica melhora a infraestrutura documental, mas não resolve todos os problemas metodológicos.

Ela não garante:

* relevância metodológica;
* qualidade da recuperação;
* interpretação correta;
* categorização;
* unidade de registro;
* unidade de contexto;
* análise de conteúdo;
* validade científica das conclusões.

Ela apenas reduz uma classe anterior de problemas:

> **a possibilidade de que a infraestrutura altere ou perca informações documentais antes que o pesquisador possa avaliá-las.**

---

# 39. Axiomas preservados

A Etapa 5.1 mantém os princípios estabelecidos nas etapas anteriores:

1. **recuperação ≠ análise;**
2. **similaridade semântica ≠ relevância metodológica;**
3. **chunk ≠ unidade de registro ou unidade de contexto;**
4. **rastreabilidade da fonte ≠ proveniência metodológica completa;**
5. **síntese do LLM ≠ evidência primária.**

E acrescenta um princípio arquitetural anterior a todos eles:

6. **representação derivada ≠ documento original.**

---

# 40. Síntese do raciocínio

O raciocínio desenvolvido na Etapa 5.1 pode ser condensado na seguinte cadeia:

```text
PDF é mais do que texto
        ↓
extração pode perder características documentais
        ↓
perda pode afetar recuperação
        ↓
recuperação influencia o que chega ao pesquisador
        ↓
portanto, transformação pode afetar seleção documental
        ↓
é necessário minimizar perdas
        ↓
preservar antes de derivar
        ↓
criar representação canônica persistente
        ↓
derivar texto, estrutura, segmentação e recuperação
        ↓
manter rastreabilidade entre todas as representações
```

---

# 41. Conclusão da Etapa 5.1

A principal conclusão desta subetapa é que o problema da extração documental do Snoopy-RAG não deve ser tratado apenas como uma questão de melhorar o parser ou substituir uma biblioteca.

O problema é arquitetural.

A arquitetura atual parte de uma transformação relativamente agressiva:

```text
PDF → texto → chunks → embeddings
```

porque essa transformação era adequada ao objetivo inicial de tornar um conjunto de PDFs pesquisável.

Com a evolução do Snoopy para uma infraestrutura de exploração documental voltada à pesquisa, essa premissa deixou de ser suficiente.

A arquitetura futura deve inverter a prioridade:

```text
DOCUMENTO ORIGINAL
        ↓
REPRESENTAÇÃO CANÔNICA PRESERVADORA
        ↓
REPRESENTAÇÕES DERIVADAS
        ↓
SEGMENTAÇÃO
        ↓
RECUPERAÇÃO
        ↓
EVIDÊNCIA
        ↓
SÍNTESE
```

O princípio fundamental é:

> **Preservar primeiro. Derivar depois.**

A representação canônica não precisa reproduzir visualmente o PDF de maneira perfeita, nem armazenar todas as transformações possíveis. Ela precisa ser suficientemente fiel para preservar as características documentais relevantes, suficientemente prática para o sistema e suficientemente estruturada para permitir novas derivações sem depender continuamente de uma nova extração do arquivo original.

A consequência mais importante é que **o documento deixa de ser tratado como uma sequência textual produzida pelo sistema e passa a ser tratado como a origem persistente de múltiplas representações derivadas**.

Essa mudança é a base sobre a qual as demais partes da Etapa 5 devem ser construídas:

```text
5.1
O que é o documento que precisamos preservar?
        ↓
REPRESENTAÇÃO CANÔNICA

5.2
Como transformar esse documento preservado em unidades?
        ↓
SEGMENTAÇÃO

5.3
Como localizar essas unidades?
        ↓
RECUPERAÇÃO

5.4
Como demonstrar de onde vieram?
        ↓
PROVENIÊNCIA

5.5
Como manter tudo coerente ao longo do tempo?
        ↓
PERSISTÊNCIA / VERSIONAMENTO / CONSISTÊNCIA

5.6
Como garantir que tudo opere de forma segura?
        ↓
SEGURANÇA / OPERAÇÕES
```

Portanto, a decisão fundamental da Etapa 5.1 pode ser expressa de maneira definitiva:

> **O Snoopy-RAG não deve utilizar uma representação textual derivada como substituta do documento. Deve preservar uma representação canônica do documento e utilizar representações derivadas, cada uma adequada à sua finalidade, mantendo entre elas uma relação rastreável e não destrutiva.**

**Fim do Registro Mestre — Etapa 5.1.**
