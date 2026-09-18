# Registro Mestre Transversal da Análise — Snoopy-RAG

## Etapa 5 — Relações Arquiteturais, Integridade Documental e Responsabilidade Metodológica

> **Status:** registro mestre transversal da Etapa 5.
> **Natureza:** síntese conceitual e arquitetural — não constitui implementação.
> **Função:** cruzar as subetapas 5.1–5.6 e explicitar os princípios que emergem da relação entre representação, segmentação, recuperação, proveniência, persistência, versionamento, segurança e operação.

---

# 1. Objetivo deste registro

Os registros individuais das subetapas 5.1–5.6 analisaram problemas específicos:

* representação canônica;
* segmentação;
* recuperação;
* proveniência;
* persistência, versionamento e consistência;
* segurança, isolamento e operação.

O Registro Mestre Geral da Etapa 5 organizou esses conteúdos verticalmente, apresentando o papel de cada subetapa.

Este documento possui outra função.

Ele não pretende repetir as seis subetapas em sequência. Seu objetivo é mostrar **como os conceitos atravessam todas elas** e por que determinadas propriedades não pertencem a um único componente do sistema.

Por exemplo:

* proveniência não é apenas uma função da interface;
* reprodutibilidade não é apenas uma propriedade do mecanismo de busca;
* consistência não é apenas uma questão do banco de dados;
* segurança não é apenas controle de acesso;
* o corpus não é apenas uma coleção de arquivos;
* a evidência não é apenas um trecho retornado pelo LLM;
* a representação canônica não é apenas uma melhoria no extrator.

Essas propriedades são sistêmicas.

Elas dependem da articulação entre várias camadas.

---

# 2. Questão transversal central

A pergunta que atravessa toda a Etapa 5 é:

> **Como construir uma infraestrutura documental na qual o sistema possa transformar, segmentar, recuperar, apresentar e reutilizar documentos sem perder a identidade, a origem, o contexto e as condições que tornam essas operações metodologicamente confiáveis?**

Essa pergunta pode ser desdobrada em cinco problemas:

1. **O que exatamente está sendo preservado?**
2. **O que exatamente está sendo derivado?**
3. **O que exatamente está sendo recuperado?**
4. **De qual estado documental e computacional isso veio?**
5. **Em quais condições podemos confiar que o resultado continua válido?**

As seis subetapas respondem a essas perguntas de maneira complementar.

---

# 3. A unidade fundamental do sistema não é o chunk

A análise transversal permite identificar uma mudança importante no modelo mental do Snoopy.

Na arquitetura inicial, o chunk aparece como unidade central:

```text
PDF → texto → chunks → embeddings → busca
```

Na arquitetura especificada pela Etapa 5, o chunk deixa de ser o centro conceitual.

A unidade fundamental passa a ser o **estado documental rastreável**.

Esse estado envolve:

```text
documento
    ↓
versão
    ↓
representação
    ↓
estrutura
    ↓
segmentação
    ↓
unidade de recuperação
    ↓
embedding
    ↓
resultado
    ↓
evidência
```

O chunk continua existindo, mas como uma unidade técnica subordinada a uma cadeia maior.

Ele não é:

* o documento;
* a totalidade do significado;
* a unidade metodológica;
* a evidência em si;
* a justificativa da relevância;
* o estado definitivo do corpus.

A consequência é decisiva:

> **O Snoopy não deve ser modelado como um sistema de chunks, mas como uma infraestrutura de estados documentais dos quais chunks e outras representações são derivados.**

---

# 4. Documento, representação e derivação

Um dos conceitos mais importantes que atravessam toda a Etapa 5 é a distinção entre:

```text
DOCUMENTO
```

e:

```text
REPRESENTAÇÃO DERIVADA DO DOCUMENTO
```

O documento original possui características próprias:

* identidade;
* conteúdo;
* páginas;
* ordem;
* estrutura;
* elementos gráficos;
* tabelas;
* imagens;
* notas;
* referências;
* relações espaciais;
* contexto documental.

Quando o sistema extrai texto, limpa conteúdo, segmenta, gera embeddings ou produz uma resposta, ele não está simplesmente “acessando o documento”.

Ele está produzindo uma nova representação.

Essa representação pode ser útil, mas também pode:

* omitir informação;
* reorganizar elementos;
* alterar relações;
* eliminar contexto;
* introduzir aproximações;
* tornar invisíveis partes do documento.

A arquitetura precisa, portanto, manter a diferença entre:

```text
o documento preservado
```

e:

```text
o documento transformado para determinada finalidade
```

Essa distinção fundamenta 5.1, mas também determina o funcionamento de 5.2, 5.3, 5.4, 5.5 e 5.6.

---

# 5. Preservação como condição de todas as etapas posteriores

A representação canônica não é apenas a primeira etapa da cadeia.

Ela estabelece o limite de confiabilidade das etapas seguintes.

Se uma informação foi destruída antes de ser preservada, nenhuma etapa posterior poderá garantir sua recuperação.

Por exemplo:

```text
PDF
 ↓
extração que elimina tabela
 ↓
chunking
 ↓
embedding
 ↓
busca
```

Nesse fluxo, o problema não está apenas no embedding ou no ranking.

A informação já deixou de existir na representação utilizada pelo sistema.

Por isso, a relação transversal é:

```text
5.1 REPRESENTAÇÃO
        ↓
define o que pode ser preservado
        ↓
5.2 SEGMENTAÇÃO
        ↓
define o que pode ser separado sem perder contexto
        ↓
5.3 RECUPERAÇÃO
        ↓
define o que pode ser encontrado
        ↓
5.4 PROVENIÊNCIA
        ↓
define o que pode ser demonstrado
        ↓
5.5 VERSIONAMENTO
        ↓
define de qual estado isso veio
        ↓
5.6 OPERAÇÃO
        ↓
define em quais condições isso continua confiável
```

A preservação, portanto, não é apenas uma propriedade do armazenamento.

Ela é uma condição de possibilidade para a recuperação e para a avaliação posterior.

---

# 6. Estrutura, segmentação e recuperação dependem umas das outras

A segmentação não pode ser definida independentemente da representação.

Se a representação não preserva estrutura, a segmentação estrutural será limitada.

Se a segmentação não preserva contexto, a recuperação poderá retornar unidades semanticamente incompletas.

Se a recuperação trabalha com unidades mal definidas, a proveniência poderá apontar corretamente para um trecho que, ainda assim, não representa adequadamente o contexto documental.

A relação é:

```text
representação preservadora
        ↓
segmentação estrutural confiável
        ↓
segmentação orientada por finalidade
        ↓
unidades de recuperação identificáveis
        ↓
recuperação mais interpretável
        ↓
evidências contextualizadas
```

Isso mostra por que não basta “melhorar o chunker”.

O chunker só pode ser avaliado adequadamente quando se sabe:

* qual representação recebe;
* qual estrutura está disponível;
* qual finalidade possui;
* que tipo de unidade produz;
* qual contexto preserva;
* como a unidade será recuperada;
* como sua origem será demonstrada.

---

# 7. A recuperação é uma hipótese operacional, não uma decisão metodológica

A recuperação transforma uma pergunta em candidatos documentais.

Ela pode utilizar:

* consultas derivadas;
* similaridade;
* filtros;
* ranking;
* limiares;
* deduplicação;
* expansão contextual;
* diferentes representações.

Mas nenhuma dessas operações determina, por si só, a relevância metodológica de um trecho.

A recuperação deve ser entendida como uma hipótese operacional:

> “Dadas esta pergunta, esta configuração e este corpus, estas unidades parecem potencialmente relevantes.”

Isso implica uma separação entre:

```text
relevância computacional
```

e:

```text
relevância metodológica
```

A primeira pode ser estimada pelo sistema.

A segunda precisa ser examinada pelo pesquisador.

Essa distinção atravessa:

* 5.2, porque a unidade recuperada é uma unidade técnica;
* 5.3, porque similaridade não equivale a relevância;
* 5.4, porque rastrear um trecho não justifica sua importância;
* 5.5, porque resultados dependem do estado do corpus;
* 5.6, porque alterações indevidas no acervo podem modificar o universo da pesquisa.

---

# 8. A evidência não nasce na síntese

O sistema pode produzir uma resposta textual, mas a resposta não deve ser confundida com a evidência.

A cadeia correta é:

```text
documento
    ↓
representação
    ↓
unidade recuperada
    ↓
evidência localizada
    ↓
contexto
    ↓
síntese
```

A síntese é uma representação derivada da evidência recuperada.

Ela não substitui a evidência primária.

Isso produz uma hierarquia:

```text
FONTE ORIGINAL
      ↓
DOCUMENTO
      ↓
EVIDÊNCIA DOCUMENTAL
      ↓
CONTEXTO RECUPERADO
      ↓
SÍNTESE DO SISTEMA
```

Quanto mais distante da fonte, maior a necessidade de explicitar a cadeia de derivação.

A consequência transversal é:

> **A resposta do LLM deve ser tratada como uma camada de apresentação e organização da evidência, não como a própria evidência.**

---

# 9. Grounding resolve apenas uma parte do problema

A presença de citações ou marcadores de evidência reduz o risco de uma resposta completamente desvinculada do corpus.

Mas isso não resolve todos os problemas.

O grounding pode ajudar a responder:

> “De onde veio esta afirmação?”

Ele não garante necessariamente:

* que todas as evidências relevantes foram recuperadas;
* que o trecho selecionado é suficiente;
* que o contexto foi preservado;
* que a interpretação é metodologicamente adequada;
* que o ranking não excluiu documentos importantes;
* que o corpus estava completo;
* que a representação não perdeu informação;
* que a síntese não simplificou excessivamente o argumento.

Por isso:

> **Grounding reduz o problema da invenção; não elimina o problema da seleção.**

Essa é uma das relações mais importantes entre 5.3, 5.4 e a responsabilidade do pesquisador.

---

# 10. Proveniência como propriedade transversal

A proveniência não começa na resposta final.

Ela deve acompanhar o processo desde a origem.

Uma cadeia completa pode ser representada assim:

```text
pergunta original
        ↓
consultas derivadas
        ↓
configuração de recuperação
        ↓
unidades candidatas
        ↓
unidades selecionadas
        ↓
contexto apresentado
        ↓
síntese
        ↓
evidência exibida
        ↓
bloco documental
        ↓
página
        ↓
versão do documento
        ↓
arquivo original
```

Essa cadeia envolve todas as subetapas:

* 5.1 define quais elementos possuem origem;
* 5.2 define como unidades se relacionam com blocos e estrutura;
* 5.3 define como resultados são selecionados;
* 5.4 formaliza a cadeia de rastreabilidade;
* 5.5 preserva os estados e versões;
* 5.6 registra as condições operacionais da produção.

Portanto, proveniência não é apenas “mostrar o PDF”.

Ela significa conseguir reconstruir:

> **qual documento, em qual versão, foi representado de qual maneira, segmentado por qual processo, recuperado por qual configuração e apresentado ao pesquisador como determinada evidência.**

---

# 11. Proveniência documental e proveniência metodológica

A análise transversal reforça uma separação que não pode ser perdida.

## Proveniência documental

Responde:

> “Onde o trecho está no documento?”

Ela envolve:

```text
evidência → unidade → bloco → página → documento → fonte
```

## Proveniência metodológica

Responde:

> “Por que esse trecho foi considerado relevante para a pesquisa?”

Ela envolve:

* pergunta de pesquisa;
* objetivo analítico;
* critérios de inclusão;
* corpus selecionado;
* categorias;
* unidade de registro;
* unidade de contexto;
* interpretação;
* justificativa do pesquisador.

O Snoopy pode apoiar a primeira e fornecer elementos para a segunda.

Mas não deve afirmar que a rastreabilidade documental substitui a justificativa metodológica.

---

# 12. Identidade como conceito transversal

A Etapa 5 mostra que o sistema precisa distinguir várias formas de identidade.

## 12.1. Identidade do documento

Permite reconhecer o documento como objeto documental.

## 12.2. Identidade do conteúdo

Permite verificar se o conteúdo binário mudou.

## 12.3. Identidade da versão

Permite distinguir estados diferentes do mesmo documento.

## 12.4. Identidade da representação

Permite saber qual processo produziu determinada representação.

## 12.5. Identidade da segmentação

Permite distinguir diferentes formas de dividir o mesmo documento.

## 12.6. Identidade da unidade de recuperação

Permite localizar e referenciar uma unidade específica.

## 12.7. Identidade do embedding

Permite saber qual representação vetorial foi produzida sob determinada configuração.

## 12.8. Identidade do resultado

Permite relacionar o resultado a uma consulta, configuração, corpus e estado documental.

Essas identidades não devem ser colapsadas em um único identificador.

O mesmo documento pode possuir:

```text
uma identidade documental
várias versões
várias representações
várias segmentações
vários conjuntos de embeddings
vários resultados de recuperação
```

---

# 13. Estado como conceito transversal

O sistema não trabalha apenas com objetos.

Ele trabalha com objetos em determinados estados.

Um documento pode estar:

* descoberto;
* em processamento;
* concluído;
* incompleto;
* falho;
* ativo;
* atualizado;
* obsoleto;
* removido.

Uma representação pode estar:

* válida;
* incompatível;
* desatualizada;
* aguardando validação.

Uma unidade pode estar:

* associada a uma versão;
* derivada de uma segmentação específica;
* vinculada a um embedding;
* invalidada por alteração da origem.

Um resultado pode estar relacionado a um estado antigo do corpus.

Por isso, a pergunta correta não é apenas:

> “Esse documento existe?”

Mas:

> **“Esse documento existe em qual estado, com quais derivados e sob quais condições de validade?”**

---

# 14. Consistência como relação entre dependências

A consistência não significa apenas ausência de duplicatas.

Ela significa que os objetos relacionados pertencem a estados compatíveis.

Exemplo de estado inconsistente:

```text
documento V2
    ↓
chunks V1
    ↓
embeddings V1
    ↓
resultado apresentado como atual
```

O sistema pode continuar funcionando tecnicamente.

A busca pode retornar resultados.

A interface pode exibir trechos.

Mas o resultado pode estar relacionado a uma versão antiga do documento.

Essa é uma falha particularmente perigosa porque não necessariamente produz erro visível.

A regra transversal é:

> **O sistema não deve apresentar como vigente uma cadeia derivada cujas dependências estejam incompatíveis com o estado documental vigente.**

---

# 15. O corpus é uma condição experimental

O corpus não é apenas um conjunto de arquivos armazenados.

Ele define o universo documental dentro do qual a recuperação ocorre.

Assim:

```text
mesma pergunta
+
mesma configuração
+
corpus diferente
```

pode produzir resultados diferentes.

Isso significa que o corpus precisa ser tratado como parte das condições da recuperação.

A reprodutibilidade exige considerar:

* quais documentos estavam disponíveis;
* quais documentos estavam ativos;
* quais versões estavam vigentes;
* quais documentos foram removidos;
* quais representações foram utilizadas;
* quais unidades estavam recuperáveis;
* quais filtros foram aplicados.

A consequência metodológica é:

> **Uma recuperação só pode ser comparada adequadamente com outra quando as condições documentais e computacionais relevantes forem conhecidas.**

---

# 16. Reprodutibilidade como propriedade em camadas

A Etapa 5 não trata reprodutibilidade como uma propriedade única.

Ela pode ser analisada em camadas:

```text
1. Reprodutibilidade do corpus
2. Reprodutibilidade da configuração
3. Reprodutibilidade da representação
4. Reprodutibilidade da segmentação
5. Reprodutibilidade da recuperação
6. Reprodutibilidade da evidência
7. Reprodutibilidade da análise
8. Reprodutibilidade da síntese
```

Essas camadas não possuem o mesmo grau de controle.

O sistema pode buscar maior estabilidade em:

* corpus;
* configuração;
* representação;
* segmentação;
* recuperação;
* evidências.

Já a interpretação metodológica continua dependente do pesquisador.

A síntese por LLM também pode apresentar variabilidade própria.

Por isso, o objetivo prioritário do Snoopy deve ser:

> **tornar a recuperação e a cadeia de evidências suficientemente identificáveis e avaliáveis, sem prometer determinismo absoluto da interpretação ou da linguagem gerada.**

---

# 17. Segurança como integridade metodológica

A análise transversal mostra que segurança não deve ser reduzida à proteção de dados pessoais ou à prevenção de invasões.

Ela também protege a integridade da pesquisa.

Um acesso indevido pode:

* alterar o corpus;
* remover documentos;
* inserir documentos;
* modificar permissões;
* misturar acervos;
* expor documentos privados;
* alterar resultados;
* contaminar caches;
* modificar estados de processamento.

Assim, segurança e metodologia estão ligadas.

A autorização de acesso a um acervo não é apenas uma decisão administrativa.

Ela define quem pode alterar as condições documentais da pesquisa.

---

# 18. Acervo como fronteira de segurança e de sentido

O acervo possui dupla função.

Ele é:

```text
fronteira de autorização
```

e:

```text
fronteira de escopo documental
```

Misturar documentos de acervos diferentes pode ser um problema de segurança e também um problema metodológico.

Da mesma forma, permitir que um usuário consulte documentos fora de seu escopo não é apenas uma falha de controle de acesso.

É uma alteração indevida do universo documental sobre o qual a pesquisa está sendo realizada.

Por isso, a arquitetura deve distinguir explicitamente:

* propriedade;
* administração;
* visibilidade;
* compartilhamento;
* origem;
* escopo de busca;
* permissão de processamento;
* permissão de exclusão.

---

# 19. Operação como condição de validade

A operação cotidiana do sistema pode alterar seus resultados.

Entre os fatores relevantes estão:

* concorrência entre workers;
* retries;
* falhas parciais;
* interrupções;
* limites de memória;
* indisponibilidade do Drive;
* indisponibilidade do banco;
* mudanças de configuração;
* cache;
* processamento incompleto;
* mistura entre desenvolvimento e produção.

Por isso, operação não é uma camada externa à arquitetura documental.

Ela determina se o sistema consegue manter válidas as relações estabelecidas nas etapas anteriores.

Um documento parcialmente processado, por exemplo, não deve ser tratado como plenamente disponível.

Um job repetido não deve criar estados duplicados ou incompatíveis.

Um worker interrompido não deve deixar o sistema acreditando que o documento foi concluído.

---

# 20. A publicação é diferente do processamento

A análise transversal permite distinguir dois momentos:

```text
PROCESSAMENTO
```

e:

```text
PUBLICAÇÃO
```

Durante o processamento, o sistema pode estar produzindo:

* representação;
* estrutura;
* segmentação;
* embeddings;
* metadados;
* relações de proveniência.

Mas esses elementos ainda podem estar incompletos.

A publicação ocorre somente quando o conjunto necessário estiver coerente:

```text
representação válida
        +
segmentação válida
        +
embeddings compatíveis
        +
proveniência preservada
        +
estado consistente
        ↓
publicação como ativo
```

Essa distinção evita que o sistema disponibilize parcialmente uma versão documental como se ela estivesse completa.

---

# 21. Cache como estado derivado

O cache costuma parecer uma questão de desempenho.

Mas, no Snoopy, ele também possui implicações metodológicas e de segurança.

Um resultado armazenado depende de:

* pergunta;
* corpus;
* versão documental;
* configuração de recuperação;
* representação;
* segmentação;
* embeddings;
* filtros;
* ranking.

Se qualquer uma dessas condições mudar, o cache pode deixar de representar o mesmo processo.

Além disso, um cache mal isolado pode expor resultados de um acervo a outro usuário.

Portanto:

> **Cache é estado derivado, não verdade permanente.**

Ele deve possuir:

* escopo;
* identidade;
* condições de validade;
* regras de invalidação;
* isolamento;
* relação com o estado do corpus.

---

# 22. A responsabilidade do pesquisador permanece fora do motor

A arquitetura especificada não transforma o Snoopy em um sistema automático de Análise de Conteúdo.

O pesquisador continua responsável por:

* definir o problema;
* delimitar o corpus;
* estabelecer critérios;
* escolher unidades de registro;
* definir unidades de contexto;
* construir categorias;
* interpretar os dados;
* avaliar a relevância metodológica;
* justificar conclusões.

O Snoopy pode apoiar:

* localização;
* recuperação;
* organização;
* comparação;
* rastreabilidade;
* preservação;
* exploração documental;
* apresentação de evidências.

Mas não deve confundir essas funções com a própria análise.

A cadeia metodológica continua sendo:

```text
pesquisa
    ↓
pergunta
    ↓
corpus
    ↓
recuperação
    ↓
evidência
    ↓
decisão do pesquisador
    ↓
análise
```

---

# 23. Relações entre as subetapas

| Relação   | Consequência                                                                              |
| --------- | ----------------------------------------------------------------------------------------- |
| 5.1 → 5.2 | A segmentação depende da informação preservada na representação canônica.                 |
| 5.2 → 5.3 | A recuperação depende de unidades bem definidas, contextualizadas e identificáveis.       |
| 5.3 → 5.4 | Todo resultado recuperado precisa poder ser relacionado à sua origem.                     |
| 5.4 → 5.5 | A proveniência precisa sobreviver a atualizações, reprocessamentos e mudanças de versão.  |
| 5.5 → 5.6 | Estados inconsistentes precisam ser impedidos por controle operacional e de concorrência. |
| 5.6 → 5.1 | Segurança, falhas e limites operacionais não podem destruir a preservação documental.     |
| 5.6 → 5.3 | Isolamento de acervos e ambientes influencia diretamente o universo da recuperação.       |
| 5.5 → 5.3 | A recuperação precisa identificar o estado do corpus e dos derivados utilizados.          |
| 5.4 → 5.3 | A seleção de evidências deve ser reconstruível, não apenas o resultado final.             |
| 5.1 → 5.4 | A granularidade da proveniência depende da granularidade preservada na representação.     |

---

# 24. Princípios transversais consolidados

A partir da articulação das seis subetapas, a Etapa 5 estabelece os seguintes princípios gerais.

## 24.1. Princípio da preservação

Nenhuma transformação derivada deve substituir silenciosamente o documento original.

## 24.2. Princípio da derivação explícita

Toda representação produzida pelo sistema deve ser reconhecida como derivada de outra representação ou do documento original.

## 24.3. Princípio da finalidade

Toda segmentação, representação ou recuperação deve possuir uma finalidade identificável.

## 24.4. Princípio da identidade

Documentos, versões, representações, unidades e resultados devem ser distinguíveis.

## 24.5. Princípio da rastreabilidade

Toda evidência deve poder ser relacionada à sua origem documental.

## 24.6. Princípio da temporalidade

Resultados devem ser associados ao estado documental e computacional que os produziu.

## 24.7. Princípio da consistência

Estados incompatíveis não devem ser tratados como uma única realidade vigente.

## 24.8. Princípio do corpus

O corpus é parte das condições experimentais da recuperação.

## 24.9. Princípio da separação epistemológica

Recuperação, síntese e interpretação não são a mesma operação.

## 24.10. Princípio da responsabilidade metodológica

A decisão sobre relevância, unidade, categoria e interpretação permanece com o pesquisador.

## 24.11. Princípio da segurança metodológica

Controle de acesso e isolamento preservam também a integridade do corpus e da pesquisa.

## 24.12. Princípio da operacionalidade observável

O sistema deve permitir reconstruir o que ocorreu, sob quais condições e com quais resultados.

---

# 25. Arquitetura transversal

A arquitetura resultante pode ser representada em duas dimensões.

## 25.1. Fluxo principal

```text
DOCUMENTO ORIGINAL
        ↓
REPRESENTAÇÃO CANÔNICA
        ↓
ESTRUTURAÇÃO
        ↓
SEGMENTAÇÃO
        ↓
UNIDADES DE RECUPERAÇÃO
        ↓
RECUPERAÇÃO
        ↓
EVIDÊNCIAS
        ↓
SÍNTESE
        ↓
PESQUISADOR
        ↓
INTERPRETAÇÃO
```

## 25.2. Camadas transversais

```text
┌─────────────────────────────────────────────────────┐
│ IDENTIDADE                                          │
│ PROVENIÊNCIA                                        │
│ VERSIONAMENTO                                       │
│ CONSISTÊNCIA                                        │
│ CORPUS E ESCOPO                                     │
│ SEGURANÇA E ISOLAMENTO                              │
│ OBSERVABILIDADE                                     │
│ RECUPERABILIDADE                                    │
└─────────────────────────────────────────────────────┘
```

Essas camadas não são etapas adicionais do fluxo.

Elas acompanham todas as etapas.

---

# 26. Formulação transversal consolidada

A síntese transversal da Etapa 5 pode ser expressa da seguinte maneira:

> **O Snoopy-RAG deverá ser concebido como uma infraestrutura documental orientada por estados, na qual o documento original seja preservado por meio de uma representação canônica, as representações derivadas sejam produzidas para finalidades explícitas, as unidades de recuperação mantenham identidade e contexto, os resultados sejam rastreáveis até suas fontes, e todos os derivados permaneçam associados a versões documentais e computacionais compatíveis. A recuperação deverá ser tratada como operação de localização de candidatos, distinta da relevância metodológica e da interpretação. Segurança, isolamento, persistência, versionamento, consistência e observabilidade deverão atravessar todo o sistema, impedindo que alterações, falhas ou acessos indevidos modifiquem silenciosamente as condições documentais e metodológicas da pesquisa.**

---

# 27. Axiomas transversais da Etapa 5

Os cinco axiomas estabelecidos anteriormente podem agora ser ampliados para a arquitetura completa.

### Axioma 1

> **Recuperação não é análise.**

### Axioma 2

> **Similaridade semântica não é relevância metodológica.**

### Axioma 3

> **Trecho recuperado não é automaticamente unidade de registro ou unidade de contexto.**

### Axioma 4

> **Rastreabilidade da fonte não é proveniência metodológica completa.**

### Axioma 5

> **Síntese do LLM é representação derivada, não evidência primária.**

### Axioma 6

> **Representação textual derivada não deve substituir silenciosamente o documento preservado.**

### Axioma 7

> **Um resultado só é interpretável quando seu estado documental e computacional pode ser identificado.**

### Axioma 8

> **Segurança e consistência operacional fazem parte da integridade metodológica.**

### Axioma 9

> **O corpus é uma condição da recuperação, não apenas um repositório de arquivos.**

### Axioma 10

> **O pesquisador continua sendo a autoridade sobre a decisão metodológica.**

---

# 28. Critérios para avaliar a futura implementação

A implementação decorrente da Etapa 5 deverá ser avaliada em pelo menos seis dimensões.

## 28.1. Integridade documental

O sistema preserva adequadamente as características relevantes dos documentos?

## 28.2. Qualidade da segmentação

As unidades produzidas preservam estrutura, contexto e continuidade?

## 28.3. Qualidade da recuperação

O sistema recupera unidades potencialmente relevantes dentro do corpus definido?

## 28.4. Rastreabilidade

É possível reconstruir a origem das evidências apresentadas?

## 28.5. Consistência temporal

Os resultados correspondem a estados documentais e computacionais compatíveis?

## 28.6. Integridade operacional

Falhas, concorrência, isolamento e reprocessamentos preservam as propriedades anteriores?

Essas dimensões devem ser avaliadas separadamente.

Um sistema pode, por exemplo:

* recuperar bons trechos, mas não preservar sua origem;
* possuir boa proveniência, mas recuperar unidades inadequadas;
* preservar documentos, mas misturar versões;
* ter boa busca, mas permitir vazamento entre acervos;
* funcionar em condições normais, mas falhar sob concorrência;
* produzir respostas convincentes, mas sem garantir completude da recuperação.

---

# 29. O que este registro impede que seja confundido

Este registro transversal impede algumas interpretações equivocadas sobre o projeto.

O Snoopy não deve ser descrito como:

* um substituto do pesquisador;
* um sistema automático de Análise de Conteúdo;
* um gerador de unidades de registro;
* um mecanismo que determina relevância metodológica;
* uma ferramenta que garante reprodução literal de respostas;
* um sistema que preserva perfeitamente qualquer PDF sem validação;
* um RAG cuja qualidade depende apenas do modelo de embedding;
* um banco de chunks sem história documental;
* uma interface de perguntas e respostas desvinculada do corpus.

A formulação mais adequada continua sendo:

> **O Snoopy-RAG é uma ferramenta de apoio à pesquisa que utiliza recuperação semântica sobre um corpus documental para localizar evidências textuais relevantes, preservar sua origem e fornecer essas evidências ao pesquisador para posterior análise e interpretação.**

A Etapa 5 amplia essa definição ao estabelecer as condições arquiteturais necessárias para que essa função seja exercida de maneira mais íntegra, rastreável e consistente.

---

# 30. Transição para a Etapa 6

A Etapa 5 termina com requisitos e princípios.

A Etapa 6 deverá investigar como realizá-los tecnicamente.

A transição será:

```text
REQUISITOS METODOLÓGICOS
        ↓
CRITÉRIOS TÉCNICOS
        ↓
ALTERNATIVAS DE IMPLEMENTAÇÃO
        ↓
COMPARAÇÃO
        ↓
SELEÇÃO JUSTIFICADA
        ↓
DESENHO TÉCNICO
        ↓
IMPLEMENTAÇÃO
        ↓
TESTES
        ↓
REANÁLISE
```

A Etapa 6 não deverá começar escolhendo uma ferramenta por preferência ou familiaridade.

Ela deverá começar perguntando:

* quais requisitos cada alternativa atende;
* quais perdas cada alternativa introduz;
* quais custos produz;
* quais limitações possui;
* como preserva a proveniência;
* como suporta versionamento;
* como permite avaliação;
* como se comporta sob falhas;
* como se integra ao ambiente real do Snoopy.

A sequência geral do projeto passa a ser:

```text
ANALISAR
   ↓
ESPECIFICAR
   ↓
DOCUMENTAR
   ↓
PROPOR
   ↓
IMPLEMENTAR
   ↓
TESTAR
   ↓
REANALISAR
   ↓
DOCUMENTAR NOVAMENTE
```

---

# 31. Conclusão

O Registro Mestre Transversal mostra que a Etapa 5 não é apenas uma coleção de melhorias possíveis.

Ela estabelece uma mudança de fundamento.

O Snoopy deixa de ser compreendido prioritariamente como:

```text
um sistema que transforma PDFs em chunks e respostas
```

e passa a ser compreendido como:

```text
uma infraestrutura que preserva estados documentais,
produz representações derivadas,
recupera evidências,
mantém sua proveniência
e protege a integridade dessas operações ao longo do tempo.
```

A propriedade central do sistema não é simplesmente “responder bem”.

É conseguir responder de maneira que o pesquisador possa compreender:

* qual documento foi utilizado;
* qual versão estava vigente;
* qual representação foi produzida;
* como o documento foi segmentado;
* por que determinada unidade foi recuperada;
* de onde veio a evidência;
* quais configurações participaram do processo;
* quais condições operacionais estavam presentes;
* e quais decisões continuam sendo responsabilidade humana.

A síntese transversal da Etapa 5 pode ser reduzida à seguinte formulação:

> **O Snoopy deve preservar o documento antes de transformá-lo, identificar cada derivação, recuperar unidades rastreáveis, manter estados compatíveis, proteger o corpus e apresentar evidências sem confundir processamento computacional com interpretação metodológica.**

É essa articulação entre preservação, derivação, recuperação, proveniência, estado, consistência e responsabilidade que constitui o verdadeiro núcleo arquitetural do novo Snoopy-RAG.
