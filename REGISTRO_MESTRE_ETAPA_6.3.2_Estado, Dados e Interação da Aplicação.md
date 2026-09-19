# Registro Mestre da Análise — Snoopy-RAG

## Etapa 6.3.2 — Estado, Dados e Interação da Aplicação

**Status:** Consolidado e aprovado para transição à Etapa 6.3.3
**Natureza:** Especificação da experiência e arquitetura conceitual de frontend — não constitui implementação nem escolha definitiva de tecnologia.

---

## 1. Objetivo da etapa

A Etapa 6.3.2 investigou como a interface do Snoopy-RAG V3 deverá representar, preservar, modificar e sincronizar os diferentes estados envolvidos na aplicação.

A investigação partiu da situação observada no V2, na qual informações de naturezas distintas encontram-se distribuídas entre estado global, variáveis locais, DOM, callbacks, URL, operações assíncronas e respostas do backend.

O objetivo não foi substituir o `appState` por outra implementação equivalente nem escolher antecipadamente um framework, biblioteca de gerenciamento de estado ou tecnologia específica.

O objetivo foi estabelecer **qual modelo de estado e interação a arquitetura de frontend deverá sustentar**.

---

# 2. Problema central

O Snoopy V3 terá uma quantidade significativamente maior de estados e relações do que a aplicação V2.

Entre eles estão:

* corpus;
* documentos;
* versões documentais;
* unidades de recuperação;
* consultas;
* resultados;
* evidências;
* contextos de leitura;
* investigações;
* jobs;
* permissões;
* configurações;
* estados de processamento;
* estados de conexão;
* estados puramente interacionais da interface.

Esses elementos não possuem a mesma natureza, o mesmo ciclo de vida nem a mesma necessidade de persistência.

Portanto, tratá-los como uma única coleção global de variáveis produziria uma arquitetura difícil de compreender, sincronizar e manter.

O problema fundamental pode ser formulado como:

> **Como a interface pode representar o estado complexo do Snoopy sem depender de uma coleção frágil de estados globais e manipulações imperativas?**

---

# 3. Princípio fundamental

O estado do Snoopy V3 deverá ser tratado como um conjunto de estados de naturezas diferentes — domínio, investigação, operação, aplicação e interface — relacionados entre si, mas não colapsados em uma única estrutura global.

O cliente deverá representar o estado governado pelo sistema sem se tornar sua autoridade, enquanto estados puramente interacionais poderão permanecer locais.

Ações, eventos e estados também deverão permanecer conceitualmente distintos.

---

# 4. Taxonomia conceitual do estado

A investigação estabeleceu uma distinção fundamental entre diferentes categorias de estado.

## 4.1 Estado persistente de domínio

Representa o estado governado pelo próprio Snoopy.

Inclui, entre outros:

* corpus;
* documentos;
* versões;
* unidades de recuperação;
* permissões;
* configurações persistentes;
* relações de proveniência.

Esse estado possui significado próprio para o sistema e não deve depender da existência de uma determinada página aberta no navegador.

---

## 4.2 Estado investigativo

Representa aquilo que possui significado para a experiência de pesquisa.

Pode incluir:

* consultas;
* resultados;
* evidências;
* configurações relevantes da recuperação;
* relações com documentos;
* contextos documentais explorados;
* provenance necessária à reconstrução da consulta.

A persistência desse estado não significa registrar cada movimento realizado na interface.

A distinção fundamental é:

> **visualizar não equivale a selecionar; abrir não equivale a considerar relevante; recuperar não equivale a aceitar como evidência.**

O sistema não deverá transformar automaticamente comportamento de interface em decisão metodológica.

---

## 4.3 Estado de operação

Representa operações em execução ou concluídas.

Exemplos:

* sincronização;
* processamento documental;
* OCR;
* geração de embeddings;
* busca;
* reranking;
* outras operações assíncronas.

Operações deverão possuir identidade e ciclo de vida próprios.

O Snoopy não deverá pressupor uma única operação global em andamento.

---

## 4.4 Estado da aplicação

Representa estados necessários para coordenar a experiência atual do cliente, sem constituírem necessariamente estado documental persistente.

Pode incluir:

* consulta atualmente ativa;
* superfície atualmente exibida;
* investigação atualmente carregada;
* estado de atualização;
* conexão com o servidor;
* estados derivados necessários à coordenação da aplicação.

---

## 4.5 Estado local da interface

Representa estados exclusivamente relacionados à interação visual.

Exemplos:

* modal aberto;
* painel expandido;
* aba selecionada;
* elemento visualmente selecionado;
* foco;
* estado de navegação transitório;
* animações;
* outros estados que não possuam significado próprio fora da interface.

Esses estados não precisam ser persistidos pelo Snoopy.

---

# 5. Persistência seletiva

O Snoopy V3 não deverá persistir indiscriminadamente todo o comportamento da interface.

Deverão ser persistidos os objetos que possuem significado próprio para o sistema ou para a investigação.

Deverão permanecer locais os estados necessários apenas para representar e operar a interface.

A regra consolidada é:

> **Persistir objetos que possuem significado próprio para o Snoopy; manter localmente apenas o estado necessário para representar e operar a interface; utilizar cache somente como otimização reconstruível.**

---

# 6. Persistência não é cache

A investigação estabeleceu uma distinção explícita entre persistência e cache.

**Persistência** significa que o estado possui significado próprio e precisa poder ser reconstruído.

**Cache** significa que determinada informação pode ser reutilizada para evitar uma nova operação, desde que sua validade possa ser verificada.

Consequentemente:

* cache não é fonte de verdade;
* cache pode ser descartado;
* cache pode ser invalidado;
* cache não deve substituir o estado persistente;
* o sistema deverá ser capaz de reconstruir a representação a partir da autoridade correspondente.

---

# 7. Consulta como objeto recuperável da experiência de pesquisa

A Etapa 6.3.1 estabeleceu que a continuidade da experiência do Snoopy não deverá ser baseada em uma conversa obrigatória.

Cada consulta deverá possuir identidade e histórico recuperáveis, permitindo ao pesquisador retornar a:

* consulta;
* resultados;
* evidências;
* documentos;
* contextos relevantes;
* configurações investigativas necessárias à compreensão do resultado.

A continuidade preservada pelo sistema deverá ser:

> **continuidade da exploração, e não continuidade obrigatória de conversa.**

Assim, o Snoopy poderá oferecer uma experiência semelhante à continuidade encontrada em interfaces conversacionais sem assumir conceitualmente a identidade de um chatbot.

---

# 8. Backend como autoridade do estado persistente

O estado persistente do Snoopy deverá possuir uma direção clara de autoridade.

O backend e o modelo persistente do Snoopy serão responsáveis por governar:

* documentos;
* versões;
* corpus;
* jobs;
* investigações persistidas;
* permissões;
* estados de processamento;
* demais objetos persistentes.

O frontend deverá manter uma representação local desses estados.

O frontend não deverá se tornar uma segunda fonte independente de verdade.

A relação fundamental será:

```text
estado persistente do Snoopy
          ↓
representação no cliente
          ↓
interface
```

e não:

```text
DOM
 ↓
variáveis locais
 ↓
callbacks
 ↓
tentativa de reconstruir o estado do sistema
```

---

# 9. Ações, eventos e estados

A investigação estabeleceu uma distinção explícita entre três conceitos.

### Ação

Representa uma solicitação para que algo aconteça.

Exemplos:

* pesquisar;
* iniciar sincronização;
* cancelar processamento;
* abrir documento;
* modificar configuração.

### Evento

Representa uma ocorrência ou mudança observável que pode comunicar ou provocar uma transição de estado.

Exemplos:

* busca concluída;
* job iniciado;
* job concluído;
* documento atualizado;
* conexão restabelecida.

### Estado

Representa a situação atual de uma entidade, operação ou contexto.

Exemplos:

* processing;
* completed;
* failed;
* cancelled;
* active;
* obsolete.

A relação conceitual é:

```text
ação
  ↓
operação / mudança
  ↓
evento
  ↓
novo estado
  ↓
representações dependentes
  ↓
interface
```

Essa distinção é conceitual e arquitetural. Ela não implica adoção de event sourcing como modelo de persistência.

---

# 10. Confirmação das ações

A interface poderá representar imediatamente uma **intenção** do usuário, mas não deverá representar como fato confirmado uma alteração persistente que ainda não tenha sido aceita pelo sistema.

Para operações persistentes ou relevantes:

```text
ação do usuário
      ↓
solicitação
      ↓
backend aceita
      ↓
operação identificável
      ↓
estado atualizado
      ↓
interface
```

Caso a operação seja rejeitada, a interface deverá refletir a rejeição.

Isso distingue:

* intenção;
* solicitação;
* operação aceita;
* operação em andamento;
* resultado confirmado.

---

# 11. Operações concorrentes

O Snoopy V3 não deverá utilizar um modelo em que exista uma única “operação atual” ou um único estado global de carregamento.

Operações independentes deverão possuir identidade e ciclo de vida próprios.

Exemplo conceitual:

```text
Consulta A        → completed
Consulta B        → processing
Sincronização     → processing
Documento X       → active
Documento Y       → processing
```

A interface deverá ser capaz de representar essas situações simultaneamente.

Isso é especialmente relevante para a evolução do V3 em direção a múltiplas superfícies e consultas independentes.

---

# 12. Cancelamento, falha e perda de comunicação

Esses estados deverão permanecer conceitualmente distintos.

### Cancelamento

Solicitado pelo usuário ou por uma política explícita do sistema.

```text
processing → cancelled
```

### Falha

A operação não conseguiu completar devido a uma condição de erro.

```text
processing → failed
```

### Perda de comunicação

O cliente deixa de conhecer temporariamente o estado atual do servidor.

Isso não autoriza o frontend a concluir que a operação falhou.

```text
processing
     ↓
conexão perdida
     ↓
estado local desconhecido
     ↓
reconexão
     ↓
revalidação
     ↓
processing / completed / failed / cancelled
```

Portanto:

> **A perda de conexão não deverá ser interpretada automaticamente como falha da operação nem como perda do estado persistente.**

---

# 13. Sincronização entre servidor e cliente

O estado do servidor poderá mudar independentemente da ação imediata do usuário.

Isso ocorre especialmente porque:

* o Google Drive é uma fonte externa;
* workers processam documentos;
* sincronizações podem ocorrer automaticamente;
* outros usuários poderão futuramente interagir com recursos compartilhados;
* operações assíncronas continuam após a interação inicial.

Portanto, a interface não poderá depender exclusivamente da ação do usuário para descobrir alterações.

A estratégia de atualização deverá ser composta, combinando conforme a natureza do estado:

* recuperação inicial ao entrar em uma superfície;
* invalidação;
* refetch;
* eventos/realtime;
* polling periódico;
* mecanismos de fallback.

Não deverá existir uma estratégia única obrigatória para todo o estado do sistema.

---

# 14. Atualização de documentos e preservação do contexto

Alterações no corpus não deverão destruir silenciosamente o contexto atualmente utilizado pelo pesquisador.

Se uma nova versão documental for publicada enquanto o pesquisador estiver trabalhando com uma versão anterior, a interface deverá poder distinguir:

```text
versão atualmente utilizada
        ≠
versão mais recente disponível
```

Uma investigação histórica também não deverá ser silenciosamente reconstruída apenas porque o corpus mudou posteriormente.

A atualização do corpus e a continuidade da investigação são estados relacionados, mas não equivalentes.

---

# 15. URL e navegação

A URL deverá funcionar como mecanismo de:

* identificação;
* navegação;
* recuperação de superfícies;
* compartilhamento de determinados estados persistentes quando apropriado.

Entretanto:

> **A URL não será uma fonte independente de verdade sobre o estado do Snoopy.**

Ela deverá representar estados que possam ser reconstruídos a partir das fontes de autoridade correspondentes.

---

# 16. Reatividade

A reatividade deverá ser tratada como **propriedade arquitetural**, e não como sinônimo de uma tecnologia específica.

Quando um estado relevante mudar, as partes da interface que dependem desse estado deverão poder atualizar-se sem que o código precise reconstruir manualmente a página inteira.

Isso será particularmente importante para:

* jobs;
* sincronizações;
* resultados;
* investigações;
* versões documentais;
* permissões;
* estados de conexão;
* erros e recuperação.

Diferentes estados poderão utilizar mecanismos de atualização diferentes.

---

# 17. Atualizações otimistas

Atualizações otimistas poderão ser utilizadas para interações locais, reversíveis e cujo resultado não dependa de confirmação do backend.

Exemplos:

* abrir/fechar painel;
* trocar aba;
* expandir evidência;
* seleção visual;
* preferências locais.

Para alterações persistentes ou governadas pelo sistema, a interface não deverá representar como confirmado um estado que ainda não tenha sido aceito pelo backend.

Exemplos:

* iniciar processamento;
* cancelar job;
* remover documento;
* publicar versão;
* alterar configuração persistente;
* alterar permissões.

---

# 18. Reconexão e recuperação

A perda de conexão não deverá exigir a reconstrução indiscriminada da interface.

Quando a comunicação for restabelecida, o cliente deverá:

1. identificar estados potencialmente desatualizados;
2. invalidar representações cuja validade não possa ser garantida;
3. recuperar o estado atual do servidor;
4. atualizar as representações dependentes;
5. preservar estados locais e investigativos que continuem válidos.

Consequentemente:

> **Perda de comunicação não significa perda de estado.**

A recuperação deverá ser baseada em **revalidação e sincronização**, e não em `window.location.reload()` como mecanismo geral de recuperação.

---

# 19. Consequências arquiteturais

As decisões desta etapa produzem consequências diretas para a arquitetura de frontend:

1. O frontend não deverá depender de um único objeto global contendo estados heterogêneos.
2. O DOM não deverá funcionar como fonte de verdade do estado da aplicação.
3. Estados de domínio, investigação, operação, aplicação e interface deverão possuir fronteiras reconhecíveis.
4. Operações assíncronas deverão possuir identidade e ciclo de vida próprios.
5. O cliente deverá representar o estado do servidor sem substituí-lo como autoridade.
6. A aplicação deverá suportar múltiplas superfícies e estados simultâneos.
7. A atualização da interface deverá ser reativa.
8. O mecanismo de sincronização deverá variar conforme a natureza do estado.
9. O sistema deverá preservar contexto histórico e investigativo diante de mudanças externas.
10. A recuperação de conexão deverá ocorrer por revalidação, não por reconstrução indiscriminada da aplicação.
11. O modelo deverá permitir persistência seletiva.
12. Cache deverá permanecer reconstruível e subordinado às fontes de verdade.
13. Ações, eventos e estados deverão ser representáveis separadamente.
14. A arquitetura deverá permitir que decisões tecnológicas sejam tomadas posteriormente sem alterar esses princípios.

---

# 20. Decisões consolidadas

### DEC-632-01 — Estados heterogêneos

O Snoopy V3 não utilizará um modelo conceitual de estado global único.

### DEC-632-02 — Autoridade do backend

O estado persistente do sistema será governado pelo backend e pela camada persistente do Snoopy.

### DEC-632-03 — Representação no cliente

O frontend manterá representações locais do estado do sistema combinadas com estados próprios da aplicação e da interface.

### DEC-632-04 — Persistência seletiva

Somente estados com significado próprio para o sistema ou investigação deverão ser persistidos.

### DEC-632-05 — Cache

Cache será tratado como otimização reconstruível e nunca como fonte de verdade.

### DEC-632-06 — Consulta recuperável

Cada consulta relevante poderá possuir identidade e histórico recuperáveis, preservando continuidade da exploração sem exigir uma conversa obrigatória.

### DEC-632-07 — Sincronização composta

A atualização do estado utilizará mecanismos diferentes conforme a natureza do estado, combinando recuperação inicial, invalidação/refetch, eventos e polling/fallback quando apropriado.

### DEC-632-08 — Operações independentes

Operações assíncronas possuirão identidade e ciclo de vida próprios.

### DEC-632-09 — Ações, eventos e estados

Ações, eventos e estados serão tratados como conceitos distintos.

### DEC-632-10 — Confirmação

Alterações persistentes somente serão consideradas confirmadas após aceitação ou atualização proveniente da autoridade do sistema.

### DEC-632-11 — Cancelamento e falha

Cancelamento, falha e perda de comunicação serão estados distintos.

### DEC-632-12 — Reatividade

A interface deverá possuir comportamento reativo, sem depender de manipulação imperativa generalizada do DOM ou de recarregamentos completos.

### DEC-632-13 — Reconexão

A recuperação de comunicação ocorrerá por revalidação e sincronização, preservando estados ainda válidos.

### DEC-632-14 — Atualização otimista

Atualizações otimistas serão restritas principalmente a estados locais e reversíveis.

---

# 21. Decisões deliberadamente não tomadas

A Etapa 6.3.2 não determina:

* React, Vue, Svelte ou outro framework;
* Redux, Zustand, MobX, Signals ou outra solução de estado;
* biblioteca específica de realtime;
* implementação específica de SSE;
* router;
* mecanismo concreto de cache;
* IndexedDB, localStorage ou outra tecnologia de persistência local;
* biblioteca de optimistic UI;
* arquitetura definitiva de componentes.

Essas decisões pertencem às etapas posteriores de arquitetura e investigação tecnológica.

---

# 22. Limites epistemológicos

O modelo de estado da interface não deverá ser confundido com o modelo metodológico da pesquisa.

A interface poderá representar:

* consulta;
* resultados;
* evidências;
* documentos;
* contexto;
* histórico;
* operações.

Isso não significa que a interface possa determinar:

* relevância metodológica;
* unidade de registro;
* unidade de contexto;
* interpretação;
* categoria analítica;
* conclusão científica.

O estado computacional permanece distinto da decisão metodológica do pesquisador.

---

# 23. Relação com as etapas anteriores e posteriores

A Etapa 6.3.2 depende diretamente dos requisitos definidos nas Etapas 5 e dos critérios técnicos da Etapa 6.1.

Também incorpora as capacidades investigadas na Etapa 6.2, especialmente:

* persistência;
* segmentação;
* recuperação;
* proveniência;
* versionamento;
* segurança;
* operação;
* processamento assíncrono.

A Etapa 6.3.2, por sua vez, fornece as bases conceituais para:

* **6.3.3 — Busca, Evidência e Leitura**;
* **6.3.4 — Corpus, Documentos e Processamento**;
* **6.3.5 — Segurança, Acessibilidade e Resiliência da Interface**;
* **6.3.6 — Arquitetura de Frontend e Tecnologias**;
* **6.4 — Projeto da Arquitetura Técnica Integrada**.

---

# 24. Princípios consolidados

A Etapa 6.3.2 estabelece os seguintes princípios:

1. **O frontend representa o estado; não o governa.**
2. **Estados de naturezas diferentes não devem ser artificialmente unificados.**
3. **Persistência deve seguir significado, não conveniência técnica.**
4. **Cache não é estado de autoridade.**
5. **Continuidade de exploração não exige continuidade de conversa.**
6. **Operações assíncronas possuem ciclo de vida próprio.**
7. **Ações, eventos e estados são conceitos distintos.**
8. **O servidor pode mudar independentemente da interface.**
9. **Sincronização deve ser composta e adequada à natureza de cada estado.**
10. **Mudança externa não deve destruir silenciosamente o contexto do pesquisador.**
11. **Perda de conexão não significa perda de estado.**
12. **Reatividade é uma propriedade arquitetural.**
13. **A interface deve refletir estados confirmados sem impedir interações locais responsivas.**
14. **O modelo de estado da interface não determina a metodologia da pesquisa.**

---

# 25. Formulação consolidada

> **O Snoopy V3 deverá representar o estado da aplicação por meio de categorias distintas de estado — domínio, investigação, operação, aplicação e interface — mantendo entre elas relações explícitas sem reduzi-las a uma única estrutura global. O estado persistente do sistema será governado pelo backend e pelo modelo persistente do Snoopy, enquanto o frontend manterá representações locais desse estado combinadas com estados próprios da aplicação e da interface. Deverão ser persistidos os objetos que possuem significado próprio para o sistema ou para a investigação, enquanto estados puramente interacionais permanecerão locais e transitórios; caches serão tratados como representações reconstruíveis e não como fontes de verdade.**
>
> **A interação deverá distinguir ações, eventos e estados, permitindo que operações assíncronas possuam identidade e ciclo de vida próprios. Alterações persistentes deverão ser consideradas confirmadas somente após aceitação ou atualização proveniente da autoridade do sistema, enquanto interações puramente locais poderão utilizar atualizações imediatas. Operações concorrentes deverão permanecer independentes e identificáveis, e cancelamento, falha e perda de comunicação deverão constituir situações distintas. A perda de conexão não deverá ser interpretada como falha da operação nem como perda do estado persistente; após a reconexão, o cliente deverá revalidar e sincronizar os estados potencialmente desatualizados.**
>
> **A sincronização entre servidor e cliente deverá utilizar mecanismos compostos, combinando recuperação inicial, invalidação, refetch, eventos e polling ou mecanismos de fallback conforme a natureza do estado. A interface deverá possuir comportamento reativo, refletindo mudanças relevantes de estado sem depender de manipulação imperativa generalizada do DOM ou de recarregamentos completos da aplicação. A continuidade preservada pelo sistema deverá ser continuidade da exploração, permitindo recuperar consultas, resultados, evidências, documentos e contextos relevantes sem transformar toda a navegação da interface em histórico persistente.**
>
> **Esse modelo deverá sustentar a evolução do Snoopy V3 para múltiplas superfícies de interação, mantendo separadas a representação computacional do estado e as decisões metodológicas do pesquisador. A escolha das tecnologias específicas de frontend deverá ocorrer posteriormente, de acordo com sua capacidade de materializar esse modelo sem alterar seus princípios fundamentais.**

---

# 26. Princípio central da etapa

> **O Snoopy V3 deverá tratar o estado como uma realidade distribuída entre domínio, investigação, operação, aplicação e interface, mantendo uma direção clara de autoridade e permitindo que o cliente represente, sincronize e atualize esses estados de forma reativa sem transformar a interface em fonte de verdade. A arquitetura deverá preservar a continuidade da exploração, a independência das operações, a distinção entre ações, eventos e estados e a recuperação segura diante de mudanças externas, concorrência ou perda de comunicação.**

---

## 27. Conclusão

A Etapa 6.3.2 está **conceitualmente encerrada e aprovada para transição à Etapa 6.3.3**.

A investigação não identificou incerteza conceitual relevante capaz de alterar as decisões consolidadas. As questões restantes — framework, gerenciamento concreto de estado, roteamento, implementação de realtime, cache, componentes e demais mecanismos tecnológicos — são decisões de arquitetura e implementação e deverão ser tratadas nas etapas posteriores.

**Estado da etapa: CONSOLIDADA E APROVADA.**
