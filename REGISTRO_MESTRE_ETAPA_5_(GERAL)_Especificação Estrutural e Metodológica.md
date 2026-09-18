# Registro Mestre da Análise — Snoopy-RAG

## Etapa 5 — Especificação Arquitetural e Metodológica

> **Status:** Registro mestre geral da Etapa 5.
> **Natureza:** especificação de análise — não constitui implementação.
> **Função:** registrar, de forma compacta e conceitualmente completa, aquilo que as subetapas 5.1–5.6 estabeleceram em conjunto.
> **Relação com os registros individuais:** este documento funciona como mapa da Etapa 5; os registros individuais das subetapas preservam o detalhamento do raciocínio, requisitos e questões específicas de cada domínio.

---

# 1. Objetivo da Etapa 5

A Etapa 5 surgiu como consequência direta da análise metodológica realizada anteriormente.

A Etapa 4 demonstrou que o principal problema arquitetural do Snoopy-RAG não era simplesmente a qualidade de um extrator, de um chunker ou de um mecanismo de busca isoladamente.

O problema mais profundo estava na própria relação entre:

```text
DOCUMENTO ORIGINAL
        ↓
REPRESENTAÇÃO
        ↓
SEGMENTAÇÃO
        ↓
RECUPERAÇÃO
        ↓
EVIDÊNCIA
        ↓
SÍNTESE
        ↓
PESQUISADOR
```

A arquitetura atual havia sido construída em torno de uma transformação relativamente simples:

```text
PDF → texto → chunks → embeddings
```

Essa estratégia foi suficiente para o objetivo inicial de tornar um conjunto de PDFs pesquisável, mas tornou-se limitada à medida que o Snoopy passou a ser pensado como infraestrutura de apoio à pesquisa.

O problema, portanto, deixou de ser:

> “Como extrair texto melhor?”

e passou a ser:

> **“Como preservar o documento, produzir representações derivadas adequadas às diferentes funções do sistema, recuperar evidências de maneira rastreável e manter todo esse processo consistente ao longo do tempo?”**

A Etapa 5 especifica a resposta arquitetural a esse problema, sem ainda escolher as tecnologias responsáveis por implementá-la.

---

# 2. Princípio metodológico da Etapa 5

A Etapa 5 adota a seguinte sequência:

```text
PROBLEMA METODOLÓGICO
        ↓
REQUISITO
        ↓
PRINCÍPIO ARQUITETURAL
        ↓
ALTERNATIVAS TÉCNICAS
        ↓
IMPLEMENTAÇÃO
        ↓
TESTE EMPÍRICO
```

A etapa atual termina no **princípio arquitetural**.

Não é função dela determinar qual biblioteca, modelo, formato de armazenamento ou serviço deverá ser utilizado.

Essa distinção é deliberada.

O objetivo é impedir que o Snoopy seja redesenhado simplesmente em torno das capacidades de uma determinada ferramenta.

Assim:

> **A Etapa 5 define o que a arquitetura precisa ser capaz de fazer; a Etapa 6 investigará como isso pode ser realizado tecnicamente.**

---

# 3. As seis subetapas

A Etapa 5 foi dividida em seis domínios relacionados:

```text
5.1 — Representação Canônica
       ↓
5.2 — Segmentação
       ↓
5.3 — Recuperação
       ↓
5.4 — Proveniência
       ↓
5.5 — Persistência, Versionamento e Consistência
       ↓
5.6 — Segurança, Isolamento e Operação
```

A ordem não representa necessariamente uma sequência temporal de implementação.

Ela representa uma dependência conceitual:

> primeiro é necessário preservar o documento; depois estruturá-lo e segmentá-lo; então recuperar unidades; manter sua origem; preservar seus estados ao longo do tempo; e garantir que todo esse sistema opere de maneira segura e consistente.

---

# 4. Etapa 5.1 — Representação Canônica

## 4.1. Problema

A representação textual atualmente derivada do PDF pode perder características relevantes do documento.

Entre essas características estão:

* estrutura espacial;
* ordem de leitura;
* tabelas;
* imagens;
* figuras;
* legendas;
* notas;
* referências;
* elementos gráficos;
* relações entre elementos;
* informação de página;
* localização dos blocos;
* conteúdo proveniente de OCR.

A transformação:

```text
PDF → texto/Markdown
```

pode, portanto, ser destrutiva.

O problema não é apenas perder informação para a busca.

A perda pode alterar a própria evidência posteriormente apresentada ao pesquisador.

---

## 4.2. Decisão arquitetural

A arquitetura deve possuir uma **representação canônica preservadora** do documento.

Essa representação:

* não é simplesmente Markdown;
* não é texto plano;
* não é um chunk;
* não é um embedding;
* não é uma resposta do LLM.

Ela deve funcionar como estado documental de referência a partir do qual representações derivadas podem ser produzidas.

O princípio estabelecido é:

> **Preservar antes de derivar.**

---

## 4.3. Arquitetura conceitual

```text
DOCUMENTO ORIGINAL
        ↓
REPRESENTAÇÃO CANÔNICA
        ↓
 ┌──────┼────────┐
 ↓      ↓        ↓
texto  estrutura elementos
        ↓
representações derivadas
```

A consequência fundamental é separar:

```text
O DOCUMENTO
```

de:

```text
AQUILO QUE O SISTEMA FEZ COM O DOCUMENTO
```

---

## 4.4. Requisitos principais

A representação canônica deve:

* minimizar transformações destrutivas;
* preservar identidade documental;
* preservar páginas;
* preservar ordem de leitura;
* preservar blocos;
* preservar estrutura;
* preservar elementos complexos;
* preservar informação espacial relevante;
* preservar relação com imagens, tabelas e figuras;
* manter origem dos elementos;
* permitir derivação de representações específicas;
* permitir reprocessamento sem necessidade de reextrair o documento original.

---

# 5. Etapa 5.2 — Segmentação

## 5.1. Problema

A segmentação atual é predominantemente heurística e baseada em características textuais e estruturais.

Ela é útil para a recuperação atual, mas não deve ser confundida automaticamente com segmentação semântica.

Além disso, o chunk técnico não possui o mesmo significado que uma unidade metodológica definida pelo pesquisador.

A distinção estabelecida é:

```text
chunk
= unidade técnica de processamento

unidade de registro
= unidade metodológica definida pelo pesquisador

unidade de contexto
= contexto necessário à compreensão da unidade de registro
```

Essa distinção é fundamental para impedir que o funcionamento interno do RAG seja confundido com a metodologia de Análise de Conteúdo.

---

## 5.2. Decisão arquitetural

A segmentação deve ocorrer em níveis.

O modelo conceitual estabelecido é:

```text
REPRESENTAÇÃO CANÔNICA
        ↓
SEGMENTAÇÃO ESTRUTURAL
        ↓
SEGMENTAÇÃO SEMÂNTICA
        ↓
UNIDADES DE RECUPERAÇÃO
        ↓
UNIDADE DE REGISTRO / CONTEXTO
       ↑
  pesquisador
```

A unidade recuperada pelo sistema continua sendo uma unidade computacional.

Ela pode fornecer material para análise, mas não se transforma automaticamente em unidade metodológica.

---

## 5.3. Princípios

A segmentação deve:

* preservar continuidade documental;
* evitar cortes arbitrários que destruam significado;
* preservar relação com títulos e estruturas superiores;
* tratar tabelas e elementos complexos adequadamente;
* manter localização original;
* permitir contexto adicional;
* diferenciar conteúdo do documento de metadados estruturais;
* possuir finalidade explícita;
* possuir identidade própria;
* manter vínculo com a representação de origem.

O princípio central é:

> **Segmentar para uma finalidade definida, sem transformar a segmentação computacional em decisão metodológica.**

---

# 6. Etapa 5.3 — Recuperação

## 6.1. Problema

A recuperação semântica atual transforma a pergunta em consultas derivadas, produz embeddings, consulta o espaço vetorial e seleciona resultados.

Esse processo é funcional, mas possui uma consequência epistemológica importante:

> **similaridade semântica não é sinônimo de relevância metodológica.**

Um trecho pode ser semanticamente próximo da pergunta e ainda assim ser inadequado para o objetivo da pesquisa.

---

## 6.2. Decisão arquitetural

O mecanismo de recuperação deve ser tratado como uma camada própria entre:

```text
PERGUNTA
   ↓
RECUPERAÇÃO
   ↓
EVIDÊNCIA
```

A recuperação deve permitir identificar e controlar:

* consulta original;
* consultas derivadas;
* corpus;
* versão do corpus;
* representação utilizada;
* segmentação;
* embedding;
* métrica;
* limiar;
* quantidade de resultados;
* ranking;
* deduplicação;
* expansão contextual;
* filtros;
* configuração da recuperação.

---

## 6.3. Reprodutibilidade

O objetivo principal de reprodutibilidade estabelecido para o Snoopy é:

> **mesmo corpus + mesma consulta + mesma configuração de recuperação → evidências recuperadas equivalentes.**

A preocupação principal não é reproduzir literalmente a frase produzida pelo LLM.

O alvo é a **reprodutibilidade da evidência recuperada**.

Entretanto, a versão atual não deve ser descrita como formalmente determinística ou cientificamente reprodutível sem validação empírica.

Fatores como decomposição por LLM, temperatura, ordenação, empates, mudanças no corpus, versões de chunking e embeddings podem alterar o resultado.

---

## 6.4. Princípio

> **O embedding localiza similaridade; o mecanismo de recuperação seleciona candidatos; a evidência permanece documental; e a relevância metodológica continua sendo uma decisão do pesquisador.**

---

# 7. Etapa 5.4 — Proveniência

## 7.1. Problema

O Snoopy já possui rastreabilidade funcional:

```text
resposta
 ↓
[TRECHO X]
 ↓
chunk
 ↓
documento
 ↓
Drive
 ↓
PDF original
```

Essa cadeia é importante, mas não constitui ainda um sistema formal de proveniência versionada.

A análise estabeleceu que a rastreabilidade precisa ser aprofundada.

---

## 7.2. Decisão arquitetural

A cadeia conceitual deve ser:

```text
PERGUNTA
   ↓
CONSULTA DERIVADA
   ↓
RESULTADO
   ↓
UNIDADE DE RECUPERAÇÃO
   ↓
BLOCO
   ↓
PÁGINA
   ↓
DOCUMENTO
   ↓
VERSÃO DOCUMENTAL
   ↓
ARQUIVO ORIGINAL
```

Cada representação derivada deve manter relação identificável com sua origem.

---

## 7.3. Duas formas de proveniência

Uma distinção central foi estabelecida:

### Proveniência documental

Responde:

> **“De onde veio este trecho?”**

### Proveniência metodológica

Responde:

> **“Por que este documento ou trecho foi considerado relevante para a pesquisa?”**

A primeira pode ser operacionalizada pelo sistema.

A segunda pertence, em última instância, ao processo metodológico do pesquisador.

A existência de uma cadeia documental perfeita não transforma automaticamente uma recuperação em justificativa metodológica.

---

## 7.4. Princípio

> **Toda evidência apresentada pelo sistema deve poder ser relacionada à representação que a produziu e, por meio dessa cadeia, ao documento e à fonte original.**

---

# 8. Etapa 5.5 — Persistência, Versionamento e Consistência

## 8.1. Problema

Persistir dados não significa possuir versionamento.

Versionar não significa automaticamente manter consistência.

E manter consistência não significa apenas evitar registros duplicados.

A questão central é:

> **Como garantir que aquilo que foi preservado, segmentado, recuperado e rastreado continue coerente quando o sistema muda ao longo do tempo?**

---

## 8.2. Decisão arquitetural

O documento deve possuir um estado temporalmente identificável.

Conceitualmente:

```text
DOCUMENTO
   │
   ├── V1
   │    ├── representação
   │    ├── segmentação
   │    └── embeddings
   │
   └── V2
        ├── representação
        ├── segmentação
        └── embeddings
```

Os derivados precisam indicar de qual estado documental foram produzidos.

---

## 8.3. Consistência

A cadeia de dependência pode ser representada como:

```text
documento
   ↓
representação
   ↓
estrutura
   ↓
segmentação
   ↓
unidade
   ↓
embedding
   ↓
recuperação
```

Quando um estado anterior muda, é necessário saber quais descendentes podem deixar de ser válidos.

O princípio estabelecido é:

> **Nenhuma representação derivada deve ser considerada vigente se sua origem ou seus parâmetros de derivação forem incompatíveis com o estado documental considerado vigente.**

---

## 8.4. Estados e reprocessamento

A arquitetura deve distinguir conceitualmente:

```text
ativo
obsoleto
removido
em processamento
falho
incompleto
```

e permitir:

* detecção de alterações;
* reprocessamento;
* invalidação;
* atualização incremental quando possível;
* recuperação após falhas;
* identificação de versões;
* reconstrução de estados anteriores quando necessário.

Invalidação não significa necessariamente exclusão física.

---

## 8.5. Corpus como estado

O corpus também participa da identidade da recuperação.

Assim:

```text
mesma pergunta
+
mesma configuração
+
corpus diferente
```

não representa o mesmo experimento de recuperação.

Consequentemente, a versão/estado do corpus precisa fazer parte do perfil de recuperação quando a reprodutibilidade for relevante.

---

# 9. Etapa 5.6 — Segurança, Isolamento e Operação

## 9.1. Problema

Segurança e operação não são questões externas à metodologia.

Um acesso indevido pode modificar o corpus de uma pesquisa.

Uma mistura entre produção e desenvolvimento pode alterar o estado documental.

Uma falha de concorrência pode produzir chunks ou embeddings incompatíveis.

Uma falha silenciosa pode modificar as condições da recuperação sem que o usuário perceba.

Portanto:

> **Segurança, isolamento e operação são condições para preservar a integridade documental e metodológica do sistema.**

---

## 9.2. Decisão arquitetural

O sistema deve possuir controle explícito de:

* identidade;
* autenticação;
* autorização;
* propriedade;
* administração;
* visibilidade;
* isolamento por acervo;
* isolamento entre ambientes;
* concorrência;
* processamento;
* publicação;
* credenciais;
* logs;
* auditoria;
* recuperação de falhas;
* limites de recursos.

---

## 9.3. Isolamento

O acervo constitui uma fronteira simultaneamente:

```text
SEGURANÇA
     +
ESCOPO DOCUMENTAL
     +
ESCOPO METODOLÓGICO
```

Portanto, resultados de diferentes acervos não podem ser misturados silenciosamente.

Da mesma forma, desenvolvimento e produção devem possuir fronteiras explícitas.

---

## 9.4. Jobs

O processamento assíncrono exige:

* reserva segura de jobs;
* prevenção de processamento concorrente indevido;
* idempotência;
* tratamento de falhas;
* identificação do ambiente;
* distinção entre estado parcial e estado publicado.

O estado:

```text
processing
```

não deve ser confundido com:

```text
documento disponível
```

---

## 9.5. Observabilidade

O sistema deve permitir reconstruir:

```text
o que aconteceu
quando
com qual documento
em qual versão
em qual ambiente
por qual processo
com qual resultado
```

sem transformar logs em cópias inseguras do acervo.

---

## 9.6. Princípio

> **O sistema deve impedir que acessos indevidos, falhas operacionais ou estados inconsistentes alterem silenciosamente as condições documentais da recuperação.**

---

# 10. Relação entre as seis subetapas

As seis subetapas não são módulos independentes.

Elas formam uma cadeia de dependências:

```text
                    DOCUMENTO ORIGINAL
                           │
                           ▼
                 ┌────────────────────┐
                 │ 5.1 REPRESENTAÇÃO  │
                 │     CANÔNICA        │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ 5.2 SEGMENTAÇÃO    │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ 5.3 RECUPERAÇÃO    │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ 5.4 PROVENIÊNCIA   │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ 5.5 PERSISTÊNCIA   │
                 │ VERSIONAMENTO      │
                 │ CONSISTÊNCIA       │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ 5.6 SEGURANÇA      │
                 │ ISOLAMENTO         │
                 │ OPERAÇÃO           │
                 └────────────────────┘
```

Mas essa representação ainda é incompleta, porque 5.4–5.6 também atravessam as etapas anteriores.

A arquitetura correta é mais próxima de:

```text
                    DOCUMENTO
                        │
                        ▼
                 REPRESENTAÇÃO
                        │
                        ▼
                  SEGMENTAÇÃO
                        │
                        ▼
                   RECUPERAÇÃO
                        │
                        ▼
                    EVIDÊNCIA
                        │
                        ▼
                  PESQUISADOR
                        │
                        ▼
                  INTERPRETAÇÃO
```

envolvida por:

```text
        ┌────────────────────────────────┐
        │       PROVENIÊNCIA             │
        │       VERSIONAMENTO             │
        │       CONSISTÊNCIA              │
        │       SEGURANÇA                │
        │       ISOLAMENTO               │
        │       OBSERVABILIDADE          │
        └────────────────────────────────┘
```

---

# 11. Arquitetura conceitual resultante

A partir das seis subetapas, a arquitetura-alvo pode ser expressa assim:

```text
                         DOCUMENTO ORIGINAL
                                │
                                ▼
                    REPRESENTAÇÃO CANÔNICA
                                │
               ┌────────────────┼────────────────┐
               ▼                ▼                ▼
           ESTRUTURA        TEXTO           ELEMENTOS
                            DERIVADO         COMPLEXOS
               └────────────────┼────────────────┘
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
                         REPRESENTAÇÃO
                         PARA BUSCA
                                │
                                ▼
                           RECUPERAÇÃO
                                │
                                ▼
                            EVIDÊNCIAS
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

Cada camada mantém relação com sua origem:

```text
representação
      ↓
estrutura
      ↓
segmentação
      ↓
unidade
      ↓
embedding
      ↓
resultado
      ↓
evidência
      ↓
documento
      ↓
fonte
```

E todas operam sob:

```text
IDENTIDADE
AUTORIZAÇÃO
ISOLAMENTO
VERSIONAMENTO
CONSISTÊNCIA
AUDITORIA
RECUPERABILIDADE
```

---

# 12. Distinções conceituais consolidadas

A Etapa 5 preserva algumas distinções que não devem ser apagadas durante a implementação.

## 12.1. Documento ≠ representação derivada

O documento original é a fonte.

Texto, Markdown, chunks e embeddings são derivados.

---

## 12.2. Representação ≠ evidência

Uma representação computacional serve para processar o documento.

A evidência continua sendo uma informação documental localizada e rastreável.

---

## 12.3. Chunk ≠ unidade de registro

Chunk é uma decisão computacional.

Unidade de registro é uma decisão metodológica.

---

## 12.4. Similaridade ≠ relevância

Similaridade é propriedade do mecanismo de recuperação.

Relevância metodológica depende do objetivo da pesquisa.

---

## 12.5. Recuperação ≠ análise

O Snoopy pode localizar e organizar material.

A análise continua sendo responsabilidade do pesquisador.

---

## 12.6. Grounding ≠ completude

Vincular uma resposta a evidências reduz o problema da invenção, mas não resolve o problema de selecionar todas as evidências relevantes.

> **Grounding reduz o problema da invenção; não elimina o problema da seleção.**

---

## 12.7. Rastreabilidade ≠ proveniência metodológica

Saber de onde veio um trecho não explica automaticamente por que ele é metodologicamente relevante.

---

## 12.8. Reprodutibilidade técnica ≠ reprodutibilidade científica

Repetir um processo computacional não garante, por si só, reprodução de uma conclusão científica.

---

# 13. Requisitos consolidados

As subetapas produziram conjuntos específicos de requisitos. No nível geral, eles podem ser agrupados em oito famílias.

### 13.1. Integridade documental

O sistema deve preservar as características documentais relevantes antes de produzir representações derivadas.

### 13.2. Integridade estrutural

A segmentação deve manter estrutura, continuidade, contexto e identidade das unidades.

### 13.3. Integridade da recuperação

A recuperação deve possuir configuração identificável, ser avaliável e respeitar os limites do corpus.

### 13.4. Integridade da proveniência

Toda evidência deve manter vínculo rastreável com sua origem.

### 13.5. Integridade temporal

Documentos, representações, segmentações e embeddings devem possuir estados distinguíveis.

### 13.6. Integridade de consistência

Derivados incompatíveis não devem ser tratados como estado vigente.

### 13.7. Integridade de segurança

Usuários, acervos e ambientes devem permanecer isolados conforme suas permissões.

### 13.8. Integridade operacional

Falhas, concorrência, retries e limitações de recursos não devem produzir estados silenciosamente inválidos.

---

# 14. Princípios gerais da Etapa 5

A partir das seis subetapas, podemos consolidar os seguintes princípios.

### Princípio 1 — Preservar antes de derivar

O sistema não deve transformar o documento de maneira destrutiva antes de preservar suas características relevantes.

### Princípio 2 — Uma representação não precisa servir para tudo

Representação canônica, texto pesquisável, segmentação, embeddings, leitura e síntese podem ser representações diferentes.

### Princípio 3 — Toda derivação deve possuir origem

Uma representação derivada deve poder ser relacionada ao estado que a produziu.

### Princípio 4 — Segmentação possui finalidade

Não existe uma única segmentação universalmente correta para todas as funções.

### Princípio 5 — Chunk não é metodologia

A unidade computacional não deve ser confundida com unidade metodológica.

### Princípio 6 — Similaridade não decide relevância

A recuperação encontra candidatos; o pesquisador interpreta sua relevância.

### Princípio 7 — A evidência é mais importante que a síntese

A frase gerada pelo LLM é uma representação derivada.

A evidência documental permanece central.

### Princípio 8 — O corpus faz parte da recuperação

Alterar o corpus pode alterar o resultado mesmo mantendo a pergunta.

### Princípio 9 — Estado importa

Documento, representação, segmentação e embeddings precisam pertencer a estados compatíveis.

### Princípio 10 — Invalidação deve ser explícita

Quando uma dependência muda, os derivados potencialmente incompatíveis precisam ser identificados.

### Princípio 11 — Segurança preserva metodologia

Isolar usuários e acervos também preserva o corpus da pesquisa.

### Princípio 12 — Falhas não podem produzir estados silenciosamente válidos

Um sistema que continua funcionando após uma falha pode ainda estar metodologicamente incorreto.

### Princípio 13 — Reprodutibilidade deve ser tratada como propriedade testável

Não basta declarar que uma recuperação é reproduzível.

É necessário testar sua estabilidade.

### Princípio 14 — O pesquisador permanece responsável pela análise

O Snoopy auxilia localização, recuperação, organização e rastreabilidade, mas não substitui interpretação metodológica.

---

# 15. O que a Etapa 5 deliberadamente não decidiu

É importante registrar também aquilo que **não** foi decidido.

A Etapa 5 não escolheu definitivamente:

* biblioteca de processamento de PDF;
* formato específico da representação canônica;
* mecanismo específico de OCR;
* modelo de embedding;
* algoritmo específico de chunking;
* mecanismo específico de reranking;
* banco ou índice adicional;
* estratégia definitiva de armazenamento de elementos multimodais;
* arquitetura definitiva de workers;
* mecanismo específico de lock;
* solução definitiva para isolamento de ambientes;
* ferramenta específica de observabilidade;
* política jurídica definitiva de retenção;
* infraestrutura definitiva de produção.

Essas decisões pertencem à etapa seguinte.

O que foi definido são as **propriedades que essas soluções deverão satisfazer**.

---

# 16. Questões que permanecem abertas

A Etapa 5 não encerra o projeto arquitetural.

Ela deixa perguntas para a Etapa 6.

Entre as principais:

### Representação

* Qual formato oferece melhor equilíbrio entre fidelidade e praticidade?
* Como representar tabelas, imagens, blocos e relações espaciais?
* Como tratar documentos escaneados?
* Quando utilizar OCR?
* Como lidar com documentos multimodais?
* Qual informação precisa ser persistida permanentemente?

### Segmentação

* Como derivar segmentações estruturais?
* Como definir unidades semânticas?
* Como preservar contexto?
* Como avaliar empiricamente diferentes estratégias?
* Como tratar elementos não textuais?

### Recuperação

* Qual estratégia de representação funciona melhor?
* É necessário combinar recuperação semântica e lexical?
* É necessário reranking?
* Como medir qualidade?
* Como garantir estabilidade?
* Como lidar com consultas complexas?

### Proveniência

* Qual granularidade de localização deve ser preservada?
* Página é suficiente?
* É necessário bloco?
* É necessário bounding box?
* Como representar cadeias de derivação?

### Persistência

* Como representar versões?
* Como detectar alterações no Drive?
* Como invalidar derivados?
* Como realizar reprocessamento incremental?
* Como manter consistência entre estados?

### Operação

* Como garantir claim seguro de jobs?
* Como estruturar retries?
* Como separar ambientes?
* Como monitorar workers?
* Como limitar recursos?
* Como realizar recuperação após falhas?

Essas perguntas não são lacunas da análise.

São precisamente o **trabalho da Etapa 6**.

---

# 17. Critério de sucesso arquitetural

A arquitetura futura não deverá ser considerada bem-sucedida simplesmente porque:

```text
o sistema responde perguntas.
```

Ela deverá ser avaliada em múltiplas dimensões:

```text
                    QUALIDADE
                       │
       ┌───────────────┼────────────────┐
       ▼               ▼                ▼
INTEGRIDADE        RECUPERAÇÃO      PROVENIÊNCIA
DOCUMENTAL         QUALITATIVA
       │               │                │
       └───────────────┼────────────────┘
                       ▼
                  CONSISTÊNCIA
                       │
                       ▼
                   SEGURANÇA
                       │
                       ▼
                OPERACIONALIDADE
                       │
                       ▼
                UTILIDADE PARA
                 O PESQUISADOR
```

A arquitetura precisa funcionar **como sistema**, não apenas como conjunto de componentes tecnicamente funcionais.

---

# 18. A grande mudança arquitetural

A arquitetura inicial do Snoopy pode ser resumida como:

```text
PDF
 ↓
texto
 ↓
chunk
 ↓
embedding
 ↓
resposta
```

A arquitetura especificada pela Etapa 5 é:

```text
                    DOCUMENTO
                        │
                        ▼
              REPRESENTAÇÃO CANÔNICA
                        │
                        ▼
                  ESTRUTURAÇÃO
                        │
                        ▼
                   SEGMENTAÇÃO
                        │
                        ▼
              UNIDADES DE RECUPERAÇÃO
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

com uma infraestrutura transversal responsável por:

```text
PROVENIÊNCIA
VERSIONAMENTO
CONSISTÊNCIA
SEGURANÇA
ISOLAMENTO
OBSERVABILIDADE
RECUPERABILIDADE
```

Essa é a principal consequência arquitetural da Etapa 5.

---

# 19. Formulação consolidada da Etapa 5

A especificação geral pode ser condensada em uma única formulação:

> **O Snoopy-RAG deverá ser estruturado como uma infraestrutura de exploração e recuperação documental que preserve uma representação canônica dos documentos antes da produção de representações derivadas, permita segmentações orientadas por finalidade, realize recuperação sobre unidades identificáveis e rastreáveis, mantenha a cadeia de proveniência até a fonte original, preserve estados documentais e computacionais versionáveis e consistentes e opere sob mecanismos explícitos de segurança, isolamento, observabilidade e recuperação de falhas. A arquitetura deverá separar representação documental, processamento computacional, recuperação semântica, síntese por modelos de linguagem e interpretação metodológica, mantendo o pesquisador como responsável pelas decisões analíticas e metodológicas.**

---

# 20. Modelo mental final da Etapa 5

A melhor forma de compreender o resultado desta etapa talvez seja abandonar temporariamente a expressão “RAG sobre PDFs”.

O objeto que estamos especificando é:

```text
                    ┌─────────────────────┐
                    │     DOCUMENTO       │
                    │      ORIGINAL       │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ REPRESENTAÇÃO       │
                    │     CANÔNICA        │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   ESTRUTURAÇÃO E    │
                    │    SEGMENTAÇÃO      │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │     RECUPERAÇÃO     │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │      EVIDÊNCIA      │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │      SÍNTESE        │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    PESQUISADOR      │
                    └──────────┬──────────┘
                               │
                               ▼
                       INTERPRETAÇÃO
```

E envolvendo todo o processo:

```text
┌──────────────────────────────────────────────────────┐
│                    PROVENIÊNCIA                      │
│                    VERSIONAMENTO                     │
│                    CONSISTÊNCIA                      │
│                    SEGURANÇA                         │
│                    ISOLAMENTO                        │
│                    OBSERVABILIDADE                   │
│                    RECUPERABILIDADE                  │
└──────────────────────────────────────────────────────┘
```

---

# 21. Conclusão

A Etapa 5 estabeleceu que o principal problema arquitetural do Snoopy-RAG não é simplesmente melhorar a extração, o chunking ou a busca isoladamente.

O problema é **preservar a relação entre documento, representação, segmentação, recuperação, evidência, estado e pesquisador**.

A arquitetura atual nasceu de um problema mais simples — tornar um conjunto de PDFs pesquisável — e foi evoluindo até uma ferramenta de recuperação e exploração documental. A análise agora mostra que essa evolução exige uma mudança de fundamento: o texto derivado não pode continuar funcionando como substituto implícito do documento.

A arquitetura deve passar de:

```text
PDF → Markdown → chunks → embeddings
```

para:

```text
PDF
 ↓
REPRESENTAÇÃO CANÔNICA
 ↓
REPRESENTAÇÕES DERIVADAS
 ↓
SEGMENTAÇÃO
 ↓
RECUPERAÇÃO
 ↓
EVIDÊNCIA
 ↓
SÍNTESE
 ↓
PESQUISADOR
```

preservando em todo o processo:

```text
ORIGEM
IDENTIDADE
CONTEXTO
VERSÃO
CONSISTÊNCIA
ESCOPO
SEGURANÇA
```

A formulação mais curta do que a Etapa 5 estabeleceu é:

> **Preservar antes de derivar; segmentar antes de recuperar; recuperar antes de sintetizar; rastrear tudo; versionar o que muda; e nunca confundir operação computacional com decisão metodológica.**

Com isso, a Etapa 5 deixa de ser apenas uma lista de “coisas que estão faltando” no Snoopy atual. Ela passa a constituir uma **especificação arquitetural coerente para o sistema que o Snoopy deverá se tornar**.

