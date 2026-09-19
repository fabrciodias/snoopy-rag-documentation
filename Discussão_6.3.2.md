Então vamos abrir a investigação pela taxonomia do estado, sem tocar ainda em framework, store ou biblioteca.

A primeira coisa que precisamos resolver é o que exatamente estamos chamando de “estado” no Snoopy V3. Se essa base ficar errada, qualquer decisão posterior de frontend vai apenas esconder o problema.

6.3.2.1 — Taxonomia do estado da aplicação
1. O problema

No V2, o estado está distribuído entre algumas estruturas explícitas — especialmente o appState — e uma série de estados implícitos mantidos por:

variáveis;
DOM;
callbacks;
resultados de requisições;
flags;
estado de autenticação;
URL;
carregamentos assíncronos;
elementos atualmente abertos na interface.

Isso funciona enquanto a aplicação possui relativamente poucas entidades e relações.

No V3, entretanto, o sistema passa a lidar simultaneamente com:

corpus
documento
versão
representação
unidade de recuperação
consulta
resultado
evidência
contexto
investigação
job
permissão
configuração

E essas coisas não têm a mesma natureza temporal.

Uma versão documental pode existir por meses.

Um job pode durar alguns minutos.

Uma consulta pode ser persistida como parte do histórico.

Um modal pode existir por alguns segundos.

Uma conexão SSE pode existir enquanto uma busca está sendo executada.

Portanto, chamar tudo simplesmente de “estado da aplicação” já é insuficiente.

2. Primeira distinção: entidade não é estado

Esse é um ponto que vem diretamente da 6.2.7.

Temos, conceitualmente:

ENTIDADE
   ↓
ESTADO
   ↓
ATIVIDADE / EXECUÇÃO

Por exemplo:

Documento
 ├── versão 1
 ├── versão 2
 └── versão 3

O documento é uma entidade lógica.

A versão é um estado historicamente distinguível dessa entidade.

Já:

processamento da versão 3

é uma atividade.

Isso significa que a interface não deveria necessariamente tratar:

documento
documento atualizado
documento sendo processado
documento processado

como simplesmente quatro valores de uma variável document.

São conceitos diferentes que podem se relacionar.

3. Segunda distinção: estado persistente e estado transitório

Aqui aparece uma divisão particularmente importante para o frontend.

Estado persistente

É aquilo que continua existindo independentemente da página estar aberta.

Exemplos:

corpus
documento
versão
unidade de recuperação
consulta registrada
resultado de investigação
evidência
histórico
job
permissão
configuração efetiva

Se o usuário fechar o navegador e voltar depois, determinados elementos desse estado ainda podem existir.

Estado transitório

Existe para permitir que uma operação ou interação aconteça, mas não necessariamente constitui um objeto persistente do Snoopy.

Exemplos:

"estou carregando"
"o usuário abriu este modal"
"esta conexão SSE está ativa"
"o usuário está digitando"
"o cursor está neste campo"
"o painel lateral está aberto"

Isso não significa que todo estado transitório seja irrelevante. Significa apenas que sua existência não deve ser confundida com a existência de uma entidade persistente do sistema.

4. Terceira distinção: estado do servidor ≠ estado do cliente

Essa provavelmente será uma das distinções centrais da 6.3.2.

Imagine:

Supabase
   │
   ├── documentos
   ├── versões
   ├── jobs
   ├── consultas
   ├── evidências
   └── permissões

Isso representa o estado governado pelo backend.

O navegador possui uma representação local desse estado:

Browser
   │
   ├── documentos carregados
   ├── job observado
   ├── consulta atual
   └── resultados exibidos

Essas duas coisas não são o mesmo estado.

O cliente possui uma representação do estado do servidor.

Isso é especialmente importante no Snoopy porque o estado pode mudar sem que o usuário faça nada diretamente.

Por exemplo:

Browser A
      │
      │ observa
      ▼
     Job
      ▲
      │
      │ altera
      │
Worker

O worker pode terminar um processamento enquanto o pesquisador está olhando para outra coisa.

A interface precisa então ser capaz de atualizar sua representação do estado.

É aqui que entra reatividade — mas ainda não precisamos decidir como implementá-la.

5. Quarta distinção: estado da aplicação ≠ estado da interface

Considere:

job.status = "processing"

Isso é estado do sistema/aplicação.

Já:

mostrar spinner

é uma consequência de apresentação.

Da mesma maneira:

documento.version = 3

não é equivalente a:

painel de versões está aberto

Ou:

evidence.selected = X

não necessariamente significa que a evidência X tenha sido alterada no servidor.

Temos, portanto:

ESTADO DO DOMÍNIO
        ↓
ESTADO DA APLICAÇÃO
        ↓
ESTADO DA INTERFACE

A interface representa o estado do sistema; ela não deveria ser a autoridade que define arbitrariamente qual é o estado do sistema.

Isso é particularmente importante para o V3.

6. Quinta distinção: estado ≠ evento

Essa diferença pode parecer pequena, mas é estrutural.

Considere:

"job está processing"

Isso é um estado.

Já:

job passou de pending para processing

é uma mudança/evento.

Podemos ter:

ESTADO
pending

EVENTO
job_started

NOVO ESTADO
processing

Isso é importante porque o Snoopy terá bastante comportamento assíncrono:

sincronização
→ processamento
→ validação
→ publicação
→ ativo

e:

consulta
→ recuperação
→ reranking
→ seleção
→ síntese

A interface precisa reagir às mudanças, mas não devemos necessariamente transformar todos os eventos em estado persistente.

7. Sexta distinção: ação/comando ≠ estado

Da mesma maneira:

"Pesquisar"

é uma ação.

Depois dessa ação podem surgir estados:

idle
   ↓
searching
   ↓
results_available

Ou:

"Sincronizar"

é uma ação.

O resultado pode ser:

sync_requested
processing
completed
failed

Essa distinção aponta para uma ideia importante para a arquitetura futura:

a interface não deveria manipular diretamente estruturas internas para produzir efeitos; ela deveria expressar ações ou comandos cujo resultado modifica o estado governado pelo sistema.

Ainda não estou dizendo que precisamos de uma arquitetura formal de CQRS, event sourcing ou qualquer coisa do gênero. Seria antecipar demais.

A questão é simplesmente separar conceitualmente:

O QUE EXISTE
     ↓
ESTADO

O QUE ACONTECEU
     ↓
EVENTO

O QUE O USUÁRIO SOLICITOU
     ↓
AÇÃO / COMANDO
8. Começa a aparecer uma taxonomia mais sólida

Com essas distinções, podemos representar preliminarmente:

                         SNOOPY
                           │
             ┌─────────────┴─────────────┐
             │                           │
       ESTADO PERSISTENTE          ESTADO TRANSITÓRIO
             │                           │
       ┌─────┴─────┐               ┌─────┴─────┐
       │           │               │           │
     domínio    investigação    aplicação    interface
       │           │               │           │
  documentos    consultas       loading      modal
  versões       resultados      conexão      seleção
  corpus        evidências      operação     foco
  jobs          histórico       erro         painel
  permissões
             │
             ▼
       representado pelo
          cliente
             │
             ▼
       ESTADO LOCAL DO CLIENTE

E ao redor disso:

AÇÃO
  ↓
mudança de estado

EVENTO
  ↓
notificação de mudança

URL
  ↕
estado navegável

PREFERÊNCIA
  ↕
estado persistente do usuário

Isso ainda é uma hipótese de modelo, não uma arquitetura final.

9. E existe uma questão particularmente importante: a URL

A URL merece ser tratada como categoria própria porque ela possui uma propriedade peculiar:

ela é estado da aplicação que pode ser externo ao processo interno da aplicação.

Por exemplo, conceitualmente:

/snoopy/search/abc123

pode identificar uma investigação.

Ou:

/snoopy/document/doc123/version/3

pode identificar um documento/contexto.

Nesse caso, o estado está simultaneamente:

no servidor
+
na aplicação
+
na URL

mas com funções diferentes.

A URL não deve necessariamente armazenar todos os dados da investigação. Ela pode funcionar como identificador navegável de um estado que existe em outro lugar.

Isso será importante para a continuidade definida na 6.3.1.

10. Uma consequência importante da 6.3.1

Na etapa anterior decidimos que:

cada consulta deve possuir identidade e histórico recuperáveis na experiência de pesquisa.

Isso significa que a consulta não pode ser tratada simplesmente como:

let currentQuery = "..."

e depois desaparecer quando outra busca começa.

Ela precisa ser alguma coisa conceitualmente mais próxima de:

Consulta
 ├── identidade
 ├── pergunta original
 ├── resultados
 ├── evidências
 ├── contextos acessados
 └── relações com a investigação

Mas atenção: isso não significa que tudo isso precise ficar carregado simultaneamente no navegador.

Essa é justamente uma questão que a 6.3.2 precisará resolver:

o que deve existir como estado persistente do sistema, o que deve ser mantido localmente pelo cliente e o que pode ser recuperado sob demanda?

Isso nos leva ao próximo problema.

11. O verdadeiro problema do V3

A questão, portanto, começa a mudar de:

“Como substituir o appState?”

para:

“Qual é a fronteira entre o estado governado pelo Snoopy, sua representação no cliente e o estado puramente necessário para a interação da interface?”

E isso gera três perguntas que considero inevitáveis para a próxima rodada:

A. O que precisa ser persistido?

Por exemplo:

consulta?
resultado?
evidência?
contexto acessado?
estado de seleção?
B. O que deve ser recuperado do servidor sob demanda?

Porque um corpus grande não significa que o navegador deva carregar o corpus inteiro.

C. O que pode existir exclusivamente no cliente?

Por exemplo:

modal aberto
painel aberto
texto sendo digitado
foco
estado de animação

A partir dessas três perguntas poderemos determinar uma fronteira de responsabilidade entre backend e frontend.

E essa fronteira é provavelmente mais importante para a arquitetura da V3 do que decidir, neste momento, qual framework será utilizado.

Estado atual da investigação

Eu consideraria esta primeira rodada ainda aberta, mas já suficientemente avançada para estabelecer uma hipótese forte:

O estado do Snoopy V3 deve ser tratado como um conjunto de estados de naturezas diferentes — domínio, investigação, operação e interface — relacionados entre si, mas não colapsados em uma única estrutura global. O cliente deve representar o estado governado pelo sistema sem se tornar sua autoridade, enquanto estados puramente interacionais podem permanecer locais. Ações, eventos e estados também devem ser conceitualmente distintos.

O próximo passo natural é investigar persistência vs. cache vs. estado local: o que exatamente precisa sobreviver à troca de página, à troca de consulta e até ao fechamento do navegador, e o que não precisa. Isso vai nos permitir transformar essa taxonomia em requisitos concretos para o modelo de estado. 

Sim — e ficou claro agora: a V2 é essencialmente uma aplicação de página única, com a maior parte da experiência acontecendo dentro dessa página por troca/manipulação de estado e DOM. E, para a V3, podemos considerar como hipótese de arquitetura que a experiência naturalmente evoluirá para múltiplas superfícies/páginas navegáveis, sem transformar isso numa discussão conceitual interminável agora. A questão da URL volta depois, quando estivermos desenhando essa arquitetura.

Vamos então para o ponto que realmente interessa.

6.3.2.2 — Persistência, cache e estado local

A pergunta é:

O que precisa sobreviver à troca de página, à troca de consulta e ao fechamento do navegador — e o que não precisa?

Eu sugiro começar não pelas tecnologias (localStorage, IndexedDB, cache HTTP etc.), mas pela necessidade de sobrevivência de cada informação.

Há, grosso modo, quatro níveis:

                 TEMPO DE VIDA
                       │
                       ▼
┌─────────────────────────────────────────────┐
│ 1. INTERAÇÃO                                │
│    dura enquanto aquela interação existe    │
├─────────────────────────────────────────────┤
│ 2. SESSÃO / APLICAÇÃO                       │
│    sobrevive a mudanças de tela            │
├─────────────────────────────────────────────┤
│ 3. INVESTIGAÇÃO / SISTEMA                   │
│    deve continuar existindo posteriormente │
├─────────────────────────────────────────────┤
│ 4. HISTÓRICO / DOCUMENTAÇÃO                 │
│    precisa permanecer reconstruível        │
└─────────────────────────────────────────────┘

Mas vamos colocar isso em coisas concretas do Snoopy.

1. Estado puramente local

Algumas coisas não precisam ser persistidas.

Exemplos:

modal aberto;
menu aberto;
painel lateral expandido;
elemento atualmente focado;
texto que o usuário ainda está digitando;
animação;
estado visual momentâneo;
indicador de carregamento puramente visual.

Se o usuário fecha a página e isso desaparece, não há perda do estado do Snoopy.

Portanto:

Estado efêmero de interface permanece local ao cliente.

Não há razão para transformar cada detalhe da interface em dado persistente.

2. Estado que deve sobreviver à navegação

Agora temos algo diferente.

Imagine:

Pesquisa A
   ↓
resultado
   ↓
evidência
   ↓
Reading Mode

O usuário abre a evidência em outra página.

Ao voltar, não deveria necessariamente retornar para uma tela “zerada”.

Da mesma forma, se a V3 tiver:

/pesquisa/123
/documento/456
/documento/456/versao/2

a navegação entre essas superfícies precisa preservar o contexto necessário para voltar.

Mas isso não significa necessariamente duplicar todos os dados em cada página.

A página pode simplesmente reconstruir seu estado a partir de:

URL
+
estado persistente no servidor
+
dados recuperados sob demanda

Isso é uma distinção importante.

Sobreviver à troca de página não significa necessariamente permanecer carregado na memória do navegador.

3. A consulta é diferente

Aqui entra diretamente a decisão da 6.3.1.

Nós decidimos que uma consulta deve ser um objeto recuperável da experiência.

Logo:

pergunta original
resultados
evidências
configuração relevante
histórico da consulta

não podem depender exclusivamente da memória do navegador.

Se o usuário fizer:

“Como a avaliação formativa se relaciona à regulação da aprendizagem?”

e depois fizer outra consulta, a primeira não deveria desaparecer simplesmente porque uma variável foi substituída.

Isso significa:

A investigação precisa possuir persistência própria.

E essa persistência pertence ao estado governado pelo Snoopy, não ao estado local da interface.

4. Resultado de busca

Aqui existe uma distinção interessante.

O resultado pode ser:

A. descartável

A API responde:

consulta → resultados

e, quando a página fecha, tudo desaparece.

B. persistente

O Snoopy registra:

Consulta X
 ├── configuração
 ├── resultados
 ├── evidências
 └── provenance

A segunda alternativa é coerente com o que decidimos na 6.3.1 se esses resultados fazem parte da investigação recuperável.

E aqui eu vejo uma consequência importante:

Não devemos persistir tudo simplesmente porque foi retornado pela API. Devemos persistir aquilo que constitui o registro reconstruível da investigação.

Isso evita transformar cada resposta HTTP em um gigantesco banco de dados de estado.

5. Cache é outra coisa

Esse ponto precisa ficar muito claro porque “persistência” e “cache” frequentemente são misturados.

Imagine:

Consulta X
    ↓
resultado Y

Se o Snoopy guarda Y porque quer que o pesquisador consiga voltar a essa investigação, isso é persistência de estado/investigação.

Se o Snoopy guarda Y apenas para evitar executar novamente uma operação cara, isso é cache.

A mesma informação pode até estar armazenada nos dois contextos, mas a finalidade é diferente.

Persistência

“Preciso conseguir reconstruir isso.”

Cache

“Posso reutilizar isso se continuar válido.”

Essa diferença é fundamental.

Um cache pode ser apagado.

Um registro persistente de investigação não deveria desaparecer simplesmente porque o navegador decidiu limpar seu cache.

6. Cache não é fonte de verdade

Isso conversa diretamente com 6.2.7 e 6.2.8.

Temos:

Google Drive
      ↓
fonte documental

Supabase/PostgreSQL
      ↓
estado lógico governante

Browser
      ↓
representação local

Cache
      ↓
otimização

O navegador pode possuir:

documentos carregados
resultados
metadados
evidências
contextos

mas isso não significa que ele seja a autoridade sobre esses objetos.

Se o cache estiver errado:

cache ≠ estado válido

A aplicação precisa poder descartá-lo e reconstruir a representação correta.

7. E o corpus?

Aqui a resposta é ainda mais clara.

Não faz sentido o navegador persistir todo o corpus documental.

Especialmente considerando o cenário real do GEPAFOR:

centenas de documentos;
PDFs potencialmente grandes;
documentos heterogêneos;
versões;
representações derivadas;
unidades de recuperação.

O navegador deve manter representações necessárias para a experiência atual, não transformar-se em uma segunda infraestrutura documental.

Portanto:

Corpus
   ↓
Servidor

Documentos
   ↓
Servidor / fonte original

Unidades
   ↓
Servidor

Embeddings
   ↓
Servidor

Proveniência
   ↓
Servidor

Enquanto:

Browser
   ↓
dados necessários para a experiência atual
8. Preferências são um caso diferente

Tema escuro, por exemplo:

theme = dark

não é estado do corpus.

Também não é investigação.

É uma preferência persistente do usuário.

Pode sobreviver ao fechamento do navegador, mas não precisa necessariamente existir no PostgreSQL.

Então temos um quarto tipo bastante claro:

Estado do sistema
Estado da investigação
Estado local da interface
Preferências do usuário

Essa distinção vai ser muito útil.

9. O que deve sobreviver ao fechamento do navegador?

Eu colocaria uma primeira classificação assim:

Informação	Sobrevive ao fechamento?	Motivo
Modal aberto	❌	puramente UI
Painel aberto	❌	puramente UI
Texto ainda digitado	❌*	interação incompleta
Tema	✅	preferência
Corpus selecionado	talvez	depende da navegação/contexto
Consulta realizada	✅	investigação
Resultados da consulta	✅**	se fazem parte do registro da investigação
Evidências recuperadas	✅	objeto da investigação
Proveniência	✅	reconstrução/auditabilidade
Documento	✅	estado do sistema
Versão documental	✅	estado histórico
Job	✅	estado operacional/histórico
Embedding	✅	estado derivado
Estado de processamento	✅	operação do sistema
Cache de resultados	❌	pode ser reconstruído

* Podemos eventualmente oferecer recuperação de rascunho, mas isso é outra decisão.

** A forma exata de persistência dos resultados ainda será consequência do desenho de 6.4.

10. O ponto mais importante: não devemos persistir a interface inteira

Essa talvez seja a conclusão mais importante desta rodada.

Seria fácil cair numa arquitetura assim:

"Salvar tudo que está acontecendo no navegador"

e acabar com algo parecido com:

appState
 ├── query
 ├── results
 ├── modal
 ├── loading
 ├── selectedEvidence
 ├── documents
 ├── theme
 ├── picker
 ├── tokens
 ├── ...

Só que isso reproduziria exatamente o problema que estamos tentando superar.

A ideia deve ser quase inversa:

Persistir objetos que possuem significado próprio para o Snoopy; manter localmente apenas o estado necessário para representar e operar a interface; utilizar cache somente como otimização.

Isso nos dá uma separação conceitual muito mais limpa:

                 SNOOPY V3
                    │
       ┌────────────┼────────────┐
       │            │            │
       ▼            ▼            ▼
   PERSISTÊNCIA   CLIENTE      CACHE
       │            │            │
       │            │            └─ otimização
       │            │
       │            ├─ estado da UI
       │            ├─ representação local
       │            └─ interação
       │
       ├─ documentos
       ├─ versões
       ├─ corpus
       ├─ jobs
       ├─ consultas
       ├─ resultados
       ├─ evidências
       └─ provenance

E há uma consequência particularmente boa:

a troca de página deixa de ser uma operação destrutiva.

A página atual pode morrer.

A interface pode ser reconstruída.

O estado importante continua existindo.

11. Onde eu acho que estamos chegando

A hipótese que eu colocaria na mesa é:

O Snoopy V3 deve distinguir explicitamente entre estado persistente do sistema/investigação, estado local e transitório da interface, preferências do usuário e cache. Objetos que possuem significado documental, investigativo, operacional ou histórico próprio devem ser governados pela persistência do Snoopy; o cliente deve manter apenas a representação necessária para a experiência e interação atuais; e o cache deve ser tratado exclusivamente como mecanismo reconstruível de otimização, nunca como fonte de verdade.

Isso também resolve uma possível confusão da nossa discussão anterior:

“ter várias páginas” não significa “perder o estado quando muda de página”.

Na verdade, é justamente o contrário. Uma arquitetura mais explícita de estado permite que:

Pesquisa
   ↓
Evidência
   ↓
Documento
   ↓
voltar
   ↓
Pesquisa

seja uma navegação entre representações do mesmo estado persistente, em vez de uma sequência de manipulações de uma única página.

Critério de encerramento desta rodada

Eu diria que ainda falta uma única decisão concreta antes de fechar 6.3.2.2:

Até onde devemos persistir o estado de uma investigação?

Ou seja: devemos guardar apenas consulta + resultados + evidências, ou também coisas como documentos/contextos que o pesquisador abriu, seleções que fez e percurso de navegação?

Essa é a última fronteira relevante aqui. Se resolvermos isso, a distinção persistência × cache × estado local fica suficientemente fechada para avançarmos.

Bora. Essa é justamente a última questão que falta fechar nessa parte.

6.3.2.2 — Continuação: até onde persiste uma investigação?

A pergunta agora é:

O que constitui a investigação que merece ser preservada, e o que é apenas o percurso momentâneo do pesquisador pela interface?

Temos três possibilidades principais.

A — Persistir apenas a consulta e seus resultados
Consulta
 ├── pergunta
 ├── configuração
 ├── resultados
 └── evidências

É simples e suficiente para recuperar “o que o Snoopy encontrou”.

Mas perde parte da experiência posterior. Se o pesquisador abriu três documentos, leu duas evidências e voltou a uma determinada passagem, isso não faria parte do registro.

B — Persistir todo o percurso de navegação
Consulta
 ├── resultados
 ├── evidências
 ├── documentos abertos
 ├── contextos abertos
 ├── páginas visitadas
 ├── seleções
 └── sequência de navegação

Isso permitiria reconstruir praticamente toda a sessão.

O problema é que começamos a transformar o Snoopy em um gravador de comportamento do usuário. Além de aumentar bastante a complexidade, isso não é necessário para cumprir o papel metodológico que definimos.

Não precisamos saber que o pesquisador abriu primeiro o documento A, depois voltou, clicou no documento B e ficou 14 segundos olhando para a página 7.

Isso é telemetria/comportamento, não necessariamente estado da investigação.

C — Persistir o resultado da investigação, não todo o percurso

Essa é a distinção que me parece mais importante.

O Snoopy deveria preservar aquilo que possui valor documental ou investigativo, mas não necessariamente cada movimento do pesquisador.

Por exemplo:

INVESTIGAÇÃO
│
├── Consulta
│   ├── pergunta original
│   ├── configuração efetiva
│   └── derivação da consulta
│
├── Resultados
│   └── unidades recuperadas
│
├── Evidências
│   └── referências aos resultados selecionados
│
├── Contextos/documentos associados
│   └── quando fizerem parte da investigação
│
└── Proveniência
    └── cadeia necessária para reconstrução

Enquanto isto:

"abriu o documento"
"fechou o painel"
"clicou neste resultado"
"voltou"
"abriu outro"
"rolou até a página 12"

pode permanecer transitório.

Mas há uma diferença importante entre "acessado" e "selecionado"

Imagine que o pesquisador faça uma busca e encontre:

TRECHO A
TRECHO B
TRECHO C
TRECHO D
TRECHO E

Ele abre o contexto de B.

Depois abre o documento de D.

Depois decide que C é realmente relevante para o trabalho.

Temos três tipos de relação diferentes:

B → foi visualizado
D → foi explorado
C → foi selecionado como evidência

Se persistirmos tudo como se tivesse o mesmo significado, perdemos justamente a distinção que estamos tentando preservar.

Então eu proporia:

A interface pode registrar localmente o percurso de navegação, enquanto a persistência do Snoopy registra relações investigativas que tenham significado próprio.

Por exemplo, no futuro poderia existir:

evidence_selected
context_referenced
document_associated

mas não precisamos registrar:

mouse_moved
panel_opened
scroll_position
E o "documento acessado"?

Aqui existe uma sutileza.

Na 6.3.1 nós colocamos entre os elementos da continuidade:

documentos/contextos acessados.

Eu manteria isso, mas com uma interpretação mais restrita.

Não significa:

“o Snoopy precisa guardar um log completo de cada documento que o usuário abriu”.

Significa:

uma investigação deve poder manter referências aos documentos e contextos que se tornaram relevantes para ela.

Isso pode acontecer, por exemplo, porque:

uma evidência veio daquele documento;
o pesquisador explicitamente adicionou o documento à investigação;
um contexto foi associado a uma evidência;
a própria investigação precisa retornar àquele ponto.

Assim, o documento entra no estado investigativo por sua relação com a investigação, e não simplesmente porque o usuário clicou nele.

E o histórico?

Aqui eu faria uma distinção entre histórico de consultas e histórico de navegação.

Histórico de consultas

Persistente.

Pesquisa 1
Pesquisa 2
Pesquisa 3
...

Porque isso faz parte da continuidade definida na 6.3.1.

Histórico de navegação

Não necessariamente persistente.

O navegador pode fornecer:

voltar
avançar

e a aplicação pode manter o contexto necessário para isso.

Não precisamos transformar cada navegação em registro histórico da investigação.

Isso nos dá uma fronteira bastante limpa
                 INVESTIGAÇÃO
                      │
        ┌─────────────┴─────────────┐
        │                           │
   SIGNIFICADO                 PERCURSO
   INVESTIGATIVO               MOMENTÂNEO
        │                           │
        ▼                           ▼
   PERSISTENTE                    LOCAL
        │                           │
   consulta                      modal
   resultados                    foco
   evidências                    painel
   provenance                    scroll
   relações relevantes           navegação transitória
   documentos associados

E isso é importante porque evita dois extremos:

Extremo 1 — memória curta demais

“Cada busca substitui a anterior.”

Não. Isso contradiz a 6.3.1.

Extremo 2 — memória de tudo

“Vamos registrar cada coisa que o pesquisador fez.”

Também não. Isso extrapola o papel do Snoopy.

Uma consequência metodológica interessante

Existe ainda uma razão para não persistirmos indiscriminadamente o percurso.

O fato de um pesquisador ter aberto uma passagem não significa que ele a considere relevante.

O fato de ter lido um documento não significa que o incorporou à investigação.

O fato de ter visualizado um resultado não significa que o considera evidência.

Essa distinção é coerente com tudo que definimos desde a Etapa 5:

o sistema não deve transformar automaticamente comportamento de interface em decisão metodológica.

Portanto:

visualizou ≠ selecionou
abriu ≠ considerou relevante
recuperou ≠ aceitou como evidência

Isso é muito importante para o desenho da UI.

Decisão provisória

Neste ponto, eu considero a alternativa C suficientemente fundamentada:

O Snoopy V3 deve persistir o estado que possui significado próprio para a investigação — consultas, resultados relevantes, evidências, configurações e provenance, além das relações investigativas necessárias com documentos e contextos — mas não deve transformar todo o percurso de navegação da interface em histórico persistente. Estados puramente interacionais permanecem locais e transitórios, enquanto o cache pode ser utilizado como representação reconstruível e otimização.

Isso também resolve a questão do fechamento do navegador:

Se o navegador fechar:

Não devemos perder:

investigações;
consultas realizadas;
resultados que constituem seu registro;
evidências;
provenance;
documentos/contextos associados à investigação;
estados persistentes do corpus;
jobs e estados operacionais.

Podemos perder:

modal aberto;
painel aberto;
seleção puramente visual;
posição de scroll;
estado momentâneo de carregamento;
conexão SSE;
outros detalhes efêmeros.

Podemos manter opcionalmente no cliente:

preferências;
alguns caches;
eventualmente rascunhos, se houver uma necessidade concreta.
E acho que aqui podemos fechar 6.3.2.2

Aplicando nosso próprio critério:

Problema identificado: sim — distinguir investigação persistente de percurso transitório.
Alternativas relevantes: persistir apenas resultados, persistir todo percurso, ou persistir o estado investigativo significativo.
Consequências: conhecidas.
Decisão: persistir significado investigativo; manter percurso efêmero local.
Incerteza capaz de alterar significativamente a decisão: não vejo uma relevante neste momento.

Então eu não abriria outra rodada sobre persistência/cache.

A próxima questão da 6.3.2 passa a ser muito mais concreta: como esses diferentes estados se relacionam e como a aplicação os atualiza — isto é, entramos na parte de estado do servidor, estado da aplicação, estado da URL, estado local e ações/comandos.

Vamos lá

Bora. 😎

Agora entramos na parte mais interessante da 6.3.2: como esses estados se relacionam sem voltarmos ao problema do appState monolítico.

6.3.2.3 — Relação entre estados e atualização da aplicação

Já temos uma decisão importante:

O Snoopy V3 não terá um único “estado da aplicação” homogêneo. Terá diferentes categorias de estado, com diferentes responsáveis e tempos de vida.

Agora precisamos responder:

Como essas categorias se relacionam e quem tem autoridade para alterá-las?

1. Primeiro: quem é a fonte da verdade?

A divisão que está começando a aparecer é:

                 SNOOPY
                   │
        ┌──────────┴──────────┐
        │                     │
   ESTADO DO SISTEMA     ESTADO DO CLIENTE
        │                     │
        │              ┌──────┴──────┐
        │              │             │
        │          aplicação       interface
        │
        ├── documentos
        ├── versões
        ├── corpus
        ├── jobs
        ├── consultas
        ├── evidências
        └── provenance

O ponto fundamental é:

o cliente não deve inventar o estado do sistema.

Se o backend diz:

job = completed

o frontend não deveria simplesmente decidir:

job = completed

porque terminou um timer local.

Da mesma forma, se uma versão documental foi publicada pelo backend, a UI deve receber/consultar essa informação, não fabricar uma versão localmente.

2. Mas o cliente também possui estado legítimo

Isso não significa que o frontend seja apenas um terminal burro.

Ele possui estados próprios.

Por exemplo:

usuário está na investigação 123
painel de evidências está aberto
evidência X está selecionada
modal de documento está aberto
tema = escuro

Esses estados são legítimos.

O problema aparece quando começamos a misturar:

job.status

com:

sidebarOpen

na mesma estrutura sem distinguir suas responsabilidades.

3. Portanto, precisamos de uma relação de autoridade

Eu colocaria assim:

                    BACKEND
                 fonte do estado
                    do sistema
                       │
                       │ API / realtime
                       ▼
                 CLIENTE
                       │
              representação local
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
       estado da aplicação   estado da UI

E a comunicação funciona nos dois sentidos, mas não simetricamente.

Backend → frontend

O backend informa:

dados;
estados;
resultados;
permissões;
eventos;
mudanças.
Frontend → backend

O frontend solicita ações:

pesquisar;
sincronizar;
abrir/associar algo;
alterar configuração;
iniciar operação;
cancelar operação, quando suportado.

O frontend não deveria simplesmente escrever diretamente em uma representação local e assumir que aquilo virou verdade.

4. Isso nos leva a uma distinção entre comando e atualização

Imagine:

Usuário clica em “Sincronizar”.

O fluxo conceitual não deveria ser:

clique
 ↓
appState.syncing = true
 ↓
"pronto"

Seria algo mais próximo de:

AÇÃO DO USUÁRIO
      │
      ▼
"solicitar sincronização"
      │
      ▼
BACKEND
      │
      ▼
JOB CRIADO
      │
      ▼
estado do sistema muda
      │
      ▼
CLIENTE recebe mudança
      │
      ▼
INTERFACE atualiza

O frontend pode imediatamente mostrar:

“Solicitando sincronização…”

Mas isso é estado local de interação.

O fato de existir um job é estado do sistema.

Essa diferença evita uma classe enorme de inconsistências.

5. O mesmo vale para busca

No V3, a busca será mais complexa que a V2.

Conceitualmente:

usuário
  │
  │ pergunta
  ▼
cliente
  │
  │ comando
  ▼
backend
  │
  ├── decomposição
  ├── filtros
  ├── recuperação lexical
  ├── recuperação semântica
  ├── fusão
  ├── contextualização
  ├── reranking
  └── seleção
  │
  ▼
resultado da investigação
  │
  ▼
cliente
  │
  ├── síntese
  ├── evidências
  └── navegação

A interface pode mostrar:

Buscando...
Recuperando evidências...
Organizando resultados...

Mas essas mensagens não devem ser confundidas com a verdade documental.

Elas são uma representação do estado operacional.

6. E onde entra a reatividade?

Aqui chegamos a um ponto que interessa bastante para a futura arquitetura.

Se o estado do sistema muda independentemente da interface, a interface precisa conseguir reagir.

Exemplo:

Usuário abre corpus
        │
        ▼
job = processing
        │
        │
        └──── worker termina
                    │
                    ▼
              job = completed
                    │
                    ▼
             frontend percebe
                    │
                    ▼
             UI se atualiza

Isso é reatividade no sentido conceitual:

uma mudança no estado relevante produz atualização das representações que dependem daquele estado.

Ainda não significa React.

Ainda não significa Vue.

Ainda não significa Signals.

Estamos falando da propriedade arquitetural.

A tecnologia será investigada depois.

7. Nem todo estado precisa ser reativo da mesma maneira

Isso é importante.

Imagine:

theme = dark

Não precisamos de um sistema sofisticado de sincronização distribuída.

Já:

job.status = processing

pode precisar de atualização em tempo real.

E:

document.version

pode exigir atualização quando o usuário retorna àquela entidade.

Então diferentes categorias de estado podem ter diferentes mecanismos de atualização:

ESTADO
 │
 ├── persistente → recuperar da API
 │
 ├── dinâmico → atualização/realtime/polling
 │
 ├── local → atualização imediata
 │
 └── preferência → armazenamento local

Isso é muito mais saudável do que tentar fazer tudo passar por um único mecanismo.

8. URL entra aqui — e agora podemos resolver a dúvida que tu levantou

A URL não precisa ser tratada como “mais uma fonte de verdade”.

Ela pode ser uma representação navegável de uma parte do estado.

Por exemplo:

/investigacao/123

significa:

“quero visualizar a investigação 123.”

A aplicação então faz:

URL
 ↓
identificador
 ↓
recuperação da investigação
 ↓
estado local da aplicação
 ↓
interface

Isso é muito diferente de:

URL = estado inteiro da aplicação

Não queremos isso.

A URL deve carregar identificadores e parâmetros necessários à navegação, não todo o conteúdo da investigação.

9. Isso combina perfeitamente com a decisão da 6.3.1

Nós dissemos:

cada consulta deve possuir identidade e histórico recuperáveis.

Então:

/investigacao/abc123

pode representar uma investigação persistente.

A página é apenas a superfície através da qual o pesquisador acessa essa investigação.

Isso nos permite:

Pesquisa
   ↓
/investigacao/123
   ↓
Evidência X
   ↓
/investigacao/123/evidencia/456
   ↓
Documento
   ↓
/documento/789/versao/2

Não precisamos decidir agora se essa é exatamente a estrutura das URLs. Isso é 6.4.

O que podemos decidir agora é o princípio:

A navegação deve ser capaz de identificar estados persistentes da experiência sem depender da manutenção de uma única página viva.

E isso confirma tua percepção de que a evolução da SPA V2 para uma aplicação com múltiplas superfícies navegáveis é natural.

10. Ações também precisam de uma fronteira

Outro problema do V2 é que a interface possui bastante coordenação imperativa.

No V3, queremos algo conceitualmente mais próximo de:

INTERAÇÃO
   ↓
AÇÃO
   ↓
SERVIÇO / API
   ↓
MUDANÇA DE ESTADO
   ↓
REPRESENTAÇÃO

Em vez de:

clique
 ↓
muda DOM
 ↓
muda variável
 ↓
chama API
 ↓
muda outra variável
 ↓
mostra modal
 ↓
recarrega página

Isso não significa que a implementação precise seguir literalmente uma arquitetura Redux/CQRS.

Significa apenas que queremos uma direção causal compreensível.

11. Isso também resolve um problema do window.location.reload()

No V2, recarregar a página é uma forma relativamente simples de dizer:

“jogue fora o estado atual e reconstrua tudo”.

É funcional, mas é uma solução grosseira.

No V3, se o estado estiver bem separado, uma operação como:

“usuário desconectou o Google Drive”

poderia produzir:

ação
 ↓
backend atualiza relação
 ↓
estado do sistema atualizado
 ↓
cliente recebe atualização
 ↓
componentes afetados reagem

Sem precisar:

window.location.reload()

A página deixa de ser o mecanismo de reset do estado.

12. Começamos a ter uma arquitetura conceitual

Neste momento, eu colocaria a hipótese assim:

                    ┌────────────────────┐
                    │       BACKEND      │
                    │                    │
                    │ estado governado   │
                    │ pelo Snoopy        │
                    └─────────┬──────────┘
                              │
                       API / eventos
                              │
                              ▼
                    ┌────────────────────┐
                    │      CLIENTE       │
                    │                    │
                    │ representação      │
                    │ local do sistema   │
                    └─────────┬──────────┘
                              │
               ┌──────────────┼──────────────┐
               ▼              ▼              ▼
          estado da       navegação       estado UI
          aplicação        / URL
               │
               ▼
          componentes

E ao redor:

cache ───────────► otimização

preferências ────► armazenamento do usuário

persistência ────► fonte de verdade

eventos ─────────► notificações de mudança

ações ───────────► solicitações de mudança

Isso é uma separação muito mais importante do que simplesmente dizer:

“vamos colocar um state manager”.

13. E acho que encontramos a decisão principal desta rodada

A decisão que me parece suficientemente fundamentada é:

O estado do Snoopy V3 deverá possuir uma direção clara de autoridade. O estado persistente do sistema será governado pelo backend e pelo modelo persistente do Snoopy; o frontend manterá uma representação local desse estado, combinada com estados próprios da aplicação e da interface. Ações do usuário deverão produzir solicitações de mudança, enquanto alterações do estado do sistema deverão atualizar as representações dependentes no cliente. A URL funcionará como mecanismo de identificação e navegação de estados persistentes, e não como fonte independente de verdade.

E alguns princípios derivados:

Backend é autoridade sobre estado persistente do sistema.
Frontend não é uma segunda fonte de verdade.
Estado local da UI não deve ser confundido com estado documental/investigativo.
Ações, eventos e estados são conceitos distintos.
Reatividade é requisito arquitetural; framework é decisão posterior.
A URL identifica/navega estados; não armazena a aplicação inteira.
Atualização de estado não deve depender de reload() como mecanismo geral.
Diferentes estados podem possuir diferentes mecanismos de atualização.
E aqui eu fecharia esta rodada.

Não vejo necessidade de entrar agora numa guerra de “Redux vs Zustand vs Signals vs Context vs MobX”. Isso seria 6.3.6 antes da hora.

A próxima questão realmente relevante da 6.3.2 é mais específica:

Como o frontend deve lidar com dados do servidor que chegam, mudam, expiram ou deixam de ser válidos?

Aí entramos em server state, cache de dados, invalidação, atualização, concorrência entre requisições e sincronização — que é o ponto onde a taxonomia que acabamos de construir começa a virar uma arquitetura de aplicação concreta.

Perfeito. E a observação da sincronização é importante, mas eu a colocaria no lugar certo da arquitetura, para não contaminarmos a investigação da 6.3.2.

O que tu trouxe gera uma consequência clara:

A atualização do estado do corpus não pode depender exclusivamente de uma ação explícita do usuário.

Isso já estava implícito na separação que fizemos entre estado do sistema e estado da interface. O corpus pode mudar enquanto ninguém está olhando para ele.

No teu cenário:

Google Drive
    │
    │ fonte externa
    ▼
Snoopy
    │
    ├── sincronização manual
    │
    └── sincronização automática periódica
             │
             ▼
        estado atualizado

E o reboot diário do MegaWare às 4h introduz outra preocupação operacional:

03:59 → servidor funcionando
04:00 → reboot
04:xx → servidor volta
      → serviços sobem
      → sincronização periódica precisa continuar funcionando

Mas não precisamos resolver agora se será cron, PM2, worker agendado, Supabase scheduler etc. Isso pertence principalmente à 6.2.8/6.4. O que podemos registrar agora é o requisito de comportamento:

A interface deve tratar o estado do corpus como potencialmente mutável independentemente da ação imediata do usuário.

Isso é importante para a próxima discussão sobre server state.

6.3.2.4 — Server State: atualização, validade e sincronização

Agora sim, vamos avançar.

A questão é:

Como o frontend deve lidar com dados do servidor que chegam, mudam, expiram ou deixam de ser válidos?

Isso parece uma questão técnica, mas tem uma consequência direta para a experiência.

Imagine que o usuário abra o corpus às 14h.

Naquele momento:

Corpus
├── 120 documentos
└── última sincronização: 13h

Enquanto ele pesquisa, uma sincronização automática detecta:

+ 3 documentos novos
+ 1 documento atualizado
- 1 documento removido

O frontend precisa decidir:

o que fazer com a representação que o usuário já estava vendo?

Não podemos simplesmente assumir:

“o usuário apertou atualizar, então agora atualizamos.”

Porque a atualização pode ter ocorrido independentemente dele.

1. Server state é inerentemente mutável

Isso é diferente de estado puramente local.

estado UI:
"modal aberto"

→ só a interface altera.

server state:
"documentos disponíveis no corpus"

→ pode mudar por múltiplos agentes.

No Snoopy teremos vários:

usuário
worker
Google Drive
sincronizador
outro usuário no futuro
processamento assíncrono
administrador

Então o frontend precisa lidar com a possibilidade de:

“o que eu estou vendo pode ter mudado no servidor desde que foi carregado.”

2. Isso gera três estados diferentes para um dado

Uma informação pode estar:

Atual
cliente = servidor
Desatualizada
cliente ≠ servidor
Desconhecida
cliente não sabe se mudou

Essa terceira categoria é particularmente importante.

Se a interface carregou um corpus às 14h e não existe mecanismo de atualização, ela não pode afirmar:

“este é o estado atual do corpus”.

Ela só sabe:

“este é o último estado que eu conheço”.

Isso parece uma sutileza, mas é exatamente o tipo de coisa que uma interface de pesquisa precisa comunicar corretamente.

3. Nem toda mudança precisa aparecer imediatamente

Também não queremos cair no extremo oposto:

“Qualquer alteração no servidor deve instantaneamente reconstruir toda a interface.”

Isso poderia ser péssimo.

Imagine o pesquisador lendo uma evidência enquanto chegam três documentos novos.

Não queremos destruir o contexto atual porque o corpus mudou.

Então podemos distinguir:

estado do servidor mudou
        │
        ├── atualização pode ser aplicada imediatamente
        │
        ├── atualização pode ser indicada ao usuário
        │
        └── atualização pode esperar uma nova consulta

A decisão depende da natureza do dado.

Por exemplo:

Job de processamento:

processing → completed

provavelmente merece atualização imediata.

Lista de documentos do corpus:

120 → 123 documentos

pode ser atualizada de maneira menos intrusiva.

Documento que o pesquisador está lendo:

se uma nova versão foi criada enquanto ele está lendo a anterior, não devemos simplesmente substituir a versão que ele está vendo.

Isso conversa diretamente com 5.5 e 6.2.7:

estado histórico não deve ser silenciosamente substituído por estado mais novo.

4. A versão documental resolve parte do problema

Esse é um excelente exemplo de como as decisões anteriores entram na UI.

Suponha:

Documento X
├── versão 1
└── versão 2 ← nova

O pesquisador está lendo:

versão 1

A sincronização detecta:

versão 2

A UI não deveria fazer:

versão 1 → versão 2

silenciosamente.

Ela pode dizer algo como:

“Uma versão mais recente deste documento está disponível.”

E permitir que o pesquisador decida quando migrar.

Isso preserva:

continuidade;
rastreabilidade;
reprodutibilidade;
contexto da investigação.

E evita um problema particularmente ruim:

o pesquisador estar olhando para uma evidência baseada na versão 1 enquanto a interface silenciosamente troca o documento para a versão 2.

5. Isso vale também para resultados de pesquisa

Imagine que a consulta foi executada sobre:

Corpus versão X

Depois chegam novos documentos.

O resultado antigo não necessariamente deve ser recalculado automaticamente.

Porque:

consulta Q
+
corpus C1
+
configuração R1
=
resultado X

Se o corpus virou C2, então:

consulta Q
+
corpus C2
+
configuração R1
=
potencialmente outro resultado

Portanto, resultado de investigação persistido não deve ser silenciosamente atualizado porque o corpus mudou.

Isso é uma consequência fortíssima da 6.2.7.

A investigação antiga continua sendo uma investigação sobre as condições em que foi executada.

Uma nova busca pode ser feita sobre o corpus atualizado.

6. Isso nos leva a uma regra muito boa

Dados persistentes do servidor podem mudar; registros históricos de investigação não devem mudar silenciosamente por causa dessas alterações.

Então:

CORPUS ATUAL
      ↓
pode mudar

INVESTIGAÇÃO ANTERIOR
      ↓
preserva seu estado histórico

NOVA INVESTIGAÇÃO
      ↓
pode utilizar o corpus atualizado

Isso é exatamente o tipo de distinção que o frontend precisa representar.

7. E aí entra a sincronização automática que tu mencionou

Ela não deve ser pensada simplesmente como:

“o sistema vai apertar o botão Sincronizar sozinho.”

Conceitualmente é:

fonte externa
     ↓
mudança detectada
     ↓
processamento
     ↓
novo estado documental
     ↓
publicação
     ↓
frontend toma conhecimento

O frontend não precisa saber necessariamente como a sincronização aconteceu.

Ele precisa saber:

o corpus mudou

e representar isso adequadamente.

Isso é uma separação muito saudável:

Sincronização
→ responsabilidade do sistema

Detecção da mudança
→ responsabilidade do backend/worker

Atualização da representação
→ responsabilidade do cliente

Decisão de explorar a mudança
→ responsabilidade do pesquisador
8. Começamos a chegar numa regra de atualização

Eu colocaria como hipótese:

O frontend não deve tratar todos os dados recebidos do servidor como dados imutáveis nem como dados que precisam ser imediatamente substituídos. Cada categoria de estado deve possuir uma política de atualização adequada à sua natureza, preservando estados históricos e evitando que mudanças externas destruam silenciosamente o contexto atual do pesquisador.

Isso vai ser muito importante depois para definir:

atualização automática;
polling;
realtime;
invalidação;
refetch;
cache;
stale data;
atualização otimista/pessimista;
sincronização de páginas.

Mas ainda não vamos escolher essas tecnologias.

E acho que essa é a próxima pergunta certa

Agora que sabemos que o server state pode mudar independentemente do usuário, precisamos investigar:

Quando o frontend deve buscar novamente o estado do servidor?

Temos alternativas como:

1. somente quando o usuário solicita
2. refetch ao entrar na página
3. polling periódico
4. atualização por eventos/realtime
5. combinação das anteriores
6. invalidação dirigida por eventos + refetch sob demanda

E aqui, inclusive, tua ideia da sincronização diária do corpus e o reboot das 4h do MegaWare podem voltar como cenário operacional real para avaliar as alternativas.

Mas eu manteria uma distinção:

sincronização do corpus ≠ sincronização da interface.

O backend pode sincronizar o Drive uma vez por dia; a interface pode ter mecanismos próprios para descobrir que o estado mudou. São duas camadas diferentes.

Essa é a próxima investigação.

Perfeito. Podemos seguir. E creio (e espero) que essa rodada vai ser um mais condensada que as anteriores por já termos uma ideia da decisão com base nas próprias rodadas anteriores 

Sim. E aqui eu concordo contigo: esta rodada deve ser bem mais curta justamente porque as anteriores já eliminaram boa parte das alternativas.

6.3.2.5 — Quando atualizar o estado do servidor no cliente?

A pergunta é:

Quando o frontend deve buscar novamente ou receber o estado atualizado do servidor?

Temos seis possibilidades conceituais:

somente por ação do usuário;
ao entrar na página;
polling periódico;
realtime/eventos;
combinação dos anteriores;
invalidação dirigida + recuperação sob demanda.
1. Só ação do usuário

Não serve como modelo geral.

Já estabelecemos que o corpus pode mudar independentemente do usuário:

Google Drive
   ↓
sincronização automática
   ↓
novo estado documental

Além disso, jobs podem terminar sozinhos e permissões/estados operacionais podem mudar.

Portanto:

Atualização exclusivamente manual é insuficiente.

2. Buscar ao entrar na página

Isso é necessário, mas também não basta.

Quando o usuário entra em:

/investigacao/123

faz sentido recuperar o estado atual da investigação.

Quando entra em:

/corpus

faz sentido recuperar o estado atual do corpus.

Mas enquanto ele permanece na página, o servidor pode mudar.

Então:

refetch na entrada é uma estratégia básica de recuperação, não uma política completa de atualização.

3. Polling periódico

Aqui já temos um candidato útil.

Por exemplo:

GET /jobs/123
        ↑
      a cada N segundos

É particularmente adequado para:

jobs;
processamento;
operações longas;
estados que não justificam infraestrutura realtime.

Mas não faz sentido ficar consultando tudo constantemente.

Imagine fazer:

GET /corpus
GET /documents
GET /investigations
GET /permissions
GET /jobs
...

a cada poucos segundos.

Seria desperdício.

4. Realtime/eventos

Também é útil, mas não precisamos transformar todo o Snoopy em um sistema realtime.

Já temos um exemplo claro:

um job está sendo processado.

O frontend pode receber:

processing
   ↓
75%
   ↓
100%
   ↓
completed

Isso melhora bastante a experiência.

Mas para o corpus documental, uma atualização em tempo real a cada alteração do Drive provavelmente seria desnecessária.

O corpus pode ser sincronizado periodicamente.

5. Então a combinação aparece naturalmente

E aqui acho que chegamos à decisão praticamente sem precisar de outra grande investigação:

              ESTADO DO SERVIDOR
                     │
          ┌──────────┼──────────┐
          │          │          │
       refetch    eventos    polling
          │          │          │
          └──────────┼──────────┘
                     ▼
             estado do cliente

Mas com políticas diferentes por tipo de estado.

Corpus/documentos
entrada na página
      +
atualização periódica/invalidação
      +
ação manual quando necessário

Não precisa de realtime contínuo.

Jobs
evento/realtime
      +
fallback por polling/refetch

Aqui realtime tem bastante valor.

Investigação persistida
entrada/retorno
      ↓
buscar estado atual

Mas sem substituir silenciosamente o registro histórico por causa de mudanças no corpus.

Dados puramente locais
sem sincronização
6. Invalidação é a peça que fecha isso

Em vez de pensar:

“A interface precisa atualizar tudo o tempo inteiro.”

Podemos pensar:

“Quando sabemos que determinado dado pode estar desatualizado, marcamos sua representação como inválida/desatualizada e recuperamos o estado necessário.”

Exemplo:

Corpus carregado
     ↓
sincronização detecta mudança
     ↓
corpus = stale
     ↓
frontend sabe que precisa atualizar
     ↓
refetch
     ↓
novo estado

Isso é muito mais escalável.

E combina perfeitamente com o que já estabelecemos sobre cache:

cache
  ↓
válido?
  ├── sim → reutilizar
  └── não → recuperar novamente
7. Uma regra importante: atualização não deve destruir contexto

Essa decisão da rodada anterior continua valendo.

Se o usuário estiver lendo:

Documento X — versão 2

e surgir:

Documento X — versão 3

não fazemos:

versão 2 → versão 3

automaticamente.

Podemos informar:

“Uma versão mais recente está disponível.”

Da mesma forma, uma investigação antiga não deve ser recalculada automaticamente porque o corpus mudou.

Portanto:

atualizar o estado conhecido pelo cliente não significa substituir automaticamente o objeto que o pesquisador está investigando.

Essa distinção é essencial.

Decisão da rodada

Eu fecharia assim:

O Snoopy V3 deverá utilizar uma estratégia composta de atualização do estado do servidor, combinando recuperação inicial ao entrar em uma superfície, invalidação/refetch quando necessário, atualização por eventos para estados dinâmicos — especialmente operações assíncronas — e polling periódico como mecanismo complementar ou de fallback. A estratégia deverá variar conforme a natureza do estado, evitando tanto a dependência exclusiva de ações do usuário quanto a atualização indiscriminada e contínua de todos os dados. Alterações do servidor não deverão destruir silenciosamente estados históricos ou contextos de investigação atualmente utilizados pelo pesquisador.

O que descartamos
❌ atualização exclusivamente manual;
❌ polling indiscriminado de tudo;
❌ realtime para absolutamente tudo;
❌ refetch como única estratégia;
❌ atualização automática que substitua silenciosamente contexto histórico.
E isso encerra esta rodada

Aplicando nosso critério:

problema identificado: sim;
alternativas relevantes: sim;
consequências: sim;
decisão: fundamentada;
incerteza capaz de alterar significativamente a decisão: não vejo nenhuma.

Então eu não abriria outra investigação sobre sincronização agora.

O próximo passo lógico da 6.3.2 é justamente o último grande pedaço: como ações do usuário e mudanças do servidor devem produzir transições de estado, ou seja, o modelo de ações/comandos, eventos e atualização da aplicação. Isso vai nos levar diretamente à questão de reatividade e, depois, podemos fechar 6.3.2 e deixar a escolha tecnológica para 6.3.6.

Então vamos para o último grande bloco de investigações da 6.3.2Sim. Aqui chegamos ao último grande bloco da 6.3.2. E concordo contigo: depois do que já fechamos, ele deve ser bem mais concentrado.

O que já está decidido nos blocos anteriores é suficiente para não reabrirmos a taxonomia de estados, persistência, cache ou sincronização. Agora falta fechar como a aplicação passa de um estado para outro.

6.3.2.6 — Ações, eventos e transições de estado

A pergunta central é:

Como uma ação do pesquisador ou uma mudança externa no sistema deve produzir uma transição de estado que seja compreensível, previsível e refletida corretamente na interface?

Aqui temos três conceitos que precisamos separar:

Ação/command → algo que alguém ou alguma parte do sistema solicita que aconteça.
pesquisar;
abrir documento;
iniciar sincronização;
selecionar evidência;
alterar consulta;
cancelar processamento.
Evento → algo que aconteceu.
busca concluída;
documento atualizado;
job iniciado;
job falhou;
nova versão publicada;
corpus ficou desatualizado.
Estado → a situação resultante.
consulta em execução;
consulta concluída;
documento em processamento;
documento ativo;
job falho;
investigação recuperável.

A relação conceitual seria:

AÇÃO
  ↓
operação no sistema
  ↓
EVENTO / mudança observável
  ↓
novo ESTADO
  ↓
representações dependentes são atualizadas
  ↓
UI reflete o novo estado

Isso resolve uma fragilidade importante do V2. Hoje, muita coisa acontece porque uma função JavaScript chama outra função, altera uma variável, mexe no DOM e eventualmente dispara outra operação. A interface acaba sendo parte do mecanismo que determina o estado.

Na V3, a lógica deveria ser invertida:

estado do sistema
       ↓
estado representado no cliente
       ↓
interface

e não:

DOM
 ↓
variável global
 ↓
callback
 ↓
outra função
 ↓
tentativa de descobrir qual é o estado atual
Um ponto importante: eventos não precisam virar um "event sourcing"

Não precisamos transformar o frontend num sistema baseado integralmente em event sourcing. Isso seria uma decisão de infraestrutura que já sabemos que não é necessária para o Snoopy.

A distinção aqui é conceitual e arquitetural:

uma coisa é registrar eventos como histórico persistente; outra é usar eventos como mecanismo para comunicar que um estado mudou.

Por exemplo:

Job 182
estado anterior: processing
        ↓
evento: job.completed
        ↓
estado atual: completed
        ↓
UI atualiza o progresso

Não significa que a UI precise reconstruir todo o sistema a partir dos eventos.

E isso leva diretamente à reatividade

Já estabelecemos que a V3 precisa ser reativa como propriedade arquitetural.

Mas "reatividade" aqui não significa necessariamente React, Vue, Svelte ou qualquer framework específico.

Significa:

quando um estado relevante muda, as partes da interface que dependem desse estado devem poder atualizar-se sem que o código precise reconstruir manualmente a página inteira.

Isso é particularmente importante para:

jobs de processamento;
sincronização do corpus;
resultados de busca;
investigação atual;
novas versões de documentos;
autenticação/permissões;
erros e estados de conexão.

E temos uma consequência muito boa das decisões anteriores:

nem todo estado precisa da mesma reatividade.

Por exemplo:

Estado	Atualização provável
Job em processamento	Realtime + fallback
Corpus	Invalidação/refetch
Documento aberto	Atualização controlada
Investigação histórica	Estável
Estado de modal	Local
Tema	Preferência local
Consulta atual	Aplicação/UI
Permissão	Estado do servidor

Isso evita tanto o frontend "burro" quanto um sistema em que absolutamente tudo fica conectado a tudo.

A decisão que eu colocaria como fechamento

Acho que podemos fechar este último bloco com uma decisão relativamente direta:

O Snoopy V3 deverá modelar explicitamente a distinção entre ações, eventos e estados. Ações representam solicitações de mudança; eventos representam mudanças ou ocorrências observáveis; estados representam a situação atual de uma entidade ou processo. A interface deverá reagir às mudanças relevantes de estado por meio de uma arquitetura reativa, sem depender de manipulação imperativa generalizada do DOM ou de recarregamentos completos da aplicação. Diferentes categorias de estado poderão utilizar mecanismos distintos de atualização, conforme sua natureza e necessidade de sincronização.

E isso fecha 6.3.2 sem ainda escolher React, Vue, Svelte, Zustand, Redux, signals, etc. Essa escolha continua corretamente guardada para a 6.3.6 — Arquitetura de Frontend e Tecnologias.

O que 6.3.2 deixa decidido, então

No fim desse grande bloco, temos praticamente a arquitetura conceitual do estado:

                    SNOOPY V3
                        │
          ┌─────────────┴─────────────┐
          │                           │
    ESTADO DO SISTEMA          ESTADO DO CLIENTE
          │                           │
   ┌──────┼──────┐             ┌──────┼──────┐
   │      │      │             │      │      │
domínio investigação operação aplicação interface
   │      │      │             │      │      │
   └──────┴──────┴──────┬──────┴──────┴──────┘
                        │
                  representação
                     reativa
                        │
                        ▼
                       UI

Com as regras já estabelecidas:

Backend é autoridade do estado persistente.
Frontend é representação + estado próprio, não segunda fonte de verdade.
Estado persistente, investigação, aplicação e interface são coisas diferentes.
Cache não é persistência.
Estado investigativo relevante sobrevive à troca de superfície/página.
Estado puramente interacional pode morrer com a interface.
URL identifica/navega estado; não é a autoridade.
Servidor pode mudar independentemente do usuário.
Sincronização será composta: entrada + invalidação/refetch + eventos + polling/fallback.
Histórico investigativo não deve ser silenciosamente alterado porque o corpus mudou.
Ações, eventos e estados são conceitos distintos.
A interface deve reagir às mudanças de estado sem depender de manipulação imperativa generalizada.

Isso me parece suficiente para considerar 6.3.2 encerrada conceitualmente. A próxima etapa natural já não é descobrir mais uma categoria de estado; é entrar na arquitetura de frontend propriamente dita, onde finalmente vamos poder discutir como materializar tudo isso tecnologicamente.

Tem mais algo a ser discutido/investigado?

Tem mais algo a ser discutido/investigado?

Sim — mas não acho que valha abrir outra investigação grande dentro da 6.3.2.

O que ainda poderia ser discutido são alguns pontos de fechamento, mas eles já estão suficientemente determinados pelas rodadas anteriores:

Transições otimistas vs. confirmadas
Ex.: ao clicar em “sincronizar”, a UI mostra imediatamente “sincronizando” ou espera o backend confirmar a criação do job? Isso é detalhe de comportamento de comando/estado e pode ser decidido na 6.3.6/6.4.
Concorrência de ações do usuário
Ex.: iniciar duas buscas, cancelar uma enquanto outra está processando, trocar de consulta durante streaming. É relevante, mas já temos a base: estados explícitos + operações identificáveis + backend como autoridade. O desenho concreto pertence à arquitetura.
Atualização de estado em múltiplas superfícies/páginas
Como uma mudança ocorrida numa superfície repercute em outra. Isso será consequência natural da decisão de termos uma aplicação com múltiplas superfícies, estado persistente e sincronização composta.
Cancelamento/interrupção
Especialmente jobs e streaming. É importante, mas entra mais naturalmente em 6.3.5 (resiliência) e 6.4.
Reconexão e recuperação da representação local
O que acontece se o navegador perde conexão e volta? Também já temos os princípios; falta só materializar depois.

Então eu não abriria 6.3.2.7, 6.3.2.8 etc. Só para cobrir esses casos.

A 6.3.2 já respondeu a pergunta arquitetural fundamental:

Como o Snoopy V3 representa, mantém e atualiza seu estado sem transformar a interface numa segunda fonte de verdade?

E a resposta está suficientemente consolidada: estados diferenciados + persistência seletiva + backend como autoridade + sincronização composta + ações/eventos/estados distintos + representação reativa no cliente.

Eu consideraria 6.3.2 encerrada. O que sobrou são decisões de implementação arquitetural que pertencem à 6.3.6/6.4, não novas descobertas conceituais.

Tu acha que dá pra trabalhar todos esses pontos em um único grande bloco ou, no máximo, dois (primeiro responda sobre a possibilidade sem entrar na discussão ainda)?

Sim. Dá tranquilamente para trabalhar todos esses pontos em um único grande bloco, e eu acho que é a opção mais coerente.

Eles são todos variações de uma mesma questão: como a aplicação se comporta diante de ações, concorrência, interrupções, mudanças e perda/restabelecimento de conexão.

Se quisermos separar em dois, faria sentido apenas por organização:

Comportamento das ações e transições — otimista vs. confirmado, concorrência, cancelamento/interrupção.
Resiliência da representação — reconexão, recuperação do estado e sincronização entre superfícies.

Mas eu preferiria um único bloco, porque as decisões são fortemente interdependentes. Se separarmos demais, corremos o risco de repetir as mesmas ideias de estado, evento, autoridade e sincronização que acabamos de fechar.

Então: um último grande bloco para fechar 6.3.2 é suficiente.

Então vamos discutir o que falta ser discutido em um último grande bloco de discussão da 6.3.2

Perfeito. Vamos tratar isso como o último grande bloco da 6.3.2, sem abrir novas microetapas.

Até aqui já definimos o que é estado, quem é autoridade, o que persiste, como sincroniza e como ações/eventos/estados se relacionam. Agora falta fechar o comportamento da aplicação diante das situações em que esse modelo é realmente colocado à prova.

Eu dividiria a discussão em quatro questões dentro do mesmo bloco:

1. Ação do usuário → estado confirmado

Quando o usuário faz alguma coisa, temos duas possibilidades extremas:

usuário clica
   ↓
UI muda imediatamente
   ↓
backend confirma ou rejeita

ou:

usuário clica
   ↓
backend processa
   ↓
backend confirma
   ↓
UI muda

Isso aparece em coisas como:

iniciar sincronização;
cancelar processamento;
selecionar/remover algo;
iniciar uma busca;
alterar uma configuração;
eventualmente excluir ou substituir uma versão.

A questão é quando a interface pode assumir um estado e quando precisa esperar a autoridade do backend.

2. Concorrência e ações sobrepostas

Depois:

O que acontece quando existem várias operações acontecendo simultaneamente?

Por exemplo:

Busca A ────────────────►
       Busca B ───────►
              Usuário abre documento
                      Sincronização começa

Ou duas sincronizações sendo solicitadas, uma busca sendo abandonada enquanto o streaming continua etc.

Precisamos decidir se a interface:

cancela automaticamente operações anteriores;
permite várias operações simultâneas;
mantém cada operação identificável;
diferencia operação atual de operação histórica;
impede ações incompatíveis;
permite que o usuário acompanhe várias coisas ao mesmo tempo.

Isso é particularmente importante porque o V3 deixa de ser uma página única com uma operação dominante.

3. Cancelamento, interrupção e falha

Aqui entra uma distinção que acho importante:

cancelar ≠ falhar.

Se o pesquisador interrompe uma busca:

running → cancelled

isso não é:

running → failed

Da mesma forma:

conexão caiu;
servidor recusou;
processamento falhou;
usuário cancelou;
operação terminou parcialmente;
operação terminou mas precisa de revisão;

são situações diferentes.

A interface precisa representar essas diferenças sem transformar tudo em aquele clássico:

❌ “Ocorreu um erro.”

Isso conversa diretamente com 6.2.8, mas aqui estamos investigando como o pesquisador percebe e controla esses estados.

4. Perda de conexão, reconexão e recuperação

Por fim:

Se a interface perder contato com o servidor, o que exatamente deve acontecer com o estado que o pesquisador estava vendo?

Esse é provavelmente o fechamento mais importante.

Imagine:

pesquisa em andamento
       ↓
conexão cai
       ↓
???

A UI não deveria simplesmente esquecer tudo.

Precisamos distinguir:

estado local que continua válido;
estado do servidor que pode estar desatualizado;
operação que continua rodando no backend;
operação cuja situação é desconhecida;
informação que precisa ser revalidada;
informação histórica que pode continuar sendo exibida.

E isso nos leva a uma regra que provavelmente será importante para o V3:

perda de conexão não significa perda de estado.

O navegador pode perder a comunicação com o servidor sem que uma investigação, um job ou um resultado tenham deixado de existir.

A questão que eu colocaria no centro deste último bloco

Em vez de investigar quatro assuntos isoladamente, eu começaria com uma pergunta única:

Como o Snoopy V3 deve se comportar quando uma ação do pesquisador ainda não foi confirmada, quando existem operações simultâneas ou quando a comunicação com o servidor é interrompida?

A partir dela, vamos decidir:

ação → confirmação → concorrência → cancelamento → falha → reconexão → recuperação

E no final precisamos sair com uma política geral de comportamento da aplicação, não com uma lista de soluções pontuais.

Essa, para mim, é realmente a última peça que falta para fechar 6.3.2 Estado, Dados e Interação da Aplicação.

Bora lá. Se possível, compilar as investigações para não alongar muito mais, abordando os pontos que, de fato, são necessários para chegar numa conclusao e sem enrolar e, ainda, quando possível, colocando mais de um tema em sequência. Precisamos concluir logo e sem deixar pontas soltas kkkk já entramos no sábado kkkkkk 

KKKK sim, já é sábado. Então vamos fazer exatamente isso: um último bloco compacto, juntando os assuntos que são dependentes entre si e parando assim que tivermos uma política coerente. Sem transformar cada detalhe em uma tese de doutorado.

6.3.2 — Último bloco: ações, concorrência, interrupção e recuperação
1. Ação do usuário e confirmação do servidor

A regra geral deve ser:

A interface pode representar imediatamente uma intenção do usuário, mas não deve representar como fato consumado uma mudança persistente que o servidor ainda não confirmou.

Isso permite uma experiência responsiva sem transformar o frontend em autoridade.

Exemplo:

Usuário → "Sincronizar"
           ↓
UI → "solicitação enviada / iniciando..."
           ↓
Backend cria Job
           ↓
evento/estado → Job = processing
           ↓
UI → "Sincronizando..."

Se o backend rejeitar:

UI → "Não foi possível iniciar a sincronização"

Não:

UI → "Sincronizando..."

e depois ficar tentando descobrir o que aconteceu.

Decisão

Para operações persistentes ou relevantes ao sistema, o frontend deve diferenciar:

intenção do usuário → operação aceita → operação em andamento → resultado confirmado.

Isso não significa que toda interação precise esperar o servidor. Estados puramente locais — abrir modal, expandir painel, mudar aba, selecionar visualmente algo — podem ser imediatos.

2. Concorrência: não tratar a aplicação como se houvesse uma única operação

Isso fica especialmente importante no V3 porque já decidimos que a experiência terá múltiplas superfícies e consultas independentes.

Não devemos assumir:

"existe uma operação atual do Snoopy."

Devemos assumir:

existem operações identificáveis, cada uma com seu próprio estado.

Por exemplo:

Consulta A → completed
Consulta B → processing
Sincronização → processing
Documento X → active
Documento Y → processing

A interface pode representar todas essas coisas simultaneamente.

Isso também resolve uma fragilidade conceitual do V2: não precisamos usar uma variável global do tipo isLoading para representar a aplicação inteira.

Pode existir:

search.loading
sync.loading
document.loading

ou, melhor ainda, estados derivados de operações identificáveis.

Decisão

Operações assíncronas deverão possuir identidade e ciclo de vida próprios. A interface não deverá pressupor uma única operação global em andamento nem utilizar um estado genérico de carregamento para representar operações independentes.

Isso conversa diretamente com jobs, investigação e múltiplas superfícies.

3. Cancelamento ≠ falha ≠ interrupção de comunicação

Aqui podemos fechar três conceitos de uma vez.

Cancelamento

O pesquisador pediu para parar.

processing → cancelled
Falha

A operação não conseguiu completar por algum problema.

processing → failed
Comunicação perdida

O navegador deixou de saber o estado atual.

processing → unknown locally

O terceiro caso é particularmente importante.

A perda de conexão não autoriza a interface a concluir que a operação falhou.

O backend pode continuar processando perfeitamente.

Então:

conexão perdida
      ↓
estado local = desconhecido
      ↓
reconectar
      ↓
consultar estado real
      ↓
processing / completed / failed / cancelled
Decisão

Cancelamento, falha e perda de comunicação deverão permanecer estados conceitualmente distintos. A perda de conexão não deverá ser interpretada automaticamente como falha da operação ou perda do estado persistente.

Isso fecha uma ponta importante da resiliência sem invadir ainda a 6.3.5.

4. Reconexão e recuperação

Agora juntamos reconexão com tudo que decidimos sobre sincronização.

Quando a conexão volta, não precisamos simplesmente "recarregar a página".

O cliente deve:

identificar quais estados podem estar desatualizados;
invalidar o que perdeu validade;
recuperar o estado do servidor;
atualizar as representações afetadas;
preservar o que continua válido localmente.

Exemplo:

Usuário está vendo Consulta A
        ↓
internet cai
        ↓
UI mantém Consulta A visível
        ↓
usuário continua podendo navegar no que já possui
        ↓
conexão volta
        ↓
estado do servidor é revalidado
        ↓
UI atualiza o que mudou

Isso é particularmente importante para a experiência de exploração que definimos em 6.3.1.

O pesquisador não deveria perder a trilha de exploração porque o Wi-Fi deu uma morrida. KKKK

Decisão

A recuperação de comunicação deverá ser baseada em revalidação e sincronização do estado, e não em reconstrução indiscriminada da interface. Estados persistentes e investigações já recuperadas deverão permanecer disponíveis enquanto sua validade puder ser preservada.

5. Atualização otimista: onde usar e onde não usar

Acho que aqui também não precisamos abrir uma investigação separada.

Pode ser otimista

Coisas puramente locais:

abrir/fechar painel;
trocar aba;
expandir evidência;
selecionar item visualmente;
mudar tema;
alterar preferência local.
Deve aguardar confirmação

Coisas que alteram o estado governado pelo Snoopy:

iniciar processamento;
cancelar job;
alterar corpus;
remover documento;
publicar nova versão;
alterar configurações persistentes;
qualquer ação de autorização/permissão.
Decisão

Atualizações otimistas poderão ser utilizadas para interações puramente locais e reversíveis, mas não deverão representar como confirmado um estado persistente cuja alteração ainda não tenha sido aceita pelo sistema.

Pronto. Não precisamos discutir optimistic UI por mais duas horas. 😂

6. O fechamento conceitual

Agora dá para juntar tudo:

AÇÃO DO USUÁRIO
       ↓
solicitação
       ↓
backend / estado local
       ↓
┌───────────────────────────────┐
│                               │
│   operação aceita             │
│        ↓                      │
│   processing                  │
│        ↓                      │
│   completed / failed /        │
│   cancelled                   │
│                               │
└───────────────────────────────┘
       ↓
evento / mudança de estado
       ↓
representação reativa
       ↓
interface

Se houver perda de comunicação:

operação em andamento
        ↓
conexão perdida
        ↓
estado local possivelmente desconhecido
        ↓
reconexão
        ↓
revalidação
        ↓
estado real

E, fundamentalmente:

o cliente nunca deve confundir sua própria percepção momentânea com a verdade do sistema.

Conclusão deste último bloco

A política consolidada que eu colocaria para a 6.3.2 é:

O Snoopy V3 deverá tratar ações do usuário, operações assíncronas, eventos e estados como conceitos distintos, permitindo que cada operação possua identidade e ciclo de vida próprios. Alterações persistentes deverão ser consideradas confirmadas somente após aceitação ou atualização proveniente da autoridade do sistema, enquanto interações puramente locais poderão utilizar atualizações imediatas e otimistas. Operações concorrentes deverão permanecer independentes e identificáveis, evitando estados globais de carregamento ou operação. Cancelamento, falha e perda de comunicação deverão ser diferenciados, e a interrupção da comunicação não deverá ser interpretada como falha ou perda do estado persistente. Após a reconexão, o cliente deverá revalidar e sincronizar os estados potencialmente desatualizados, preservando tanto quanto possível o contexto de investigação e as informações já recuperadas. A interface deverá refletir essas transições por meio de uma arquitetura reativa, sem depender de recarregamentos completos ou de manipulação imperativa generalizada do DOM.

E, com isso, eu fecharia a 6.3.2.

Não vejo mais nenhuma ponta conceitual que justifique outra rodada de investigação aqui.

O que restou — qual mecanismo de estado, qual modelo de componentes, qual router, qual biblioteca de reatividade, como implementar SSE, como organizar as páginas, etc. — é precisamente o que deve aparecer depois, na arquitetura de frontend/tecnologias e, principalmente, na 6.4, quando vamos juntar frontend + backend + persistência + recuperação + provenance em uma arquitetura única.

Ou seja: não precisamos mais cavar o buraco da 6.3.2. Já chegamos no fundo. KKKK