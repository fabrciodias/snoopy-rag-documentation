# REGISTRO MESTRE DA ANÁLISE — ETAPA 3
## Reconstrução Histórica do Projeto Snoopy-RAG

**Projeto:** `fabrciodias/snoopy-rag`  
**Etapa:** 3 — Reconstrução Histórica  
**Data da reconstrução:** 10/09/2026  
**Base principal do estado atual:** `main` — `3a7f4ef4d0fc04eaa18cde1cb56ee01d8115b6f0`  
**Período histórico observado:** 09/05/2026–09/09/2026  
**Branches históricas relevantes:** `main`, `v2-saas`, `v2-beta`, `v2-dev`  
**Natureza deste documento:** registro de análise, não documentação operacional final.

---

# 1. Objetivo da Etapa 3

A Etapa 3 não pretende produzir uma simples lista de commits. O objetivo é reconstruir **como e por que o Snoopy-RAG se transformou no sistema atual**, distinguindo:

1. o problema que existia antes da implementação;
2. as primeiras hipóteses de solução;
3. as mudanças técnicas que realmente ocorreram;
4. os problemas que provocaram cada mudança importante;
5. as decisões que alteraram a arquitetura;
6. as mudanças de objetivo ou de escopo;
7. aquilo que foi consequência não planejada;
8. aquilo que foi introduzido posteriormente pela necessidade metodológica;
9. o que o histórico do Git comprova;
10. o que foi reconstruído a partir das conversas e, portanto, deve ser tratado como reconstrução contextual, e não como fato diretamente registrado no código.

A tese histórica central que emerge é:

> **O Snoopy-RAG não nasceu como a arquitetura atual. Ele começou como uma solução local para localizar informação em uma coleção de PDFs e evoluiu, sucessivamente, para um motor RAG, depois para uma aplicação multiacervo/multiusuário, e finalmente para uma infraestrutura de exploração documental orientada à rastreabilidade e à pesquisa acadêmica.**

A transformação não ocorreu de uma única vez. Ela foi cumulativa e, em vários momentos, reativa a problemas concretos.

---

# 2. Fontes e grau de confiança

A reconstrução combina três camadas de evidência.

## 2.1. Histórico do GitHub

É a fonte primária para saber **quando uma mudança foi incorporada ao código e como ela foi descrita pelo próprio commit**.

O repositório possui atualmente quatro branches históricas relevantes:

- `main`
- `v2-saas`
- `v2-beta`
- `v2-dev`

No estado consultado, `main` aponta para `3a7f4ef...`, `v2-dev` para `57ba476...`, `v2-beta` para `076bc97...` e `v2-saas` para `f8dd5a7...`.

O primeiro commit do repositório é:

`3d5f3f9f8a076eb5d377ce443267a70ad5f22060` — **09/05/2026** — `Commit inicial: estrutura limpa e gitignore`.

A comparação desse commit com o `main` atual registra **70 commits de avanço e nenhum de atraso**, confirmando a grande distância entre o ponto de partida do repositório e o estado atual.

## 2.2. Código e estado atual

O código permite verificar se uma ideia histórica permaneceu, foi substituída ou deixou de existir.

Isso é especialmente importante porque mensagens de commit frequentemente descrevem uma intenção ou uma solução intermediária. O estado atual pode ser diferente.

Exemplo: em determinado momento o sistema usou ChromaDB e FlashRank; no estado atual, a busca usa Supabase/pgvector e não possui mais o reranker FlashRank no caminho principal.

## 2.3. Conversas de desenvolvimento

As conversas fornecem o contexto que o Git não registra:

- por que o projeto foi iniciado;
- como a ideia foi apresentada originalmente;
- o papel da conversa com o orientador;
- quando Bardin entrou no problema;
- por que a ideia de SaaS ganhou força;
- como a necessidade de rastreabilidade mudou;
- como as decisões de arquitetura foram percebidas pelo próprio autor.

Essas informações são usadas como **contexto causal**, não como substitutas do histórico técnico.

---

# 3. Linha histórica condensada

| Período | Estado predominante | Mudança central |
|---|---|---|
| 09/05 | Repositório vazio/limpo | início formal do projeto |
| 10–12/05 | V1 local | PDF → texto → chunks → embeddings → busca |
| 12/05 | V1 automatizado | pipeline em lote, estado, limpeza efêmera, CLI/API |
| 14/05 | V1 quase funcional | interface web começa a envolver o motor |
| 14–17/05 | transição para SaaS | Auth, Supabase, acervos, sincronização Drive |
| 16/05 | V2 Core | banco central, RLS, `folder_id`, multiacervo |
| 17–18/05 | V2 arquitetural | pipeline em RAM, frontend modular, documentação V2 |
| 21/05 | virada metodológica | Bardin, metadados ABNT, contexto/evidência |
| 22–27/05 | estabilização | embeddings, OCR, tradução, Reading View |
| 26/05 | V2.0.0 Core | consolidação Cloud-Native + Modo Leitura |
| 16/06 | refinamento de leitura | mobile, smart header, UX |
| 15/07 | V2/SaaS consolidado | Realtime, jobs, progresso, backoff, OAuth |
| 15/07 | correções operacionais | worker, links Drive, parser de citações |
| 09/09 | V2.0.1-beta | cross-platform + `/api/sync` assíncrono |
| 10/09 | análise | reconstrução histórica |

---

# 4. Antes do Git: o problema que originou o Snoopy

A história conceitual começa antes de `3d5f3f9`.

O problema inicial não era “construir uma arquitetura SaaS”, “implementar um sistema multi-tenant” ou “automatizar Bardin”.

O problema era muito mais simples:

> **Como encontrar, dentro de uma pasta do Google Drive cheia de PDFs acadêmicos, o documento ou trecho relevante para uma pergunta?**

A ideia inicial consistia essencialmente em:

**Google Drive → PDFs → processamento → busca → resposta curta + indicação do PDF original.**

A necessidade prática era reduzir o trabalho de localizar o documento relevante.

Nesse estágio, a rastreabilidade imaginada era sobretudo **documental**: saber qual PDF continha a informação e poder voltar ao arquivo original.

A noção de RAG existia inicialmente mais como mecanismo implícito do que como projeto metodológico conscientemente formulado. A percepção de que aquilo constituía um sistema RAG foi amadurecendo durante a implementação.

Esse ponto é importante para a história porque impede uma leitura retrospectiva equivocada segundo a qual o projeto teria sido concebido desde o primeiro dia como a arquitetura acadêmica atual.

---

# 5. O início formal: V1 local

## 5.1. 09/05 — criação do repositório

O primeiro commit, `3d5f3f9`, apenas estabelece uma estrutura limpa e um `.gitignore`.

Isso marca o nascimento formal do repositório, não necessariamente o nascimento da ideia.

Os PDFs e artefatos locais são explicitamente excluídos do Git, preservando a separação entre código e corpus documental.

---

# 6. 10/05 — o motor documental aparece

O primeiro salto técnico relevante ocorre em `dbd9f77`, em 10/05:

**`feat: implementa extrator PyMuPDF por blocos e limpador semântico`**

A arquitetura começa a tomar forma:

**PDF → extração → limpeza → texto utilizável.**

A escolha do PyMuPDF por blocos já revela uma preocupação prática: não simplesmente ler o PDF como uma sequência bruta de caracteres, mas obter unidades textuais que possam ser reorganizadas.

O projeto ainda está distante do sistema atual.

---

# 7. 10/05 — chunking e embeddings

Em seguida surgem os componentes que transformam o extrato textual em material pesquisável semanticamente.

O commit `0c019b4`, em 10/05, introduz o chunking com a descrição:

**`feat: implementa chunking semantico com rastreabilidade de contexto`**

Logo depois, `50f9091`, também em 10/05, integra:

**ChromaDB + Gemini Embeddings + processamento em lote.**

Nesse momento aparece a forma clássica do primeiro RAG:

**documento → chunks → embeddings → banco vetorial → busca semântica.**

O banco vetorial era local, baseado em ChromaDB.

Isso é importante porque a arquitetura V1 não era apenas um protótipo de extração. O núcleo de recuperação semântica já estava sendo construído.

---

# 8. 11/05 — transformação do protótipo em pipeline

Em `14c6bea`, de 11/05:

**`feat: automatiza esteira de processamento em lote e consolida pipeline do RAG`**

A descrição explicita:

- extração de múltiplos PDFs;
- chunking em lote;
- JSON mestre unificado;
- injeção de links do Drive;
- fluxo `Sync → Extract → Chunk → Embed`.

Aqui o projeto deixa de parecer uma ferramenta para processar um PDF isolado e começa a operar sobre um **corpus documental**.

Esse é um primeiro ponto de inflexão.

A unidade de trabalho deixa de ser “um PDF de teste” e passa a ser “uma coleção de documentos”.

---

# 9. 11–12/05 — identidade, cache e efemeridade

O histórico mostra uma sequência de experimentos arquiteturais muito rápida.

`d48262e`, em 11/05:

**`fix: implementa cache hit com hash MD5 e injeta rastreabilidade do drive`**

A ideia de hash aparece como mecanismo de identificação do conteúdo e economia de processamento.

`4416ce6`, ainda em 11/05, tenta uma arquitetura de dossiês documentais isolados:

- `source.pdf`
- `semantic.md`
- `metadata.json`
- UUID
- timestamp
- MD5.

Depois `347962c`, em 12/05, muda novamente a estratégia:

**`refactor: implementa limpeza de disco e pipeline efêmero`**

O sistema passa a:

- manter um `drive_state.json`;
- acompanhar `modifiedTime`;
- processar uma fila;
- apagar o PDF após gerar o Markdown;
- registrar falhas de extração;
- evitar manter desnecessariamente os PDFs locais.

A direção arquitetural já começa a apontar para um princípio que permanece no V2:

> **o armazenamento local não é o corpus permanente; é uma área de trabalho do processamento.**

---

# 10. 12/05 — busca mais controlada e evidência

O commit `63bb9f2` adiciona:

- persistência em streaming/JSONL;
- melhoria do prompt de decomposição;
- ancoragem de autor/obra;
- citações `[TRECHO X]`;
- logs em tempo real;
- correção de duplicação de seções.

Esse commit é historicamente importante porque a busca deixa de ser somente:

**“encontre algo parecido”**

e passa a tentar responder:

**“encontre algo parecido e me permita saber qual trecho foi usado.”**

A marca `[TRECHO X]` nasce nesse estágio como mecanismo operacional de ligação entre síntese e evidência.

Mais tarde essa ideia será transformada em uma interface muito mais rica.

---

# 11. 12/05 — CLI vira API

`0fdf228`, de 12/05:

**`refactor: finaliza pipeline determinístico e transforma CLI em API`**

A mudança introduz:

- identificação determinística de documentos baseada no ID do Drive;
- correção dos links;
- separação de logs em `stderr`;
- saída final exclusivamente em JSON;
- estrutura preparada para consumo pelo frontend.

Isso marca uma passagem fundamental:

**motor de linha de comando → serviço consumível por interface web.**

A interface deixa de ser um possível acessório e passa a ser parte natural da arquitetura.

---

# 12. 12/05 — configuração do Drive

O histórico mostra também uma preocupação precoce com não deixar o `FOLDER_ID` fixado diretamente no código.

Os commits `36f4917`, `f92db52`, `6c2ee2b` e `a2490b9` trabalham na centralização da configuração da pasta do Drive.

A solução ainda seria substituída posteriormente pelo modelo de seleção de acervo via Google Picker, mas o problema já estava identificado:

> **a pasta do Drive é configuração externa do sistema, não deveria ser uma constante inseparável do código.**

---

# 13. 14/05 — o V1 chega perto de sua forma funcional

Em 14/05, `f81daeac` introduz:

- Google OAuth via Supabase;
- login/logout;
- nova UI;
- histórico local;
- suporte a tráfego anônimo e autenticado;
- tratamento de múltiplas citações.

Esse commit é especialmente importante para compreender a transição.

O projeto ainda estava sendo pensado como uma ferramenta local, mas a interface web e a identidade dos usuários começam a entrar no sistema.

A partir daqui, o problema deixa de ser apenas:

**“como buscar?”**

e começa a ser:

**“quem está buscando, em qual acervo e onde os resultados ficam?”**

---

# 14. 16/05 — nasce o V2 Core

O commit `95095e0`, em 16/05, é uma das maiores rupturas históricas:

**`refactor: fundação V2 CORE, RLS no Supabase e motor in-memory blindado`**

São introduzidos:

- `folders`;
- `documents`;
- `chunks`;
- `jobs`;
- Supabase;
- RLS;
- `folder_id` obrigatório na busca;
- MD5 em RAM;
- output JSON estruturado.

A arquitetura muda de maneira qualitativa.

### V1

**máquina local → arquivos locais → ChromaDB local**

### V2

**aplicação → banco central → acervos → documentos → chunks → busca vetorial**

A mudança não é simplesmente “trocar o banco”.

É a passagem de uma ferramenta pessoal para uma infraestrutura compartilhada.

---

# 15. Por que SaaS?

O histórico técnico sozinho mostra a transição, mas as conversas explicam o motivo.

A ideia inicial era manter o aplicativo no servidor MegaWare e facilitar o acesso.

Depois surgiu uma dificuldade prática: instalar/manter o sistema em cada máquina seria inconveniente.

A ideia de centralização ganhou força.

O SaaS, portanto, não surgiu originalmente porque “multiusuário” era o grande objetivo de pesquisa.

Ele surgiu como consequência da necessidade de:

- acesso de diferentes dispositivos;
- evitar instalação manual;
- manter o processamento centralizado;
- não depender de um corpus inteiro armazenado em cada computador.

Essa distinção é importante.

**Multiusuário foi uma consequência arquitetural importante; não foi o problema acadêmico original.**

---

# 16. 16–17/05 — acervos tornam-se primeira classe

`a6721fc`, em 17/05, introduz o motor de sincronização Google Drive:

- Google Picker;
- sincronização;
- diff de arquivos;
- download somente de PDFs inéditos;
- fila sequencial;
- execução do pipeline;
- uso da service key no backend.

`69ce78d`, também em 16/05, adiciona:

- SSE;
- roteamento por acervo;
- provisionamento de “Meu Acervo Pessoal”;
- fallback para “Acervo Público”;
- proteção contra race conditions;
- remoção do FlashRank;
- correção da filtragem de chunks.

Aqui o conceito de **acervo** deixa de ser apenas uma pasta externa e passa a ser uma fronteira lógica interna do sistema.

Isso será decisivo posteriormente para a segurança e para a reprodutibilidade contextual.

---

# 17. 17/05 — histórico e soft delete

`01d1681` adiciona:

- histórico de buscas na nuvem;
- `search_history`;
- RLS;
- soft delete de acervos;
- agrupamento de múltiplos trechos do mesmo PDF;
- upsert inteligente;
- deduplicação por texto;
- temperatura 0.1;
- exigência de citações `[TRECHO X]`.

Esse momento mostra outra mudança de prioridade.

O sistema não está apenas tentando produzir respostas.

Ele começa a preservar o **estado da interação** e a controlar melhor a relação entre:

**pergunta → trechos → síntese.**

---

# 18. 17/05 — limpeza arquitetural

`821b825`, em 17/05:

**`refactor: limpeza da arquitetura V1, módulos puros no Python e ES6 no Front-end`**

Remove:

- watcher legado;
- embedder legado;
- drive_sync legado;
- reset_db;
- outros componentes locais.

Transforma os módulos Python em componentes mais puros e orientados a RAM.

No frontend, o antigo `app.js` é dividido em:

- `auth.js`;
- `api.js`;
- `ui.js`;
- `main.js`.

Historicamente, esse é um momento em que o V2 deixa de ser apenas “V1 conectado a um banco”.

Ele começa a possuir uma arquitetura própria.

---

# 19. 18/05 — V2 passa a ser também uma identidade documental

Os commits `c29857d` e `23f8564`, em 18/05, fazem uma mudança simbólica importante:

- o README anterior é preservado como `README_V1.md`;
- nasce um novo README focado no V2;
- a licença MIT é adicionada.

Essa decisão mostra que a ruptura V1/V2 já era percebida como arquiteturalmente significativa.

Também é importante para a documentação futura:

> `README_V1.md` não deve ser tratado como documentação atual; ele é parte da história do projeto.

---

# 20. 21/05 — a entrada de Bardin muda o significado do sistema

Em `1c3eed5`, de 21/05:

**`feat: arquitetura para Análise de Conteúdo (Bardin) e metadados ABNT`**

O commit adiciona:

- layout de scroll duplo;
- autores e ano no documento;
- citações ABNT;
- seleção de trechos;
- alternância de tema.

Esse é provavelmente o maior ponto de inflexão conceitual depois da passagem para SaaS.

Antes, a rastreabilidade era principalmente:

**resposta → PDF**

Agora começa a ser:

**resposta → trecho → contexto → documento → referência bibliográfica.**

A conversa com o orientador e a introdução da Análise de Conteúdo de Bardin explicam por que isso aconteceu.

O sistema passa a ser pensado não somente como um “chat com PDFs”, mas como ferramenta auxiliar à **exploração de um corpus documental**.

---

# 21. A distinção entre chunk e unidade de registro

A partir da evolução metodológica, uma questão importante foi explicitada:

> **chunk não é automaticamente unidade de registro.**

O chunk é uma unidade técnica de processamento/retrieval.

A unidade de registro é uma decisão metodológica do pesquisador.

A unidade de contexto também não é simplesmente “o chunk retornado”.

O Snoopy pode:

- localizar textos;
- recuperar passagens;
- organizar documentos;
- mostrar contexto;
- facilitar comparação;
- apresentar evidências.

Mas a decisão sobre:

- qual é a unidade de registro;
- qual contexto é pertinente;
- como codificar;
- como categorizar;
- como interpretar;

continua pertencendo ao pesquisador.

Essa fronteira é uma das consequências mais importantes da evolução histórica do projeto.

---

# 22. 21/05 — mudança de identidade visual e mobilidade

`f165147`, em 21/05, refatora:

- `folders`;
- branding;
- sidebar;
- seletor de acervo;
- sincronização;
- menu mobile;
- gaveta de evidências;
- referências ABNT;
- interceptor de busca.

O projeto chega a uma forma mais próxima de produto utilizável.

A mudança de branding para LPP-Acervo mostra também que o projeto chegou a flertar com uma identidade institucional mais ampla.

Posteriormente, a identidade Snoopy-RAG permanece como referência do projeto técnico.

---

# 23. 22/05 — otimização do pipeline

`372411f`, em 22/05:

- muda modelo de embedding;
- adiciona cleanup em `finally`;
- padroniza logs;
- reorganiza Python;
- reduz redundâncias;
- tenta otimizar RAM.

Esse tipo de commit mostra uma característica recorrente da história do Snoopy:

> **o crescimento funcional sempre gerou uma segunda camada de trabalho para tornar o sistema executável de forma estável.**

A arquitetura não evoluiu apenas por novas funcionalidades; ela evoluiu também por problemas de memória, CPU, API, concorrência, infraestrutura e UX.

---

# 24. 26/05 — V2.0.0 Core

`e6633e6`, em 26/05:

**`feat(core): release v2.0.0-core, migração de embedding e Modo Leitura`**

É um marco formal.

O commit consolida:

- versão `2.0.0-core`;
- arquitetura Cloud-Native;
- Modo Leitura;
- migração para `gemini-embedding-2`;
- 768 dimensões;
- tradução contextual;
- correções do Supabase;
- correções do Markdown;
- blindagem do tagger.

O Modo Leitura é particularmente importante.

O sistema passa a permitir que o pesquisador saia da síntese e vá para uma visualização do documento textual armazenado, localizando o trecho citado.

Isso concretiza na interface a ideia que começou com `[TRECHO X]`.

---

# 25. 27/05 — OCR e infraestrutura deixam marcas

`f86ab02`, em 27/05, registra:

- correções de PM2;
- restauração da tradução;
- smart header;
- melhorias de leitura;
- sanitização de caracteres Unicode de Private Use Area;
- limpeza de artefatos de OCR.

A presença de um tratamento específico para OCR mostra que o sistema já estava encontrando problemas reais dos documentos, e não apenas problemas abstratos de arquitetura.

Isso reforça uma característica do projeto:

> a qualidade do retrieval depende da qualidade da representação documental produzida antes da busca.

---

# 26. Junho — amadurecimento da interface de leitura

O commit `ee50156`, em 16/06, trabalha principalmente na experiência mobile:

- remoção de overlay fantasma;
- correção do menu;
- sincronização com scroll;
- animações;
- aproveitamento da tela.

Esse período não representa uma mudança de filosofia comparável à passagem V1→V2 ou à entrada de Bardin.

É melhor interpretá-lo como **maturação da interface de um sistema cuja arquitetura principal já estava estabelecida**.

---

# 27. Julho — V2/SaaS torna-se sistema realtime

O commit `f8dd5a7`, em 15/07, é o grande marco de consolidação do SaaS:

**`fix & feat(saas): pipeline realtime, telemetria de sync, backoff e blindagem OAuth`**

Ele reúne:

### Segurança e isolamento

- RLS;
- Realtime em `jobs`.

### Pipeline

- exponential backoff;
- tratamento de rate limit;
- progresso de vetorização.

### Backend

- tratamento de expiração do token do Drive;
- correção de falso “Sincronização Concluída”.

### Frontend

- `supabase.channel()`;
- atualizações realtime;
- barra de progresso;
- Progressive Disclosure.

### OAuth

- auto-logout;
- mensagens semânticas para token expirado.

A mudança mostra que a aplicação já não era apenas um sistema de consulta.

Ela possuía agora um **processo assíncrono de ingestão observável pelo usuário**.

---

# 28. Um detalhe importante: o PR de SaaS

O PR #1 foi aberto como:

**`Release: V2 SaaS Beta - Realtime, UI Fluida e Estabilização`**

e foi fechado/merged em 15/07.

Entretanto, o próprio estado do PR mostra base e head apontando para o mesmo SHA `f8dd5a7`, com zero commits e zero arquivos alterados.

Portanto, o PR deve ser tratado historicamente como um **marcador de release/documentação**, e não como a origem efetiva das mudanças que aparecem em seu corpo.

A implementação já estava presente no commit que ele referencia.

Isso é um bom exemplo de por que a reconstrução histórica não pode simplesmente interpretar todo PR como causa de toda mudança descrita nele.

---

# 29. Julho — problemas operacionais reais

O commit `add07a6`, em 15/07, mostra uma fase de correção operacional:

- dependências isoladas no `.venv` correto;
- tratamento de exceções no worker;
- paralisação do worker de desenvolvimento;
- correção do parser de múltiplos `[TRECHO X]`;
- restauração dos links completos do Drive.

Essa mudança é particularmente reveladora.

O sistema já possuía:

**API + worker + banco + frontend + múltiplos ambientes.**

Com isso surgiram problemas que simplesmente não existiam no V1:

- concorrência entre workers;
- estados inconsistentes;
- parsing de citações;
- diferenças entre desenvolvimento e produção.

A complexidade foi consequência direta da arquitetura maior.

---

# 30. V2, SaaS, Core, Beta e Dev: como interpretar os nomes

O histórico apresenta nomenclatura que pode induzir a uma falsa sequência linear.

A interpretação correta é:

### V1
Filosofia **Local-First**.

### V2 / SaaS
Filosofia **centralizada/Cloud-Native/SaaS**.

### Core
Estado de consolidação do núcleo V2.

### Beta
Estado de desenvolvimento/teste.

### Dev
Linha de desenvolvimento.

Portanto:

> **V2 e SaaS representam principalmente a mesma grande transição arquitetural; Core/Beta/Dev são estados ou linhas dessa transição, não quatro arquiteturas completamente independentes.**

Isso explica por que branches como `v2-saas`, `v2-beta` e `v2-dev` devem ser lidas como partes de uma história ramificada, e não como versões sequenciais perfeitamente ordenadas.

---

# 31. Setembro — o problema de desenvolvimento em Windows

Em setembro, o sistema começou a ser executado em ambientes diferentes.

Produção:

- MegaWare;
- Debian/Linux;
- PM2;
- Cloudflare Tunnel;
- `acervo.snoopyflix.online`.

Desenvolvimento:

- HP;
- Windows;
- `dev.snoopyflix.online`;
- Cloudflare Tunnel;
- porta 3001.

O worker de desenvolvimento não chegou a ser usado de maneira ativa; o desenvolvimento usa o ambiente da máquina Windows para API/testes, enquanto o worker de produção permanece no servidor.

Essa configuração gerou um problema arquitetural relevante: produção e desenvolvimento compartilham o mesmo banco e a mesma tabela `jobs`.

O commit `add07a6` já havia mostrado que o worker de desenvolvimento precisou ser paralisado para evitar conflito.

Isso é historicamente importante porque mostra que o modelo atual de jobs evoluiu a partir de uma arquitetura essencialmente concebida para **um worker ativo por vez**.

---

# 32. 09/09 — cross-platform e timeout

O commit final atual:

`3a7f4ef4d0fc04eaa18cde1cb56ee01d8115b6f0`

de 09/09/2026:

**`fix: corrige compatibilidade cross-platform e timeout de sincronização no backend`**

resolve dois problemas que aparecem somente porque o sistema já alcançou uma arquitetura distribuída.

### Problema 1 — Python

O Node não podia mais assumir um único caminho:

```text
./.venv/bin/python3
````

A solução passou a detectar dinamicamente:

-  Windows: `.venv/Scripts/python.exe` 
-  Unix/Linux: `.venv/bin/python3` 

e força UTF-8 no processo filho.

### Problema 2 — sincronização

A rota `/api/sync` permanecia aberta enquanto downloads eram realizados.

Com Cloudflare/proxy, isso podia gerar timeout.

A solução foi separar:

**requisição HTTP → aceitação da sincronização → trabalho em background**

em vez de:

**requisição HTTP → executar toda a sincronização → responder.**

Esse é um excelente exemplo de evolução histórica por necessidade:

> a arquitetura assíncrona atual não é somente uma escolha abstrata de design; ela é também uma resposta a um problema concreto produzido pelo uso do sistema através da infraestrutura real.

---

# 33. A história da rastreabilidade

Uma das evoluções mais importantes do Snoopy pode ser descrita como quatro estágios.

## Estágio 1 — rastrear o documento

No V1:

**resultado → PDF do Drive**

A pergunta era:

> “De qual documento isso veio?”

## Estágio 2 — rastrear o trecho

Depois surgem:

`[TRECHO 1]`, `[TRECHO 2]`, etc.

A pergunta passa a ser:

> “Qual passagem sustentou a resposta?”

## Estágio 3 — mostrar contexto

Com a arquitetura de Bardin e a interface de evidências:

> “Qual é o contexto textual em que essa passagem aparece?”

## Estágio 4 — leitura documental

Com o Modo Leitura:

> “Posso sair da síntese e voltar à representação textual do documento para examiná-lo.”

Essa trajetória é mais importante historicamente do que qualquer recurso visual individual.

Ela mostra uma mudança de concepção:

**rastreabilidade de fonte → rastreabilidade de evidência.**

---

# 34. A história da pergunta de pesquisa

A própria pergunta feita ao sistema também evoluiu.

### Primeira formulação implícita

> “Qual PDF responde minha pergunta?”

### Depois

> “Quais trechos desses PDFs são relevantes?”

### Depois

> “Como impedir que o modelo invente?”

### Depois

> “Como mostrar de onde veio cada afirmação?”

### Depois

> “Como recuperar o contexto necessário para análise?”

### Depois

> “Como fazer isso de maneira reproduzível e auditável?”

Essa evolução ajuda a explicar por que o Snoopy atual parece muito maior do que o problema que o originou.

Ele não cresceu apenas porque mais funcionalidades foram adicionadas.

Ele cresceu porque **o próprio problema foi sendo redefinido conforme as limitações da solução anterior eram descobertas**.

---

# 35. O papel do NotebookLM na história

A descoberta posterior do NotebookLM não pertence ao desenvolvimento inicial do código, mas é importante para compreender a posição atual do projeto.

Quando o NotebookLM foi descoberto pelo usuário, ficou evidente que, para um estudante comum, o Snoopy podia parecer uma solução mais complexa para uma tarefa que o NotebookLM já executa de maneira conveniente.

Isso não invalida o projeto.

Na verdade, ajuda a esclarecer seu nicho.

A diferenciação pretendida passou a ser menos:

> “Snoopy consegue conversar com PDFs.”

e mais:

> **“Snoopy organiza recuperação documental de forma orientada à pesquisa, ao corpus, à evidência e à rastreabilidade.”**

Não há, entretanto, base experimental suficiente para afirmar que NotebookLM seja ou não reprodutível no mesmo sentido. Essa questão deve permanecer aberta até testes controlados.

---

# 36. O papel de Bardin na transformação do projeto

Bardin não estava no núcleo original.

Ele entra depois que o V1 já tinha um motor de busca funcional.

Essa ordem é metodologicamente importante.

O Snoopy não foi criado originalmente para “automatizar a Análise de Conteúdo”.

Ele foi adaptado para se tornar uma ferramenta que pudesse **apoiar a exploração de um corpus usado em pesquisa de Análise de Conteúdo**.

Essa distinção deve permanecer em toda documentação acadêmica.

O sistema não executa automaticamente a metodologia de Bardin.

Ele fornece infraestrutura computacional para partes preliminares ou auxiliares do trabalho:

-  localização; 
-  recuperação; 
-  organização; 
-  consulta; 
-  exposição de contexto; 
-  navegação entre evidência e documento. 

---

# 37. A evolução dos acervos

O conceito de acervo também sofreu uma evolução.

## Inicialmente

Uma pasta específica do Google Drive.

## Depois

Um conjunto de documentos indexados localmente.

## Depois

Uma coleção identificada por `folder_id`.

## Depois

Um recurso associado a usuário, estado, sincronização e permissões.

## Atualmente

O acervo é uma **fronteira lógica de corpus** dentro da infraestrutura.

Isso permite:

-  acervo pessoal; 
-  acervo público; 
-  múltiplos acervos; 
-  diferentes corpora; 
-  isolamento da busca; 
-  futura administração de acervos institucionais. 

O conceito de acervo, portanto, nasceu de uma necessidade de armazenamento/sincronização e terminou assumindo também um papel metodológico.

---

# 38. A evolução da identidade documental

Outro eixo histórico:

### V1

O documento era essencialmente um PDF local com link.

### V1 avançado

O documento ganha:

-  hash; 
-  metadata; 
-  ID; 
-  estado; 
-  link Drive. 

### V2

O documento passa a possuir uma identidade persistente no banco:

- `document_id`; 
- `folder_id`; 
- `drive_file_id`; 
- `document_hash`; 
-  título; 
-  autores; 
-  ano; 
-  link. 

### Atual

O documento é uma entidade que conecta:

**fonte externa → representação textual → chunks → embeddings → evidências → leitura.**

Essa transformação é central para a arquitetura atual.

---

# 39. A evolução do embedding

A história dos embeddings também mostra que a arquitetura não é independente dos fornecedores.

A sequência registrada inclui:

-  primeiros embeddings Gemini; 
- `gemini-embedding-001`; 
- `text-embedding-004`; 
-  posteriormente `gemini-embedding-2`; 
-  manutenção de 768 dimensões por compatibilidade. 

A migração para `gemini-embedding-2` ocorreu porque o `text-embedding-004` foi descontinuado pelo Google.

A decisão de manter 768 dimensões evitou reconstruir toda a estrutura vetorial.

Isso é historicamente relevante porque mostra que a “matemática” do banco não foi escolhida em isolamento: ela também precisou acompanhar mudanças da API externa.

---

# 40. A evolução da busca

A busca passou por pelo menos quatro formas.

## 40.1. ChromaDB direto

Consulta semântica local.

## 40.2. ChromaDB + FlashRank

Tentativa de melhorar a ordem dos resultados por reranking.

## 40.3. Supabase/pgvector + decomposição

A consulta passa por microqueries geradas por LLM.

## 40.4. Estado atual

A lógica é aproximadamente:

**pergunta → decomposição → embeddings das microqueries →** **`match_chunks`** **→ união/deduplicação → seleção de contextos → síntese.**

O FlashRank foi posteriormente removido do caminho principal por custo de CPU/tempo.

Isso é importante para a documentação: o sistema atual não deve ser descrito como “ChromaDB + FlashRank”, mesmo que isso faça parte da história.

---

# 41. O que “determinístico” significava em cada momento

O histórico utiliza linguagem como “determinístico”, mas o significado mudou.

Na V1, determinismo estava associado principalmente a:

-  IDs; 
-  hash; 
-  controle de duplicação; 
-  estrutura de processamento. 

Na V2, o conceito passa a envolver também:

-  corpus; 
- `folder_id`; 
-  documentos; 
-  chunks; 
-  recuperação. 

Hoje, entretanto, não existe evidência suficiente para dizer que a recuperação inteira é formalmente determinística.

Há fatores variáveis:

-  decomposição feita por LLM; 
-  embeddings; 
-  ordenação/ties do retrieval; 
-  cache; 
-  modelo; 
-  parâmetros. 

Assim, a reconstrução histórica deve registrar:

> **a intenção de determinismo/reprodutibilidade apareceu cedo, mas sua interpretação como requisito metodológico rigoroso foi amadurecendo depois e ainda não constitui uma garantia formal no código atual.**

---

# 42. A evolução do cache

O cache também mudou de função.

No V1, o cache era principalmente uma otimização:

**não processe novamente aquilo que já foi processado.**

Depois surgem mecanismos de cache relacionados a:

-  hash do documento; 
-  embeddings; 
-  resultados de pesquisa. 

No estado atual, o histórico de busca e o cache podem interferir na observação de experimentos de reprodutibilidade.

Isso cria uma distinção que deve permanecer na documentação:

**repetir uma consulta usando o mesmo resultado em cache não equivale a demonstrar que o motor recuperaria os mesmos trechos novamente.**

---

# 43. A evolução do worker

A V1 tinha orquestradores locais como o `watcher`.

A V2 substitui esse modelo por uma tabela de `jobs` e um worker dedicado.

Isso cria um sistema conceitualmente mais robusto:

**API cria trabalho → job persistente → worker processa → banco registra estado → frontend observa.**

Mas a implementação atual ainda assume várias condições:

-  worker efetivamente único; 
-  compartilhamento de banco entre ambientes; 
-  ausência de um identificador de ambiente na tabela `jobs`; 
-  coordenação operacional por configuração. 

Assim, o worker representa um avanço arquitetural enorme, mas também introduz uma nova classe de problemas que não existia no V1.

---

# 44. O desenvolvimento assistido por IA como elemento histórico

O processo de desenvolvimento também faz parte da história do projeto.

A implementação foi fortemente assistida por modelos de IA, especialmente para execução de código, enquanto o usuário exerceu principalmente funções de:

-  definição de requisitos; 
-  arquitetura; 
-  revisão; 
-  testes; 
-  curadoria; 
-  identificação de problemas; 
-  tomada de decisões. 

Isso significa que o histórico do Snoopy não é adequadamente descrito nem como:

> “Fabrício escreveu manualmente todo o código”

nem como:

> “a IA construiu o projeto sozinha”.

O modelo mais preciso é:

> **desenvolvimento assistido por IA, com direção, curadoria e validação humanas.**

Isso será importante em documentação futura se houver necessidade de explicar a autoria do projeto.

---

# 45. O que não deve ser confundido como causa

A reconstrução histórica permite eliminar algumas interpretações erradas.

## Não foi assim:

“Bardin foi o motivo de o Snoopy existir.”

Foi o contrário:

**o Snoopy já existia; Bardin alterou sua direção metodológica.**

## Não foi assim:

“Multiusuário era o objetivo original.”

Mais precisamente:

**o problema de distribuição/acesso levou à centralização e, daí, à necessidade de multiacervo/multiusuário.**

## Não foi assim:

“O Snoopy foi concebido desde o início como um RAG acadêmico completo.”

Mais precisamente:

**o RAG apareceu durante a construção e só posteriormente recebeu uma moldura metodológica mais explícita.**

## Não foi assim:

“Reprodutibilidade já estava garantida no V1.”

Mais precisamente:

**havia preocupações com determinismo, duplicação e rastreabilidade; o requisito de reprodutibilidade metodológica foi posteriormente refinado e ainda não está formalmente garantido.**

---

# 46. O grande movimento histórico

A história completa pode ser resumida em cinco transformações.

## 46.1. De arquivo para corpus

**PDF isolado → coleção documental.**

## 46.2. De corpus local para corpus persistente

**arquivos locais → Supabase.**

## 46.3. De resposta para evidência

**síntese → trechos recuperados → contexto → documento.**

## 46.4. De ferramenta pessoal para infraestrutura

**aplicação local → SaaS/multiacervo/multiusuário.**

## 46.5. De busca semântica para suporte metodológico

**“encontre algo parecido” → “recupere evidência de um corpus de pesquisa de forma rastreável e examinável”.**

---

# 47. A causalidade mais provável da evolução

A cadeia causal reconstruída é:

**necessidade de localizar literatura**

→ criação do motor local

→ necessidade de pesquisar vários PDFs

→ criação do pipeline em lote

→ necessidade de responder via interface

→ necessidade de acesso por diferentes máquinas

→ centralização em Supabase

→ necessidade de separar acervos/usuários

→ necessidade de evidência mais explícita

→ entrada de Bardin

→ necessidade de contexto e referência

→ Modo Leitura

→ necessidade de ingestão assíncrona observável

→ jobs + Realtime

→ uso real em ambientes distintos

→ problemas de worker, tokens, proxy e cross-platform

→ arquitetura atual.

Essa cadeia é mais fiel à história do que uma sequência de “versões”.

---

# 48. O que a história revela sobre a arquitetura atual

A arquitetura atual contém camadas que correspondem diretamente a problemas históricos anteriores.

| Componente atualProblema histórico que ajudou a originá-lo |                                   |
| ---------------------------------------------------------- | --------------------------------- |
| `folders`                                                  | múltiplos acervos                 |
| `documents`                                                | identidade documental persistente |
| `chunks`                                                   | recuperação semântica             |
| `jobs`                                                     | processamento assíncrono          |
| `search_history`                                           | persistência da pesquisa          |
| `drive_file_id`                                            | retorno à fonte                   |
| `document_hash`                                            | deduplicação                      |
| `folder_id`                                                | isolamento de corpus              |
| RLS                                                        | multiusuário                      |
| Realtime                                                   | visibilidade do processamento     |
| Reading Mode                                               | rastreabilidade/contexto          |
| ABNT metadata                                              | pesquisa acadêmica                |
| SSE                                                        | comunicação de processamento      |
| cache                                                      | economia de processamento         |
| dynamic Python path                                        | múltiplos ambientes               |
| background sync                                            | timeout/proxy                     |

A arquitetura, portanto, pode ser lida como uma espécie de **sedimentação das soluções para problemas encontrados ao longo do desenvolvimento**.

---

# 49. O que ainda permanece historicamente aberto

Nem todas as lacunas podem ser resolvidas pelo Git.

Permanecem perguntas como:

1.  quais testes específicos foram realizados em cada estágio; 
2.  quais documentos constituíam os corpora de teste em cada momento; 
3.  quais resultados de retrieval foram considerados bons ou ruins; 
4.  quando exatamente cada decisão foi tomada em conversa; 
5.  quais ideias foram abandonadas sem chegar a commit; 
6.  como cada modelo de embedding foi comparado empiricamente; 
7.  se houve experimentos sistemáticos de reprodutibilidade; 
8.  quais foram os critérios concretos usados para considerar uma versão “funcional”. 

Essas lacunas devem permanecer explicitamente marcadas.

Não devem ser preenchidas com inferência retrospectiva.

---

# 50. PRs e branches: conclusão histórica

O histórico de branches mostra:

- `main`: linha de produção/fonte de verdade atual; 
- `v2-saas`: linha histórica de consolidação SaaS; 
- `v2-beta`: estado intermediário; 
- `v2-dev`: desenvolvimento recente. 

O merge final de setembro levou `v2-dev` para `main`.

O PR #3 registra:

-  base: `add07a6`; 
-  head: `57ba476`; 
-  merge: `3a7f4ef`. 

Portanto, o estado atual de `main` incorpora a mudança cross-platform/background que estava em `v2-dev`.

O PR #2, embora contenha a mesma mudança em sua descrição, não foi merged. Não deve ser tratado como origem efetiva do estado atual.

O PR #1, por sua vez, foi merged, mas sem alterações próprias, funcionando essencialmente como release marker.

---

# 51. Marco zero, V1 e V2: definição histórica consolidada

### Marco zero

Ideia de facilitar a localização de documentos acadêmicos em uma pasta do Drive.

### V1 — Local-First

Construção do motor:

**Drive → PDF → texto → chunk → embedding → busca → resposta/evidência.**

### V2 — Cloud/SaaS

Transformação do motor em infraestrutura:

**usuário → acervo → sincronização → jobs → pipeline → banco central → busca → evidência.**

### V2 metodológico

Integração da necessidade de pesquisa:

**pergunta → recuperação → contexto → documento → referência → leitura.**

### V2 operacional

Transformação em serviço utilizável:

**API + worker + Realtime + OAuth + Cloudflare + PM2 + múltiplos ambientes.**

---

# 52. Avaliação histórica final

A principal conclusão desta etapa é que o Snoopy-RAG não deve ser descrito historicamente como um projeto que simplesmente “cresceu”.

Ele **mudou de natureza várias vezes**.

Primeiro era uma ferramenta de recuperação documental local.

Depois virou um RAG.

Depois virou uma aplicação centralizada de acervos.

Depois virou uma infraestrutura multiusuário.

Depois passou a ser orientado à evidência e ao contexto.

Depois recebeu uma função explícita de apoio à pesquisa de Análise de Conteúdo.

E finalmente passou a exigir uma infraestrutura operacional própria para sustentar tudo isso.

Cada transformação deixou restos da anterior no código, nos nomes, nas branches e nas decisões.

É justamente por isso que a história apresenta nomenclaturas aparentemente contraditórias, componentes que foram substituídos, configurações antigas e commits com responsabilidades diferentes.

Isso não é simplesmente “bagunça”.

É também a **história material da evolução do projeto**.

---

# 53. Princípios históricos que devem ser preservados nas próximas etapas

1.  Não apagar a história V1 só porque o V2 é superior. 
2.  Não chamar toda a arquitetura atual de “planejada desde o início”. 
3.  Distinguir intenção de implementação efetiva. 
4.  Distinguir commit de decisão causal. 
5.  Distinguir branch de versão conceitual. 
6.  Distinguir chunk técnico de unidade metodológica. 
7.  Distinguir rastreabilidade de documento de rastreabilidade de evidência. 
8.  Distinguir reprodução de cache de reprodutibilidade do retrieval. 
9.  Não apresentar limitações atuais como se fossem decisões deliberadas originais. 
10.  Não atribuir a Bardin o nascimento do projeto. 
11.  Não atribuir ao SaaS um objetivo original que ele não teve. 
12.  Preservar o papel do usuário como arquiteto/curador/supervisor em um processo de desenvolvimento assistido por IA. 
13.  Tratar as branches históricas como evidências complementares, não como uma sequência artificial. 
14.  Registrar explicitamente aquilo que o Git não consegue provar. 

---

# 54. Status da Etapa 3

**Reconstrução histórica principal: concluída.**

A etapa está suficientemente consolidada para alimentar a próxima análise sem precisar reabrir toda a história do projeto do zero.

A principal contribuição da Etapa 3 para as etapas seguintes é estabelecer uma distinção que será essencial:

> **o sistema atual é o resultado de uma sequência de respostas a problemas concretos, e não a implementação linear de uma especificação inicial completa.**

Isso permite que a Etapa 4 — análise metodológica — seja feita sobre o sistema real e sobre a trajetória que o levou até ele, sem confundir:

-  RAG com método de pesquisa; 
-  chunk com unidade de registro; 
-  recuperação com interpretação; 
-  evidência computacional com evidência científica; 
-  intenção de reprodutibilidade com garantia formal. 

---

# 55. Próxima etapa

## Etapa 4 — Análise metodológica e epistemológica

A próxima etapa deverá investigar, com base na arquitetura histórica reconstruída:

1.  o que exatamente o Snoopy faz no processo de pesquisa; 
2.  em que ponto começa e termina a recuperação documental; 
3.  o que pertence ao trabalho do pesquisador; 
4.  como RAG se relaciona com exploração de corpus; 
5.  como Bardin se relaciona com o sistema sem ser “automatizado” por ele; 
6.  o estatuto de chunk, trecho e contexto; 
7.  unidade de registro versus unidade de contexto; 
8.  evidência documental versus síntese gerada; 
9.  rastreabilidade; 
10.  auditabilidade; 
11.  reprodutibilidade; 
12.  limites epistemológicos do uso de LLM; 
13.  riscos de automatização excessiva; 
14.  possibilidades reais de uso em uma pesquisa acadêmica. 

A Etapa 4 deverá partir da história reconstruída aqui, mas não ficará presa a ela: deverá confrontar o comportamento do sistema com a literatura metodológica e técnica pertinente.

---

## Apêndice A — Marcos Git de maior relevância histórica

| SHADataMarco |       |                                  |
| ------------ | ----- | -------------------------------- |
| `3d5f3f9`    | 09/05 | nascimento formal do repositório |
| `dbd9f77`    | 10/05 | extração/limpeza PDF             |
| `0c019b4`    | 10/05 | chunking                         |
| `50f9091`    | 10/05 | ChromaDB + embeddings            |
| `14c6bea`    | 11/05 | pipeline em lote                 |
| `d48262e`    | 11/05 | hash/cache/rastreabilidade       |
| `347962c`    | 12/05 | pipeline efêmero                 |
| `63bb9f2`    | 12/05 | `[TRECHO X]`                     |
| `0fdf228`    | 12/05 | CLI → API                        |
| `f81daeac`   | 14/05 | Auth + UI SaaS inicial           |
| `95095e0`    | 16/05 | V2 Core/Supabase/RLS             |
| `69ce78d`    | 16/05 | SSE + acervos                    |
| `a6721fc`    | 17/05 | Google Picker + sync             |
| `01d1681`    | 17/05 | histórico cloud + soft delete    |
| `821b825`    | 17/05 | limpeza arquitetural V1          |
| `c29857d`    | 18/05 | documentação V2                  |
| `1c3eed5`    | 21/05 | Bardin/ABNT/contexto             |
| `f165147`    | 21/05 | UI/acervos/mobile                |
| `372411f`    | 22/05 | otimização                       |
| `e6633e6`    | 26/05 | V2.0.0 Core + Reading Mode       |
| `f86ab02`    | 27/05 | OCR/infra/translation            |
| `ee50156`    | 16/06 | refinamento mobile               |
| `f8dd5a7`    | 15/07 | SaaS Realtime/jobs/backoff/OAuth |
| `add07a6`    | 15/07 | worker/links/parser              |
| `3a7f4ef`    | 09/09 | cross-platform + sync background |

---

## Apêndice B — fórmula histórica resumida

```
```

```
PROBLEMA ORIGINAL
    ↓
"Como achar o PDF relevante?"
    ↓
V1 LOCAL
    PDF
    ↓
    texto
    ↓
    chunks
    ↓
    embeddings
    ↓
    busca semântica
    ↓
    resposta + fonte
    ↓
PROBLEMAS DE ESCALA/ACESSO
    ↓
V2 / SAAS
    usuário
    ↓
    acervo
    ↓
    Google Drive
    ↓
    jobs
    ↓
    pipeline
    ↓
    Supabase
    ↓
    chunks + embeddings
    ↓
    busca
    ↓
PROBLEMA METODOLÓGICO
    ↓
    "De onde veio a resposta?"
    ↓
    trecho
    ↓
    contexto
    ↓
    documento
    ↓
    referência
    ↓
    Reading Mode
    ↓
PROBLEMA OPERACIONAL
    ↓
    worker / realtime / OAuth / proxy / Windows/Linux
    ↓
ESTADO ATUAL
    ↓
    infraestrutura de recuperação documental orientada à evidência
```

---

**Fim do Registro Mestre da Etapa 3.**
