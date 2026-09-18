# Registro Mestre da Análise — Snoopy-RAG

## Etapa 6.3.1 — Modelo da Experiência de Pesquisa

**Status:** Consolidado e aprovado para transição à Etapa 6.3.2
**Natureza:** Especificação da experiência de pesquisa — não constitui implementação, escolha tecnológica ou projeto técnico de frontend.

---

## 1. Objetivo da etapa

A Etapa 6.3.1 teve como objetivo investigar como o pesquisador deverá utilizar o Snoopy V3 como um todo, estabelecendo o modelo geral da experiência de pesquisa antes da definição de estado, componentes, tecnologias ou arquitetura concreta de frontend.

A questão central foi:

> **Como deve ser a experiência de pesquisa proporcionada pelo Snoopy V3?**

A investigação partiu das capacidades e requisitos já estabelecidos nas Etapas 5, 6.1 e 6.2 e também do comportamento efetivamente existente na V2, especialmente no fluxo de consulta, recuperação, síntese, evidência, contexto e leitura.

---

# 2. Conclusão central

O Snoopy V3 deverá proporcionar uma **experiência de exploração flexível e iterativa de um corpus documental**, e não apenas uma experiência de perguntas e respostas.

O pesquisador deverá poder transitar entre:

```text
consulta
   ↓
recuperação
   ↓
resultados
   ↓
evidências
   ↓
contexto
   ↓
leitura
   ↓
documento original
   ↓
nova consulta
```

sem que o sistema imponha um percurso metodológico obrigatório.

A interface deverá estruturar as relações informacionais e documentais necessárias para a investigação, mas a interpretação do material, a avaliação de sua relevância metodológica e as decisões de pesquisa permanecerão sob responsabilidade do pesquisador.

---

# 3. O Snoopy como ambiente de exploração documental

A investigação estabeleceu que definir o Snoopy simplesmente como uma ferramenta de busca ou como um sistema de perguntas e respostas seria insuficiente.

A busca é uma capacidade fundamental do sistema, mas constitui apenas uma etapa dentro de uma experiência mais ampla.

O pesquisador poderá entrar no sistema com uma pergunta pontual e, a partir dos resultados encontrados, descobrir uma informação que gere uma nova pergunta, um novo documento ou uma nova linha de exploração.

Portanto, a intenção inicial da interação não deverá determinar rigidamente a experiência subsequente.

Uma consulta aparentemente pontual poderá evoluir naturalmente para exploração documental.

O modelo adotado é:

```text
PERGUNTA
   ↓
RECUPERAÇÃO
   ↓
EXPLORAÇÃO DOS RESULTADOS
   ↓
EVIDÊNCIA
   ↓
CONTEXTO
   ↓
DOCUMENTO
   ↓
NOVA PERGUNTA
```

A interface deverá permitir essa transição sem exigir que o pesquisador escolha previamente entre um “modo consulta” e um “modo exploração”.

---

# 4. Exploração flexível e iterativa

A experiência deverá ser:

### 4.1 Contínua

O pesquisador deverá poder prosseguir de uma descoberta para outra sem precisar reiniciar artificialmente sua investigação.

### 4.2 Flexível

O sistema não deverá impor uma sequência metodológica fixa.

### 4.3 Reversível

O pesquisador deverá poder retornar a resultados, evidências, documentos e contextos anteriormente acessados.

### 4.4 Transparente

O pesquisador deverá conseguir compreender o que foi produzido pelo sistema, quais evidências foram recuperadas e de onde essas evidências vieram.

### 4.5 Orientada pela agência do pesquisador

O Snoopy deverá apoiar a investigação sem decidir pelo pesquisador o significado metodológico dos materiais recuperados.

Esses princípios definem o modelo geral da experiência, mas não determinam ainda como serão implementados tecnicamente.

---

# 5. Estrutura informacional ≠ percurso metodológico

Uma distinção fundamental estabelecida nesta etapa é que o Snoopy poderá organizar a informação sem transformar essa organização em uma metodologia de pesquisa obrigatória.

A interface poderá estruturar relações como:

```text
consulta
   ↓
resultado
   ↓
unidade de recuperação
   ↓
evidência
   ↓
contexto
   ↓
documento
```

sem determinar:

* quais evidências o pesquisador deverá considerar relevantes;
* quais categorias deverá utilizar;
* como deverá interpretar os resultados;
* qual unidade metodológica deverá adotar;
* qual conclusão deverá alcançar.

O sistema organiza e apresenta a infraestrutura documental necessária para a investigação.

A decisão metodológica permanece humana.

---

# 6. Relação entre operações automáticas e ações do pesquisador

A experiência deverá manter uma distinção compreensível entre aquilo que o Snoopy executa automaticamente e aquilo que o pesquisador decide.

Entre as operações do sistema estão, por exemplo:

* decomposição operacional da consulta;
* recuperação lexical e semântica;
* filtragem;
* combinação de resultados;
* deduplicação;
* contextualização;
* reranking;
* seleção computacional;
* geração da síntese;
* associação entre síntese e evidências.

Entre as ações do pesquisador estão:

* formular perguntas;
* examinar resultados;
* selecionar o que deseja explorar;
* abrir evidências;
* acessar contextos;
* ler documentos;
* comparar materiais;
* formular novas perguntas;
* interpretar os materiais;
* tomar decisões metodológicas.

A interface não deverá ocultar essa distinção a ponto de apresentar uma decisão computacional como se fosse uma decisão do pesquisador.

---

# 7. O papel da síntese

A investigação inicialmente considerou diferentes papéis possíveis para a síntese, incluindo sua utilização como resultado central, recurso opcional ou camada auxiliar.

A análise do comportamento efetivo da V2 foi fundamental para resolver essa questão.

Na V2, o fluxo é conceitualmente:

```text
PERGUNTA ORIGINAL
       ↓
DECOMPOSIÇÃO EM MICROQUERIES
       ↓
RECUPERAÇÃO
       ↓
SELEÇÃO DOS TRECHOS
       ↓
PERGUNTA ORIGINAL + TRECHOS
       ↓
LLM
       ↓
SÍNTESE
       ↓
[TRECHO X] DENTRO DA SÍNTESE
```

Os marcadores `[TRECHO X]` não funcionam simplesmente como uma lista bibliográfica localizada abaixo da resposta.

Eles fazem parte da própria síntese e estabelecem uma relação entre as afirmações produzidas e as evidências recuperadas.

A síntese, portanto, constitui simultaneamente:

1. uma resposta à consulta;
2. uma condensação das evidências recuperadas;
3. uma porta de entrada para a exploração dessas evidências.

---

# 8. Síntese não é análise global do corpus

A síntese continuará tendo como função primária responder à consulta do pesquisador a partir das evidências recuperadas.

Ela não deverá assumir como responsabilidade obrigatória:

* representar a estrutura global do corpus;
* identificar automaticamente convergências metodológicas;
* identificar divergências metodológicas;
* produzir agrupamentos temáticos;
* determinar interpretações metodológicas;
* substituir a exploração dos documentos.

A discussão sobre convergência e divergência foi considerada uma questão potencialmente interessante, mas não necessária para definir o modelo geral da experiência.

Transformá-la em requisito da síntese poderia deslocar o Snoopy em direção a uma função de análise da literatura que não constitui responsabilidade estabelecida do sistema.

Portanto:

> **A síntese deverá auxiliar a compreensão das evidências recuperadas, mas não deverá substituir a exploração ou a interpretação do corpus pelo pesquisador.**

---

# 9. Síntese como porta de entrada para evidências

A oposição entre “síntese como resposta” e “síntese como instrumento de exploração” foi considerada inadequada.

A síntese pode responder à pergunta e, simultaneamente, servir como ponto de entrada para a exploração.

O modelo conceitual estabelecido é:

```text
                 SÍNTESE
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
      RESPOSTA             EVIDÊNCIAS
                              │
                              ↓
                           CONTEXTO
                              │
                              ↓
                         MODO LEITOR
                              │
                              ↓
                      FONTE ORIGINAL
```

Assim, a síntese não constitui necessariamente o destino final da experiência.

Ela poderá funcionar como uma camada condensada de orientação sobre aquilo que foi recuperado.

---

# 10. Evidências como elemento central da experiência

A investigação identificou que as evidências constituem a ponte fundamental entre a síntese e o corpus documental.

A cadeia de experiência é:

```text
afirmação
   ↓
evidência recuperada
   ↓
contexto documental
   ↓
documento
   ↓
fonte original
```

Isso está alinhado à arquitetura metodológica estabelecida anteriormente:

```text
documento
   ↓
elemento / fragmento
   ↓
unidade de recuperação
   ↓
evidência
   ↓
síntese
```

Consequentemente, a interface V3 deverá tornar as evidências individualmente acessíveis e navegáveis.

Foi identificada como consequência concreta a possibilidade de substituir ou complementar as atuais referências documentais da V2 por elementos que apresentem cada evidência individualmente, contendo, por exemplo:

* identificador do trecho;
* descrição breve;
* referência bibliográfica;
* acesso ao contexto;
* acesso ao documento original.

A definição detalhada dessa interface pertence à Etapa 6.3.3 — Busca, Evidência e Leitura.

---

# 11. Continuidade da experiência

Foi investigado o que deverá permanecer acessível quando o pesquisador encerra uma consulta e inicia outra.

Foram consideradas quatro alternativas:

### A — Substituição

A nova consulta substitui completamente a anterior, que permanece acessível somente pelo histórico.

### B — Continuidade no mesmo espaço

As consultas formam uma sequência contínua semelhante a uma conversa.

### C — Consultas independentes com resultados persistentes

Cada consulta é independente, mas resultados e evidências anteriores permanecem recuperáveis.

### D — Combinação

A interface pode substituir a consulta atualmente exibida, mas mantém histórico das investigações e permite retornar independentemente aos resultados, evidências, documentos e contextos anteriores.

A alternativa D foi adotada como princípio conceitual.

---

# 12. Continuidade da exploração ≠ continuidade de conversa

A continuidade necessária ao Snoopy não deverá ser entendida como continuidade obrigatória de uma conversa com a LLM.

No ChatGPT, por exemplo, a conversa constitui o principal objeto de continuidade:

```text
conversa
 ├── pergunta
 ├── resposta
 ├── pergunta
 ├── resposta
 └── resposta
```

No Snoopy, o objeto principal é a investigação documental:

```text
investigação
 ├── consulta
 │    ├── resultados
 │    ├── evidências
 │    └── documentos/contextos
 │
 ├── consulta
 │    ├── resultados
 │    ├── evidências
 │    └── documentos/contextos
 │
 └── consulta
      ├── resultados
      ├── evidências
      └── documentos/contextos
```

Portanto:

> **A continuidade a ser preservada é continuidade de exploração, não continuidade de texto.**

---

# 13. Consulta como objeto recuperável da experiência

Cada consulta deverá possuir identidade e histórico recuperáveis dentro da experiência de pesquisa.

Uma consulta deverá estar associada, conceitualmente, aos objetos produzidos ou acessados em seu percurso:

```text
consulta
   ├── configuração/restrições
   ├── resultados
   ├── evidências
   └── documentos/contextos acessados
```

A síntese poderá constituir parte desse resultado, mas não deverá ser confundida com a totalidade da investigação.

O pesquisador deverá poder:

* iniciar uma nova consulta;
* iniciar uma consulta relacionada;
* iniciar uma consulta independente;
* retornar a uma consulta anterior;
* retornar a resultados anteriores;
* retornar a evidências anteriores;
* retornar a documentos ou contextos anteriormente acessados.

A forma técnica pela qual esse histórico será representado pertence às etapas posteriores.

---

# 14. Campo de pesquisa como elemento de continuidade

A experiência deverá permitir que o pesquisador formule uma nova pergunta sem necessariamente abandonar imediatamente o contexto que está explorando.

Isso é compatível com a ideia de manter o campo de pesquisa acessível enquanto resultados, evidências ou documentos estão sendo examinados.

Essa interação poderá apresentar uma experiência de continuidade semelhante, em alguns aspectos, à interação conversacional, mas isso não implica que o Snoopy deverá ser concebido como chatbot.

A semelhança está na continuidade da interação.

A diferença está no objeto da interação:

> **no Snoopy, a conversa é uma forma possível de interação; o corpus documental é o objeto da exploração.**

A implementação concreta dessa continuidade será definida posteriormente.

---

# 15. História e reversibilidade

O histórico não deverá ser entendido apenas como uma lista textual de perguntas.

Seu objetivo é preservar o caminho documental já percorrido pelo pesquisador.

Conceitualmente, esse caminho envolve:

```text
consulta
   ↓
resultados
   ↓
evidências
   ↓
contextos
   ↓
documentos
```

O histórico deverá permitir a recuperação desse percurso sem exigir que o pesquisador refaça a investigação para encontrar novamente um material já localizado.

A definição de quais dados serão efetivamente persistidos, bem como sua representação técnica, pertence à Etapa 6.3.2 e à arquitetura integrada da Etapa 6.4.

---

# 16. Limites da experiência

A experiência de pesquisa deverá permanecer flexível, mas não deverá transformar o Snoopy em um sistema geral de gerenciamento de pesquisa.

Não foram estabelecidos como requisitos desta etapa:

* gerenciamento completo de projetos de pesquisa;
* sistema de notas;
* sistema de categorias metodológicas;
* codificação de documentos;
* mapas conceituais;
* análise automática da literatura;
* classificação metodológica automática;
* gerenciamento completo de revisão sistemática;
* substituição das bases bibliográficas;
* substituição do pesquisador.

Essas possibilidades poderão ser consideradas futuramente caso necessidades concretas surjam, mas não fazem parte do modelo de experiência definido para a V3.

---

# 17. Relação com o caso de uso do GEPAFOR

O contexto de uso do GEPAFOR reforça a necessidade desse modelo de experiência.

O Snoopy deverá ser capaz de apoiar uma investigação em que pesquisadores trabalham com um corpus heterogêneo formado por documentos provenientes de diferentes bases acadêmicas.

Nesse contexto, a experiência não consiste apenas em perguntar:

> “Qual é a resposta?”

Ela envolve também:

```text
o que foi encontrado?
        ↓
em quais documentos?
        ↓
quais trechos são relevantes?
        ↓
sobre o que esses trechos tratam?
        ↓
qual é o contexto?
        ↓
qual é a fonte?
        ↓
que nova pergunta surgiu?
```

Isso reforça o entendimento do Snoopy como infraestrutura de exploração documental.

O caso de uso do GEPAFOR não altera os requisitos arquiteturais estabelecidos anteriormente, mas aumenta a importância da clareza, navegabilidade, reversibilidade e rastreabilidade da experiência.

---

# 18. Decisões consolidadas

A Etapa 6.3.1 consolidou as seguintes decisões:

### DEXP-01 — Natureza da experiência

O Snoopy V3 será concebido como ambiente de exploração de um corpus documental, e não apenas como ferramenta de perguntas e respostas.

### DEXP-02 — Exploração iterativa

A experiência deverá permitir exploração flexível e iterativa.

### DEXP-03 — Ausência de percurso metodológico obrigatório

A interface não deverá impor um percurso metodológico ao pesquisador.

### DEXP-04 — Agência do pesquisador

Decisões metodológicas e interpretativas permanecerão sob responsabilidade do pesquisador.

### DEXP-05 — Distinção sistema/pesquisador

A interface deverá preservar uma distinção compreensível entre operações automáticas do sistema e ações deliberadas do pesquisador.

### DEXP-06 — Síntese

A síntese continuará sendo uma capacidade integrante do Snoopy V3, tendo como função primária responder à consulta a partir das evidências recuperadas.

### DEXP-07 — Síntese como porta de entrada

A síntese poderá funcionar como porta de entrada para as evidências, sem se tornar uma ferramenta de análise global do corpus.

### DEXP-08 — Evidências navegáveis

As evidências deverão ser individualmente acessíveis e relacionadas às suas origens documentais.

### DEXP-09 — Continuidade

A experiência deverá preservar a continuidade das investigações sem transformar essa continuidade em uma conversa obrigatória.

### DEXP-10 — Histórico

O pesquisador deverá poder retornar a consultas, resultados, evidências, contextos e documentos anteriormente acessados.

### DEXP-11 — Continuidade da exploração

A continuidade preservada será continuidade de exploração, e não continuidade obrigatória de texto ou conversa.

### DEXP-12 — Consulta como objeto recuperável

Cada consulta deverá possuir identidade e histórico recuperáveis dentro da experiência de pesquisa, associados aos resultados e evidências correspondentes.

---

# 19. Decisões descartadas

Foram explicitamente descartadas ou não adotadas como requisitos:

### 19.1 — Snoopy como chatbot

Não será adotado como modelo conceitual principal.

### 19.2 — Experiência baseada exclusivamente em pergunta → resposta

Considerada insuficiente para o objetivo de exploração documental.

### 19.3 — Escolha prévia entre “modo consulta” e “modo exploração”

Não será exigida, pois a intenção do pesquisador pode mudar durante a própria investigação.

### 19.4 — Síntese como substituta da exploração

Não será adotada.

### 19.5 — Síntese como análise global obrigatória do corpus

Não será adotada.

### 19.6 — Percurso metodológico rígido

Não será adotado.

### 19.7 — Continuidade conversacional como requisito central

Não será adotada.

### 19.8 — Expansão imediata para uma plataforma completa de gestão de pesquisa

Não faz parte do escopo atual.

---

# 20. Consequências para etapas posteriores

A Etapa 6.3.1 não define ainda a arquitetura técnica, mas produz consequências diretas para as etapas seguintes.

### Para 6.3.2 — Estado, Dados e Interação

Será necessário investigar como representar:

* consultas;
* resultados;
* evidências;
* documentos;
* contextos acessados;
* histórico;
* estado atual da investigação;
* transições entre estados.

Também deverá ser definida a distinção entre:

* estado do servidor;
* estado da aplicação;
* estado da URL;
* estado local da interface;
* preferências persistentes.

### Para 6.3.3 — Busca, Evidência e Leitura

Deverá ser detalhada a experiência concreta de:

```text
consulta
   ↓
resultado
   ↓
evidência
   ↓
contexto
   ↓
leitura
   ↓
documento original
```

Incluindo o papel das evidências individualizadas e sua relação com a síntese.

### Para 6.3.4 — Corpus, Documentos e Processamento

Deverá ser definido como o pesquisador compreenderá:

* corpus;
* documentos;
* versões;
* processamento;
* sincronização;
* jobs;
* progresso;
* falhas;
* estados documentais.

### Para 6.3.5 — Segurança, Acessibilidade e Resiliência

A experiência definida nesta etapa deverá ser preservada sob:

* falhas;
* carregamento;
* interrupções;
* conteúdo não confiável;
* diferentes formas de interação;
* limitações de acessibilidade.

### Para 6.3.6 — Arquitetura de Frontend e Tecnologias

A arquitetura de frontend deverá ser escolhida em função da experiência definida nesta etapa, e não o contrário.

---

# 21. Limites epistemológicos

As conclusões desta etapa não transformam a interface em metodologia de pesquisa.

O Snoopy continua não sendo responsável por:

* determinar relevância metodológica;
* interpretar definitivamente os documentos;
* definir categorias;
* determinar unidades de registro;
* produzir conclusões científicas;
* substituir o pesquisador.

A interface deve tornar a infraestrutura documental e computacional compreensível e navegável.

Ela não deve transformar suas próprias estruturas computacionais em decisões metodológicas.

---

# 22. Princípios consolidados da experiência

A experiência de pesquisa do Snoopy V3 deverá observar os seguintes princípios:

1. **Exploração antes de conclusão.**
2. **Flexibilidade antes de percurso rígido.**
3. **Evidência antes de confiança na síntese.**
4. **Fonte documental acessível.**
5. **Continuidade da exploração, não continuidade obrigatória de conversa.**
6. **Agência do pesquisador.**
7. **Transparência das operações do sistema.**
8. **Reversibilidade do percurso exploratório.**
9. **Separação entre recuperação, síntese e interpretação.**
10. **A interface deve apoiar a pesquisa sem prescrever sua metodologia.**

---

# 23. Formulação consolidada

A experiência de pesquisa do Snoopy V3 deverá ser baseada na exploração flexível, iterativa e rastreável de um corpus documental. O pesquisador deverá poder formular consultas, examinar resultados, acessar evidências, navegar para seus contextos e documentos de origem e formular novas perguntas sem ser submetido a um percurso metodológico obrigatório.

A síntese continuará sendo uma capacidade integrante da experiência, tendo como função primária responder às consultas a partir das evidências recuperadas. Ela poderá funcionar como uma porta de entrada para a exploração das evidências, mas não deverá substituir a análise ou a interpretação do corpus pelo pesquisador.

A continuidade da experiência deverá preservar o caminho documental percorrido pelo pesquisador, permitindo retornar a consultas, resultados, evidências, contextos e documentos anteriores. Essa continuidade não deverá transformar o Snoopy em um chatbot nem exigir continuidade conversacional.

A interface deverá, portanto, mediar a relação entre pesquisador e infraestrutura documental, tornando compreensíveis e navegáveis as relações entre consulta, recuperação, evidência, contexto, leitura e fonte, enquanto preserva a distinção entre operações computacionais do sistema e decisões metodológicas do pesquisador.

---

# 24. Princípio central da Etapa 6.3.1

> **O Snoopy V3 deverá proporcionar uma experiência de exploração documental flexível, iterativa, reversível e rastreável, na qual o pesquisador possa transitar entre consultas, resultados, evidências, contextos e documentos sem ser submetido a um percurso metodológico obrigatório. A síntese deverá auxiliar a compreensão das evidências recuperadas e poderá servir como porta de entrada para sua exploração, mas a interpretação e as decisões metodológicas permanecerão sob responsabilidade do pesquisador. A continuidade preservada pelo sistema deverá ser continuidade de exploração, e não continuidade obrigatória de conversa.**

---

## 25. Conclusão da etapa

A Etapa 6.3.1 é considerada **suficientemente investigada e encerrada**.

O problema foi identificado, as alternativas relevantes foram compreendidas, suas principais consequências foram analisadas, as decisões foram suficientemente fundamentadas e não permanece, dentro do escopo desta subetapa, uma incerteza relevante capaz de alterar significativamente o modelo definido.

As questões ainda abertas — representação do estado, desenho específico da navegação, estrutura das evidências, arquitetura de frontend e tecnologias — pertencem às etapas subsequentes e não constituem lacunas da presente investigação.

A próxima etapa deverá partir deste modelo já consolidado.

**Etapa seguinte: 6.3.2 — Estado, Dados e Interação da Aplicação.**
