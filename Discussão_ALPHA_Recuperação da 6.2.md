Sim. Fiz a leitura pensando não em “o que a 6.2 decidiu para o Snoopy definitivo?”, mas em uma pergunta bem mais cruel:

Se eu arrancar tudo que é futuro, sofisticado ou substituível, o que da 5 + 6.1 + 6.2 ainda precisa existir para a V3 Alpha funcionar sem violar os princípios que já fechamos?

A documentação do repositório separa justamente essas investigações em 5.1–5.6, 6.1 e 6.2.1–6.2.8.

E cheguei a uma conclusão importante: a Alpha não precisa implementar a 6.2 inteira em miniatura. Ela precisa implementar um núcleo vertical coerente dessas decisões. Algumas coisas precisam existir como funcionalidade; outras precisam existir apenas como propriedade arquitetural; e uma terceira classe pode ser explicitamente postergada.

1. Primeiro: o que estou chamando de "indispensável"

Eu separaria o resultado em três níveis:

🔴 I — Indispensável para a Alpha

Se isso não existir, a Alpha quebra funcionalmente ou viola uma decisão fundamental das Etapas 5/6.

🟡 II — Indispensável como princípio arquitetural, mas pode ter implementação mínima

A Alpha precisa respeitar a propriedade, mas não precisa implementar toda a sofisticação investigada na 6.2.

⚪ III — Pós-Alpha

Foi investigado e é importante para o Snoopy definitivo, mas sua ausência não impede a Alpha de cumprir seu objetivo.

Essa distinção é importantíssima porque, caso contrário, a gente vai cometer o erro clássico:

"A 6.2 disse que versionamento completo é importante → então vamos implementar versionamento completo na Alpha."

Não. A própria documentação não sustenta isso.

2. O núcleo que emerge da Etapa 5

A Etapa 5 estabeleceu uma cadeia de dependências bastante rígida:

REPRESENTAÇÃO
      ↓
ESTRUTURA
      ↓
SEGMENTAÇÃO
      ↓
RECUPERAÇÃO
      ↓
PROVENIÊNCIA
      ↓
VERSIONAMENTO
      ↓
CONSISTÊNCIA
      ↓
OPERAÇÃO SEGURA

A própria 6.1 reafirma essa dependência: representação determina o que pode ser preservado; estrutura determina as relações documentais; segmentação determina as unidades recuperáveis; recuperação determina o que pode ser encontrado; proveniência determina o que pode ser demonstrado; versionamento determina de qual estado veio; consistência determina se essas relações continuam válidas; operação mantém tudo isso confiável.

Portanto, para a Alpha, não faz sentido pensar:

"Vamos fazer busca e depois vemos o documento."

A Alpha precisa conseguir fazer:

DOCUMENTO
   ↓
PROCESSAMENTO
   ↓
REPRESENTAÇÃO
   ↓
SEGMENTAÇÃO
   ↓
INDEXAÇÃO
   ↓
RECUPERAÇÃO
   ↓
EVIDÊNCIA
   ↓
FONTE

Esse é o vertical slice mínimo.

3. 6.2.1 — Representação Canônica e Processamento

Aqui está uma das partes que eu considero inegociáveis.

A investigação da 6.2.1 consolidou que o PDF continua sendo a fonte primária e que o Snoopy não deve tratar texto linear ou Markdown como representação canônica. A representação canônica deve preservar, em grau suficiente, identidade, páginas, conteúdo, estrutura, posição, ordem, relações, recursos e proveniência, permitindo gerar derivações posteriores sem nova interpretação destrutiva.

🔴 Indispensável

A Alpha precisa possuir uma representação documental mais rica que um simples texto/Markdown.

No mínimo:

Documento
 ├── identidade
 ├── páginas
 ├── elementos/blocos
 ├── conteúdo textual
 ├── localização
 ├── ordem
 └── relações estruturais mínimas

Isso não significa que vamos construir agora o "Canonical Document definitivo".

Significa que o pipeline da Alpha não pode continuar sendo conceitualmente:

PDF → Markdown → chunks

O mínimo correto é:

PDF
 ↓
representação documental
 ↓
derivação textual
 ↓
segmentação
Por quê?

Porque a evidência da Alpha precisa conseguir dizer:

"Este resultado veio desta unidade, que corresponde a este trecho, nesta página, neste documento."

Sem isso, A.2 — Evidência → Contexto → Fonte — vira apenas uma fachada.

🟡 Indispensável como princípio

A representação canônica precisa ser:

independente do parser específico;
separada das representações derivadas;
reprocessável;
identificável;
relacionada à fonte original.

Isso não significa que precisamos implementar agora todos os mecanismos de reprocessamento, versionamento e múltiplos derivados.

A propriedade necessária é:

o parser produz uma representação que pertence ao Snoopy, não uma representação que pertence à biblioteca escolhida.

Isso é particularmente importante porque 6.2.1 rejeitou a ideia de simplesmente escolher Docling, MinerU, PyMuPDF etc. e chamar a saída deles de "Canonical".

4. 6.2.2 — OCR
Aqui existe uma distinção importante.

A documentação não concluiu "use Tesseract", "use Surya", "use PaddleOCR" ou qualquer outro.

Ela concluiu que OCR é uma capacidade condicional do processamento documental.

A investigação também descartou a ideia de um OCR universal obrigatório e reconheceu que OCR especializado, percepção documental, processamento localizado e composição podem ter papéis diferentes.

🔴 Indispensável

A Alpha precisa ser capaz de lidar com:

PDF textual

e, quando necessário:

PDF digitalizado
      ↓
OCR
      ↓
representação textual reconhecida

Ou seja:

OCR quando necessário.

Isso inclusive já aparece no nosso mapa de extração da Alpha.

🟡 Indispensável como arquitetura

O sistema precisa distinguir:

conteúdo nativo
      ≠
conteúdo reconhecido por OCR
      ≠
interpretação estrutural

E o OCR não pode destruir a representação da página original.

⚪ Pós-Alpha

Não precisamos agora:

múltiplos OCRs em paralelo;
fallback sofisticado;
seleção automática por região;
GPU obrigatória;
arbitragem complexa entre OCRs;
estratégia definitiva de OCR para cada classe documental.

A 6.2.2 deliberadamente deixou isso para a etapa de arquitetura técnica.

5. 6.2.3 — Estrutura, Layout, Tabelas e Multimodalidade

Aqui eu faria uma distinção ainda mais cuidadosa.

A documentação concluiu que não existe uma ferramenta única que cubra perfeitamente estrutura, layout, tabelas, figuras, fórmulas etc. A direção tecnológica foi uma composição seletiva, na qual cada componente possui uma função clara.

Mas isso não significa que a Alpha precise implementar toda a multimodalidade.

🔴 Indispensável

A representação Alpha precisa preservar, quando disponíveis e relevantes:

Página
documento → página → unidade
Ordem de leitura

O sistema não pode simplesmente assumir que a ordem dos objetos extraídos é a ordem textual correta.

Estrutura básica

Pelo menos:

títulos;
seções;
parágrafos/blocos;
relações hierárquicas quando disponíveis.
Tabelas

Aqui eu colocaria uma exigência mínima:

uma tabela relevante não pode simplesmente ser achatada e perder silenciosamente sua estrutura.

Não precisamos de um sistema perfeito de tabelas na Alpha.

Mas precisamos de uma representação que permita pelo menos identificar que aquilo é uma tabela e preservar informação suficiente para tratamento posterior.

🟡 Indispensável como princípio

O sistema deve admitir elementos complexos:

texto
tabela
figura
imagem
gráfico
fórmula

mesmo que nem todos tenham processamento sofisticado na Alpha.

E aqui entra um princípio que considero muito importante para C:

degradação graciosa.

Se determinado elemento não puder ser interpretado adequadamente, ele não deve simplesmente desaparecer da representação.

⚪ Pós-Alpha

Podemos deixar para depois:

interpretação multimodal sofisticada;
análise visual de figuras;
compreensão semântica de gráficos;
extração avançada de fórmulas;
arbitragem entre parsers;
múltiplos pipelines especializados;
reconstrução visual completa do documento.
6. 6.2.4 — Persistência da Representação

Essa é outra parte em que a Alpha precisa pegar o núcleo, não a investigação inteira.

A 6.2.4 consolidou:

Google Drive
     ↓
fonte documental original

PostgreSQL/Supabase
     ↓
estado lógico

storage local
     ↓
artefatos derivados

Também estabeleceu que o Snoopy não precisa duplicar permanentemente os PDFs originais e que existência física de um artefato não equivale à sua publicação lógica.

🔴 Indispensável

Para Alpha:

Fonte
Google Drive

continua sendo a origem documental.

Estado lógico
PostgreSQL/Supabase

precisa guardar pelo menos a identidade e o estado dos documentos.

Artefatos

Artefatos derivados que precisarem sobreviver ao processamento devem ter armazenamento persistente adequado.

Relação lógica ↔ físico

Precisamos conseguir dizer:

document_id
   ↓
artefato processado
🔴 E existe um ponto muito importante:

A Alpha já definiu:

PENDING
PROCESSING
ACTIVE
FAILED
REJECTED
REMOVED

Então o princípio da 6.2.4:

existência física ≠ publicação lógica

entra diretamente na Alpha.

Um PDF processado pela metade não pode simplesmente aparecer no índice porque existe um arquivo intermediário no filesystem.

🟡 Indispensável como princípio

Publicação deve ser coerente:

geração
 ↓
armazenamento
 ↓
validação
 ↓
registro
 ↓
ACTIVE

Não precisa ser ainda a arquitetura definitiva de publicação transacional de todos os derivados.

Mas a Alpha não pode publicar estado parcialmente processado.

⚪ Pós-Alpha

Ficam claramente fora:

versionamento documental completo;
snapshots;
genealogia completa;
reprocessamento incremental;
múltiplas versões simultâneas;
migração sofisticada de storage;
object storage;
escalabilidade distribuída.

A própria 6.2.4 não identificou necessidade dessas coisas para a situação atual.

7. 6.2.5 — Segmentação

Aqui temos uma das decisões mais importantes de toda a 6.2.

A conclusão foi:

Algoritmos externos propõem; o Snoopy governa.

O Snoopy deve possuir seu próprio motor de segmentação enquanto autoridade, ainda que utilize bibliotecas externas para determinadas operações.

🔴 Indispensável

A Alpha precisa de uma segmentação que:

1. parta da representação preservada
representação
 ↓
estrutura
 ↓
segmentação

e não:

texto achatado
 ↓
chunk
2. preserve identidade da unidade

Cada unidade precisa ter identidade própria.

Algo conceitualmente como:

document_id
unit_id
location
source elements
3. preserve contexto estrutural

A unidade não deve ser apenas uma string.

Ela deve poder se relacionar com:

título;
seção;
elementos vizinhos;
página;
tabela;
legenda etc.
4. separar unidade de contexto

Isso é fundamental.

UNIDADE
   +
CONTEXTO
   ↓
representação contextualizada
   ↓
embedding / síntese

e não:

"chunk gigante contextualizado"

como se aquilo fosse a própria identidade documental.

🟡 Indispensável como arquitetura

A Alpha deve manter separadas:

document identity
unit identity
index representation

E:

Profile
Run
Unit

devem permanecer conceitualmente distinguíveis.

Mas isso não significa que a Alpha precise ter múltiplos perfis funcionando.

Podemos ter:

AlphaProfile
    ↓
Run
    ↓
Units

e deixar múltiplos perfis para depois.

⚪ Pós-Alpha
múltiplos perfis;
comparação de segmentações;
reprocessamento sofisticado;
múltiplos algoritmos concorrentes;
avaliação empírica extensa;
MMR/estratégias avançadas de contexto;
segmentação especializada para todos os elementos.
8. 6.2.6 — Recuperação

Aqui a coisa fica mais interessante porque uma parte substancial já foi absorvida pela Trilha A.

A 6.2.6 chegou a uma arquitetura funcional:

consulta
   ↓
filtros estruturados
   ↓
lexical + semântica
   ↓
fusion
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
evidência

E explicitamente considerou PostgreSQL + FTS + pgvector uma alternativa arquitetural real, não apenas provisória.

🔴 Indispensável

Para Alpha eu considero:

Busca estruturada

Filtros precisam ser diferentes de relevância textual.

Ex.:

autor = X
idioma = francês

não são simplesmente termos para embedding.

Busca lexical

Necessária.

Busca semântica

Necessária.

Recuperação híbrida

A Alpha deve ser capaz de combinar os sinais.

Deduplicação por identidade

Se:

U123
← busca lexical

U123
← busca semântica

isso não pode virar duas evidências independentes.

Contextualização

Necessária para a evidência e para a qualidade da seleção.

Reranking

Eu colocaria como indispensável para a arquitetura Alpha, mas não necessariamente com toda a sofisticação futura.

A própria investigação separa:

retrieval
≠
reranking

e o nosso mapa da Alpha já previa explicitamente reranking.

Seleção

Ranking não é seleção.

A Alpha precisa de uma etapa que determine quais candidatos realmente chegam ao pesquisador/síntese.

🟡 Indispensável como princípio

A recuperação precisa ser reconstruível.

Ou seja, deve ser possível saber minimamente:

consulta
 ↓
perfil/configuração
 ↓
corpus
 ↓
unidades
 ↓
resultado

Não precisamos ainda registrar toda a genealogia da execução.

Mas não podemos ter uma caixa-preta:

"o embedding achou isso".

⚪ Pós-Alpha

A 6.2 deixa deliberadamente para depois:

RRF como escolha definitiva;
múltiplos rerankers;
MMR;
decomposição sofisticada;
múltiplos motores de busca;
estratégias específicas por tipo de consulta;
avaliação extensa de recall/precision;
reprodução histórica completa de cada busca.
9. 6.2.7 — Proveniência e Versionamento

Aqui mora provavelmente a maior armadilha da compilação.

A 6.2.7 investigou uma arquitetura de proveniência bastante sofisticada:

Entity
State
Activity/Run
Derivation

com corpus snapshots, versões, temporalidade etc.

Isso não significa que tudo isso seja Alpha.

A Trilha A já definiu a proveniência mínima necessária.

🔴 Indispensável

A Alpha precisa conseguir reconstruir:

evidence
   ↓
unit
   ↓
document
   ↓
location
   ↓
original source

Ou, em termos mínimos:

document_id
unit_id
location
source

E contexto suficiente para abrir a evidência e chegar à fonte.

Isso é exatamente o que A.2 já consolidou.

🔴 Também precisa existir identidade de execução

A busca/processing precisa possuir uma identidade operacional.

Não necessariamente toda a ontologia completa de Activity/Derivation, mas pelo menos algo equivalente a:

operation_id

para que possamos saber que determinada recuperação/processamento ocorreu.

Isso conversa diretamente com B.5.

🟡 Indispensável como princípio

A Alpha deve separar:

IDENTIDADE
≠
ESTADO
≠
CONTEÚDO
≠
PROVENIÊNCIA

E também:

proveniência documental
≠
proveniência da recuperação

Essa distinção não é "feature futura". É arquitetura.

⚪ Pós-Alpha

Aqui eu cortaria sem dó:

corpus snapshots;
temporalidade completa;
auditoria bidirecional completa;
grafo de proveniência completo;
W3C PROV como implementação;
OpenLineage;
event sourcing;
histórico integral;
exportação de provenance.

Tudo isso foi investigado. Mas não é necessário para a Alpha.

10. 6.2.8 — Segurança, Operação e Resiliência

Essa etapa é traiçoeira porque possui decisões tecnológicas já consolidadas.

Mas nem todas são necessárias no mesmo grau.

🔴 Indispensável
Autenticação

A Alpha precisa saber quem está operando.

A 6.2.8 consolidou Supabase Auth.

Autorização no backend

Isso é obrigatório.

Não:

frontend decide

mas:

backend autoriza
Isolamento por acervo/recurso

O corpus é uma fronteira de segurança.

RLS como defesa adicional

Não substitui autorização de domínio.

Jobs

Processamento documental não deve depender de uma requisição HTTP longa.

Precisamos de:

job
 ↓
queue
 ↓
worker
 ↓
processing
 ↓
publication
Idempotência

Se um job for reentregue:

mesmo documento
+
mesma operação

não pode corromper o estado.

Falha explícita

Precisamos distinguir:

PROCESSING
FAILED
ACTIVE

etc.

Publicação somente de estado válido

Isso atravessa 5.5 + 6.2.4 + 6.2.8.

🟡 Indispensável como arquitetura

A investigação escolheu:

PGMQ / Supabase Queues
+
Python workers

para V3.

Isso já é uma decisão tecnológica suficientemente consolidada para entrar em C.

Mas os detalhes de:

retry;
visibility timeout;
attempts;
run;
constraints;
tempos;

podem ser decididos na implementação arquitetural.

⚪ Pós-Alpha

Não vejo motivo para colocar na Alpha:

Kubernetes;
Redis/BullMQ;
múltiplos servidores;
autoscaling;
multi-region;
Prometheus;
Grafana;
ELK;
service mesh;
disaster recovery sofisticado;
RPO/RTO formal;
infraestrutura distribuída.

A própria 6.2.8 deliberadamente não encontrou requisito para isso na V3 inicial.
11. Então o que realmente sobra?

Depois de passar 5 → 6.1 → 6.2 pelo filtro da Alpha A+B, eu chegaria a este inventário preliminar:

Domínio	Indispensável na Alpha	Grau
Fonte documental	Google Drive como fonte original	🔴
Identidade documental	document_id + identidade externa	🔴
Processamento	pipeline documental assíncrono	🔴
Representação	representação documental estruturada	🔴
Página/localização	página + localização suficiente	🔴
Ordem de leitura	preservada/representável	🔴
Estrutura básica	blocos + hierarquia relevante	🔴
OCR	quando necessário	🔴
Tabelas	não destruir silenciosamente estrutura	🔴
Elementos complexos	representáveis, mesmo que sem processamento completo	🟡
Persistência lógica	PostgreSQL/Supabase	🔴
Artefatos	storage local persistente quando necessário	🔴
Estados documentais	PENDING/PROCESSING/ACTIVE/FAILED/REJECTED/REMOVED	🔴
Publicação válida	não publicar processamento parcial	🔴
Segmentação estrutural	sim	🔴
Refinamento semântico	sim, em grau mínimo	🔴
Unidade identificável	unit_id	🔴
Contexto estrutural	separado da unidade	🔴
Proveniência da unidade	unidade → elemento/página/documento	🔴
Perfil/run	identidade mínima	🟡
Busca lexical	sim	🔴
Busca semântica	sim	🔴
Busca híbrida	sim	🔴
Filtros estruturados	sim	🔴
Deduplicação	por identidade	🔴
Contextualização	sim	🔴
Reranking	sim	🔴
Seleção	separada do ranking	🔴
Evidência	objeto identificável	🔴
Proveniência mínima	evidence → unit → document → source	🔴
Identidade da operação	operation_id	🔴
Versionamento completo	não	⚪
Corpus snapshots	não	⚪
Proveniência em grafo completa	não	⚪
Event sourcing	não	⚪
OCR múltiplo/fallback sofisticado	não	⚪
Multimodalidade avançada	não	⚪
Múltiplos perfis de segmentação	não	⚪
Múltiplos rerankers	não	⚪
Busca distribuída especializada	não	⚪
Kubernetes/Redis etc.	não	⚪
12. Mas tem uma coisa ainda mais importante: a Alpha não precisa "implementar" tudo isso do mesmo jeito

Aqui está a distinção que eu acho que deve entrar explicitamente no registro da compilação.

Temos três tipos de requisito:

A. Capacidade funcional

Tem que funcionar na Alpha.

Ex.:

PDF
→ processamento
→ unidade
→ busca
→ evidência
→ fonte
B. Invariante arquitetural

Precisa ser respeitado, mas pode ter implementação mínima.

Ex.:

document_id ≠ unit_id

ou:

SSE ≠ fonte de verdade

ou:

conteúdo derivado ≠ documento
C. Capacidade futura

Foi investigada porque faz parte do Snoopy, mas não entra na Alpha.

Ex.:

Corpus Snapshot
Temporalidade completa
Event sourcing
Múltiplos perfis

Essa separação evita que C vire uma monstruosidade.

13. O que eu considero o verdadeiro "núcleo vertical" da Alpha

Se eu tivesse que representar tudo isso em uma única cadeia técnica, seria:

                    GOOGLE DRIVE
                         │
                         ▼
                  DOCUMENTO FONTE
                         │
                         ▼
                PROCESSAMENTO ASYNC
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
        REPRESENTAÇÃO          ARTEFATOS
        ESTRUTURADA             FÍSICOS
              │
              ├── páginas
              ├── elementos
              ├── estrutura
              ├── localização
              └── OCR quando necessário
                         │
                         ▼
                  SEGMENTAÇÃO
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
           UNIT_ID              CONTEXTO
              │                     │
              └──────────┬──────────┘
                         ▼
                     INDEXAÇÃO
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
           LEXICAL              SEMÂNTICA
              │                     │
              └──────────┬──────────┘
                         ▼
                      FUSION
                         │
                         ▼
                    CANDIDATOS
                         │
                         ▼
                  DEDUPLICAÇÃO
                         │
                         ▼
                  CONTEXTUALIZAÇÃO
                         │
                         ▼
                    RERANKING
                         │
                         ▼
                     SELEÇÃO
                         │
                         ▼
                    EVIDÊNCIA
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
           CONTEXTO              PROVENIÊNCIA
                                    │
                                    ▼
                             DOCUMENTO / PÁGINA
                                    │
                                    ▼
                              READING / FONTE

E transversalmente:

IDENTIDADE
SEGURANÇA
ESTADOS
JOBS
PERSISTÊNCIA
PUBLICAÇÃO
IDEMPOTÊNCIA
PROVENIÊNCIA
14. E isso revela uma coisa interessante sobre a Alpha

A Alpha não é simplesmente:

"V2 com interface nova".

Mas também não é:

"Snoopy V3 definitivo em miniatura".

Ela é uma espécie de prova vertical dos princípios arquiteturais fundamentais.

A pergunta da Alpha passa a ser:

Conseguimos pegar um documento real, preservá-lo suficientemente, derivar unidades sem destruí-lo, recuperá-las, apresentar uma evidência identificável, voltar à fonte e manter todo esse percurso coerente diante de processamento assíncrono e falhas?

Se a resposta for sim, temos uma Alpha que realmente demonstra o salto arquitetural da V3.

Se ela só fizer:

PDF → texto → embedding → resposta

com uma interface bonita em Vue, teremos trocado a carroceria e mantido o motor conceitual da V2.

E isso seria exatamente o que essa compilação precisa impedir.

Minha proposta para o próximo passo

Eu não fecharia ainda esse inventário como decisão definitiva.

Acho que chegamos ao ponto certo para fazer uma segunda passada, agora muito mais cirúrgica:

pegar cada item acima e confrontá-lo literalmente com os requisitos da 5 e os critérios correspondentes da 6.1, perguntando:

"Se removermos isso da Alpha, qual requisito já consolidado deixa de ser satisfeito?"

Isso vai nos permitir separar o que é realmente 🔴 indispensável do que eu classifiquei como indispensável por inferência arquitetural.

Depois dessa segunda passada, aí sim podemos produzir o documento:

“Inventário de Requisitos Indispensáveis para a V3 Alpha — Extração das Etapas 5, 6.1 e 6.2”

que será a ponte direta para a Trilha C — Arquitetura Alpha Integrada.

E acho que essa segunda passada vale a pena. É justamente onde a gente evita enfiar na Alpha cinco toneladas de arquitetura que pertencem à mansão inteira.

Em relação ao OCR, o OCR não vai entrar na Alpha. Ele não vai entrar na Alpha porque vai dar um certo trabalho pra gente configurar, então ele não vai entrar na Alpha. Ele fica pra pós-Alpha. Não porque ele não é importante, ele é algo básico, mas a gente não vai ter tempo pra implementar ele.

Cara, a discussão que tu propôs, ela é uma discussão válida. Só que a gente não tem tempo. A gente não tem tempo pra ficar discutindo várias coisas. Tem que ser o mais enxuto possível, no sentido da discussão, no caso. Tipo, pegar o que é de suma importância pra deixar e que dê pra mostrar, entendeu? Suma importância, o que é realmente importante, o que é fundamental para a V3. É isso que a gente precisa implementar, entendeu? Porque aí, depois de definir as coisas da etapa 6.2 que vão ir também, a gente ainda tem a etapa C e depois a gente tem a etapa D. E aí que eu vou implementar. Já é 1 hora e 06 da manhã. Então bom, a gente... tipo, seria uma discussão muito boa pra ser feita se a gente tivesse tempo. Essa é a questão. Mas não tem. Esse é o problema. Então bom, o que a gente vai implementar, tipo, tudo que tu colocou ali, de fato tá tudo certo. Só que o que a gente vai ter que implementar é aquilo que é fundamental para a V3. E lembra, a gente vai apresentar apenas uma maquete, apenas a... Isso não quer dizer que tem que ser feito às coxas. Tem que ser bem feito. Por isso a gente vai adicionar o que é fundamental e não só mascarar a V2 atual pra deixar bonitinho, entendeu? O teu é, a gente precisa daquilo que é fundamental para a V3. E aí depois dessa apresentação de terça-feira, aí a gente começa a discutir mais a fundo. Eu vou te contextualizar, te treinar tal qual o outro chat tava treinado, e aí a gente volta de fato pra discussão pesada de implementação de tudo, entendeu? Mas no momento a gente tá correndo contra o tempo. Esse que é o problema. Não tô dizendo pra tu correr e se apressar demais. Não. Tô dizendo que a gente tem que trabalhar com o que é fundamental pro surgimento da V3. Não pra V3 deixar de ser um conceito e surgir na prática.

Sim. Agora ficou **muito mais claro qual é o critério de corte** — e isso muda a forma como devemos fazer essa extração.

Não precisamos fazer agora a auditoria epistemológica completa de tudo que a 5/6.1/6.2 exigiria numa arquitetura definitiva. Isso seria ótimo depois da terça, mas **é exatamente o tipo de discussão que agora consumiria o tempo que precisamos para a C e a D**.

O critério da extração deve ser simplesmente:

> **“O que da Etapa 5 + 6.1 + 6.2 é fundamental para que a V3 Alpha deixe de ser apenas uma evolução visual da V2 e passe a demonstrar, ainda que em maquete funcional, os princípios arquiteturais centrais da V3?”**

E, principalmente:

> **“Se tirarmos isso agora, a apresentação deixa de demonstrar alguma mudança fundamental da V3?”**

Se a resposta for não, **pós-Alpha**.

### Sobre OCR: fechado

**OCR fica fora da Alpha.**

Não porque deixou de ser requisito importante — ele continua sendo uma capacidade fundamental do Snoopy definitivo — mas porque **não é fundamental para o surgimento demonstrável da V3 na terça-feira**, considerando o tempo disponível.

Então:

**OCR → pós-Alpha.**

E isso é perfeitamente coerente com a ideia de Alpha como recorte temporal. Não precisamos transformar “fundamental para o Snoopy” em “obrigatório para a Alpha”.

---

## Então eu mudaria o nosso corte

Eu não colocaria mais aquela tabela anterior como “tudo que é indispensável”. Ela estava muito próxima de uma **arquitetura mínima do Snoopy**, e não de uma **Alpha mínima da V3**.

Para a Alpha, eu separaria brutalmente:

### 🔴 Fundamental para o surgimento da V3

Coisas que precisam aparecer **na implementação**, mesmo que em versão mínima:

* **Representação documental estruturada**, em vez de simplesmente PDF → Markdown → chunk.
* **Preservação mínima de identidade, página/localização e estrutura** do documento.
* **Segmentação governada pelo Snoopy**, com unidades identificáveis.
* **Recuperação que reflita a arquitetura definida**:

  * busca lexical;
  * busca semântica;
  * combinação/híbrida;
  * deduplicação;
  * contextualização;
  * reranking;
  * seleção.
* **Evidência como objeto identificável**, não apenas um pedaço de texto jogado na resposta.
* **Relação evidência → unidade → documento → localização → fonte**.
* **Corpus/documentos com estados de processamento**, para que um documento não seja simplesmente “jogado no banco e pronto”.
* **Processamento assíncrono**, na medida em que isso foi consolidado como parte do modelo operacional.
* **Separação entre estado persistente e apresentação**, já estabelecida na B.
* **Structured Response → Renderer → interface**, porque isso é uma mudança arquitetural real em relação à simples renderização textual.
* **Investigação → evidência → fonte/leitura → retorno à investigação**, que já foi fechado em A e B.

Isso é o **núcleo da demonstração da V3**.

---

### 🟡 Fundamental como princípio, mas implementação pode ser mínima

Aqui entram coisas que **não podem ser violadas**, mas não precisamos construir toda a solução definitiva agora:

* distinção entre identidade do documento, unidade de segmentação e representação/indexação;
* separação entre unidade e contexto;
* backend como autoridade sobre estado persistente;
* `operation_id` para operações assíncronas;
* distinção entre falha da operação e perda de comunicação;
* publicação somente de processamento válido/completo;
* idempotência básica;
* autorização no backend;
* conteúdo documental/modelado tratado como dado não confiável;
* estrutura preparada para crescimento posterior sem amarrar o domínio ao framework Vue.

Ou seja: **não precisamos implementar “a mansão”, mas também não podemos construir a casinha de modo que tenhamos que demolir a fundação depois.**

---

### ⚪ Pós-Alpha

E aqui podemos cortar sem dó:

* OCR;
* versionamento completo;
* snapshots;
* histórico documental completo;
* grafo completo de proveniência;
* múltiplas versões do documento;
* múltiplos perfis de segmentação;
* comparação entre estratégias de segmentação;
* múltiplos rerankers;
* avaliação sistemática de recall/precision;
* MMR e outras sofisticações de recuperação;
* decomposição sofisticada de consultas;
* multimodalidade avançada;
* compreensão de gráficos/imagens/fórmulas;
* Reading Mode definitivo;
* colaboração;
* múltiplas investigações persistentes;
* infraestrutura distribuída;
* observabilidade avançada;
* offline-first;
* etc.

E **não precisamos discutir agora se cada uma delas seria arquiteturalmente desejável**. Já sabemos que ficam para depois.

---

## O ponto mais importante

A Alpha não precisa provar que **“o Snoopy inteiro funciona”**.

Ela precisa provar que existe uma **nova espinha dorsal**:

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

E isso precisa estar conectado àquilo que acabamos de fechar em A e B:

```text
                    ┌──────────────┐
                    │   CORPUS     │
                    └──────┬───────┘
                           ↓
                    processamento
                           ↓
                    representação
                           ↓
                      segmentação
                           ↓
                     indexação
                           ↓
                    ┌──────────────┐
                    │ INVESTIGAÇÃO │
                    └──────┬───────┘
                           ↓
                      recuperação
                           ↓
                       evidência
                           ↓
                     contextualização
                           ↓
                         fonte
                           ↓
                       leitura
                           ↓
                    volta à investigação
```

**Isso é o que diferencia a V3 da V2.**

Se fizermos apenas uma interface bonita em Vue em cima do mecanismo antigo, teremos uma **V2 maquiada**.

Se tentarmos implementar toda a arquitetura definitiva agora, não chegamos à terça.

Então o corte correto é justamente o que tu acabou de estabelecer:

> **Implementar o mínimo que materializa a arquitetura nova. Não o mínimo que faz uma demonstração parecer nova.**

E acho que esse deve ser o nosso princípio operacional daqui até a apresentação:

> **A Alpha não precisa conter tudo que o Snoopy será. Ela precisa conter aquilo sem o qual a V3 ainda não teria realmente começado a existir.**

Com isso, eu **não faria outra rodada de discussão geral**. O próximo passo é pegar os registros da **5, 6.1 e 6.2** e extrair diretamente deles esse conjunto enxuto. A partir daí fechamos o inventário, vamos para **C — Arquitetura Alpha Integrada**, e depois para **D — mapa de implementação**.


Ah, então vamos lá, cara. Vamos fazer isso pra gente poder caminhar pra conclusão dessa etapa e poder passar pra C e pra D, que serão as mais complicadas. Então vamos lá. Tipo, agora que a gente já definiu o escopo, a gente pode ter mais calma, porque com esse escopo menor a gente pode ter um pouco mais de calma.

É isso. Agora que o critério de corte está fechado, dá para fazer a extração com calma sem cair na armadilha de tentar reconstruir o Snoopy inteiro às 2 da manhã. 😂

Fiz o cruzamento tomando como base os Registros Mestres da Etapa 5, a tradução da 6.1 e as oito investigações da 6.2. O repositório oficial é este: snoopy-rag-documentation

E vou aplicar uma regra bem mais rígida:

Não entra na Alpha aquilo que é apenas importante para o Snoopy definitivo. Entra aquilo que é necessário para materializar a mudança arquitetural que faz a V3 existir.

Inventário de requisitos indispensáveis para a V3 Alpha
1. O núcleo que realmente precisa entrar

Depois do corte, eu vejo seis blocos fundamentais vindos da Etapa 5/6.2:

1. Representação documental mínima
            ↓
2. Segmentação governada
            ↓
3. Recuperação híbrida
            ↓
4. Evidência + proveniência mínima
            ↓
5. Processamento e publicação controlados
            ↓
6. Persistência/estado do corpus

Eles não têm exatamente o mesmo peso. Alguns são funcionalidades que precisam aparecer; outros são princípios que precisam estar presentes na arquitetura, ainda que de forma mínima.

1. Representação documental mínima
🔴 Entra na Alpha

Esse é provavelmente o ponto mais importante vindo da 5.1.

A V3 não pode continuar conceitualmente como:

PDF
 ↓
texto/Markdown
 ↓
chunks
 ↓
embeddings

A mudança fundamental da V3 é:

DOCUMENTO
 ↓
REPRESENTAÇÃO DOCUMENTAL
 ↓
DERIVAÇÕES
 ↓
SEGMENTAÇÃO / RECUPERAÇÃO

Isso está no centro da Etapa 5.1 e é reiterado pela 6.2.1.

O que precisamos efetivamente ter

Uma representação suficientemente estruturada para manter, no mínimo:

identidade do documento;
relação com a fonte original;
páginas;
elementos/blocos;
conteúdo textual;
ordem/localização;
estrutura mínima necessária para relacionar uma unidade recuperada ao documento.

Algo conceitualmente próximo de:

Documento
 ├── identidade
 ├── fonte
 └── páginas
      ├── bloco
      ├── bloco
      ├── bloco
      └── ...

Não precisamos implementar toda a representação canônica definitiva.

Não precisamos agora preservar perfeitamente:

coordenadas completas;
todos os elementos visuais;
relações multimodais complexas;
reconstrução perfeita do PDF;
todas as características tipográficas.

Mas a arquitetura precisa abandonar a ideia de que o texto linear é o próprio documento.

O que fica fora
representação canônica completa;
preservação multimodal completa;
reconstrução visual perfeita;
otimização definitiva do modelo documental.
2. OCR
⚪ Pós-Alpha

Aqui está fechado por decisão de escopo:

OCR não será implementado na V3 Alpha.

Isso não contradiz a 5.1/6.2.2.

A 6.2.2 estabelece OCR como capacidade importante para o Snoopy, especialmente para documentos digitalizados. Mas a Alpha não precisa implementar todas as capacidades fundamentais do sistema definitivo.

Portanto:

PDF textual
   ↓
representação
   ↓
Alpha

é suficiente.

Documentos que dependam de OCR ficam para depois.

E tem uma distinção importante: não devemos remover OCR do modelo arquitetural do Snoopy. Só não vamos implementá-lo agora.

3. Estrutura e layout

Aqui eu faria um corte importante em relação à minha resposta anterior.

🔴 Entra — mas em nível mínimo

A Alpha precisa preservar:

página;
ordem de leitura;
blocos;
estrutura textual básica.

Isso é necessário para que a unidade recuperada consiga voltar ao documento de maneira significativa.

Mas:

🟡 Tabelas, imagens, gráficos etc.

Não precisamos implementar agora um pipeline multimodal completo.

A exigência para a Alpha é mais simples:

A representação não deve ser arquitetada de forma que elementos documentais complexos sejam conceitualmente impossíveis de preservar depois.

Por exemplo, não precisamos implementar hoje:

PDF
 ↓
detecção perfeita de tabela
 ↓
estrutura de células
 ↓
relações semânticas
 ↓
indexação multimodal

Mas também não devemos construir uma representação que seja simplesmente:

PDF → string gigante

e depois descobrir que não existe lugar para uma tabela.

Portanto:
Capacidade	Alpha
Página	🔴
Ordem de leitura	🔴
Blocos	🔴
Estrutura textual básica	🔴
Tabela plenamente estruturada	⚪
Imagem compreendida	⚪
Gráfico interpretado	⚪
Fórmula multimodal	⚪
Relações multimodais avançadas	⚪

Isso é muito mais compatível com o nosso objetivo de terça.

4. Persistência e estado documental
🔴 Entra

A V3 precisa ter uma distinção real entre:

documento existe
        ≠
documento foi processado
        ≠
documento está disponível para recuperação

Esse ponto aparece tanto na 5.5 quanto na 6.2.4 e 6.2.8.

Precisamos, portanto, de um estado mínimo do documento.

Já fechamos em A:

PENDING
PROCESSING
ACTIVE
FAILED
REJECTED
REMOVED

E:

Somente ACTIVE participa da recuperação.

Isso é importante demais para ser apenas “coisa de produção”. É justamente uma das diferenças entre uma arquitetura documental e simplesmente colocar arquivos numa pasta e gerar embeddings.

Também entra

A distinção:

fonte original
      ≠
estado lógico
      ≠
artefato derivado

Na arquitetura consolidada:

Google Drive
    ↓
fonte documental

PostgreSQL/Supabase
    ↓
estado lógico

armazenamento físico
    ↓
artefatos derivados

A forma concreta como isso será implementado fica para C.

5. Processamento controlado
3. Estrutura e layout

Aqui eu faria um corte importante em relação à minha resposta anterior.

🔴 Entra — mas em nível mínimo

A Alpha precisa preservar:

página;
ordem de leitura;
blocos;
estrutura textual básica.

Isso é necessário para que a unidade recuperada consiga voltar ao documento de maneira significativa.

Mas:

🟡 Tabelas, imagens, gráficos etc.

Não precisamos implementar agora um pipeline multimodal completo.

A exigência para a Alpha é mais simples:

A representação não deve ser arquitetada de forma que elementos documentais complexos sejam conceitualmente impossíveis de preservar depois.

Por exemplo, não precisamos implementar hoje:

PDF
 ↓
detecção perfeita de tabela
 ↓
estrutura de células
 ↓
relações semânticas
 ↓
indexação multimodal

Mas também não devemos construir uma representação que seja simplesmente:

PDF → string gigante

e depois descobrir que não existe lugar para uma tabela.

Portanto:
Capacidade	Alpha
Página	🔴
Ordem de leitura	🔴
Blocos	🔴
Estrutura textual básica	🔴
Tabela plenamente estruturada	⚪
Imagem compreendida	⚪
Gráfico interpretado	⚪
Fórmula multimodal	⚪
Relações multimodais avançadas	⚪

Isso é muito mais compatível com o nosso objetivo de terça.

4. Persistência e estado documental
🔴 Entra

A V3 precisa ter uma distinção real entre:

documento existe
        ≠
documento foi processado
        ≠
documento está disponível para recuperação

Esse ponto aparece tanto na 5.5 quanto na 6.2.4 e 6.2.8.

Precisamos, portanto, de um estado mínimo do documento.

Já fechamos em A:

PENDING
PROCESSING
ACTIVE
FAILED
REJECTED
REMOVED

E:

Somente ACTIVE participa da recuperação.

Isso é importante demais para ser apenas “coisa de produção”. É justamente uma das diferenças entre uma arquitetura documental e simplesmente colocar arquivos numa pasta e gerar embeddings.

Também entra

A distinção:

fonte original
      ≠
estado lógico
      ≠
artefato derivado

Na arquitetura consolidada:

Google Drive
    ↓
fonte documental

PostgreSQL/Supabase
    ↓
estado lógico

armazenamento físico
    ↓
artefatos derivados

A forma concreta como isso será implementado fica para C.

5. Processamento controlado
🔴 Entra

A Alpha precisa demonstrar que processamento documental é uma operação, não simplesmente uma função que recebe um PDF e magicamente retorna chunks.

A cadeia mínima é:

documento
 ↓
job/operação
 ↓
processamento
 ↓
validação
 ↓
publicação
 ↓
ACTIVE

Isso conversa diretamente com 6.2.8.

Fundamental:
estados explícitos;
processamento identificável;
falha explícita;
não publicar processamento incompleto;
possibilidade de reprocessamento seguro em algum nível.
Sobre fila/worker

Aqui precisamos ser cuidadosos.

A investigação 6.2.8 consolidou:

PGMQ / Supabase Queues
        ↓
Python workers

como direção arquitetural para processamento assíncrono.

E a Alpha já estabeleceu que processamento deve ser assíncrono.

Então:

🔴 O princípio assíncrono entra.
🟡 A implementação exata da infraestrutura será decidida em C.

Ou seja, não devemos agora transformar “PGMQ” em requisito metodológico da Alpha.

C vai pegar:

processamento assíncrono + operação identificável + estados + publicação segura

e transformar isso na arquitetura concreta.

6. Segmentação
🔴 Entra integralmente no sentido arquitetural

Esse é outro dos pontos que não pode ser simplesmente herdado da V2.

A Etapa 5.2 e a 6.2.5 estabelecem a separação entre:

DOCUMENTO
      ↓
ELEMENTOS / ESTRUTURA
      ↓
SEGMENTAÇÃO
      ↓
UNIDADES DE RECUPERAÇÃO

A unidade de recuperação precisa ter identidade própria.

Algo conceitualmente como:

document_id
     ↓
unit_id
     ↓
localização/contexto
Fundamentalmente precisamos preservar:
identidade da unidade;
relação unidade → documento;
localização;
contexto estrutural;
distinção entre unidade e contexto.

E permanece vigente o princípio:

Algoritmos externos propõem; o Snoopy governa.

Ou seja, podemos usar uma biblioteca/algoritmo de segmentação.

O que não podemos fazer é deixar a biblioteca definir silenciosamente o modelo documental do Snoopy.

O que NÃO entra

Não precisamos agora:

múltiplos perfis de segmentação;
comparação de algoritmos;
experimentação sistemática de chunk sizes;
múltiplas estratégias simultâneas;
avaliação estatística completa;
resegmentação sofisticada;
MMR como extensão da segmentação.
Uma segmentação funcional e coerente basta.
7. Recuperação

Aqui está um dos blocos que mais claramente precisa entrar, porque é parte do que a V3 efetivamente pretende demonstrar.

🔴 Entra

A cadeia mínima consolidada é:

consulta
 ↓
filtros
 ↓
recuperação lexical
 +
recuperação semântica
 ↓
combinação
 ↓
deduplicação
 ↓
contextualização
 ↓
reranking
 ↓
seleção
 ↓
evidência

Não precisamos transformar isso numa plataforma de busca gigantesca.

Mas precisamos demonstrar que a recuperação da V3 possui essa arquitetura conceitual.

Portanto entram:

Busca lexical 🔴
Busca semântica 🔴
Recuperação híbrida 🔴
Filtros estruturados 🔴
Deduplicação 🔴
Contextualização 🔴
Reranking 🔴
Seleção 🔴

Isso é importante porque não queremos chamar simplesmente:

vector_search()

e colocar o resultado na tela.

A V3 deve demonstrar a separação:

recuperar candidatos
        ≠
ranquear candidatos
        ≠
selecionar evidências
O que não entra
Não precisamos agora:

vários rerankers;
comparação experimental entre rerankers;
MMR;
múltiplos motores de busca;
decomposição sofisticada de consultas;
avaliação completa de precision/recall;
sistema de recuperação altamente configurável.

E aqui também vale uma distinção:

Reranking entra como capacidade; o reranker específico entra em C.

8. Evidência
🔴 Entra

Isso é absolutamente central para a V3.

Uma coisa recuperada não pode ser apenas:

"texto do chunk"

A evidência precisa ser identificável.

Conceitualmente:

Evidence
 ├── evidence_id
 ├── unit_id
 ├── context
 ├── document_id
 └── location

E isso conversa diretamente com o que já fechamos na Alpha A.

9. Proveniência mínima
🔴 Entra

Mas somente a mínima necessária para demonstrar rastreabilidade.

Não precisamos construir agora o grande grafo de proveniência da arquitetura definitiva.

Precisamos conseguir fazer:

EVIDÊNCIA
   ↓
UNIDADE
   ↓
DOCUMENTO
   ↓
LOCALIZAÇÃO
   ↓
FONTE ORIGINAL

Isso é o mínimo necessário para cumprir uma das mudanças fundamentais do Snoopy.

A pessoa vê uma evidência e consegue perguntar:

“Isso veio de onde?”

E o sistema consegue responder.

Isso é muito diferente de simplesmente apresentar uma síntese com uma referência textual decorativa.

10. Versionamento
⚪ Pós-Alpha

Aqui o corte é bem tranquilo.

A 5.5/6.2.7 estabelecem versionamento como parte importante da arquitetura definitiva.

Mas a própria Alpha A já estabeleceu:

Full versioning fica fora da Alpha.

Então:

document_id
+
estado atual
+
fonte

sim.

Sistema completo de:

v1
v2
v3
snapshots
histórico
derivação entre versões

não.

11. Segurança

Aqui também precisamos separar princípio de sistema de segurança completo.

🔴 Entra

Da 5.6/6.2.8, o fundamental para a Alpha é:

autenticação;
backend como autoridade de autorização;
conteúdo documental tratado como dado não confiável;
isolamento mínimo do corpus;
nenhuma confiança em identificadores fornecidos pelo frontend;
RLS como camada complementar quando aplicável.

Isso já conversa com B.

⚪ Não entra

Não precisamos agora:

RBAC + ReBAC + ABAC completo;
sistema sofisticado de compartilhamento;
políticas complexas entre múltiplos usuários;
threat modeling completo;
pentest;
infraestrutura de segurança avançada;
CSP sofisticada;
auditoria de segurança.

A Alpha precisa ser corretamente segura em suas fronteiras fundamentais, não uma plataforma corporativa de segurança.

12. Resiliência operacional
🟡 Entra como princípio mínimo

A Alpha já decidiu em B que precisamos distinguir:

PROCESSANDO
      ≠
FALHOU
      ≠
CANCELADO
      ≠
PERDEU CONEXÃO

Isso é reforçado pela 6.2.8.

Portanto precisamos de:

estados explícitos;
falhas explícitas;
publicação atômica;
não considerar comunicação perdida como falha confirmada;
recuperação/revalidação do estado.

Mas não precisamos implementar agora toda a infraestrutura de resiliência da 6.2.8.

Nada de:

disaster recovery completo;
backup/restore testado;
múltiplos servidores;
failover;
autoscaling;
observabilidade distribuída;
alertas sofisticados.
Então o inventário final fica assim
Área	Alpha	O que efetivamente entra
Representação documental	🔴	representação estruturada mínima
Identidade documental	🔴	document_id + relação com fonte
Página/localização	🔴	localização suficiente para voltar à fonte
Ordem/estrutura	🔴	páginas, blocos, ordem e estrutura básica
OCR	⚪	pós-Alpha
Tabelas	🟡	arquitetura não pode impedir preservação futura
Imagens/gráficos	🟡	arquitetura preparada; processamento avançado depois
Persistência lógica	🔴	estado documental persistente
Estados documentais	🔴	PENDING → PROCESSING → ACTIVE/FAILED/etc.
Processamento assíncrono	🔴	operação de processamento independente
Queue/worker específico	🟡	decisão concreta em C
Segmentação	🔴	unidades identificáveis e governadas
Contexto	🔴	contexto separado da unidade
Perfis múltiplos	⚪	pós-Alpha
Busca lexical	🔴	entra
Busca semântica	🔴	entra
Busca híbrida	🔴	entra
Filtros estruturados	🔴	entra
Deduplicação	🔴	entra
Contextualização	🔴	entra
Reranking	🔴	entra
Múltiplos rerankers	⚪	pós-Alpha
Seleção	🔴	separada de ranking
Evidência identificável	🔴	entra
Proveniência mínima	🔴	evidência → unidade → documento → localização → fonte
Proveniência completa	⚪	pós-Alpha
Versionamento completo	⚪	pós-Alpha
Autenticação	🔴	mínimo funcional
Autorização backend	🔴	entra
Modelo sofisticado de autorização	⚪	pós-Alpha
RLS	🟡	camada complementar, conforme arquitetura C
Estados/falhas	🔴	entra
Idempotência	🟡	princípio mínimo
Publicação atômica	🟡	princípio; implementação em C
Observabilidade avançada	⚪	pós-Alpha
Disaster recovery	⚪	pós-Alpha
Multimodalidade avançada	⚪	pós-Alpha

E aqui está, para mim, a verdadeira fronteira da Alpha

Se reduzirmos tudo ao que precisa efetivamente existir para podermos dizer “a V3 nasceu”, temos:

                         V3 ALPHA
                            │
            ┌───────────────┴────────────────┐
            │                                │
       DOCUMENTO                         INVESTIGAÇÃO
            │                                │
            ▼                                ▼
   REPRESENTAÇÃO                      CONSULTA + FILTROS
            │                                │
            ▼                                ▼
      SEGMENTAÇÃO                    BUSCA LEXICAL
            │                                +
            ▼                         BUSCA SEMÂNTICA
       UNIDADES                            │
            │                              ▼
            └──────────────────────► RECUPERAÇÃO HÍBRIDA
                                           │
                                           ▼
                                      DEDUPLICAÇÃO
                                           │
                                           ▼
                                     CONTEXTUALIZAÇÃO
                                           │
                                           ▼
                                       RERANKING
                                           │
                                           ▼
                                       SELEÇÃO
                                           │
                                           ▼
                                       EVIDÊNCIA
                                           │
                                           ▼
                                    PROVENIÊNCIA
                                           │
                                           ▼
                                    FONTE / LEITURA
                                           │
                                           ▼
                                 VOLTA À INVESTIGAÇÃO

E atravessando tudo:

IDENTIDADE
ESTADO
PERSISTÊNCIA
PROCESSAMENTO
SEGURANÇA
OPERAÇÕES

Esse, sim, é o esqueleto da V3.

Todo o resto pode esperar.

O que eu considero definitivamente cortado da nossa pauta de agora

Para não voltarmos a abrir essas portas durante C:

OCR → pós-Alpha.
Versionamento completo → pós-Alpha.
Proveniência completa → pós-Alpha.
Multimodalidade avançada → pós-Alpha.
Múltiplos perfis de segmentação → pós-Alpha.
Múltiplos rerankers → pós-Alpha.
Avaliação formal do retrieval → pós-Alpha.
Infraestrutura distribuída → pós-Alpha.
Observabilidade avançada → pós-Alpha.
Disaster recovery → pós-Alpha.
Reading Mode definitivo → pós-Alpha.

E principalmente: não vamos mais investigar tecnologia para essas coisas agora.

O resultado da extração

A meu ver, chegamos ao que a etapa que tu chamou de “resgate do indispensável de 5, 6.1 e 6.2” precisava produzir.

Não é ainda a arquitetura.

É o inventário de capacidades e invariantes que C é obrigada a respeitar.

Agora C pode receber exatamente isto:

ETAPA 5
   +
6.1
   +
6.2
   ↓
┌─────────────────────────────┐
│ INVENTÁRIO ALPHA            │
│                             │
│ Representação               │
│ Segmentação                 │
│ Retrieval                   │
│ Evidência                   │
│ Proveniência                │
│ Corpus/Processamento        │
│ Persistência                │
│ Segurança/Operação          │
└──────────────┬──────────────┘
               +
          ALPHA A
               +
          ALPHA B
               ↓
        C — ARQUITETURA
               ↓
        D — IMPLEMENTAÇÃO

E eu pararia a investigação exatamente aqui. Não tem ganho em continuar cavando a 6.2 agora. O próximo trabalho difícil é pegar esse inventário e transformar os princípios em componentes, responsabilidades, tecnologias, contratos, fluxo de dados e ordem de implementação. Aí sim começa a C — e aí temos bastante trabalho pela frente.