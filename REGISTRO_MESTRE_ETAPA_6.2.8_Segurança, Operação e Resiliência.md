# Snoopy-RAG — Consolidação da Etapa 6.2.8

## Segurança, Operação e Resiliência

**Natureza:** Consolidação da investigação tecnológica e arquitetural
**Status:** Documento de revisão — ainda não constitui o Registro Mestre
**Etapa:** 6.2.8
**Subetapas:** 6.2.8.1 a 6.2.8.4

---

## 1. Finalidade da Etapa 6.2.8

A Etapa 6.2.8 investigou como o Snoopy-RAG V3 deverá operar de maneira segura, consistente, recuperável e sustentável, considerando não apenas o processamento normal, mas também acesso, concorrência, falhas, degradação, observabilidade e recuperação.

A investigação foi organizada em quatro dimensões:

1. **6.2.8.1 — Segurança, Autorização e Isolamento**
2. **6.2.8.2 — Filas, Concorrência e Processamento Assíncrono**
3. **6.2.8.3 — Falhas, Integridade Operacional e Recuperação**
4. **6.2.8.4 — Recursos, Observabilidade e Continuidade**

A decisão de manter essas quatro dimensões dentro de um único futuro Registro Mestre não representa condensação da investigação. Elas foram tratadas como partes de uma mesma função arquitetural: **operação segura e confiável do Snoopy**.

A lógica geral resultante pode ser expressa como:

> **identificar quem pode fazer → controlar quem executa → garantir que falhas não corrompam o estado → perceber quando algo está errado → recuperar o sistema.**

---

# 2. 6.2.8.1 — Segurança, Autorização e Isolamento

## 2.1. Distinções fundamentais

A investigação estabeleceu que:

* autenticação não é autorização;
* autorização não é isolamento;
* identidade não é permissão;
* capacidade técnica não é autorização de negócio.

A autenticação responde **quem é a identidade**.
A autorização responde **o que essa identidade pode fazer**.
O isolamento responde **sobre quais recursos e contextos essa operação pode ocorrer**.

No Snoopy, isso é especialmente importante porque o sistema trabalha simultaneamente com usuários, acervos, documentos, versões, unidades, evidências, jobs, workers e ambientes.

---

## 2.2. O acervo como fronteira de segurança

A principal unidade de autorização não deve ser simplesmente o sistema como um todo.

O Snoopy deve raciocinar sobre uma relação semelhante a:

```text
identidade
    ↓
acervo
    ↓
documento
    ↓
versão
    ↓
unidade
    ↓
operação
```

Assim, o fato de uma pessoa estar autenticada não implica autorização sobre todos os acervos.

Essa conclusão surgiu diretamente do problema dos identificadores enviados pelo cliente: `folder_id`, `document_id`, `job_id`, `evidence_id` e futuros identificadores não podem funcionar como autorização implícita. Conhecer um identificador não pode equivaler a possuir acesso ao recurso correspondente.

---

## 2.3. Autorização no servidor

O frontend pode refletir permissões para fins de experiência de uso, mas não deve constituir a fronteira de segurança.

A autorização de domínio deverá ocorrer no servidor.

O backend deve verificar, no contexto da operação:

* quem está realizando a operação;
* qual recurso está sendo acessado;
* qual acervo está envolvido;
* qual operação está sendo solicitada;
* qual regra autoriza ou nega essa operação.

Isso deverá impedir que um cliente altere simplesmente um identificador para tentar acessar outro recurso.

---

## 2.4. Backend e RLS

A investigação não tratou RLS como solução isolada.

A arquitetura definida conceitualmente é de **defesa em profundidade**:

```text
Usuário
   ↓
Autenticação
   ↓
Backend
   ↓
Autorização de domínio
   ↓
PostgreSQL
   ↓
RLS / enforcement adicional
   ↓
Dados
```

O backend continua sendo responsável pela autorização de negócio.

RLS funciona como camada adicional de proteção, especialmente para isolamento de dados.

Isso é particularmente importante porque credenciais privilegiadas como `service_role` podem bypassar RLS. Portanto:

> **credencial privilegiada não é autorização de negócio.**

O worker ou backend pode possuir capacidade técnica ampla sem que isso determine automaticamente quais recursos devem ser acessados em cada operação.

---

## 2.5. Modelo de autorização composto

A investigação comparou RBAC, ABAC e ReBAC.

### RBAC

Foi considerado adequado para capacidades globais, como:

* administração do sistema;
* funções administrativas;
* capacidades gerais de usuário;
* capacidades técnicas de serviços.

Porém, RBAC puro não representa naturalmente relações específicas com acervos sem produzir uma explosão de papéis.

### ABAC

Foi considerado adequado para condições contextuais, como:

* identidade;
* propriedade;
* visibilidade;
* tipo de recurso;
* operação;
* ambiente;
* estado.

Porém, um modelo exclusivamente baseado em atributos pode tornar as políticas excessivamente complexas e espalhadas.

### ReBAC

Foi considerado especialmente adequado para relações entre identidades e acervos/recursos:

```text
Fabrício ──owner──> Acervo A
João     ──reader─> Acervo A
Osmar    ──admin──> Acervo A
```

Isso representa naturalmente o compartilhamento futuro de acervos em modo somente leitura.

A conclusão arquitetural foi, portanto:

```text
RBAC
  → capacidades/papéis globais

+

ReBAC
  → relações entre identidades e recursos

+

ABAC
  → condições contextuais quando necessárias
```

Não se trata de adotar três sistemas independentes, mas de utilizar esses modelos como princípios complementares dentro da própria arquitetura do Snoopy.

---

## 2.6. Usuário, worker e job

O worker não deve ser tratado como usuário humano.

Devem ser distinguidos:

* identidade humana;
* identidade de serviço;
* backend;
* worker;
* scheduler;
* administração/ROOT.

A operação válida resulta da combinação:

```text
CAPACIDADE
     +
PERMISSÃO
     +
CONTEXTO
     ↓
OPERAÇÃO AUTORIZADA
```

Um worker pode ter capacidade para processar documentos, mas a execução concreta continua vinculada ao contexto do job, do acervo e da versão documental.

---

## 2.7. Segurança: decisão consolidada

A arquitetura V3 deverá utilizar:

* identidade autenticada;
* autorização contextual por recurso/acervo;
* papéis globais quando apropriado;
* relações específicas entre identidades e recursos;
* condições contextuais quando necessárias;
* backend como autoridade de autorização de domínio;
* PostgreSQL/RLS como enforcement adicional;
* separação entre propriedade, administração, visibilidade e permissão;
* separação entre identidades humanas e identidades técnicas.

A implementação concreta de tabelas, middleware, funções, RLS, JWT/claims, ROOT e compartilhamento fica para a 6.3. A investigação considerou desnecessário introduzir, neste momento, plataformas externas como OpenFGA, Keycloak, OPA ou equivalentes apenas para ampliar a lista de tecnologias.

---

# 3. 6.2.8.2 — Filas, Concorrência e Processamento Assíncrono

## 3.1. Problema central

A existência da tabela `jobs` não garante, por si só, uma fila concorrente segura.

O problema fundamental é:

> **como transformar trabalho disponível em uma execução específica de maneira atômica, recuperável e observável?**

Foram identificados como requisitos:

* claim atômico;
* exclusão entre workers;
* identificação da execução;
* recuperação de jobs abandonados;
* retries;
* idempotência;
* vínculo com versão documental;
* proteção contra versões antigas;
* histórico de execução;
* isolamento entre ambientes.

---

## 3.2. Job não é fila

Uma das decisões mais importantes da investigação foi separar:

### `jobs`

Entidade de domínio e estado histórico do processamento.

Responde:

* que trabalho existe;
* sobre qual documento/versão;
* qual operação;
* qual estado;
* quais tentativas ocorreram;
* qual execução está associada;
* qual resultado foi produzido.

### PGMQ / Supabase Queues

Mecanismo operacional de entrega da mensagem aos workers.

Responde:

* existe mensagem aguardando consumo;
* qual consumidor recebeu temporariamente;
* quando a mensagem reaparece;
* se ela foi consumida;
* se deve ser arquivada/removida.

Portanto:

> **Queue message ≠ Job.**

A separação evita transformar a fila em uma segunda fonte de verdade do estado documental.

---

## 3.3. Tecnologia escolhida: PGMQ / Supabase Queues

Foram consideradas:

* PostgreSQL puro com `FOR UPDATE SKIP LOCKED`;
* PGMQ / Supabase Queues;
* Redis Streams;
* BullMQ + Redis;
* pg_cron como scheduler auxiliar;
* pg_net como mecanismo auxiliar.

O PostgreSQL puro é tecnicamente suficiente, mas exigiria construir manualmente grande parte da semântica de fila.

Redis/BullMQ são tecnicamente capazes, porém introduzem uma segunda infraestrutura central.

A solução escolhida para a V3 é:

> **PGMQ / Supabase Queues.**

A justificativa não é simplesmente “já usamos Supabase”, mas o fato de o PostgreSQL já ter sido estabelecido como camada governante do estado lógico. PGMQ acrescenta a semântica operacional de fila mantendo essa camada no mesmo ecossistema, sem exigir Redis.

---

## 3.4. Visibility timeout e reentrega

A entrega deverá utilizar **visibility timeout**, e não `pop`.

O fluxo será conceitualmente:

```text
PGMQ.read
    ↓
mensagem fica invisível
    ↓
worker processa
    ↓
sucesso → publicação → delete/archive
falha   → timeout → mensagem reaparece
```

Se o processamento for longo, o worker poderá renovar a visibilidade.

Isso evita que a mensagem permaneça indefinidamente presa caso o worker morra.

---

## 3.5. Exactly-once não significa exactly-once processing

A investigação estabeleceu uma distinção importante:

> entrega controlada da mensagem não equivale a execução única do pipeline.

Um worker pode:

1. processar;
2. publicar corretamente;
3. cair antes de remover a mensagem;
4. provocar nova entrega.

Portanto, a arquitetura precisa assumir possibilidade de reexecução.

O princípio consolidado é:

```text
at-least-once delivery
+
idempotent processing
+
atomic publication
```

A fila controla entrega; o Snoopy controla validade e idempotência.

---

## 3.6. Job, Attempt e Run

A investigação distinguiu:

```text
Job
 ├── Attempt 1 → failed
 ├── Attempt 2 → failed
 └── Attempt 3 → completed
```

O job representa o trabalho.

A tentativa representa uma execução concreta.

Isso também integra o modelo de proveniência operacional da 6.2.7.

Não ficou decidido que haverá necessariamente uma tabela chamada `attempts`; isso pertence à 6.3.

---

## 3.7. Concorrência entre versões

Foi identificado um race condition crítico:

```text
V1 → Job A ───────────► termina tarde

V2 → Job B → publicada
```

A conclusão tardia do Job A não pode permitir que V1 volte a ser publicada como estado ativo.

A publicação deve verificar se o estado documental que a execução pretende publicar ainda é válido.

Isso vincula diretamente:

* versionamento;
* concorrência;
* publicação;
* proveniência.

Uma versão obsoleta pode continuar registrada historicamente, mas não pode sobrescrever silenciosamente uma versão mais recente.

---

## 3.8. Decisão consolidada da 6.2.8.2

```text
PostgreSQL/Supabase
    │
    ├── estado documental
    ├── provenance
    ├── jobs
    │
    └── PGMQ
          ↓
       workers
          ↓
       processing
          ↓
       validation
          ↓
       atomic publication
```

Decisões:

* **PGMQ/Supabase Queues** como fila;
* `jobs` como estado de domínio;
* workers externos ao banco;
* visibility timeout;
* reentrega;
* renovação de visibilidade;
* processamento idempotente;
* publicação antes da remoção da mensagem;
* histórico de job/attempt/run;
* isolamento explícito entre produção e desenvolvimento;
* Redis/BullMQ fora da V3 inicial;
* PostgreSQL puro mantido como alternativa técnica/fallback.

Os detalhes de tabelas, campos, constraints, funções, tempos e política de retry ficam para 6.3.

---

# 4. 6.2.8.3 — Falhas, Integridade Operacional e Recuperação

## 4.1. Princípio central: fail closed

A regra fundamental é:

> **um estado parcial nunca deve ser publicado como estado válido.**

O processamento pode passar por:

```text
PROCESSING
    ↓
VALIDATING
    ↓
PUBLISHING
    ↓
ACTIVE
```

Somente após todas as condições necessárias serem satisfeitas o estado pode se tornar ativo.

Assim, uma falha durante:

* extração;
* OCR;
* estruturação;
* segmentação;
* embedding;
* persistência;
* proveniência;
* publicação

não deve produzir silenciosamente um documento parcialmente utilizável.

---

## 4.2. PostgreSQL como barreira de integridade

O processamento pesado não deve ocorrer dentro de uma transação longa.

O worker realiza o processamento externamente.

A publicação final ocorre em uma operação transacional curta:

```text
worker
   ↓
solicita publicação
   ↓
transação PostgreSQL
   ↓
verifica pré-condições
   ↓
persiste derivados/provenance
   ↓
altera estado
   ↓
COMMIT
```

Em caso de falha:

```text
ROLLBACK
```

O PostgreSQL passa, portanto, a funcionar como barreira para as transições críticas do estado lógico.

---

## 4.3. Integridade não é apenas existência

A existência de um registro não prova sua validade.

Um embedding pode existir, por exemplo, mas estar associado:

* à versão errada;
* à unidade errada;
* ao modelo errado;
* a configuração incompatível;
* a dimensão incompatível.

A validade deve considerar a cadeia:

```text
document version
    ↓
representation
    ↓
segmentation
    ↓
unit
    ↓
embedding
    ↓
configuration/model
```

---

## 4.4. Classificação das falhas

A arquitetura deve distinguir pelo menos:

* falha transitória;
* falha permanente;
* falha desconhecida;
* falha de infraestrutura;
* falha de dados;
* falha de software;
* falha de integridade.

Falhas transitórias podem gerar retry.

Falhas permanentes não devem permanecer em retry infinito.

Falhas desconhecidas devem possuir retry limitado e posterior diagnóstico.

Essa política deverá ser preferencialmente determinística, não delegada a um LLM.

---

## 4.5. Falha não é sinônimo de estado documental inválido

A investigação também preservou uma distinção metodológica importante.

Por exemplo, OCR abaixo do nível mínimo pode resultar em:

```text
documento processado
→ qualidade insuficiente
→ não entra no retrieval
→ possível revisão humana
```

Isso não precisa ser representado simplesmente como “erro técnico”.

A taxonomia final deverá permitir distinguir conceitos como:

* `FAILED`;
* `REJECTED`;
* `BLOCKED`;
* `NEEDS_REVIEW`;
* `OBSOLETE`.

Os nomes finais ficam para 6.3.

---

## 4.6. Recuperação

A recuperação deve combinar:

* PGMQ para reentrega;
* workers idempotentes;
* estados explícitos;
* transações PostgreSQL;
* publicação atômica;
* versionamento;
* histórico.

Não há necessidade, para a V3, de introduzir um workflow engine adicional como Temporal, Airflow, Prefect, Dagster ou equivalente.

A combinação:

```text
PostgreSQL
+
PGMQ
+
workers
+
estados
+
versionamento/provenance
```

é considerada suficiente para a arquitetura investigada.

---

## 4.7. Decisão consolidada da 6.2.8.3

O Snoopy V3 deverá tratar falhas por meio de:

* transações PostgreSQL;
* estados explícitos;
* PGMQ para reentrega;
* workers idempotentes;
* publicação transacional;
* retries limitados;
* distinção entre falhas transitórias e permanentes;
* preservação de estados históricos válidos;
* proteção contra publicação tardia de versões obsoletas;
* distinção entre falha técnica, estado inválido, obsolescência e necessidade de revisão.

---

# 5. 6.2.8.4 — Recursos, Observabilidade e Continuidade

## 5.1. Saúde funcional ≠ disponibilidade técnica

A investigação rejeitou a ideia de que:

```text
HTTP 200 = Snoopy funcionando
```

O sistema pode possuir:

* API funcionando;
* banco funcionando;
* Tunnel funcionando;

e ainda assim estar funcionalmente degradado porque:

* não existem workers;
* a fila está acumulando jobs;
* embeddings estão falhando;
* existem representações incompletas;
* há incompatibilidade de versões.

Portanto, a V3 deverá distinguir **disponibilidade técnica** de **saúde funcional**.

---

## 5.2. Limites de recursos

A V3 deverá possuir limites explícitos para:

* tamanho de arquivo;
* concorrência;
* memória;
* espaço temporário;
* tempo máximo de processamento;
* chamadas externas;
* retries;
* tamanho de contexto.

Esses limites são operacionais e não devem ser confundidos com decisões metodológicas.

Por exemplo:

> um PDF exceder o limite de processamento não significa que o documento seja metodologicamente inválido.

---

## 5.3. Concorrência e backpressure

A concorrência do worker deverá ser configurável.

Não se deve simplesmente aumentar o número de workers porque há jobs pendentes.

Os gargalos podem estar em:

* RAM;
* CPU;
* disco;
* PostgreSQL;
* serviços externos.

A fila também precisa ser observável quanto à pressão acumulada.

Indicadores importantes incluem:

* número de jobs pendentes;
* idade do job pendente mais antigo;
* taxa de processamento;
* taxa de falhas;
* duração média;
* percentis de processamento.

---

## 5.4. Observabilidade tecnológica

A solução investigada é composta.

### Supabase

Será a camada-base para:

* logs;
* métricas;
* diagnóstico do PostgreSQL e serviços;
* Reports;
* observabilidade da infraestrutura Supabase.

A Metrics API também oferece possibilidade futura de integração com ferramentas externas.

### PM2

Continua responsável pela supervisão dos processos locais:

* autorestart;
* restart delay;
* backoff;
* limite de memória;
* limite de reinícios;
* logs;
* inicialização após reboot.

### Cloudflare Tunnel

Possui monitoramento próprio.

Mas:

> Tunnel saudável ≠ aplicação saudável.

### Snoopy

Precisa produzir sua própria observabilidade de aplicação:

* estado dos workers;
* estado da fila;
* jobs;
* processamento;
* documentos;
* embeddings;
* retrieval;
* reranking;
* erros;
* latências;
* espaço em disco;
* dependências.

---

## 5.5. Health, readiness e diagnóstico

A investigação propôs três níveis conceituais:

```text
Liveness
→ processo está vivo?

Readiness
→ processo + dependências essenciais funcionam?

Details
→ qual componente está degradado?
```

Esses mecanismos não devem executar verificações excessivamente pesadas a cada requisição.

---

## 5.6. Logs e eventos

Devem ser distinguidos:

### Logs técnicos

Ex.:

* timeout;
* HTTP 500;
* erro de conexão;
* exceção.

### Eventos de domínio

Ex.:

```text
JOB_CLAIMED
JOB_RETRY
DOCUMENT_VERSION_CREATED
REPRESENTATION_PUBLISHED
EMBEDDING_FAILED
DOCUMENT_MARKED_OBSOLETE
```

Não foi considerada necessária a adoção de event sourcing completo.

A combinação:

```text
estado atual
+
histórico relevante
+
logs técnicos
+
provenance
```

é suficiente para a V3 investigada.

---

## 5.7. Alertas

A arquitetura não deve produzir centenas de alertas.

Os alertas devem representar condições que exigem intervenção, como:

* workers ausentes por período prolongado;
* fila envelhecendo excessivamente;
* taxa de falhas elevada;
* saturação do banco;
* CPU/I/O anormal;
* Tunnel degradado;
* espaço em disco baixo.

---

## 5.8. Continuidade e backup

A arquitetura física consolidada é:

```text
Google Drive
→ documento original

Servidor local
→ artefatos derivados persistentes

Supabase/PostgreSQL
→ estado lógico
```

Consequentemente:

> **backup do PostgreSQL não equivale a backup completo do Snoopy.**

Os artefatos físicos persistentes precisam de cópia independente.

A investigação estabeleceu como requisito que eles não tenham como única cópia o servidor que os produz.

O Google Drive continua sendo a fonte documental original, o que permite reconstrução dos derivados quando necessário, embora isso não elimine a necessidade de backup dos derivados devido ao custo de reprocessamento e às mudanças de ferramentas/modelos.

---

## 5.9. RPO, RTO e Disaster Recovery

A arquitetura deve reconhecer:

* **RPO:** quanto trabalho/dados pode ser perdido;
* **RTO:** quanto tempo pode ser necessário para recuperar.

Os valores numéricos ainda não foram definidos.

Também ficou estabelecido que a recuperação precisa considerar não apenas arquivos e banco, mas:

* versão do Snoopy;
* ferramentas;
* modelos;
* configurações;
* perfis de segmentação;
* configuração de embeddings.

Caso contrário, reprocessar o mesmo PDF no futuro pode produzir estado diferente do estado original.

Portanto, a proveniência definida na 6.2.7 também é componente da continuidade.

---

## 5.10. Reconstruibilidade da infraestrutura

Não foi considerado necessário adotar Terraform, Ansible ou infraestrutura como código formal na V3 inicial.

O requisito é mais fundamental:

> **o Snoopy não pode depender do conhecimento tácito de uma única máquina para ser reconstruído.**

Devem ser documentados/versionados, no mínimo:

* aplicação;
* workers;
* PM2;
* Cloudflare;
* configurações;
* diretórios persistentes;
* procedimentos de recuperação.

---

## 5.11. Tecnologias deliberadamente não adotadas inicialmente

A investigação não encontrou requisito que justifique, na V3 inicial:

* Kubernetes;
* Docker Swarm;
* Prometheus self-hosted;
* Grafana obrigatório;
* ELK/Loki;
* Datadog obrigatório;
* Sentry obrigatório;
* Terraform obrigatório;
* service mesh;
* múltiplos servidores;
* autoscaling;
* multi-region;
* failover sofisticado.

Essas tecnologias permanecem alternativas futuras condicionadas a necessidades reais de escala, disponibilidade ou diagnóstico.

---

## 5.12. Backup não é evidência de recuperação

Uma decisão conceitual importante para a futura validação:

> **backup é capacidade de recuperação; restore testado é evidência de recuperação.**

Na Etapa 6.5 deverá ser possível testar algo próximo de:

```text
backup
 ↓
restore
 ↓
estado coerente
 ↓
worker
 ↓
fila
 ↓
documentos
 ↓
retrieval
```

Isso segue o princípio transversal:

> **documentação declara capacidade; teste demonstra comportamento.**

---

# 6. Convergência arquitetural da 6.2.8

As quatro subetapas não produziram quatro soluções independentes.

Elas convergem para uma arquitetura operacional única:

```text
                    IDENTIDADE
                        │
                        ▼
               AUTORIZAÇÃO / CONTEXTO
                        │
                        ▼
                     ACERVO
                        │
                        ▼
                      JOB
                        │
                        ▼
                  PGMQ / QUEUE
                        │
                        ▼
                     WORKER
                        │
                        ▼
                   PROCESSAMENTO
                        │
                 ┌──────┴──────┐
                 │             │
              sucesso        falha
                 │             │
                 ▼             ▼
             validação     classificação
                 │             │
                 ▼        retry / review /
        publicação          terminal
         transacional
                 │
                 ▼
                ACTIVE
                 │
                 ▼
          OBSERVABILIDADE
                 │
                 ▼
       DIAGNÓSTICO / ALERTA
                 │
                 ▼
         BACKUP / RESTORE
                 │
                 ▼
            CONTINUIDADE
```

O eixo central é:

```text
segurança
   ↓
execução controlada
   ↓
integridade
   ↓
observabilidade
   ↓
recuperabilidade
```

---

# 7. Decisões tecnológicas consolidadas

A investigação chegou às seguintes decisões tecnológicas para a V3:

| Função                           | Direção consolidada                     |
| -------------------------------- | --------------------------------------- |
| Identidade                       | Supabase Auth                           |
| Autorização de domínio           | Backend                                 |
| Modelo conceitual de autorização | RBAC + ReBAC + ABAC contextual          |
| Enforcement adicional            | PostgreSQL/RLS                          |
| Estado lógico                    | PostgreSQL/Supabase                     |
| Fila                             | PGMQ / Supabase Queues                  |
| Workers                          | Python                                  |
| Processamento pesado             | Fora das transações                     |
| Publicação crítica               | Transacional no PostgreSQL              |
| Reentrega                        | Visibility timeout / PGMQ               |
| Idempotência                     | Governada pelo Snoopy                   |
| Supervisão local                 | PM2                                     |
| Exposição externa                | Cloudflare Tunnel                       |
| Logs de infraestrutura           | Supabase + PM2 + Cloudflare             |
| Observabilidade de aplicação     | métricas/eventos próprios do Snoopy     |
| Estado documental original       | Google Drive                            |
| Artefatos derivados persistentes | armazenamento local                     |
| Backup lógico                    | PostgreSQL/Supabase                     |
| Backup físico                    | cópia independente dos artefatos locais |
| Workflow engine externo          | não necessário inicialmente             |
| Broker Redis                     | não necessário inicialmente             |
| Orquestrador externo             | não necessário inicialmente             |

As decisões acima são **direções arquiteturais/tecnológicas consolidadas**, enquanto detalhes de implementação continuam deliberadamente reservados à 6.3.

---

# 8. O que foi deliberadamente deixado para a Etapa 6.3

A 6.2.8 não pretendeu produzir ainda o projeto técnico detalhado.

Ficam para 6.3, entre outros:

### Segurança

* esquema exato de relações de autorização;
* tabelas e constraints;
* representação de owner/reader/admin;
* política detalhada de público/privado/compartilhado;
* implementação concreta de RLS;
* middleware;
* funções de autorização;
* ROOT;
* claims/JWT;
* política por endpoint.

### Filas

* estrutura final de `jobs`;
* eventual estrutura de `attempts/runs`;
* formato da mensagem PGMQ;
* nomes das filas;
* visibility timeout;
* política de renovação;
* número de workers;
* concorrência;
* backoff;
* limites de retry;
* constraints de idempotência.

### Falhas

* taxonomia final dos estados;
* critérios exatos de retry;
* critérios de quarantine/revisão;
* invariantes implementadas como constraints/functions;
* estratégia de limpeza de artefatos parciais.

### Observabilidade

* métricas exatas;
* nomes dos eventos;
* thresholds;
* alertas;
* health endpoints concretos;
* retenção de logs;
* política operacional.

### Continuidade

* RPO;
* RTO;
* estratégia concreta de backup dos artefatos locais;
* periodicidade;
* retenção;
* procedimento operacional de restore;
* documentação/reconstrução do servidor.

A distinção foi preservada porque esses pontos pertencem ao **projeto técnico da solução**, e não à investigação de alternativas da 6.2.

---

# 9. Tecnologias e abordagens que não se mostraram necessárias

A investigação também produziu uma lista de complexidades que deliberadamente não serão introduzidas apenas por antecipação:

* sistema externo dedicado de autorização;
* Redis/BullMQ;
* workflow engine;
* Kubernetes;
* orquestração distribuída;
* observabilidade externa obrigatória;
* event sourcing completo;
* infraestrutura multi-node;
* autoscaling;
* multi-region;
* failover complexo;
* infraestrutura como código obrigatória.

Isso não significa que essas tecnologias sejam inadequadas em absoluto.

Significa que **nenhuma necessidade atual da arquitetura investigada justificou seu custo e sua complexidade para a V3 inicial**.

---

# 10. Princípios transversais resultantes

A Etapa 6.2.8 consolidou alguns princípios que atravessam todo o Snoopy V3:

### 1. Identidade não é autorização

Saber quem é o usuário não determina automaticamente o que ele pode acessar.

### 2. Identificador não é permissão

`folder_id`, `document_id`, `job_id` etc. são referências, não provas de autorização.

### 3. Capacidade não é permissão

Um componente tecnicamente capaz de executar uma operação não deve fazê-lo fora do contexto autorizado.

### 4. Fila não é estado de domínio

PGMQ entrega trabalho; `jobs` representa o trabalho e seu estado.

### 5. Reentrega deve ser esperada

A arquitetura deve suportar execução repetida sem corromper o estado.

### 6. Falha deve ser explícita

Falha não pode ser confundida com sucesso parcial.

### 7. Estado parcial não deve ser publicado

A publicação de um estado válido deve ser atômica.

### 8. Estado antigo não pode sobrescrever estado novo

A execução deve estar vinculada à versão documental que pretende produzir.

### 9. Observabilidade deve enxergar o domínio

Não basta saber que a API responde; é necessário saber se o Snoopy está efetivamente processando e recuperando documentos.

### 10. Backup não prova recuperação

Somente um restore testado demonstra capacidade real de recuperação.

### 11. Proveniência também sustenta continuidade

Reconstruir um estado anterior exige conhecer não apenas os arquivos, mas as condições sob as quais os derivados foram produzidos.

### 12. Complexidade deve ser justificada por requisito

Uma tecnologia não entra porque é sofisticada ou popular, mas porque satisfaz uma necessidade arquitetural que as alternativas mais simples não satisfazem.

---

# 11. Estado epistemológico da Etapa 6.2.8

A investigação não afirma que a implementação V3 será efetivamente segura, resiliente ou recuperável.

Ela estabelece **quais propriedades a arquitetura deverá possuir e quais mecanismos tecnológicos serão utilizados para tentar realizá-las**.

Portanto:

```text
6.2.8
→ estabelece arquitetura operacional e escolhas tecnológicas

6.3
→ transforma essas escolhas em projeto técnico concreto

6.4
→ implementa

6.5
→ verifica experimentalmente se o comportamento corresponde ao especificado
```

A distinção permanece:

> **documentação declara capacidade; teste demonstra comportamento.**

---

# 12. Síntese final da investigação

A Etapa 6.2.8 levou o Snoopy de uma visão de “aplicação com usuários, jobs e servidor” para uma arquitetura operacional na qual **identidade, autorização, execução, estado, falha, observabilidade e recuperação são partes de uma mesma cadeia de integridade**.

A segurança será contextual e orientada por relações entre identidades e recursos, utilizando papéis globais e condições contextuais quando necessários, com o backend como autoridade de domínio e PostgreSQL/RLS como camada adicional de enforcement.

O processamento assíncrono utilizará PGMQ/Supabase Queues como mecanismo de entrega, enquanto `jobs` continuará representando o estado de domínio e histórico do processamento. Workers serão idempotentes, a reentrega será esperada e a publicação válida precederá a remoção da mensagem.

Falhas deverão resultar em estados explícitos e recuperáveis. O Snoopy deverá falhar fechado: estados parcialmente processados não serão publicados como ativos. Transações PostgreSQL governarão as transições críticas, enquanto PGMQ permitirá reentrega e recuperação de execuções interrompidas.

A operação será monitorada por uma composição de Supabase, PM2, Cloudflare Tunnel e observabilidade própria do Snoopy. A saúde funcional deverá ser distinguida da simples disponibilidade técnica. Limites de recursos, concorrência e backpressure deverão ser explícitos e observáveis.

A continuidade será baseada na separação entre fonte documental original, estado lógico e artefatos físicos derivados. PostgreSQL/Supabase terá sua política de backup, enquanto artefatos locais persistentes terão cópia independente. A infraestrutura deverá ser reconstruível e o restore deverá ser posteriormente testado.

A arquitetura não adotará inicialmente infraestrutura adicional de grande porte que não tenha sido justificada pelos requisitos: Redis/BullMQ, workflow engines, Kubernetes, observabilidade externa obrigatória, multi-node, autoscaling, multi-region e mecanismos equivalentes permanecem alternativas futuras.

Em conjunto, a 6.2.8 estabelece uma arquitetura cujo princípio operacional pode ser resumido como:

> **autorizar corretamente → executar sob contexto controlado → preservar a integridade do estado → detectar degradação → recuperar sem perder a história.**
