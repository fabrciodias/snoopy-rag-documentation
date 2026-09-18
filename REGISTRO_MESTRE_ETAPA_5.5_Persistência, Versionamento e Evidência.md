# Registro Mestre da Análise — Snoopy-RAG

## Etapa 5.5 — Persistência, Versionamento e Consistência

> **Status:** Especificação metodológica e arquitetural — não constitui implementação.
> **Função:** definir como o Snoopy deve manter documentos, representações derivadas, resultados e estados do processamento coerentes ao longo do tempo.
> **Dependências:** Etapas 5.1, 5.2, 5.3 e 5.4.
> **Próxima etapa:** 5.6 — Segurança e Operações.

---

# 1. Objetivo da Etapa 5.5

As etapas anteriores estabeleceram uma cadeia relativamente clara:

```text
5.1 → preservar
5.2 → estruturar e segmentar
5.3 → recuperar
5.4 → rastrear
```

A Etapa 5.5 acrescenta uma pergunta que até agora estava implícita:

> **Como garantir que aquilo que foi preservado, segmentado, recuperado e rastreado continue coerente quando o sistema muda ao longo do tempo?**

Essa é a questão da **persistência, versionamento e consistência**.

O problema não é simplesmente armazenar dados em um banco.

O problema é preservar relações entre estados diferentes do sistema:

```text
documento
   ↓
representação
   ↓
segmentação
   ↓
embedding
   ↓
recuperação
```

quando qualquer uma dessas coisas pode mudar.

Um documento pode ser atualizado no Google Drive. Uma nova representação pode ser produzida. O algoritmo de segmentação pode mudar. O modelo de embedding pode ser substituído. Um job pode falhar no meio do processamento. Um conjunto antigo de chunks pode permanecer no banco. Um cache pode continuar servindo resultados produzidos sob uma configuração anterior.

Assim, o problema central desta etapa é:

> **O Snoopy precisa distinguir estados documentais e computacionais diferentes e manter consistência entre eles.**

---

# 2. O problema do tempo

O Snoopy atual pode ser representado, em uma fotografia estática, por:

```text
Drive
 ↓
document
 ↓
chunks
 ↓
embeddings
```

Mas um sistema real não permanece estático.

Depois de algum tempo:

```text
Drive
 ↓
documento alterado
 ↓
???
```

O sistema precisa decidir o que acontece com:

* documento anterior;
* representação anterior;
* chunks antigos;
* embeddings antigos;
* resultados de busca anteriores;
* cache;
* referências de evidência;
* histórico de processamento.

Portanto, o objeto central não é apenas:

```text
DOCUMENTO
```

mas:

```text
DOCUMENTO
   ├── estado/versão A
   ├── estado/versão B
   └── estado/versão C
```

A arquitetura precisa saber qual dessas versões está vigente e quais derivados pertencem a cada uma.

---

# 3. O estado atual já possui persistência, mas não versionamento formal

A arquitetura atual já possui uma propriedade importante: documentos, chunks, embeddings e jobs são persistidos no Supabase. O PDF físico, por outro lado, é utilizado como matéria-prima temporária em `data/raw_pdfs/{drive_file_id}.pdf`, não como cópia permanente do documento.  

Isso significa que existe:

```text
estado persistente
```

mas não necessariamente:

```text
histórico formal de estados
```

Essa diferença é fundamental.

Persistência responde:

> “Onde está o estado atual?”

Versionamento responde:

> “Qual estado era esse quando determinado resultado foi produzido?”

Consistência responde:

> “Os objetos relacionados pertencem ao mesmo estado documental e computacional?”

---

# 4. O documento deve possuir identidade temporal

Na Etapa 5.4 foi estabelecido que a proveniência precisa conseguir distinguir versões relevantes.

Isso leva a uma consequência direta:

```text
documento
```

não deve ser entendido apenas como um objeto permanente cujo conteúdo pode ser substituído silenciosamente.

Conceitualmente:

```text
DOCUMENTO
   │
   ├── VERSÃO 1
   │      ├── representação
   │      ├── segmentação
   │      └── embeddings
   │
   ├── VERSÃO 2
   │      ├── representação
   │      ├── segmentação
   │      └── embeddings
   │
   └── VERSÃO 3
          ├── representação
          ├── segmentação
          └── embeddings
```

Isso não implica que todas as versões precisem necessariamente ser mantidas indefinidamente.

A questão é que o sistema deve possuir uma **política explícita de identidade e substituição**.

Não pode simplesmente sobrescrever:

```text
documento X
```

e deixar:

```text
chunks antigos
embeddings antigos
resultados antigos
```

sem saber a qual estado pertencem.

---

# 5. O hash como identidade do conteúdo

O pipeline atual calcula um hash do documento durante o processamento. Na análise anterior, esse hash foi entendido como mecanismo de identificação binária do arquivo dentro do contexto atual, e não como deduplicação semântica.

Essa distinção deve permanecer.

Um hash pode responder:

> “O conteúdo binário deste arquivo é igual ao conteúdo binário daquele arquivo?”

Mas não responde:

> “Esses dois documentos são semanticamente o mesmo trabalho?”

Nem:

> “Esses dois arquivos são metodologicamente equivalentes?”

Portanto:

```text
HASH
 ↓
identidade/integridade do conteúdo
```

não:

```text
HASH
 ↓
identidade metodológica
```

Na arquitetura futura, a identidade documental pode utilizar esse tipo de mecanismo como parte de sua estratégia, mas a semântica da identidade deve ser explicitamente definida.

---

# 6. Documento e seus derivados formam uma unidade de consistência

Uma representação canônica, uma segmentação e um embedding não são objetos independentes.

Existe uma relação:

```text
representação R1
   ↓
segmentação S1
   ↓
embedding E1
```

Se a representação mudar:

```text
representação R2
```

não é seguro assumir que:

```text
S1
E1
```

continuam correspondendo a ela.

Da mesma maneira, se a segmentação mudar:

```text
S2
```

os embeddings produzidos sobre `S1` não representam necessariamente as unidades de `S2`.

Portanto, a arquitetura deve preservar a noção de **compatibilidade entre versões derivadas**.

---

# 7. A regra fundamental de consistência

Pode-se formular a regra:

> **Nenhuma representação derivada deve ser considerada vigente se sua origem ou seus parâmetros de derivação forem incompatíveis com o estado documental que o sistema considera vigente.**

Em forma simplificada:

```text
DOCUMENTO V2
     ↓
REPRESENTAÇÃO V2
     ↓
SEGMENTAÇÃO V2
     ↓
EMBEDDING V2
```

é coerente.

Já:

```text
DOCUMENTO V2
     ↓
REPRESENTAÇÃO V2
     ↓
SEGMENTAÇÃO V1
     ↓
EMBEDDING V1
```

é potencialmente inconsistente.

A arquitetura precisa ser capaz de identificar essa situação.

---

# 8. Mudança do documento original

A origem externa do Snoopy é o Google Drive.

Isso cria uma característica particular: o sistema não controla diretamente a permanência do conteúdo original.

O usuário pode:

* editar;
* substituir;
* remover;
* renomear;
* mover;
* restaurar;
* modificar o conteúdo do arquivo.

Assim, o Snoopy precisa distinguir:

```text
arquivo atualmente existente no Drive
```

de:

```text
estado documental anteriormente processado pelo Snoopy
```

Essa diferença é crucial para auditoria.

Se o documento mudou, uma evidência antiga não pode ser tratada automaticamente como se tivesse sido produzida sobre o novo documento.

---

# 9. Detecção de mudanças

A arquitetura deve possuir uma estratégia explícita para responder:

> **“O documento que está no Drive ainda corresponde ao documento que foi processado?”**

A resposta pode envolver informações como:

```text
identidade do arquivo
+
identidade/hash do conteúdo
+
metadados de alteração
+
estado de processamento
```

O ponto importante nesta etapa não é escolher uma API ou mecanismo específico.

É estabelecer que:

> **mudança no documento original deve ser detectável e deve provocar uma decisão explícita sobre a validade das representações derivadas.**

---

# 10. Invalidação de derivados

Suponhamos:

```text
Documento V1
 ↓
chunks V1
 ↓
embeddings V1
```

e o documento muda:

```text
Documento V2
```

A arquitetura não deve continuar tratando automaticamente:

```text
chunks V1
embeddings V1
```

como representantes atuais de V2.

É necessário algum mecanismo conceitual de:

```text
VALIDAÇÃO
```

ou:

```text
INVALIDAÇÃO
```

dos derivados.

O estado poderia ser pensado como:

```text
Documento V1
   ↓
derivados válidos
```

e depois:

```text
Documento V2
   ↓
derivados V1 → obsoletos/inválidos
   ↓
novo processamento
   ↓
derivados V2 → válidos
```

A palavra importante aqui é **vigência**.

Um objeto antigo não precisa necessariamente ser apagado.

Ele precisa deixar de ser confundido com o estado atual.

---

# 11. Apagar não é o mesmo que invalidar

Essa distinção merece destaque.

Existem pelo menos três estados conceituais:

```text
ATIVO
```

```text
OBSOLETO
```

```text
REMOVIDO
```

Um chunk de uma versão anterior pode estar:

```text
obsoleto
```

sem precisar desaparecer imediatamente.

Isso é importante porque ele pode continuar sendo necessário para:

* auditoria;
* histórico;
* reprodução de uma busca antiga;
* investigação de uma inconsistência;
* comparação entre versões.

Já um documento efetivamente removido do acervo pode exigir outra política.

Portanto:

> **A arquitetura não deve usar exclusão física como único mecanismo de controle de estado.**

---

# 12. Reprocessamento

O reprocessamento não deve ser entendido simplesmente como:

```text
apagar tudo
↓
processar novamente
```

O Snoopy precisa poder distinguir:

```text
novo documento
```

de:

```text
mesmo documento, nova representação
```

de:

```text
mesmo documento, nova segmentação
```

de:

```text
mesmo documento, novo embedding
```

Isso é particularmente importante porque as Etapas 5.1–5.3 estabelecem diferentes níveis de representação.

Por exemplo:

```text
PDF
 ↓
representação canônica
```

pode permanecer a mesma enquanto:

```text
segmentação
```

é aprimorada.

Nesse caso, não seria metodologicamente desejável reextrair o PDF desnecessariamente.

A arquitetura definida na Etapa 5.1 justamente busca tornar possível o reprocessamento de representações derivadas sem repetir transformações destrutivas sobre o documento original.

---

# 13. Reprocessamento incremental

A consequência natural é a possibilidade de **reprocessamento incremental**.

Em vez de:

```text
qualquer alteração
 ↓
reprocessar tudo
```

a arquitetura deve ser capaz de raciocinar:

```text
o que mudou?
```

e:

```text
quais derivados dependem dessa mudança?
```

Exemplo conceitual:

```text
PDF original
   ↓
representação canônica permanece válida
   ↓
nova segmentação
   ↓
novos retrieval units
   ↓
novos embeddings
```

Ou:

```text
PDF mudou
   ↓
representação canônica precisa ser atualizada
   ↓
estrutura derivada torna-se potencialmente inválida
   ↓
segmentação precisa ser refeita
   ↓
embeddings precisam ser refeitos
```

O objetivo é evitar tanto:

```text
reprocessamento desnecessário
```

quanto:

```text
reutilização indevida de derivados antigos.
```

---

# 14. Embeddings possuem versão própria

A Etapa 5.3 estabeleceu que o modelo de embedding e suas dimensões fazem parte do perfil de recuperação.

Isso possui uma consequência direta para a persistência.

Se:

```text
unidade U
 ↓
embedding modelo A
```

e posteriormente o sistema passa a utilizar:

```text
embedding modelo B
```

o vetor antigo não deve ser confundido com o novo.

Conceitualmente:

```text
U
├── embedding A
└── embedding B
```

ou:

```text
U versão 1
   ↓
embedding A
```

e:

```text
U versão 2
   ↓
embedding B
```

O mecanismo exato ainda não é uma decisão desta etapa.

O requisito é:

> **O sistema deve saber qual representação vetorial corresponde a qual versão da unidade e qual configuração de embedding a produziu.**

---

# 15. Mudança da segmentação

A mesma questão existe para o chunking.

Atualmente, o Snoopy utiliza uma segmentação heurística/estrutural. A análise metodológica já estabeleceu que a arquitetura futura deverá separar:

```text
estrutura
 ↓
segmentação estrutural
 ↓
segmentação semântica
 ↓
unidade de recuperação
```

Isso significa que uma mudança na estratégia de segmentação pode produzir:

```text
unidades completamente diferentes
```

a partir do mesmo documento.

Portanto:

```text
documento igual
```

não implica necessariamente:

```text
corpus recuperável igual.
```

O perfil do processamento precisa distinguir versões da segmentação.

---

# 16. O corpus também possui estado

A unidade de versionamento não deve ser apenas o documento individual.

O próprio corpus pode mudar.

Por exemplo:

```text
CORPUS V1
 ├── A
 ├── B
 └── C
```

pode tornar-se:

```text
CORPUS V2
 ├── A
 ├── B
 ├── C
 └── D
```

ou:

```text
CORPUS V3
 ├── A
 ├── C
 └── D
```

Isso importa porque a recuperação ocorre dentro de uma fronteira de corpus.

A Etapa 5.3 estabeleceu que:

> **o corpus faz parte do perfil de recuperação.**

Portanto:

```text
mesma pergunta
+
configuração igual
+
corpus diferente
```

pode legitimamente produzir:

```text
resultado diferente
```

Isso não é instabilidade do algoritmo.

É mudança do objeto de recuperação.

---

# 17. Corpus não é apenas uma coleção de documentos

O corpus deve ser entendido como:

```text
conjunto documental
+
fronteira de acesso
+
estado/versionamento
```

Portanto, conceitualmente:

```text
CORPUS
   │
   ├── documentos
   ├── versões
   ├── estado
   └── configuração de escopo
```

Essa definição será importante também para a segurança e o isolamento tratados em 5.6.

---

# 18. Consistência entre documento e chunks

O problema mais simples é:

```text
document
```

existe, mas seus chunks desapareceram.

Ou:

```text
chunks
```

existem, mas o documento correspondente não está mais disponível.

Ou ainda:

```text
chunks V1
```

estão associados a:

```text
document V2
```

sem que a arquitetura consiga distinguir isso.

A regra arquitetural deve ser:

> **Um conjunto de unidades recuperáveis deve sempre possuir uma relação consistente com uma versão documental identificável.**

Isso evita a existência de:

```text
chunks órfãos
```

ou:

```text
chunks semanticamente órfãos
```

mesmo quando seus `document_id` ainda existem.

---

# 19. Consistência entre chunks e embeddings

Também deve existir:

```text
UNIDADE V1
   ↓
EMBEDDING V1
```

e não:

```text
UNIDADE V2
   ↓
EMBEDDING V1
```

sem que isso seja explicitamente suportado.

Essa questão é particularmente importante porque a busca vetorial pode produzir resultados perfeitamente plausíveis mesmo quando os vetores pertencem a uma representação antiga.

Ou seja:

> **um sistema pode continuar funcionando tecnicamente enquanto está semanticamente inconsistente.**

Esse é um dos tipos mais perigosos de erro arquitetural.

Não necessariamente gera uma exceção.

Pode simplesmente produzir resultados errados.

---

# 20. Consistência entre jobs e estado documental

A fila também precisa participar desse modelo.

Atualmente, o processamento ocorre por meio de jobs persistentes:

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



Isso é adequado como desacoplamento operacional, mas não resolve sozinho a consistência documental.

Por exemplo:

```text
job
 ↓
processing
```

pode estar produzindo uma nova representação enquanto outro processo altera o estado do documento.

A arquitetura precisa ser capaz de responder:

> “Este job está produzindo qual versão do documento e quais derivados?”

Assim, um job não deve representar simplesmente:

```text
“processar arquivo X”
```

mas conceitualmente:

```text
“produzir determinada representação/estado derivado do documento X”
```

---

# 21. Falhas durante o processamento

Imagine:

```text
PDF
 ↓
extração ✓
 ↓
representação ✓
 ↓
segmentação ✓
 ↓
embeddings ✓
 ↓
persistência ✗
```

O sistema não pode simplesmente assumir:

```text
completed
```

nem deixar um estado parcialmente produzido parecer completo.

Isso exige distinguir:

```text
processamento iniciado
```

de:

```text
derivados válidos e disponíveis
```

A fila atual já distingue `processing`, `completed` e `failed`, mas a nova arquitetura deve levar essa ideia para o estado documental como um todo.

---

# 22. Estado parcial

Um processamento pode produzir objetos intermediários.

Por exemplo:

```text
documento
 ↓
representação canônica
 ↓
segmentação
 ↓
100 chunks produzidos
 ↓
falha no chunk 101
```

Esses 100 chunks não devem automaticamente entrar no corpus recuperável como se representassem uma versão completa do documento.

É necessário algum conceito de:

```text
estado parcial
```

ou:

```text
estado não publicado
```

seguido de:

```text
publicação/ativação
```

somente quando os requisitos de consistência forem satisfeitos.

Isso evita que o motor de busca consulte uma representação incompleta sem saber disso.

---

# 23. Processamento e publicação são conceitos diferentes

Uma distinção útil é:

```text
PROCESSAR
```

versus:

```text
TORNAR DISPONÍVEL PARA RECUPERAÇÃO
```

O pipeline pode estar processando:

```text
Documento V2
```

enquanto o corpus continua usando:

```text
Documento V1
```

Até que a nova versão esteja consistente.

Conceitualmente:

```text
V1 → ATIVA
V2 → EM PROCESSAMENTO
```

Depois:

```text
V1 → OBSOLETA
V2 → ATIVA
```

Isso é muito mais seguro do que substituir objetos gradualmente enquanto a busca continua executando.

---

# 24. Publicação atômica do estado

Surge então um princípio arquitetural importante:

> **Uma nova versão documental deve tornar-se recuperável como um estado coerente, e não como uma coleção de objetos substituídos independentemente.**

Conceitualmente:

```text
V2
 ├── representação ✓
 ├── segmentação ✓
 ├── embeddings ✓
 └── proveniência ✓
        ↓
     PUBLICAR
        ↓
    V2 ATIVA
```

O mecanismo técnico de implementação ainda não é definido aqui.

O requisito, entretanto, é claro:

> **O pesquisador não deve receber uma mistura acidental de V1 e V2 como se fosse um único estado documental.**

---

# 25. Cache e persistência

O cache representa outro estado derivado.

A arquitetura atual armazena histórico/resultados de busca e pode reutilizar resultados para reduzir processamento e tokens. A análise anterior já observou que isso pode mascarar testes de reprodutibilidade. 

Na arquitetura futura, o cache deve ser entendido como:

```text
resultado derivado
```

e não como:

```text
verdade permanente sobre a consulta.
```

Portanto, um resultado armazenado precisa estar relacionado, conceitualmente, a:

```text
pergunta
+
corpus
+
versão documental
+
perfil de recuperação
```

Se qualquer elemento relevante mudar:

```text
cache antigo
```

pode deixar de ser válido.

---

# 26. Invalidação de cache

A regra geral deve ser:

> **Um cache só é reutilizável quando o estado sobre o qual ele foi produzido permanece compatível com o estado atual exigido pela consulta.**

Por exemplo:

```text
consulta Q
corpus V10
configuração R3
 ↓
cache X
```

não deve ser automaticamente reutilizado para:

```text
consulta Q
corpus V11
configuração R3
```

nem:

```text
consulta Q
corpus V10
configuração R4
```

nem:

```text
consulta Q
corpus V10
configuração R3
pipeline V2
```

se essas mudanças forem relevantes para o resultado.

---

# 27. Consistência e reprodutibilidade

A persistência é uma das bases da reprodutibilidade.

A cadeia pode ser vista assim:

```text
CORPUS
+
REPRESENTAÇÃO
+
SEGMENTAÇÃO
+
EMBEDDING
+
CONFIGURAÇÃO DE RECUPERAÇÃO
+
VERSÃO DO PIPELINE
        ↓
RESULTADO
```

Se qualquer elemento relevante desaparecer ou for sobrescrito, torna-se difícil reproduzir o resultado.

Isso reforça uma conclusão já estabelecida:

> **A reprodutibilidade da recuperação depende não apenas do algoritmo, mas do estado documental sobre o qual o algoritmo operou.**

---

# 28. Consistência temporal da evidência

A Etapa 5.4 estabeleceu a cadeia:

```text
evidência
 ↓
unidade
 ↓
página
 ↓
documento
 ↓
versão
 ↓
original
```

A Etapa 5.5 acrescenta:

```text
quando?
```

Portanto:

```text
EVIDÊNCIA
   ↓
produzida em
   ↓
estado documental V1
   ↓
sob configuração R1
```

Se o documento posteriormente virar V2, a evidência histórica não deve ser silenciosamente reinterpretada como evidência produzida sobre V2.

Isso é essencial para qualquer auditoria posterior.

---

# 29. Remoção de documentos

A remoção é diferente de atualização.

Se um documento sai do corpus:

```text
Documento X
 ↓
removido
```

o sistema precisa decidir o que acontece com:

* chunks;
* embeddings;
* resultados de busca;
* histórico;
* referências;
* cache;
* proveniência.

A solução não deve ser definida aqui em termos de mecanismo específico.

Mas a regra conceitual deve ser:

> **Remover um documento do corpus ativo não deve apagar automaticamente a capacidade de explicar que ele existiu e participou de determinado estado documental, quando essa preservação for necessária para auditoria e histórico.**

Isso também precisa respeitar políticas de retenção e privacidade.

---

# 30. Remoção lógica versus física

Podem existir duas operações conceitualmente diferentes:

```text
REMOVER DO CORPUS ATIVO
```

e:

```text
DESTRUIR PERMANENTEMENTE
```

A primeira significa:

> “Não utilizar este documento nas novas recuperações.”

A segunda significa:

> “Eliminar seus dados persistidos conforme a política aplicável.”

Misturar essas duas operações pode destruir histórico desnecessariamente.

Portanto, a arquitetura precisa definir explicitamente o ciclo de vida documental.

---

# 31. Concorrência

A fila atual possui uma fragilidade conhecida: o worker consulta um job `pending` e posteriormente o marca como `processing`, sem um mecanismo transacional robusto de reserva no código analisado. Com múltiplos workers, dois processos poderiam potencialmente selecionar o mesmo job. 

Isso é inicialmente uma questão operacional, mas possui consequência direta para consistência.

Se dois workers processarem o mesmo documento:

```text
worker A → versão X
worker B → versão X
```

podemos ter:

* duplicação;
* escrita concorrente;
* estados parcialmente sobrescritos;
* embeddings duplicados;
* resultados inconsistentes.

Ou, em um cenário mais grave:

```text
worker A → documento V2
worker B → documento V3
```

e a ordem de conclusão pode fazer a versão antiga sobrescrever a nova.

Portanto:

> **Controle de concorrência é parte da consistência documental.**

A solução técnica será tratada posteriormente.

---

# 32. Idempotência

Outro princípio importante é a **idempotência** (reexecutar uma operação sem produzir efeitos inconsistentes adicionais).

Se um job for executado duas vezes:

```text
processar documento X
```

o resultado não deve produzir arbitrariamente:

```text
duplicação de documentos
duplicação de chunks
duplicação de embeddings
```

ou múltiplas versões indistinguíveis.

A arquitetura precisa conseguir responder:

> “Esta operação já foi aplicada a este estado?”

Isso é particularmente importante para:

* retries;
* falhas;
* reinicialização do worker;
* sincronização repetida;
* processamento concorrente.

---

# 33. Estado desejado versus estado observado

Uma arquitetura robusta precisa distinguir:

```text
O que o sistema pretendia produzir
```

de:

```text
O que efetivamente produziu.
```

Por exemplo:

```text
job → processar Documento V2
```

não significa que:

```text
Documento V2
 ↓
todos os derivados
```

existam corretamente.

O estado observado deve poder demonstrar:

```text
representação ✓
segmentação ✓
embeddings ✓
proveniência ✓
publicação ✓
```

ou:

```text
representação ✓
segmentação ✓
embeddings ✗
```

Isso será importante para diagnóstico e recuperação de falhas.

---

# 34. Integridade entre camadas

Podemos representar a consistência desejada como:

```text
DOCUMENTO V
     │
     ▼
REPRESENTAÇÃO V
     │
     ▼
SEGMENTAÇÃO V
     │
     ▼
UNIDADES V
     │
     ▼
EMBEDDINGS V
     │
     ▼
RECUPERAÇÃO R
```

Cada camada precisa saber de qual estado anterior depende.

A arquitetura não precisa necessariamente duplicar o identificador de versão em absolutamente todos os objetos.

Mas precisa existir uma forma inequívoca de reconstruir essas relações.

---

# 35. Persistência como grafo de dependências

Uma maneira mais precisa de modelar a arquitetura futura é pensar nos dados como um **grafo de dependências**:

```text
DOCUMENTO
   │
   ▼
REPRESENTAÇÃO
   │
   ▼
ESTRUTURA
   │
   ▼
SEGMENTAÇÃO
   │
   ▼
UNIDADE
   │
   ▼
EMBEDDING
```

Se um nó muda:

```text
REPRESENTAÇÃO V1
       ↓
       X
```

o sistema deve conseguir identificar os nós dependentes:

```text
SEGMENTAÇÃO V1
UNIDADES V1
EMBEDDINGS V1
```

Esses objetos podem então ser:

```text
reutilizados
reprocessados
invalidados
ou substituídos
```

conforme a política definida.

Isso é muito mais seguro do que pensar o pipeline apenas como uma sequência de scripts.

---

# 36. O pipeline como transformação versionada

A visão das etapas anteriores pode ser refinada:

```text
DOCUMENTO
   ↓
TRANSFORMAÇÃO T1
   ↓
REPRESENTAÇÃO
   ↓
TRANSFORMAÇÃO T2
   ↓
SEGMENTAÇÃO
   ↓
TRANSFORMAÇÃO T3
   ↓
EMBEDDING
```

Cada transformação deve possuir uma identidade suficientemente explícita para que o sistema saiba:

> “Este resultado foi produzido por qual versão da transformação?”

Isso inclui, conceitualmente:

* versão da representação;
* versão da segmentação;
* versão do embedding;
* versão do pipeline;
* configuração relevante.

Essa informação já aparece como requisito de perfil de recuperação na análise anterior. 

---

# 37. Consistência não significa imutabilidade absoluta

Um cuidado importante:

> **Versionar não significa nunca alterar nada.**

O Snoopy precisa poder evoluir.

Pode haver:

```text
representação V1
```

e depois:

```text
representação V2
```

O objetivo não é congelar o sistema.

É evitar que:

```text
V2
```

apague silenciosamente a possibilidade de distinguir:

```text
V1
```

quando essa distinção for relevante.

Portanto:

```text
evolução
+
identidade
+
histórico
```

são compatíveis.

---

# 38. Requisitos de Persistência, Versionamento e Consistência

A partir das análises anteriores, podem ser estabelecidos os seguintes requisitos.

### RPVC1 — Persistência documental

O estado documental necessário à recuperação deve possuir persistência identificável.

### RPVC2 — Identidade

Documentos devem possuir identidade persistente independente de sua representação textual.

### RPVC3 — Identidade de conteúdo

A arquitetura deve possuir mecanismo para identificar mudanças relevantes no conteúdo documental.

### RPVC4 — Versionamento

Alterações relevantes devem poder gerar estados documentais distinguíveis.

### RPVC5 — Vigência

O sistema deve distinguir estado atual, estado anterior e estado inválido/obsoleto.

### RPVC6 — Derivação

Representações derivadas devem estar vinculadas ao estado documental de que derivam.

### RPVC7 — Compatibilidade

Representações, segmentações e embeddings devem possuir relações de compatibilidade identificáveis.

### RPVC8 — Invalidação

Mudanças em uma camada devem permitir identificar derivados potencialmente inválidos.

### RPVC9 — Reprocessamento

O sistema deve permitir reprocessamento sem assumir que toda etapa anterior precisa ser repetida.

### RPVC10 — Reprocessamento incremental

Quando possível, apenas os derivados afetados por uma alteração devem precisar ser reconstruídos.

### RPVC11 — Embedding versionado

Embeddings devem ser distinguíveis segundo a representação e configuração que os produziram.

### RPVC12 — Segmentação versionada

Mudanças na estratégia de segmentação devem produzir estados distinguíveis.

### RPVC13 — Corpus versionado

O estado do corpus utilizado em uma recuperação deve ser identificável.

### RPVC14 — Consistência documento/unidade

Unidades recuperáveis devem possuir relação consistente com uma versão documental.

### RPVC15 — Consistência unidade/embedding

Vetores devem corresponder à unidade e representação corretas.

### RPVC16 — Consistência de job

Jobs devem identificar o estado documental que pretendem produzir.

### RPVC17 — Estado parcial

Processamentos incompletos não devem ser apresentados automaticamente como estados documentais completos.

### RPVC18 — Publicação

Uma nova versão deve tornar-se recuperável somente quando seu estado necessário estiver consistente.

### RPVC19 — Concorrência

Processamentos concorrentes não devem produzir estados documentais incompatíveis.

### RPVC20 — Idempotência

Reexecuções e retries não devem produzir inconsistências ou duplicações indevidas.

### RPVC21 — Falhas

Falhas intermediárias devem produzir estados distinguíveis de processamento concluído.

### RPVC22 — Histórico

Resultados históricos devem permanecer associados ao estado documental sobre o qual foram produzidos.

### RPVC23 — Cache

Resultados em cache devem possuir relação com o estado do corpus e da configuração que os originou.

### RPVC24 — Invalidação de cache

Mudanças relevantes no corpus, representação ou configuração devem permitir invalidar resultados incompatíveis.

### RPVC25 — Remoção

A remoção do corpus ativo deve ser distinguível da destruição permanente dos dados.

### RPVC26 — Integridade

O sistema deve poder verificar se os derivados correspondem ao estado documental declarado.

### RPVC27 — Auditoria temporal

Deve ser possível determinar, quando necessário, qual versão documental sustentava determinada evidência.

### RPVC28 — Reprodutibilidade

Informações necessárias para reproduzir a recuperação devem sobreviver à evolução do sistema.

### RPVC29 — Independência de implementação

Os princípios de consistência não devem depender de uma tecnologia específica.

### RPVC30 — Recuperabilidade

O sistema deve possuir condições para recuperar-se de falhas sem produzir silenciosamente um estado documental inconsistente.

---

# 39. Modelo conceitual de ciclo de vida documental

O ciclo de vida pode ser representado assim:

```text
                 DESCOBERTO
                     │
                     ▼
                 PROCESSANDO
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
      CONCLUÍDO               FALHO
          │
          ▼
     VALIDANDO
          │
          ▼
        ATIVO
          │
          ├──────────────┐
          ▼              ▼
      ATUALIZADO      REMOVIDO
          │              │
          ▼              ▼
       OBSOLETO      INATIVO
```

Esse é um **modelo conceitual**, não uma determinação dos estados que necessariamente deverão existir na implementação.

A finalidade é deixar claro que:

```text
processamento
≠
validade
≠
vigência
≠
existência física
```

---

# 40. Modelo conceitual de versões

Uma arquitetura futura pode ser pensada como:

```text
DOCUMENTO X
│
├── V1
│   ├── representação R1
│   ├── segmentação S1
│   └── embeddings E1
│
├── V2
│   ├── representação R2
│   ├── segmentação S2
│   └── embeddings E2
│
└── V3
    ├── representação R3
    ├── segmentação S3
    └── embeddings E3
```

Uma recuperação:

```text
consulta Q
 ↓
corpus C
 ↓
versão documental V2
 ↓
configuração R5
 ↓
resultados
```

pode então ser distinguida de:

```text
consulta Q
 ↓
corpus C
 ↓
versão documental V3
 ↓
configuração R5
 ↓
resultados
```

Mesmo que a pergunta seja literalmente igual.

---

# 41. Relação com a Etapa 5.4

A Etapa 5.4 dizia:

```text
evidência
 ↓
unidade
 ↓
documento
 ↓
versão
 ↓
original
```

A Etapa 5.5 transforma “versão” em um conceito operacional.

Agora podemos estabelecer:

```text
evidência
 ↓
unidade U2
 ↓
segmentação S2
 ↓
representação R2
 ↓
documento V2
 ↓
arquivo original correspondente
```

E, paralelamente:

```text
pergunta
 ↓
consulta
 ↓
perfil de recuperação R5
 ↓
resultado
```

A proveniência passa a possuir dimensão temporal.

---

# 42. Relação com a Etapa 5.3

A recuperação depende do estado persistente.

A cadeia completa é:

```text
CORPUS V
+
REPRESENTAÇÃO V
+
SEGMENTAÇÃO V
+
EMBEDDING V
+
PERFIL DE RECUPERAÇÃO
       ↓
RESULTADO
```

Portanto, a persistência não é uma camada administrativa colocada depois da recuperação.

Ela é uma **condição de validade da própria recuperação**.

Sem versionamento:

> “este resultado veio daqui”

pode continuar sendo verdade,

mas:

> “este resultado foi produzido sobre este estado do corpus”

pode se tornar impossível de demonstrar.

---

# 43. Relação com a Etapa 5.1

A representação canônica ganha uma importância ainda maior.

Se a representação canônica for preservada:

```text
PDF
 ↓
CANÔNICA V1
```

podemos produzir:

```text
SEGMENTAÇÃO A
```

e posteriormente:

```text
SEGMENTAÇÃO B
```

sem necessariamente voltar a destruir e reinterpretar o PDF original.

Isso permite:

```text
CANÔNICA
   ├── derivação A
   ├── derivação B
   └── derivação C
```

Em outras palavras:

> **a representação canônica não é apenas uma proteção contra perda de informação; ela também é uma âncora de versionamento.**

---

# 44. Relação com a Etapa 5.2

A segmentação também passa a ser uma entidade versionável.

Isso é particularmente importante porque o mesmo documento pode possuir:

```text
segmentação estrutural V1
```

e:

```text
segmentação estrutural V2
```

sem que o documento tenha mudado.

O sistema deve então ser capaz de distinguir:

```text
mudança documental
```

de:

```text
mudança de representação derivada.
```

Essa distinção permitirá estudar empiricamente o efeito de diferentes estratégias de segmentação sem confundir a mudança do documento com a mudança do processamento.

---

# 45. Avaliação empírica da consistência

A arquitetura futura deve ser testada em situações que provoquem mudanças reais.

## Teste 1 — Reprocessamento idêntico

Processar o mesmo documento duas vezes sob a mesma configuração e verificar se o estado resultante é consistente.

## Teste 2 — Documento alterado

Modificar o PDF no Drive e verificar se o sistema identifica a mudança.

## Teste 3 — Reprocessamento

Atualizar o documento e verificar se os derivados antigos deixam de ser tratados como atuais.

## Teste 4 — Mudança de segmentação

Aplicar uma nova estratégia de segmentação sem alterar o documento e verificar se os estados são distinguidos.

## Teste 5 — Mudança de embedding

Alterar a configuração de embeddings e verificar se os vetores antigos não são confundidos com os novos.

## Teste 6 — Falha intermediária

Interromper o processamento depois de uma etapa intermediária e verificar se o estado parcial não entra indevidamente no corpus ativo.

## Teste 7 — Retry

Executar novamente um job que falhou e verificar idempotência.

## Teste 8 — Concorrência

Executar múltiplos workers sobre a mesma fila e verificar se um job/documento pode ser processado simultaneamente de maneira inconsistente.

## Teste 9 — Cache

Modificar o corpus e verificar se resultados incompatíveis não continuam sendo servidos como atuais.

## Teste 10 — Histórico

Produzir uma evidência sobre V1, atualizar o documento para V2 e verificar se a evidência antiga continua identificando corretamente V1.

## Teste 11 — Remoção

Remover um documento do corpus e verificar a diferença entre:

```text
não participar de novas buscas
```

e:

```text
destruição completa do histórico
```

---

# 46. Critérios de qualidade

A qualidade da persistência e versionamento não deve ser medida por:

> “Os dados estão no banco.”

Isso é insuficiente.

Os critérios relevantes são:

```text
IDENTIDADE
   +
CONSISTÊNCIA
   +
VERSIONAMENTO
   +
INTEGRIDADE
   +
RECUPERABILIDADE
   +
AUDITABILIDADE
```

Uma arquitetura pode armazenar milhões de registros e ainda assim ser inconsistente se não souber quais registros pertencem ao mesmo estado documental.

---

# 47. O problema mais perigoso: inconsistência silenciosa

Há um tipo de erro particularmente relevante para o Snoopy:

```text
erro explícito
```

é relativamente fácil de detectar:

```text
job failed
```

Já:

```text
inconsistência silenciosa
```

pode ser muito mais perigosa:

```text
documento V2
+
chunks V1
+
embeddings V1
```

O sistema pode continuar respondendo.

O usuário pode não perceber.

A busca pode até parecer boa.

Mas a cadeia documental estará errada.

Portanto:

> **A arquitetura deve priorizar mecanismos que tornem inconsistências detectáveis, em vez de simplesmente presumir que os estados persistidos são compatíveis.**

---

# 48. Princípio de publicação consistente

Uma consequência importante de toda a etapa pode ser condensada em:

> **O estado utilizado pela recuperação deve ser um estado documental explicitamente válido e internamente consistente.**

Não:

```text
“os dados que já estão no banco”
```

mas:

```text
“o estado documental atualmente publicado para recuperação”
```

Isso muda bastante a concepção do sistema.

O banco deixa de ser simplesmente um depósito de objetos.

Ele passa a representar **estados documentais publicáveis**.

---

# 49. Princípio de não mistura

Outro princípio deve ser registrado:

> **Estados documentais, versões de representação e configurações de recuperação diferentes não devem ser misturados silenciosamente em uma única recuperação.**

Por exemplo, não se deve produzir:

```text
resultado
 ├── chunk V1
 ├── chunk V2
 └── chunk V1
```

sem que exista uma justificativa arquitetural explícita para isso.

A regra normal deve ser:

```text
uma recuperação
 ↓
um estado documental coerente
```

Essa propriedade é especialmente importante para testes de reprodutibilidade.

---

# 50. Princípio de dependência explícita

A arquitetura deve conseguir responder:

> “Se eu alterar X, o que deixa de ser válido?”

Idealmente:

```text
alterei PDF
 ↓
representação afetada
 ↓
segmentação afetada
 ↓
unidades afetadas
 ↓
embeddings afetados
 ↓
cache afetado
```

Enquanto:

```text
alterei apenas configuração de ranking
```

pode não exigir:

```text
reextração do PDF
```

ou:

```text
novo embedding
```

Essa noção de dependência permitirá que a implementação futura seja mais eficiente sem sacrificar consistência.

---

# 51. O princípio geral da Etapa 5.5

Podemos agora formular o princípio central:

> **O Snoopy-RAG deverá tratar documentos, representações derivadas, unidades de recuperação, embeddings e resultados como estados relacionados por dependências explícitas e versionáveis, de modo que alterações no documento, na representação ou no processamento possam ser detectadas, propagadas ou invalidadas sem permitir que estados incompatíveis sejam utilizados silenciosamente na recuperação.**

Uma segunda formulação complementa:

> **Persistir não significa apenas armazenar dados; significa preservar a identidade, a vigência, as dependências e a consistência dos estados documentais sobre os quais o sistema opera.**

---

# 52. Síntese da Etapa 5.5

A Etapa 5.5 revela que a arquitetura documental do Snoopy não pode ser pensada apenas como:

```text
documento → chunks → embeddings
```

Ela precisa ser pensada como:

```text
documento
   ↓
estado documental
   ↓
representação
   ↓
segmentação
   ↓
unidades
   ↓
embeddings
   ↓
recuperação
```

com cada estágio possuindo uma relação identificável com o estado anterior.

Quando o documento muda:

```text
não basta atualizar document
```

É necessário determinar:

```text
o que permanece válido?
o que ficou obsoleto?
o que precisa ser reprocessado?
o que precisa ser invalidado?
o que pode continuar sendo utilizado historicamente?
```

Quando a segmentação muda:

```text
não basta substituir chunks.
```

É necessário distinguir:

```text
unidades antigas
```

de:

```text
unidades novas.
```

Quando o embedding muda:

```text
não basta substituir vetores.
```

É necessário saber:

```text
qual vetor corresponde a qual representação
e a qual configuração.
```

Quando o corpus muda:

```text
não basta adicionar/remover documentos.
```

É necessário reconhecer que:

```text
o objeto sobre o qual a recuperação opera também mudou.
```

E quando um job falha:

```text
não basta registrar failed.
```

É necessário impedir que um estado parcialmente produzido seja confundido com um estado documental válido.

---

# 53. Arquitetura conceitual consolidada após a 5.5

As cinco primeiras etapas da nova especificação agora podem ser reunidas:

```text
                 DOCUMENTO ORIGINAL
                         │
                         ▼
              5.1 REPRESENTAÇÃO CANÔNICA
                         │
                         ▼
              5.2 ESTRUTURAÇÃO / SEGMENTAÇÃO
                         │
                         ▼
                 UNIDADES DE RECUPERAÇÃO
                         │
                         ▼
                  5.3 REPRESENTAÇÃO
                         │
                         ▼
                     RECUPERAÇÃO
                         │
                         ▼
                    5.4 PROVENIÊNCIA
                         │
                         ▼
                      EVIDÊNCIA
                         │
                         ▼
                     SÍNTESE
                         │
                         ▼
                    PESQUISADOR
```

E atravessando todo esse fluxo:

```text
        ┌──────────────────────────────────────┐
        │ 5.5 PERSISTÊNCIA / VERSIONAMENTO    │
        │      / CONSISTÊNCIA                  │
        └──────────────────────────────────────┘
```

Ou seja:

> **5.5 não é uma etapa depois da recuperação. É uma propriedade transversal que mantém as etapas anteriores coerentes ao longo do tempo.**

---

# 54. Conclusão

A Etapa 5.1 estabeleceu:

> **preservar antes de transformar.**

A 5.2 estabeleceu:

> **estruturar antes de segmentar.**

A 5.3 estabeleceu:

> **recuperar sem confundir similaridade com relevância metodológica.**

A 5.4 estabeleceu:

> **rastrear a evidência até sua origem e distinguir isso da proveniência metodológica.**

A 5.5 acrescenta:

> **manter tudo isso coerente enquanto o sistema muda.**

A consequência é uma mudança importante no modelo mental do Snoopy.

Não temos mais simplesmente:

```text
PDF → processamento → banco
```

Temos:

```text
DOCUMENTO
   ↓
ESTADO DOCUMENTAL
   ↓
REPRESENTAÇÕES DERIVADAS
   ↓
DEPENDÊNCIAS
   ↓
ESTADO RECUPERÁVEL
   ↓
EVIDÊNCIAS
```

e cada estado precisa possuir:

```text
IDENTIDADE
+
VERSÃO
+
VIGÊNCIA
+
DEPENDÊNCIAS
+
INTEGRIDADE
```

O ponto mais importante é talvez este:

> **O Snoopy não pode permitir que um estado antigo continue parecendo atual apenas porque seus dados ainda existem.**

Isso vale para documentos, chunks, embeddings, caches, resultados e referências de evidência.

A persistência deixa, portanto, de ser apenas armazenamento e passa a ser **memória estruturada do sistema**.

O princípio consolidado da Etapa 5.5 é:

> **O Snoopy-RAG deverá manter estados documentais e computacionais identificáveis, versionáveis e internamente consistentes, preservando as relações de dependência entre documento, representação, segmentação, unidades de recuperação, embeddings, recuperação e evidências. Alterações, reprocessamentos, falhas, remoções e mudanças de configuração deverão produzir estados distinguíveis e permitir a invalidação ou reconstrução dos derivados afetados, sem misturar silenciosamente estados incompatíveis.**

E a cadeia da especificação chega agora a:

```text
PRESERVAR
    ↓
ESTRUTURAR
    ↓
SEGMENTAR
    ↓
REPRESENTAR
    ↓
RECUPERAR
    ↓
RASTREAR
    ↓
PERSISTIR / VERSIONAR / VALIDAR
    ↓
SINTETIZAR
    ↓
INTERPRETAR
```

O próximo ponto, a **5.6**, fecha o bloco arquitetural atacando a fronteira que atravessa todas essas camadas: **quem pode produzir, acessar, alterar ou observar cada estado, e como o sistema deve operar de forma segura e confiável em produção, desenvolvimento, fila, autenticação e múltiplos acervos**.
