# REGISTRO MESTRE DA ANÁLISE — ETAPA 2
## Reconstrução da Arquitetura do Snoopy-RAG

**Projeto:** Snoopy-RAG  
**Repositório analisado:** `fabrciodias/snoopy-rag`  
**Branch de referência:** `main`  
**Objetivo desta etapa:** reconstruir o sistema como fluxo, estados, responsabilidades e relações causais, sem alterar código.  
**Data da análise:** 10/09/2026

---

# 1. Finalidade da Etapa 2

A Etapa 1 respondeu principalmente **“o que existe?”**: arquivos, componentes, banco, infraestrutura, histórico, configurações e fragilidades observáveis.

A Etapa 2 responde a uma pergunta diferente:

> **“Como o Snoopy funciona como sistema?”**

O objetivo não é apenas desenhar uma arquitetura bonita. É acompanhar o objeto documental desde sua origem no Google Drive até o momento em que uma evidência é apresentada ao pesquisador, e fazer também o caminho inverso: partir de uma evidência apresentada e verificar de onde ela veio.

A reconstrução deve distinguir quatro coisas que facilmente se confundem:

1. **intenção arquitetural** — aquilo que o projeto pretende fazer;
2. **implementação atual** — aquilo que o código efetivamente faz;
3. **estado persistido** — aquilo que permanece no banco;
4. **estado transitório** — aquilo que existe apenas durante uma operação.

A principal conclusão desta etapa é que o Snoopy não deve ser entendido simplesmente como “um chatbot que consulta PDFs”. Ele é uma cadeia documental com cinco movimentos principais:

**seleção do acervo → ingestão documental → transformação/indexação → recuperação de evidências → apresentação/síntese.**

A LLM aparece em vários pontos dessa cadeia, mas não é o sistema inteiro.

---

# 2. Visão sistêmica

O fluxo atual pode ser reconstruído assim:

```text
                    GOOGLE DRIVE
                         │
                         │ pasta + PDFs
                         ▼
                 ┌─────────────────┐
                 │ /api/sync       │
                 │ Node / Express  │
                 └────────┬────────┘
                          │
                 download temporário
                          │
                          ▼
                data/raw_pdfs/{id}.pdf
                          │
                          │ job = pending
                          ▼
                 ┌─────────────────┐
                 │ worker.py       │
                 │ fila persistente │
                 └────────┬────────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │ pipeline.py     │
                 └────────┬────────┘
                          │
          ┌───────────────┼────────────────┐
          ▼               ▼                ▼
      extractor        cleaner          tagger
       PDF → texto    texto → MD      metadados LLM
          │               │                │
          └───────────────┴────────────────┘
                          │
                          ▼
                     chunker.py
                          │
                          ▼
                chunks documentais
                          │
                          ▼
                Gemini embeddings
                          │
                          ▼
                 Supabase/pgvector
                  documents + chunks
                          │
                          │
        ┌─────────────────┴──────────────────┐
        │                                    │
        │ pergunta                           │
        ▼                                    │
  navegador / frontend                       │
        │                                    │
        ▼                                    │
  /api/search                                │
        │                                    │
        ▼                                    │
  search.py                                  │
        │                                    │
        ├── decomposição LLM                │
        │                                    │
        ├── embedding da(s) consulta(s)      │
        │                                    │
        ├── match_chunks                     │
        │                                    │
        ├── deduplicação                     │
        │                                    │
        └── seleção dos top 10               │
                         │                    │
                         ▼                    │
                    LLM síntese              │
                         │                    │
                         ▼                    │
              resposta + fontes + chunks
                         │
                         ▼
                  frontend / UI
                         │
          ┌──────────────┼────────────────┐
          ▼              ▼                ▼
       resposta       evidências       leitura
          │              │                │
          │              │                ▼
          │              │        /api/document-chunks
          │              │                │
          │              │                ▼
          │              │          chunks completos
          │              │
          └──────────────┴──► Drive / documento
```

Essa representação revela uma característica importante: **há dois pipelines diferentes**.

### Pipeline A — documental

Google Drive → PDF → texto → Markdown → metadados → chunks → embeddings → banco.

### Pipeline B — interrogativo

Pergunta → decomposição → embedding → busca vetorial → seleção de contextos → síntese → evidência → leitura.

Os dois pipelines se encontram no banco.

---

# 3. Os objetos fundamentais do sistema

Para compreender a arquitetura, é melhor pensar nos objetos que circulam por ela do que apenas nos arquivos de código.

## 3.1. Acervo

O acervo é representado conceitualmente por `folders`.

Ele funciona como **fronteira lógica de busca**.

O `folder_id` acompanha praticamente todo o caminho:

- seleção no frontend;
- sincronização;
- criação de jobs;
- documentos;
- chunks;
- consulta vetorial.

Isso significa que o acervo não é apenas uma preferência visual da interface. Ele é parte do contexto lógico da recuperação.

O Snoopy não pergunta simplesmente:

> “Quais chunks são semanticamente semelhantes?”

Ele pergunta:

> “Quais chunks semanticamente semelhantes pertencem a este acervo/contexto autorizado?”

Essa diferença é arquiteturalmente fundamental.

---

# 4. O documento como entidade persistente

Depois da ingestão, o documento passa a ser representado por `documents`.

A entidade mantém, entre outros dados:

- `id`;
- `user_id`;
- `folder_id`;
- `title`;
- `authors`;
- `publication_year`;
- `document_hash`;
- `drive_link`;
- `drive_file_id`.

O PDF físico não é mantido como fonte permanente no servidor. O arquivo temporário é usado para processamento e depois removido pelo pipeline.

Portanto:

```text
PDF físico temporário
        ↓
processamento
        ↓
documento lógico persistente
        +
chunks persistentes
        +
referência ao Drive
```

O documento persistente não é uma cópia integral do PDF. Ele é uma representação documental indexada.

---

# 5. O chunk como unidade computacional

O `chunk` é a principal unidade de recuperação atual.

Cada chunk persistido contém:

- `id`;
- `document_id`;
- `folder_id`;
- `user_id`;
- `content`;
- `section`;
- `embedding`.

O chunk é simultaneamente:

1. conteúdo textual;
2. unidade vetorizada;
3. unidade retornável pela busca;
4. unidade apresentada como `[TRECHO X]`.

Isso é extremamente importante para a compreensão metodológica posterior.

**Chunk não deve ser confundido automaticamente com unidade de registro de Bardin.**

O chunk é uma decisão de engenharia de informação. A unidade de registro é uma decisão metodológica do pesquisador.

O Snoopy atualmente entrega chunks como evidência recuperada. A identificação formal da unidade de registro/contexto continua sendo uma etapa do pesquisador.

---

# 6. Ingestão: do Drive à fila

A ingestão começa no frontend.

O usuário seleciona uma pasta do Google Drive pelo Google Picker. O frontend recebe o identificador da pasta e o envia ao backend.

A sincronização é feita por `/api/sync`.

O backend:

1. recebe `folder_id` e token do Google;
2. identifica o usuário quando há sessão;
3. consulta no banco o `drive_id` associado ao acervo;
4. consulta a API do Google Drive;
5. restringe a consulta a PDFs dentro daquela pasta;
6. compara os arquivos do Drive com documentos e jobs já existentes;
7. identifica arquivos ainda não processados ou enfileirados;
8. encerra a conexão HTTP quando há novos arquivos;
9. continua o download e a criação dos jobs em background.

A alteração recente do `/api/sync` é arquiteturalmente relevante: **a sincronização deixou de ser uma operação HTTP longa que espera todo o processamento.**

Agora:

```text
requisição
   ↓
descoberta dos arquivos
   ↓
aceitação da tarefa
   ↓
HTTP termina
   ↓
download + jobs em background
   ↓
worker processa
```

Isso separa claramente:

**“o sistema recebeu a solicitação”**

de

**“o documento foi efetivamente processado”.**

Essa distinção precisa permanecer na documentação futura.

---

# 7. A fila como mecanismo de desacoplamento

A tabela `jobs` funciona como uma fila persistente.

O job registra:

- usuário;
- acervo;
- nome do arquivo;
- `drive_file_id`;
- status;
- erro;
- progresso;
- timestamps.

O worker consulta jobs `pending`, ordena por `created_at`, seleciona o mais antigo e muda seu estado para `processing`.

Depois:

```text
pending
   ↓
processing
   ↓
completed
```

ou:

```text
pending
   ↓
processing
   ↓
failed
```

Isso desacopla a descoberta/download dos documentos da transformação pesada.

O frontend não precisa permanecer conectado durante:

- extração;
- limpeza;
- tagging;
- chunking;
- embeddings;
- inserção dos chunks.

O frontend observa a fila através do Supabase Realtime.

---

# 8. Estado transitório do PDF

O PDF baixado pelo `/api/sync` é armazenado temporariamente em:

```text
data/raw_pdfs/{drive_file_id}.pdf
```

Esse arquivo existe apenas como matéria-prima operacional.

O `pipeline.py` recebe seu caminho e:

1. abre o arquivo;
2. calcula MD5;
3. extrai texto;
4. limpa o conteúdo;
5. extrai metadados;
6. registra o documento;
7. gera chunks;
8. gera embeddings;
9. grava chunks;
10. remove o PDF.

Portanto, o armazenamento local não é o acervo.

O servidor local funciona como **área de trabalho temporária**.

Isso explica a filosofia da V2: o servidor não precisa guardar permanentemente todos os PDFs para que o corpus continue pesquisável.

---

# 9. Identidade documental: Drive ID versus MD5

Existem duas identidades importantes.

## 9.1. Identidade externa

`drive_file_id`

Identifica o arquivo dentro do Google Drive.

É utilizado para:

- localizar o PDF;
- baixar o conteúdo;
- evitar reprocessamento por arquivo já conhecido;
- reconstruir o link do Drive.

## 9.2. Identidade binária

`document_hash`

É o MD5 calculado a partir dos bytes do PDF.

O banco possui uma restrição única por:

```text
(folder_id, document_hash)
```

Portanto, o hash representa a identidade do **conteúdo binário do PDF dentro daquele acervo**, e não sua identidade bibliográfica.

Consequência:

Dois PDFs diferentes no Drive podem ser tratados como documentos distintos pelo `drive_file_id`, mas se forem binariamente idênticos no mesmo acervo, o MD5 impede a duplicação documental.

Isso não significa deduplicação semântica.

---

# 10. Extração: PDF → texto

O `extractor.py` utiliza PyMuPDF.

A extração percorre páginas e blocos de texto, preservando apenas blocos considerados textuais.

O fluxo é aproximadamente:

```text
PDF
 ↓
páginas
 ↓
blocos
 ↓
texto dos blocos
 ↓
normalização de espaços/quebras
 ↓
raw_text
```

Esse ponto é uma fronteira de perda de informação.

O sistema não conserva o layout original como objeto estrutural completo.

Consequentemente, elementos como:

- tabelas;
- diagramas;
- relações espaciais;
- fórmulas;
- elementos gráficos;

podem não sobreviver de forma fiel à representação textual.

Essa limitação pertence à cadeia documental e não apenas ao “modelo de IA”.

---

# 11. Limpeza: texto → Markdown

O `cleaner.py` transforma o texto extraído em uma representação mais organizada.

Há três operações conceitualmente importantes:

1. remoção de determinados caracteres/ruídos;
2. remoção de blocos repetitivos;
3. inferência heurística de títulos/seções.

A saída é um Markdown simplificado, no qual títulos reconhecidos passam a ser marcados como:

```text
## TÍTULO DA SEÇÃO
```

Essa transformação é importante porque o chunker depende dessa estrutura.

Logo:

```text
qualidade do extractor
        ↓
qualidade do cleaner
        ↓
qualidade do chunker
        ↓
qualidade da recuperação
```

Uma falha anterior na cadeia pode aparecer mais tarde como “erro da busca”, mesmo quando o problema original ocorreu na extração.

---

# 12. Metadados semânticos

O `tagger.py` utiliza uma LLM para extrair metadados do início do documento.

A estrutura esperada inclui:

- título;
- autores;
- ano;
- tipo documental;
- idioma;
- palavras-chave.

Na persistência atual, os campos centrais utilizados pelo sistema incluem título, autores e ano.

Essa etapa não produz o conteúdo recuperável principal. Ela produz **estrutura descritiva sobre o documento**.

Isso cria duas camadas:

```text
CONTEÚDO
→ chunks
→ embeddings

DESCRIÇÃO DO DOCUMENTO
→ title / authors / year / etc.
```

Essa separação será importante para futuros filtros por autor, ano ou tipo documental.

---

# 13. Chunking atual

O chunker trabalha sobre o Markdown.

A lógica atual:

1. divide por parágrafos;
2. acompanha a seção atual;
3. agrupa parágrafos;
4. tenta manter chunks próximos de aproximadamente 1500 caracteres;
5. cria metadado de seção;
6. produz uma lista de objetos.

O chunking, portanto, é **estrutural/heurístico**, e não uma segmentação metodológica de unidades de registro.

Um chunk pode conter:

- vários parágrafos;
- uma seção;
- partes de uma seção;
- uma quantidade variável de conteúdo.

A decisão de chunking afeta diretamente a recuperação vetorial.

Se um conceito estiver dividido de forma ruim, o embedding pode representar contexto excessivamente amplo ou excessivamente estreito.

---

# 14. Embeddings: transformação do conteúdo em espaço vetorial

Cada chunk é transformado em um vetor de dimensão 768.

Na persistência, o embedding fica associado ao chunk.

A mesma operação conceitual acontece com a pergunta durante a busca:

```text
texto do documento ──→ embedding ──→ vetor
pergunta             ──→ embedding ──→ vetor
```

A recuperação compara os vetores.

O sistema atual usa o operador de distância cosseno no PostgreSQL/pgvector.

A função `match_chunks` converte a distância em similaridade:

```text
similarity = 1 - cosine_distance
```

A busca aplica um limiar e ordena pela distância vetorial.

---

# 15. Busca: a pergunta não vai diretamente ao banco

A busca possui uma camada intermediária de interpretação.

A função `decompose_query()` utiliza a LLM para transformar a pergunta em microconsultas.

Isso existe especialmente para perguntas que envolvem:

- vários autores;
- vários conceitos;
- relações entre conceitos.

A intenção é evitar que uma consulta como:

```text
Autor A + conceito X + Autor B + conceito Y
```

seja tratada como um único bloco semântico.

O roteador tenta produzir:

```text
Autor A + conceito X
Autor B + conceito Y
...
```

Portanto a pergunta sofre uma transformação:

```text
pergunta original
      ↓
decomposição semântica
      ↓
microconsultas
      ↓
embeddings
```

Isso significa que a recuperação atual não é uma busca vetorial “pura” da pergunta original.

Existe uma etapa generativa anterior à recuperação.

---

# 16. Recuperação vetorial

Para cada microconsulta:

1. gera-se um embedding;
2. chama-se `match_chunks`;
3. informa-se `user_id`;
4. informa-se `folder_id`;
5. aplica-se threshold `0.4`;
6. solicita-se no máximo 5 resultados.

Os resultados retornados contêm:

- id;
- documento;
- conteúdo;
- seção;
- similaridade;
- título;
- autores;
- ano;
- link do Drive.

Todos os resultados das microconsultas são reunidos.

Depois ocorre deduplicação.

---

# 17. Deduplicação

A deduplicação não utiliza o ID do chunk.

Ela normaliza o texto:

```text
res['text']
→ remove diferenças de espaços
→ lowercase
→ usa texto normalizado como chave
```

Consequentemente, dois resultados textualmente iguais provenientes de microconsultas diferentes são considerados o mesmo contexto para a etapa seguinte.

Isso é importante para a interpretação da recuperação:

**o número final de contextos não é simplesmente o número de resultados retornados pelas chamadas vetoriais.**

Há uma transformação intermediária.

---

# 18. Seleção dos dez contextos

Depois da deduplicação, o código simplesmente pega:

```text
unique_results[:10]
```

Esses dez resultados formam o contexto entregue à LLM de síntese.

A arquitetura atual, portanto, pode ser descrita como:

```text
microquery 1 → top 5
microquery 2 → top 5
microquery 3 → top 5
...
        ↓
união
        ↓
deduplicação
        ↓
ordem resultante
        ↓
primeiros 10
```

Isso é diferente de um reranker global sofisticado.

A antiga arquitetura V1 possuía mecanismos de reranking que não fazem parte do `search.py` atual.

---

# 19. O ponto crítico: o contexto recuperado

Aqui ocorre a principal transição metodológica.

O sistema não entrega inicialmente uma “verdade”.

Ele entrega:

> **contextos documentais recuperados segundo similaridade semântica.**

A LLM recebe esses contextos com marcadores:

```text
[TRECHO 1]
...

[TRECHO 2]
...

[TRECHO 3]
...
```

Esses números são criados durante a construção do prompt.

Eles não são IDs persistentes do banco.

Isso é essencial:

```text
chunk.id
≠
[TRECHO X]
```

`chunk.id` é identidade persistente.

`TRECHO X` é identidade contextual temporária dentro de uma resposta.

---

# 20. Síntese: segunda utilização da LLM

Depois de recuperar os contextos, a LLM recebe:

- pergunta original;
- contextos recuperados;
- regras de citação.

O prompt exige:

- uso estrito dos trechos;
- citação imediata;
- formato `[TRECHO X]`;
- ausência de informação externa aos contextos.

A resposta textual da LLM é, portanto, uma camada **derivada**.

A hierarquia é:

```text
DOCUMENTO ORIGINAL
       ↓
CHUNK
       ↓
CONTEXTO RECUPERADO
       ↓
SÍNTESE LLM
```

A síntese não substitui o contexto.

Ela é uma interpretação gerada sobre os contextos recuperados.

---

# 21. Evidência e síntese são objetos diferentes

A resposta final contém pelo menos três coisas conceitualmente distintas:

## 21.1. `answer`

Texto gerado pela LLM.

## 21.2. `chunks`

Lista dos contextos efetivamente entregues à LLM, numerados como `[TRECHO X]`.

## 21.3. `sources`

Agrupamento documental que relaciona os trechos aos documentos de origem.

Essa estrutura é uma das partes mais importantes do Snoopy:

```text
resposta
   ↓
[TRECHO 3]
   ↓
chunk 3 da resposta
   ↓
documento X
   ↓
Drive
```

A interface torna esse caminho navegável.

---

# 22. A função dos `[TRECHO X]`

Os marcadores são uma camada de ligação entre:

**texto gerado**

e

**evidência recuperada.**

O frontend converte os marcadores em elementos clicáveis.

Ao clicar, o usuário pode:

- localizar o contexto;
- consultar a referência;
- entrar no Modo Leitura.

Portanto, o marcador não é uma citação bibliográfica convencional.

É uma **referência operacional de evidência**.

Essa distinção deverá aparecer claramente na documentação acadêmica futura.

---

# 23. Modo Leitura

O Modo Leitura representa uma segunda rota de evidência.

O usuário parte de um chunk recuperado e solicita o documento correspondente.

O frontend envia:

```text
title + folder_id
```

para:

```text
/api/document-chunks
```

O backend procura o documento no acervo e depois recupera seus chunks completos em ordem crescente de ID.

O frontend reconstrói uma representação contínua do documento a partir desses chunks.

Depois tenta localizar o chunk-alvo comparando textos normalizados.

Assim:

```text
TRECHO recuperado
      ↓
documento de origem
      ↓
todos os chunks do documento
      ↓
localização aproximada do chunk
      ↓
contexto ampliado
```

O Modo Leitura não recupera o PDF original diretamente.

Ele recupera a **representação textual processada do documento armazenada nos chunks**.

O link para o Drive continua sendo a ponte para a fonte original.

---

# 24. Duas noções diferentes de “fonte”

O sistema atualmente trabalha com duas fontes em sentidos diferentes.

## Fonte A — fonte original

O PDF armazenado no Google Drive.

## Fonte B — fonte recuperada

O texto do chunk armazenado no Supabase.

O fluxo ideal de auditoria é:

```text
PDF no Drive
      ↓
extração
      ↓
chunk persistido
      ↓
chunk recuperado
      ↓
TRECHO X
      ↓
afirmação na síntese
```

Mas é importante não afirmar que o chunk é byte-a-byte equivalente a uma página do PDF.

Ele é resultado de transformação.

Portanto:

**rastreabilidade do documento original não é a mesma coisa que preservação perfeita do documento original.**

---

# 25. Autenticação e fronteira de usuário

O frontend mantém uma sessão Supabase.

Quando há sessão:

```text
Supabase user
   ↓
access token
   ↓
frontend
   ↓
Authorization header
   ↓
Node
   ↓
userId
```

O Node utiliza uma chave de serviço do Supabase para operações de backend.

Isso coloca o backend em posição privilegiada.

A RLS do banco é importante, mas o backend não pode ser tratado como se fosse automaticamente limitado pelas mesmas regras do cliente quando utiliza service role.

Essa questão pertence à camada de autorização, não à camada de recuperação semântica.

Na arquitetura conceitual:

```text
AUTENTICAÇÃO
      ↓
AUTORIZAÇÃO DO ACERVO
      ↓
RECUPERAÇÃO
```

Não são a mesma coisa.

---

# 26. `folder_id` como contexto e como fronteira de isolamento

O `folder_id` desempenha duas funções:

1. identifica o acervo escolhido;
2. restringe a busca vetorial.

A função `match_chunks` recebe explicitamente `p_folder_id`.

Isso significa que a busca não opera sobre todos os chunks globalmente.

Ela opera dentro da fronteira do acervo selecionado.

Esse é um dos motivos pelos quais o conceito de “acervo” é estrutural no Snoopy-RAG.

---

# 27. Acervo público e acervo privado

A arquitetura atual possui:

```text
Acervo Público
        +
Acervo Privado do usuário
```

No frontend, o acervo público aparece como opção padrão.

O acervo privado é carregado quando o usuário possui uma pasta vinculada ao Drive.

A distinção entre os dois não deve ser reduzida a:

```text
folder.user_id = usuário
```

porque o acervo público foi modelado como recurso sem proprietário individual.

A autorização precisa ser entendida conceitualmente como:

```text
recurso público
OU
recurso pertencente ao usuário
OU
usuário administrativo autorizado
```

Essa separação entre **identidade do recurso** e **autoridade administrativa** deverá ser preservada nas etapas futuras.

---

# 28. Histórico de pesquisa

A pesquisa também produz um objeto persistente secundário:

`search_history`.

O frontend salva a pergunta depois que recebe o resultado.

O histórico serve principalmente à interface.

Ele não constitui o corpus documental nem a evidência científica.

Logo:

```text
corpus
≠
resultados
≠
histórico de perguntas
```

São camadas diferentes.

---

# 29. Cache

Existe ainda uma camada transitória no Node:

```text
cache[hash]
```

A chave é derivada de:

```text
userId + folder_id + query
```

O cache mantém a resposta em memória do processo.

Isso significa que uma repetição exata pode não executar novamente:

- decomposição da consulta;
- embeddings da consulta;
- busca vetorial;
- síntese da LLM.

Para testes de reprodutibilidade, o cache precisa ser considerado uma variável experimental.

Uma repetição com cache não é uma repetição independente do pipeline.

---

# 30. Reprodutibilidade: o que o sistema realmente consegue garantir hoje

A arquitetura possui uma intenção forte de rastreabilidade, mas não deve ser descrita como formalmente reprodutível ainda.

Há pelo menos três camadas:

## 30.1. Reprodutibilidade documental

Mesmo PDF → mesmo MD5 → mesma identidade binária dentro do acervo.

## 30.2. Reprodutibilidade da recuperação

Mesmo corpus + mesma configuração + mesma consulta deveria idealmente produzir os mesmos contextos.

Mas isso ainda depende de:

- decomposição da pergunta por LLM;
- modelo de embedding;
- chunking;
- threshold;
- ordenação vetorial;
- empates;
- estado do corpus;
- cache.

## 30.3. Reprodutibilidade da síntese

A síntese LLM não deve ser tratada como necessariamente idêntica apenas porque a pergunta é a mesma.

O objeto metodologicamente mais importante para reprodução é o conjunto de evidências recuperadas.

Assim:

```text
REPRODUÇÃO FORTE DESEJÁVEL

mesmo corpus
+
mesma configuração
+
mesma consulta
        ↓
mesmos contextos recuperados
        ↓
síntese pode variar
```

Isso é muito mais defensável do que exigir literalmente a mesma redação gerada pela LLM.

---

# 31. A cadeia completa em sentido forward

O teste arquitetural principal da Etapa 2 pode ser resumido como:

```text
1. Usuário seleciona pasta
2. Pasta recebe folder_id interno
3. folder_id aponta para drive_id
4. Drive fornece PDFs
5. PDF recebe drive_file_id
6. PDF é baixado temporariamente
7. Job é criado
8. Worker assume job
9. Pipeline calcula document_hash
10. Extractor produz raw_text
11. Cleaner produz Markdown
12. Tagger produz metadados
13. Documento é persistido
14. Chunker produz chunks
15. Embeddings são gerados
16. Chunks são persistidos
17. Usuário formula pergunta
18. Node identifica userId/folder_id
19. search.py decompõe a pergunta
20. Cada microconsulta vira embedding
21. match_chunks recupera candidatos
22. resultados são deduplicados
23. até 10 contextos são selecionados
24. contextos recebem [TRECHO X]
25. LLM produz síntese
26. resposta + chunks + sources retornam ao frontend
27. frontend cria vínculos clicáveis
28. usuário abre evidência
29. evidência aponta para documento
30. Modo Leitura reconstrói o contexto documental
31. link do Drive permite chegar ao original
```

---

# 32. A cadeia completa em sentido backward

Partindo de uma afirmação exibida ao pesquisador:

```text
AFIRMAÇÃO DA RESPOSTA
        ↓
[TRECHO X]
        ↓
chunks[X] da resposta
        ↓
conteúdo textual
        ↓
document_id
        ↓
document
        ↓
drive_file_id
        ↓
Google Drive
        ↓
PDF original
```

Há uma observação importante:

A implementação atual mantém a relação direta entre o `[TRECHO X]` e o conteúdo do resultado através da estrutura `chunks` retornada pela busca.

A interface também relaciona o trecho ao documento por meio de `sources`.

Isso forma uma cadeia de rastreabilidade funcional.

Porém, ela ainda não constitui um sistema formal de proveniência versionada.

---

# 33. O que é persistente e o que é transitório

## Persistente

- usuário;
- acervo;
- documento;
- hash documental;
- metadados;
- chunks;
- embeddings;
- jobs;
- histórico;
- referência ao Drive.

## Transitório

- PDF baixado para `raw_pdfs`;
- processo Python;
- `raw_text`;
- Markdown em memória;
- lista de chunks antes da persistência;
- embeddings da consulta;
- microconsultas;
- `all_results`;
- `unique_results`;
- `top_results`;
- contexto do prompt;
- `[TRECHO X]`;
- cache de respostas do Node.

Essa distinção é central para compreender a filosofia V2.

---

# 34. O que permanece fora do sistema

O Snoopy não persiste atualmente como objeto científico formal:

- unidade de registro;
- unidade de contexto definida pelo pesquisador;
- código;
- categoria;
- interpretação metodológica;
- decisão analítica do pesquisador;
- justificativa de inclusão/exclusão de uma evidência;
- versão metodológica de uma análise Bardin.

Portanto, a arquitetura atual é uma **infraestrutura de recuperação e exploração documental**, não um sistema completo de análise de conteúdo.

Essa fronteira deve ser preservada.

---

# 35. Relação com Bardin

A relação arquitetural pode ser formulada assim:

```text
CORPUS
  ↓
Snoopy
  ↓
recuperação de contextos relevantes
  ↓
PESQUISADOR
  ↓
identificação da unidade de registro
  ↓
delimitação da unidade de contexto
  ↓
codificação
  ↓
categorização
  ↓
interpretação
```

O Snoopy pode reduzir o custo de localizar material relevante.

Ele não deve transformar automaticamente:

```text
chunk = unidade de registro
```

nem:

```text
similaridade = relevância metodológica definitiva
```

A similaridade é um critério computacional de recuperação, não uma decisão metodológica final.

---

# 36. Onde estão as decisões arquiteturais mais importantes

A reconstrução mostra que as decisões mais impactantes não estão concentradas na interface.

Elas estão em:

### 36.1. Identidade do corpus

`folder_id`.

### 36.2. Identidade documental

`drive_file_id` + `document_hash`.

### 36.3. Representação textual

Extractor + cleaner.

### 36.4. Granularidade

Chunker.

### 36.5. Representação semântica

Embedding.

### 36.6. Estratégia de recuperação

Decomposição + busca vetorial + threshold + top-K + deduplicação.

### 36.7. Evidência

`chunks` + `[TRECHO X]` + `sources`.

### 36.8. Síntese

LLM condicionada aos contextos recuperados.

### 36.9. Contexto ampliado

Modo Leitura.

Esses pontos devem receber atenção especial nas próximas etapas.

---

# 37. O papel do Node.js

O Node/Express é o **orquestrador de fronteira**.

Ele conecta:

```text
Browser
 ↕
Node
 ↕
Python
 ↕
Supabase
 ↕
Google / Gemini
```

Ele não é o motor semântico principal.

Suas responsabilidades atuais incluem:

- servir frontend;
- fornecer configuração pública;
- autenticar tokens;
- receber buscas;
- controlar cache;
- iniciar Python;
- transmitir logs;
- coordenar sincronização;
- baixar PDFs;
- inserir jobs;
- recuperar chunks para leitura;
- traduzir conteúdo.

A fronteira Node/Python é uma decisão arquitetural importante da V2.

---

# 38. O papel do Python

O Python contém os dois motores especializados:

## Ingestão

`pipeline.py`

## Recuperação

`search.py`

Isso produz uma divisão interessante:

```text
Node
→ coordenação e transporte

Python
→ processamento documental e semântico

Supabase
→ persistência e recuperação vetorial

Google Drive
→ fonte documental externa

Gemini
→ transformação semântica por LLM/embedding
```

---

# 39. O papel do Supabase

O Supabase não é apenas “onde ficam os vetores”.

Ele funciona como:

- banco documental;
- banco de usuários;
- controle lógico de acervos;
- fila de jobs;
- histórico;
- armazenamento de chunks;
- armazenamento de embeddings;
- RPC de recuperação;
- fonte de eventos Realtime.

Ele é, na prática, o **estado persistente central do sistema**.

---

# 40. O papel do Google Drive

O Google Drive permanece como fonte documental externa.

O Snoopy não substitui o Drive.

Ele cria uma representação pesquisável do conteúdo do Drive.

A relação pode ser pensada assim:

```text
Google Drive = fonte original / arquivo
Supabase = representação indexada
Snoopy = camada de exploração e recuperação
```

Essa distinção é particularmente importante para a rastreabilidade acadêmica.

---

# 41. O papel da LLM

A LLM participa de três funções diferentes.

## Função 1 — roteamento

Decomposição da pergunta.

## Função 2 — catalogação

Extração de metadados documentais.

## Função 3 — síntese

Produção da resposta final baseada nos contextos.

Já o embedding é outra operação:

```text
LLM generativa
≠
modelo de embedding
```

A arquitetura não deve ser descrita genericamente como “a IA lê tudo”.

Ela realiza transformações específicas em momentos específicos.

---

# 42. O principal ponto de fragilidade conceitual

Existe uma diferença entre:

> “o sistema encontrou o trecho”

e

> “o sistema provou que aquele trecho é a melhor evidência”.

A arquitetura atual implementa a primeira afirmação.

A segunda exige critérios metodológicos adicionais.

O vetor responde a uma questão de proximidade semântica.

O pesquisador decide relevância metodológica.

Essa diferença será central para a Etapa 4.

---

# 43. O principal ponto de fragilidade operacional

A fila é persistente, mas o worker atual não contém um mecanismo de reserva/lock transacional robusto no código observado.

O fluxo é:

```text
select pending
      ↓
pega o primeiro
      ↓
update processing
```

Em um único worker, isso funciona operacionalmente.

Em múltiplos workers concorrentes, há potencial para corrida:

```text
worker A → lê job pending
worker B → lê o mesmo job pending
worker A → processing
worker B → processing
```

Isso não significa que o sistema atual esteja quebrado. Significa que a arquitetura atual pressupõe, na prática, uma execução de worker por vez.

Essa conclusão deve entrar no registro de fragilidades, não ser confundida com o funcionamento nominal.

---

# 44. Separação entre produção e desenvolvimento

A reconstrução operacional já estabelecida na Etapa 1 deve ser incorporada à leitura arquitetural:

```text
PRODUÇÃO
MegaWare / Linux
        ↓
server.js
        ↓
worker.py
        ↓
Supabase compartilhado
```

e:

```text
DESENVOLVIMENTO
HP / Windows
        ↓
server.js
        ↓
Supabase compartilhado
```

O worker de desenvolvimento não foi colocado em operação no fluxo observado.

Isso significa que o ambiente de desenvolvimento pode testar a API e a busca sobre o estado compartilhado, mas a fila persistente não possui, no modelo atual, uma separação explícita por ambiente.

Logo:

```text
jobs
```

é global para o projeto Supabase, e não explicitamente:

```text
jobs_production
jobs_development
```

Essa é uma fragilidade arquitetural de isolamento de ambientes.

---

# 45. O problema da representação do contexto

A recuperação trabalha com chunks.

O Modo Leitura trabalha com todos os chunks do documento.

Isso produz dois níveis de contexto:

```text
contexto de recuperação
≈ chunk selecionado

contexto de leitura
≈ documento reconstruído a partir dos chunks
```

Esse desenho é coerente com a evolução metodológica do projeto:

```text
encontrar o trecho
      ↓
ver a evidência
      ↓
ver o contexto maior
```

O próximo passo metodológico natural será estudar como o pesquisador pode delimitar explicitamente:

```text
unidade de registro
        +
unidade de contexto
```

dentro dessa estrutura.

---

# 46. O que a arquitetura já faz muito bem

Sem transformar esta etapa em avaliação promocional, há algumas decisões estruturalmente boas:

1. o corpus possui fronteira explícita;
2. documentos possuem identidade persistente;
3. PDFs não precisam permanecer no disco;
4. processamento pesado foi desacoplado da requisição HTTP;
5. jobs são persistidos;
6. chunks e embeddings são persistidos;
7. recuperação retorna o texto efetivamente usado;
8. síntese recebe contexto explicitamente delimitado;
9. resposta carrega referências operacionais;
10. usuário pode ampliar a evidência para o documento;
11. Drive continua sendo a referência do arquivo original;
12. cache considera usuário e acervo na chave;
13. a arquitetura separa processamento documental de recuperação.

Esses elementos explicam por que o Snoopy atual já ultrapassou a categoria de protótipo simples de RAG.

---

# 47. O que ainda não deve ser afirmado

A documentação futura não deve afirmar, sem testes adicionais, que o sistema:

- garante reprodução literal de qualquer resposta;
- garante recuperação determinística em todos os casos;
- preserva integralmente o layout dos PDFs;
- identifica automaticamente unidades de registro de Bardin;
- garante que o trecho recuperado seja metodologicamente o melhor;
- possui isolamento absoluto entre ambientes;
- possui escalabilidade horizontal do worker;
- possui versionamento formal do corpus;
- possui proveniência científica formal;
- mantém o PDF localmente como arquivo permanente.

Essas afirmações ultrapassariam o que a implementação atual demonstra.

---

# 48. Modelo mental final da arquitetura

O Snoopy-RAG pode ser compreendido como uma máquina de transformação documental:

```text
                FONTE
                  │
                  ▼
             PDF no Drive
                  │
                  ▼
            REPRESENTAÇÃO
                  │
                  ▼
         texto + metadados
                  │
                  ▼
             GRANULARIZAÇÃO
                  │
                  ▼
               chunks
                  │
                  ▼
             VETORIZAÇÃO
                  │
                  ▼
             índice vetorial
                  │
                  │
            PERGUNTA
                  ▼
             roteamento
                  │
                  ▼
              embedding
                  │
                  ▼
             recuperação
                  │
                  ▼
             contextos
                  │
                  ├──────────────┐
                  ▼              ▼
              evidência       síntese
                  │              │
                  └──────┬───────┘
                         ▼
                    PESQUISADOR
                         │
                         ▼
                  leitura/verificação
                         │
                         ▼
                    FONTE ORIGINAL
```

A arquitetura inteira existe para encurtar o caminho:

```text
pergunta
   ↓
material relevante
   ↓
evidência
   ↓
fonte
```

sem eliminar o pesquisador desse circuito.

---

# 49. Conclusão da Etapa 2

A reconstrução permite afirmar que o Snoopy-RAG atual é composto por **dois pipelines acoplados por um estado documental persistente**:

### Pipeline documental

```text
Drive
→ sincronização
→ job
→ worker
→ extração
→ limpeza
→ metadados
→ chunking
→ embeddings
→ Supabase
```

### Pipeline de pesquisa

```text
pergunta
→ decomposição
→ embeddings
→ busca vetorial
→ deduplicação
→ seleção de contextos
→ síntese
→ evidências
→ leitura
→ Drive
```

O elo central é:

```text
document
   ↕
chunks
```

e o elo metodológico central é:

```text
contexto recuperado
   ↓
pesquisador
   ↓
unidade de registro/contexto
```

A principal conclusão arquitetural da Etapa 2 é, portanto:

> **O Snoopy-RAG não deve ser modelado como uma única sequência “PDF → IA → resposta”. Ele é uma infraestrutura documental em que uma representação persistente do corpus permite que dois fluxos distintos — ingestão e investigação — se encontrem em unidades textuais recuperáveis e rastreáveis.**

Essa formulação será a base para a Etapa 3 (reconstrução histórica) e, principalmente, para a Etapa 4 (análise metodológica).

---

# 50. Pontos que devem ser carregados para as próximas etapas

## Para a Etapa 3 — História

- transição V1 → V2/SaaS;
- mudança de armazenamento local para Supabase;
- introdução da fila;
- evolução da evidência;
- introdução de Bardin;
- surgimento do Modo Leitura;
- evolução da preocupação com rastreabilidade;
- evolução da arquitetura de sincronização;
- mudanças no motor de busca;
- desaparecimento do reranking V1;
- evolução do modelo de acervo.

## Para a Etapa 4 — Metodologia

- chunk ≠ unidade de registro;
- contexto recuperado ≠ decisão analítica;
- similaridade semântica ≠ relevância metodológica;
- LLM de síntese ≠ evidência;
- `[TRECHO X]` ≠ citação bibliográfica;
- rastreabilidade documental ≠ proveniência metodológica formal;
- reproducibilidade de recuperação ≠ reproducibilidade de texto gerado.

## Para a Etapa 5 — Lacunas

- lock/concorrência da fila;
- isolamento de ambiente;
- autorização backend;
- versionamento do corpus;
- determinismo da recuperação;
- índices vetoriais;
- atualização/remoção de documentos;
- consistência documento/chunks;
- recuperação por título no Modo Leitura;
- robustez da extração PDF;
- configuração efetiva dos modelos;
- gerenciamento de tokens Google;
- persistência/invalidacão do cache.

---

# 51. Estado da Etapa

**Etapa 2 — Reconstrução da Arquitetura: CONSOLIDADA EM NÍVEL CONCEITUAL.**

Nenhuma alteração de código foi feita como parte desta análise.

A arquitetura foi reconstruída a partir do estado atual do `main` e deve ser tratada como descrição do funcionamento observado, não como especificação futura.

**Próxima etapa recomendada: Etapa 3 — Reconstrução Histórica do Projeto.**
