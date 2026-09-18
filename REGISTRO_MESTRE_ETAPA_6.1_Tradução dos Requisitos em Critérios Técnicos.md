# Registro Mestre da Análise — Snoopy-RAG

# Etapa 6.1 — Tradução dos Requisitos em Critérios Técnicos

**Status:** Registro Mestre da Etapa 6.1 — consolidado para transição à investigação tecnológica da Etapa 6.2.

**Natureza:** Especificação de critérios técnicos derivados dos requisitos metodológicos e arquiteturais da Etapa 5. Não constitui ainda seleção de tecnologias, projeto técnico definitivo ou implementação.

**Função:** Estabelecer uma ponte formal entre os requisitos definidos na Etapa 5 e a investigação das alternativas tecnológicas que será realizada na Etapa 6.2.

**Escopo:** Requisitos das Etapas 5.1, 5.2, 5.3, 5.4, 5.5 e 5.6.

---

# 1. Finalidade da Etapa 6.1

A Etapa 5 estabeleceu os problemas metodológicos e arquiteturais que a nova arquitetura do Snoopy-RAG deverá enfrentar.

Esses problemas foram organizados em seis dimensões principais:

```text
5.1 REPRESENTAÇÃO CANÔNICA
5.2 SEGMENTAÇÃO
5.3 RECUPERAÇÃO
5.4 PROVENIÊNCIA
5.5 PERSISTÊNCIA, VERSIONAMENTO E CONSISTÊNCIA
5.6 SEGURANÇA, ISOLAMENTO E OPERAÇÃO
```

A Etapa 6 inicia a transição entre essa especificação arquitetural e sua realização tecnológica.

Essa transição não deve ocorrer diretamente da seguinte maneira:

```text
requisito
↓
biblioteca encontrada
↓
adaptação do projeto
```

A abordagem definida para a Etapa 6 é:

```text
REQUISITO
↓
CRITÉRIO TÉCNICO
↓
ALTERNATIVAS TECNOLÓGICAS
↓
COMPARAÇÃO
↓
ESCOLHA
↓
PROJETO TÉCNICO
↓
IMPLEMENTAÇÃO
↓
VALIDAÇÃO
```

A Etapa 6.1 ocupa especificamente o segundo nível dessa cadeia.

Seu objetivo é transformar cada requisito relevante em uma propriedade técnica observável e investigável que uma solução tecnológica deverá possuir, total ou parcialmente, para que o requisito arquitetural correspondente possa ser satisfeito.

---

# 2. Princípio fundamental: critério técnico não é tecnologia

Um critério técnico não deve ser formulado como uma escolha antecipada de ferramenta.

Por exemplo, diante do requisito de preservação da localização espacial de elementos documentais, não se deve formular imediatamente:

> “usar determinada biblioteca de processamento de PDF”.

O critério deve ser formulado como:

> **A solução deve representar elementos documentais com localização espacial recuperável e associável ao conteúdo correspondente, permitindo preservar essa informação nas derivações posteriores.**

Somente depois, durante a Etapa 6.2, deverão ser investigadas alternativas capazes de satisfazer esse critério.

Essa distinção impede que a arquitetura V3 seja construída a partir da adaptação do problema à primeira tecnologia encontrada.

O procedimento deverá permanecer:

```text
problema arquitetural
↓
propriedade necessária
↓
critério técnico
↓
investigação das alternativas
↓
avaliação das capacidades e limitações
↓
decisão arquitetural
```

Portanto, a Etapa 6.1 **não escolhe bibliotecas, modelos, bancos, formatos ou serviços**.

Ela define aquilo que essas tecnologias deverão demonstrar.

---

# 3. Natureza dos critérios técnicos

Os requisitos da Etapa 5 não produzem necessariamente critérios independentes e equivalentes.

Durante a investigação da Etapa 6.2, os critérios deverão ser compreendidos em pelo menos três categorias.

## 3.1. Critérios de existência

São propriedades que uma solução precisa possuir para desempenhar determinada função arquitetural.

Se uma alternativa não possui uma capacidade essencial para determinada função, ela pode ser considerada inadequada para essa função.

Exemplo:

> Uma solução destinada à representação documental não pode ser considerada suficiente para a representação canônica pretendida se não possuir qualquer mecanismo para preservar a relação entre conteúdo e sua localização documental quando essa informação é relevante.

---

## 3.2. Critérios de qualidade

São propriedades que podem ser atendidas em diferentes graus.

Uma solução pode, por exemplo, representar tabelas, mas apresentar perdas em determinados tipos de tabelas.

Nesse caso, a questão não é simplesmente:

> “suporta tabelas?”

mas:

> “que tipos de tabelas preserva, quais informações conserva, quais informações perde e em quais condições?”

Portanto, a avaliação deverá considerar a qualidade da capacidade, e não apenas sua existência nominal.

---

## 3.3. Critérios de custo e operabilidade

Uma alternativa pode satisfazer determinado requisito e ainda apresentar compromissos relevantes relacionados a:

* tempo de processamento;
* custo financeiro;
* memória;
* CPU;
* GPU;
* armazenamento;
* dependência de serviços externos;
* limites operacionais;
* complexidade de integração;
* complexidade de manutenção;
* disponibilidade;
* escalabilidade;
* requisitos de infraestrutura.

Essas propriedades não eliminam automaticamente uma alternativa.

Elas constituem elementos para sua comparação posterior.

---

# 4. Critérios técnicos para representação documental

A representação canônica constitui a primeira base técnica da arquitetura V3.

Os requisitos RCAN1–RCAN22 estabelecem que o documento deve ser preservado antes das transformações destinadas à recuperação, segmentação ou síntese. Entre as propriedades requeridas estão fidelidade documental, estrutura, relações espaciais, conteúdo multimodal, reprocessamento, persistência, rastreabilidade e separação entre representação documental e análise metodológica.

---

## CT-REP-01 — Fidelidade documental

A solução deve conseguir preservar, dentro de limites tecnicamente justificáveis:

* conteúdo textual;
* páginas;
* ordem de leitura;
* blocos;
* estrutura lógica;
* elementos complexos;
* relações espaciais;
* elementos não textuais;
* informação necessária à inspeção e reconstrução humana.

Fidelidade documental não significa necessariamente reprodução visual perfeita do PDF.

O objetivo é preservar características documentais relevantes para as funções posteriores do sistema.

---

## CT-REP-02 — Representação de conteúdo não textual

A solução deve possuir mecanismos para representar, identificar e relacionar elementos que não sejam simplesmente texto linear, incluindo:

* tabelas;
* figuras;
* imagens;
* gráficos;
* legendas;
* outros elementos documentais relevantes.

O critério não exige que todos esses elementos sejam imediatamente convertidos em texto ou incorporados ao mecanismo vetorial.

O requisito fundamental é impedir que elementos documentais relevantes desapareçam simplesmente porque uma determinada representação derivada é textual.

---

## CT-REP-03 — Preservação de relações espaciais

Quando disponíveis na fonte, informações espaciais relevantes devem permanecer representáveis.

Isso inclui, conforme aplicável:

* posição;
* página;
* região;
* relações espaciais;
* coordenadas;
* bounding boxes ou representação equivalente.

---

## CT-REP-04 — Distinção entre conteúdo nativo e OCR

A solução deve distinguir:

```text
conteúdo textual nativo
```

de:

```text
conteúdo textual obtido por OCR
```

Essa condição deverá permanecer identificável nas representações e relações de proveniência posteriores.

---

## CT-REP-05 — Separação entre representação canônica e representações derivadas

A arquitetura tecnológica deve permitir distinguir a representação canônica de:

* texto puro;
* Markdown;
* chunks;
* embeddings;
* respostas de modelos de linguagem;
* outras representações derivadas.

Nenhuma dessas representações deve substituir silenciosamente a representação documental preservada.

---

## CT-REP-06 — Reprocessabilidade

A solução deve permitir produzir novas representações derivadas a partir da representação preservada sem exigir necessariamente nova extração do documento original.

O objetivo é possibilitar, por exemplo:

```text
DOCUMENTO
↓
REPRESENTAÇÃO PRESERVADA
↓
SEGMENTAÇÃO A
↓
EMBEDDING A
```

e posteriormente:

```text
REPRESENTAÇÃO PRESERVADA
↓
SEGMENTAÇÃO B
↓
EMBEDDING B
```

sem que a segunda estratégia exija necessariamente retornar ao PDF original.

---

## CT-REP-07 — Rastreabilidade da derivação

Representações derivadas relevantes devem possuir relação identificável com sua origem.

Deve ser possível reconstruir, conforme aplicável:

```text
representação derivada
↓
representação preservada
↓
documento
```

Essa capacidade será posteriormente integrada à proveniência completa do sistema.

---

## CT-REP-08 — Utilidade computacional e humana

A representação persistida deve ser suficientemente estruturada para atender tanto:

* processamento computacional;
* quanto inspeção, auditoria e reconstrução humana.

A representação não deve ser projetada exclusivamente para consumo automático.

---

## CT-REP-09 — Custo e praticidade

Alternativas de representação deverão ser avaliadas considerando simultaneamente:

```text
fidelidade
×
armazenamento
×
processamento
×
complexidade operacional
×
reutilização
```

A alternativa tecnicamente mais fiel não será automaticamente considerada a melhor solução arquitetural.

---

## CT-REP-10 — Reversibilidade prática

As transformações derivadas devem permitir reconstruir a relação:

```text
derivado
↓
origem preservada
↓
documento
```

Não é necessário que toda transformação seja matematicamente reversível.

O requisito é a existência de uma relação prática de retorno à origem.

---

# 5. Critérios técnicos para segmentação

A segmentação deverá ser tratada como uma transformação derivada da representação documental preservada.

A arquitetura não deverá confundir:

```text
documento
```

com:

```text
unidade de recuperação
```

nem:

```text
unidade de recuperação
=
unidade de registro
```

Os critérios abaixo derivam dos requisitos RSEG1–RSEG24.

---

## CT-SEG-01 — Derivação a partir da representação preservada

A segmentação deve consumir a representação documental preservada e não substituí-la.

---

## CT-SEG-02 — Consciência estrutural

A solução deve permitir trabalhar com a estrutura documental antes da definição das unidades de recuperação.

Deve ser possível representar relações como:

```text
documento
↓
página
↓
seção
↓
subseção
↓
parágrafo
↓
elemento complexo
```

conforme a estrutura efetivamente existente no documento.

---

## CT-SEG-03 — Continuidade documental

A solução deve permitir preservar continuidade entre elementos relacionados, inclusive quando a continuidade atravessa:

* páginas;
* blocos;
* seções;
* outros limites físicos.

Uma quebra física não deve ser tratada automaticamente como quebra semântica.

---

## CT-SEG-04 — Contexto estrutural separado do conteúdo

A solução deve permitir associar contexto estrutural às unidades sem necessariamente incorporá-lo ao conteúdo textual original.

Exemplo:

```text
seção = Resultados
subseção = 3.2
página = 7
```

Esse contexto poderá posteriormente ser utilizado para recuperação, apresentação ou síntese, sem que seja confundido com o conteúdo original.

---

## CT-SEG-05 — Preservação de elementos complexos

A segmentação não deve depender da destruição de:

* tabelas;
* figuras;
* imagens;
* notas;
* referências;
* legendas;
* relações multimodais.

---

## CT-SEG-06 — Tamanho como restrição operacional

Limites de tamanho devem ser tratados como restrições operacionais e não como definição automática de significado.

Uma quantidade fixa de caracteres ou tokens não constitui, por si só, uma unidade semanticamente adequada.

---

## CT-SEG-07 — Finalidade configurável

A definição das unidades de recuperação deve poder ser avaliada de acordo com a finalidade da recuperação.

Não deverá ser presumida a existência de uma única segmentação universalmente ótima.

---

## CT-SEG-08 — Contextualização sem fusão

A arquitetura deve permitir acrescentar contexto adicional a uma unidade sem destruir a distinção entre:

```text
unidade primária
```

e:

```text
contexto complementar
```

---

## CT-SEG-09 — Overlap controlável

Quando houver sobreposição entre unidades, ela deverá possuir finalidade definida e ser avaliável quanto a:

* redundância;
* cobertura;
* recuperação;
* efeitos sobre ranking;
* duplicação de evidências.

---

## CT-SEG-10 — Identidade independente

Cada unidade de recuperação deverá possuir identidade relacionada à:

```text
versão documental
+
representação de origem
+
estratégia de segmentação
```

---

## CT-SEG-11 — Independência metodológica

Nenhuma unidade de recuperação deverá ser automaticamente classificada como:

* unidade de registro;
* unidade de contexto;
* categoria metodológica;
* evidência metodológica definitiva.

A unidade de recuperação é uma construção computacional destinada à recuperação documental.

---

## CT-SEG-12 — Reprocessamento independente

A arquitetura deve permitir testar diferentes estratégias de segmentação sobre a mesma representação preservada.

Isso deverá possibilitar comparação entre estratégias sem exigir necessariamente nova extração dos documentos.

---

# 6. Critérios técnicos para recuperação

A recuperação constitui a camada intermediária entre a pergunta do pesquisador e as evidências documentais.

A tecnologia de recuperação não deverá ser tratada como responsável pela interpretação metodológica da pesquisa.

Os critérios abaixo derivam dos requisitos RRET1–RRET30.

---

## CT-RET-01 — Recuperação desacoplada da síntese

A arquitetura deve permitir executar:

```text
consulta
↓
recuperação
↓
resultados
```

sem depender obrigatoriamente de:

```text
LLM
↓
síntese
```

A qualidade da recuperação deve poder ser avaliada independentemente da qualidade textual da resposta gerada.

---

## CT-RET-02 — Consulta reproduzível

A execução deve preservar:

* consulta original;
* consultas derivadas;
* estratégia de decomposição, quando utilizada.

---

## CT-RET-03 — Controle explícito do corpus

A recuperação deve operar sobre um universo documental identificável.

O sistema não deve depender da ideia implícita de que “todos os documentos disponíveis” constituem automaticamente o corpus da busca.

---

## CT-RET-04 — Parâmetros de recuperação controláveis

A solução deve permitir controlar e registrar, conforme a estratégia utilizada:

* limiar;
* quantidade de resultados;
* filtros;
* métrica;
* parâmetros de ranking;
* demais parâmetros relevantes.

---

## CT-RET-05 — Ranking estável e reconstruível

A solução deve permitir determinar as condições que fizeram uma unidade aparecer antes de outra.

Quando houver empate ou equivalência, deverá existir mecanismo de desempate suficientemente estável para permitir reconstrução da execução.

---

## CT-RET-06 — Deduplicação por identidade

A arquitetura deve distinguir:

```text
mesma unidade recuperada múltiplas vezes
```

de:

```text
unidades diferentes com conteúdo semelhante
```

A deduplicação não deverá destruir evidências distintas simplesmente porque possuem texto semelhante.

---

## CT-RET-07 — Contextualização desacoplada

A recuperação deve permitir acrescentar contexto sem apagar a distinção entre:

```text
unidade recuperada
```

e:

```text
contexto adicional
```

---

## CT-RET-08 — Versionamento da representação vetorial

A solução deve permitir identificar qual representação vetorial foi produzida a partir de determinada unidade e sob qual configuração.

A relação deverá ser preservada como:

```text
unidade
↓
embedding
↓
versão/configuração
```

---

## CT-RET-09 — Perfil de recuperação reconstruível

Uma execução deverá poder ser caracterizada suficientemente para reconstrução posterior, incluindo, conforme aplicável:

```text
consulta
+
consultas derivadas
+
corpus
+
versão documental
+
representação
+
segmentação
+
embedding
+
métrica
+
threshold
+
top-K
+
ranking
+
deduplicação
+
contexto
```

---

## CT-RET-10 — Estratégias intercambiáveis

A arquitetura não deverá tornar impossível a investigação de diferentes estratégias de recuperação.

Entre as possibilidades a serem investigadas posteriormente estão:

* recuperação semântica;
* recuperação lexical;
* recuperação híbrida;
* reranking;
* estratégias multicamadas;
* outras combinações pertinentes.

A existência desse critério não significa que todas essas estratégias deverão ser implementadas.

---

## CT-RET-11 — Avaliação independente

A tecnologia deverá permitir medir a qualidade da recuperação sem depender da resposta final produzida pelo modelo de linguagem.

---

## CT-RET-12 — Reprodutibilidade operacional

As condições necessárias para repetir uma recuperação deverão ser registráveis.

O objetivo não é necessariamente garantir que o modelo produza exatamente a mesma frase em toda execução, mas permitir reproduzir ou avaliar a estabilidade da recuperação de evidências sob condições controladas.

---

## CT-RET-13 — Separação entre similaridade e relevância

A arquitetura não deverá transformar automaticamente:

```text
score de similaridade
```

em:

```text
relevância metodológica
```

O embedding localiza similaridade; o mecanismo de recuperação seleciona candidatos; a decisão sobre relevância metodológica permanece com o pesquisador.

---

# 7. Critérios técnicos para proveniência

A proveniência deverá ser tratada como uma cadeia de relações e não como um simples campo de origem associado ao resultado final.

A arquitetura deverá permitir reconstruir, quando aplicável:

```text
pergunta
↓
consulta derivada
↓
configuração de recuperação
↓
resultado
↓
unidade de recuperação
↓
elemento estrutural
↓
página
↓
versão documental
↓
documento original
```

---

## CT-PROV-01 — Identidade persistente dos elementos

Documentos, versões, representações, unidades e resultados deverão possuir identidades distinguíveis.

Não deverá existir uma única identidade genérica responsável por representar todos esses estados.

---

## CT-PROV-02 — Cadeia hierárquica

A solução deverá permitir reconstruir relações como:

```text
documento
↓
versão
↓
página
↓
bloco
↓
elemento
↓
unidade
```

conforme aplicável.

---

## CT-PROV-03 — Proveniência das derivações

De uma representação derivada deverá ser possível chegar à representação preservada que lhe deu origem.

---

## CT-PROV-04 — Proveniência da recuperação

De um resultado de busca deverá ser possível reconstruir:

```text
pergunta
↓
consulta derivada
↓
configuração
↓
resultado
↓
unidade
↓
documento
```

---

## CT-PROV-05 — Proveniência da contextualização

A tecnologia deverá distinguir:

```text
evidência primária
```

de:

```text
contexto acrescentado
```

---

## CT-PROV-06 — Proveniência da síntese

Quando uma síntese for produzida, deverá existir relação identificável entre a resposta e as evidências fornecidas ao modelo.

---

## CT-PROV-07 — Acesso à fonte

Uma evidência deverá permitir retornar à fonte documental correspondente quando esta estiver disponível.

---

## CT-PROV-08 — Proveniência multimodal e OCR

A cadeia de proveniência deverá ser capaz de sobreviver a derivações provenientes de:

* OCR;
* tabelas;
* imagens;
* gráficos;
* outros elementos não textuais.

---

## CT-PROV-09 — Auditoria bidirecional

A solução deverá permitir tanto:

```text
evidência
↓
origem
```

quanto:

```text
documento
↓
representações derivadas produzidas
```

---

## CT-PROV-10 — Proveniência temporal

Resultados históricos deverão permanecer associados ao estado documental sob o qual foram produzidos.

---

## CT-PROV-11 — Separação da proveniência metodológica

A tecnologia não deverá apresentar rastreabilidade documental como se fosse proveniência metodológica completa.

As decisões metodológicas do pesquisador deverão permanecer distinguíveis das operações automáticas do sistema.

---

# 8. Critérios técnicos para persistência, versionamento e consistência

Persistência, versionamento e consistência constituem dimensões distintas.

A questão técnica não é apenas:

> “o dado pode ser armazenado?”

É também:

> “qual estado documental esse dado representa?”

> “de qual versão foi derivado?”

> “quais dependências possui?”

> “ainda é compatível com o estado vigente?”

---

## CT-PVC-01 — Identidade temporal

A solução deve distinguir diferentes versões de um documento.

Uma atualização não deverá necessariamente significar simplesmente sobrescrever o estado anterior.

---

## CT-PVC-02 — Dependências explícitas

Deve ser possível representar relações como:

```text
Documento V2
↓
Representação R2
↓
Segmentação S3
↓
Unidades
↓
Embedding E4
```

e identificar as dependências entre esses estados.

---

## CT-PVC-03 — Detecção de incompatibilidade

A solução deve permitir identificar quando uma alteração na origem torna determinadas representações derivadas potencialmente incompatíveis ou obsoletas.

---

## CT-PVC-04 — Reprocessamento seletivo

A arquitetura deve permitir reconstruir apenas as partes necessárias quando isso for tecnicamente possível.

---

## CT-PVC-05 — Publicação coerente

Estados parcialmente processados não devem ser apresentados como plenamente disponíveis.

A disponibilidade pública de um documento deve depender da consistência das representações e dependências necessárias.

---

## CT-PVC-06 — Concorrência e idempotência

A infraestrutura deve suportar:

* processamento concorrente;
* retries;
* interrupções;
* repetição de jobs;

sem produzir duplicação ou estados incompatíveis.

---

## CT-PVC-07 — Histórico reconstruível

Resultados históricos deverão permanecer associados às versões documentais e configurações correspondentes.

---

## CT-PVC-08 — Cache dependente do estado

O cache deverá ser tratado como estado derivado.

Sua validade deverá depender, conforme aplicável, de:

```text
consulta
+
corpus/versão
+
configuração de recuperação
+
dependências relevantes
```

---

## CT-PVC-09 — Integridade verificável

A arquitetura deverá permitir verificar relações como:

```text
embedding
→ unidade correta
→ representação correta
→ versão documental correta
```

---

## CT-PVC-10 — Recuperação de falhas

Após uma falha, o sistema deverá conseguir retornar a um estado documental e computacional conhecido e consistente.

Não deverá ser suficiente simplesmente interromper o processamento sem que o estado resultante seja identificável.

---

# 9. Critérios técnicos para segurança, isolamento e operação

Segurança e operação não deverão ser tratadas como camadas externas à arquitetura documental.

Alterações operacionais podem modificar:

* corpus;
* documentos;
* permissões;
* resultados;
* caches;
* estados de processamento;
* condições de recuperação.

Por isso, segurança e operação constituem também condições de integridade metodológica.

---

## CT-OPS-01 — Identidade operacional

Operações relevantes deverão possuir identidade suficiente para reconstruir qual agente ou componente realizou determinada ação.

---

## CT-OPS-02 — Separação entre autenticação e autorização

A arquitetura deverá distinguir:

```text
quem é o usuário?
```

de:

```text
o que esse usuário pode fazer?
```

---

## CT-OPS-03 — Autorização por recurso

A autorização deverá ser aplicável aos recursos relevantes, incluindo:

* acervos;
* documentos;
* jobs;
* evidências;
* operações.

---

## CT-OPS-04 — Isolamento de acervos

A solução deverá impedir acesso ou mistura indevida entre diferentes acervos.

O acervo constitui simultaneamente:

```text
fronteira de segurança
```

e:

```text
fronteira de escopo documental
```

---

## CT-OPS-05 — Isolamento entre ambientes

Produção e desenvolvimento deverão possuir fronteiras operacionais explícitas.

O estado de um ambiente não deverá interferir silenciosamente nas condições documentais ou computacionais do outro.

---

## CT-OPS-06 — Concorrência segura

Jobs deverão possuir mecanismos que impeçam que múltiplos workers assumam simultaneamente a mesma tarefa de maneira incompatível.

---

## CT-OPS-07 — Publicação somente de estado válido

Falhas parciais não deverão produzir documentos aparentemente concluídos.

---

## CT-OPS-08 — Gestão de credenciais

Credenciais e tokens deverão possuir:

* proteção contra exposição;
* escopo compatível com sua função;
* ciclo de vida controlado;
* privilégios mínimos necessários.

---

## CT-OPS-09 — Controle de arquivos temporários

Arquivos temporários deverão possuir:

* isolamento;
* controle de acesso;
* ciclo de vida definido;
* eliminação quando não forem mais necessários.

---

## CT-OPS-10 — Observabilidade

A operação deverá permitir identificar o estado de componentes relevantes, incluindo:

```text
API
↓
banco
↓
fila
↓
worker
↓
dependências externas
```

Também deverá ser possível identificar em qual estágio uma operação falhou.

---

## CT-OPS-11 — Recuperação operacional

Falhas deverão possuir caminhos conhecidos de:

* retry;
* recuperação;
* reprocessamento.

---

## CT-OPS-12 — Controle de recursos

A arquitetura deverá possuir mecanismos para limitar e controlar:

* tamanho dos documentos;
* processamento;
* armazenamento;
* chamadas externas;
* crescimento da fila.

---

## CT-OPS-13 — Cache isolado

Estados armazenados em cache não deverão atravessar indevidamente fronteiras de:

```text
usuário
+
acervo
+
estado documental
+
configuração
```

---

## CT-OPS-14 — Privacidade e retenção

A arquitetura deverá permitir estabelecer políticas deliberadas de:

* armazenamento;
* acesso;
* retenção;
* exclusão;
* auditoria.

---

## CT-OPS-15 — Menor privilégio e defesa em profundidade

Cada componente deverá possuir apenas os privilégios necessários à sua função.

Nenhum mecanismo isolado deverá ser considerado a única barreira de segurança.

---

## CT-OPS-16 — Integridade metodológica operacional

Operações administrativas ou técnicas não deverão modificar silenciosamente as condições documentais da pesquisa.

A cadeia:

```text
corpus
↓
representação
↓
recuperação
↓
evidência
```

deverá permanecer protegida contra alterações operacionais não identificáveis.

---

# 10. Convergência dos critérios técnicos

Os requisitos das seis subetapas não constituem 171 problemas tecnológicos independentes.

Eles convergem para alguns grandes eixos de capacidade.

| Eixo técnico      | Capacidade necessária                                                 |
| ----------------- | --------------------------------------------------------------------- |
| **Representação** | Preservar o documento e sua estrutura antes das derivações            |
| **Estrutura**     | Representar relações entre elementos documentais                      |
| **Segmentação**   | Criar unidades recuperáveis sem destruir a representação preservada   |
| **Recuperação**   | Localizar unidades de maneira configurável, rastreável e avaliável    |
| **Proveniência**  | Reconstruir a cadeia entre origem, derivação, recuperação e evidência |
| **Temporalidade** | Identificar o estado documental de cada representação                 |
| **Consistência**  | Impedir que derivados incompatíveis sejam tratados como válidos       |
| **Operação**      | Processar, repetir, falhar e recuperar sem corromper estados          |
| **Segurança**     | Controlar quem pode acessar ou modificar condições documentais        |
| **Avaliação**     | Permitir medir essas propriedades independentemente da síntese        |

Esses eixos não substituem os critérios individuais.

Eles funcionam como uma estrutura de leitura para a investigação da Etapa 6.2.

---

# 11. Relação de dependência entre os critérios

Os critérios não devem ser tratados como um conjunto plano.

Existe uma dependência arquitetural fundamental:

```text
REPRESENTAÇÃO
     │
     ▼
ESTRUTURA
     │
     ▼
SEGMENTAÇÃO
     │
     ▼
RECUPERAÇÃO
     │
     ▼
PROVENIÊNCIA
     │
     ▼
VERSIONAMENTO
     │
     ▼
CONSISTÊNCIA
     │
     ▼
OPERAÇÃO SEGURA
```

A representação determina aquilo que pode ser preservado.

A estrutura determina aquilo que pode ser compreendido como relação documental.

A segmentação determina aquilo que pode ser separado em unidades recuperáveis.

A recuperação determina aquilo que pode ser encontrado.

A proveniência determina aquilo que pode ser demonstrado.

O versionamento determina de qual estado aquilo veio.

A consistência determina se as relações entre esses estados continuam válidas.

A operação determina em quais condições essas propriedades continuam confiáveis.

---

# 12. Avaliação como dimensão transversal

A avaliação não deve ser considerada apenas uma etapa posterior da implementação.

Ela constitui uma capacidade que a arquitetura tecnológica deve possibilitar.

A avaliação deverá poder observar separadamente, conforme a função:

```text
representação
→ integridade documental

segmentação
→ qualidade das unidades

recuperação
→ qualidade da seleção

proveniência
→ completude da rastreabilidade

versionamento
→ coerência temporal

operação
→ estabilidade e recuperabilidade
```

A qualidade da síntese produzida por um modelo de linguagem não deverá ser utilizada como substituta da avaliação dessas camadas.

---

# 13. Consequência para a investigação da Etapa 6.2

A Etapa 6.2 deverá investigar alternativas tecnológicas a partir dos critérios estabelecidos neste registro.

A pergunta de investigação não deverá ser:

> **“Qual é a melhor biblioteca para o Snoopy-RAG?”**

A pergunta deverá ser:

> **“Quais combinações de tecnologias conseguem satisfazer os critérios necessários para transformar o documento original em uma representação preservadora, reutilizável, segmentável, recuperável e rastreável, quais limitações apresentam e sob quais condições?”**

Uma solução poderá ser composta por múltiplos componentes.

Não existe exigência de correspondência:

```text
1 função arquitetural
=
1 ferramenta
```

Uma função poderá ser realizada por uma composição de mecanismos especializados.

Por exemplo, conceitualmente:

```text
documento
+
processamento estrutural
+
OCR quando necessário
+
representação multimodal
+
derivação textual
+
persistência
```

Essa composição constitui apenas uma possibilidade arquitetural de investigação e **não uma decisão tecnológica**.

---

# 14. Ordem de investigação

Os critérios definidos neste registro deverão orientar a investigação na seguinte ordem geral:

```text
6.2.1 — Representação canônica e processamento de documentos
        ↓
6.2.2 — OCR e documentos digitalizados
        ↓
6.2.3 — Estrutura, layout, tabelas e elementos multimodais
        ↓
6.2.4 — Persistência da representação
        ↓
6.2.5 — Segmentação
        ↓
6.2.6 — Recuperação
        ↓
6.2.7 — Proveniência e versionamento
        ↓
6.2.8 — Operação
```

Essa ordem preserva a dependência estabelecida nas Etapas 5.1–5.6.

A investigação deverá evitar selecionar mecanismos de segmentação antes de saber adequadamente o que a representação documental consegue preservar, assim como evitar definir mecanismos de recuperação antes de definir quais unidades serão efetivamente recuperadas.

---

# 15. Estado epistemológico deste registro

Este documento define **critérios de investigação**, não conclusões tecnológicas.

Portanto, neste ponto:

* nenhuma biblioteca foi escolhida;
* nenhum modelo foi escolhido como solução definitiva;
* nenhum formato de representação foi definido como obrigatório;
* nenhum mecanismo de OCR foi definido como obrigatório;
* nenhuma estratégia de segmentação foi definida como definitiva;
* nenhuma estratégia de recuperação foi definida como definitiva;
* nenhuma arquitetura de persistência foi fechada;
* nenhuma implementação foi realizada;
* nenhum desempenho foi presumido;
* nenhuma capacidade foi considerada demonstrada apenas por documentação de fornecedor ou projeto.

A investigação da Etapa 6.2 deverá distinguir entre:

```text
capacidade declarada
```

e:

```text
comportamento demonstrado
```

A documentação declara capacidade.

Os testes demonstram comportamento.

---

# 16. Princípio central da Etapa 6.1

A tradução dos requisitos da Etapa 5 para critérios técnicos estabelece o seguinte princípio:

> **Uma tecnologia não deverá ser escolhida porque parece adequada ao Snoopy-RAG; deverá ser considerada adequada na medida em que demonstrar capacidade de satisfazer os requisitos arquiteturais necessários, dentro das condições, limitações, custos e compromissos identificados pela investigação.**

A arquitetura deverá, portanto, ser construída na direção:

```text
REQUISITO
↓
CRITÉRIO
↓
ALTERNATIVA
↓
EVIDÊNCIA
↓
COMPARAÇÃO
↓
ESCOLHA
```

e não:

```text
TECNOLOGIA ENCONTRADA
↓
ADAPTAÇÃO DO PROBLEMA
```

---

# 17. Princípio de encerramento

A Etapa 6.1 estabelece a ponte entre a especificação arquitetural e a investigação tecnológica.

A Etapa 5 respondeu:

> **O que o Snoopy-RAG precisa preservar, permitir, demonstrar e controlar?**

A Etapa 6.1 transforma essa resposta em:

> **Que propriedades técnicas uma solução deverá possuir para tornar isso possível?**

A Etapa 6.2 deverá então investigar:

> **Quais alternativas tecnológicas conseguem satisfazer essas propriedades, em que grau, com quais perdas, custos, limitações e possibilidades de composição?**

Somente depois dessa investigação deverão ser consolidadas as escolhas da arquitetura técnica.

A sequência metodológica da Etapa 6 fica, portanto:

```text
ETAPA 5
ESPECIFICAÇÃO METODOLÓGICA E ARQUITETURAL
              ↓
ETAPA 6.1
CRITÉRIOS TÉCNICOS
              ↓
ETAPA 6.2
INVESTIGAÇÃO DAS ALTERNATIVAS
              ↓
ETAPA 6.3
PROJETO TÉCNICO DA SOLUÇÃO ESCOLHIDA
              ↓
ETAPA 6.4
IMPLEMENTAÇÃO
              ↓
ETAPA 6.5
TESTES E VALIDAÇÃO
```

**A Etapa 6.1, portanto, não escolhe a arquitetura. Ela estabelece a régua pela qual a arquitetura poderá ser escolhida.**
