# Registro Mestre da Análise --- Snoopy-RAG

## Etapa 6.2.2 --- OCR e Documentos Digitalizados: Investigação de Alternativas Tecnológicas

**Status:** Registro Mestre da Etapa 6.2.2 --- consolidado para
transição à Etapa 6.3.\
**Natureza:** investigação tecnológica; não constitui escolha
definitiva, arquitetura concreta ou implementação.

------------------------------------------------------------------------

## 1. Finalidade

A Etapa 6.2.2 teve como objetivo investigar e caracterizar as
alternativas tecnológicas capazes de fornecer ao Snoopy reconhecimento
de texto e processamento documental quando a informação textual não está
disponível, ou não está disponível de maneira suficientemente adequada,
como texto digital nativo.

A pergunta central foi:

> **Quais tecnologias podem fornecer ao Snoopy as capacidades de
> reconhecimento e reconstrução documental necessárias, quais são seus
> pontos fortes e limitações e que papel cada alternativa poderia
> desempenhar em uma arquitetura futura?**

A etapa não buscou identificar "o melhor OCR". A investigação distinguiu
motores de OCR, ferramentas de percepção documental, pipelines
integrados e serviços externos de Document AI.

A seleção e composição concretas permanecem para a Etapa 6.3.

------------------------------------------------------------------------

## 2. Contexto tecnológico

OCR não é uma categoria tecnológica homogênea. Foram identificadas
quatro classes principais:

1.  **motores de OCR**, voltados principalmente ao reconhecimento de
    caracteres e palavras;
2.  **OCR + percepção documental**, acrescentando layout, localização,
    ordem de leitura e identificação de elementos;
3.  **pipelines de processamento documental**, combinando extração, OCR,
    estrutura, tabelas, fórmulas, imagens e outras capacidades;
4.  **serviços externos de Document AI**, que fornecem capacidades
    semelhantes por infraestrutura de terceiros.

Essa distinção impede que ferramentas com escopos diferentes sejam
tratadas como concorrentes equivalentes.

Também permanece válida a distinção estabelecida nas etapas anteriores:

> **A saída de uma ferramenta não é automaticamente a representação
> canônica do Snoopy.**

O documento original continua sendo a fonte primária. As saídas de OCR,
layout, parsing, tabelas, fórmulas e outros mecanismos são
representações derivadas e devem poder ser vinculadas à origem.

------------------------------------------------------------------------

## 3. Critérios de investigação

As alternativas foram examinadas segundo:

### 3.1 Reconhecimento

-   texto impresso e digitalizado;
-   idiomas;
-   caracteres especiais;
-   números;
-   referências;
-   texto em imagens.

### 3.2 Informação espacial

-   páginas;
-   blocos;
-   linhas;
-   palavras;
-   coordenadas;
-   bounding boxes/polígonos;
-   confiança.

### 3.3 Estrutura

-   títulos;
-   parágrafos;
-   listas;
-   colunas;
-   ordem de leitura;
-   cabeçalhos/rodapés;
-   notas;
-   referências;
-   figuras.

### 3.4 Elementos científicos

-   tabelas;
-   fórmulas;
-   gráficos;
-   legendas;
-   elementos visuais contendo texto.

### 3.5 Granularidade

-   documento;
-   página;
-   região;
-   bloco;
-   elemento específico.

### 3.6 Operação

-   execução local;
-   CPU/GPU;
-   custo computacional;
-   dependência externa;
-   controle;
-   flexibilidade;
-   integração;
-   possibilidade de substituição de componentes.

### 3.7 Governança

-   licença;
-   licença de modelos/pesos;
-   dependência de fornecedor;
-   versionamento;
-   impacto sobre controle e reprodutibilidade.

A comparação é **qualitativa e arquitetural**, não um benchmark de
precisão.

------------------------------------------------------------------------

# 4. Tesseract

## Caracterização

Tesseract é o exemplo mais claro de motor OCR especializado. Pode
produzir texto, PDF pesquisável, hOCR e TSV, incluindo informações
espaciais e de confiança em níveis como página, bloco, parágrafo, linha
e palavra.

## Pontos fortes

-   execução local;
-   baixo acoplamento externo;
-   controle elevado;
-   maturidade;
-   reconhecimento textual;
-   coordenadas;
-   confiança;
-   formatos adequados a processamento posterior;
-   custo operacional relativamente baixo.

## Pontos fracos

Não constitui, sozinho, solução completa para interpretação documental
científica complexa. Não tem como função central reconstruir hierarquia,
tabelas semanticamente estruturadas, fórmulas, figuras ou relações
documentais complexas.

## Papel possível

Candidato a **componente OCR especializado**, especialmente para regiões
específicas, processamento leve, fallback ou tarefas nas quais não seja
necessário um pipeline documental completo.

------------------------------------------------------------------------

# 5. Surya

## Caracterização

Surya ultrapassa o OCR textual e oferece OCR, análise de layout, ordem
de leitura e reconhecimento de tabelas, além de identificação de
diferentes categorias de elementos documentais.

## Pontos fortes

-   OCR;
-   localização;
-   layout;
-   ordem de leitura;
-   tabelas;
-   identificação de elementos;
-   execução local;
-   possibilidade de processamento por página/região.

É particularmente interessante para documentos em múltiplas colunas e
situações nas quais a posição dos elementos interfere na reconstrução.

## Pontos fracos

É significativamente mais pesado que um OCR textual tradicional. A
versão atual utiliza modelos grandes e pode exigir recursos
computacionais relevantes; embora existam caminhos em CPU, seu
desempenho pode ser muito inferior ao obtido em GPU.

Também não se deve tratar sua classificação como descrição infalível do
documento: modelos de percepção podem classificar ou omitir conteúdo
inadequadamente.

## Papel possível

Candidato a **OCR + percepção visual/documental**, especialmente para
layout, ordem de leitura, regiões e casos complexos.

------------------------------------------------------------------------

# 6. PaddleOCR / PP-Structure

## Caracterização

PaddleOCR constitui uma stack modular que combina OCR com análise de
layout, tabelas, fórmulas, gráficos, ordem de leitura e parsing
documental.

## Pontos fortes

-   ampla cobertura;
-   OCR;
-   layout;
-   tabelas;
-   fórmulas;
-   gráficos;
-   ordem de leitura;
-   resultados estruturados;
-   execução local;
-   CPU e aceleração por hardware;
-   modularidade.

## Pontos fracos

A amplitude aumenta a complexidade. Diferentes módulos possuem custos e
possibilidades de erro distintos. Utilizar a stack inteira
indiscriminadamente não é necessariamente desejável.

Como nas demais alternativas, sua saída não deve substituir a
representação própria do Snoopy.

## Papel possível

Candidato a **pipeline local modular de OCR e parsing documental**,
podendo fornecer diferentes capacidades conforme o caso.

------------------------------------------------------------------------

# 7. Docling

## Caracterização

Docling representa uma camada mais ampla de processamento documental.
Sua `DoclingDocument` representa textos, tabelas, imagens, grupos,
hierarquia, páginas, localização e proveniência. O OCR é um componente
configurável e pode utilizar diferentes engines.

## Pontos fortes

-   representação documental estruturada;
-   hierarquia;
-   layout;
-   proveniência;
-   tabelas;
-   imagens;
-   fórmulas;
-   OCR intercambiável;
-   execução local;
-   CPU ou aceleradores;
-   configuração de componentes;
-   possibilidade de operação offline.

Uma característica particularmente relevante é o desacoplamento entre
processamento documental e um único engine de OCR.

## Pontos fracos

`DoclingDocument` não deve ser adotado automaticamente como Canonical do
Snoopy: continua sendo uma representação produzida por uma ferramenta.

Pipelines mais completas também podem possuir custo computacional
relevante.

## Papel possível

Candidato especialmente forte a **camada de processamento/orquestração
documental e representação intermediária estruturada**, além de
integração de diferentes engines.

------------------------------------------------------------------------

# 8. MinerU

## Caracterização

MinerU é uma alternativa de parsing documental abrangente,
particularmente relevante para documentos complexos e científicos. Sua
saída contempla texto, títulos, listas, tabelas, imagens, gráficos,
fórmulas, cabeçalhos, rodapés, notas e referências, com localização e
ordem de leitura.

Também possui mecanismos para documentos escaneados e OCR.

## Pontos fortes

-   ampla cobertura documental;
-   OCR;
-   layout;
-   ordem de leitura;
-   tabelas;
-   fórmulas;
-   imagens;
-   gráficos;
-   documentos científicos;
-   PDFs escaneados;
-   execução local;
-   diferentes ambientes de hardware;
-   inspeção dos resultados.

## Pontos fracos

A abrangência aumenta a complexidade e o custo computacional. Algumas
modalidades possuem requisitos relevantes de memória e processamento.

A licença também exige análise específica antes de adoção definitiva,
inclusive em relação a componentes e modelos.

## Papel possível

Candidato a **pipeline integrado de parsing documental científico**,
especialmente para documentos complexos e escaneados.

------------------------------------------------------------------------

# 9. Serviços externos de Document AI

Incluem, entre outros, Google Document AI, Azure Document Intelligence,
Amazon Textract e Adobe PDF Extract.

## Capacidades

Esses serviços podem fornecer combinações de:

-   OCR;
-   layout;
-   coordenadas;
-   confiança;
-   tabelas;
-   estrutura documental;
-   elementos de página;
-   ordem de leitura, conforme o serviço.

## Pontos fortes

-   infraestrutura pronta;
-   capacidade documental elevada;
-   escalabilidade;
-   ausência de necessidade de manter localmente todos os modelos;
-   possibilidade de uso especializado.

## Pontos fracos

Introduzem:

-   internet;
-   credenciais;
-   custo por processamento;
-   limites de uso;
-   latência;
-   dependência de fornecedor;
-   políticas de dados;
-   mudanças de modelos/serviços;
-   menor controle sobre o ambiente.

Uma mudança do serviço pode alterar resultados futuros. Por isso,
versão/configuração do serviço deverá fazer parte da proveniência caso
seja utilizado.

## Papel possível

Podem atuar como **fallback, componente especializado, referência
comparativa ou alternativa de infraestrutura**, dependendo das condições
futuras.

------------------------------------------------------------------------

# 10. Comparação por tipo de documento

## 10.1 PDF completamente digitalizado

-   **Tesseract:** forte para reconhecimento textual, limitado para
    reconstrução complexa.
-   **Surya:** acrescenta layout e ordem de leitura.
-   **PaddleOCR:** combina OCR e módulos estruturais.
-   **Docling:** oferece pipeline documental estruturada e OCR
    intercambiável.
-   **MinerU:** oferece parsing abrangente, incluindo OCR.
-   **Serviços externos:** oferecem OCR/estrutura sem infraestrutura
    local.

## 10.2 Múltiplas colunas

O problema passa a ser também reconstrução da ordem de leitura.

Surya, PaddleOCR, Docling e MinerU possuem capacidades explícitas
relacionadas a layout e ordem de leitura. Tesseract fornece informações
de segmentação e posição, mas não deve ser tratado como equivalente a
interpretação documental completa.

## 10.3 Tabelas

Tesseract fornece texto e posição, podendo servir de matéria-prima para
outra etapa.

Surya, PaddleOCR, Docling e MinerU possuem capacidades específicas para
tabelas.

Para o Snoopy, a representação final não deve ser apenas texto ou
Markdown: a estrutura da tabela deve poder existir como elemento
derivado, vinculado à origem.

## 10.4 Fórmulas

PaddleOCR, Docling e MinerU possuem capacidades específicas para
fórmulas; Surya também possui identificação de regiões de equação.

Fórmulas não devem ser reduzidas simplesmente a uma sequência de
caracteres OCR. A representação visual/original deve permanecer
preservada, podendo coexistir com uma representação matemática derivada.

## 10.5 Figura contendo texto

Tesseract pode ser útil quando a região textual já foi identificada.

Surya, PaddleOCR, Docling e MinerU possuem capacidades de identificação
de elementos visuais e localização, tornando plausível uma futura
estratégia de OCR localizado.

## 10.6 Documento híbrido

Este é um caso central para o Snoopy: texto nativo, imagens, tabelas e
regiões escaneadas podem coexistir.

Docling e MinerU são especialmente interessantes por trabalharem com
diferentes formas de processamento documental. A composição concreta,
entretanto, permanece para a Etapa 6.3.

------------------------------------------------------------------------

# 11. Comparação transversal

  -----------------------------------------------------------------------------------------------------
  Alternativa      OCR       Layout     Ordem de   Tabelas   Fórmulas   Estrutura    Local   Controle
                                        leitura                         documental           
  ---------------- --------- ---------- ---------- --------- ---------- ------------ ------- ----------
  **Tesseract**    forte     limitado   limitado   não é     não é foco baixa        sim     muito alto
                                                   foco                                      

  **Surya**        forte     forte      forte      sim       parcial    média/alta   sim     alto

  **PaddleOCR /    forte     forte      forte      forte     forte      alta         sim     alto
  PP-Structure**                                                                             

  **Docling**      depende   forte      forte      forte     sim        muito alta   sim     alto
                   do engine                                                                 

  **MinerU**       forte     forte      forte      forte     forte      muito alta   sim     alto

  **Serviços       forte     forte      varia      varia     varia      alta         não     menor
  externos**                                                                                 
  -----------------------------------------------------------------------------------------------------

A matriz é qualitativa e não estabelece ranking de precisão.

------------------------------------------------------------------------

# 12. Características operacionais

  ------------------------------------------------------------------------------
  Alternativa     Custo           Dependência    Flexibilidade   Papel
                  computacional   externa                        especializado
                  relativo                                       
  --------------- --------------- -------------- --------------- ---------------
  **Tesseract**   baixo           muito baixa    alta            muito alta

  **Surya**       alto            baixa          alta            alta

  **PaddleOCR**   médio--alto,    baixa          muito alta      alta
                  conforme                                       
                  módulos                                        

  **Docling**     variável,       baixa          muito alta      muito alta
                  conforme                                       
                  pipeline                                       

  **MinerU**      alto, conforme  baixa          alta            alta
                  backend                                        

  **Serviços      baixo           alta           alta            alta
  externos**      localmente /                                   
                  custo por uso                                  
  ------------------------------------------------------------------------------

Essas classificações não substituem testes próprios.

------------------------------------------------------------------------

# 13. Implicações arquiteturais descobertas

A investigação tecnológica revelou que existem capacidades distintas que
podem ser distribuídas entre componentes:

### 13.1 OCR especializado

Um motor textual simples continua relevante mesmo dentro de uma
arquitetura documental complexa.

### 13.2 Percepção documental

Layout, posição e ordem de leitura são capacidades distintas do
reconhecimento textual.

### 13.3 Processamento especializado

Tabelas, fórmulas, figuras e gráficos podem exigir mecanismos próprios.

### 13.4 Processamento localizado

É tecnicamente plausível reconhecer apenas páginas, regiões ou elementos
que necessitem de OCR.

### 13.5 Composição

Diferentes tecnologias podem ocupar funções diferentes.

### 13.6 Alternativas condicionais

Uma ferramenta alternativa pode ser acionada apenas em casos específicos
de dificuldade.

### 13.7 Processamento assíncrono

Como o Snoopy já trabalha com jobs assíncronos, ferramentas mais pesadas
podem ser operacionalmente viáveis desde que memória, concorrência,
falhas e recursos sejam controlados.

### 13.8 GPU como aceleração

A investigação não justifica tratar GPU como requisito universal. A
arquitetura futura deve, sempre que possível, possuir uma rota básica
sem GPU e aproveitar aceleração quando disponível.

------------------------------------------------------------------------

# 14. Questões de licença e dependência

A escolha tecnológica deverá considerar separadamente:

-   licença do código;
-   licença dos modelos/pesos;
-   componentes auxiliares;
-   condições de distribuição;
-   restrições comerciais;
-   dependência de serviços externos.

A análise jurídica definitiva não pertence a esta etapa, mas a licença
deve ser critério explícito da seleção posterior.

------------------------------------------------------------------------

# 15. O que a investigação descartou

A investigação não sustenta:

### 15.1 Um OCR universal

Não foi encontrado um único vencedor que maximize simultaneamente
reconhecimento, estrutura, tabelas, fórmulas, imagens, custo e controle.

### 15.2 OCR como simples conversão imagem → texto

Reconhecimento textual correto pode coexistir com reconstrução
documental incorreta.

### 15.3 Adoção automática da representação de uma ferramenta como Canonical

Qualquer ferramenta produz uma representação tecnológica derivada.

### 15.4 Múltiplos OCRs sempre em paralelo

Não há justificativa para redundância universal. Isso acrescentaria
custo, complexidade e reconciliação.

### 15.5 GPU obrigatória

Existem rotas locais sem GPU, embora algumas ferramentas se beneficiem
fortemente de aceleração.

### 15.6 Serviços externos como solução necessariamente inadequada

Eles são tecnicamente válidos, mas introduzem uma classe diferente de
dependência e governança.

------------------------------------------------------------------------

# 16. Complementação metodológica

Embora o foco desta etapa seja tecnológico, a investigação confirmou
restrições importantes para etapas posteriores:

1.  OCR é representação derivada, não o documento.
2.  O original permanece a fonte primária.
3.  Fidelidade envolve texto, estrutura, posição, ordem, tabelas,
    fórmulas e figuras.
4.  Documentos híbridos devem poder conter texto nativo e conteúdo
    reconhecido.
5.  OCR localizado é tecnicamente plausível.
6.  Composição deve significar especialização funcional, não acúmulo
    indiscriminado.
7.  A elegibilidade de uma representação para recuperação deverá ser
    tratada e validada posteriormente.

Essas questões complementam o resultado tecnológico, mas não constituem
o foco principal do presente registro.

------------------------------------------------------------------------

# 17. Alternativas que permanecem em aberto

Permanecem deliberadamente sem decisão:

1.  ferramenta ou conjunto de ferramentas adotado;
2.  componente principal;
3.  papel de Tesseract, Surya, PaddleOCR, Docling e MinerU;
4.  eventual uso de serviços externos;
5.  componentes de fallback;
6.  detecção de páginas/regiões que necessitam de OCR;
7.  granularidade;
8.  representação intermediária;
9.  reconciliação entre resultados;
10. modelos e versões;
11. CPU/GPU;
12. requisitos mínimos de infraestrutura.

Essas questões pertencem à Etapa 6.3.

------------------------------------------------------------------------

# 18. Conclusões consolidadas

**C1.** OCR não é uma categoria única: existem motores especializados,
percepção documental, parsing integrado e serviços externos.

**C2.** Não existe vencedor universal; as alternativas possuem
diferentes combinações de capacidade, custo, controle e complexidade.

**C3.** Tesseract é relevante como componente OCR simples, local e
controlável.

**C4.** Surya amplia o problema para OCR + percepção de layout e ordem
de leitura, com maior custo computacional.

**C5.** PaddleOCR oferece uma stack modular e ampla de OCR e
processamento documental.

**C6.** Docling é candidato especialmente relevante como camada de
processamento/orquestração documental e integração de diferentes
engines.

**C7.** MinerU é candidato forte para parsing documental científico e
documentos complexos, condicionado a requisitos computacionais e de
licenciamento.

**C8.** Serviços externos constituem alternativas tecnicamente válidas,
mas introduzem custos e dependências específicas.

**C9.** Composição tecnológica é plausível e pode distribuir
responsabilidades.

**C10.** Composição não deve significar redundância indiscriminada.

**C11.** Alternativas condicionais são mais justificáveis que
redundância universal.

**C12.** OCR localizado é uma possibilidade tecnológica relevante.

**C13.** O processamento assíncrono do Snoopy favorece a utilização de
ferramentas mais pesadas quando operacionalmente justificável.

**C14.** GPU deve ser tratada como aceleração possível, não requisito
universal.

**C15.** A representação produzida por uma ferramenta não é
automaticamente o Canonical do Snoopy.

**C16.** Licenciamento deve integrar a seleção tecnológica.

**C17.** A escolha definitiva pertence à Etapa 6.3.

------------------------------------------------------------------------

# 19. Síntese tecnológica

O espaço investigado pode ser representado assim:

``` text
                 TECNOLOGIAS DE RECONHECIMENTO
                              │
        ┌─────────────────────┼──────────────────────┐
        │                     │                      │
        ▼                     ▼                      ▼
 OCR especializado     percepção documental    Document AI externo
        │                     │                      │
   Tesseract            Surya / PaddleOCR      Google / Azure /
                              │                AWS / Adobe
                              │
                              ▼
                    parsing documental
                              │
                       Docling / MinerU
```

Essa classificação não representa hierarquia de qualidade. Representa
diferentes posições no espaço tecnológico.

A principal consequência é que a arquitetura futura não precisa
necessariamente escolher uma única ferramenta para todo o problema.

------------------------------------------------------------------------

# 20. Princípio tecnológico consolidado

> **O Snoopy-RAG não deve ser concebido como dependente de um único
> motor universal de OCR. O espaço tecnológico disponível apresenta
> diferentes classes de soluções --- desde motores OCR especializados
> até pipelines de processamento documental e serviços externos --- com
> diferentes capacidades, custos, níveis de controle e limitações. A
> escolha futura deverá considerar qual capacidade cada tecnologia
> fornece, em que condições ela pode ser executada e qual função
> justificável ela pode desempenhar na arquitetura.**

------------------------------------------------------------------------

# 21. Limite da investigação

A investigação tecnológica é considerada suficientemente abrangente
quando novas ferramentas deixam de acrescentar uma capacidade, restrição
ou posição arquitetural significativamente diferente das alternativas já
caracterizadas.

Neste ponto, foram cobertas as principais classes de solução relevantes:

-   OCR especializado;
-   OCR + percepção documental;
-   parsing documental integrado;
-   serviços externos de Document AI.

Não há necessidade metodológica de ampliar indefinidamente a lista de
ferramentas.

------------------------------------------------------------------------

# 22. Transição para a Etapa 6.3

A Etapa 6.2.2 entrega à Etapa 6.3:

-   conjunto caracterizado de alternativas;
-   capacidades;
-   pontos fortes;
-   limitações;
-   custos e exigências operacionais;
-   diferenças de controle e dependência;
-   papéis potenciais;
-   condições que devem orientar a composição.

A passagem seguinte será:

> **requisito → critério técnico → alternativa investigada → escolha →
> composição → arquitetura concreta**

------------------------------------------------------------------------

# 23. Princípio de encerramento

> **A Etapa 6.2.2 não escolheu "o melhor OCR". Ela identificou e
> caracterizou o espaço de tecnologias disponíveis para reconhecimento e
> processamento documental, demonstrou que essas tecnologias ocupam
> papéis diferentes e estabeleceu as bases para que a arquitetura do
> Snoopy possa selecionar e combinar capacidades de maneira
> justificada.**

**Estado da Etapa 6.2.2: CONSOLIDADA.**

