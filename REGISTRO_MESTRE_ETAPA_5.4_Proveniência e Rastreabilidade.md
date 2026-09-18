# Registro Mestre da Análise — Snoopy-RAG

## Etapa 5.4 — Proveniência e Rastreabilidade

> **Status:** Especificação metodológica e arquitetural — não constitui implementação.
> **Função:** definir como o Snoopy deve preservar e reconstruir a origem das evidências e das transformações realizadas sobre elas.
> **Dependência:** Etapas 4, 5.1, 5.2 e 5.3.
> **Próximas dependências:** Etapas 5.5 e 5.6.

---

# 1. Objetivo da Etapa 5.4

A Etapa 5.4 trata da **proveniência e da rastreabilidade** das informações manipuladas pelo Snoopy-RAG.

A questão central não é simplesmente:

> “De qual PDF veio este trecho?”

O sistema já consegue fazer isso funcionalmente em boa parte dos casos. A questão arquitetural mais importante é:

> **“Como o Snoopy deve representar e preservar a cadeia de relações que permite reconstruir de onde uma evidência veio, quais transformações sofreu, como foi recuperada e em que ponto apareceu na resposta?”**

Essa diferença é fundamental.

A arquitetura atual possui uma cadeia funcional de rastreabilidade:

```text
resposta
   ↓
[TRECHO X]
   ↓
chunk
   ↓
documento
   ↓
drive_file_id
   ↓
Google Drive
   ↓
PDF original
```

Essa cadeia já representa uma propriedade importante do sistema. Entretanto, ela ainda não constitui uma **proveniência formal, versionada e completa**. A própria reconstrução da arquitetura anterior identificou essa distinção: a implementação atual possui rastreabilidade funcional, mas não um sistema formal de proveniência versionada. 

A Etapa 5.4 deve, portanto, definir o que uma arquitetura futura precisa preservar para que essa cadeia deixe de depender apenas de relações implícitas entre objetos e passe a ser uma propriedade estrutural do sistema.

---

# 2. Contexto das etapas anteriores

A questão da proveniência não surgiu isoladamente.

Ela aparece desde a origem do Snoopy. O objetivo inicial já incluía a possibilidade de retornar ao PDF original, e posteriormente a preocupação evoluiu de simplesmente indicar a fonte para apresentar trecho, contexto e Modo Leitura. 

A evolução pode ser resumida assim:

```text
PDF
 ↓
resposta
 ↓
fonte
```

depois:

```text
PDF
 ↓
trecho
 ↓
fonte
```

e posteriormente:

```text
PDF
 ↓
trecho
 ↓
contexto
 ↓
leitura do documento
 ↓
fonte original
```

A arquitetura atual já distingue `answer`, `chunks` e `sources`, sendo os `[TRECHO X]` uma ligação operacional entre a síntese e os contextos recuperados, e não uma citação bibliográfica convencional. 

Essa evolução é importante porque revela que a rastreabilidade não é um detalhe de interface. Ela é parte do **núcleo epistemológico do Snoopy**.

---

# 3. O problema central: rastrear não é apenas apontar para a fonte

Existe uma diferença entre:

### Rastreabilidade simples

> “Este trecho veio do documento X.”

e:

### Proveniência

> “Este trecho deriva do bloco Y da página Z do documento X, foi transformado pela representação A, segmentado pela estratégia B, recuperado pela consulta C, selecionado sob a configuração D e apresentado ao pesquisador como evidência E.”

A segunda descrição contém não apenas uma origem, mas uma **história de derivação**.

Essa diferença torna-se especialmente importante porque o Snoopy não trabalha diretamente com o documento original em todas as etapas.

O documento passa por transformações:

```text
PDF original
   ↓
representação canônica
   ↓
representação estrutural
   ↓
texto pesquisável
   ↓
segmentação
   ↓
unidade de recuperação
   ↓
embedding
   ↓
resultado de recuperação
   ↓
contexto
   ↓
síntese
```

Cada seta representa uma operação potencialmente capaz de alterar, reorganizar ou selecionar informação.

Portanto:

> **A origem de uma informação não é suficiente; é necessário preservar sua relação com as representações derivadas que surgiram a partir dela.**

---

# 4. Rastreabilidade documental e proveniência metodológica

Esta é provavelmente a distinção conceitual mais importante desta etapa.

## 4.1. Rastreabilidade documental

A rastreabilidade documental responde:

> **“De onde veio esta informação?”**

Por exemplo:

```text
TRECHO 3
 ↓
retrieval_unit_184
 ↓
bloco 27
 ↓
página 8
 ↓
documento X
 ↓
PDF original
```

Essa é uma propriedade que o sistema pode e deve fornecer.

---

## 4.2. Proveniência metodológica

A proveniência metodológica responde a outra pergunta:

> **“Por que esta informação foi considerada relevante para esta investigação?”**

Isso pode envolver:

* pergunta de pesquisa;
* estratégia de busca;
* critérios de inclusão;
* critérios de exclusão;
* recorte documental;
* categoria analítica;
* unidade de registro;
* unidade de contexto;
* interpretação;
* decisão do pesquisador.

Esses elementos não são equivalentes à origem documental.

A arquitetura atual deliberadamente não persiste como objetos científicos formais a unidade de registro, unidade de contexto definida pelo pesquisador, código, categoria, interpretação ou decisão analítica. 

Isso deve continuar sendo respeitado.

Portanto:

```text
PROVENIÊNCIA DOCUMENTAL
        ≠
PROVENIÊNCIA METODOLÓGICA
```

O Snoopy deve oferecer a primeira de maneira robusta.

A segunda pode eventualmente ser apoiada por funcionalidades futuras, mas **não deve ser presumida simplesmente porque o sistema recuperou um trecho**.

---

# 5. O problema epistemológico da transformação

A Etapa 5.1 estabeleceu que o documento original não deve ser substituído por uma representação textual derivada.

Isso possui uma consequência direta para a proveniência.

Se o sistema faz:

```text
PDF
 ↓
texto
 ↓
chunk
```

e posteriormente perde a relação precisa entre o chunk e o documento, torna-se impossível saber exatamente o que aquele chunk representa.

Mesmo quando a relação documental é preservada, existe outro problema: o chunk não é necessariamente equivalente ao conteúdo original.

A arquitetura atual reconhece explicitamente que o chunk é resultado de transformação e não é byte-a-byte equivalente a uma página do PDF. 

Logo:

> **Rastreabilidade não significa equivalência.**

Poder apontar para o PDF original não significa que o trecho armazenado no banco reproduza perfeitamente aquilo que estava na página.

Essa distinção deve ser preservada na nova arquitetura.

---

# 6. A cadeia de proveniência que o Snoopy deve conseguir reconstruir

A arquitetura futura deve permitir, conceitualmente, reconstruir a seguinte cadeia:

```text
AFIRMAÇÃO APRESENTADA
        ↓
EVIDÊNCIA ASSOCIADA
        ↓
UNIDADE DE RECUPERAÇÃO
        ↓
SEGMENTAÇÃO
        ↓
BLOCO DOCUMENTAL
        ↓
PÁGINA
        ↓
DOCUMENTO
        ↓
REPRESENTAÇÃO CANÔNICA
        ↓
ARQUIVO ORIGINAL
```

Quando a evidência tiver sido recuperada por uma consulta derivada:

```text
PERGUNTA ORIGINAL
        ↓
CONSULTA DERIVADA
        ↓
CONFIGURAÇÃO DE RECUPERAÇÃO
        ↓
UNIDADE RECUPERADA
        ↓
BLOCO
        ↓
PÁGINA
        ↓
DOCUMENTO
        ↓
ARQUIVO ORIGINAL
```

E quando uma síntese tiver sido produzida:

```text
PERGUNTA
   ↓
CONSULTA(S)
   ↓
RESULTADOS
   ↓
EVIDÊNCIAS SELECIONADAS
   ↓
CONTEXTO ENVIADO AO MODELO
   ↓
SÍNTESE
   ↓
[TRECHO X]
   ↓
EVIDÊNCIA
   ↓
DOCUMENTO ORIGINAL
```

Essa cadeia não significa que toda afirmação gerada seja necessariamente verdadeira ou metodologicamente válida.

Ela significa apenas que o sistema deve tornar possível **auditar a relação entre afirmação, evidência e documento**.

---

# 7. Proveniência deve acompanhar as transformações

Uma consequência direta das Etapas 5.1–5.3 é que a proveniência não pode ser adicionada apenas no final.

Ela deve acompanhar o documento desde sua entrada.

A arquitetura conceitual passa a ser:

```text
DOCUMENTO ORIGINAL
       │
       │ origem
       ▼
REPRESENTAÇÃO CANÔNICA
       │
       │ deriva
       ▼
REPRESENTAÇÃO ESTRUTURAL
       │
       │ segmentação
       ▼
UNIDADE DE RECUPERAÇÃO
       │
       │ representação vetorial
       ▼
EMBEDDING
       │
       │ recuperação
       ▼
RESULTADO
       │
       │ seleção
       ▼
EVIDÊNCIA
       │
       │ contextualização
       ▼
SÍNTESE
```

Cada representação derivada deve manter uma relação identificável com aquilo de que deriva.

O princípio é:

> **Nenhuma representação derivada deve se tornar órfã de sua origem.**

---

# 8. Identidade versus conteúdo

A proveniência exige separar duas coisas:

### Identidade do objeto

> “Qual documento, versão, página, bloco ou unidade é este?”

e:

### Conteúdo do objeto

> “Qual é o texto, imagem, tabela ou outro conteúdo associado a ele?”

Essa distinção permite evitar problemas em que o mesmo texto aparece em vários documentos ou em diferentes versões de um mesmo documento.

Por exemplo:

```text
documento A
 └── unidade 42
       └── "avaliação formativa..."
```

não deve ser identificado simplesmente pelo conteúdo textual.

A identidade deve considerar sua posição e relação documental.

Isso também reforça uma decisão estabelecida na Etapa 5.3:

> **deduplicação deve considerar a identidade da unidade documental, e não apenas igualdade textual.**

Do contrário, duas ocorrências legítimas do mesmo trecho poderiam ser tratadas como uma única evidência.

---

# 9. Localização documental

A proveniência precisa representar uma localização suficientemente precisa para permitir a verificação humana.

O nível mínimo desejável é:

```text
documento
 ↓
página
 ↓
unidade
```

Mas a arquitetura ideal pode preservar uma granularidade maior:

```text
documento
 ↓
página
 ↓
bloco
 ↓
elemento
 ↓
unidade de recuperação
```

Quando disponível, informações espaciais podem acrescentar:

```text
bounding box
(x, y, width, height)
```

Isso é especialmente importante para documentos nos quais a ordem textual não é suficiente para determinar a relação entre elementos.

Por exemplo:

```text
página 5
 ├── título
 ├── coluna esquerda
 ├── coluna direita
 ├── tabela
 ├── legenda
 └── nota de rodapé
```

Um sistema que guarda apenas:

```text
"texto da página 5"
```

perde parte da proveniência estrutural.

A Etapa 5.1 já estabeleceu que página, bloco, ordem de leitura, estrutura e relações espaciais são características relevantes que devem ser preservadas quando disponíveis.

---

# 10. Proveniência de tabelas, imagens e elementos não textuais

A proveniência não pode ser pensada exclusivamente para texto.

Considere uma tabela:

```text
PDF
 ↓
tabela da página 12
 ↓
estrutura tabular
 ↓
representação textual para busca
```

Se a busca retornar um trecho textual derivado da tabela, o Snoopy precisa permitir responder:

> “Esse texto veio de onde?”

A resposta não deve ser simplesmente:

> “Da página 12.”

Deve ser possível identificar que a origem foi:

```text
documento X
 → página 12
 → elemento tabela
 → tabela Y
 → representação textual derivada
 → unidade recuperada
```

O mesmo vale para:

* figuras;
* gráficos;
* legendas;
* imagens contendo texto;
* notas;
* referências;
* caixas laterais;
* elementos OCR.

A representação canônica é justamente o ponto a partir do qual essas relações devem ser preservadas.

---

# 11. Proveniência de OCR

Documentos digitalizados introduzem uma camada adicional.

Temos:

```text
imagem da página
      ↓
OCR
      ↓
texto reconhecido
      ↓
segmentação
      ↓
recuperação
```

O texto produzido pelo OCR não deve ser tratado como se fosse o conteúdo textual original sem qualificação.

A cadeia precisa permitir distinguir:

```text
texto nativo do PDF
```

de:

```text
texto obtido por OCR
```

Isso é relevante porque erros de reconhecimento podem alterar:

* palavras;
* números;
* caracteres;
* símbolos;
* fórmulas;
* referências;
* nomes próprios.

Assim, quando uma evidência tiver origem em OCR, essa informação deve fazer parte de sua proveniência.

Não necessariamente para poluir a interface do usuário, mas para que o sistema consiga responder tecnicamente:

> **“Como esse texto chegou a existir dentro do Snoopy?”**

---

# 12. Proveniência das consultas

A Etapa 5.3 estabeleceu que uma pergunta pode ser decomposta em consultas derivadas.

Portanto, o resultado da busca precisa manter relação com a consulta que o produziu.

Exemplo:

```text
Pergunta original:
"Como a avaliação formativa participa da regulação da aprendizagem?"
```

pode resultar em:

```text
consulta 1:
"avaliação formativa regulação aprendizagem"

consulta 2:
"feedback regulação aprendizagem"

consulta 3:
"avaliação regulação processos aprendizagem"
```

Se um documento apareceu porque foi recuperado pela consulta 2, isso deve ser reconstruível.

A cadeia torna-se:

```text
pergunta original
       ↓
decomposição
       ↓
consulta derivada #2
       ↓
embedding / mecanismo de busca
       ↓
unidade recuperada
```

Isso é importante porque a decomposição não é uma operação epistemologicamente neutra.

Ela é uma **transformação operacional da consulta**.

O sistema não deve esconder essa transformação atrás de um único campo “query”.

---

# 13. Proveniência da configuração de recuperação

Também não basta registrar qual consulta produziu o resultado.

O resultado depende da configuração utilizada.

A Etapa 5.3 estabeleceu o conceito de **perfil de recuperação**, que deve incluir elementos como:

```text
consulta original
consultas derivadas
corpus
versão do corpus
versão da representação
versão da segmentação
modelo de embedding
dimensões
métrica
threshold
top-k
ranking
deduplicação
contextualização
versão do pipeline
```

Portanto:

```text
resultado
```

deve ser conceitualmente associado a:

```text
perfil de recuperação
```

e não apenas a:

```text
query
```

Isso permite distinguir:

> “o sistema encontrou este trecho”

de:

> “o sistema encontrou este trecho sob determinada configuração de recuperação.”

Essa diferença é essencial para reprodutibilidade.

---

# 14. Proveniência e versionamento

Uma evidência não deve ser considerada imutável simplesmente porque seu texto permanece armazenado.

Se o documento original for substituído, atualizado ou reprocessado, podem surgir:

```text
Documento X — versão 1
        ↓
chunks v1
        ↓
embeddings v1
```

e posteriormente:

```text
Documento X — versão 2
        ↓
chunks v2
        ↓
embeddings v2
```

Se o sistema mantiver apenas:

```text
document_id = X
```

sem distinguir versões, uma referência histórica pode perder sua precisão.

A proveniência precisa, portanto, permitir algo conceitualmente próximo de:

```text
documento
 ├── versão 1
 │    ├── representação
 │    ├── unidades
 │    └── embeddings
 │
 └── versão 2
      ├── representação
      ├── unidades
      └── embeddings
```

Isso não significa que toda versão necessariamente precise permanecer disponível indefinidamente.

Significa que o sistema precisa ter uma política explícita para distinguir:

* estado atual;
* estado anterior;
* evidência histórica;
* representação vigente;
* derivados invalidados.

Essa questão será aprofundada na Etapa 5.5.

---

# 15. Proveniência e atualização do corpus

O corpus é parte do contexto da recuperação.

Portanto, quando um documento:

* é adicionado;
* removido;
* substituído;
* alterado;
* reprocessado;

isso pode alterar os resultados de uma busca.

Uma resposta antiga não deve parecer ter sido produzida sobre o corpus atual se foi produzida sobre outro estado documental.

Idealmente:

```text
busca A
 ↓
corpus versão 17
 ↓
resultado X
```

deve permanecer distinguível de:

```text
busca B
 ↓
corpus versão 21
 ↓
resultado Y
```

Isso é especialmente importante para qualquer tentativa futura de avaliação de estabilidade ou reprodutibilidade.

---

# 16. Proveniência da síntese

A síntese do LLM é uma representação derivada.

Isso foi estabelecido na Etapa 4 e deve permanecer como princípio:

```text
documento original
      ↓
evidência
      ↓
síntese
```

A síntese não deve substituir a evidência.

Por isso, quando uma resposta contiver:

```text
"A avaliação formativa favorece a regulação da aprendizagem..."
```

e houver:

```text
[TRECHO 1]
[TRECHO 4]
```

a arquitetura deve conseguir estabelecer a relação entre:

```text
afirmação
   ↓
evidência(s)
   ↓
unidade(s) recuperada(s)
   ↓
documento(s)
```

Mas isso não deve ser interpretado como uma prova automática de que a afirmação está metodologicamente correta.

O sistema pode demonstrar:

> “Estas foram as evidências apresentadas como suporte.”

Ele não pode concluir automaticamente:

> “Logo, a afirmação é cientificamente válida.”

Essa fronteira permanece essencial.

---

# 17. `[TRECHO X]` como referência operacional

Os marcadores:

```text
[TRECHO 1]
[TRECHO 2]
[TRECHO 3]
```

são uma solução funcional importante da arquitetura atual.

Eles estabelecem uma ligação entre:

```text
síntese
   ↕
evidência recuperada
```

e a interface permite navegar dessa evidência até o documento.

A análise anterior classificou corretamente esses marcadores como **referências operacionais de evidência**, e não como citações bibliográficas convencionais. 

A arquitetura futura deve preservar essa distinção.

Um `[TRECHO X]` não deve ser confundido com:

* citação bibliográfica;
* unidade de registro;
* categoria;
* interpretação;
* comprovação automática da afirmação.

Ele é uma **âncora navegável para uma evidência recuperada**.

---

# 18. O papel do Modo Leitura

O Modo Leitura demonstra por que a proveniência precisa ser mais robusta.

Na implementação atual, o fluxo é aproximadamente:

```text
chunk recuperado
       ↓
documento
       ↓
chunks do documento
       ↓
reconstrução textual
       ↓
localização aproximada do chunk
       ↓
contexto ampliado
```

A implementação atual reconstrói o documento a partir dos chunks armazenados e não do PDF original. 

Isso é funcional, mas revela uma fragilidade conceitual:

> **a leitura ampliada atualmente depende da representação textual processada, não da representação documental original.**

Na arquitetura futura, o Modo Leitura deve poder aproveitar a cadeia de proveniência para responder de maneira mais precisa:

```text
evidência
 ↓
posição documental
 ↓
contexto estrutural
 ↓
documento
 ↓
fonte original
```

Assim, o Modo Leitura deixa de ser apenas uma reconstrução de chunks e passa a ser uma visualização derivada da estrutura documental preservada.

---

# 19. Proveniência não significa armazenar tudo em todos os lugares

Existe um risco oposto.

Ao perceber a importância da proveniência, poderíamos tentar colocar todas as informações em cada objeto:

```text
chunk
 ├── document_id
 ├── page
 ├── block
 ├── bbox
 ├── query
 ├── embedding_model
 ├── corpus_version
 ├── pipeline_version
 ├── original_pdf_hash
 ├── ...
```

Isso pode gerar redundância e inconsistência.

A solução arquitetural não deve ser:

> “Duplicar toda a história em todos os objetos.”

Deve ser:

> **“Construir relações identificáveis entre objetos e manter os metadados necessários nos níveis apropriados.”**

Assim, a proveniência pode ser modelada como uma rede:

```text
DOCUMENTO
   │
   ├── possui → PÁGINA
   │              │
   │              └── possui → BLOCO
   │                              │
   │                              └── deriva → UNIDADE
   │
   └── possui → VERSÃO
```

e:

```text
CONSULTA
   │
   └── produz → RESULTADO
                  │
                  └── seleciona → UNIDADE
```

e:

```text
SÍNTESE
   │
   └── referencia → EVIDÊNCIA
                       │
                       └── deriva → UNIDADE
```

A proveniência passa então a ser uma **rede de relações**, não uma coleção de cópias.

---

# 20. Proveniência e integridade

A proveniência também precisa estar associada à identidade do documento.

O sistema atual já calcula um hash documental durante o pipeline. O MD5 foi entendido na análise como identidade binária do documento dentro do contexto atual, e não como deduplicação semântica.

Na arquitetura futura, um mecanismo de identificação documental deve permitir estabelecer:

```text
arquivo original
      ↓
identidade/hash
      ↓
versão documental
      ↓
representação canônica
```

O objetivo não é transformar hash em prova absoluta de identidade científica.

É permitir verificar:

> “Qual arquivo/versão serviu de origem para esta representação?”

Essa informação será particularmente importante quando o documento do Drive for alterado.

---

# 21. Proveniência e auditoria

Uma arquitetura com boa proveniência deve permitir pelo menos dois tipos de auditoria.

## Auditoria para frente

Partindo do documento:

> “O que o Snoopy fez com este documento?”

```text
PDF
 ↓
representação
 ↓
segmentação
 ↓
unidades
 ↓
embeddings
 ↓
recuperações
 ↓
evidências
```

## Auditoria para trás

Partindo de uma resposta:

> “De onde veio esta informação?”

```text
afirmação
 ↓
evidência
 ↓
unidade
 ↓
bloco
 ↓
página
 ↓
documento
 ↓
versão
 ↓
PDF original
```

As duas direções são importantes.

A primeira permite investigar o comportamento do sistema.

A segunda permite verificar o resultado.

---

# 22. Proveniência e reproducibilidade

A proveniência é uma das condições para a reprodutibilidade, mas não é sinônimo dela.

Podemos ter:

```text
boa rastreabilidade
```

sem ter:

```text
recuperação reproduzível
```

Por exemplo, o sistema pode informar perfeitamente:

> “Este resultado veio do documento X, página 7.”

mas, quando a mesma pergunta for executada novamente, retornar outro trecho.

Nesse caso existe rastreabilidade, mas a estabilidade da recuperação ainda é insuficiente.

A relação correta é:

```text
PROVENIÊNCIA
     +
CONFIGURAÇÃO
     +
VERSIONAMENTO
     +
ESTABILIDADE
     ↓
CONDIÇÕES PARA REPRODUÇÃO DA RECUPERAÇÃO
```

Isso reforça a conclusão da Etapa 5.3 de que a reprodutibilidade pretendida pelo Snoopy é principalmente a **reprodutibilidade da evidência recuperada**, e não a reprodução literal da redação do LLM. 

---

# 23. Proveniência e cache

O cache introduz uma distinção importante.

Se uma resposta é reutilizada:

```text
pergunta
 ↓
cache
 ↓
resposta antiga
```

não houve uma nova recuperação.

Se o cache é ignorado:

```text
pergunta
 ↓
nova recuperação
 ↓
novos resultados
```

houve processamento novo.

Portanto, para fins de auditoria e avaliação, deve ser possível distinguir:

```text
resultado obtido por nova execução
```

de:

```text
resultado recuperado do cache
```

A análise atual já identificou que o cache pode mascarar testes de reprodutibilidade. 

Na arquitetura futura, o cache não deve apagar a história da operação que originou o resultado.

---

# 24. Proveniência como propriedade do sistema, não apenas da interface

É importante evitar uma solução superficial em que a interface simplesmente exibe:

> “Fonte: Documento X”.

Isso seria uma **apresentação de fonte**, não necessariamente proveniência.

A proveniência deve existir abaixo da interface:

```text
dados persistentes
       ↓
relações de derivação
       ↓
API
       ↓
interface
```

Assim, diferentes interfaces podem consumir a mesma cadeia.

Por exemplo:

```text
interface de busca
Modo Leitura
exportação
auditoria
avaliação experimental
```

todos poderiam utilizar a mesma estrutura de proveniência.

---

# 25. Proveniência e exportação

Uma consequência natural é que uma evidência exportada pelo Snoopy não deveria perder sua origem.

Por exemplo, se no futuro o sistema permitir exportar:

```text
TRECHO
DOCUMENTO
PÁGINA
CONTEXTO
```

o resultado deveria manter informações suficientes para que o pesquisador saiba:

```text
qual documento
qual versão
qual localização
qual unidade
qual contexto
```

e, quando aplicável:

```text
qual consulta
qual configuração
qual recuperação
```

Isso não transforma a exportação automaticamente em um registro metodológico completo.

Mas impede que a evidência se torne um fragmento textual sem origem.

---

# 26. Proveniência e unidade de registro

A arquitetura precisa preservar novamente uma distinção fundamental:

```text
unidade de recuperação
        ≠
unidade de registro
```

Um pesquisador pode recuperar:

```text
chunk 17
```

e posteriormente identificar dentro dele:

```text
unidade de registro A
```

ou:

```text
unidade de registro A
+
unidade de registro B
```

ou até:

```text
nenhuma unidade de registro relevante
```

Portanto, a proveniência deve permitir que uma futura unidade metodológica seja associada a uma parte do documento sem transformar retroativamente o chunk na própria unidade.

A cadeia poderia futuramente ser:

```text
documento
 ↓
bloco
 ↓
unidade de recuperação
 ↓
unidade de registro definida pelo pesquisador
 ↓
código/categoria
```

Mas os últimos elementos pertencem a uma camada metodológica que atualmente está fora do núcleo do Snoopy. 

---

# 27. Proveniência metodológica como camada futura

Isso permite uma arquitetura conceitualmente limpa:

```text
CAMADA DOCUMENTAL
documento
 ↓
representação
 ↓
estrutura
 ↓
unidades
```

```text
CAMADA DE RECUPERAÇÃO
consulta
 ↓
recuperação
 ↓
evidências
```

```text
CAMADA METODOLÓGICA
pesquisador
 ↓
unidade de registro
 ↓
código
 ↓
categoria
 ↓
interpretação
```

A proveniência documental conecta as duas primeiras.

Uma eventual camada metodológica futura poderá estabelecer relações adicionais.

Mas:

> **O Snoopy não deve inventar a terceira camada simplesmente porque possui as duas primeiras.**

---

# 28. Requisitos de Proveniência e Rastreabilidade

A partir das discussões anteriores, podem ser definidos os seguintes requisitos arquiteturais.

### RPROV1 — Origem documental

Toda unidade recuperável deve possuir relação identificável com seu documento de origem.

### RPROV2 — Localização

Quando disponível, a unidade deve manter localização documental suficiente para verificação humana.

### RPROV3 — Hierarquia documental

A proveniência deve permitir reconstruir relações como:

```text
documento → página → bloco → elemento → unidade
```

quando essas estruturas existirem.

### RPROV4 — Derivação

Representações derivadas devem manter relação identificável com suas representações de origem.

### RPROV5 — Não-orfandade

Nenhuma unidade de recuperação deve existir sem uma origem documental identificável.

### RPROV6 — Distinção de representação

O sistema deve distinguir documento original, representação canônica, representações derivadas e síntese.

### RPROV7 — Consulta original

A recuperação deve manter relação com a pergunta original.

### RPROV8 — Consultas derivadas

Quando houver decomposição da pergunta, as consultas derivadas devem permanecer identificáveis.

### RPROV9 — Configuração

O resultado de recuperação deve ser associado às configurações relevantes que determinaram sua produção.

### RPROV10 — Corpus

A recuperação deve manter relação com o corpus considerado.

### RPROV11 — Versionamento

A proveniência deve permitir distinguir versões relevantes do documento, representação e pipeline.

### RPROV12 — Embedding

A unidade vetorial deve manter relação com a versão da representação e configuração de embedding que a originou.

### RPROV13 — Ranking

A posição de uma unidade no resultado deve ser reconstruível em relação à execução de recuperação correspondente.

### RPROV14 — Deduplicação

A proveniência deve sobreviver às operações de deduplicação e seleção.

### RPROV15 — Contextualização

Contextos adicionados à evidência devem permanecer distinguíveis da unidade primária recuperada.

### RPROV16 — Síntese

A síntese deve manter relação explícita com as evidências fornecidas ao modelo.

### RPROV17 — Referência operacional

Marcadores como `[TRECHO X]` devem apontar para evidências identificáveis e navegáveis.

### RPROV18 — Fonte original

A cadeia deve permitir chegar ao arquivo original quando este estiver disponível.

### RPROV19 — OCR

Quando texto for derivado de OCR, essa condição deve ser identificável na cadeia de proveniência.

### RPROV20 — Elementos não textuais

Tabelas, imagens, gráficos e outros elementos devem poder possuir origem identificável.

### RPROV21 — Auditoria para trás

De uma evidência ou afirmação deve ser possível reconstruir sua origem documental.

### RPROV22 — Auditoria para frente

De um documento deve ser possível identificar quais representações derivadas foram produzidas a partir dele.

### RPROV23 — Cache

A proveniência deve distinguir execução nova de resultado reutilizado por cache.

### RPROV24 — Integridade

A identidade da versão documental deve permanecer associada às representações derivadas correspondentes.

### RPROV25 — Histórico

Resultados históricos não devem perder a referência ao estado documental sob o qual foram produzidos.

### RPROV26 — Separação metodológica

Rastreabilidade documental não deve ser apresentada como proveniência metodológica completa.

### RPROV27 — Unidade de registro

Nenhuma unidade recuperada deve ser automaticamente classificada como unidade de registro.

### RPROV28 — Pesquisador

Decisões metodológicas do pesquisador devem permanecer distinguíveis das operações automáticas do sistema.

### RPROV29 — Exportação

Informações exportadas pelo sistema devem preservar a origem documental essencial da evidência.

### RPROV30 — Verificação

A arquitetura deve permitir que um pesquisador humano verifique uma evidência retornando à fonte correspondente.

---

# 29. Modelo conceitual de proveniência

A estrutura geral pode ser representada assim:

```text
                    DOCUMENTO ORIGINAL
                           │
                           ▼
                 REPRESENTAÇÃO CANÔNICA
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
         ESTRUTURA      TEXTO          ELEMENTOS
             │        PESQUISÁVEL      MULTIMODAIS
             └─────────────┬─────────────┘
                           ▼
                  SEGMENTAÇÃO ESTRUTURAL
                           │
                           ▼
                   SEGMENTAÇÃO SEMÂNTICA
                           │
                           ▼
                 UNIDADES DE RECUPERAÇÃO
                           │
                     ┌─────┴─────┐
                     ▼           ▼
                 EMBEDDING    OUTRAS REPRESENTAÇÕES
                     │
                     └─────┬─────┘
                           ▼
                       RECUPERAÇÃO
                           ▲
                           │
                PERGUNTA → CONSULTAS
                           │
                           ▼
                       RESULTADOS
                           │
                           ▼
                        EVIDÊNCIAS
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                 CONTEXTO       SÍNTESE
                    │             │
                    └──────┬──────┘
                           ▼
                       PESQUISADOR
                           │
                           ▼
                    FONTE ORIGINAL
```

A característica essencial desse modelo é que **todas as representações continuam relacionadas à origem documental**.

---

# 30. Duas cadeias que precisam coexistir

Uma forma particularmente útil de pensar o Snoopy é separar duas cadeias.

## Cadeia documental

```text
PDF
 ↓
representação
 ↓
estrutura
 ↓
unidade
 ↓
evidência
```

Ela responde:

> **“De onde isso veio?”**

## Cadeia de investigação

```text
pergunta
 ↓
consulta
 ↓
recuperação
 ↓
seleção
 ↓
evidência
 ↓
síntese
```

Ela responde:

> **“Como isso foi encontrado?”**

O ponto de encontro é:

```text
                    ┌──────── DOCUMENTO
                    │
PERGUNTA → RECUPERAÇÃO → EVIDÊNCIA
                    │
                    └──────── SÍNTESE
```

Portanto, uma proveniência adequada precisa preservar **as duas dimensões simultaneamente**.

---

# 31. O que a proveniência não pode afirmar

Mesmo uma arquitetura de proveniência muito robusta não permite afirmar automaticamente:

* que o documento é metodologicamente relevante;
* que o trecho é uma unidade de registro;
* que o trecho representa adequadamente o contexto;
* que uma categoria analítica é válida;
* que a interpretação do pesquisador está correta;
* que uma síntese do LLM é verdadeira;
* que uma evidência é suficiente para sustentar uma conclusão;
* que a recuperação foi cientificamente reproduzível apenas porque foi registrada.

A proveniência responde principalmente:

> **“Qual é a história documental e computacional desta informação?”**

Não:

> **“Qual é o significado científico definitivo desta informação?”**

Essa última pergunta permanece no domínio da análise e interpretação.

---

# 32. Avaliação empírica da proveniência

A proveniência também precisa ser testável.

Um conjunto de testes futuros deve verificar se, dado um resultado:

```text
resultado X
```

é possível reconstruir:

```text
pergunta
→ consulta derivada
→ configuração
→ unidade
→ bloco
→ página
→ documento
→ versão
→ original
```

### Teste 1 — Evidência → fonte

Selecionar evidências aleatórias e verificar se todas levam ao documento correto.

### Teste 2 — Evidência → localização

Verificar se a localização indicada corresponde ao conteúdo apresentado.

### Teste 3 — Documento → derivados

Selecionar documentos e verificar se suas unidades derivadas permanecem corretamente vinculadas.

### Teste 4 — Versão

Alterar/reprocessar um documento e verificar se versões antigas e novas não são confundidas.

### Teste 5 — OCR

Selecionar documentos digitalizados e verificar se a origem OCR permanece identificável.

### Teste 6 — Tabelas e imagens

Verificar se elementos não textuais mantêm relação com sua localização documental.

### Teste 7 — Consulta

Verificar se cada resultado pode ser associado à consulta que o produziu.

### Teste 8 — Síntese

Verificar se cada marcador `[TRECHO X]` corresponde exatamente a uma evidência recuperada.

### Teste 9 — Cache

Comparar execução nova e resultado servido do cache.

### Teste 10 — Auditoria reversa

Partir de uma afirmação apresentada pela interface e reconstruir toda a cadeia até a fonte original.

---

# 33. Critério de qualidade

A qualidade da proveniência não deve ser medida apenas por:

> “Existe um link para o PDF?”

Um critério mais adequado é:

> **Dado um resultado apresentado ao pesquisador, o sistema consegue reconstruir de maneira consistente, verificável e suficientemente precisa a cadeia que liga a afirmação à evidência, à unidade recuperada, à localização documental, ao documento e à representação original que lhe deu origem?**

Podemos decompor isso em:

```text
COMPLETUDE
   +
PRECISÃO
   +
CONSISTÊNCIA
   +
VERSIONAMENTO
   +
VERIFICABILIDADE HUMANA
```

---

# 34. Relação com as Etapas 5.1, 5.2 e 5.3

A Etapa 5.4 depende diretamente das anteriores.

### 5.1 — Representação canônica

Define **o que deve ser preservado**.

Sem representação canônica, a proveniência só consegue apontar para uma representação já degradada.

```text
5.1
preservar origem
```

### 5.2 — Segmentação

Define **como o documento é dividido sem romper sua estrutura**.

Sem uma segmentação identificável, torna-se difícil saber exatamente qual parte do documento uma unidade representa.

```text
5.2
estruturar origem
```

### 5.3 — Recuperação

Define **como uma unidade é encontrada e selecionada**.

Sem registrar a consulta e sua configuração, não sabemos como a evidência entrou no resultado.

```text
5.3
recuperar origem
```

### 5.4 — Proveniência

Integra as três:

```text
PRESERVAR
   ↓
ESTRUTURAR
   ↓
RECUPERAR
   ↓
RASTREAR
```

---

# 35. Relação com a Etapa 5.5

A Etapa 5.4 define **quais relações precisam existir**.

A Etapa 5.5 deverá tratar de como essas relações sobrevivem ao tempo.

Isso inclui:

* versões;
* hashes;
* atualizações;
* remoções;
* invalidação;
* consistência;
* reprocessamento;
* concorrência;
* persistência;
* histórico;
* estados intermediários.

A distinção é:

```text
5.4
“O que precisa ser rastreável?”
```

versus:

```text
5.5
“Como manter essa rastreabilidade consistente ao longo do tempo?”
```

---

# 36. Relação com a Etapa 5.6

A proveniência também possui implicações operacionais e de segurança.

Não basta que um objeto possua:

```text
folder_id
document_id
```

se um usuário puder acessar a proveniência de outro acervo.

A análise atual já identificou que o backend utiliza `SUPABASE_SERVICE_KEY`, que ultrapassa as barreiras do RLS, tornando necessária uma validação explícita de autorização no backend. 

Portanto, a proveniência precisa respeitar:

```text
proveniência
     ↓
escopo autorizado
     ↓
usuário / acervo
```

A informação de origem não deve se transformar em um mecanismo de vazamento entre acervos.

---

# 37. O princípio central da Etapa 5.4

A formulação mais importante desta etapa pode ser:

> **Toda evidência recuperada pelo Snoopy deve possuir uma cadeia identificável de proveniência que permita relacioná-la à unidade documental que lhe deu origem, à sua localização, ao documento e à versão da representação utilizada, preservando também, quando aplicável, a consulta e a configuração de recuperação que produziram o resultado.**

E uma segunda formulação complementa:

> **Rastreabilidade documental não deve ser confundida com proveniência metodológica: o sistema deve registrar de onde a evidência veio e como foi recuperada, mas a decisão sobre sua relevância metodológica, sua classificação e sua interpretação permanece com o pesquisador.**

---

# 38. A cadeia epistemicamente correta do Snoopy

Depois das Etapas 5.1–5.4, a cadeia conceitual do Snoopy pode ser refinada para:

```text
DOCUMENTO ORIGINAL
        ↓
PRESERVAÇÃO
        ↓
ESTRUTURAÇÃO
        ↓
SEGMENTAÇÃO
        ↓
REPRESENTAÇÃO
        ↓
RECUPERAÇÃO
        ↓
RASTREABILIDADE
        ↓
EVIDÊNCIA
        ↓
SÍNTESE
        ↓
PESQUISADOR
        ↓
INTERPRETAÇÃO
```

Ou, em uma formulação ainda mais sintética:

> **preservar → estruturar → segmentar → representar → recuperar → rastrear → sintetizar → interpretar**

A importância dessa cadeia é que cada etapa possui uma função própria.

Não se deve permitir que:

```text
chunk
```

substitua:

```text
documento
```

nem que:

```text
embedding
```

substitua:

```text
evidência
```

nem que:

```text
síntese
```

substitua:

```text
fonte
```

nem que:

```text
recuperação
```

substitua:

```text
interpretação
```

---

# 39. Síntese da Etapa 5.4

A análise da arquitetura atual mostra que o Snoopy já possui uma forma funcional de rastreabilidade. A resposta contém os chunks utilizados, os marcadores `[TRECHO X]` conectam a síntese às evidências e os resultados possuem relação com documentos e com o Google Drive. Essa estrutura já constitui uma propriedade relevante do sistema. 

O problema é que essa rastreabilidade ainda está concentrada principalmente no **resultado final**. Ela precisa ser transformada em uma propriedade transversal da arquitetura.

A nova arquitetura deve permitir reconstruir:

```text
PERGUNTA
   ↓
CONSULTA
   ↓
CONFIGURAÇÃO
   ↓
RESULTADO
   ↓
EVIDÊNCIA
   ↓
UNIDADE
   ↓
BLOCO
   ↓
PÁGINA
   ↓
DOCUMENTO
   ↓
VERSÃO
   ↓
REPRESENTAÇÃO CANÔNICA
   ↓
ARQUIVO ORIGINAL
```

Ao mesmo tempo, deve preservar a distinção:

```text
rastreabilidade documental
        ≠
proveniência metodológica
```

O sistema pode dizer **de onde veio** uma evidência e **como ela foi recuperada**.

Ele não deve decidir sozinho **o que essa evidência significa metodologicamente**.

---

# 40. Conclusão

A principal consequência da Etapa 5.4 é que a proveniência deixa de ser entendida como uma funcionalidade de interface — “clicar no trecho e abrir o PDF” — e passa a ser entendida como uma **propriedade estrutural do sistema documental**.

O Snoopy precisa preservar não apenas o caminho:

```text
trecho → PDF
```

mas a história:

```text
pergunta
 ↓
consulta
 ↓
recuperação
 ↓
evidência
 ↓
unidade documental
 ↓
bloco
 ↓
página
 ↓
documento
 ↓
versão
 ↓
representação canônica
 ↓
arquivo original
```

Isso é especialmente importante porque o projeto está deixando de ser simplesmente um mecanismo de perguntas sobre PDFs. A arquitetura está sendo concebida como uma infraestrutura de exploração documental na qual a evidência precisa permanecer **localizável, verificável e contextualizável**.

A proveniência é, portanto, o mecanismo que impede que a cadeia:

```text
DOCUMENTO → TRANSFORMAÇÃO → RECUPERAÇÃO → SÍNTESE
```

se transforme em uma caixa-preta.

O princípio consolidado desta etapa é:

> **O Snoopy-RAG deverá preservar uma cadeia de proveniência que permita reconstruir a origem documental, a localização e as transformações relevantes de cada evidência recuperada, relacionando-a às consultas e configurações de recuperação que participaram de sua seleção. Essa rastreabilidade deverá permanecer distinta da proveniência metodológica: o sistema pode documentar a origem e o percurso computacional da evidência, mas a relevância metodológica, a delimitação da unidade de registro, a categorização e a interpretação permanecem decisões do pesquisador.**

E isso nos leva diretamente à próxima questão arquitetural:

```text
5.1  PRESERVAR
       ↓
5.2  ESTRUTURAR E SEGMENTAR
       ↓
5.3  RECUPERAR
       ↓
5.4  RASTREAR
       ↓
5.5  PERSISTIR, VERSIONAR E MANTER CONSISTENTE
```

A **Etapa 5.5** será, portanto, o ponto em que a proveniência deixa de ser apenas uma cadeia conceitualmente definida e passa a enfrentar o problema do **tempo**: documentos mudam, versões são reprocessadas, chunks são invalidados, embeddings envelhecem, jobs falham, caches permanecem e diferentes estados do acervo precisam continuar distinguíveis.
