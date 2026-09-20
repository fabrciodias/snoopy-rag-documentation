# REGISTRO MESTRE — RESGATE DO FUNDAMENTAL PARA A V3 ALPHA

**Projeto:** Snoopy-RAG
**Etapa:** Resgate do Fundamental das Etapas 5, 6.1 e 6.2
**Objetivo:** Consolidar os requisitos das Etapas 5, 6.1 e 6.2 que são indispensáveis para que a V3 Alpha deixe de ser apenas uma evolução visual da V2 e passe a materializar, de forma funcional e coerente, os princípios arquiteturais fundamentais da V3.

---

## 1. Natureza desta etapa

Esta etapa não constitui uma nova investigação arquitetural nem uma revisão das decisões já consolidadas nas Etapas 5, 6.1 e 6.2.

Seu objetivo é realizar uma **extração orientada ao escopo da Alpha**.

As Etapas 5, 6.1 e 6.2 definem uma arquitetura e capacidades mais amplas para o Snoopy. A V3 Alpha, entretanto, é um recorte temporal e funcional destinado à implementação e demonstração inicial da nova arquitetura.

Portanto, nem tudo que é fundamental para o Snoopy definitivo precisa ser implementado na Alpha.

O critério de inclusão adotado é:

> **Uma capacidade entra na Alpha quando sua ausência impediria que a V3 surgisse materialmente como uma evolução arquitetural real do Snoopy, em vez de representar apenas uma evolução visual ou uma adaptação da V2.**

A Alpha não busca implementar o Snoopy definitivo em escala reduzida. Busca materializar seu **núcleo arquitetural fundamental**.

---

# 2. Princípio geral de escopo

A Alpha deve implementar o mínimo necessário para demonstrar uma nova espinha dorsal:

```text
DOCUMENTO
   ↓
REPRESENTAÇÃO
   ↓
SEGMENTAÇÃO
   ↓
RECUPERAÇÃO
   ↓
EVIDÊNCIA
   ↓
PROVENIÊNCIA
   ↓
INVESTIGAÇÃO
   ↓
FONTE
```

Essa espinha dorsal deve estar integrada aos fluxos já definidos nas Alpha A e B:

```text
Investigação
    ↓
Recuperação
    ↓
Resultados
    ↓
Evidência
    ↓
Contexto
    ↓
Fonte / Leitura
    ↓
Retorno à investigação
```

A implementação deve ser suficientemente real para demonstrar essa arquitetura, mas não deve incorporar capacidades cuja implementação completa não seja necessária para o surgimento da V3 Alpha.

Princípio operacional:

> **Comprimir o escopo, não os princípios.**

---

# 3. Inventário do fundamental

O resgate consolidou **sete blocos fundamentais** para a V3 Alpha:

1. Representação documental estruturada;
2. Segmentação governada pelo Snoopy;
3. Recuperação híbrida;
4. Evidência e proveniência mínima;
5. Processamento documental e publicação controlada;
6. Persistência e separação de responsabilidades;
7. Operação e segurança fundamentais.

Além deles, a **síntese estruturada** e o **Structured Response / Presentation Renderer**, já consolidados nas Alpha A e B, precisam ser considerados na arquitetura integrada da Alpha.

---

# 4. Representação documental estruturada

## 4.1 Requisito

A Alpha deve abandonar a concepção de processamento equivalente a:

```text
PDF
 ↓
texto/Markdown
 ↓
chunks
 ↓
embeddings
```

A V3 deve introduzir uma representação documental intermediária e estruturada:

```text
DOCUMENTO
 ↓
REPRESENTAÇÃO DOCUMENTAL
 ↓
DERIVAÇÕES
 ↓
SEGMENTAÇÃO
 ↓
RECUPERAÇÃO
```

A representação documental deve preservar, em nível mínimo:

* identidade do documento;
* relação com a fonte original;
* páginas;
* elementos/blocos;
* conteúdo textual;
* ordem;
* localização;
* estrutura necessária para relacionar unidades recuperadas ao documento de origem.

A representação canônica definitiva permanece mais ampla e não precisa ser completamente implementada na Alpha.

## 4.2 Limite da Alpha

Não são requisitos funcionais da Alpha:

* reconstrução visual perfeita do documento;
* preservação multimodal completa;
* representação definitiva de todas as propriedades tipográficas;
* coordenadas e relações espaciais completas;
* processamento avançado de elementos não textuais.

A arquitetura deve, entretanto, evitar uma representação que impossibilite a incorporação dessas capacidades posteriormente.

---

# 5. OCR

## 5.1 Decisão

**OCR fica fora da V3 Alpha.**

Essa exclusão é exclusivamente de escopo temporal e de implementação.

OCR permanece como capacidade importante do Snoopy definitivo, especialmente para documentos digitalizados, conforme investigado na Etapa 6.2.

A ausência de OCR na Alpha não constitui rejeição da capacidade nem alteração da arquitetura definitiva.

## 5.2 Escopo Alpha

A Alpha poderá trabalhar com documentos cujo conteúdo textual esteja disponível sem OCR.

Documentos que dependam de OCR ficam para o pós-Alpha.

> **OCR: pós-Alpha.**

---

# 6. Estrutura e layout documental

## 6.1 Fundamental para a Alpha

A representação utilizada pela Alpha deve preservar, em nível mínimo:

* página;
* ordem de leitura;
* blocos;
* estrutura textual básica;
* localização suficiente para retorno à fonte.

Isso é necessário para que uma unidade recuperada possa ser relacionada ao documento de maneira significativa.

## 6.2 Elementos complexos

Tabelas, imagens, gráficos, fórmulas e outros elementos multimodais não precisam ser plenamente processados na Alpha.

A exigência é que a arquitetura documental não os torne estruturalmente impossíveis de representar ou incorporar futuramente.

Portanto:

| Capacidade                           | Alpha       |
| ------------------------------------ | ----------- |
| Página                               | Fundamental |
| Ordem de leitura                     | Fundamental |
| Blocos                               | Fundamental |
| Estrutura textual básica             | Fundamental |
| Tabelas plenamente estruturadas      | Pós-Alpha   |
| Interpretação de imagens             | Pós-Alpha   |
| Interpretação de gráficos            | Pós-Alpha   |
| Interpretação multimodal de fórmulas | Pós-Alpha   |
| Processamento multimodal avançado    | Pós-Alpha   |

---

# 7. Segmentação governada

## 7.1 Requisito

A Alpha deve possuir unidades de recuperação identificáveis e relacionadas estruturalmente ao documento.

Fluxo:

```text
DOCUMENTO
   ↓
ELEMENTOS / ESTRUTURA
   ↓
SEGMENTAÇÃO
   ↓
UNIDADES DE RECUPERAÇÃO
```

Cada unidade deve possuir identidade própria e relação com:

* documento;
* localização;
* contexto estrutural.

A distinção entre unidade e contexto deve ser preservada.

## 7.2 Princípio de governança

Permanece vigente:

> **Algoritmos externos propõem; o Snoopy governa.**

Um algoritmo ou biblioteca de segmentação pode ser utilizado na implementação. Entretanto, o algoritmo não deve determinar silenciosamente o modelo conceitual das unidades do Snoopy.

A identidade e a semântica da unidade pertencem ao Snoopy.

## 7.3 Fora da Alpha

Não são necessários:

* múltiplos perfis de segmentação;
* comparação sistemática de algoritmos;
* experimentação extensiva;
* resegmentação sofisticada;
* avaliação comparativa formal de estratégias.

Uma estratégia funcional e coerente de segmentação é suficiente para a Alpha.

---

# 8. Recuperação híbrida

## 8.1 Requisito

A recuperação constitui um dos núcleos funcionais da V3 Alpha.

O fluxo mínimo é:

```text
CONSULTA
   ↓
FILTROS ESTRUTURADOS
   ↓
RECUPERAÇÃO LEXICAL + SEMÂNTICA
   ↓
FUSÃO
   ↓
DEDUPLICAÇÃO
   ↓
CONTEXTUALIZAÇÃO
   ↓
RERANKING
   ↓
SELEÇÃO
   ↓
EVIDÊNCIA
```

Devem ser preservadas as distinções entre:

* recuperação;
* ranking;
* seleção;
* evidência;
* interpretação metodológica.

Uma unidade recuperada não se torna automaticamente uma evidência metodologicamente interpretada.

## 8.2 Capacidades que entram na Alpha

* busca lexical;
* busca semântica;
* recuperação híbrida;
* filtros estruturados;
* fusão dos resultados;
* deduplicação por identidade;
* contextualização;
* reranking;
* seleção.

## 8.3 Limites

Não entram na Alpha:

* múltiplos rerankers;
* comparação sistemática entre rerankers;
* MMR;
* múltiplos motores especializados;
* decomposição sofisticada de consultas;
* avaliação formal de precision/recall;
* experimentação extensiva da recuperação.

O reranking é requisito arquitetural.

A tecnologia/modelo específico do reranker será definida na Etapa C.

---

# 9. Evidência

## 9.1 Requisito

A evidência deve ser um objeto identificável da investigação.

Não deve ser representada apenas como texto recuperado sem identidade.

Conceitualmente:

```text
EVIDENCE
 ├── evidence_id
 ├── unit_id
 ├── context
 ├── document_id
 └── location
```

A forma concreta do modelo será definida na Arquitetura Alpha Integrada.

## 9.2 Relação com a investigação

A evidência deve poder ser:

* identificada;
* contextualizada;
* aberta;
* relacionada ao documento;
* relacionada à fonte;
* reaberta a partir da investigação.

Isso mantém o princípio já consolidado na Alpha A:

> **resultado não é automaticamente evidência; evidência é um objeto navegável da investigação.**

---

# 10. Proveniência mínima

## 10.1 Requisito

A Alpha precisa demonstrar rastreabilidade documental mínima.

A cadeia necessária é:

```text
EVIDÊNCIA
   ↓
UNIDADE
   ↓
DOCUMENTO
   ↓
LOCALIZAÇÃO
   ↓
FONTE ORIGINAL
```

O mínimo necessário inclui:

* `document_id`;
* `unit_id`;
* localização;
* fonte original;
* contexto textual/estrutural necessário.

Essa estrutura deve permitir responder:

> **“De onde veio esta evidência?”**

## 10.2 Limite

A Alpha não implementará a arquitetura completa de proveniência prevista para o Snoopy definitivo.

Ficam para o pós-Alpha:

* grafo completo de proveniência;
* reconstrução temporal completa;
* histórico detalhado de derivações;
* implementação completa de modelos formais de proveniência;
* auditoria documental completa.

---

# 11. Processamento documental e publicação

## 11.1 Requisito

O processamento documental deve ser tratado como uma operação identificável, e não como uma transformação invisível.

Fluxo mínimo:

```text
DOCUMENTO
   ↓
OPERAÇÃO / JOB
   ↓
PROCESSAMENTO
   ↓
VALIDAÇÃO
   ↓
PUBLICAÇÃO
   ↓
ACTIVE
```

O processamento deve possuir estados explícitos.

Estados definidos na Alpha:

```text
PENDING
PROCESSING
ACTIVE
FAILED
REJECTED
REMOVED
```

Somente documentos em `ACTIVE` entram na recuperação.

## 11.2 Princípios fundamentais

A implementação deve preservar:

* processamento identificável;
* falha explícita;
* idempotência em nível apropriado;
* controle mínimo de concorrência;
* publicação somente de processamento válido;
* ausência de publicação parcial/incompleta.

## 11.3 Assincronicidade

O processamento documental deve ser assíncrono.

A infraestrutura concreta para isso será definida na Etapa C.

A decisão arquitetural sobre fila/worker permanece compatível com a investigação da Etapa 6.2, incluindo a possibilidade de utilização de PGMQ/Supabase Queues e workers Python, mas a tecnologia não constitui, por si só, o requisito fundamental.

O requisito é:

> **processamento assíncrono, identificável e controlado.**

---

# 12. Persistência e separação de responsabilidades

## 12.1 Requisito

A Alpha deve preservar a distinção entre:

```text
FONTE ORIGINAL
≠
ESTADO LÓGICO
≠
ARTEFATO DERIVADO
```

No contexto consolidado:

```text
Google Drive
   ↓
fonte documental original

PostgreSQL / Supabase
   ↓
estado lógico e dados do sistema

armazenamento físico
   ↓
artefatos derivados, quando necessários
```

A forma concreta de integração será definida na Etapa C.

## 12.2 Identidade

Também deve permanecer a distinção entre:

* identidade externa da fonte (`drive_file_id`);
* identidade interna do documento (`document_id`);
* identidade da unidade (`unit_id`);
* identidade da operação (`operation_id`).

Essas identidades não devem ser confundidas.

## 12.3 Fora da Alpha

Não são necessários:

* versionamento documental completo;
* snapshots;
* histórico integral de versões;
* reconstrução temporal;
* mecanismos sofisticados de armazenamento de artefatos;
* estratégias definitivas de migração.

---

# 13. Síntese

A síntese faz parte da experiência funcional da Alpha.

O fluxo não deve ser:

```text
LLM
 ↓
resposta
 ↓
referência
```

A relação correta é:

```text
RECUPERAÇÃO
   ↓
EVIDÊNCIAS
   ↓
CONTEXTO
   ↓
SÍNTESE
```

A síntese é derivada da investigação e não substitui as fontes primárias.

A evidência continua sendo objeto independente e navegável.

---

# 14. Structured Response e apresentação

A Alpha deve manter a separação já consolidada na Alpha B entre:

1. representação documental canônica;
2. representação estruturada da resposta;
3. representação de apresentação.

Fluxo:

```text
StructuredResponse
        ↓
Snoopy Presentation Renderer
        ↓
HTML / Components
        ↓
Interface
```

Markdown pode existir como representação derivada para exportação, cópia ou interoperabilidade, mas não constitui requisito para a apresentação da interface.

As evidências devem ser representadas semanticamente, com identidade estável, e não por hacks textuais como marcadores inseridos na resposta.

A implementação concreta do renderer será definida na Etapa C.

---

# 15. Operação, segurança e resiliência

## 15.1 Fundamentos que entram na Alpha

A Alpha deve respeitar:

* backend como autoridade de autorização;
* conteúdo documental, recuperado, derivado e produzido por modelos como dado não confiável;
* ausência de injeção arbitrária de HTML;
* estados operacionais explícitos;
* distinção entre falha e perda de comunicação;
* revalidação após reconexão;
* operações repetíveis com idempotência quando aplicável;
* publicação coerente;
* isolamento adequado do corpus.

## 15.2 Comunicação

A comunicação deve preservar o modelo já consolidado na Alpha B:

```text
REQUEST / RESPONSE
+
OPERAÇÕES ASSÍNCRONAS
+
SSE
```

Uma operação assíncrona possui identidade (`operation_id`).

SSE atua como canal de atualização e não como fonte definitiva do estado.

Em caso de perda de comunicação:

```text
SSE perdido
   ↓
reconexão
   ↓
refetch / revalidação
   ↓
estado atual
```

Perda de comunicação não deve ser interpretada automaticamente como falha da operação.

## 15.3 Fora da Alpha

Não entram:

* threat modeling completo;
* auditoria de segurança;
* pentest;
* infraestrutura distribuída;
* disaster recovery completo;
* failover sofisticado;
* autoscaling;
* observabilidade avançada;
* mecanismos complexos de autorização;
* infraestrutura offline-first.

---

# 16. Capacidades explicitamente pós-Alpha

As seguintes capacidades não fazem parte da implementação da V3 Alpha, embora algumas permaneçam relevantes para a arquitetura definitiva:

* OCR;
* processamento avançado de documentos digitalizados;
* multimodalidade avançada;
* compreensão de gráficos;
* compreensão de imagens;
* compreensão multimodal de fórmulas;
* tabelas plenamente estruturadas e exploráveis;
* múltiplos perfis de segmentação;
* comparação sistemática de segmentadores;
* múltiplos rerankers;
* MMR;
* decomposição sofisticada de consultas;
* avaliação formal e extensiva da recuperação;
* versionamento completo;
* snapshots;
* histórico documental completo;
* proveniência completa;
* grafo completo de proveniência;
* Reading Mode definitivo;
* múltiplas investigações persistentes;
* colaboração;
* infraestrutura distribuída;
* observabilidade avançada;
* disaster recovery;
* offline-first;
* GraphQL;
* WebSockets caso SSE seja suficiente;
* event sourcing;
* CQRS;
* microserviços;
* API gateway;
* sincronização complexa;
* demais capacidades não necessárias para a materialização inicial da V3.

A ausência dessas capacidades na Alpha não deve impedir sua incorporação posterior, desde que a implementação Alpha preserve as fronteiras arquiteturais necessárias.

---

# 17. Síntese do inventário

O núcleo indispensável da V3 Alpha pode ser resumido como:

```text
REPRESENTAÇÃO DOCUMENTAL
        ↓
SEGMENTAÇÃO GOVERNADA
        ↓
RECUPERAÇÃO HÍBRIDA
        ↓
EVIDÊNCIA
        ↓
PROVENIÊNCIA MÍNIMA
        ↓
SÍNTESE
        ↓
STRUCTURED RESPONSE
        ↓
PRESENTATION RENDERER
```

Atravessando o fluxo:

```text
IDENTIDADE
ESTADO
PERSISTÊNCIA
PROCESSAMENTO
OPERAÇÕES
SEGURANÇA
```

E conectado à experiência:

```text
Investigação
   ↓
Resultados
   ↓
Evidência
   ↓
Contexto
   ↓
Fonte / Leitura
   ↓
Retorno à Investigação
```

---

# 18. Relação com as etapas anteriores

O inventário desta etapa não substitui as decisões já tomadas.

A composição correta é:

```text
ETAPA 5
Princípios e requisitos arquiteturais
        +
ETAPA 6.1
Critérios técnicos
        +
ETAPA 6.2
Investigação e decisões técnicas
        ↓
RESGATE DO FUNDAMENTAL
        +
ALPHA A
Experiência de investigação e corpus
        +
ALPHA B
Interface, segurança e frontend
        ↓
C — ARQUITETURA ALPHA INTEGRADA
```

A Etapa C deverá integrar esses três conjuntos em uma arquitetura implementável.

---

# 19. Critério de encerramento desta etapa

Esta etapa é considerada encerrada quando:

1. os requisitos fundamentais das Etapas 5, 6.1 e 6.2 necessários à Alpha foram identificados;
2. o escopo de implementação foi reduzido ao necessário para materializar a V3;
3. as capacidades explicitamente pós-Alpha foram separadas;
4. não existem, no inventário, requisitos definitivos confundidos com requisitos de implementação imediata;
5. OCR e demais capacidades não essenciais não retornam ao escopo da Alpha sem nova decisão;
6. o inventário pode ser utilizado diretamente como entrada da Arquitetura Alpha Integrada.

---

# 20. Decisão final

Fica consolidado que a V3 Alpha **não será uma V2 maquiada**, mas também não buscará implementar o Snoopy definitivo em sua totalidade.

A Alpha implementará o núcleo necessário para materializar a nova arquitetura:

> **representação documental → segmentação → recuperação → evidência → proveniência → investigação → fonte**, integrado ao processamento, persistência, operações e apresentação estruturada definidos nas etapas anteriores.

Capacidades importantes para o Snoopy definitivo, mas não necessárias para o surgimento demonstrável da V3 dentro do prazo da Alpha, permanecerão explicitamente pós-Alpha.

O próximo passo é a **Etapa C — Arquitetura Alpha Integrada**, que deverá transformar este inventário, juntamente com Alpha A e Alpha B, em um mapa técnico concreto de implementação.

**Status:** CONSOLIDADO / ENCERRADO
**Próxima etapa:** C — Arquitetura Alpha Integrada
