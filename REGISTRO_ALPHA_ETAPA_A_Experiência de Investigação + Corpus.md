# Registro Mestre da Análise — Snoopy-RAG

## Trilha Alpha A — Experiência de Investigação + Corpus

**Status:** Consolidado e aprovado para transição à Trilha B
**Natureza:** especificação funcional e arquitetural mínima da Alpha
**Escopo:** condensação do essencial das Etapas 6.3.3 e 6.3.4, complementada pelas decisões necessárias para a execução da Alpha.

---

## 1. Finalidade da Trilha Alpha A

A Trilha A estabelece o comportamento mínimo necessário para que o Snoopy-RAG Alpha funcione como um ambiente de exploração documental.

A experiência deve permitir que o pesquisador percorra, de maneira contínua e rastreável, o caminho:

**consulta → recuperação → resultados → evidência → contexto → fonte → retorno à investigação**

Ao mesmo tempo, a Alpha deve possuir um corpus documental definido, documentos identificáveis e um processamento mínimo capaz de transformar documentos admitidos em unidades recuperáveis.

A Trilha A não pretende implementar a arquitetura documental completa definida nas Etapas 5 e 6.2. Ela estabelece somente o subconjunto indispensável para que a Alpha seja funcional e coerente com essa arquitetura.

---

# 2. A.1 — Consulta → Resultados

## 2.1 Consulta original

A consulta formulada pelo pesquisador é o objeto inicial da investigação.

A consulta original deve permanecer distinguível das operações internas realizadas posteriormente pelo sistema.

O sistema pode transformar ou decompor a consulta para fins de recuperação, mas essas transformações são operações computacionais derivadas e não substituem a consulta original do pesquisador.

---

## 2.2 Recuperação

A recuperação transforma a consulta em candidatos documentais recuperáveis.

A Alpha poderá utilizar as capacidades de recuperação já estabelecidas na arquitetura V3, especialmente:

* recuperação lexical;
* recuperação semântica;
* filtros estruturados quando necessários.

Essas capacidades são complementares.

Restrições estruturadas, como autoria ou outros metadados, não devem ser confundidas com sinais de similaridade textual.

---

## 2.3 Resultados identificáveis

Os resultados da recuperação devem corresponder a unidades documentais identificáveis.

A identidade da unidade recuperada deve ser preservada durante a investigação.

A recuperação de uma unidade não significa automaticamente que ela constitui uma evidência metodologicamente relevante.

**Recuperação computacional ≠ decisão metodológica.**

---

## 2.4 Seleção e evidência

A Alpha deve permitir a passagem mínima de:

**unidades recuperadas → unidades exploradas/selecionadas → evidências**

A interface não precisa transformar essa distinção em um complexo sistema de anotação na Alpha, mas a arquitetura não deve colapsar esses conceitos.

---

## 2.5 Síntese

A síntese permanece como uma capacidade integrada à experiência de investigação.

O fluxo herdado do V2 que deverá ser preservado conceitualmente é:

**consulta original → decomposição quando necessária → recuperação/filtros → passagens recuperadas → LLM recebe consulta original + evidências recuperadas → síntese fundamentada → marcadores de evidência → navegação para evidência/fonte**

A síntese não constitui a evidência primária.

Ela funciona como uma camada de compreensão e como possível porta de entrada para a exploração das evidências.

---

# 3. A.2 — Evidência → Contexto → Fonte

## 3.1 Identidade da evidência

A evidência deve possuir identidade própria e permanecer reabrível por essa identidade.

A Alpha não deve depender de mecanismos frágeis de localização baseados em:

* título do documento;
* nome da pasta;
* texto coincidente;
* conteúdo de um chunk como identificador.

A identidade documental deve ser estruturalmente separada do conteúdo textual apresentado.

---

## 3.2 Unidade, contexto e fonte

A experiência deve distinguir:

**unidade recuperada → contexto → documento de origem → fonte original**

A unidade recuperada mantém sua própria identidade.

A contextualização é uma representação derivada e não altera a identidade da unidade.

---

## 3.3 Contexto mínimo

A Alpha deverá apresentar contexto suficiente para permitir a compreensão da unidade recuperada.

Foram estabelecidos dois níveis mínimos:

### Nível 0

Unidade isolada.

### Nível 1

Unidade acompanhada de contexto textual próximo.

Esse nível é obrigatório na experiência normal da Alpha.

### Nível 2

Unidade acompanhada de contexto estrutural, como título ou estrutura documental próxima, quando essa informação estiver disponível de maneira simples e confiável.

---

## 3.4 Localização mínima

A evidência deverá permitir identificar, no mínimo:

* documento de origem;
* identidade da unidade;
* localização documental mínima;
* referência à fonte original.

Sempre que possível, a localização deverá incluir informações como:

* página;
* elemento ou bloco de origem;
* tipo do elemento;
* identificador estrutural.

A Alpha não precisa implementar ainda a navegação espacial completa do documento.

---

## 3.5 Fonte original

O Google Drive permanece como fonte documental original.

A Alpha deve preservar o caminho lógico:

**evidência → documento → fonte original**

A implementação completa de um Reading Mode de alta precisão, com navegação espacial e reconstrução visual avançada, permanece fora do escopo da Alpha.

---

# 4. A.3 — Retorno à Investigação

## 4.1 Continuidade da exploração

A Alpha deverá preservar a continuidade mínima da investigação.

O pesquisador deve poder transitar entre:

**consulta → resultados → evidência → contexto → fonte**

sem perder a investigação atual.

Essa continuidade não significa que o Snoopy deva transformar a experiência em uma conversa contínua obrigatória.

O princípio é:

> **continuidade de exploração, não continuidade obrigatória de conversa.**

---

## 4.2 Preservação da consulta e dos resultados

Ao abrir uma evidência ou fonte, o estado da consulta e dos resultados atuais não deve ser destruído.

O pesquisador deve poder retornar à investigação em andamento.

---

## 4.3 Reabertura da evidência

Uma evidência deve permanecer reabrível por sua identidade.

A abertura da fonte documental não deve eliminar a referência à unidade recuperada.

---

## 4.4 Nova consulta

O pesquisador deve poder iniciar uma nova consulta sem perder necessariamente a possibilidade de retornar ao contexto da investigação atual.

A Alpha não precisa implementar ainda:

* histórico persistente completo;
* múltiplas investigações independentes;
* workspace de pesquisa;
* anotações;
* coleções;
* colaboração;
* comparação avançada;
* exportação sofisticada.

---

## 4.5 Fluxo mínimo consolidado

A experiência de investigação da Alpha é representada por:

**PERGUNTA → CONSULTA → RECUPERAÇÃO → RESULTADOS → EVIDÊNCIA → CONTEXTO → FONTE/LEITURA → VOLTAR ou NOVA BUSCA**

---

# 5. A.4 — Corpus e Processamento Mínimo

## 5.1 Representação do corpus

O corpus é uma entidade lógica que define o universo documental da investigação.

O pertencimento de um documento ao corpus é independente de sua disponibilidade para recuperação.

Portanto:

**pertencer ao corpus ≠ estar disponível para pesquisa**

O corpus possui identidade própria e relação explícita com seus documentos.

O corpus não deve se tornar uma “superentidade” responsável por processamento, jobs, embeddings ou demais estados operacionais.

A separação conceitual é:

**Corpus → define o universo**
**Document → representa o documento e sua disponibilidade**
**Job → representa a execução do processamento**

---

## 5.2 Fonte documental

O Google Drive permanece como fonte documental original.

O PostgreSQL/Supabase permanece como camada governante do estado lógico do Snoopy.

O PDF original não precisa ser duplicado permanentemente pelo Snoopy.

A identidade da fonte externa e a identidade documental interna são distintas:

* `drive_file_id` → identifica a fonte externa;
* `document_id` → identifica a entidade documental do Snoopy.

---

## 5.3 Entrada de documentos

A admissão de documentos na Alpha ocorre automaticamente dentro da fronteira do corpus definida pelo pesquisador.

A descoberta de um arquivo no Drive não deve ser confundida com:

* processamento;
* validação;
* disponibilidade para recuperação.

O fluxo conceitual é:

**fonte externa → descoberta → admissão como Document → Job de processamento → representação recuperável → disponibilidade**

Não é necessária uma etapa manual de aprovação documento por documento para a Alpha.

---

## 5.4 Documento e Job

O `Document` possui identidade própria e não depende da existência de um `Job`.

O `Job` representa uma tentativa de executar determinada operação sobre o documento.

Assim:

**Document = entidade documental**
**Job = execução operacional**

O sucesso ou fracasso de um processamento não elimina a identidade do documento.

---

## 5.5 Estados mínimos

A Alpha utilizará somente os estados indispensáveis para representar seu ciclo de vida.

Conceitualmente:

**PENDING → PROCESSING → ACTIVE**

Com estados alternativos:

**PROCESSING → FAILED**

e:

**PENDING → REJECTED**

Além disso:

**ACTIVE → REMOVED**

### Semântica

**PENDING**
Documento pertencente ao corpus, ainda não processado.

**PROCESSING**
Processamento em andamento.

**ACTIVE**
Possui os derivados mínimos válidos para recuperação e pode participar da busca.

**FAILED**
O processamento foi tentado, mas falhou. O documento continua pertencendo ao corpus, porém não é recuperável.

**REJECTED**
O documento foi identificado dentro da fronteira do corpus, mas não atende aos critérios necessários para processamento. O motivo da rejeição deve ser identificável.

**REMOVED**
O documento deixou de participar do corpus ativo ou da recuperação.

Os nomes concretos dos estados podem ser ajustados na arquitetura integrada, desde que sua semântica seja preservada.

A Alpha não implementará ainda a máquina de estados completa prevista para a arquitetura V3.

---

# 6. Critério de disponibilidade para recuperação

Um documento somente poderá participar da recuperação quando os derivados mínimos necessários estiverem válidos e publicados.

O fluxo mínimo é:

**Document → extração → representação necessária → segmentação → embeddings → ACTIVE**

Enquanto esse processo não estiver concluído, o documento não participa da busca.

Isso impede que documentos parcialmente processados apareçam como resultados válidos.

---

# 7. Processamento

O processamento mínimo da Alpha mantém a separação estabelecida anteriormente entre:

* representação documental;
* segmentação;
* indexação/embeddings;
* recuperação.

O processamento deve produzir somente aquilo que é necessário para tornar o documento recuperável na Alpha.

A arquitetura não deve transformar uma etapa operacional em identidade documental.

---

# 8. Erros e reprocessamento

Uma falha de processamento produz:

**PROCESSING → FAILED**

O documento continua pertencendo ao corpus.

O erro deve ser identificável e recuperável para fins de diagnóstico.

O reprocessamento não cria um novo documento.

O fluxo é:

**FAILED → novo Job → PROCESSING → ACTIVE**

A Alpha não exige mecanismo sofisticado de retomada parcial.

Quando necessário, o processamento pode ser reexecutado.

---

# 9. Atualização

A Alpha não implementará versionamento documental completo.

Quando o arquivo de origem for alterado, o comportamento mínimo será:

**Document ACTIVE → alteração detectada → reprocessamento → ACTIVE**

O `document_id` permanece o mesmo.

A arquitetura deve preservar espaço conceitual para que, posteriormente, o documento possua versões historicamente identificáveis.

Essa capacidade, entretanto, não faz parte da implementação Alpha.

---

# 10. Remoção

Quando um documento deixa de pertencer à fronteira ativa do corpus, ele deixa de participar da recuperação.

Na Alpha:

**ACTIVE → REMOVED**

A remoção lógica é suficiente.

A destruição imediata dos artefatos físicos derivados não é requisito da Alpha e pode ser tratada posteriormente conforme a política definitiva de persistência e invalidação.

O objetivo imediato é impedir que o documento removido continue sendo recuperado.

---

# 11. Sincronização mínima com o Google Drive

Sincronização e processamento são operações distintas.

A sincronização responde:

> **“O estado documental que o Snoopy conhece ainda corresponde minimamente à fonte original?”**

O comportamento mínimo é:

**Drive → descoberta de mudanças → reconciliação do estado documental → processamento quando necessário**

A sincronização deve distinguir:

* novo documento;
* documento alterado;
* documento inalterado;
* documento removido.

### Novo

**novo → admitido → processamento**

### Alterado

**alterado → reprocessamento**

### Inalterado

Nenhuma operação de processamento necessária.

### Removido

**removido → REMOVED**

A Alpha não exige ainda:

* sincronização sofisticada;
* histórico completo das mudanças;
* snapshots;
* reconstrução temporal;
* versionamento documental;
* mecanismos avançados de reconciliação.

---

# 12. Princípios consolidados da Trilha A

A Alpha deverá preservar as seguintes distinções:

**consulta ≠ consulta derivada**
**recuperação ≠ relevância metodológica**
**resultado ≠ evidência automaticamente aceita**
**unidade ≠ contexto**
**evidência ≠ fonte**
**corpus ≠ documentos pesquisáveis**
**documento ≠ job**
**processamento ≠ disponibilidade**
**identidade ≠ versão**
**fonte externa ≠ entidade documental interna**
**sincronização ≠ processamento**
**síntese ≠ evidência primária**

Essas distinções são parte da coerência arquitetural da Alpha.

---

# 13. Fora do escopo da Alpha

Ficam deliberadamente postergados:

* versionamento documental completo;
* snapshots de corpus;
* histórico documental completo;
* reconstrução temporal;
* múltiplas investigações persistentes;
* workspace de pesquisa;
* anotações e coleções;
* colaboração;
* comparação avançada;
* Reading Mode completo;
* navegação espacial avançada;
* sincronização sofisticada;
* retomada parcial sofisticada;
* máquina de estados operacional completa;
* políticas avançadas de invalidação;
* infraestrutura de escala horizontal;
* observabilidade avançada;
* mecanismos avançados de exportação e auditoria.

Esses itens não constituem lacunas acidentais da Alpha. São **questões conscientemente postergadas** para etapas posteriores.

---

# 14. Critério de encerramento da Trilha A

A Trilha A é considerada concluída quando o Snoopy Alpha for capaz de realizar, de maneira coerente:

**definir um corpus → admitir documentos → processá-los → disponibilizar documentos válidos para recuperação → recuperar unidades identificáveis → apresentar evidências contextualizadas → conduzir o pesquisador à fonte → preservar a investigação atual ao retornar.**

Com isso, a Alpha possui uma experiência mínima de exploração documental integrada ao corpus, sem exigir a implementação antecipada da arquitetura V3 completa.

---

## 15. Princípio central

> **A Alpha deve permitir que o pesquisador investigue um corpus documental real, percorra da consulta à evidência e da evidência à fonte, e retorne à investigação sem perder seu contexto, enquanto o sistema mantém uma distinção clara entre universo documental, documento, processamento, disponibilidade e fonte original.**

A Trilha A está, portanto, **consolidada e encerrada**, ficando pronta para a transição à **Trilha B — Interface + Segurança + Arquitetura de Frontend**.
