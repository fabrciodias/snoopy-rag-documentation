# D.9 — Contrato de Implementação para o Executor

## 1. Natureza deste documento

Este documento não introduz uma nova etapa arquitetural.

As decisões arquiteturais do Snoopy V3 Alpha foram consolidadas nas etapas D.1–D.8. A D.9 estabelece as **regras de execução da implementação** dessas decisões.

O executor — neste caso, o Gemini — possui liberdade para escolher detalhes locais de implementação quando esses detalhes não estiverem definidos pela arquitetura. Essa liberdade, entretanto, não inclui reinterpretar, simplificar, substituir ou ampliar silenciosamente decisões já fechadas.

A regra fundamental é:

> **O executor pode decidir como implementar aquilo que foi decidido. Não pode decidir novamente o que já foi decidido.**

---

# 2. Hierarquia das decisões

Durante a implementação, deve ser respeitada a seguinte ordem de autoridade:

1. **Decisões consolidadas nas etapas D.1–D.8**
2. Decisões anteriores das etapas A–C quando incorporadas e não contraditas pelos registros D
3. Estado real do V2, utilizado como substrato de migração e fonte de informação sobre o que efetivamente existe
4. Decisões locais de implementação tomadas pelo executor

O código existente do V2 **não possui autoridade arquitetural sobre a V3**.

Quando houver conflito entre uma estrutura existente no V2 e uma decisão arquitetural V3 já consolidada, a arquitetura V3 prevalece, preservando-se os dados e comportamentos compatíveis sempre que possível.

Quando a documentação não determinar uma decisão e o V2 também não fornecer informação suficiente, o executor deve distinguir:

* **decisão local de implementação**, que pode ser tomada;
* **decisão arquitetural ausente**, que deve ser sinalizada antes de prosseguir.

---

# 3. Regra de não-reinterpretação

Nenhuma das seguintes decisões pode ser alterada silenciosamente:

### 3.1 Unidade documental

`RetrievalUnit` é a unidade conceitual de recuperação da V3.

O executor não deve tratar o conceito de `chunk` do V2 como se fosse automaticamente equivalente a `RetrievalUnit`.

O código de chunking existente pode ser reutilizado, adaptado ou substituído como algoritmo, mas não deve determinar sozinho o modelo conceitual da V3.

---

### 3.2 Representação documental

`DocumentRepresentation` é a representação canônica do documento para o processamento interno.

Markdown **não deve ser promovido a representação documental canônica**.

Markdown pode existir como:

* formato derivado;
* formato auxiliar;
* exportação;
* interoperabilidade;
* artefato temporário.

Mas o fluxo arquitetural não pode voltar a ser:

`PDF → Markdown → chunks`

como representação fundamental do documento.

O mínimo da representação deve preservar aquilo que foi estabelecido em D.3/D.7/D.8: identidade, páginas, blocos, texto, ordem, localização e estrutura básica.

---

### 3.3 Operações

`Operation` é a abstração arquitetural para execução assíncrona.

O conceito de `jobs` existente no V2 pode ser utilizado como substrato de implementação ou mecanismo de transição, mas não pode simplesmente ser renomeado e considerado equivalente à arquitetura V3.

A execução deve preservar a separação entre:

`entidade → estado da entidade`

e

`Operation → estado da execução`.

Retry de uma operação deve produzir uma nova `operation_id`.

---

### 3.4 Recuperação

`Retrieval` não é sinônimo de busca vetorial.

A arquitetura Alpha exige a cadeia:

`filtros → corpus ACTIVE → lexical + semantic → fusion → dedup → contextualização → reranking → selection → RetrievalResult`

O executor pode escolher os algoritmos e bibliotecas concretos quando esses detalhes não estiverem definidos, mas não pode eliminar silenciosamente etapas fundamentais da cadeia apenas porque uma implementação mais simples é conveniente.

---

### 3.5 Resultado e evidência

`RetrievalResult ≠ Evidence`.

Um resultado recuperado não deve ser transformado automaticamente em evidência apenas para simplificar a implementação.

A passagem:

`RetrievalResult → Evidence`

é uma responsabilidade arquitetural própria.

A evidência deve possuir identidade estável (`evidence_id`) e proveniência mínima suficiente para reconstruir:

`Evidence → RetrievalUnit → Document → Location → Original Source`.

---

### 3.6 Síntese

A cadeia deve permanecer:

`Retrieval → Evidence → Context → Synthesis`

A síntese produzida pelo LLM é derivada.

Ela não substitui:

* o documento original;
* a representação documental;
* a RetrievalUnit;
* a evidência;
* a proveniência.

O executor não deve transformar a saída do LLM na nova fonte de verdade do sistema.

---

### 3.7 StructuredResponse

`StructuredResponse` é a representação semântica da resposta da investigação.

A cadeia principal deve permanecer:

`StructuredResponse → Presentation Renderer → interface`

e não:

`LLM → Markdown → interface`

Markdown pode existir como saída derivada, mas não pode substituir o contrato estrutural.

As referências a evidências devem utilizar identidade semântica estável, e não depender de marcadores textuais frágeis como `[TRECHO 1]`.

---

### 3.8 Frontend

O frontend Vue 3 + TypeScript é responsável por:

* apresentação;
* interação;
* navegação;
* estado visual;
* estado local apropriado;
* consumo dos contratos da API;
* reação às mudanças comunicadas pelo SSE.

O frontend não deve assumir responsabilidades do domínio.

Ele não decide, por exemplo:

* quando um documento é `ACTIVE`;
* se uma operação realmente terminou;
* se um usuário possui autorização;
* se um resultado constitui evidência;
* qual documento é fonte de verdade;
* como a recuperação deve ser realizada.

---

### 3.9 SSE

SSE é um **canal de atualização**, não uma fonte de verdade.

A regra é:

`evento SSE → identificar recurso alterado → GET/revalidação → estado confirmado → interface`

Nunca:

`evento SSE → assumir que o payload é o estado definitivo`.

Perda de SSE não significa falha da operação.

Timeout não significa automaticamente falha.

Desconexão não significa automaticamente cancelamento.

O estado deve continuar recuperável por meio da API autoritativa.

---

### 3.10 Publicação

Um documento não pode ser marcado como `ACTIVE` apenas porque o processamento parcial terminou.

O fluxo mínimo é:

`PROCESSING`
→ `DocumentRepresentation`
→ validação
→ `RetrievalUnit[]`
→ embeddings + índice lexical
→ validação
→ `ACTIVE`

Uma publicação parcialmente processada não deve entrar no corpus pesquisável.

---

# 4. Regra de migração do V2

A migração deve ser **progressiva**, não uma reescrita cega.

O executor deve primeiro compreender o que existe e então classificar cada estrutura como:

* preservar;
* adaptar;
* substituir;
* remover;
* validar.

A reutilização de código é desejável quando o código existente possui uma responsabilidade compatível com a arquitetura V3.

A reutilização deixa de ser desejável quando preserva uma responsabilidade arquitetural incompatível.

Portanto:

> **Reutilizar código não significa preservar a arquitetura do V2.**

O executor deve evitar tanto:

* reescrever tudo desnecessariamente;
* quanto manter estruturas do V2 apenas porque já funcionam.

---

# 5. Regra sobre o banco real

O banco existente deve ser tratado como **estado real do sistema**, não como modelo-alvo da V3.

Antes de realizar migrações estruturais importantes, o executor deve verificar:

* tabelas existentes;
* colunas;
* relações;
* constraints;
* índices;
* funções/RPCs;
* políticas RLS;
* dados existentes;
* relações de documentos/acervo;
* mecanismo atual de processamento;
* mecanismos atuais de busca.

Não é permitido inventar o schema real com base apenas nos nomes utilizados pelo código.

Se houver divergência entre código e banco, a divergência deve ser investigada.

---

# 6. Regra de segurança

Credenciais, chaves e tokens existentes não devem ser expostos em código, documentação, commits ou respostas.

Credenciais que tenham sido expostas durante o processo de desenvolvimento devem ser tratadas conforme a necessidade de rotação já estabelecida no baseline.

O executor deve manter:

* autorização no backend;
* RLS como defesa em profundidade;
* separação entre conteúdo de documentos e código executável;
* ausência de HTML arbitrário confiável;
* ausência de stack traces ou detalhes internos em respostas públicas da API.

Conteúdo recuperado de documentos, LLMs, arquivos ou fontes externas deve ser tratado como **dados não confiáveis**.

---

# 7. Regra contra atalhos arquiteturais

Não são aceitáveis implementações que produzam uma demonstração visualmente convincente enquanto deixam de existir os contratos arquiteturais fundamentais.

Em particular, não é aceitável:

* simular `RetrievalUnit` usando apenas os chunks antigos;
* fingir proveniência sem cadeia reconstruível;
* retornar evidência apenas como texto sem identidade;
* colocar o estado inteiro no frontend;
* considerar SSE como banco de dados;
* marcar documentos como `ACTIVE` antes da indexação válida;
* chamar uma busca vetorial de “hybrid retrieval” sem a arquitetura correspondente;
* gerar StructuredResponse apenas como uma string Markdown;
* criar uma “vertical slice” que contorne os componentes reais do fluxo.

A Alpha deve demonstrar **arquitetura funcionando**, não apenas uma interface funcionando.

---

# 8. Regra de escopo

O executor deve respeitar o limite da Alpha.

Não devem ser introduzidos, salvo necessidade arquitetural demonstrada:

* OCR;
* multimodalidade avançada;
* múltiplos perfis de segmentação;
* múltiplos rerankers;
* MMR;
* query decomposition sofisticada;
* avaliação formal de Information Retrieval;
* versionamento completo;
* snapshots;
* histórico completo de proveniência;
* colaboração;
* infraestrutura distribuída;
* microserviços;
* Kubernetes;
* event sourcing;
* CQRS;
* GraphQL;
* offline-first;
* observabilidade avançada;
* disaster recovery avançado;
* outras capacidades explicitamente classificadas como pós-Alpha.

A ausência dessas capacidades não deve ser tratada como falha da Alpha.

> **Comprimir o escopo, não os princípios.**

---

# 9. Liberdade de implementação do executor

O executor possui liberdade para escolher detalhes que não estejam arquiteturalmente fechados.

Por exemplo, pode decidir, dentro das fronteiras estabelecidas:

* nomes internos de funções;
* organização de pequenos módulos;
* biblioteca HTTP;
* biblioteca de serialização;
* mecanismo concreto de composição de estado Vue;
* implementação local de um adapter;
* algoritmo concreto de uma etapa quando a arquitetura exige a etapa, mas não fixa o algoritmo;
* formato interno de objetos intermediários;
* estratégia de testes unitários;
* detalhes de tratamento de exceções;
* convenções de código.

Essa liberdade existe para evitar que a documentação tente decidir cada detalhe da implementação.

Porém:

> **Uma decisão local não pode alterar uma fronteira arquitetural.**

---

# 10. Quando o executor deve parar

O executor deve interromper a implementação daquela parte e sinalizar a questão quando encontrar uma decisão que:

1. não foi definida em D.1–D.8;
2. não pode ser resolvida como simples detalhe local;
3. altera uma entidade, estado, fronteira, fluxo ou contrato;
4. cria dependência arquitetural nova;
5. exige remover ou reinterpretar uma decisão já consolidada;
6. entra em conflito com o estado real do V2 de maneira que não possa ser resolvida localmente.

Exemplos:

* necessidade de criar uma nova entidade arquitetural;
* necessidade de alterar estados de Document ou Operation;
* necessidade de transformar Reading em entidade persistida;
* necessidade de substituir HTTP/SSE por outro modelo;
* necessidade de alterar o fluxo documental;
* necessidade de tornar Markdown canônico;
* necessidade de transformar `RetrievalResult` em `Evidence` automaticamente;
* necessidade de introduzir um novo serviço arquitetural;
* necessidade de mudar o significado de `ACTIVE`.

Nesses casos, o executor não deve “resolver sozinho” para manter o progresso.

Deve apresentar:

1. o problema encontrado;
2. a decisão afetada;
3. por que a implementação não pode prosseguir sem uma escolha;
4. as alternativas técnicas relevantes;
5. a consequência de cada alternativa.

---

# 11. Regra para decisões não arquiteturais

Nem toda dúvida exige interrupção.

Se a questão puder ser resolvida sem modificar:

* entidades;
* estados;
* contratos;
* fronteiras;
* responsabilidades;
* fluxo documental;
* fluxo de investigação;
* segurança;
* escopo da Alpha;

ela pode ser resolvida pelo executor.

O objetivo é evitar tanto a invenção arquitetural quanto a burocratização da implementação.

---

# 12. Regra de rastreabilidade

Cada fase de implementação deve poder ser relacionada à arquitetura definida.

O executor deve saber identificar:

`decisão arquitetural → componente implementado → teste/validação`

Especialmente para os elementos fundamentais:

* Document;
* DocumentRepresentation;
* RetrievalUnit;
* Investigation;
* RetrievalResult;
* Evidence;
* Operation;
* StructuredResponse;
* publicação `ACTIVE`;
* Retrieval híbrido;
* proveniência;
* Renderer;
* recuperação após perda de comunicação.

Não é necessário transformar o código em documentação excessivamente burocrática, mas as responsabilidades não podem desaparecer dentro de módulos genéricos.

---

# 13. Regra de teste

O executor não deve considerar uma etapa concluída apenas porque:

* o código compila;
* a aplicação inicia;
* a interface renderiza;
* uma consulta retorna alguma coisa;
* o fluxo feliz funciona superficialmente.

Os critérios de conclusão definidos em D.7 e D.8 devem ser utilizados.

Em particular, devem ser verificadas as invariantes:

* somente documentos `ACTIVE` entram na recuperação;
* documento inválido não se torna `ACTIVE`;
* publicação parcial não é pesquisável;
* `RetrievalUnit` possui identidade e localização;
* `RetrievalResult ≠ Evidence`;
* proveniência é reconstruível;
* retry produz nova `operation_id`;
* timeout não é automaticamente tratado como falha;
* perda de SSE não destrói o estado;
* revalidação recupera o estado real;
* investigação sobrevive à navegação;
* Evidence Drawer abre evidência identificável;
* Reading permite retorno à investigação;
* StructuredResponse é renderizado estruturalmente;
* conteúdo não confiável não é tratado como HTML confiável;
* autorização ocorre no backend.

---

# 14. Regra da Vertical Slice

A Alpha só pode ser considerada funcionalmente demonstrada quando existir um fluxo real:

`Google Drive`
→ `Document`
→ `Operation`
→ `DocumentRepresentation`
→ `Segmentation`
→ `RetrievalUnit[]`
→ `Lexical + Vector Index`
→ `Publication`
→ `ACTIVE`
→ `Investigation`
→ `Hybrid Retrieval`
→ `RetrievalResult[]`
→ `Evidence`
→ `Context`
→ `Synthesis`
→ `StructuredResponse`
→ `Presentation Renderer`
→ `Investigation UI`
→ `Evidence Drawer`
→ `Reading / Source`
→ `Return`

A implementação não deve substituir esse fluxo por mocks, atalhos ou caminhos paralelos apenas para demonstrar a interface.

---

# 15. Regra contra Frankenstein arquitetural

Durante a migração, componentes V2 e V3 podem coexistir temporariamente.

Essa coexistência deve ser explícita.

Adapters, wrappers e camadas de compatibilidade são aceitáveis quando ajudam a migração.

O que deve ser evitado é uma mistura em que:

* o frontend V3 dependa diretamente de detalhes V2;
* o domínio conheça estruturas de banco;
* o novo Retrieval dependa implicitamente do modelo antigo de chunks;
* `server.js` continue acumulando todas as responsabilidades;
* o estado do V3 dependa de variáveis globais do V2;
* duas arquiteturas diferentes passem a controlar simultaneamente a mesma responsabilidade.

A migração pode ser gradual.

A arquitetura não deve ser ambígua.

---

# 16. Regra de comunicação durante a implementação

Ao reportar progresso, o executor deve distinguir:

### Implementado

Aquilo que efetivamente está funcionando.

### Adaptado

Aquilo que foi reutilizado do V2, mas passou a cumprir uma responsabilidade V3.

### Temporário

Aquilo que existe como mecanismo de transição e ainda será substituído.

### Pendente

Aquilo que faz parte da Alpha mas ainda não foi implementado.

### Bloqueado

Aquilo que não pode prosseguir sem decisão arquitetural ou informação externa.

### Pós-Alpha

Aquilo que deliberadamente não será implementado nesta etapa.

Essa distinção impede que código temporário seja apresentado como arquitetura definitiva.

---

# 17. Regra de desvios

Se, durante a implementação, for descoberta uma necessidade real de alterar uma decisão arquitetural consolidada, o executor não deve realizar a alteração silenciosamente.

Deve registrar:

* decisão original;
* evidência descoberta;
* motivo do conflito;
* impacto;
* proposta de alteração;
* partes da implementação afetadas.

A alteração só deve ser considerada arquiteturalmente válida depois de explicitamente revisada.

---

# 18. Regra final

O executor deve trabalhar com a seguinte ordem mental:

> **Primeiro compreender. Depois mapear. Depois adaptar. Depois implementar. Depois validar.**

Não:

> “Encontrar código parecido → renomear → encaixar → declarar V3.”

O Snoopy V3 Alpha é uma evolução arquitetural do V2, não uma troca cosmética de nomes.

---

# 19. Contrato resumido para o executor

O executor pode:

* reutilizar código;
* adaptar módulos;
* escolher bibliotecas;
* decidir detalhes locais;
* criar adapters;
* reorganizar implementação;
* otimizar internamente;
* escolher algoritmos concretos quando a arquitetura não os fixou.

O executor não pode, sem revisão:

* redefinir entidades;
* redefinir estados;
* redefinir fronteiras;
* redefinir fluxos;
* transformar chunk em unidade conceitual;
* transformar Markdown em representação canônica;
* transformar jobs novamente em modelo arquitetural de execução;
* tratar busca vetorial como Retrieval completo;
* transformar resultado automaticamente em evidência;
* transformar LLM em fonte de verdade;
* transformar Markdown em StructuredResponse;
* mover domínio para o frontend;
* transformar SSE em fonte de verdade;
* publicar documento antes da indexação válida;
* introduzir microserviços ou infraestrutura distribuída sem necessidade arquitetural;
* ampliar o escopo da Alpha;
* esconder conflitos arquiteturais para manter o cronograma.

Quando uma decisão arquitetural estiver ausente, conflitante ou insuficiente para prosseguir:

> **Parar, explicitar o problema e solicitar decisão. Não inventar.**

---

# 20. Critério de encerramento do contrato

O contrato D.9 estará sendo respeitado quando o executor conseguir implementar a Alpha mantendo simultaneamente:

**identidade → estado → representação → segmentação → recuperação → evidência → proveniência → síntese → StructuredResponse → apresentação → fonte**

sem que nenhuma dessas relações fundamentais seja substituída por um atalho apenas porque o V2 já possuía uma implementação diferente.

O objetivo não é impedir o executor de pensar.

É garantir que ele **pense dentro das fronteiras que já foram deliberadamente estabelecidas**.
