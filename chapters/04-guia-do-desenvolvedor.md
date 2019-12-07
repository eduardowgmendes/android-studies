## Visão geral dos serviços

Um Service é um componente do aplicativo que pode realizar operações longas e não fornece uma interface do usuário. Outro componente do aplicativo pode iniciar um serviço e ele continuará em execução em segundo plano mesmo que o usuário alterne para outro aplicativo. Além disso, um componente poderá se vincular a um serviço para interagir com ele e até estabelecer comunicação entre processos (IPC). Por exemplo, um serviço pode lidar com transações de rede, reproduzir música, executar E/S de arquivos ou interagir com um provedor de conteúdo, tudo a partir do segundo plano.

Veja os três tipos diferentes de serviços:

### Primeiro plano
O serviço em primeiro plano realiza uma operação que é perceptível ao usuário. Por exemplo, um aplicativo de áudio usaria um serviço em primeiro plano para reproduzir uma faixa de áudio. Serviços em primeiro plano precisam exibir uma Notificação. Serviços em primeiro plano continuam a executar mesmo se o usuário não estiver interagindo com o aplicativo. 

### Segundo plano
O serviço em segundo plano realiza uma operação que não é perceptível ao usuário. Por exemplo, o serviço usado por um aplicativo para compactar armazenamento normalmente é de segundo plano. 

**Observação**: se seu aplicativo é direcionado à API de nível 26 ou posterior, o sistema impõe restrições à execução de serviços em segundo plano quando o próprio aplicativo não estiver em primeiro plano. Na maioria desses casos, o aplicativo usará um job programado. 

### Vinculado
Um serviço é vinculado quando um componente do aplicativo chama `bindService()` para vinculá-lo. Um serviço vinculado oferece uma interface servidor-cliente que permite que os componentes interajam com a interface, enviem solicitações, recebam resultados, mesmo em processos com comunicação interprocessual (IPC). Um serviço vinculado permanece em execução somente enquanto outro componente do aplicativo estiver vinculado a ele. Vários componentes podem ser vinculados ao serviço de uma só vez, mas quando todos desfizerem o vínculo, o serviço será eliminado.
