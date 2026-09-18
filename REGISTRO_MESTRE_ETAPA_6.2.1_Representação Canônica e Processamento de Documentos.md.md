# Registro Mestre da Análise — Snoopy-RAG
## Etapa 6.2.1 — Representação Canônica e Processamento de Documentos

**Status:** Registro Mestre da Etapa 6.2.1 — consolidado  
**Natureza:** registro de investigação tecnológica e consolidação de conclusões para a V3  
**Relação com a Etapa 6.1:** tradução e verificação das necessidades da representação documental diante das alternativas tecnológicas  
**Relação com a Etapa 6.2:** primeira subetapa da investigação das alternativas tecnológicas  
**Escopo:** representação canônica e processamento inicial de documentos, com foco em documentos PDF científicos do corpus do GEPAFOR

---

# 1. Finalidade da Etapa 6.2.1

A Etapa 6.2.1 teve como finalidade investigar quais abordagens tecnológicas podem fornecer ao Snoopy uma representação documental suficientemente fiel, estruturada, rastreável e reprocessável para satisfazer os critérios técnicos derivados dos requisitos das etapas anteriores.

A investigação partiu de uma distinção fundamental:

> **Extrair texto de um PDF e construir uma representação documental são problemas diferentes.**

O Snoopy V2 trata atualmente uma extração textual como base do processamento posterior. A V3 deve preservar uma representação documental mais rica antes de produzir as representações destinadas à segmentação, recuperação e síntese.

O procedimento seguido foi:

> **requisito → critério técnico → alternativas → comparação → conclusão**

A etapa não constitui ainda o projeto técnico definitivo nem determina as bibliotecas, modelos, formatos físicos ou tecnologias concretas da implementação.

---

# 2. Problema arquitetural investigado

O problema central identificado foi a transformação potencialmente destrutiva:

```text
PDF
 ↓
extração textual
 ↓
limpeza
 ↓
chunk
 ↓
embedding
```

Esse fluxo transforma uma representação textual derivada na principal base para as operações seguintes. Quando a extração perde estrutura, posição, relações ou elementos não textuais, as camadas posteriores deixam de ter acesso a essas informações por meio da representação processada.

A direção desejada para a V3 é:

```text
PDF original
 ↓
processamento documental
 ↓
representação canônica
 ↓
representações derivadas
 ↓
segmentação
 ↓
unidades de recuperação
 ↓
embeddings / índices
 ↓
recuperação
 ↓
síntese
```

O objetivo não é impedir transformações, mas impedir que a primeira transformação determine irreversivelmente aquilo que o sistema poderá fazer posteriormente.

---

# 3. Espaço de soluções investigado

A investigação mostrou que o problema não deve ser reduzido à escolha de um único “parser de PDF”. Foram identificadas pelo menos quatro famílias:

1. **Acesso estrutural ao PDF** — acesso direto a objetos nativos, texto, palavras, blocos, coordenadas, imagens e outros elementos.
2. **Document parsing / document understanding** — interpretação do documento e reconstrução de estruturas como títulos, parágrafos, tabelas, figuras, seções e ordem de leitura.
3. **OCR e análise de layout** — reconstrução de informação quando a camada textual não existe ou não é confiável.
4. **Serviços especializados de extração** — utilização de APIs externas para produzir representações documentais estruturadas.

Também foram consideradas ferramentas especializadas para funções como tabelas, OCR, fórmulas, layout e elementos visuais.

A conclusão arquitetural é que essas tecnologias **não necessariamente competem pelo mesmo papel**.

---

# 4. Distinção entre acesso, interpretação e derivação

A investigação consolidou três níveis tecnológicos distintos.

## 4.1 Acesso ao documento

É a camada mais próxima da fonte original. Sua pergunta é:

> **O que existe física ou estruturalmente no arquivo?**

Pode fornecer páginas, blocos, linhas, spans, caracteres, coordenadas, fontes, imagens e outros objetos.

## 4.2 Interpretação documental

É a camada que tenta responder:

> **O que esses elementos significam enquanto estrutura documental?**

Aqui entram classificações como título, parágrafo, tabela, figura, legenda, seção, cabeçalho, rodapé e relações hierárquicas.

## 4.3 Representação derivada

É a camada destinada a uma finalidade específica, incluindo:

- Markdown;
- texto linear;
- HTML;
- texto para embedding;
- representações textuais de tabelas;
- descrições de imagens;
- unidades de recuperação.

A distinção fundamental é:

> **Uma representação utilizada para recuperação não precisa ser a mesma representação utilizada para preservação.**

---

# 5. Avaliação das principais abordagens

## 5.1 PyMuPDF

A investigação mostrou que o uso atual do PyMuPDF pelo Snoopy é apenas uma utilização simples de uma capacidade mais ampla. O sistema atual utiliza essencialmente `page.get_text("blocks")`, enquanto a biblioteca oferece níveis mais ricos, incluindo blocos, palavras, linhas, spans, caracteres, coordenadas, bounding boxes, fontes, imagens e outras informações estruturais.

Isso torna o acesso nativo particularmente valioso como camada próxima da fonte.

Sua limitação arquitetural não é simplesmente falta de informação, mas a diferença entre:

> **saber onde os objetos estão**

 e:

> **saber qual estrutura documental eles formam.**

A ordem interna do PDF pode não coincidir com a ordem natural de leitura, especialmente em documentos de múltiplas colunas. Portanto, acesso geométrico não equivale automaticamente a reconstrução semântica.

**Papel potencial:** acesso nativo, preservação de informação de baixo nível, geometria, localização e complementação de parsers estruturais.

Não se mostrou, isoladamente, suficiente para a interpretação documental completa.

## 5.2 pdfplumber

O pdfplumber apresentou força principalmente na dimensão geométrica e em operações especializadas, especialmente na análise de tabelas.

Sua capacidade de trabalhar com caracteres, palavras, linhas, retângulos, curvas, posições e estratégias de detecção tabular demonstra o valor de mecanismos especializados.

Entretanto, uma tabela detectada geometricamente continua sendo uma **interpretação** da disposição espacial e pode estar correta ou incorreta.

**Papel potencial:** componente especializado e complementar, não representação documental central.

## 5.3 Docling

O Docling apresentou forte correspondência com o problema arquitetural porque possui explicitamente um modelo documental estruturado. Sua representação distingue textos, tabelas, imagens, grupos, corpo principal, elementos periféricos, hierarquia, relações entre elementos, ordem de leitura, informações de layout e proveniência.

Sua estrutura é muito mais próxima do que a V3 necessita do que uma simples sequência textual.

A proveniência pode fornecer referências de página, bounding box e intervalo de caracteres, constituindo uma âncora importante para rastreabilidade.

Entretanto:

> **A representação do Docling não deve ser confundida automaticamente com a representação canônica do Snoopy.**

Ela pode funcionar como representação documental intermediária e como fonte para a construção do modelo do Snoopy.

## 5.4 MinerU

O MinerU apresentou uma abordagem particularmente relevante para documentos complexos e multimodais. Sua representação trabalha com páginas, blocos, linhas, spans e diferentes tipos de elementos, incluindo texto, títulos, listas, tabelas, imagens, gráficos, equações, legendas e notas.

A existência de representações intermediárias mais ricas e de representações simplificadas destinadas ao processamento posterior evidencia, de forma prática, a distinção entre representação documental rica e representação linear.

Consequentemente:

> **Markdown ou uma lista de conteúdo não devem ser tratados como sinônimos da representação documental mais rica.**

**Papel potencial:** parsing documental complexo e multimodal, especialmente como componente de uma arquitetura composta.

## 5.5 Adobe PDF Extract

O Adobe PDF Extract demonstrou uma abordagem baseada em serviço externo capaz de produzir elementos estruturados como títulos, parágrafos, listas, tabelas, figuras, notas e referências, além de informações de posição, páginas, estrutura e ordem de leitura.

Sua importância na investigação é também a de **referência externa de capacidade**.

Como serviço externo, introduz questões de custo, disponibilidade, latência, credenciais, privacidade, dependência de fornecedor e reprodutibilidade.

**Papel potencial:** alternativa tecnológica, benchmark de capacidade, possível fallback ou componente especializado. Não se mostrou, neste momento, como fundação exclusiva mais coerente para o Snoopy.

## 5.6 Google Document AI

O Google Document AI apresentou capacidades relevantes de estrutura documental, incluindo texto, âncoras textuais, layout, páginas, blocos, parágrafos, linhas, tokens, tabelas, elementos visuais, posição e confiança.

A estrutura de âncoras demonstra que:

> **texto e estrutura podem ser representações relacionadas sem serem a mesma coisa.**

Como serviço externo, apresenta questões semelhantes às de outras APIs comerciais.

**Papel potencial:** alternativa tecnológica e referência de capacidade, sem determinar o modelo documental interno do Snoopy.

---

# 6. Não existe uma única dimensão de fidelidade

A fidelidade documental não pode ser reduzida a uma única métrica. Devem ser distinguidas pelo menos:

| Dimensão | Pergunta |
|---|---|
| Fidelidade textual | O conteúdo textual foi preservado corretamente? |
| Fidelidade estrutural | A estrutura documental foi preservada? |
| Fidelidade espacial | É possível saber onde o elemento estava? |
| Fidelidade multimodal | Imagens, gráficos, fórmulas e outros elementos continuam acessíveis? |
| Fidelidade relacional | É possível saber quais elementos pertencem uns aos outros? |
| Fidelidade temporal | É possível identificar qual representação/processamento foi utilizado? |
| Rastreabilidade | É possível voltar ao documento original? |
| Reprocessabilidade | É possível gerar novas derivações sem reinterpretar novamente o PDF? |

Assim, “qual é o melhor parser?” é uma pergunta inadequada para o Snoopy.

A pergunta correta é:

> **Qual combinação de capacidades satisfaz o contrato necessário para a representação canônica?**

---

# 7. Modelo abstrato da representação canônica

A comparação das alternativas permitiu identificar uma estrutura abstrata comum:

```text
DOCUMENTO
│
├── identidade / origem
│
├── páginas
│   └── elementos
│       ├── tipo
│       ├── conteúdo
│       ├── posição
│       ├── ordem
│       └── relações
│
├── estrutura documental
│   ├── hierarquia
│   ├── agrupamentos
│   └── ordem de leitura
│
└── recursos
    ├── imagens
    ├── tabelas
    ├── fórmulas
    └── outros elementos
```

Esse modelo é abstrato e independente de uma ferramenta específica. Docling, MinerU, Adobe e Google Document AI apresentam implementações diferentes de partes dessa ideia; PyMuPDF e pdfplumber fornecem informação de baixo nível que pode contribuir para sua construção.

---

# 8. O significado de “canônico”

“Canônico” não significa “contém absolutamente tudo que existe no PDF”. Isso transformaria a arquitetura em uma reprodução desnecessariamente complexa do próprio formato PDF.

A representação canônica deve ser entendida como:

> **A representação persistente mais rica que o Snoopy necessita para preservar a identidade, estrutura, conteúdo e proveniência do documento e permitir derivar novamente as representações utilizadas pelas camadas posteriores.**

Uma informação tende a pertencer à representação canônica quando é necessária para:

- recuperar o documento;
- reconstruir sua estrutura;
- localizar evidência;
- interpretar elementos;
- produzir novas derivações;
- ou reprocessar o documento sem retornar necessariamente ao PDF.

Informações sem função para essas operações podem permanecer apenas no arquivo original ou ser descartadas quando não houver necessidade arquitetural de preservá-las.

---

# 9. Fonte, representação canônica, derivados e resultados

A investigação consolidou quatro níveis:

## 9.1 Fonte

O documento original, especialmente o PDF original.

## 9.2 Representação canônica

O estado documental estruturado e persistente construído pelo Snoopy.

## 9.3 Representações derivadas

Representações produzidas para finalidades específicas, como texto linear, Markdown, HTML, texto para busca, representação textual de tabelas e descrição de imagens.

## 9.4 Resultados de processamento

Estados ainda mais derivados, como chunks, unidades de recuperação, embeddings, índices, rankings, respostas e descrições produzidas por modelos.

A cadeia é:

```text
PDF ORIGINAL
     │
     ↓
REPRESENTAÇÃO CANÔNICA
     │
     ├───────────────┐
     ↓               ↓
DERIVADOS       DERIVADOS
TEXTUAIS        ESTRUTURAIS / VISUAIS
     │
     ↓
SEGMENTAÇÃO
     │
     ↓
UNIDADES DE RECUPERAÇÃO
     │
     ↓
EMBEDDINGS / ÍNDICES
     │
     ↓
RECUPERAÇÃO
     │
     ↓
SÍNTESE
```

Portanto:

> **Chunk não é representação canônica.**

> **Embedding não é representação canônica.**

> **Markdown não é representação canônica.**

> **Resposta de LLM não é representação canônica.**

Todos são estados derivados.

---

# 10. Modelo híbrido: observação e interpretação

Foi investigada a questão de a representação canônica conter apenas informação observada ou também interpretações produzidas pelo processamento.

Foram considerados três modelos:

### Modelo A — canônico mínimo

Preserva principalmente página, objetos, texto, coordenadas e imagens.

**Vantagem:** maior separação entre fonte e interpretação.  
**Desvantagem:** parte da estrutura documental precisaria ser reconstruída repetidamente.

### Modelo B — canônico interpretado

Preserva diretamente estruturas como título, seção, parágrafo, tabela e figura.

**Vantagem:** grande utilidade para processamento posterior.  
**Desvantagem:** incorpora interpretações no próprio modelo.

### Modelo C — híbrido

Preserva simultaneamente:

```text
DOCUMENTO
│
├── camada observacional
│
└── camada estrutural interpretada
```

O **modelo híbrido foi consolidado como a hipótese arquitetural mais compatível com os requisitos**, porque permite preservar a informação de origem enquanto registra interpretações necessárias ao processamento.

Isso não significa que toda interpretação seja tratada como verdade documental. A arquitetura deve manter a distinção entre aquilo que foi observado e aquilo que foi reconhecido, inferido ou reconstruído.

---

# 11. Conteúdo original, reconhecido e interpretado

A investigação identificou uma distinção necessária entre:

### Conteúdo original

Aquilo que estava no documento-fonte.

### Conteúdo reconhecido

Aquilo que um mecanismo de parsing ou OCR identificou a partir da fonte.

### Conteúdo interpretado

Aquilo que um modelo ou mecanismo inferiu, classificou ou descreveu.

Exemplo:

```text
FIGURA ORIGINAL
       │
       ├── OCR → texto detectado
       │
       ├── modelo multimodal → descrição
       │
       └── parser → classificação
```

Essas saídas não possuem o mesmo estatuto. A arquitetura não deve tratar texto reconhecido e descrição gerada por IA como se fossem exatamente o mesmo tipo de informação.

---

# 12. Proveniência do processamento

A proveniência não deve registrar apenas:

```text
 elemento → página → documento
```

Quando relevante, deve também ser possível representar:

```text
 elemento
   ↓
 método de obtenção
   ↓
 processador / modelo
   ↓
 versão
   ↓
 configuração
```

Exemplo:

```text
texto X
 ├── origem: página 7
 ├── método: OCR
 ├── ferramenta/modelo: identificado
 ├── versão: identificada
 └── confiança: quando disponível
```

A proveniência fornecida por uma ferramenta é uma **âncora de origem**, não a totalidade da proveniência do Snoopy.

O Snoopy deverá acrescentar suas próprias identidades e relações, incluindo documento, versão documental, representação, elemento, derivação, processamento e histórico.

---

# 13. Posição, ordem e estrutura

A representação canônica não deve confundir:

### Posição

> Onde o elemento está?

### Ordem

> Em que sequência ele deve ser lido?

### Estrutura

> A que elemento, seção ou agrupamento ele pertence?

Em documentos de múltiplas colunas, posição espacial não determina necessariamente a ordem de leitura. Da mesma forma, uma figura pode estar fisicamente entre dois parágrafos sem que isso esgote sua relação estrutural com a seção e sua legenda.

Consequentemente, essas dimensões devem ser preservadas separadamente.

---

# 14. Páginas como entidades explícitas

Páginas não devem ser tratadas apenas como números incidentais associados a trechos textuais.

Uma representação robusta deve permitir conceitualmente:

```text
documento
└── páginas
    ├── página 1
    │   ├── dimensões
    │   ├── orientação
    │   └── elementos
    ├── página 2
    └── ...
```

Isso prepara a arquitetura para páginas com orientações diferentes, elementos que atravessam páginas, tabelas continuadas, relações entre elementos e visualização posterior.

---

# 15. Tabelas

As tabelas forneceram uma das evidências mais fortes contra a ideia de texto como representação canônica.

Uma tabela pode possuir simultaneamente:

```text
estrutura tabular
```

e:

```text
representação textual
```

A representação textual pode ser útil para recuperação, mas a preservação deve manter sua estrutura:

```text
tabela
 ├── linha
 │   ├── célula
 │   ├── célula
 │   └── célula
 └── ...
```

Células mescladas e relações entre linhas e colunas podem ser perdidas quando uma tabela é simplesmente achatada em texto ou Markdown.

Portanto:

> **A estrutura tabular deve sobreviver na representação canônica; qualquer representação textual da tabela deve ser derivada dela.**

---

# 16. Figuras, gráficos e outros recursos visuais

O mesmo princípio se aplica a figuras e gráficos.

Uma figura não deve ser reduzida simplesmente a `[FIGURA]`.

Uma representação conceitual mais adequada é:

```text
Figura
 ├── referência ao recurso visual original
 ├── localização
 ├── legenda
 ├── relações estruturais
 └── representações derivadas
      ├── descrição
      └── texto associado
```

A imagem original pode ser preservada ou referenciada separadamente do modelo estrutural. Assim, uma descrição textual pode ser usada para recuperação sem destruir o recurso visual que originou a descrição.

---

# 17. Recursos binários e estado documental

A representação canônica não precisa ser um único arquivo ou um único objeto físico.

É possível conceitualmente ter:

```text
DOCUMENTO
│
├── metadados
├── estrutura
├── elementos
├── texto
├── proveniência
└── assets
    ├── imagens
    ├── páginas renderizadas
    └── outros recursos
```

Esses componentes podem constituir conjuntamente um único **estado documental canônico**, ainda que sua persistência física seja distribuída.

A forma física de persistência — JSON, tabelas, objetos, arquivos ou combinação — permanece fora da decisão desta etapa e será investigada na 6.2.4.

---

# 18. Reprocessabilidade como critério central

A propriedade mais forte que emergiu da investigação foi a **reprocessabilidade**.

A pergunta operacional é:

> **Se amanhã mudarmos uma etapa posterior, conseguimos produzir uma nova derivação sem reinterpretar novamente o PDF?**

Por exemplo:

```text
representação canônica
        ↓
novo segmentador
        ↓
novas unidades
        ↓
novos embeddings
```

em vez de:

```text
PDF
 ↓
nova extração
 ↓
nova interpretação
 ↓
novo chunking
```

A representação canônica deve ser suficientemente rica para permitir múltiplas derivações posteriores.

Isso não significa que toda operação futura deverá obrigatoriamente funcionar sem acesso ao PDF. Uma nova análise visual específica pode exigir o recurso visual original. Por isso, recursos originais devem permanecer acessíveis quando forem necessários às capacidades futuras.

O critério correto é:

> **não exigir nova interpretação destrutiva quando a informação necessária já poderia ter sido preservada na representação canônica ou nos recursos associados.**

---

# 19. Arquitetura composta

Foram consideradas quatro alternativas arquiteturais:

## A — parser nativo como núcleo

```text
PDF
 ↓
acesso nativo
 ↓
modelo próprio
```

Maior controle, mas grande responsabilidade de reconstrução estrutural.

## B — parser documental como núcleo

```text
PDF
 ↓
parser documental
 ↓
modelo canônico
```

Maior capacidade pronta, mas maior dependência da interpretação de uma ferramenta específica.

## C — composição

```text
                         ┌→ acesso nativo
                         │
PDF → processamento ─────┼→ interpretação estrutural
                         │
                         ├→ OCR quando necessário
                         │
                         ├→ análise de tabelas
                         │
                         ├→ análise de fórmulas
                         │
                         └→ outros componentes especializados
                                  ↓
                         modelo documental Snoopy
                                  ↓
                          representações derivadas
                                  ↓
                         segmentação / recuperação
```

## D — serviço externo

```text
PDF
 ↓
API especializada
 ↓
modelo canônico
```

A alternativa **C — composição — foi consolidada como a alternativa arquitetural conceitualmente mais coerente até este ponto da investigação**, porque o documento pode combinar diferentes tipos de informação e nenhum componente precisa ser igualmente especializado em todas elas.

A razão não é buscar sofisticação por si mesma. É permitir que cada componente responda por uma capacidade específica enquanto o Snoopy mantém controle sobre integração, identidade, representação, proveniência e derivação.

---

# 20. Limitações e custos da composição

A preferência pela composição não elimina seus custos.

## Complexidade

Cada componente possui versão, dependências, bugs, formatos, limites e comportamento próprio.

## Conflitos

Diferentes componentes podem produzir interpretações diferentes. A arquitetura precisará registrar, comparar e resolver essas situações sem apagar silenciosamente a origem de cada resultado.

## Custo computacional

Mais componentes podem significar maior consumo de CPU e RAM, maior tempo de processamento, eventual necessidade de GPU e eventual uso de APIs pagas.

## Reprodutibilidade

Quando vários componentes participam da construção de uma representação, a reconstrução exige preservar versões, configurações e documento original.

Portanto:

> **Composição aumenta a capacidade potencial, mas também aumenta a superfície de controle necessária.**

Esse custo será aprofundado nas etapas de operação, persistência, versionamento e validação.

---

# 21. O que a investigação eliminou

As seguintes alternativas foram consideradas inadequadas como representação canônica:

### PDF → Markdown

Markdown continua útil, mas é representação textual derivada e pode perder geometria, relações espaciais, detalhes de tabelas e outros elementos.

### PDF → string

É ainda mais claramente insuficiente e corresponde ao problema central do modelo atual.

### Chunk como documento

Chunk é unidade derivada para recuperação/uso posterior, não representação documental.

### Embedding como documento

Embedding é representação matemática específica para determinada operação e não preserva a estrutura documental necessária.

### Saída de LLM como documento

Uma saída de LLM é interpretação ou síntese derivada e não pode substituir a fonte documental.

---

# 22. Consequências arquiteturais consolidadas

A Etapa 6.2.1 consolidou as seguintes consequências para a V3:

1. O PDF original permanece como fonte primária.
2. O Snoopy não deve tratar texto linear ou Markdown como representação canônica.
3. A representação canônica deve ser estrutural.
4. Elementos devem existir como entidades representáveis, e não apenas como partes de uma string.
5. Páginas devem possuir identidade e propriedades próprias.
6. Posição espacial deve ser preservável.
7. Ordem de leitura deve ser representável separadamente da posição.
8. Estrutura hierárquica e relações entre elementos devem ser representáveis.
9. Tabelas devem possuir representação estrutural própria.
10. Figuras, gráficos, fórmulas e outros elementos complexos devem permanecer acessíveis.
11. Representações textuais devem ser derivadas da representação mais rica.
12. Recursos binários originais podem ser preservados ou referenciados separadamente.
13. Proveniência espacial deve poder chegar, quando disponível, a granularidade de elemento/região/intervalo.
14. Proveniência de localização fornecida por ferramentas deve ser complementada pela proveniência própria do Snoopy.
15. O sistema deve distinguir fonte original, conteúdo reconhecido e conteúdo interpretado.
16. Quando disponível, informação de confiança pode ser preservada como metadado de processamento, sem ser confundida com verdade documental.
17. A representação canônica deve ser desacoplada do formato interno de qualquer ferramenta específica.
18. Diferentes componentes podem contribuir para sua construção.
19. A alternativa arquitetural composta C é, neste ponto, a mais coerente com os requisitos.
20. A principal propriedade a validar posteriormente é a capacidade de produzir novas derivações sem nova interpretação destrutiva do documento.

---

# 23. O que ainda não foi decidido

A Etapa 6.2.1 **não** escolhe definitivamente:

- Docling;
- MinerU;
- PyMuPDF;
- pdfplumber;
- Adobe;
- Google Document AI;
- OCR específico;
- qualquer combinação específica desses componentes;
- formato físico da representação;
- banco de dados;
- object/document store;
- JSON como formato canônico;
- qualquer esquema de persistência.

Também permanecem em aberto:

- o modelo final de dados do Snoopy;
- quais propriedades serão obrigatoriamente persistidas;
- quais informações poderão ser reconstruídas sob demanda;
- como conflitos entre processadores serão resolvidos;
- como confiança será formalmente modelada;
- quais componentes serão usados em cada tipo de documento;
- quais componentes serão ativados por fallback;
- quais capacidades serão executadas sempre ou apenas quando necessárias.

Essas decisões dependem da investigação das etapas seguintes e, principalmente, do projeto técnico da 6.3.

---

# 24. Relação com as etapas seguintes

## 6.2.2 — OCR e documentos digitalizados

Deverá investigar quando OCR é necessário, como distinguir texto nativo e reconhecido, precisão e limitações, localização, confiança, preservação da imagem original e relação entre OCR e representação canônica.

## 6.2.3 — Estrutura, layout, tabelas e elementos multimodais

Deverá aprofundar múltiplas colunas, ordem de leitura, hierarquia, tabelas, células mescladas, gráficos, figuras, fórmulas, legendas, notas, relações espaciais e recursos visuais.

## 6.2.4 — Persistência da representação

Deverá investigar como armazenar o estado documental canônico, seus elementos, relações, recursos e proveniência sem transformar a estrutura física de persistência em restrição conceitual.

## 6.2.5 — Segmentação

Partirá da representação preservada para definir como construir unidades de recuperação sem confundi-las com a estrutura documental ou com unidades metodológicas.

---

# 25. Critério de encerramento da 6.2.1

A investigação foi considerada suficientemente madura quando deixou de acrescentar novas propriedades essenciais ao modelo e passou a demonstrar principalmente diferentes implementações das mesmas capacidades.

O critério utilizado foi:

> **Se uma nova rodada não acrescentar uma capacidade, uma restrição ou uma decisão necessária ao modelo, não há necessidade de prolongá-la.**

A comparação tecnológica mostrou que diferentes abordagens já cobrem, em maior ou menor grau, as dimensões necessárias e que a questão central deixou de ser descobrir “o parser definitivo”.

A 6.2.1 pode, portanto, ser encerrada sem afirmar que todos os detalhes tecnológicos foram resolvidos.

---

# 26. Princípio central da Etapa 6.2.1

> **A representação canônica do Snoopy deve funcionar como uma camada documental estável entre a fonte original e as representações derivadas do sistema, preservando identidade, páginas, conteúdo, estrutura, posição, ordem, relações, recursos e proveniência em grau suficiente para permitir múltiplas derivações posteriores sem nova interpretação destrutiva do documento.**

“Estável” não significa imutável. Significa que mudanças em chunking, embeddings, busca, interface e estratégia de síntese não devem exigir nova interpretação do PDF quando a informação necessária já estiver preservada na representação canônica ou em seus recursos associados.

---

# 27. Princípio de encerramento

A principal conclusão da Etapa 6.2.1 pode ser sintetizada assim:

> **O Snoopy não deve escolher uma ferramenta e fazer dela o documento. Deve preservar o documento por meio de uma representação canônica própria e utilizar ferramentas diferentes, quando necessário, para fornecer as capacidades de acesso, interpretação e reconhecimento que alimentam essa representação.**

A investigação fortaleceu a alternativa arquitetural **C — composição**, mas a escolha concreta dos componentes permanece para as etapas posteriores.

A pergunta “qual parser usar?” deixa de ser a pergunta central.

A pergunta arquitetural passa a ser:

> **“Qual contrato a representação canônica do Snoopy estabelece com o restante do sistema?”**

A resposta consolidada nesta etapa é que esse contrato deve permitir:

```text
DOCUMENTO ORIGINAL
        ↓
PRESERVAÇÃO
        ↓
REPRESENTAÇÃO CANÔNICA
        ↓
REPRESENTAÇÕES DERIVADAS
        ↓
SEGMENTAÇÃO
        ↓
RECUPERAÇÃO
        ↓
SÍNTESE
```

mantendo, ao longo dessas transformações, a possibilidade de retornar ao documento original e reconstruir a cadeia de derivação.

---

## Estado da Etapa

**6.2.1 — Representação Canônica e Processamento de Documentos: CONSOLIDADA.**

**Próxima subetapa:** 6.2.2 — OCR e documentos digitalizados.

**Decisão tecnológica definitiva:** ainda não realizada.

**Alternativa arquitetural mais forte no estado atual da investigação:** C — arquitetura composta.

**Princípio preservado:** requisito → critério → alternativa → comparação → escolha.
