## Broadcast Receivers 
Conheça uma das classes mais importantes da arquitetura do Android. Ela é utilizada para que aplicações possam interceptar eventos do sistema, ou seja, mensagens geradas por uma intent. 

Na prática, o broadcast receiver é uma classe que consegue interceptar uma intent, sendo uma intent uma mensagem que pode ser enviada pela sua própria aplicação, por uma aplicação de terceiros ou pelo próprio sistema operacional. 

Um broadcast receiver sempre executa em segundo plano (background) e não utiliza interface gráfica. Seu objetivo é interceptar uma mensagem (intent) e processá-la sem que o usuário perceba. 

## Introdução 
A classe `android.content.BroadcastReceiver` é utilizada para interceptar mensagens enviadas por uma intent. Quando a mensagem é interceptada, o método `onReceive(context, intent)` é chamado, e executa em segundo plano, sem mostrar nenhuma tela ao usuário. 

Um broadcast receiver pode ser utilizado para enviar e receber mensagens dentro da própria aplicação ou para interceptar eventos do sistema. Um exemplo de evento do sistema é a mensagem SMS_RECEIVED, a qual é enviada pelo Android sempre que uma nova mensagem SMS é recebida pelo sistema operacional. Portanto, caso seja interessante para sua aplicação, é possível interceptar essas mensagens, ou seja, esses eventos, a fim de executar alguma aplicação. 

Um broadcast receiver consegue interceptar uma intent/mensagem mesmo se aplicação estiver parada, e esse é um de seus grandes benefícios.

## Configurando um receiver de forma estática
Um receiver pode ser configurado de forma estática no arquivo `AndroidManifest.xml` ou de forma dinâmica no código: 

De forma estática, no arquivo `AndroidManifest.xml` com a tag `<receiver>`. Isso é útil quando precisamos interceptar a mensagem mesmo se a aplicação estiver fechada. É o caso de interceptar uma mensagem SMS ou Push, pois geralmente esse tipo de mensagem precisa ser interceptada mesmo com a aplicação fechada. 

```xml 
<manifest ...>
	<application ...>
		<activity android:name=".MainActivity" .../>
		<receiver android:name=".HelloReceiver">
			<intent-filter>
				<action android:name="BINGO"/>
				<category android:name="android.intent.category.DEFAULT"/>
			</intent-filter>
		</receiver>
	</application>
</manifest>
```
Registrar o receiver dinamicamente na activity. Nesse caso, o receiver fica atrelado ao ciclo de vida da activity, ou seja, o receiver ficará ativo somente enquanto a activity estiver aberta. Esse recurso é muito utilizado para enviar mensagens internas da aplicação. Por exemplo, um receiver pode interceptar um evento de que o estado da conexão com a internet mudou, e enviar uma mensagem local para avisar a activity se a aplicação está online ou offline.   


  
