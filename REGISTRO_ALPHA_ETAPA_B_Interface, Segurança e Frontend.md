# Registro Mestre da Análise — Snoopy-RAG

## Trilha Alpha B — Interface, Segurança e Frontend

**Status:** Consolidado e aprovado para transição à etapa de extração dos requisitos indispensáveis e à Arquitetura Alpha Integrada
**Natureza:** especificação funcional, arquitetural e tecnológica mínima da interface da Alpha
**Escopo:** condensação do essencial das Etapas 6.3.5 e 6.3.6, complementada pelas decisões necessárias para a implementação da experiência de investigação da Alpha.

---

# 1. Finalidade da Trilha Alpha B

A Trilha B estabelece a estrutura mínima necessária para que a Alpha possua uma interface capaz de sustentar a experiência de investigação definida na Trilha A.

A interface deve permitir que o pesquisador percorra, de maneira contínua e compreensível, o caminho:

**investigação → evidência → fonte/leitura → retorno à investigação**

sem transformar a interface em uma camada responsável pela lógica documental, pela recuperação ou pela interpretação metodológica.

A Trilha B define:

* superfícies necessárias;
* continuidade da investigação;
* estados visíveis;
* comportamento diante de carregamento, erro e perda de comunicação;
* segurança de conteúdo;
* sanitização;
* acessibilidade essencial;
* organização mínima do frontend;
* estado da aplicação;
* comunicação com o backend;
* operações assíncronas;
* atualização em tempo real;
* streaming;
* roteamento;
* tecnologias necessárias para a implementação da Alpha.

A Trilha B não pretende estabelecer a arquitetura frontend definitiva do Snoopy. Seu objetivo é fornecer uma arquitetura mínima, coerente e implementável para a Alpha.

---

# 2. Princípio central da Trilha B

A interface deve servir à arquitetura do Snoopy, e não o contrário.

As fronteiras estabelecidas nas etapas anteriores permanecem independentes do framework frontend:

**domínio → infraestrutura → contratos → apresentação**

O framework constitui uma camada de implementação da interface.

Portanto:

> **Vue implementa a interface da Alpha; Vue não define a arquitetura do Snoopy.**

A escolha tecnológica da Alpha deve reduzir complexidade operacional sem transformar convenções do framework em decisões conceituais do sistema.

---

# 3. B.1 — Superfícies e fluxo de investigação

## 3.1 Organização geral

A Alpha será organizada em torno de uma superfície principal de **Investigação**.

As superfícies necessárias são:

1. **Investigação**
2. **Evidência**
3. **Reading/Fonte**
4. **Corpus/Acervo**

Além delas, existem superfícies auxiliares e transversais, como:

* autenticação;
* configurações;
* tema;
* histórico recente.

Essas superfícies auxiliares não constituem o núcleo da experiência de investigação.

---

## 3.2 Investigação

A superfície de Investigação constitui o centro da experiência.

Ela deve concentrar:

* consulta;
* resultados;
* síntese;
* estados da busca;
* acesso às evidências;
* relação entre resultados e evidências.

A Home não constitui uma arquitetura independente.

Na Alpha:

**Home = estado inicial da Investigação**

A existência de uma tela inicial visualmente distinta não deve criar um segundo modelo de domínio.

---

## 3.3 Evidência

A Evidência permanece como uma expansão contextual da investigação.

Ela será apresentada como:

* drawer;
* painel;
* ou mecanismo equivalente de expansão contextual.

A Evidência não constitui uma investigação independente.

Sua função é apresentar, no contexto da investigação atual:

* unidade recuperada;
* contexto textual;
* contexto estrutural disponível;
* identificação mínima;
* documento de origem;
* localização;
* acesso à fonte/Reading.

A abertura da evidência não deve destruir a consulta ou os resultados atuais.

---

## 3.4 Reading/Fonte

Reading/Fonte constitui uma superfície própria.

Sua função é permitir a exploração do documento de origem e preservar a relação:

**investigação → evidência → documento/fonte**

Ao abrir Reading/Fonte, a investigação de origem deve permanecer identificável.

O retorno deve ser possível sem reconstruir artificialmente a investigação.

A Alpha não exige ainda o Reading Mode definitivo com:

* reconstrução visual completa;
* navegação espacial avançada;
* reconstrução documental de alta fidelidade;
* anotações;
* comparação avançada.

---

## 3.5 Corpus/Acervo

Corpus/Acervo constitui uma superfície transversal.

Sua função é permitir visualizar e acompanhar o universo documental e seus estados essenciais, incluindo:

* documentos;
* disponibilidade;
* processamento;
* erros;
* sincronização mínima.

O Corpus não substitui o domínio documental nem deve concentrar lógica de processamento.

A superfície apenas apresenta e opera sobre estados cuja autoridade permanece no backend.

---

## 3.6 Relação entre superfícies

O fluxo mínimo é:

**INVESTIGAÇÃO → EVIDÊNCIA → READING/FONTE → RETORNO À INVESTIGAÇÃO**

O Corpus permanece transversal à experiência:

**CORPUS ↔ PROCESSAMENTO ↔ INVESTIGAÇÃO**

A navegação por URL pode representar e permitir a navegação entre superfícies, mas a URL não constitui a fonte de verdade do estado da investigação.

---

# 4. B.2 — Estados visíveis

## 4.1 Princípio

A interface deve tornar visível o estado real das operações sem exigir que o pesquisador compreenda a infraestrutura responsável por elas.

O sistema deve distinguir, no mínimo:

* aguardando/iniciando;
* processando;
* concluído;
* falhou;
* cancelado;
* comunicação perdida;
* reconectando.

---

## 4.2 Operações independentes

A Alpha não utilizará um estado global único como:

`isLoading`

Operações distintas devem possuir identidade e estado próprios.

Exemplos:

* busca #123 → PROCESSING;
* sincronização #456 → PROCESSING;
* processamento documental #789 → FAILED.

Uma operação em andamento não deve bloquear ou representar falsamente todas as demais atividades da interface.

---

## 4.3 Comunicação perdida ≠ operação falha

A interface deve distinguir:

**falha da operação**

de:

**incapacidade momentânea de confirmar o estado da operação.**

Quando a comunicação com o backend for perdida, o frontend não pode afirmar que a operação falhou apenas porque deixou de receber sua resposta.

A operação pode:

* ter continuado;
* ter falhado;
* não ter sido iniciada;
* ter sido concluída antes da perda de comunicação.

A interface deve representar essa incerteza de maneira apropriada.

---

# 5. B.3 — Segurança de conteúdo e representação estruturada

## 5.1 Conteúdo não confiável

Todo conteúdo proveniente de documentos, recuperação, fontes derivadas, modelos ou outras entradas externas deve ser tratado como **dados não confiáveis**.

Isso inclui:

* texto documental;
* resultados recuperados;
* evidências;
* conteúdo produzido por LLM;
* Markdown;
* HTML derivado;
* metadados provenientes de fontes externas.

Esse conteúdo não deve ser tratado automaticamente como HTML confiável ou como código executável.

---

## 5.2 Renderização

A Alpha não deve inserir arbitrariamente conteúdo recebido do backend diretamente no DOM.

O princípio é:

**dados estruturados → representação controlada → componentes/interface**

Quando Markdown for utilizado como representação derivada, o fluxo deverá ser:

**Markdown → parser → HTML/estrutura → sanitização → DOM**

HTML arbitrário proveniente do conteúdo não deve ser considerado confiável.

---

## 5.3 Autorização

A autorização pertence ao backend.

O frontend deve:

* refletir permissões;
* ocultar ou desabilitar ações quando apropriado;
* apresentar estados de acesso.

O frontend não constitui a fronteira de segurança.

O backend permanece responsável por determinar se determinada operação é autorizada.

A utilização de mecanismos como RLS não elimina a necessidade de autorização no backend, especialmente diante de credenciais privilegiadas.

---

## 5.4 Erros apresentados ao pesquisador

Mensagens de erro destinadas ao usuário não devem expor:

* stack traces;
* credenciais;
* segredos;
* caminhos internos;
* detalhes de infraestrutura;
* informações sensíveis;
* detalhes internos desnecessários.

O diagnóstico técnico deve permanecer separado da mensagem de erro apresentada na interface.

---

# 6. Representações distintas

A Alpha preserva três conceitos diferentes:

### 6.1 Representação documental canônica

É a representação responsável pela preservação estrutural do documento, definida nas etapas anteriores.

Ela pertence à arquitetura documental.

Não é uma representação específica da interface.

---

### 6.2 Representação estruturada da resposta

É a representação semântica da resposta de investigação.

Ela pode representar:

* texto;
* hierarquia;
* blocos;
* evidências;
* referências;
* tabelas;
* relações de navegação;
* outros elementos semânticos necessários à resposta.

Essa representação deve ser independente da apresentação visual.

---

### 6.3 Representação de apresentação

É a forma como a resposta estruturada é transformada em elementos da interface.

O fluxo principal da Alpha será:

**StructuredResponse → Snoopy Presentation Renderer → HTML/componentes → interface**

Markdown pode existir como representação derivada para:

* exportação;
* cópia;
* interoperabilidade;
* documentação;
* modos textuais.

Markdown não constitui requisito estrutural da interface.

---

# 7. Evidências como objetos semânticos

A interface não deve depender de marcadores textuais frágeis como:

`[TRECHO 1]`

A evidência deve ser representada como objeto semântico identificável.

O renderer deve receber informações equivalentes a:

* tipo de objeto;
* `evidence_id`;
* `unit_id`;
* relação com documento;
* localização;
* contexto;
* outras referências necessárias.

A identidade da evidência permanece independente da forma como ela é visualizada.

---

# 8. B.4 — Acessibilidade essencial

## 8.1 Princípio

Acessibilidade deve fazer parte da construção dos componentes da Alpha e não ser tratada como etapa posterior de correção.

O princípio básico é:

> **HTML semântico primeiro.**

---

## 8.2 Controles

Sempre que possível, a interface deve utilizar elementos nativos adequados:

* `button`;
* `input`;
* `textarea`;
* `dialog`;
* controles de formulário;
* elementos semânticos de estrutura.

Não devem ser utilizados elementos genéricos, como `div`, simulando controles quando elementos nativos forem adequados.

---

## 8.3 Evidência e navegação

Ao abrir um drawer ou painel de evidência:

* o foco deve ser direcionado adequadamente;
* o fechamento deve permitir retorno coerente do foco;
* `Esc` deve funcionar quando apropriado.

A passagem:

**Investigação → Evidência → Fonte**

deve preservar, tanto quanto tecnicamente possível, a origem funcional da navegação.

---

## 8.4 Estados visuais

Estados importantes não devem depender exclusivamente de cor.

Informações como:

* processamento;
* erro;
* sucesso;
* indisponibilidade;
* reconexão;

devem possuir outras formas de representação.

---

## 8.5 Limites da Alpha

Não fazem parte do escopo imediato:

* certificação WCAG completa;
* auditoria formal;
* testes exaustivos com leitores de tela;
* design system completo;
* internacionalização completa;
* avaliação formal de acessibilidade.

O critério da Alpha é garantir acessibilidade funcional e semântica razoável nos componentes críticos.

---

# 9. B.4 — Resiliência da interface

## 9.1 Princípio

A interface deve assumir que operações podem ser assíncronas e que a comunicação pode falhar.

O pesquisador não deve precisar administrar a infraestrutura, mas deve receber informação suficiente para compreender o estado da investigação.

---

## 9.2 Estados de operação

A interface deve distinguir:

**espera/início → processamento → conclusão**

e também:

**falha**

**cancelamento**

**perda de comunicação**

**reconexão**

Não devem ser utilizados percentuais de progresso fictícios.

---

## 9.3 Reconexão

Após uma perda de conexão, o comportamento preferencial é:

**reconectar → consultar novamente o estado atual → atualizar a interface**

A reconexão não deve destruir a investigação atual.

---

## 9.4 Retry

A Alpha não utilizará retry automático indiscriminado.

Retry automático somente é aceitável para operações claramente idempotentes e seguras.

Nos demais casos:

**retry manual**

é o comportamento padrão da Alpha.

Timeout também não deve ser interpretado automaticamente como prova de que a operação não foi executada.

---

## 9.5 SSE

O SSE permanece como mecanismo adequado para a Alpha porque já existe na V2.

Entretanto, SSE não constitui a fonte de verdade do sistema.

Sua função é comunicar:

* eventos;
* alterações;
* mudanças relevantes de estado.

Após perda da conexão SSE:

**reconectar → refetch/revalidação → estado atual**

O estado definitivo deve ser recuperável por requisição ao backend.

---

# 10. B.5 — Estado da aplicação

## 10.1 Domínios de estado

O frontend da Alpha deve separar quatro domínios principais:

### 10.1.1 INVESTIGATION

Contém o estado necessário para preservar a investigação atual:

* consulta;
* resultados;
* evidências recuperadas;
* evidência atualmente explorada;
* contexto;
* relação com a fonte;
* estado de retorno.

---

### 10.1.2 CORPUS

Representa a visão frontend do estado do corpus:

* corpus atual;
* documentos;
* disponibilidade;
* processamento;
* erros;
* informações mínimas de atualização.

O backend permanece autoridade sobre esse domínio.

---

### 10.1.3 OPERATIONS

Representa operações assíncronas identificáveis.

Cada operação deve possuir identidade e estado próprios.

Exemplos:

* busca;
* sincronização;
* processamento;
* reprocessamento.

---

### 10.1.4 UI

Representa estado puramente relacionado à interface:

* drawer aberto/fechado;
* foco;
* aba;
* menu;
* tema;
* preferências visuais;
* detalhes de apresentação.

---

## 10.2 Distinções fundamentais

A arquitetura frontend deve preservar:

**visualizado ≠ selecionado ≠ recuperado ≠ evidência**

Da mesma forma:

**estado de domínio ≠ cache ≠ estado persistente ≠ estado visual**

O cache não deve ser tratado como autoridade do domínio.

---

## 10.3 Investigação e superfícies

A Investigação deve sobreviver à mudança de superfície.

Reading recebe referências como:

* `investigation_id`;
* `evidence_id`;
* `document_id`;

e constrói sua própria apresentação.

Reading não deve duplicar arbitrariamente o estado da investigação.

---

## 10.4 Escopo da Alpha

A Alpha possui uma única investigação ativa.

Entretanto, essa limitação não deve ser transformada em uma arquitetura que impossibilite futuras múltiplas investigações.

A existência futura de múltiplas investigações deve permanecer conceitualmente possível.

---

# 11. B.5 — Comunicação com o backend

A comunicação da Alpha será organizada em três mecanismos complementares.

## 11.1 Request/Response

Utilizado para operações pontuais, como:

* obter estado;
* recuperar evidência;
* consultar documento;
* iniciar determinada operação;
* buscar estado atual.

---

## 11.2 Operações assíncronas

Operações potencialmente demoradas devem possuir identidade própria.

Uma ação assíncrona deve retornar algo equivalente a:

**operation_id**

em vez de representar a execução apenas como:

**“OK”**

O frontend pode então acompanhar o estado da operação independentemente das demais.

---

## 11.3 Atualizações em tempo real

SSE pode comunicar alterações relevantes ao frontend.

Entretanto:

> **SSE comunica mudanças; não substitui o estado persistente.**

O frontend deve ser capaz de recuperar o estado atual diretamente.

---

# 12. Fluxo de investigação e comunicação

O fluxo conceitual da Alpha é:

**consulta**

↓

**criação/identificação da investigação**

↓

**operação de recuperação**

↓

**resultados**

↓

**seleção/exploração**

↓

**evidência**

↓

**contexto**

↓

**documento/fonte**

↓

**retorno**

A investigação não deve depender da permanência de uma conexão SSE nem da manutenção de uma única requisição HTTP aberta.

---

# 13. Evidência recuperável por identidade

A evidência deve poder ser recuperada diretamente por sua identidade.

O backend deve ser capaz de resolver a cadeia:

**evidence_id → unit_id → document_id → localização → fonte**

A interface não deve precisar reconstruir essa relação apenas a partir do texto apresentado.

---

# 14. Streaming

## 14.1 Princípio

A Alpha não exige streaming estruturado de tokens ou blocos apenas para produzir uma experiência visual de resposta progressiva.

A resposta estruturada completa pode ser disponibilizada após a conclusão da operação.

---

## 14.2 Distinção entre streaming e atualização de estado

Não devem ser confundidos:

**streaming da resposta**

com:

**atualização em tempo real do estado da operação.**

O SSE da Alpha atende principalmente ao segundo caso.

---

## 14.3 Possibilidade futura

Streaming estruturado pode ser incorporado posteriormente caso exista requisito funcional real para:

* resposta incremental;
* blocos progressivos;
* síntese parcial;
* atualização incremental da interface.

Essa capacidade não é necessária para o fechamento da Alpha.

---

# 15. B.6 — Arquitetura mínima do frontend

## 15.1 Framework

O framework escolhido para a implementação da Alpha é:

**Vue 3 + TypeScript**

A escolha é operacional e não constitui decisão definitiva da arquitetura do Snoopy.

A arquitetura deve preservar as fronteiras entre:

* domínio;
* infraestrutura;
* contratos;
* apresentação.

Essas fronteiras não devem depender de Vue.

---

## 15.2 Ecossistema frontend mínimo

A trajetória tecnológica da Alpha será organizada em torno de:

**Vue 3 + TypeScript**

com:

* Composition API;
* Vue Router;
* mecanismo de estado compatível com os domínios definidos;
* infraestrutura própria para comunicação HTTP;
* infraestrutura própria para SSE;
* renderer estruturado;
* componentes de interface.

O uso de Pinia ou mecanismo equivalente deve ocorrer conforme a necessidade real do estado da aplicação, e não como requisito conceitual independente.

---

## 15.3 Organização arquitetural mínima

A estrutura conceitual do frontend é:

```text
SNOOPY
│
├── DOMAIN
│   ├── Investigation
│   ├── Corpus
│   ├── Evidence
│   └── Operations
│
├── INFRASTRUCTURE
│   ├── HTTP
│   └── SSE
│
├── RENDERER
│   └── StructuredResponse → Components
│
└── FRONTEND
    ├── Investigation
    ├── Evidence
    ├── Reading
    └── Corpus
```

Essa estrutura representa responsabilidades, não necessariamente uma árvore definitiva de arquivos.

---

# 16. Responsabilidade das camadas

## 16.1 Domain

Representa conceitos e regras próprias do Snoopy.

Não deve depender diretamente de componentes visuais.

---

## 16.2 Infrastructure

Concentra comunicação externa, incluindo:

* HTTP;
* SSE;
* mecanismos de persistência/cache quando necessários.

A infraestrutura não deve transformar respostas externas diretamente em estado visual.

---

## 16.3 Features

Organiza a experiência funcional da Alpha:

* Investigation;
* Evidence;
* Reading;
* Corpus.

Cada feature utiliza os contratos necessários sem absorver indiscriminadamente responsabilidades de outras camadas.

---

## 16.4 Renderer

Recebe a representação estruturada da resposta e transforma seus elementos semânticos em componentes de apresentação.

O renderer deve possuir um contrato controlado pelo Snoopy.

A implementação pode utilizar mecanismos próprios do Vue, mas a estrutura semântica da resposta não deve ser definida pelo framework.

---

# 17. Roteamento

O roteamento será utilizado para representar a navegação entre superfícies.

A Alpha poderá possuir rotas equivalentes a:

* investigação;
* evidência;
* leitura;
* corpus.

Entretanto:

> **URL ≠ fonte de verdade do estado da investigação.**

A rota representa localização/navegação.

O estado de domínio da investigação deve permanecer em sua própria estrutura.

A abertura de uma evidência ou fonte não deve depender exclusivamente da reconstrução da investigação a partir da URL.

---

# 18. Arquitetura substituível do frontend

A arquitetura da Alpha deve manter a seguinte relação:

```text
SNOOPY
│
├── DOMÍNIO
│
├── INFRAESTRUTURA
│
└── FRONTEND
    │
    └── Vue
        │
        └── Web Platform
```

A escolha de Vue não deve contaminar:

* identidade documental;
* representação canônica;
* segmentação;
* recuperação;
* evidência;
* proveniência;
* investigação;
* operações;
* contratos de backend.

Isso permite que a tecnologia frontend seja reavaliada posteriormente sem exigir a reconstrução dos fundamentos do Snoopy.

---

# 19. Decisão tecnológica sobre o framework

Foram consideradas alternativas de framework frontend, especialmente React, Vue e Svelte.

A investigação não identificou uma diferença funcional que tornasse uma dessas tecnologias obrigatoriamente superior para os requisitos conceituais da Alpha.

React e Vue atenderam aos requisitos fundamentais de:

* componentização;
* reatividade;
* TypeScript;
* separação entre estado e apresentação;
* composição de lógica;
* renderer estruturado;
* integração com APIs da plataforma web;
* crescimento da aplicação.

Svelte foi posteriormente descartado para a Alpha por não apresentar vantagem suficiente para justificar sua adoção neste contexto.

A decisão por Vue foi tomada por razão **operacional e de prazo**.

A trajetória integrada:

**Vue 3 → Composition API → composables → mecanismo de estado quando necessário → Vue Router**

reduz decisões periféricas necessárias para colocar a Alpha em funcionamento.

A decisão não deve ser interpretada como conclusão de que Vue é arquiteturalmente superior ao React para o Snoopy definitivo.

---

# 20. O que permanece sob responsabilidade do Snoopy

Independentemente do framework, o Snoopy permanece responsável por definir:

* modelo de investigação;
* identidade de evidência;
* contrato da resposta estruturada;
* relação evidência → fonte;
* estados de operação;
* contratos de comunicação;
* semântica do renderer;
* fronteiras de domínio;
* segurança;
* regras de autorização;
* comportamento de reconexão;
* semântica documental.

O framework apenas fornece mecanismos de implementação da interface.

---

# 21. Fora do escopo da Trilha B

Ficam deliberadamente postergados:

* arquitetura frontend definitiva;
* design system completo;
* certificação WCAG;
* auditoria formal de segurança;
* threat modeling completo;
* CSP sofisticada;
* hardening completo de infraestrutura;
* gestão avançada de sessões;
* políticas avançadas de retenção;
* pentest;
* compartilhamento sofisticado;
* múltiplas investigações persistentes;
* workspace completo;
* colaboração;
* offline-first;
* service workers;
* fila offline no navegador;
* sincronização offline;
* retry frontend sofisticado;
* WebSockets quando SSE for suficiente;
* streaming estruturado avançado;
* GraphQL;
* event sourcing;
* CQRS;
* microserviços;
* API gateway;
* filas frontend complexas;
* sincronização bidirecional complexa;
* observabilidade frontend avançada;
* design system definitivo;
* internacionalização completa.

Esses itens não constituem lacunas acidentais da Alpha.

São questões conscientemente postergadas para etapas posteriores.

---

# 22. Princípios consolidados da Trilha B

A Alpha deverá preservar as seguintes distinções:

**Home ≠ arquitetura de investigação independente**

**Investigação ≠ Evidência**

**Evidência ≠ Fonte**

**URL ≠ estado da investigação**

**estado de domínio ≠ cache**

**estado persistente ≠ estado visual**

**visualizado ≠ selecionado ≠ recuperado ≠ evidência**

**comunicação perdida ≠ operação falhou**

**timeout ≠ operação não executada**

**SSE ≠ fonte de verdade**

**SSE ≠ streaming de resposta**

**streaming ≠ requisito da Alpha**

**autorização ≠ interface**

**conteúdo documental ≠ HTML confiável**

**Markdown ≠ representação estrutural da resposta**

**representação documental ≠ representação da resposta**

**representação da resposta ≠ apresentação visual**

**evidência semântica ≠ marcador textual**

**framework frontend ≠ arquitetura do Snoopy**

**Vue ≠ domínio**

**renderer ≠ conteúdo**

Essas distinções fazem parte da coerência arquitetural da Alpha.

---

# 23. Critério de implementação da Trilha B

A Trilha B fornece uma arquitetura frontend suficientemente definida quando for possível implementar a Alpha sem necessidade de decidir novamente:

* quais superfícies existem;
* como a investigação atravessa as superfícies;
* quais estados precisam ser representados;
* como operações assíncronas são identificadas;
* como erros e perda de comunicação são tratados;
* qual é a fronteira de segurança;
* como conteúdo é sanitizado e renderizado;
* como evidências são representadas;
* como o estado da aplicação é separado;
* como o frontend se comunica com o backend;
* qual é o papel do SSE;
* qual é o papel do streaming;
* como o roteamento se relaciona à investigação;
* qual framework será utilizado na Alpha;
* quais responsabilidades pertencem ao frontend e quais permanecem no domínio/infraestrutura.

As decisões de baixo nível — bibliotecas específicas, nomes definitivos de arquivos, componentes individuais, implementação detalhada dos stores/composables, cliente HTTP específico, detalhes finais do renderer e demais escolhas de implementação — pertencem à **Arquitetura Alpha Integrada (Trilha C)**.

---

# 24. Critério de encerramento da Trilha B

A Trilha B é considerada concluída quando o Snoopy Alpha possuir uma definição suficientemente precisa para que sua interface possa ser implementada de maneira coerente:

**Investigação → Evidência → Reading/Fonte → Retorno**

com:

**estados visíveis → operações identificáveis → comunicação recuperável → conteúdo seguro → renderer estruturado → estado separado → roteamento → frontend Vue + TypeScript**

sem que permaneçam decisões arquiteturais fundamentais abertas dentro do escopo da interface.

A partir deste ponto, a discussão não deve retornar à investigação conceitual de frameworks ou reabrir decisões já consolidadas sem evidência nova.

As decisões restantes devem ser incorporadas à **Arquitetura Alpha Integrada**, juntamente com os requisitos indispensáveis recuperados das Etapas 5, 6.1 e 6.2.

---

# 25. Princípio central

> **A interface da Alpha deve permitir que o pesquisador percorra a investigação, explore evidências, alcance suas fontes e retorne sem perder o contexto, enquanto o frontend mantém separadas apresentação, estado, comunicação e domínio, tratando todo conteúdo externo como dado não confiável e mantendo o framework subordinado aos contratos e às fronteiras arquiteturais do Snoopy.**

A Trilha B está, portanto, **consolidada e encerrada**, ficando pronta para a etapa de **extração dos requisitos indispensáveis das Etapas 5, 6.1 e 6.2** e posterior construção da **Trilha C — Arquitetura Alpha Integrada**.
