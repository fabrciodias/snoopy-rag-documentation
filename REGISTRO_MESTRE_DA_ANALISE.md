# Registro Mestre da Análise — Snoopy-RAG

> Documento de trabalho interno da análise do projeto.
> Não é README, documentação técnica final nem documento acadêmico.
> Sua função é preservar o raciocínio, as evidências, as decisões e as conclusões obtidas durante as Etapas 0 e 1 da análise, para que possam ser retomadas posteriormente sem depender da memória da conversa.

**Data da consolidação:** 10/09/2026  
**Base principal analisada:** branch `main`  
**Commit mais recente analisado:** `3a7f4ef4d0fc04eaa18cde1cb56ee01d8115b6f0`  
**Versão indicada pelo projeto:** `2.0.1-beta`

---

# ETAPA 0 — ALINHAMENTO

## 0.1. Objetivo desta análise

A análise do Snoopy-RAG não deve ser tratada simplesmente como uma revisão de código. O objetivo é compreender o sistema como um projeto técnico, metodológico e histórico: de onde surgiu, qual problema pretendia resolver, como sua arquitetura evoluiu, quais decisões foram tomadas, o que o sistema efetivamente faz hoje, quais limitações possui e até onde suas funções podem ser relacionadas à pesquisa científica e à Análise de Conteúdo.

A documentação posterior deverá ser produzida a partir deste registro, e não o contrário. Por isso, este documento preserva inclusive decisões provisórias, fragilidades e distinções conceituais que não necessariamente aparecerão em um README ou manual de uso.

## 0.2. Problema que originou o projeto

O Snoopy-RAG nasceu de uma necessidade prática ligada ao acervo de artigos científicos utilizado no GEPAFOR.

A ideia inicial estava relacionada a uma pasta do Google Drive contendo PDFs de pesquisas. O problema era simples de formular: diante de uma pergunta de pesquisa, como localizar rapidamente, dentro daquele conjunto de PDFs, os documentos e trechos potencialmente relevantes?

A ideia inicial não começou com uma arquitetura RAG completamente formulada. O conceito de RAG foi sendo explicitado durante a implementação.

O primeiro objetivo era essencialmente:

**Google Drive → PDFs → processamento → busca → resposta curta + indicação do PDF de origem.**

A necessidade de localizar a fonte original sempre esteve presente. A síntese por IA era útil, mas a possibilidade de retornar ao documento original era fundamental.

## 0.3. Evolução do problema

O problema técnico foi se tornando mais complexo à medida que o sistema era usado e discutido.

A sequência conceitual reconstruída é aproximadamente:

1. **“Como pesquisar os PDFs desta pasta?”**
2. **“Como fazer isso funcionar para mais de uma pessoa?”**
3. **“Como saber de onde veio a resposta?”**
4. **“Como apresentar o contexto do trecho?”**
5. **“Como relacionar a recuperação ao trabalho de Análise de Conteúdo?”**
6. **“Como tornar a recuperação de evidências auditável e reproduzível?”**

A introdução da perspectiva de Bardin ocorreu depois que o motor de busca textual/semântico da V1 já estava funcional. Isso alterou significativamente o significado do projeto: o sistema deixou de ser apenas uma ferramenta de perguntas sobre PDFs e passou a ser pensado como apoio computacional à exploração documental.

## 0.4. Critério de reprodutibilidade adotado

Um dos critérios centrais definidos durante a análise é:

> Se duas pessoas possuem exatamente o mesmo corpus e fazem exatamente a mesma pergunta, os trechos recuperados devem ser os mesmos.

Uma formulação mais realista também foi estabelecida:

> Mesmo que não possuam o corpus inteiro, se possuem os mesmos documentos relevantes e utilizam a mesma consulta e configuração de recuperação, os trechos recuperados devem ser os mesmos.

A preocupação principal é, portanto, a **reprodutibilidade da evidência recuperada**, e não a reprodução literal da redação produzida pelo LLM.

## 0.5. Relação com Bardin

O Snoopy-RAG não implementa o método de Bardin automaticamente.

A distinção fundamental estabelecida é:

- **chunk** = unidade técnica de segmentação criada pelo sistema;
- **unidade de registro** = unidade metodológica identificada pelo pesquisador;
- **unidade de contexto** = contexto utilizado para compreender adequadamente a unidade de registro.

Um trecho retornado pelo Snoopy pode servir como material documental para a análise, mas não se transforma automaticamente em unidade de registro simplesmente por ter sido recuperado pelo algoritmo.

A função do sistema é auxiliar o pesquisador na localização, recuperação e organização do material. A identificação, delimitação, codificação, categorização e interpretação continuam sendo atividades do pesquisador.

## 0.6. Papel do pesquisador

O sistema deve ser entendido como ferramenta de apoio à pesquisa.

A arquitetura pode localizar documentos, recuperar trechos semanticamente relacionados, apresentar contexto, preservar metadados, indicar a fonte original e organizar resultados.

Não deve assumir automaticamente a decisão metodológica final, a interpretação do pesquisador, a criação de categorias válidas, a definição definitiva das unidades de registro ou a validação científica de uma conclusão.

## 0.7. Motivação inicial e público

A primeira necessidade surgiu dentro do fluxo de trabalho do GEPAFOR. A possibilidade de transformar a ferramenta em SaaS veio posteriormente, principalmente porque uma instalação local por usuário seria inconveniente.

O objetivo de múltiplos usuários não foi o ponto inicial do projeto; ele apareceu como consequência da necessidade de acesso em diferentes dispositivos e de reduzir a dependência de instalação local.

Posteriormente, houve interesse em uma disponibilização pública mais ampla. Essa possibilidade perdeu prioridade devido às exigências e dificuldades relacionadas à configuração/validação pública do Google OAuth. O foco retornou ao GEPAFOR e a usuários de teste controlados.

A existência de acervos privados continua sendo conceitualmente importante: diferentes pesquisadores podem trabalhar com diferentes corpora, bases, periódicos e recortes, sem que todos os documentos precisem ser misturados em um único acervo.

---

# ETAPA 1 — AUTÓPSIA E LEVANTAMENTO DO SISTEMA

## 1.1. Identidade do repositório

Repositório analisado:

`fabrciodias/snoopy-rag`

Branches relevantes:

- `main`
- `v2-dev`
- `v2-saas`
- `v2-beta`

Interpretação histórica:

- **V1** = filosofia/arquitetura Local-First;
- **V2 / SaaS** = mudança arquitetural para filosofia centralizada/SaaS;
- **Core, Beta e Dev** = estados/linhas de desenvolvimento da V2.

A nomenclatura histórica não deve ser artificialmente corrigida, pois isso apagaria parte da evolução real.

## 1.2. Estado atual de `main`

Commit mais recente:

`3a7f4ef4d0fc04eaa18cde1cb56ee01d8115b6f0`

Mensagem:

`fix: corrige compatibilidade cross-platform e timeout de sincronização no backend`

Mudanças relevantes:

- resolução dinâmica do executável Python;
- UTF-8 garantido no processo filho;
- processamento de `/api/sync` desacoplado da requisição HTTP;
- processamento em background para evitar timeout de proxy/Cloudflare;
- ajustes na interface;
- versão `2.0.1-beta`.

O comportamento atual de sync deve ser descrito como:

**solicitação aceita → processamento iniciado em background → acompanhamento por estado/job.**

## 1.3. Estrutura principal

```text
.gitignore
LICENSE
README.MD
README_V1.md
package.json
requirements.txt
server.js
src/
  chunker.py
  cleaner.py
  extractor.py
  pipeline.py
  search.py
  tagger.py
  worker.py
ui/
  api.js
  auth.js
  index.html
  main.js
  style.css
  ui.js
```

Existe também configuração local do Supabase CLI (`supabase/`) no ambiente de desenvolvimento.

## 1.4. Arquitetura geral

```text
                    ┌──────────────────────┐
                    │      Navegador       │
                    │ HTML/CSS/JS vanilla  │
                    └──────────┬───────────┘
                               │ HTTP / SSE
                               ▼
                    ┌──────────────────────┐
                    │ Node / Express       │
                    │     server.js        │
                    └───────┬───────┬──────┘
                            │       │
                            │       └──────────────► Google Drive API
                            ▼
                    ┌──────────────────────┐
                    │ Python               │
                    │ search.py / worker   │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Pipeline de ingestão │
                    │       pipeline.py    │
                    └──────────┬───────────┘
                               │
                ┌──────────────┼──────────────┐
                ▼              ▼              ▼
           extractor.py    cleaner.py     tagger.py
                │              │              │
                └──────────────┼──────────────┘
                               ▼
                         chunker.py
                               │
                               ▼
                       Gemini Embeddings
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Supabase / Postgres  │
                    │       pgvector       │
                    └──────────┬───────────┘
                               │
                               ▼
                         match_chunks()
                               │
                               ▼
                         search.py
                               │
                               ▼
                            LLM
                               │
                               ▼
                  resposta + fontes/trechos
```

## 1.5. Ingestão

O fluxo começa com a seleção de uma pasta do Google Drive por meio do Google Picker.

O backend consulta PDFs (`mimeType=application/pdf`, `trashed=false`) e compara arquivos encontrados com documentos já processados e jobs já enfileirados.

```text
arquivos encontrados no Drive
        -
arquivos já processados
        -
arquivos já enfileirados
        =
novos arquivos a processar
```

O processamento efetivo é realizado pelo worker.

## 1.6. Worker e jobs

O worker consulta `jobs` e processa trabalhos pendentes em ordem de criação.

```text
pending
   ↓
processing
   ↓
completed
```

Em caso de falha:

```text
processing
   ↓
failed
```

O job registra usuário, pasta, nome do arquivo, ID do Drive, status, log de erro, timestamps e progresso.

Fragilidade importante: produção e desenvolvimento compartilham o mesmo banco Supabase e a tabela `jobs`, sem marcação explícita de ambiente. Um worker de desenvolvimento poderia potencialmente disputar jobs com produção. O worker de desenvolvimento no HP ainda não foi iniciado, portanto a fragilidade existe arquiteturalmente, mas não está sendo exercida pelo fluxo normal atual.

## 1.7. Pipeline

```text
PDF
 ↓
MD5
 ↓
extração de texto
 ↓
limpeza
 ↓
metadados via LLM
 ↓
chunking
 ↓
embeddings
 ↓
Supabase
```

O MD5 é calculado antes do processamento pesado.

A deduplicação representa identidade binária do PDF dentro do contexto da pasta; não é detecção semântica de documentos equivalentes.

## 1.8. Extração

O `extractor.py` utiliza PyMuPDF com `get_text("blocks")`, mantendo blocos textuais e normalizando espaços/quebras.

A representação textual pode perder ou distorcer:

- tabelas;
- diagramas;
- relações espaciais;
- elementos gráficos;
- fórmulas;
- estruturas complexas de layout.

## 1.9. Cleaner

Foi identificada uma questão no fluxo de atribuição de texto do cleaner: uma segunda expressão regular atribuía novamente ao texto bruto, potencialmente descartando uma limpeza anterior.

O usuário realizou a correção localmente, mas ela ainda não havia sido incorporada ao `main` nesta consolidação.

## 1.10. Chunker

O `chunker.py` realiza segmentação estrutural/heurística considerando parágrafos, seções Markdown `##`, agrupamento de parágrafos e tamanho aproximado de 1500 caracteres.

O chunk é uma decisão técnica de recuperação e não uma decisão metodológica de Análise de Conteúdo.

Foi considerada uma futura segunda camada de segmentação destinada à apresentação/recuperação de contexto, mas isso não faz parte da arquitetura atual.

## 1.11. Tagger e metadados

O `tagger.py` utiliza LLM para extrair:

- `titulo_original`;
- `autores`;
- `ano_publicacao`;
- `tipo_documental`;
- `idioma`;
- `palavras_chave`.

A estrutura já permite futuros filtros por autor, ano/período e outros metadados.

## 1.12. Embeddings

Os chunks são transformados em embeddings e armazenados em pgvector.

O schema atual registra:

```text
embedding public.vector(768)
```

O histórico passou por `gemini-embedding-001` com 768 dimensões. A configuração atual utiliza o modelo de embedding definido pelo ambiente do projeto.

## 1.13. Busca semântica

`search.py` utiliza LLM para decompor a pergunta em microconsultas, procurando evitar mistura indevida de autores/conceitos.

Cada microconsulta é embeddada e enviada ao RPC `match_chunks` com:

```text
match_threshold = 0.4
match_count = 5
p_user_id
p_folder_id
```

Os resultados são coletados, deduplicados e os primeiros são enviados ao LLM para síntese.

FlashRank/reranking existiu historicamente, mas não está presente na implementação atual de `search.py`.

## 1.14. `match_chunks`

A função usa:

```sql
c.embedding <=> query_embedding
```

e:

```sql
1 - (c.embedding <=> query_embedding)
```

para calcular similaridade.

A ordenação é por distância crescente:

```sql
ORDER BY c.embedding <=> query_embedding
```

com limite de `match_count`.

O filtro exige `folder_id = p_folder_id` e inclui condição relacionada ao usuário ou à pasta pública GEPAFOR.

## 1.15. Índice vetorial

O schema exportado analisado não apresentou declarações explícitas de índices HNSW ou IVFFlat.

Isso deve ser registrado como constatação do export analisado, não como afirmação absoluta sobre qualquer estado futuro do banco.

## 1.16. Banco de dados

Tabelas principais:

```text
folders
documents
chunks
jobs
search_history
```

`folders` representa o acervo lógico.

`documents` representa documentos ingeridos e preserva título, autores, ano, hash, link/ID do Drive, conteúdo etc.

`chunks` representa segmentos usados na recuperação.

`jobs` controla processamento assíncrono.

`search_history` registra consultas.

## 1.17. Acervo público GEPAFOR

Existe uma pasta pública/lógica associada a um UUID específico utilizado na função `match_chunks` e nas políticas RLS.

Os valores nulos possuem significado intencional:

```text
folders.user_id = NULL
```

significa que a pasta não pertence a um usuário individual.

```text
folders.drive_id = NULL
```

significa que a pasta lógica pública existe no Supabase, mas ainda não está ligada a uma pasta real do Google Drive.

Quando o acervo público/laboratório tiver uma pasta Drive configurada, o `drive_id` será preenchido.

O UUID público hardcoded deverá futuramente ser substituído por configuração/administração mais explícita.

## 1.18. Administração versus propriedade

A distinção é:

**proprietário da pasta ≠ administrador da pasta.**

A pasta pública pode não possuir `user_id`, mas ainda pode ter uma conta ROOT/admin responsável por administrá-la.

Modelo futuro conceitual:

```text
pasta pública
OU
pasta pertencente ao usuário
OU
usuário ROOT/admin autorizado
```

## 1.19. RLS

RLS está habilitado nas tabelas principais.

Há políticas para acesso aos próprios registros e leitura pública do acervo GEPAFOR onde apropriado.

Porém o backend utiliza `SUPABASE_SERVICE_KEY`.

A service role bypassa RLS; portanto, RLS não substitui a autorização do backend.

O backend precisa validar que a pasta solicitada é:

- pertencente ao usuário;
- pública;
- ou administrável pelo usuário autorizado.

## 1.20. Possível fragilidade de autorização

A rota de recuperação de chunks/documentos que recebe `folder_id` e utiliza service role exige atenção.

Não foi afirmada uma exploração comprovada. O ponto identificado é arquitetural: se o backend confiar diretamente em um `folder_id` fornecido pelo cliente sem verificar propriedade/publicidade/administração, pode existir risco de acesso indevido entre acervos.

Deve ser testado posteriormente.

## 1.21. Realtime

`jobs` foi incluída na publicação realtime do Supabase, sustentando atualizações de estado na interface.

## 1.22. Evidência e rastreabilidade

A apresentação de evidência evoluiu em etapas.

Na primeira fase, `[TRECHO X]` permitia chegar ao PDF original no Drive.

Depois, foi criada arquitetura para apresentar trecho/contexto original.

Posteriormente surgiu Reading Mode, com visualização do texto extraído integralmente do documento.

A evolução pode ser resumida:

```text
resultado
  ↓
fonte original
  ↓
trecho/contexto
  ↓
leitura do documento
```

## 1.23. Produto principal

O produto metodologicamente mais importante não é a frase gerada pelo LLM.

É:

```text
pergunta
   ↓
recuperação
   ↓
evidências/trechos
   ↓
fonte
```

A síntese é uma camada auxiliar.

## 1.24. Reprodutibilidade atual

A intenção de reprodutibilidade é clara, mas a garantia formal ainda não existe.

Possíveis fontes de variação:

- decomposição da consulta por LLM;
- temperatura não nula;
- ordenação/tie-breaking;
- comportamento do índice;
- versão do chunking;
- modelo de embedding;
- alterações no corpus;
- cache;
- versão do código.

Conclusão:

**Intenção:** recuperação reprodutível.

**Estado demonstrado pelo código atual:** ainda não há garantia formal de reprodução em todas as condições.

## 1.25. Cache

O sistema armazena histórico/resultados de busca e pode reutilizar resultados para reduzir processamento e tokens.

Testes de reprodutibilidade devem distinguir cache de novo processamento.

## 1.26. Perfil de recuperação futuro

Convém registrar:

```text
embedding_model
embedding_dimensions
chunking_version
similarity_metric
similarity_threshold
top_k
query_decomposition_version
ranking_version
corpus_version
```

## 1.27. Testes futuros

1. mesma pergunta + mesmo corpus + mesma configuração, repetida;
2. dois usuários no mesmo corpus público;
3. adição de documento irrelevante;
4. adição de documento relevante;
5. duplicatas em pastas diferentes;
6. reingestão;
7. com/sem cache;
8. versões diferentes do chunker;
9. versões diferentes do embedding;
10. alterações no RPC.

---

# 1.28. Infraestrutura

## Produção

MegaWare, Debian/Linux.

```text
acervo.snoopyflix.online
        ↓
Cloudflare Tunnel: Snoopy-Prod
        ↓
localhost:3000
```

## Desenvolvimento

HP, Windows.

```text
dev.snoopyflix.online
        ↓
Cloudflare Tunnel: Snoopy-Dev
        ↓
localhost:3001
```

O usuário testa localmente e externamente, inclusive por telefone.

## PM2

Na máquina de produção existem:

```text
prod-api
prod-worker
dev-api
dev-worker
```

Atualmente ativos na produção:

```text
prod-api
prod-worker
```

`dev-api` e `dev-worker` estão parados e não representam o ambiente real de desenvolvimento.

## 1.29. Compatibilidade cross-platform

O sistema precisa operar entre Linux e Windows, com possibilidade futura de clientes Android/tablet.

O último commit eliminou dependência de caminhos manuais do executável Python por meio de resolução dinâmica e UTF-8.

## 1.30. Google Cloud

Serviços efetivamente relevantes ao fluxo incluem:

- Google OAuth;
- Google Drive API;
- Google Picker;
- serviços Gemini utilizados pelo projeto.

A documentação final deve listar apenas APIs cujo uso seja comprovado.

## 1.31. Google Picker

A API key utilizada pelo Picker foi observada sem restrição de aplicação.

Isso é item de hardening futuro, não bloqueador funcional.

## 1.32. Google OAuth

O fluxo efetivo utiliza Google OAuth.

Supabase Auth possui Google e Email habilitados, mas a interface utiliza Google.

Há configurações antigas, incluindo:

```text
v2beta.snoopyflix.online
```

e:

```text
http://localhost:3333
```

Essas configurações são tratadas como legado/higiene de configuração, sem classificá-las automaticamente como vulnerabilidade crítica.

## 1.33. Banco compartilhado

Produção e desenvolvimento usam o mesmo Supabase.

O usuário utiliza exclusão do próprio usuário para resetar dados de teste, contando com `ON DELETE CASCADE`.

Existe também script SQL capaz de limpar todas as tabelas/usuários. Sua execução afetaria todos os usuários e exigiria nova autenticação, permissões e reprocessamento.

Estado atual: funcional, porém operacionalmente frágil.

## 1.34. Fluxo futuro

Foi considerada uma rotina única no `package.json` para iniciar:

```text
server
+
worker
```

Isso ainda não existe e deve ser tratado apenas como melhoria futura.

---

# 1.35. Histórico V1 → V2

## V1 — Local-First

Fluxo:

```text
Google Drive
    ↓
download local
    ↓
Markdown
    ↓
chunks
    ↓
embeddings
    ↓
ChromaDB
    ↓
busca semântica
    ↓
terminal
```

Principais marcos:

```text
dbd9f77  extractor PyMuPDF + cleaner
16e3ede  Gemini metadata tagger
0c019b4  chunking
50f9091  ChromaDB + embeddings
517af15  terminal semantic search
```

FlashRank/reranking também existiu historicamente.

## Transição V2

A necessidade de múltiplos usuários, acesso em dispositivos diferentes, persistência e acervo centralizado levou à arquitetura SaaS/Supabase.

## Fundação V2 Core

Commit:

`95095e02126149470479c4ceb4773f1cd847e4fa`

Consolidou:

- `folders`;
- `documents`;
- `chunks`;
- `jobs`;
- RLS;
- constraints;
- `match_chunks` com `folder_id`;
- MD5 em memória;
- isolamento lógico por pasta;
- saída estruturada para futura auditoria.

Outro marco:

`e6633e65a65c0ce4694daaf5526ba7b93a849141`

relacionado à V2.0.0-core, Reading Mode e mudanças de modelo.

## 1.36. Bardin

Bardin não estava no núcleo da primeira versão.

Depois que o motor de busca já funcionava, a Análise de Conteúdo mudou a exigência do sistema:

```text
PDF → resposta
```

passou a ser pensado como:

```text
PDF
 ↓
segmentação
 ↓
recuperação
 ↓
trecho/contexto
 ↓
fonte
 ↓
interpretação do pesquisador
```

## 1.37. Ideia futura de unidades de registro

Possível evolução:

1. recuperar contexto;
2. apresentar texto;
3. pesquisador marcar unidade de registro;
4. manter unidade de contexto associada;
5. exportar estrutura;
6. filtrar por autor/ano/período;
7. recuperar contextos associados.

Isso é futuro, não funcionalidade atual.

## 1.38. NotebookLM

O usuário posteriormente percebeu que NotebookLM oferece uma experiência próxima da ideia inicial de conversar com PDFs/Drive.

Isso levantou a questão de diferenciação.

Não há base empírica suficiente para afirmar que NotebookLM seja menos reproduzível.

A diferenciação pretendida do Snoopy está sobretudo em necessidades de pesquisa metodológica:

- controle do corpus;
- rastreabilidade;
- recuperação explícita de trechos;
- preocupação com unidade de registro/contexto;
- filtros metodológicos;
- auditabilidade;
- possibilidade de estudar a própria recuperação.

## 1.39. Definição consolidada

> O Snoopy-RAG é uma ferramenta de apoio à pesquisa que utiliza recuperação semântica sobre um corpus documental para localizar evidências textuais relevantes, preservar sua origem e fornecer essas evidências ao pesquisador para posterior análise e interpretação.

## 1.40. O que não deve ser afirmado

O Snoopy não deve ser descrito como:

- substituto do pesquisador;
- implementação automática de Bardin;
- mecanismo que define sozinho unidades de registro;
- mecanismo que produz categorias metodologicamente válidas sem intervenção humana;
- garantia formal de reprodutibilidade na versão atual;
- reprodução visual perfeita dos PDFs;
- detector semântico perfeito de duplicatas;
- sistema de isolamento entre ambientes totalmente robusto;
- SaaS público finalizado.

## 1.41. Fragilidades

1. Reprodutibilidade ainda não formalmente garantida.
2. Decomposição da consulta depende de LLM.
3. Temperatura não é zero.
4. Ordenação/tie-breaking precisa ser testada.
5. Schema exportado não mostrou índice vetorial explícito.
6. Cache pode mascarar comportamento em testes.
7. Produção e desenvolvimento compartilham banco.
8. Jobs não possuem identificação explícita de ambiente.
9. Worker de desenvolvimento não é utilizado.
10. Backend utiliza service role e precisa garantir autorização própria.
11. Possível risco de IDOR em rotas com `folder_id`, a confirmar.
12. UUID da pasta pública está hardcoded.
13. API key do Picker sem restrição.
14. Configurações OAuth antigas permanecem.
15. Correção do cleaner ainda não está no `main`.
16. `requirements.txt` ainda contém `chromadb`, apesar de a busca atual não utilizá-lo.
17. Extração textual pode perder estrutura visual.
18. Chunking atual é heurístico.
19. Não existe ainda fluxo completo de marcação/exportação de unidades de registro.
20. Comando único para API + worker ainda não foi implementado.

Esses itens não significam que o sistema seja inutilizável. O estado atual é funcional, mas possui dívida técnica e pontos que precisam ser explicitados antes de alegações fortes de robustez metodológica.

## 1.42. Princípios

### Evidência antes da síntese

O trecho recuperado é mais importante do que a frase produzida pelo LLM.

### Fonte preservada

Toda evidência relevante deve permanecer ligada ao documento de origem.

### Chunk não é unidade metodológica

A segmentação técnica não deve ser confundida com a unidade definida pelo pesquisador.

### RAG não substitui metodologia

O sistema apoia exploração e recuperação; a análise científica continua sendo responsabilidade do pesquisador.

### Reprodutibilidade precisa ser testada

Não basta declarar determinismo; é necessário demonstrar sob quais condições ele ocorre.

### Acervo é parte do experimento

Corpus, versão do corpus e configuração de recuperação precisam ser considerados.

### Histórico não deve ser apagado

A evolução do projeto é parte de sua compreensão.

### Estado atual e futuro devem permanecer separados

Funcionalidades planejadas não podem ser descritas como existentes.

---

# 1.43. Mapa de evidências

A análise foi construída a partir de:

## A. Repositório GitHub

Código, estrutura, branches, commits, dependências, README e histórico.

## B. Schema exportado do Supabase

Tabelas, campos, FKs, RLS, políticas, RPC `match_chunks`, métrica, threshold, grants, realtime e constraints.

## C. Infraestrutura/configurações observadas

Produção MegaWare/Debian, desenvolvimento HP/Windows, Cloudflare Tunnel, PM2, domínios, Google OAuth, Picker, APIs relevantes e Supabase.

## D. Reconstrução histórica e decisões

Origem, V1→V2, SaaS, Bardin, rastreabilidade, reprodução e diferenciação em relação a ferramentas como NotebookLM.

---

# 1.44. Estado da Etapa 1

A Etapa 1 pode ser considerada **consolidada para fins de transição à Etapa 2**.

Isso não significa que nenhuma descoberta futura seja possível. Significa que já existe informação suficiente para passar da coleta/autópsia para uma reconstrução arquitetural profunda.

A Etapa 1 respondeu principalmente:

- o que existe;
- onde existe;
- como as partes principais funcionam;
- qual é o estado atual;
- quais são as principais fragilidades;
- como o sistema chegou até aqui.

---

# PRÓXIMA ETAPA

## ETAPA 2 — RECONSTRUÇÃO DA ARQUITETURA

A próxima etapa deve abandonar a lógica de “lista de arquivos” e reconstruir o sistema como um organismo único.

O caminho completo da consulta deverá ser acompanhado:

```text
pesquisador
   ↓
pergunta
   ↓
interface
   ↓
API
   ↓
decomposição da consulta
   ↓
embedding
   ↓
RPC / pgvector
   ↓
chunks recuperados
   ↓
deduplicação / seleção
   ↓
LLM
   ↓
resposta
   ↓
evidências
   ↓
documento original
   ↓
pesquisador
```

E o caminho de ingestão:

```text
Google Drive
   ↓
Picker
   ↓
sync
   ↓
jobs
   ↓
worker
   ↓
pipeline
   ↓
extração
   ↓
limpeza
   ↓
metadados
   ↓
chunking
   ↓
embeddings
   ↓
Supabase
```

O objetivo será responder não apenas “o que cada componente faz?”, mas:

> **Qual é o papel de cada componente dentro do sistema e quais propriedades metodológicas, técnicas e operacionais dependem dele?**

---

## Nota de preservação

Este arquivo é um **registro de análise**, não uma especificação final.

Caso seja reenviado em uma conversa futura, deve ser tratado como memória de trabalho da análise do Snoopy-RAG e utilizado para recuperar o estado conceitual alcançado nas Etapas 0 e 1 antes de avançar para etapas posteriores.
