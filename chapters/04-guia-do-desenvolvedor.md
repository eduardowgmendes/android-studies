## Visão geral dos serviços

Um Service é um componente do aplicativo que pode realizar operações longas e não fornece uma interface do usuário. Outro componente do aplicativo pode iniciar um serviço e ele continuará em execução em segundo plano mesmo que o usuário alterne para outro aplicativo. Além disso, um componente poderá se vincular a um serviço para interagir com ele e até estabelecer comunicação entre processos (IPC). Por exemplo, um serviço pode lidar com transações de rede, reproduzir música, executar E/S de arquivos ou interagir com um provedor de conteúdo, tudo a partir do segundo plano.

Veja os três tipos diferentes de serviços:

### Primeiro plano
O serviço em primeiro plano realiza uma operação que é perceptível ao usuário. Por exemplo, um aplicativo de áudio usaria um serviço em primeiro plano para reproduzir uma faixa de áudio. Serviços em primeiro plano precisam exibir uma Notificação. Serviços em primeiro plano continuam a executar mesmo se o usuário não estiver interagindo com o aplicativo. 

### Segundo plano
O serviço em segundo plano realiza uma operação que não é perceptível ao usuário. Por exemplo, o serviço usado por um aplicativo para compactar armazenamento normalmente é de segundo plano. 

**Observação**: se seu aplicativo é direcionado à API de nível 26 ou posterior, o sistema impõe [restrições à execução de serviços em segundo plano]() quando o próprio aplicativo não estiver em primeiro plano. Na maioria desses casos, o aplicativo usará um job programado. 

### Vinculado
Um serviço é vinculado quando um componente do aplicativo chama `bindService()` para vinculá-lo. Um serviço vinculado oferece uma interface servidor-cliente que permite que os componentes interajam com a interface, enviem solicitações, recebam resultados, mesmo em processos com comunicação interprocessual (IPC). Um serviço vinculado permanece em execução somente enquanto outro componente do aplicativo estiver vinculado a ele. Vários componentes podem ser vinculados ao serviço de uma só vez, mas quando todos desfizerem o vínculo, o serviço será eliminado.

## Limites da execução em segundo plano
Sempre que um aplicativo é executado em segundo plano, ele consome parte dos recursos limitados do dispositivo, como a RAM. Isso pode resultar em uma experiência reduzida para o usuário, especialmente se ele estiver usando um aplicativo que consuma muitos recursos, como um jogo ou um vídeo. Para aprimorar a experiência do usuário, o Android 8.0 (API de nível 26) restringe o que os aplicativos podem fazer durante a execução em segundo plano. Este documento descreve as mudanças no sistema operacional e como você pode atualizar seu aplicativo para que ele funcione adequadamente com as novas restrições.

### Visão geral
Muitos aplicativos e serviços do Android podem ser executados simultaneamente. Por exemplo, um usuário pode jogar em uma janela, navegar na Web em outra e usar um terceiro aplicativo para reproduzir música. Quanto mais aplicativos são executados ao mesmo tempo, maior será a carga para o sistema. Se mais aplicativos ou serviços forem executados em segundo plano, isso deixará o sistema sobrecarregado, o que pode resultar em uma experiência reduzida para o usuário. Por exemplo, o aplicativo de música pode ser desligado repentinamente.

Para diminuir a chance desses problemas, o Android 8.0 estabelece restrições sobre o que os aplicativos podem fazer enquanto os usuários não estão interagindo diretamente com eles. Os aplicativos são limitados de duas maneiras:

Restrições de serviços em segundo plano: enquanto um aplicativo está ocioso, há limites para o uso dos serviços em segundo plano. Isso não se aplica aos serviços em primeiro plano, que são mais evidentes para o usuário.

Restrições de transmissão: com exceções limitadas, os aplicativos não podem usar os manifestos para registrar-se em transmissões implícitas. Os aplicativos ainda poderão se registrar para essas transmissões no tempo de execução e usar o manifesto para registro em transmissões explícitas direcionadas especificamente a eles.

**Observação**: por padrão, essas restrições se aplicam somente a aplicativos direcionados ao Android 8.0 (API de nível 26) ou superior. No entanto, os usuários podem ativar a maioria dessas restrições para qualquer aplicativo na tela Configurações, mesmo que o aplicativo seja direcionado a um nível de API menor que 26.

Na maioria dos casos, os aplicativos podem contornar essas restrições usando jobs JobScheduler. Essa abordagem permite que um aplicativo realize atividades quando ele não está sendo executado ativamente, mas ainda dá ao sistema liberdade para programar esses jobs de uma forma que não afete a experiência do usuário. O Android 8.0 oferece diversas melhorias para JobScheduler que facilitam a substituição de serviços e broadcast receivers com jobs programados. Para saber mais, consulte Melhorias no JobScheduler.

### Restrições de serviços em segundo plano

Os serviços executados em segundo plano podem consumir recursos do dispositivo, gerando uma experiência ruim para o usuário. Para mitigar esse problema, o sistema aplica diversas restrições aos serviços.

O sistema distingue entre aplicativos de primeiro plano e de segundo plano. A definição de segundo plano para fins de restrições de serviço é diferente da definição usada pelo gerenciamento de memória. Um aplicativo pode estar no segundo plano em termos de gerenciamento de memória, mas em primeiro plano em termos da capacidade de iniciar serviços. Um aplicativo será considerado como estando em primeiro plano se qualquer uma das condições a seguir for verdadeira:
