## Intents e filtros de intents

O Intent é um objeto de mensagem que pode ser usado para solicitar uma ação de outro componente do aplicativo. Embora os intents facilitem a comunicação entre os componentes de diversas formas, há três casos fundamentais de uso:

* Como iniciar uma atividade
	* Uma Activity representa uma única tela em um aplicativo. É possível iniciar uma nova 	instância de uma Activity passando um Intent para startActivity(). O Intent descreve a atividade a iniciar e carrega todos os dados necessários.Se você quiser receber um resultado da atividade quando ela finalizar, chame startActivityForResult(). Sua atividade recebe o resultado como um objeto Intent separado no callback de onActivityResult() da atividade. Para saber mais, consulte o guia Activities.

* Como iniciar um serviço
	* O Service é um componente que realiza operações em segundo plano sem interface do usuário. Com o Android 5.0 (API nível 21) e posteriores, é possível iniciar um serviço JobScheduler. Para saber mais sobre JobScheduler, consulte a API-reference documentation.

Para versões anteriores ao Android 5.0 (API nível 21), é possível iniciar um serviço usando os métodos da classe Service. É possível iniciar um serviço para realizar uma operação que acontece uma vez (como fazer o download de um arquivo) passando um Intent a startService(). O Intent descreve o serviço a iniciar e carrega todos os dados necessários.

Se o serviço for projetado com uma interface servidor-cliente, é possível vincular ao serviço em outro componente passando um Intent a bindService(). Para mais informações, consulte o guia Serviços.
