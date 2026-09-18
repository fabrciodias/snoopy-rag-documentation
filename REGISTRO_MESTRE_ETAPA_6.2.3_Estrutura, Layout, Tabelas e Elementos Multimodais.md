# Registro Mestre da Análise — Snoopy-RAG
## Etapa 6.2.3 — Estrutura, Layout, Tabelas e Elementos Multimodais

**Status:** Registro Mestre da Etapa 6.2.3 — consolidado  
**Natureza:** investigação de alternativas tecnológicas; não constitui ainda o projeto técnico definitivo nem a implementação.  
**Relação com as etapas anteriores:** a Etapa 5 estabelece o que o Snoopy precisa ser capaz de fazer; a Etapa 6.1 traduz esses requisitos gerais em critérios técnicos; a Etapa 6.2 investiga alternativas tecnológicas capazes de atender a esses critérios; a Etapa 6.3 realizará a escolha e o projeto da composição tecnológica concreta.

---

## 1. Finalidade da Etapa 6.2.3

A Etapa 6.2.3 investiga alternativas tecnológicas para tratar estrutura documental, layout, ordem de leitura, tabelas, figuras, gráficos, fórmulas e demais elementos multimodais relevantes ao corpus do Snoopy-RAG.

A discussão das necessidades específicas é necessária para orientar a investigação, mas não constitui o objetivo final desta etapa. O objetivo principal é identificar tecnologias, comparar suas capacidades e limitações e determinar seus possíveis papéis dentro de uma arquitetura composta.

A escolha concreta da arquitetura permanece reservada à Etapa 6.3.

---

## 2. Necessidades específicas que orientam a investigação

A representação documental do Snoopy precisa conservar, tanto quanto possível, as características estruturais relevantes do documento original.

Entre as propriedades relevantes estão:

- posição espacial dos elementos;
- ordem de leitura;
- hierarquia documental;
- tipo de elemento;
- relações entre elementos;
- tabelas e sua estrutura interna;
- figuras, gráficos e imagens;
- fórmulas e elementos científicos;
- cabeçalhos, rodapés, notas, legendas, referências e outros elementos auxiliares.

Essas propriedades não são equivalentes entre si. Posição espacial não é ordem de leitura; ordem de leitura não é hierarquia; tipo de elemento não é relação estrutural.

A arquitetura precisa, portanto, permitir que essas dimensões sejam representadas de maneira distinguível.

---

## 3. Observação documental e interpretação estrutural

A investigação consolidou a necessidade de distinguir duas classes de informação.

### 3.1 Observação

Corresponde a informações obtidas diretamente ou próximas da estrutura física do documento, como:

- páginas;
- coordenadas;
- bounding boxes;
- dimensões;
- texto;
- palavras;
- blocos;
- imagens;
- linhas;
- objetos;
- características geométricas.

### 3.2 Interpretação

Corresponde às propriedades inferidas a partir dessas observações, como:

- identificação de uma tabela;
- identificação de uma legenda;
- definição de uma seção;
- determinação de colunas;
- ordem de leitura;
- hierarquia documental;
- relação entre figura e legenda;
- continuidade lógica de uma tabela entre páginas.

Uma interpretação pode estar incorreta sem que a observação original esteja incorreta.

Consequentemente, a representação do Snoopy não deve depender exclusivamente de uma interpretação produzida por uma ferramenta. As observações relevantes devem permanecer disponíveis para revisão, reprocessamento e comparação.

---

## 4. Representação canônica

Nenhuma das tecnologias investigadas deve ser adotada diretamente como representação canônica do Snoopy.

Ferramentas como Docling, MinerU, PaddleOCR ou serviços externos produzem representações próprias. Essas representações podem ser utilizadas como fontes de observações e interpretações, mas o Snoopy deve possuir uma representação lógica própria.

A representação canônica deve permitir relacionar, conceitualmente:

```text
Documento
├── identidade / origem
├── páginas
│   └── elementos
│       ├── conteúdo
│       ├── tipo
│       ├── posição
│       ├── dimensão
│       ├── ordem
│       └── proveniência
├── estrutura documental
│   ├── hierarquia
│   ├── agrupamentos
│   └── ordem de leitura
└── relações
    ├── referências
    ├── legendas
    ├── continuidade
    └── outras relações documentais
```

Essa representação não precisa ser uma reprodução visual do PDF. Seu objetivo é preservar informações documentais relevantes de forma estruturada e reprocessável.

---

## 5. Tabelas

Tabelas devem ser tratadas como entidades estruturais próprias, e não apenas como sequências de strings.

Quando possível, a representação deve conservar:

- identidade da tabela;
- localização;
- fragmentos físicos;
- células;
- posição das células;
- linhas;
- colunas;
- cabeçalhos;
- relações entre cabeçalhos e dados;
- células mescladas;
- estruturas hierárquicas;
- continuidade entre páginas;
- conteúdo;
- proveniência.

Uma tabela multipágina pode ser representada conceitualmente como uma entidade lógica composta por múltiplos fragmentos físicos.

A existência dos fragmentos é observação documental. A conclusão de que os fragmentos constituem uma única tabela lógica é interpretação estrutural.

Representações como matriz de strings, Markdown ou HTML podem ser derivadas da tabela canônica, mas não devem ser a única forma de preservação.

---

## 6. Geometria

A geometria constitui uma camada observacional importante.

Devem poder ser preservadas informações espaciais em diferentes níveis, por exemplo:

```text
Página
└── Tabela
    └── Fragmento
        └── Célula
```

A geometria pode cumprir funções de:

1. preservação documental;
2. reconstrução estrutural;
3. validação;
4. reprocessamento;
5. visualização;
6. análise de conflitos entre interpretações.

Entretanto, geometria não é equivalente a estrutura semântica.

Uma coordenada ou bounding box não determina, por si só, que um elemento seja uma coluna, cabeçalho, legenda, seção ou outro tipo estrutural.

---

## 7. Ordem de leitura e layout

A ordem de leitura deve ser considerada uma interpretação estrutural e não uma verdade absoluta.

Documentos com múltiplas colunas ou layouts complexos podem produzir diferentes interpretações de sequência.

Por isso, a representação deve conservar separadamente:

- posição espacial;
- observações documentais;
- ordem de leitura produzida;
- informações estruturais que permitam revisar essa ordem.

A preservação da posição permite que uma interpretação de ordem posteriormente considerada inadequada seja corrigida sem necessariamente repetir toda a extração do documento original.

---

## 8. Elementos multimodais

Figuras, gráficos, imagens e fórmulas devem ser tratados como classes próprias de elementos documentais quando identificáveis.

A distinção fundamental é:

**recurso original ≠ interpretação do recurso.**

Uma figura pode possuir:

- localização;
- recurso visual original;
- legenda;
- relação com o texto;
- interpretação estrutural;
- eventuais representações derivadas.

Da mesma maneira, uma fórmula pode conservar sua representação visual original e, quando possível, possuir uma representação matemática derivada.

O objetivo é representar o elemento documental para fins de recuperação, contextualização e rastreabilidade, e não produzir automaticamente uma interpretação científica do seu significado.

---

## 9. Degradação graciosa

A insuficiência de uma interpretação estrutural não deve resultar na perda do documento.

Quando uma tabela, figura, fórmula ou outro elemento não puder ser completamente interpretado, o sistema deve preservar aquilo que consegue observar e limitar apenas os derivados que dependam da interpretação ausente ou incerta.

Isso conduz ao princípio:

> Uma interpretação insuficiente deve limitar o derivado dependente dela, não destruir ou alterar silenciosamente a representação documental original.

Assim, uma tabela cuja estrutura não pôde ser reconstruída com segurança não precisa tornar o documento inteiro inválido. Da mesma forma, uma interpretação estrutural incerta não deve ser apresentada ao mecanismo de recuperação como se fosse uma certeza documental.

---

# 10. Alternativas tecnológicas investigadas

## 10.1 PyMuPDF

O PyMuPDF trabalha próximo da estrutura física do PDF e fornece acesso a texto, blocos, palavras, imagens, coordenadas e informações detalhadas de posicionamento.

**Força principal:** observação de baixo nível e proximidade com a fonte.

**Limitação principal:** não resolve sozinho a interpretação documental de alto nível.

**Papel potencial:** camada de observação, extração e geometria.

---

## 10.2 pdfplumber

O pdfplumber é particularmente relevante para análise espacial e processamento de tabelas, oferecendo acesso a linhas, interseções, palavras, bounding boxes, células e estratégias de detecção de tabelas.

**Força principal:** análise geométrica e reconstrução especializada de tabelas.

**Limitação principal:** não constitui um parser documental completo e não resolve sozinho a interpretação estrutural e semântica.

**Papel potencial:** componente especializado de geometria e tabelas.

---

## 10.3 Docling

O Docling opera em nível mais alto de processamento documental e oferece representação estruturada de textos, tabelas, figuras, grupos e outros elementos, além de hierarquia, body/furniture, bounding boxes, proveniência e ordem dos elementos.

Também integra capacidades de OCR, layout, tabelas, fórmulas e processamento de imagens.

**Força principal:** processamento documental estruturado e integração de múltiplas capacidades.

**Limitação principal:** sua representação é própria da ferramenta e não deve ser confundida com o modelo canônico do Snoopy. Suas inferências estruturais também precisam ser tratadas como interpretações sujeitas a erro.

**Papel potencial:** componente de processamento e interpretação estrutural.

---

## 10.4 MinerU

O MinerU oferece parsing documental amplo, incluindo texto, títulos, parágrafos, listas, tabelas, imagens, gráficos, fórmulas, cabeçalhos, rodapés, notas, coordenadas, estrutura intermediária e ordem de leitura.

A existência de uma representação intermediária mais rica e de representações derivadas simplificadas é particularmente compatível com a arquitetura desejada.

**Força principal:** amplitude de processamento documental e estrutura intermediária rica.

**Limitação principal:** custo e complexidade operacional variáveis conforme o backend, além da necessidade de avaliar suas condições de licença e adequação ao modelo do Snoopy.

**Papel potencial:** parsing documental estruturado e produção de representações derivadas.

---

## 10.5 PaddleOCR / PP-StructureV3

O PaddleOCR, especialmente por meio do PP-StructureV3, oferece capacidades de análise de layout, OCR, reconhecimento de tabelas, fórmulas e leitura de documentos complexos, incluindo múltiplas colunas e diferentes categorias de elementos.

**Força principal:** análise visual/documental modular e abrangente.

**Limitação principal:** quanto maior a dependência de interpretação visual/modelada, maior a necessidade de preservar separadamente as observações originais.

**Papel potencial:** análise visual, layout, OCR e reconhecimento especializado.

---

## 10.6 Surya

O Surya oferece OCR, detecção de linhas, análise de layout, ordem de leitura e reconhecimento de elementos documentais.

**Força principal:** capacidade moderna de análise visual/documental.

**Limitações principais:** maior custo computacional e necessidade de controle sobre caminhos específicos de processamento. A investigação também encontrou casos documentados em que determinadas classificações de layout podem resultar na omissão de conteúdo textual dentro de diagramas em determinados fluxos.

**Papel potencial:** componente especializado de OCR/layout ou fallback.

---

## 10.7 Adobe PDF Extract

O Adobe PDF Extract oferece processamento estruturado de alto nível, com elementos como headings, parágrafos, listas, figuras, tabelas, células, seções, localização e ordem.

**Força principal:** capacidade ampla de extração documental estruturada.

**Limitações principais:**

- serviço externo;
- custo;
- dependência de rede;
- credenciais;
- dependência do fornecedor;
- política de envio dos documentos;
- mudanças futuras no serviço;
- menor controle sobre o ambiente de processamento.

**Papel potencial:** serviço especializado, fallback, comparação ou processamento de casos específicos.

---

## 10.8 Google Document AI

O Google Document AI oferece OCR, layout, blocos, parágrafos, linhas, tokens, tabelas, polígonos, confiança e estrutura documental.

**Força principal:** processamento documental externo abrangente.

**Limitações principais:** custo, dependência externa, menor controle sobre o ambiente e dependência das versões e comportamento do serviço.

**Papel potencial:** alternativa especializada ou fallback externo.

---

# 11. Comparação consolidada

| Tecnologia | Capacidade principal | Força | Limitação | Papel potencial |
|---|---|---|---|---|
| PyMuPDF | Texto, objetos, posição, geometria | Proximidade com o PDF | Interpretação estrutural limitada | Observação |
| pdfplumber | Geometria e tabelas | Análise espacial | Não é parser completo | Especialista em geometria/tabelas |
| Docling | Estrutura documental | Representação estruturada e integração | Não é o canônico do Snoopy | Processamento estrutural |
| MinerU | Parsing amplo | Estrutura intermediária rica | Custo/complexidade variável | Parsing documental |
| PaddleOCR | Layout/OCR/tabelas/fórmulas | Modularidade e análise visual | Dependência crescente de interpretação | Análise visual |
| Surya | OCR/layout/leitura | Capacidade visual moderna | Peso computacional e casos específicos | Especialista/fallback |
| Adobe PDF Extract | Estrutura documental | Alto nível e amplitude | Serviço externo | Especialista/fallback |
| Google Document AI | OCR/layout/tabelas | Amplitude documental | Serviço externo/custo | Especialista/fallback |

---

# 12. Resultado da comparação

A investigação não identifica um único “melhor parser” para o problema do Snoopy.

As alternativas ocupam níveis diferentes do processamento documental:

- PyMuPDF e pdfplumber são especialmente adequados à observação e análise geométrica;
- Docling e MinerU oferecem processamento documental estruturado;
- PaddleOCR/PP-StructureV3 e Surya oferecem análise visual, layout e reconhecimento especializado;
- Adobe PDF Extract e Google Document AI oferecem processamento documental estruturado por serviços externos;
- ferramentas especializadas podem complementar funções específicas, especialmente em elementos como fórmulas.

As diferenças entre essas alternativas fornecem fundamento tecnológico para uma arquitetura composta.

---

# 13. Composição tecnológica

A investigação favorece uma arquitetura composta, mas composição não significa utilizar todas as ferramentas.

A composição é justificável quando uma tecnologia fornece uma capacidade que outra não fornece adequadamente.

Um exemplo conceitual seria:

```text
PDF
 │
 ├── observação / extração geométrica
 │        ↓
 │     PyMuPDF
 │
 ├── interpretação documental
 │        ↓
 │     Docling / MinerU
 │
 └── análise especializada
          ↓
      pdfplumber / PaddleOCR / outros
```

Nesse modelo, os componentes possuem funções diferenciadas.

Em contraste, utilizar múltiplas tecnologias para analisar indiscriminadamente o mesmo conteúdo sem uma responsabilidade funcional clara introduziria:

- resultados conflitantes;
- necessidade de arbitragem;
- maior custo;
- maior complexidade;
- maior quantidade de estados e versões;
- maior dificuldade de reprodução.

Portanto:

> **Composição não significa redundância.**

Princípio consolidado:

> **Cada tecnologia incorporada à arquitetura deve possuir uma função documental claramente justificada e uma relação definida com as demais.**

---

# 14. O que a Etapa 6.2.3 não decide

A 6.2.3 não determina ainda:

- qual parser será o principal;
- qual ferramenta será utilizada em cada documento;
- a ordem exata dos componentes;
- as condições de ativação de cada componente;
- os mecanismos de fallback;
- quais tecnologias serão efetivamente instaladas;
- o formato físico definitivo da representação canônica;
- a arbitragem definitiva entre resultados conflitantes;
- os modelos específicos que serão utilizados em cada caminho.

Essas decisões pertencem à Etapa 6.3.

A 6.2.3 fornece a base tecnológica para essas decisões.

---

# 15. Conclusões consolidadas

1. Não foi identificada uma ferramenta única que cubra adequadamente todas as necessidades de estrutura, layout, tabelas e elementos multimodais do Snoopy.

2. As tecnologias investigadas apresentam níveis e especializações diferentes e, portanto, podem desempenhar funções complementares.

3. PyMuPDF e pdfplumber são particularmente relevantes para observação e geometria.

4. Docling e MinerU são alternativas fortes para processamento documental estruturado.

5. PaddleOCR/PP-StructureV3 e Surya são alternativas relevantes para análise visual e reconhecimento especializado.

6. Adobe PDF Extract e Google Document AI são alternativas externas de processamento estruturado, especialmente relevantes como serviços especializados, fallback ou comparação.

7. Tabelas, figuras, gráficos e fórmulas não devem ser reduzidos a texto linear quando sua estrutura for relevante.

8. Geometria deve ser preservada como informação observacional e não confundida com interpretação estrutural.

9. Ordem de leitura e hierarquia são interpretações que podem conter erros e devem poder ser revisadas.

10. A representação canônica deve pertencer logicamente ao Snoopy e não ser simplesmente a saída nativa de um fornecedor.

11. Representações como Markdown, HTML, matrizes e demais formatos voltados à recuperação ou visualização devem ser tratadas como derivados.

12. A insuficiência de uma interpretação estrutural deve limitar os derivados dependentes dela, sem destruir ou alterar silenciosamente o documento preservado.

13. A direção tecnológica resultante é uma **composição seletiva de tecnologias**, com responsabilidades funcionalmente diferenciadas.

14. A composição deve evitar redundância injustificada e só incorporar uma tecnologia quando sua capacidade representar uma contribuição arquitetural clara.

---

# 16. Princípio central da Etapa 6.2.3

> **A representação estrutural do Snoopy deverá preservar as observações documentais relevantes e permitir a incorporação de interpretações estruturais produzidas por diferentes tecnologias sem confundi-las com a própria realidade documental. A investigação tecnológica favorece uma composição seletiva, na qual componentes especializados possam contribuir para observação, geometria, estrutura, layout, tabelas, figuras, fórmulas e outros elementos, mantendo uma representação canônica própria do Snoopy e permitindo a revisão ou reconstrução dos derivados sem perda da informação original.**

---

# 17. Critério de encerramento da 6.2.3

A investigação é considerada suficientemente estabelecida porque:

- as necessidades específicas relevantes foram identificadas;
- as principais alternativas tecnológicas foram investigadas;
- suas capacidades e limitações relevantes foram comparadas;
- seus papéis potenciais foram estabelecidos;
- a composição tecnológica foi identificada como direção viável;
- não há necessidade de selecionar a arquitetura concreta nesta etapa.

A escolha de **quais componentes utilizar, em qual ordem, sob quais condições e com quais mecanismos de fallback** será realizada na Etapa 6.3.

---

## Encerramento

A Etapa 6.2.3 está, portanto, **CONSOLIDADA**.

O resultado tecnológico da investigação não é a escolha de uma ferramenta vencedora, mas a identificação de um espaço de composição no qual diferentes tecnologias podem desempenhar funções distintas.

A próxima subetapa da Etapa 6.2 é:

> **6.2.4 — Persistência da Representação**

Aplicando o mesmo princípio metodológico:

**necessidades específicas → alternativas tecnológicas → comparação → papéis possíveis → conclusão.**

Sem transformar a discussão necessária em uma nova árvore infinita de subetapas.
