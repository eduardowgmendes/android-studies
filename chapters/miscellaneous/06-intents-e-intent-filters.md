## Intents e filtros de intents

O Intent é um objeto de mensagem que pode ser usado para solicitar uma ação de outro componente do aplicativo. Embora os intents facilitem a comunicação entre os componentes de diversas formas, há três casos fundamentais de uso:

### Iniciar uma atividade
Uma Activity representa uma única tela em um aplicativo. É possível iniciar uma nova 	instância de uma Activity passando um Intent para startActivity(). O Intent descreve a atividade a iniciar e carrega todos os dados necessários.Se você quiser receber um resultado da atividade quando ela finalizar, chame startActivityForResult(). Sua atividade recebe o resultado como um objeto Intent separado no callback de onActivityResult() da atividade. Para saber mais, consulte o guia Activities.

### Iniciar um serviço
O Service é um componente que realiza operações em segundo plano sem interface do usuário. Com o Android 5.0 (API nível 21) e posteriores, é possível iniciar um serviço JobScheduler. Para saber mais sobre JobScheduler, consulte a API-reference documentation. Para versões anteriores ao Android 5.0 (API nível 21), é possível iniciar um serviço usando os métodos da classe Service. É possível iniciar um serviço para realizar uma operação que acontece uma vez (como fazer o download de um arquivo) passando um Intent a startService(). O Intent descreve o serviço a iniciar e carrega todos os dados necessários. Se o serviço for projetado com uma interface servidor-cliente, é possível vincular ao serviço em outro componente passando um Intent a bindService(). Para mais informações, consulte o guia Serviços.

### Fornecer uma transmissão
Transmissão é uma mensagem que qualquer aplicativo pode receber. O sistema fornece diversas transmissões para eventos do sistema, como quando o sistema inicializa ou o dispositivo inicia o carregamento. Você pode fornecer uma transmissão a outros aplicativos passando um Intent ao sendBroadcast() ou ao sendOrderedBroadcast().

O restante desta página explica como os intents funcionam e como usá-los. Para informações relacionadas, consulte Interação com outros aplicativos e Compartilhamento de conteúdo.  

## Tipos de intents

Há dois tipos de intents:

* Os intents explícitos especificam qual aplicativo atenderá ao intent, fornecendo o nome do pacote do aplicativo de destino ou o nome da classe de um componente totalmente qualificado. Normalmente, usa-se um intent explícito para iniciar um componente no próprio aplicativo porque se sabe o nome de classe da atividade ou do serviço que se quer iniciar. Por exemplo, iniciar uma nova atividade em resposta a uma ação do usuário ou iniciar um serviço para fazer o download de um arquivo em segundo plano.
* Os intents implícitos não nomeiam nenhum componente específico, mas declaram uma ação geral a realizar, o que permite que um componente de outro aplicativo a processe. Por exemplo, se você quiser exibir ao usuário uma localização em um mapa, pode usar um intent implícito para solicitar que outro aplicativo capaz exiba uma localização especificada no mapa.

A figura 1 mostra como um intent é usado ao iniciar uma atividade. Quando o objeto Intent nomeia um componente específico da atividade de forma explícita, o sistema inicia o componente de imediato.

![Intent implícito](https://raw.githubusercontent.com/eduardowgmendes/android-studies/master/images/intent-filters_2x.png)

**Figura1**: Ilustração de como um intent implícito é fornecido pelo sistema para iniciar outra atividade: 

1. A atividade A cria um Intent com uma descrição de ação e a passa para `startActivity()`. 
2. O sistema Android busca, em todos os aplicativos, um filtro de intents que corresponda ao intent. 
3. Ao encontrar uma correspondência o sistema inicia a atividade correspondente (Atividade B) chamando seu método onCreate() e passando-lhe o Intent. 

Ao criar um intent implícito, o sistema Android encontra o componente adequado para iniciar, comparando o conteúdo do intent aos filtros de intents declarados no arquivo de manifesto de outros aplicativos no dispositivo. Se o intent corresponder a um filtro de intents, o sistema iniciará esse componente e entregará o objeto Intent. Se diversos filtros de intents corresponderem, o sistema exibirá uma caixa de diálogo para que o usuário selecione o aplicativo que deseja usar.

O filtro de intents é uma expressão em um arquivo de manifesto do aplicativo que especifica o tipo de intents que o componente gostaria de receber. Por exemplo, ao declarar um filtro de intents para uma atividade, outros aplicativos se tornam capazes de iniciar diretamente sua atividade com o determinado tipo de intent. Do mesmo modo, se você não declarar nenhum filtro de intents para uma atividade, ela poderá ser iniciada somente com um intent explícito.

**Atenção**: para garantir a segurança do seu aplicativo, sempre use um intent explícito ao iniciar um Service e não declare filtros de intents para os serviços. O uso de um intent implícito para iniciar um serviço representa um risco de segurança, porque não é possível determinar qual serviço responderá ao intent e o usuário não poderá ver que serviço é iniciado. A partir do Android 5.0 (API de nível 21), o sistema lança uma exceção ao chamar bindService() com um intent implícito.

## Criação de um intent

Um objeto Intent carrega informações que o sistema Android usa para determinar o componente a iniciar (como o nome exato do componente ou categoria do componente que deve receber o intent), além de informações que o componente receptor usa para realizar a ação adequadamente (como a ação a tomar e os dados a usar).

As informações principais contidas em um Intent são as seguintes:

### Nome do componente
É o nome do componente a iniciar. É opcional, mas é a informação fundamental que torna um intent explícito, o que significa que o intent deve ser entregue somente ao componente do aplicativo definido pelo nome do componente. Sem nome de componente, o intent será implícito, e o sistema decidirá qual componente deve receber o intent com base nas informações de outro intent (como a ação, os dados e a categoria — descritos abaixo). Portanto, se for necessário iniciar um componente específico no seu aplicativo, você deverá especificar o nome do componente.

**Observação**: ao iniciar um Service, deve-se sempre especificar o nome do componente. Caso contrário, não será possível determinar qual serviço responderá ao intent e o usuário não poderá ver que serviço é iniciado.

Esse campo do Intent é um objeto ComponentName que pode ser especificado usando um nome de classe totalmente qualificado do componente-alvo, inclusive o nome do pacote do aplicativo. Por exemplo, com.example.ExampleActivity. É possível definir o nome do componente com setComponent(), setClass(), setClassName() ou com o construtor Intent.

### Ação

String que especifica a ação genérica a realizar (como exibir ou selecionar).

No caso de um intent de transmissão, essa é a ação que entrou em vigor e que está sendo relatada. A ação determina amplamente como o resto do intent é estruturado — especificamente, o que está contido nos dados e em extras.

É possível especificar as ações para uso por intents dentro do aplicativo (ou para uso por outros aplicativos para chamar componentes no seu aplicativo), mas normalmente são usadas constantes de ação definidas pela classe Intent ou por outras classes de biblioteca. Veja algumas ações comuns para iniciar uma atividade:


* `ACTION_VIEW`
Use essa ação em um intent com startActivity() quando houver informações de que uma atividade possa ser exibida ao usuário, como uma foto para exibição em um aplicativo de galeria ou um endereço para exibição em um aplicativo de mapa.

* `ACTION_SEND`
Também conhecida como o intent de share, deve ser usada em um intent com startActivity() quando houver alguns dados que o usuário possa compartilhar por meio de outro aplicativo, como um app de e-mails ou de compartilhamento social.

Consulte a referência da classe [Intent](https://developer.android.com/reference/android/content/Intent.html) para saber mais constantes que definem ações genéricas. Outras ações são definidas em outros locais na biblioteca do Android, como nas Settings para ações que abrem telas específicas no aplicativo de Configurações do sistema.

É possível especificar a ação para um intent com setAction() ou com um construtor Intent.

Se você definir as próprias ações, verifique se incluiu o nome do pacote do seu aplicativo como prefixo, conforme o exemplo a seguir:

```java

static final String ACTION_TIMETRAVEL = "com.example.action.TIMETRAVEL";

```

### Dados
É o URI (um objeto Uri) que referencia os dados a serem aproveitados e/ou o tipo MIME desses dados. O tipo dos dados fornecidos geralmente é determinado pela ação do intent. Por exemplo, se a ação for ACTION_EDIT, os dados deverão conter o URI do documento a editar.

Ao criar um intent, em geral, é importante especificar o tipo de dados (seu tipo MIME) em adição ao URI. Por exemplo: uma atividade capaz de exibir imagens provavelmente não será capaz de reproduzir um arquivo de áudio, mesmo que os formatos do URI sejam similares. Portanto, especificar o tipo MIME dos dados ajuda o sistema Android a encontrar o melhor componente para receber o intent. Contudo, o tipo MIME, às vezes, pode ser inferido a partir do URI — especificamente quando os dados são um URI de content:. A URI de content: indica que os dados se localizam no dispositivo e são controlados por um ContentProvider, que torna o tipo MIME dos dados visível para o sistema.

Para definir só um URI de dados, chame setData(). Para definir só o tipo MIME, chame setType(). Se necessário, é possível definir ambos explicitamente com setDataAndType().

**Atenção**: se você desejar definir o URI e o tipo MIME, não chame setData() e setType(), pois um anula o outro. Sempre use setDataAndType() para definir o URI e o tipo MIME juntos.

### Categoria
É uma string que contém informações adicionais sobre o tipo de componente que deve processar o intent. Qualquer número de descrições de categoria pode ser inserido em um intent, mas a maioria dos intents não requer nenhuma categoria. Veja algumas categorias comuns: 


* CATEGORY_BROWSABLE
A atividade-alvo permite ser iniciada por um navegador da Web para exibir dados referenciados por um link, como uma imagem ou uma mensagem de e-mail. 

* CATEGORY_LAUNCHER
A atividade é a atividade inicial de uma tarefa e é listada no inicializador do aplicativo do sistema. 

Consulte a descrição da classe [Intent](https://developer.android.com/reference/android/content/Intent.html#standard-categories) para ver a lista completa de categorias.

É possível especificar uma categoria com `addCategory()`.

As propriedades listadas abaixo (nome do componente, ação, dados e categoria) representam as características de definição de um intent. Ao ler estas propriedades, o sistema Android será capaz de definir o componente de aplicativo que ele deve iniciar. Contudo, um intent pode carregar informações adicionais que não afetam o modo com que é tratado para um componente do aplicativo. Os intents também podem fornecer o seguinte:

### Extras
São pares de chave-valor que carregam informações adicionais necessárias para realizar a ação solicitada. Assim como algumas ações usam determinados tipos de URIs de dados, outras também usam determinados extras.

É possível adicionar dados extras com diversos métodos putExtra(), cada um aceitando dois parâmetros: o nome principal e o valor. Também é possível criar um objeto Bundle com todos os dados extras e, em seguida, inserir o Bundle no elemento Intent com putExtras().

Por exemplo, ao criar um intent para enviar um e-mail com ACTION_SEND, é possível especificar o recipiente to com a chave EXTRA_EMAIL e especificar subject com a chave EXTRA_SUBJECT.

A classe Intent especifica diversas constantes EXTRA_* para tipos de dados padronizados. Se for necessário declarar chaves extras (para intents que seu aplicativo receba), certifique-se de incluir o nome do pacote do aplicativo como prefixo, conforme o exemplo abaixo:

```java

static final String EXTRA_GIGAWATTS = "com.example.EXTRA_GIGAWATTS";

```

**Atenção**: não use dados Parcelable ou Serializable ao enviar um intent que você espera que outro aplicativo receba. Caso um aplicativo tente acessar dados em um objeto Bundle, mas não tenha acesso à classe parcelada ou serializada, o sistema cria um `RuntimeException`.

### Sinalizadores

Sinalizadores são definidos na classe Intent e funcionam como metadados para o intent. Os sinalizadores podem instruir o sistema Android a inicializar uma atividade (por exemplo, a qual tarefa a atividade deve pertencer) e como tratá-la após a inicialização (por exemplo, se ela pertence a uma lista de atividades recentes).

Para mais informações, consulte o método [`setFlags()`](https://developer.android.com/reference/android/content/Intent.html#setFlags(int)). 
