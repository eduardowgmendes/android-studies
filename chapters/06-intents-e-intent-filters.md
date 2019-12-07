## Intents e filtros de intents

O Intent é um objeto de mensagem que pode ser usado para solicitar uma ação de outro componente do aplicativo. Embora os intents facilitem a comunicação entre os componentes de diversas formas, há três casos fundamentais de uso:

* Como iniciar uma atividade
	* Uma Activity representa uma única tela em um aplicativo. É possível iniciar uma nova 	instância de uma Activity passando um Intent para startActivity(). O Intent descreve a atividade a iniciar e carrega todos os dados necessários.Se você quiser receber um resultado da atividade quando ela finalizar, chame startActivityForResult(). Sua atividade recebe o resultado como um objeto Intent separado no callback de onActivityResult() da atividade. Para saber mais, consulte o guia Activities.

* Como iniciar um serviço
	* O Service é um componente que realiza operações em segundo plano sem interface do usuário. Com o Android 5.0 (API nível 21) e posteriores, é possível iniciar um serviço JobScheduler. Para saber mais sobre JobScheduler, consulte a API-reference documentation. Para versões anteriores ao Android 5.0 (API nível 21), é possível iniciar um serviço usando os métodos da classe Service. É possível iniciar um serviço para realizar uma operação que acontece uma vez (como fazer o download de um arquivo) passando um Intent a startService(). O Intent descreve o serviço a iniciar e carrega todos os dados necessários. Se o serviço for projetado com uma interface servidor-cliente, é possível vincular ao serviço em outro componente passando um Intent a bindService(). Para mais informações, consulte o guia Serviços.

* Como fornecer uma transmissão
	* Transmissão é uma mensagem que qualquer aplicativo pode receber. O sistema fornece diversas transmissões para eventos do sistema, como quando o sistema inicializa ou o dispositivo inicia o carregamento. Você pode fornecer uma transmissão a outros aplicativos passando um Intent ao sendBroadcast() ou ao sendOrderedBroadcast().

O restante desta página explica como os intents funcionam e como usá-los. Para informações relacionadas, consulte Interação com outros aplicativos e Compartilhamento de conteúdo.  

## Tipos de intents

Há dois tipos de intents:


* Os intents explícitos especificam qual aplicativo atenderá ao intent, fornecendo o nome do pacote do aplicativo de destino ou o nome da classe de um componente totalmente qualificado. Normalmente, usa-se um intent explícito para iniciar um componente no próprio aplicativo porque se sabe o nome de classe da atividade ou do serviço que se quer iniciar. Por exemplo, iniciar uma nova atividade em resposta a uma ação do usuário ou iniciar um serviço para fazer o download de um arquivo em segundo plano.
* Os intents implícitos não nomeiam nenhum componente específico, mas declaram uma ação geral a realizar, o que permite que um componente de outro aplicativo a processe. Por exemplo, se você quiser exibir ao usuário uma localização em um mapa, pode usar um intent implícito para solicitar que outro aplicativo capaz exiba uma localização especificada no mapa.

A figura 1 mostra como um intent é usado ao iniciar uma atividade. Quando o objeto Intent nomeia um componente específico da atividade de forma explícita, o sistema inicia o componente de imediato.

![Intent implícito]()

**Figura1**: Ilustração de como um intent implícito é fornecido pelo sistema para iniciar outra atividade: [1] A atividade A cria um Intent com uma descrição de ação e a passa para startActivity(). [2] O sistema Android busca, em todos os aplicativos, um filtro de intents que corresponda ao intent. Ao encontrar uma correspondência, [3] o sistema inicia a atividade correspondente (Atividade B) chamando seu método onCreate() e passando-lhe o Intent. 

Ao criar um intent implícito, o sistema Android encontra o componente adequado para iniciar, comparando o conteúdo do intent aos filtros de intents declarados no arquivo de manifesto de outros aplicativos no dispositivo. Se o intent corresponder a um filtro de intents, o sistema iniciará esse componente e entregará o objeto Intent. Se diversos filtros de intents corresponderem, o sistema exibirá uma caixa de diálogo para que o usuário selecione o aplicativo que deseja usar.

O filtro de intents é uma expressão em um arquivo de manifesto do aplicativo que especifica o tipo de intents que o componente gostaria de receber. Por exemplo, ao declarar um filtro de intents para uma atividade, outros aplicativos se tornam capazes de iniciar diretamente sua atividade com o determinado tipo de intent. Do mesmo modo, se você não declarar nenhum filtro de intents para uma atividade, ela poderá ser iniciada somente com um intent explícito.

**Atenção**: para garantir a segurança do seu aplicativo, sempre use um intent explícito ao iniciar um Service e não declare filtros de intents para os serviços. O uso de um intent implícito para iniciar um serviço representa um risco de segurança, porque não é possível determinar qual serviço responderá ao intent e o usuário não poderá ver que serviço é iniciado. A partir do Android 5.0 (API de nível 21), o sistema lança uma exceção ao chamar bindService() com um intent implícito.

