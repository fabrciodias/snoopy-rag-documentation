# CONTEXTO DE TRANSFERÊNCIA — SNOOPY-RAG

## 1. O que é este chat

Este chat é dedicado à **análise, arquitetura, metodologia e documentação do Snoopy-RAG**.

**Não fazer alterações no código neste chat.**
**Não fazer writes no GitHub.**
GitHub pode ser consultado em modo somente leitura quando necessário para analisar o estado real do projeto.

A implementação será feita posteriormente em outro contexto, principalmente com Gemini. Este chat deve produzir:

* decisões arquiteturais;
* critérios;
* especificações;
* comparação de tecnologias;
* mapa de implementação;
* documentação;
* revisão crítica da implementação posteriormente.

O princípio é:

> **arquitetura primeiro → módulos → arquivos → implementação**

---

# 2. Estado atual do projeto

Projeto: **Snoopy-RAG**

Repositório principal:
`https://github.com/fabrciodias/snoopy-rag`

Repositório de documentação:
`https://github.com/fabrciodias/snoopy-rag-documentation`

O Snoopy-RAG é uma ferramenta de apoio à pesquisa que utiliza recuperação sobre um corpus documental para localizar evidências relevantes, preservar sua origem e fornecer essas evidências ao pesquisador para posterior análise e interpretação.

O sistema surgiu no GEPAFOR/UFMA a partir de um acervo de PDFs no Google Drive e evoluiu de uma ferramenta de busca em PDFs para uma infraestrutura de exploração documental com:

* recuperação semântica;
* evidências rastreáveis;
* contextualização;
* síntese baseada em evidências;
* possibilidade de Reading Mode;
* corpus controlado;
* preocupação explícita com reprodutibilidade, proveniência e preservação documental.

O orientador é o **Prof. Dr. Osmar Pedrochi Junior**.

---

# 3. Princípio metodológico central

O Snoopy **não substitui o pesquisador** e não realiza automaticamente a análise metodológica.

Especialmente em relação à Análise de Conteúdo/Bardin:

* chunk técnico ≠ unidade de registro;
* chunk técnico ≠ unidade de contexto;
* recuperação ≠ análise;
* similaridade ≠ relevância metodológica;
* evidência recuperada ≠ evidência metodologicamente interpretada;
* síntese por LLM ≠ fonte primária.

O sistema deve apoiar a exploração documental.

A cadeia conceitual é:

```text
pergunta
→ recuperação
→ evidências
→ contexto
→ fonte documental
→ pesquisador
→ interpretação/análise
```

---

# 4. Arquitetura conceitual consolidada

A arquitetura não deve ser pensada como:

```text
PDF → IA → resposta
```

mas como infraestrutura documental com dois fluxos:

```text
PIPELINE DOCUMENTAL

Drive
 ↓
sincronização
 ↓
documento
 ↓
representação canônica
 ↓
estrutura
 ↓
segmentação
 ↓
unidades de recuperação
 ↓
embeddings
 ↓
persistência
```

e:

```text
PIPELINE DE INVESTIGAÇÃO

pergunta
 ↓
restrições/filtros
 ↓
decomposição quando necessária
 ↓
recuperação
 ├── lexical
 ├── semântica
 └── multimodal quando pertinente
 ↓
candidatos
 ↓
deduplicação
 ↓
contextualização
 ↓
reranking
 ↓
seleção
 ↓
evidências
 ↓
síntese
 ↓
pesquisador
```

---

# 5. Etapas já consolidadas

## Etapa 5 — Requisitos metodológicos e arquiteturais

Foi estabelecido que o sistema deve possuir:

### Representação canônica

Preservar o documento antes de aplicar transformações destrutivas.

A representação canônica deve preservar, conforme disponível:

* identidade;
* páginas;
* ordem de leitura;
* blocos;
* hierarquia;
* parágrafos;
* listas;
* tabelas;
* células;
* captions;
* referências;
* notas;
* imagens;
* gráficos;
* equações;
* OCR;
* relações espaciais;
* proveniência.

A representação canônica **não é Markdown**, chunk, embedding ou saída do LLM.

---

### Segmentação

Modelo consolidado:

```text
elementos
 ↓
fragmentos estruturais
 ↓
unidades de perfil
 ↓
embedding/recuperação
```

O Snoopy deverá possuir **seu próprio motor de segmentação no sentido de governar a derivação das unidades**, podendo delegar algoritmos especializados a componentes externos.

Princípio:

> **Algoritmos externos propõem; o Snoopy governa.**

Existem:

* identidade documental;
* identidade da unidade de segmentação;
* identidade da representação/indexação.

Contexto não altera a identidade da unidade.

---

### Recuperação

A recuperação deve combinar, quando necessário:

```text
filtros estruturados
+
busca lexical
+
busca semântica
 ↓
fusão
 ↓
dedup
 ↓
contextualização
 ↓
reranking
 ↓
seleção
```

Restrições estruturadas, como autoria ou idioma, são distintas dos sinais de relevância textual.

Exemplo importante:

> “mostrar artigos de determinado autor”

é fundamentalmente uma consulta sobre **metadados estruturados**, e não simplesmente uma busca semântica.

Reranking é etapa separada da recuperação inicial.

Possível arquitetura:

```text
PostgreSQL + pgvector
+
PostgreSQL FTS
+
fusão
+
reranker local
```

Mas tecnologias concretas só devem ser escolhidas quando a investigação correspondente estiver consolidada.

---

### Proveniência

A cadeia deve poder ser reconstruída:

```text
pergunta
→ consulta derivada
→ recuperação
→ unidade
→ elemento/bloco
→ página
→ versão documental
→ documento
→ fonte original
```

Proveniência documental, execução e recuperação são dimensões distintas.

Versionamento e proveniência também são distintos:

* versionamento = qual estado;
* proveniência = como aquele estado/objeto surgiu.

---

### Persistência

Decisão já consolidada:

* Google Drive continua sendo a **fonte documental original**;
* PostgreSQL/Supabase governa o **estado lógico**;
* armazenamento local guarda artefatos físicos derivados que precisem persistir;
* PDFs originais não precisam ser duplicados permanentemente pelo Snoopy;
* artefatos temporários podem ser descartados;
* backup do armazenamento local é necessário;
* NAS/cloud externo só será considerado conforme necessidade futura.

---

### Segurança/operação

Decisões consolidadas:

* autenticação ≠ autorização ≠ isolamento;
* autorização deve ocorrer no backend;
* RLS é defesa adicional, não substituto da autorização;
* service role do Supabase ignora RLS;
* corpus/acervo é fronteira de segurança;
* jobs precisam de identidade própria;
* processamento deve ser idempotente;
* concorrência deve ser controlada;
* publicação deve ser atômica;
* falha parcial não pode publicar estado incompleto;
* prod/dev devem ser isolados;
* PGMQ/Supabase Queues + workers Python são suficientes inicialmente;
* não introduzir Celery/Temporal/Kubernetes/etc. sem necessidade;
* observabilidade deve ser proporcional ao estágio do projeto.

---

# 6. Etapa 6.1

A Etapa 6.1 traduziu os requisitos da Etapa 5 em **critérios técnicos**, sem ainda escolher tecnologias.

Existe um Registro Mestre próprio:

**“Registro Mestre da Análise — Snoopy-RAG — Etapa 6.1 — Tradução dos Requisitos em Critérios Técnicos”**

A regra fundamental:

> requisito → critério técnico → alternativas → escolha

Não:

> tecnologia encontrada → adaptar o Snoopy.

---

# 7. Etapa 6.2

A Etapa 6.2 investigou alternativas tecnológicas.

Foram consolidados os seguintes pontos:

### 6.2.1 — Representação/Processamento

Conclusão:

* não existe parser universal;
* arquitetura composta é mais adequada;
* Snoopy possui seu próprio modelo canônico;
* ferramentas externas fornecem observação/extração/interpretação;
* PDF original permanece como fonte;
* distinção entre original, reconhecido e interpretado;
* Docling, PyMuPDF, pdfplumber, MinerU, Adobe/Document AI e outros foram investigados.

### 6.2.2 — OCR

Conclusões:

* OCR é derivado;
* original nunca é substituído;
* documentos de baixa fidelidade não devem entrar automaticamente na recuperação;
* classificação de qualidade:

  * adequada → automática;
  * inadequada → bloqueada;
  * inconclusiva → revisão humana opcional;
* OCR localizado é preferível quando possível;
* Tesseract, Surya, PaddleOCR, Docling, MinerU e serviços externos foram investigados.

### 6.2.3 — Estrutura/Layout/Tabelas/Multimodal

Conclusões:

* preservar posição, ordem, hierarquia, tipo e relações separadamente;
* tabelas são elementos de primeira classe;
* geometria é evidência estrutural, não semântica;
* tabelas multipágina precisam manter identidade lógica;
* degradação deve ser graciosa;
* arquitetura composta continua preferível.

### 6.2.4 — Persistência

Decisão:

* Google Drive = fonte original;
* PostgreSQL/Supabase = estado lógico;
* armazenamento local = artefatos derivados persistentes;
* backup local;
* sem overengineering de escala nesta fase.

### 6.2.5 — Segmentação

Decisão:

* **Strategy D: motor de segmentação próprio do Snoopy**;
* múltiplos perfis;
* perfis podem ser híbridos;
* estrutura intermediária compartilhada;
* unidades referenciam elementos/fragmentos;
* contexto é separado da identidade da unidade;
* algoritmos externos podem ser usados como componentes.

### 6.2.6 — Recuperação

Decisões:

* filtros estruturados;
* lexical + semântica;
* fusão;
* dedup por identidade;
* contextualização;
* reranking;
* seleção;
* reranker local é alternativa prioritária;
* `BAAI/bge-reranker-v2-m3` apareceu como candidato;
* pgvector + PostgreSQL FTS é forte candidato;
* RRF é mecanismo interessante de fusão;
* não foi fechado um stack definitivo antes da arquitetura integrada.

### 6.2.7 — Proveniência/Versionamento

Decisões:

* identidade ≠ versão ≠ conteúdo;
* hash não é identidade lógica;
* provenance pode ser reconstruída por relações tipadas;
* entidade Derivation explícita não é obrigatória;
* modelo conceitual:

  * Entity
  * State
  * Activity/Run
  * Derivation
* PostgreSQL é candidato natural;
* W3C PROV e OpenLineage são referências conceituais, não necessariamente dependências;
* não adotar graph DB/event sourcing como núcleo sem necessidade.

### 6.2.8 — Segurança, Operação e Resiliência

Registro Mestre aprovado.

Decisões:

* identidade/autorização/isolamento separados;
* RBAC + ReBAC + ABAC apenas quando necessário;
* PGMQ/Supabase Queues + workers;
* job claim/lease;
* idempotência;
* retries controlados;
* publicação atômica;
* estados operacionais distinguíveis;
* limites de recursos;
* observabilidade proporcional;
* backup em camadas;
* restore precisa ser testado depois.

---

# 8. Regra atual de trabalho da Etapa 6

O objetivo original era investigar tudo exaustivamente antes de implementar.

Porém existe uma restrição temporal excepcional:

**Hoje é domingo, 20/09/2026, aproximadamente 00:07.**
A apresentação é na **terça-feira**.

Portanto foi criada uma trilha temporária:

# TRILHA ALPHA

Objetivo:

> **colocar uma V3 Alpha coerente e funcional em pé para a apresentação de terça-feira, sem abandonar a arquitetura maior.**

Não é a arquitetura definitiva.

É uma **planta da mansão**, não a mansão inteira.

A Alpha deve obedecer aos princípios já consolidados, mas várias capacidades avançadas ficam explicitamente pós-Alpha.

---

# 9. Fluxo Alpha

```text
A — Experiência + Corpus
 ↓
B — Interface + Segurança + Frontend
 ↓
rescan seletivo de Stage 5 + 6.1 + 6.2
 ↓
C — Arquitetura Integrada da Alpha
 ↓
D — Mapa de Implementação
 ↓
E — Validação rápida para terça
```

IMPORTANTE:

A implementação não será feita neste chat.

Este chat produz o **mapa de implementação**.

Gemini/outro contexto executará o código.

O mapa D recebe:

```text
Etapa 5
+
Etapa 6.1
+
pontos indispensáveis da Etapa 6.2
+
Alpha A
+
Alpha B
+
Alpha C
```

Não apenas A+B+C.

---

# 10. Alpha A — já encerrada

Alpha A = **Experiência de Investigação + Corpus**

Master Record aprovado.

### A.1 — Query → Results

* consulta original distinta das operações de recuperação;
* filtros estruturados;
* lexical/semantic retrieval;
* resultados identificáveis;
* resultado ≠ automaticamente evidência metodológica;
* síntese baseada nos resultados;
* evidências navegáveis.

### A.2 — Evidence → Context → Source

* unidade;
* contexto;
* fonte;
* identidade estável;
* mínimo: document_id + unit_id + localização + fonte original;
* contexto textual e estrutural;
* Reading Mode completo fica pós-Alpha.

### A.3 — Return to Investigation

Fluxo:

```text
PERGUNTA
→ CONSULTA
→ RECUPERAÇÃO
→ RESULTADOS
→ EVIDÊNCIA
→ CONTEXTO
→ FONTE/LEITURA
→ VOLTAR ou NOVA BUSCA
```

Abertura da fonte não pode destruir investigação atual.

### A.4 — Corpus/Processing

Separação fundamental:

```text
Corpus = universo documental
Document = existência/disponibilidade
Job = execução
```

Estados mínimos:

```text
PENDING
PROCESSING
ACTIVE
FAILED
REJECTED
REMOVED
```

Só `ACTIVE` entra na busca.

Drive continua sendo fonte original.

`drive_file_id` = identidade externa.
`document_id` = identidade interna.

Alpha não implementa versionamento completo.

---

# 11. Alpha B — situação atual

Estamos agora em:

# B — Interface, Segurança e Frontend

B condensa o essencial de 6.3.5 + 6.3.6.

Escopo final:

* superfícies/telas necessárias;
* continuidade da investigação;
* estados visíveis;
* loading, erro, cancelamento e reconexão;
* segurança de conteúdo e sanitização;
* acessibilidade essencial;
* arquitetura mínima do frontend;
* organização do estado;
* comunicação frontend ↔ backend;
* mecanismo de atualização em tempo real;
* roteamento/navegação;
* **renderer da resposta estruturada**;
* tecnologias necessárias.

Critério de fechamento:

> **A interface necessária para a Alpha pode ser implementada sem decisões arquiteturais fundamentais em aberto.**

---

# 12. Alpha B já concluída até B.5

## B.1 — Superfícies e fluxo

Superfícies conceituais:

1. **Investigação**
2. **Evidência**
3. **Fonte/Reading**
4. **Corpus/Acervo**

Investigação é a superfície principal.

Evidência é preferencialmente painel/drawer contextual.

Fonte/Reading é superfície navegável própria, mas preserva retorno à investigação.

Corpus é contexto transversal.

URL não é fonte de verdade do estado da investigação.

---

## B.2 — Estados visíveis

A interface deve distinguir:

* busca em andamento;
* processamento;
* concluído;
* falha;
* cancelamento quando aplicável;
* perda de comunicação;
* reconexão.

Operações concorrentes são independentes.

Não usar um `isLoading` global.

Perda de conexão ≠ falha da operação.

---

## B.3 — Segurança, conteúdo e renderer

Decisão importante:

**Markdown NÃO é a representação principal da resposta.**

A arquitetura será:

```text
StructuredResponse
        ↓
Snoopy Renderer
        ↓
HTML / componentes
        ↓
Interface
```

Markdown pode existir como representação derivada para:

* exportação;
* interoperabilidade;
* documentação;
* outros usos.

A resposta deve ser uma representação estruturada contendo elementos semânticos como:

* texto;
* hierarquia;
* evidências;
* referências;
* relações de navegação.

Evidência deve ser objeto semântico com identidade estável, não depender de hacks como `[TRECHO X]` para resolver a entidade.

Segurança:

* conteúdo documental é não confiável;
* não tratar conteúdo como HTML confiável;
* sanitização quando houver HTML/Markdown derivado;
* autorização no backend;
* erros não devem vazar segredos/internals.

---

## B.4.1 — Acessibilidade

* HTML semântico;
* interação nativa;
* foco gerenciado;
* drawer devolve foco ao trigger;
* Esc quando apropriado;
* estados não dependem somente de cor;
* não buscar certificação WCAG integral antes da Alpha;
* acessibilidade deve nascer nos componentes.

---

## B.4.2 — Resiliência

* waiting/processing/completed distintos;
* failure/cancel/disconnect distintos;
* reconexão revalida estado;
* preserva investigação;
* retry automático só onde houver idempotência;
* Alpha usa retry manual por padrão;
* SSE pode ser usado como canal de atualização;
* SSE não é fonte de verdade;
* sem offline-first/circuit breaker/etc. na Alpha.

---

## B.5.1 — Estado e composição

Quatro domínios:

```text
INVESTIGAÇÃO
CORPUS
OPERAÇÕES
UI
```

Investigação:

* query;
* results;
* evidence;
* active evidence;
* context;
* relação com source.

Corpus:

* corpus atual;
* documentos;
* disponibilidade;
* processamento;
* erros.

Operações:

* cada operação assíncrona tem identidade/estado próprio.

UI:

* drawer;
* foco;
* tabs;
* tema;
* preferências.

Importante:

```text
visualized ≠ selected ≠ retrieved ≠ evidence
```

Backend = autoridade sobre estado persistente.

Cache = reconstruível.

Arquitetura deve permitir futuramente múltiplas investigações, mas Alpha terá apenas uma ativa.

---

## B.5.2 — Comunicação Frontend ↔ Backend

Três mecanismos:

```text
A. request/response
B. operações assíncronas com operation_id
C. realtime updates
```

SSE é candidato natural para C porque já existe no V2.

SSE comunica mudança; não é source of truth.

Depois de reconexão:

```text
SSE reconecta
→ revalidação/refetch
→ estado atual
```

Objetos importantes possuem identidades estáveis.

Evidência deve ser recuperável diretamente por identidade.

Evitar resolver evidência por:

* título;
* pasta;
* texto;
* posição circunstancial.

---

# 13. B.6.1 — investigação atual: escolha do framework

Estamos investigando **React × Vue**.

Svelte foi descartado da disputa.

Não porque seja ruim, mas porque:

* SvelteKit adiciona uma complexidade que não pareceu justificar a vantagem;
* React e Vue se mostraram mais adequados ao contexto;
* a disputa real agora é React × Vue.

IMPORTANTE:

**Ainda NÃO existe vencedor definido.**

O usuário explicitamente NÃO quer que o assistente declare um vencedor prematuramente.

A investigação até aqui mostrou:

### React

Vantagens:

* ecossistema maior;
* liberdade arquitetural;
* enorme disponibilidade de bibliotecas;
* escape hatches claros;
* integração fácil com Web APIs/bibliotecas externas;
* combina muito com a filosofia “Snoopy governa”.

Riscos:

* mais decisões;
* maior superfície de composição;
* liberdade pode virar desorganização;
* arquitetura fica mais por nossa conta.

Metáfora do usuário:

> **React é uma arma: pode ser usada para se defender ou para se matar.**

### Vue

Vantagens:

* ecossistema mais integrado;
* Vue Router;
* Pinia;
* Composition API;
* boa integração com Vite/Vitest;
* menos decisões periféricas;
* mais convenções.

Riscos:

* pode orientar mais a arquitetura pelas convenções do ecossistema;
* possibilidade de “guarda-costas” limitar liberdade em certos cenários.

Metáfora do usuário:

> **Vue é um guarda-costas: pode te salvar, mas pode te deixar de mãos atadas.**

---

# 14. Resultado das rodadas do B.6.1

Foram comparados:

1. capacidades gerais;
2. crescimento do frontend;
3. ecossistema/manutenção;
4. extensibilidade/escape hatches;
5. arquitetura concreta.

A conclusão atual é:

> **React e Vue continuam tecnicamente empatados.**

React parece ligeiramente superior nas qualidades associadas à liberdade.

Mas também ligeiramente superior nos riscos associados à liberdade.

Vue parece ligeiramente superior nas qualidades associadas à integração/convenção.

Mas também carrega o risco correspondente de maior dependência das convenções.

Portanto:

> **0 × 0.**

O usuário percebeu que a investigação não está produzindo um vencedor, mas está produzindo informação relevante.

Não encerrar artificialmente.

---

# 15. Questão decisiva que surgiu

A próxima investigação deve responder:

> **Qual dos dois produz a arquitetura necessária do Snoopy com menos complexidade acidental, sem sacrificar capacidade?**

Isso deve ser analisado **concretamente**, não por lista abstrata de características.

A arquitetura a comparar é:

```text
Investigation
Corpus
Operations
Evidence
Reading
ResponseRenderer
HTTP
SSE
Routing
State
```

Precisamos projetar a mesma Alpha em:

```text
Snoopy + React
```

e:

```text
Snoopy + Vue
```

comparando:

* estrutura de módulos;
* responsabilidades;
* estado;
* fluxo de dados;
* renderer;
* comunicação;
* SSE;
* routing;
* componentes;
* dependências;
* complexidade acidental;
* capacidade preservada;
* manutenção;
* possibilidade de crescimento.

**Não declarar vencedor antes da comparação.**

---

# 16. Restrição temporal nova e muito importante

Estamos no domingo, 20/09/2026, aproximadamente 00:07.

A apresentação é terça-feira.

Isso muda o ritmo da investigação:

> **não precisamos abandonar a investigação, mas precisamos controlar seu escopo.**

O objetivo imediato é conseguir uma **V3 Alpha funcional e coerente para terça-feira**.

Depois da apresentação:

* retomaremos as discussões profundas;
* poderemos reconsiderar decisões;
* poderemos mudar framework;
* poderemos mudar linguagem;
* poderemos alterar tecnologias;
* poderemos aprofundar versionamento/proveniência/retrieval/etc.

O framework escolhido agora **não é uma prisão eterna**.

Ele será adotado para a Alpha/Beta inicialmente, mas a arquitetura deve manter fronteiras suficientes para permitir mudança futura caso a investigação completa mostre que outra tecnologia é mais adequada.

O usuário explicitamente disse:

> o framework escolhido agora pode mudar depois da investigação principal.

Portanto não otimizar a decisão apenas para “nunca mais mudar”.

Mas também não usar isso como desculpa para escolher de qualquer jeito.

---

# 17. Próximos passos obrigatórios

Depois de fechar a comparação React × Vue:

### B.6.1

Fechar escolha do framework para Alpha/Beta inicial, ou registrar empate + critério de decisão consciente se a investigação realmente não conseguir diferenciar.

### Depois:

**B fecha.**

Então fazer:

### RESCAN SELETIVO

Revisar:

* Etapa 5;
* Etapa 6.1;
* principalmente Etapa 6.2;

e extrair **somente o indispensável para a Alpha**.

Não reabrir toda a Etapa 6.2.

### C — Arquitetura Integrada da Alpha

Integrar:

```text
Etapa 5
+
6.1
+
pontos indispensáveis de 6.2
+
Alpha A
+
Alpha B
```

e produzir:

* componentes;
* módulos;
* responsabilidades;
* contratos;
* dados;
* fluxos;
* estados;
* tecnologias;
* dependências;
* fronteiras;
* falhas;
* comunicação;
* mapa técnico.

### D — Mapa de Implementação

Este chat produz o mapa.

Outro contexto/Gemini implementa.

### E — Validação rápida

Validar o mínimo necessário para a apresentação de terça.

---

# 18. Regras de trabalho deste chat

1. **Não fazer writes no GitHub.**
2. GitHub somente leitura quando necessário.
3. Não implementar código aqui.
4. Não gerar Master Record antes de a discussão estar consolidada.
5. Não inventar decisões para “terminar logo”.
6. Não reabrir decisões já fechadas sem nova evidência.
7. Se uma questão não for essencial para a Alpha, registrar como pós-Alpha.
8. Não transformar toda investigação em dez subetapas desnecessárias.
9. Qualidade > velocidade, mas agora com escopo controlado pelo prazo.
10. Quando uma tecnologia for investigada, comparar contra a **arquitetura real do Snoopy**, não contra uma lista genérica de features.
11. O usuário quer crítica real, não “passar pano”.
12. Não confundir popularidade com adequação arquitetural.
13. O sistema deve preservar a autonomia arquitetural do Snoopy.
14. O framework deve ser tratado como componente substituível em princípio.
15. Documentação declara capacidade; testes demonstram comportamento.

---

# 19. Fontes que serão enviadas junto deste contexto

O novo chat receberá os Registros Mestres e discussões já produzidos, especialmente:

* Etapa 5;
* Etapa 6.1;
* Discussões/Registros da 6.2.1–6.2.8;
* Alpha A;
* Discussão completa da Alpha B;
* demais documentos de análise relevantes.

**Os documentos enviados são a fonte principal.**

Este contexto serve para orientar a continuidade e preservar o estado da conversa.

Não substituir os documentos por memória geral.

---

# 20. Estado exato no momento da migração

**Etapa atual:**

```text
B — Interface, Segurança e Frontend
 └── B.6.1 — Framework frontend
       ├── Svelte → descartado
       └── React × Vue → em investigação
```

**Última conclusão:**

> React e Vue permanecem empatados tecnicamente, embora React esteja começando a formar uma justificativa arquitetural ligeiramente mais forte devido à liberdade e à compatibilidade com a arquitetura própria já definida para o Snoopy.

**Próxima tarefa imediata:**

> Comparar concretamente a arquitetura Alpha implementada conceitualmente com React versus Vue, buscando responder qual produz a arquitetura necessária com menor complexidade acidental sem sacrificar capacidade.

**Ainda não escolher React.**

**Ainda não escolher Vue.**

Depois dessa investigação, B.6.1 deve ser consolidada e B poderá ser encerrada.
