# Registro Mestre da Análise — Snoopy-RAG

## Etapa 5.6 — Segurança, Isolamento e Operação

> **Status:** Especificação metodológica e arquitetural — não constitui implementação.
> **Função:** definir os requisitos necessários para que o Snoopy-RAG opere de forma segura, isolada, observável e recuperável, sem comprometer as propriedades metodológicas estabelecidas nas etapas anteriores.
> **Dependências:** Etapas 5.1, 5.2, 5.3, 5.4 e 5.5.
> **Próxima etapa:** alternativas técnicas, implementação e validação.

---

# 1. Objetivo da Etapa 5.6

As etapas anteriores trataram do documento, de suas representações, da segmentação, da recuperação, da rastreabilidade e da persistência.

A Etapa 5.6 trata de uma questão transversal:

> **Como garantir que essa arquitetura possa operar sem que segurança, concorrência, isolamento, falhas ou limitações operacionais destruam as propriedades documentais e metodológicas que acabamos de estabelecer?**

Não se trata simplesmente de “proteger o servidor”.

No Snoopy-RAG, segurança e operação possuem consequências metodológicas.

Se um usuário consegue consultar documentos que pertencem a outro acervo, por exemplo, o problema não é apenas de segurança: o corpus da pesquisa foi alterado indevidamente.

Se uma versão de desenvolvimento altera o acervo de produção, o problema não é apenas operacional: a reprodutibilidade da recuperação pode ser comprometida.

Se dois workers processam o mesmo documento simultaneamente e produzem estados incompatíveis, o problema não é apenas de concorrência: pode haver inconsistência entre documento, segmentação, embeddings e evidências.

Portanto:

> **Segurança, isolamento e operação são condições para preservar a integridade documental e metodológica do Snoopy-RAG.**

---

# 2. Contexto herdado das etapas anteriores

A arquitetura atual já possui mecanismos importantes de isolamento e processamento.

O sistema possui usuários, acervos, documentos, chunks, jobs e políticas de acesso no banco. A arquitetura também desacopla o processamento pesado da requisição HTTP por meio de jobs persistentes. 

O problema é que algumas dessas garantias ainda dependem de pressupostos externos ou de lógica implementada no backend.

Um exemplo particularmente importante é o uso de `service role`: o backend possui privilégios que não devem ser confundidos com autorização do usuário. A análise anterior já identificou que o backend precisa aplicar sua própria lógica de autorização e que existe um possível risco de IDOR em rotas que recebem `folder_id`, ainda não confirmado como exploração efetiva. 

Também foi identificado que desenvolvimento e produção compartilham atualmente o mesmo banco, enquanto os jobs não possuem uma identificação explícita de ambiente. O worker de desenvolvimento não participa normalmente do fluxo observado. Isso cria uma fragilidade de isolamento entre ambientes. 

Além disso, a API key utilizada pelo Google Picker não possui atualmente restrição configurada, e existem configurações OAuth antigas que permanecem no projeto. Esses pontos foram classificados como itens de hardening (fortalecimento de segurança), e não como prova de comprometimento do sistema. 

---

# 3. Segurança não pode ser tratada como camada isolada

Uma arquitetura documental possui diferentes fronteiras de segurança.

Podemos representá-las conceitualmente assim:

```text
USUÁRIO
   │
   ▼
IDENTIDADE
   │
   ▼
AUTORIZAÇÃO
   │
   ▼
ACERVO
   │
   ▼
DOCUMENTO
   │
   ▼
REPRESENTAÇÕES
   │
   ▼
RECUPERAÇÃO
   │
   ▼
EVIDÊNCIA
```

Cada nível possui uma pergunta diferente:

* **Quem é o usuário?**
* **O que esse usuário pode acessar?**
* **Qual acervo está sendo consultado?**
* **Quais documentos pertencem a esse acervo?**
* **Quais representações podem ser utilizadas?**
* **Quais resultados podem ser retornados?**
* **De onde veio a evidência?**

A segurança não deve depender apenas da interface.

Esconder um botão no frontend não constitui controle de acesso.

O backend deve verificar a autorização independentemente do que o cliente envia.

---

# 4. Identidade e autenticação

O Snoopy precisa distinguir claramente:

```text
identidade
    ≠
autorização
```

Autenticação responde:

> “Quem é este usuário?”

Autorização responde:

> “O que este usuário pode fazer?”

A existência de um `userId` válido não implica autorização para acessar qualquer `folder_id`.

Da mesma forma:

```text
user A
```

não deve significar:

```text
acesso a todos os documentos
```

mas sim algo conceitualmente próximo de:

```text
user A
   ↓
acervos autorizados
   ↓
documentos desses acervos
   ↓
representações correspondentes
```

---

# 5. Isolamento por acervo

O acervo é uma fronteira metodológica e de segurança.

Nas etapas anteriores, já foi estabelecido que o corpus não é apenas um conjunto técnico de documentos. Ele faz parte das condições da recuperação.

Portanto:

> **Não misturar acervos é simultaneamente requisito de segurança e requisito de reprodutibilidade.**

Se dois pesquisadores possuem:

```text
Acervo A
```

e

```text
Acervo B
```

uma consulta idêntica não deve retornar resultados de ambos apenas porque os documentos estão armazenados no mesmo banco.

A fronteira lógica deve ser preservada em:

* recuperação;
* persistência;
* cache;
* jobs;
* documentos;
* chunks;
* embeddings;
* histórico;
* leitura;
* síntese;
* evidências.

---

# 6. Autorização no backend

A autorização deve ser considerada uma propriedade do domínio, e não uma responsabilidade do frontend.

Conceitualmente:

```text
requisição
    ↓
identidade
    ↓
autorização
    ↓
verificação do recurso
    ↓
operação
```

Uma requisição que contenha:

```text
folder_id = X
```

não deve ser suficiente para acessar X.

O sistema deve verificar se:

```text
usuário → autorizado → folder X
```

antes de executar operações relacionadas ao acervo.

O mesmo princípio deve ser aplicado a:

* documentos;
* jobs;
* resultados;
* evidências;
* arquivos;
* operações administrativas.

---

# 7. Administração não é propriedade

Essa distinção merece ser preservada explicitamente.

Um usuário pode possuir capacidade administrativa sobre um acervo sem ser seu “dono” documental no sentido comum.

Da mesma forma, um acervo público não deve ser representado simplesmente como:

```text
user_id = NULL
```

e tratado automaticamente como “qualquer coisa pode acessar”.

A análise atual já identificou esse problema conceitual na pasta pública do GEPAFOR: `user_id NULL` representa ausência de propriedade individual, enquanto `drive_id NULL` indica que a pasta lógica ainda não está ligada a uma pasta real do Drive. O UUID da pasta pública também está atualmente hardcoded. 

A arquitetura futura deve separar conceitualmente:

```text
propriedade
administração
visibilidade
permissão
origem
```

---

# 8. Acervos públicos e privados

O Snoopy precisa suportar, conceitualmente, diferentes regimes de acesso:

```text
ACERVO PRIVADO
    ↓
usuários autorizados

ACERVO COMPARTILHADO
    ↓
grupo autorizado

ACERVO PÚBLICO
    ↓
qualquer usuário autorizado pela política do sistema
```

Isso é diferente de codificar a regra como:

```text
user_id IS NULL
```

A representação técnica deve refletir a regra de negócio.

O objetivo é impedir que uma convenção de banco de dados seja confundida com uma política de autorização.

---

# 9. Ambiente de desenvolvimento e produção

A separação entre desenvolvimento e produção é particularmente importante no Snoopy porque ambos podem operar sobre o mesmo tipo de informação documental.

O estado observado atualmente é:

```text
PRODUÇÃO ─┐
          ├── Supabase compartilhado
DESENV. ──┘
```

com jobs sem identificação explícita de ambiente. 

Isso significa que uma operação de desenvolvimento pode, potencialmente, afetar estado utilizado pela produção.

A arquitetura desejada deve estabelecer uma fronteira explícita:

```text
PRODUÇÃO
   │
   ├── banco/estado de produção
   ├── jobs de produção
   └── workers de produção

DESENVOLVIMENTO
   │
   ├── banco/estado de desenvolvimento
   ├── jobs de desenvolvimento
   └── workers de desenvolvimento
```

Caso algum recurso precise ser compartilhado, esse compartilhamento deve ser explícito e controlado.

---

# 10. O problema dos jobs

A fila é parte importante da arquitetura documental.

O fluxo atual já possui estados conceituais:

```text
pending
   ↓
processing
   ↓
completed / failed
```

e os jobs são persistidos. 

Entretanto, uma fila persistente não é automaticamente uma fila concorrente segura.

É necessário responder:

> **O que acontece quando dois workers tentam processar o mesmo job simultaneamente?**

---

# 11. Concorrência

Uma operação conceitualmente ingênua poderia ser:

```text
Worker A → procura pending
Worker B → procura pending

Worker A → encontra Job X
Worker B → encontra Job X

Worker A → processa X
Worker B → processa X
```

O resultado pode ser:

* processamento duplicado;
* documentos duplicados;
* chunks duplicados;
* embeddings duplicados;
* estados inconsistentes;
* competição por arquivos temporários;
* consumo desnecessário de recursos;
* resultados diferentes para o mesmo documento.

A arquitetura precisa possuir um mecanismo conceitual de **reserva/claim atômico do job**.

Não importa, nesta etapa, qual tecnologia realizará isso.

O requisito é:

> **Um job destinado a um único processamento não pode ser simultaneamente assumido por múltiplos workers como se fosse um trabalho livre.**

---

# 12. Idempotência

Concorrência não é o único problema.

Falhas podem ocorrer depois de uma parte do processamento:

```text
extração ✓
limpeza ✓
metadados ✓
chunking ✓
embeddings ✗
```

O worker pode precisar tentar novamente.

A operação precisa ser **idempotente** (repeti-la não produz efeitos indevidos adicionais).

Idealmente:

```text
processar documento X
        ↓
estado consistente
```

e:

```text
processar documento X novamente
        ↓
mesmo estado lógico
```

sem produzir múltiplas cópias incompatíveis.

Isso se conecta diretamente à Etapa 5.5: consistência e idempotência são necessárias para que falhas e reprocessamentos não produzam estados silenciosamente divergentes.

---

# 13. Falha parcial

O sistema não deve publicar como ativo um documento cujo processamento terminou apenas parcialmente.

Por exemplo:

```text
documento ✓
representação ✓
segmentação ✓
embeddings ✗
```

não deve aparecer para o mecanismo de recuperação como se fosse:

```text
documento completamente disponível
```

A arquitetura deve distinguir:

```text
processado parcialmente
```

de:

```text
processado e disponível para recuperação
```

Esse requisito é particularmente importante porque a ausência de embeddings, chunks ou metadados pode não gerar um erro visível ao usuário; simplesmente produzirá um corpus incompleto.

---

# 14. Publicação transacional do estado documental

A ideia estabelecida na Etapa 5.5 pode ser aprofundada aqui.

Conceitualmente:

```text
                 PROCESSAMENTO
                      │
                      ▼
              estado provisório
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
     representação segmentação embeddings
          │           │           │
          └───────────┼───────────┘
                      ▼
                 validação
                      │
                      ▼
                   PUBLICAR
                      │
                      ▼
                  estado ativo
```

A recuperação deve enxergar apenas estados considerados válidos.

Assim, o sistema evita o cenário:

```text
documento novo
+
chunks antigos
+
embeddings antigos
```

que seria particularmente perigoso porque pode funcionar aparentemente bem.

---

# 15. Gestão de credenciais e segredos

Credenciais devem ser tratadas como infraestrutura sensível.

Isso inclui, conceitualmente:

* chaves de banco;
* service role;
* credenciais OAuth;
* tokens do Google;
* chaves de APIs externas;
* segredos de infraestrutura;
* credenciais de serviços.

O princípio fundamental é:

> **Segredo necessário apenas ao backend não deve chegar ao cliente.**

A existência de `.env` local não transforma automaticamente um segredo em seguro; é necessário controlar onde ele é carregado, utilizado, registrado e exposto.

---

# 16. Tokens e integração com Google

O Google Drive é uma dependência externa importante do pipeline documental.

A arquitetura deve distinguir:

```text
credencial do usuário
        ≠
credencial do sistema
        ≠
referência ao arquivo
```

O `drive_file_id`, por exemplo, é identificador de um recurso. Ele não deve ser tratado como se fosse uma credencial.

Da mesma forma, tokens precisam possuir:

* ciclo de vida definido;
* armazenamento apropriado;
* renovação;
* revogação;
* tratamento de expiração;
* tratamento de falha de autorização.

O objetivo é impedir que uma falha na integração externa produza estados documentais incorretos ou que credenciais sejam expostas desnecessariamente.

---

# 17. Google Picker

A análise atual identificou que a API key utilizada pelo Picker não possui restrição configurada. Isso deve ser tratado como requisito de hardening, não como evidência de exploração. 

O requisito arquitetural é:

> **Credenciais utilizadas por componentes client-side devem possuir escopo e restrições compatíveis com sua finalidade.**

Isso significa minimizar:

```text
quem pode usar
onde pode usar
o que pode acessar
```

sem confundir a chave do Picker com autorização para acessar o acervo interno do Snoopy.

---

# 18. Proteção da service role

O uso de privilégios elevados no backend pode ser necessário para algumas operações.

Porém:

```text
service role
      ≠
usuário autorizado
```

A arquitetura deve estabelecer uma fronteira clara:

```text
CLIENTE
   ↓
autenticação
   ↓
autorização da aplicação
   ↓
BACKEND
   ↓
credencial privilegiada
   ↓
BANCO
```

O banco não deve ser considerado o único responsável pela autorização quando o backend utiliza uma credencial que pode ultrapassar as políticas normais de acesso.

---

# 19. Logs

Uma infraestrutura de pesquisa precisa ser observável.

Porém, logging excessivo também pode criar problemas de segurança.

O sistema precisa registrar informações suficientes para responder:

* o que aconteceu?
* quando aconteceu?
* com qual documento?
* em qual job?
* em qual ambiente?
* em qual versão?
* com qual resultado?
* houve falha?
* onde ocorreu?

Mas deve evitar registrar desnecessariamente:

* tokens;
* credenciais;
* segredos;
* conteúdo documental sensível;
* informações pessoais não necessárias.

Portanto:

> **Observabilidade deve aumentar a capacidade de diagnóstico sem transformar o log em uma cópia insegura do acervo.**

---

# 20. Observabilidade do pipeline

A operação ideal precisa permitir acompanhar o caminho:

```text
job
 ↓
documento
 ↓
representação
 ↓
segmentação
 ↓
embeddings
 ↓
estado publicado
```

e identificar em qual estágio ocorreu uma falha.

Isso é particularmente importante porque o pipeline é assíncrono.

Uma requisição HTTP pode simplesmente gerar:

```text
job criado
```

e o problema surgir vários minutos depois no worker.

Portanto, a arquitetura precisa permitir distinguir:

```text
requisição aceita
```

de:

```text
processamento concluído
```

e:

```text
documento efetivamente disponível para recuperação
```

---

# 21. Estado observado × estado desejado

O sistema deve distinguir:

```text
estado que deveria existir
```

de:

```text
estado que efetivamente existe
```

Exemplo:

```text
Job = completed
```

não deveria ser interpretado automaticamente como:

```text
documento = consistente
```

sem que as dependências necessárias estejam presentes.

A Etapa 5.5 já estabeleceu que a publicação precisa representar um estado documental coerente. A Etapa 5.6 acrescenta que a operação deve ser capaz de detectar quando essa expectativa não foi cumprida.

---

# 22. Recuperação diante de falhas

Um sistema operacionalmente robusto precisa responder:

> **O que acontece quando alguma coisa quebra?**

As falhas possíveis incluem:

* API externa indisponível;
* Google Drive indisponível;
* banco indisponível;
* worker encerrado;
* processo interrompido;
* documento inválido;
* OCR falhando;
* geração de embeddings falhando;
* timeout;
* quota externa atingida;
* rede interrompida;
* atualização concorrente;
* corrupção de estado.

Cada falha não precisa necessariamente de um tratamento diferente na interface, mas precisa resultar em um estado conhecido.

---

# 23. Recuperabilidade

Recuperabilidade não significa necessariamente manter tudo para sempre.

Significa que o sistema deve possuir um caminho conhecido para voltar a um estado operacional consistente.

Conceitualmente:

```text
falha
 ↓
detecção
 ↓
estado conhecido
 ↓
retry / rollback / reprocessamento
 ↓
validação
 ↓
publicação
```

O pior cenário não é:

```text
o sistema parou
```

mas:

```text
o sistema continuou funcionando
mas ninguém sabe se seus dados continuam coerentes.
```

---

# 24. Atualização e remoção

A integração com uma fonte externa como o Google Drive exige tratamento para:

```text
arquivo criado
arquivo modificado
arquivo substituído
arquivo movido
arquivo removido
arquivo inacessível
```

A Etapa 5.5 já estabeleceu que alterações precisam produzir estados distinguíveis e invalidar derivados incompatíveis.

A Etapa 5.6 acrescenta a necessidade operacional de que essas transições sejam detectáveis e tratadas de forma segura.

---

# 25. Limites de recursos

O worker não deve ser considerado uma máquina de capacidade infinita.

É necessário considerar limites de:

* memória;
* CPU;
* armazenamento temporário;
* tamanho de PDF;
* número de páginas;
* tempo de processamento;
* chamadas externas;
* geração de embeddings;
* concorrência;
* tamanho de filas.

Isso é especialmente importante porque documentos científicos podem variar enormemente em complexidade.

Um PDF pequeno e textual não possui o mesmo custo operacional de um PDF grande, escaneado e repleto de imagens.

---

# 26. Isolamento de arquivos temporários

Atualmente o PDF baixado para processamento é material operacional temporário; a análise arquitetural anterior identificou `data/raw_pdfs/{drive_file_id}.pdf` como esse estado transitório, enquanto documentos, chunks e embeddings permanecem persistidos. 

O diretório temporário deve ser tratado como espaço operacional, não como acervo permanente.

Isso implica:

* evitar colisão entre jobs;
* evitar exposição HTTP acidental;
* limpar arquivos abandonados;
* impedir acesso entre usuários;
* evitar retenção desnecessária;
* controlar tamanho acumulado.

---

# 27. Worker e API

A separação entre:

```text
API
```

e:

```text
worker
```

é arquiteturalmente adequada ao processamento assíncrono.

Porém, a operação precisa tornar claro:

```text
API disponível
```

não significa necessariamente:

```text
worker disponível
```

Um sistema pode responder normalmente às requisições enquanto a fila deixa de ser processada.

Portanto, a saúde do sistema deve considerar pelo menos:

```text
API
Banco
Fila
Worker
Dependências externas
```

---

# 28. Saúde do sistema

A operação precisa possuir indicadores que permitam saber se o sistema está realmente funcional.

Exemplos conceituais:

```text
API:          saudável
Banco:        saudável
Worker:       ativo
Fila:         processando
Google:       acessível
Armazenamento: disponível
```

Não é necessário definir aqui a tecnologia de monitoramento.

O requisito é que uma falha relevante não permaneça invisível.

---

# 29. Backpressure e crescimento da fila

Se a velocidade de entrada de documentos superar a velocidade de processamento:

```text
entrada > processamento
```

a fila cresce.

Isso pode gerar:

* aumento de latência;
* consumo de armazenamento;
* excesso de chamadas externas;
* saturação do worker;
* falhas em cascata.

A arquitetura deve ser capaz de representar esse estado e impor limites quando necessário.

---

# 30. Rate limiting

O sistema também precisa considerar a possibilidade de uso excessivo da API.

Isso possui duas dimensões:

### Proteção operacional

Impedir que uma quantidade excessiva de requisições derrube o serviço.

### Proteção econômica

Impedir que chamadas excessivas a modelos ou serviços externos produzam consumo inesperado.

O controle deve respeitar a natureza do Snoopy: uma busca e uma sincronização de acervo não possuem o mesmo custo.

---

# 31. Cache e segurança

O cache já foi identificado anteriormente como uma possível fonte de mascaramento em testes.

Ele também precisa ser considerado uma fronteira de segurança.

Uma resposta armazenada para:

```text
usuário A + acervo X
```

não pode aparecer para:

```text
usuário B + acervo Y
```

A chave de cache deve incorporar as dimensões que determinam sua validade.

Além disso:

```text
mudança de corpus
mudança de configuração
mudança de versão documental
mudança de retrieval profile
```

pode invalidar uma resposta previamente armazenada.

A análise anterior já observou que o cache atual considera usuário e acervo na chave, mas sua persistência e invalidação continuam sendo pontos a especificar. 

---

# 32. Segurança e rastreabilidade

Segurança não deve destruir a rastreabilidade.

O sistema precisa continuar conseguindo responder:

```text
quem executou?
quando?
sobre qual acervo?
sobre qual versão documental?
com qual configuração?
qual evidência foi recuperada?
```

sem transformar essas informações em exposição indevida de dados.

Isso cria uma tensão importante:

```text
AUDITABILIDADE
      ↕
PRIVACIDADE
```

A arquitetura deve preservar ambas.

---

# 33. Auditoria de operações

Além da proveniência documental definida na Etapa 5.4, existe uma segunda camada:

> **proveniência operacional.**

Ela responde perguntas como:

* quem iniciou a sincronização?
* qual job processou o documento?
* quando ele falhou?
* quando foi reprocessado?
* quem alterou uma configuração?
* quando uma versão foi publicada?
* quando um documento deixou de estar ativo?

Isso não é a mesma coisa que proveniência metodológica.

Temos, portanto:

```text
PROVENIÊNCIA DOCUMENTAL
onde veio a evidência?

PROVENIÊNCIA METODOLÓGICA
por que ela foi considerada relevante?

PROVENIÊNCIA OPERACIONAL
como e quando o sistema produziu esse estado?
```

As três devem permanecer conceitualmente separadas.

---

# 34. Privacidade e retenção

O Snoopy trabalha com documentos potencialmente pertencentes a pesquisadores, grupos de pesquisa ou instituições.

Portanto, a arquitetura deve definir, posteriormente:

* o que é armazenado;
* por quanto tempo;
* quem pode acessar;
* o que é apagado;
* o que permanece para auditoria;
* o que acontece quando um usuário é removido;
* o que acontece quando um acervo é excluído.

Não cabe à Etapa 5.6 estabelecer uma política jurídica específica.

Cabe estabelecer o requisito:

> **A retenção de dados deve ser deliberada, identificável e compatível com a finalidade do sistema.**

---

# 35. Princípio do menor privilégio

Cada componente deve possuir somente os privilégios necessários à sua função.

Conceitualmente:

```text
frontend
→ mínimo necessário

API
→ operações necessárias

worker
→ processamento necessário

banco
→ persistência necessária

integrações externas
→ escopo necessário
```

Isso reduz o impacto de uma eventual falha.

O princípio é particularmente importante porque o backend atualmente utiliza uma credencial privilegiada; portanto, quanto maior o privilégio técnico, maior a necessidade de controle lógico anterior. 

---

# 36. Defesa em profundidade

Nenhuma única camada deve ser considerada suficiente.

Por exemplo:

```text
Autenticação
      +
Autorização
      +
Isolamento de acervo
      +
RLS / controle de banco
      +
Validação de recursos
      +
Logs
      +
Auditoria
```

formam uma defesa mais robusta do que qualquer mecanismo isolado.

O objetivo não é pressupor que todos os mecanismos terão a mesma responsabilidade.

É evitar:

> **“Se esta única verificação falhar, todo o sistema fica exposto.”**

---

# 37. Requisitos de Segurança e Operação

A partir da análise, podem ser consolidados os seguintes requisitos.

### RSO1 — Identidade

Cada operação relevante deve possuir uma identidade operacional identificável.

### RSO2 — Autenticação

O sistema deve distinguir usuários autenticados de requisições não autenticadas.

### RSO3 — Autorização

A autenticação não deve ser considerada autorização.

### RSO4 — Autorização por recurso

Acesso a acervos, documentos, jobs e evidências deve ser verificado no backend.

### RSO5 — Isolamento de acervo

Documentos de diferentes acervos não devem ser misturados indevidamente.

### RSO6 — Isolamento entre usuários

Usuários não devem acessar recursos privados uns dos outros.

### RSO7 — Separação de propriedade e administração

Permissões administrativas não devem ser confundidas com propriedade documental.

### RSO8 — Política explícita de acervos públicos

Acesso público deve ser uma política explícita, não apenas uma consequência de `user_id NULL`.

### RSO9 — Isolamento de ambientes

Produção e desenvolvimento devem possuir fronteiras explícitas.

### RSO10 — Isolamento de jobs

Jobs devem possuir contexto operacional suficiente para impedir processamento indevido entre ambientes.

### RSO11 — Concorrência

Um job não deve ser assumido simultaneamente por múltiplos workers de maneira incompatível.

### RSO12 — Idempotência

Reprocessamentos e retries não devem produzir duplicação ou inconsistência silenciosa.

### RSO13 — Falha parcial

Estados incompletos não devem ser publicados como documentos plenamente disponíveis.

### RSO14 — Publicação coerente

Um estado documental só deve tornar-se ativo quando suas dependências necessárias estiverem consistentes.

### RSO15 — Segredos

Credenciais e tokens devem permanecer fora de superfícies não autorizadas.

### RSO16 — Escopo de credenciais

Credenciais externas devem possuir escopo compatível com sua finalidade.

### RSO17 — Tokens

Tokens devem possuir ciclo de vida controlado.

### RSO18 — Arquivos temporários

Arquivos temporários devem ser isolados, controlados e eliminados quando não forem mais necessários.

### RSO19 — Observabilidade

Falhas relevantes devem ser detectáveis.

### RSO20 — Diagnóstico

O sistema deve permitir identificar em qual estágio uma operação falhou.

### RSO21 — Logs

Logs devem registrar informação operacional suficiente sem expor dados desnecessários.

### RSO22 — Auditoria

Operações relevantes devem possuir histórico operacional suficiente para reconstrução.

### RSO23 — Recuperabilidade

Falhas devem produzir caminhos conhecidos de retry, recuperação ou reprocessamento.

### RSO24 — Limites de recursos

O sistema deve possuir limites para documentos, processamento, armazenamento e chamadas externas.

### RSO25 — Controle de carga

O sistema deve conseguir lidar com crescimento da fila sem degradação silenciosa.

### RSO26 — Rate limiting

Uso excessivo deve poder ser limitado de maneira compatível com o custo da operação.

### RSO27 — Cache seguro

Resultados armazenados não podem atravessar fronteiras de usuário ou acervo.

### RSO28 — Invalidação de cache

Mudanças incompatíveis devem invalidar estados derivados.

### RSO29 — Privacidade

Retenção e acesso aos dados devem ser deliberados e controláveis.

### RSO30 — Menor privilégio

Cada componente deve possuir apenas os privilégios necessários à sua função.

### RSO31 — Defesa em profundidade

Nenhum mecanismo isolado deve ser considerado a única barreira de segurança.

### RSO32 — Integridade metodológica

Mecanismos de segurança e operação não devem permitir alterações silenciosas no corpus utilizado pela recuperação.

### RSO33 — Integridade temporal

Operações devem manter identificável o estado documental e computacional sobre o qual foram executadas.

### RSO34 — Ambiente operacional

A saúde do sistema deve considerar API, banco, fila, worker e dependências externas.

### RSO35 — Recuperação consistente

Após falhas, o sistema deve retornar a um estado documental conhecido e consistente.

---

# 38. Modelo operacional conceitual

A arquitetura operacional pode ser representada como:

```text
                    USUÁRIO
                       │
                       ▼
                 AUTENTICAÇÃO
                       │
                       ▼
                 AUTORIZAÇÃO
                       │
                       ▼
                ┌─────────────┐
                │   ACERVO    │
                └──────┬──────┘
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
        DOCUMENTOS             BUSCA
             │                   │
             ▼                   ▼
           JOBS             RECUPERAÇÃO
             │                   │
             ▼                   ▼
          WORKER             EVIDÊNCIA
             │                   │
             └─────────┬─────────┘
                       ▼
                 ESTADO VALIDADO
                       │
                       ▼
                    AUDITORIA
                       │
                       ▼
                 FONTE ORIGINAL
```

Esse modelo acrescenta uma dimensão à arquitetura anteriormente estabelecida:

```text
documento
    ↓
representação
    ↓
segmentação
    ↓
recuperação
```

passa a operar dentro de:

```text
identidade
→ autorização
→ isolamento
→ processamento
→ validação
→ publicação
→ observabilidade
→ recuperação
```

---

# 39. Relação com as Etapas 5.1–5.5

A Etapa 5.6 não substitui as anteriores.

Ela garante suas condições operacionais.

### 5.1 — Representação canônica

Precisa ser protegida contra acesso ou alteração indevidos.

### 5.2 — Segmentação

Precisa ser produzida no contexto documental correto.

### 5.3 — Recuperação

Precisa respeitar usuário, acervo, versão e configuração.

### 5.4 — Proveniência

Precisa permanecer preservada durante as operações e reprocessamentos.

### 5.5 — Persistência e versionamento

Precisa sobreviver a falhas, concorrência, atualização e recuperação.

### 5.6 — Segurança e operação

Mantém todas essas propriedades funcionando como um sistema real.

---

# 40. O problema mais perigoso

Assim como na Etapa 5.5, o maior risco não é necessariamente uma falha explícita.

É uma falha **silenciosa**.

Exemplos:

```text
usuário A vê documento de B
```

é grave e relativamente evidente.

Mas:

```text
produção consulta chunks de desenvolvimento
```

ou:

```text
documento V2 utiliza embedding V1
```

ou:

```text
job duplicado gera duas versões incompatíveis
```

podem passar despercebidos.

O sistema continua respondendo.

Só que a resposta deixou de representar corretamente o corpus.

Por isso:

> **A segurança operacional do Snoopy deve ser avaliada não apenas pela capacidade de impedir acessos indevidos, mas pela capacidade de impedir estados documentais incorretos que pareçam válidos.**

---

# 41. Critérios de avaliação

A futura implementação da arquitetura deve poder ser testada por categorias.

## Segurança

* usuário A não acessa acervo privado de B;
* usuário não autorizado não acessa documento diretamente;
* `folder_id` arbitrário não contorna autorização;
* acervo público segue política explícita;
* credenciais não são expostas.

## Isolamento

* produção não interfere em desenvolvimento;
* desenvolvimento não interfere em produção;
* jobs possuem contexto de ambiente;
* cache não atravessa acervos;
* resultados não atravessam usuários.

## Concorrência

* dois workers não processam o mesmo job indevidamente;
* retry não duplica estado;
* falha parcial não publica documento incompleto.

## Recuperação

* worker pode ser reiniciado;
* jobs interrompidos podem ser recuperados;
* documentos parcialmente processados podem ser reprocessados;
* estados inválidos podem ser identificados.

## Observabilidade

* falhas são detectáveis;
* jobs podem ser rastreados;
* documentos podem ser relacionados a seus processamentos;
* eventos importantes possuem histórico operacional.

## Integridade metodológica

* corpus permanece corretamente delimitado;
* evidência continua associada ao acervo correto;
* versão documental permanece identificável;
* alterações operacionais não modificam silenciosamente as condições da recuperação.

---

# 42. Testes adversariais

Assim como os PDFs da Etapa 5.1 precisam de casos adversariais, a operação também precisa.

Exemplos:

```text
usuário A → folder B
```

```text
worker A + worker B → mesmo job
```

```text
worker interrompido → processamento parcial
```

```text
produção + desenvolvimento → mesmo banco
```

```text
cache A → consulta B
```

```text
token expirado → sincronização
```

```text
Drive indisponível → job
```

```text
documento removido durante processamento
```

```text
documento atualizado durante processamento
```

```text
grande volume de jobs simultâneos
```

O objetivo não é provar que “não existem bugs”.

É descobrir se o sistema possui estados de falha conhecidos e recuperáveis.

---

# 43. Limites epistemológicos

A segurança operacional também possui limites.

Não se deve afirmar:

* que nenhum acesso indevido é possível;
* que o sistema é invulnerável;
* que qualquer falha será automaticamente recuperada;
* que produção e desenvolvimento estão isolados se isso não tiver sido testado;
* que a autorização está correta apenas porque existe RLS;
* que a existência de logs significa auditabilidade completa;
* que backup implica recuperação validada;
* que uma operação idempotente é automaticamente segura;
* que um sistema funcional é necessariamente consistente.

Assim como nas etapas metodológicas anteriores:

> **propriedade arquitetural não testada é requisito, não evidência de funcionamento.**

---

# 44. Princípios consolidados da Etapa 5.6

Podemos condensar a etapa em alguns princípios.

### 1. Autenticação não é autorização

Saber quem é o usuário não determina automaticamente o que ele pode acessar.

### 2. Acervo é fronteira de segurança

O corpus de uma pesquisa deve permanecer delimitado.

### 3. Produção e desenvolvimento não devem compartilhar estado silenciosamente

Ambientes distintos devem possuir fronteiras explícitas.

### 4. Job deve ser assumido de forma segura

Concorrência não pode produzir processamento duplicado ou incompatível.

### 5. Falha parcial não é sucesso

Estado incompleto não deve ser publicado como completo.

### 6. Privilégio técnico não substitui autorização

Uma credencial privilegiada do backend aumenta a responsabilidade da camada de autorização.

### 7. Operação precisa ser observável

Um sistema que não consegue dizer o que aconteceu não consegue garantir recuperação confiável.

### 8. Cache é estado derivado

Ele possui escopo, validade e condições de invalidação.

### 9. Segurança e metodologia estão conectadas

Um acesso indevido pode alterar o corpus da pesquisa.

### 10. Estado operacional precisa ser reconstruível

Após uma falha, deve ser possível determinar o que aconteceu e retornar a um estado consistente.

---

# 45. Arquitetura consolidada das Etapas 5.1–5.6

Com a conclusão da Etapa 5, podemos agora enxergar a arquitetura desejada de maneira mais completa:

```text
                    DOCUMENTO ORIGINAL
                           │
                           ▼
              REPRESENTAÇÃO CANÔNICA
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
         ESTRUTURA      TEXTO        ELEMENTOS
             │         PESQUISÁVEL   MULTIMODAIS
             └─────────────┬─────────────┘
                           ▼
               SEGMENTAÇÃO ESTRUTURAL
                           │
                           ▼
                SEGMENTAÇÃO SEMÂNTICA
                           │
                           ▼
                 UNIDADES DE RECUPERAÇÃO
                           │
                           ▼
                      EMBEDDINGS
                           │
                           ▼
                      RECUPERAÇÃO
                           │
                           ▼
                       EVIDÊNCIA
                           │
                           ▼
                       SÍNTESE
                           │
                           ▼
                     PESQUISADOR
                           │
                           ▼
                  INTERPRETAÇÃO
```

Tudo isso deve existir dentro de:

```text
┌───────────────────────────────────────────────┐
│ IDENTIDADE                                   │
│ AUTORIZAÇÃO                                  │
│ ISOLAMENTO DE ACERVO                         │
│ ISOLAMENTO DE AMBIENTE                       │
│ CONCORRÊNCIA                                 │
│ VERSIONAMENTO                                │
│ CONSISTÊNCIA                                 │
│ OBSERVABILIDADE                              │
│ AUDITORIA                                    │
│ RECUPERABILIDADE                             │
└───────────────────────────────────────────────┘
```

---

# 46. Formulação consolidada

A Etapa 5.6 pode ser sintetizada no seguinte requisito arquitetural:

> **O Snoopy-RAG deverá operar com controle explícito de identidade, autorização, isolamento de acervos e ambientes, concorrência, processamento, publicação, observabilidade e recuperação de falhas, preservando a integridade e a rastreabilidade dos estados documentais e computacionais. Operações assíncronas deverão ser resistentes a concorrência, repetição e falhas parciais; recursos e credenciais deverão possuir escopo compatível com suas funções; e o sistema deverá impedir que alterações operacionais, acessos indevidos ou estados inconsistentes modifiquem silenciosamente as condições do corpus, da recuperação ou da evidência apresentada ao pesquisador.**

---

# 47. Conclusão da Etapa 5

Com a 5.6, a especificação das lacunas e respostas arquiteturais fica completa.

A sequência construída ao longo da Etapa 5 é:

```text
5.1
PRESERVAR
    ↓
5.2
ESTRUTURAR E SEGMENTAR
    ↓
5.3
RECUPERAR
    ↓
5.4
RASTREAR
    ↓
5.5
PERSISTIR E VERSIONAR
    ↓
5.6
PROTEGER, OPERAR E RECUPERAR
```

Isso produz uma arquitetura conceitual muito diferente daquela que poderíamos obter simplesmente “melhorando o RAG”.

O objeto central deixa de ser:

```text
PDF → texto → chunk → embedding → resposta
```

e passa a ser:

```text
DOCUMENTO
   ↓
ESTADO DOCUMENTAL
   ↓
REPRESENTAÇÕES DERIVADAS
   ↓
RECUPERAÇÃO
   ↓
EVIDÊNCIA
   ↓
PESQUISADOR
```

mantendo, transversalmente:

```text
PROVENIÊNCIA
VERSIONAMENTO
CONSISTÊNCIA
SEGURANÇA
ISOLAMENTO
OBSERVABILIDADE
```

A conclusão mais importante da Etapa 5 é, portanto:

> **O problema do Snoopy-RAG não é mais simplesmente encontrar uma implementação melhor de RAG. É definir uma infraestrutura documental na qual transformação, recuperação, evidência, persistência e operação possam coexistir sem que uma camada destrua as propriedades das demais.**

E isso nos deixa exatamente no ponto previsto pelo nosso princípio de trabalho:

> **PROBLEMA METODOLÓGICO → REQUISITO → PRINCÍPIO ARQUITETURAL → ALTERNATIVAS TÉCNICAS → IMPLEMENTAÇÃO → TESTE EMPÍRICO**

As Etapas **5.1–5.6** definiram os requisitos e princípios. **Ainda não escolhemos tecnologias.** Essa separação é importante: agora podemos entrar na próxima fase sem fazer arquitetura guiada pela biblioteca da moda ou pelo que o código atual já consegue fazer.

