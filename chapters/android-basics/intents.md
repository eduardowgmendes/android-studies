## Intents 
A classe `android.content.Intent` é o coração do Android. Uma intent é uma mensagem da aplicação para o sistema operacional, solicitando que algo seja realizado, e representa um importante papel na arquitetura do Android para integrar diferentes aplicações. 

Cabe ao sistema operacional interpretar essa mensagem e tomar as providências necessárias, que pode ser abrir um aplicativo, fazer uma ligação para determinado número, tirar uma foto ou até abrir o browser em uma página da internet. 

## Intent - envio de mensagem ao Android 
A classe `android.content.Intent` representa uma ação que a aplicação deseja executar. A tradução de intent para o português é intenção, de forma que podemos dizer que uma intent representa a intenção da aplicação para realizar determinada tarefa. Na prática, essa intenção é enviada ao sistema operacional como uma mensagem, chamada de broadcast. Ao receber a mensagem, o sistema operacional tomará as decisões necessárias, dependendo do conteúdo da mensagem: 

Uma intent pode ser utilizada para: 

* enviar uma mensagem para o sistema operacional  
* abrir uma nova tela da aplicação, utilizando o método `startActivity(intent);`
* ligar para um número de celular 
* abrir o browser em uma página da internet 
* enviar uma mensagem por SMS ou email
* exibir algum endereço, localização ou rota no Google Maps
* abrir a agenda de contatos para selecionar um contato 
* abrir a galeria de fotos ou vídeos para selecionar um arquivo de multimídia
* tocar uma música ou vídeo 
* abrir a câmera e solicitar que uma foto ou vídeo sejam gravados
* enviar mensagens de uma tela para outra dentro do mesmo aplicativo, ou até trocar mensagens entre diferentes aplicações
* integrar diferentes aplicações
* abrir o Google Play para fazer a instalação de algum determinado aplicativo
* executar algum processamento pesado em segundo plano usando as classes [`BroadcastReceiver`](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/android-basics/broadcast-receiver.md) e `Service`
* e muito mais

**Nota:** uma intent é entre muitas coisas a forma utlizada para fazer com que aplicações em processos diferentes se comuniquem, utilizando uma mensagem, que dependendo do seu conteúdo, pode ser interceptada por qualquer aplicação. 

    
