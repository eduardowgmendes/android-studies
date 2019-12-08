## Guia para a arquitetura do app

Este guia engloba as práticas e a arquitetura recomendadas para a criação de apps robustos com qualidade de produção.

Esta página pressupõe familiaridade básica com o framework do Android. Se você não tem experiência com o desenvolvimento de apps Android, confira nossos Guias do desenvolvedor para dar os primeiros passos e saber mais sobre os conceitos mencionados neste guia.


### Experiências dos usuários de apps para dispositivos móveis
Na maioria dos casos, os apps para computador têm um único ponto de entrada em uma área de trabalho ou um acesso rápido de programas e, em seguida, são executados como um único processo monolítico. Por outro lado, os apps Android têm uma estrutura muito mais complexa. Em geral, um app Android contém vários componentes de app, inclusive atividades, fragmentos, serviços, provedores de conteúdo e broadcast receivers.

### Componentes de aplicativo
Componentes de aplicativo são os blocos de construção de um app Android. Cada componente é um ponto de entrada por onde o sistema ou o usuário podem entrar no aplicativo. Alguns componentes dependem de outros.

Há quatro tipos diferentes de componentes de aplicativo:

* Atividades
* Serviços
* Broadcast receivers
* Provedores de conteúdo

Cada tipo tem uma finalidade distinta e tem um ciclo de vida específico que define a forma como o componente é criado e destruído. As seções a seguir descrevem os quatro tipos de componentes do aplicativo.

### Atividades

Uma atividade é o ponto de entrada para a interação com o usuário. Ela representa uma tela única com uma interface do usuário. Por exemplo, um app de e-mails pode ter uma atividade que mostra uma lista de novos e-mails, outra atividade que compõe um e-mail e outra ainda que lê e-mails. Embora essas atividades funcionem juntas para formar uma experiência do usuário coesa no app de e-mails, elas são independentes entre si. Portanto, um aplicativo diferente pode iniciar qualquer uma dessas atividades (se o app de e-mails permitir). Por exemplo, um aplicativo de câmera pode iniciar a atividade no app de e-mails que compõe o novo e-mail para que o usuário compartilhe uma foto. Uma atividade facilita as principais interações a seguir entre sistema e aplicativo: 

* Acompanhamento do que interessa ao usuário atualmente (o que está na tela) para garantir que o sistema permaneça executando processos que hospedam a atividade.
* Conhecimento dos processos usados anteriormente que contêm coisas a que o usuário pode retornar (atividades interrompidas) e, portanto, priorização da manutenção desses processos.
* Auxílio ao aplicativo quanto aos processos interrompidos para que o usuário possa retornar às atividades com o estado anterior restaurado.
* Oferecimento de uma maneira de os aplicativos implementarem os fluxos de usuários entre si e o sistema coordenar esses fluxos. Aí vai o exemplo mais clássico de todos.

Você implementa uma atividade como uma subclasse da classe Activity. Para mais informações sobre a classe Activity, consulte o guia do desenvolvedor Atividades.

### Serviços

O serviço é um ponto de entrada para manter um aplicativo em execução no segundo plano, seja qual for o motivo. É um componente executado em segundo plano para realizar operações de execução longa ou trabalho para processos remotos. Serviços não apresentam uma interface do usuário. Por exemplo, um serviço pode tocar música em segundo plano enquanto o usuário está em um aplicativo diferente ou buscar dados na rede sem bloquear a interação do usuário com uma atividade. Outro componente, como uma atividade, pode iniciar o serviço e deixá-lo executar ou vincular-se a ele para interagir. Na verdade, há dois serviços semânticos muito diferentes que dizem ao sistema como gerenciar um aplicativo: serviços iniciados dizem ao sistema que os mantenha em execução até o trabalho deles ser concluído. Isso pode ser a sincronização de dados em segundo plano ou a reprodução de músicas, mesmo após o usuário sair do aplicativo. A sincronização de dados em segundo plano ou a reprodução de músicas também representam dois tipos diferentes de serviços iniciados que modificam a forma como o sistema lida com eles: 


* A reprodução de música é algo diretamente perceptível ao usuário, então o aplicativo informa isso ao sistema, dizendo que precisa ficar em primeiro plano, com uma notificação que avisa ao usuário sobre essa situação. Nesse caso, o sistema entende que é necessário tentar ao máximo manter o processo do serviço em execução, porque, se ele desaparecer, o usuário será prejudicado.
* Um serviço de segundo plano comum não é algo que o usuário tenha consciência direta durante a execução. Dessa forma, o sistema tem maior liberdade para gerenciar os processos. Pode ser permitida a interrupção (e o reinício do processo em algum momento futuro), caso seja necessário RAM para coisas imediatamente mais importantes ao usuário.

Os serviços vinculados são executados porque algum outro aplicativo (ou o sistema) informa que precisa usá-los. É basicamente o serviço fornecendo uma API para outro processo. O sistema então percebe que há uma dependência entre esses processos. Assim, se o processo A for vinculado a um serviço no processo B, ele sabe que precisa manter o B (e o serviço referente a ele) em execução para o A. Além disso, se o processo A é algo importante para o usuário, ele sabe tratar o processo B como algo também importante. Por terem flexibilidade (para o bem e para o mal), os serviços tornaram-se um elemento básico muito útil para todos os tipos de conceitos de sistema de nível mais alto. Planos de fundo interativos, detectores de notificação, protetores de tela, métodos de entrada, serviços de acessibilidade e muitos outros recursos fundamentais do sistema são criados como serviços que os aplicativos implementam e a que o sistema se vincula quando há a necessidade de execução. 

Os serviços são implementados como uma subclasse de Service. Para mais informações sobre a classe Service, consulte o guia do desenvolvedor [Serviços]().

### Broadcast receivers

O broadcast receiver é um componente que faz o sistema entregar eventos ao aplicativo fora de fluxo de usuários comum. Isso permite que o aplicativo responda a anúncios de transmissão por todo o sistema. Como os broadcast receivers são mais uma entrada bem definida no aplicativo, o sistema consegue entregar transmissões até a aplicativos que não estejam em execução no momento. Por exemplo, um aplicativo pode programar um alarme para postar uma notificação que avise o usuário sobre um evento futuro. Ao entregar esse alarme a um receptor de transmissão, o aplicativo não precisa permanecer em execução até o alarme ser desativado. Muitas transmissões têm origem no sistema — por exemplo, uma transmissão que informa o desligamento da tela, bateria com pouca carga ou captura de imagem. Aplicativos também podem iniciar transmissões — por exemplo, para permitir que outros aplicativos saibam que alguns dados foram salvos no dispositivo e que estão prontos para o uso. Embora os broadcast receivers não exibam nenhuma interface do usuário, eles podem criar uma notificação na barra de status para alertar ao usuário quando ocorre um evento de transmissão. Mais comumente, no entanto, um broadcast receiver é somente um gateway para outros componentes e realiza uma quantidade mínima de trabalho. Por exemplo, ele pode programar um JobService para executar um trabalho com base no evento com JobScheduler. 

Os broadcast receivers são implementados como subclasses de BroadcastReceiver e cada transmissão é entregue como um objeto Intent. Para mais informações, consulte a classe BroadcastReceiver.

### Provedores de conteúdo

Provedores de conteúdo gerenciam um conjunto compartilhado de dados do aplicativo que você pode armazenar nos sistemas de arquivos, em banco de dados SQLite, na Web ou em qualquer local de armazenamento persistente acessível ao seu aplicativo. Por meio do provedor de conteúdo, outros aplicativos podem consultar ou até modificar os dados, caso o provedor de conteúdo permita. Por exemplo, o sistema Android oferece um provedor de conteúdo que gerencia os dados de contato do usuário. Assim qualquer aplicativo com as permissões adequadas pode consultar parte do provedor de conteúdo (como ContactsContract.Data) para ler e gravar informações sobre uma pessoa específica. É tentador pensar nos provedores de conteúdo como uma abstração em um banco de dados, porque há bastante APIs e suporte integrado a eles para esse caso comum. No entanto, de uma perspectiva de desenvolvimento de sistemas, eles têm um propósito principal diferente. Para o sistema, o provedor de conteúdo é um ponto de entrada em um aplicativo para a publicação de itens de dados nomeados, identificados por um esquema URI. Assim um aplicativo pode decidir como quer mapear os dados que contém para um namespace URI. Isso transfere esses URIs a outras entidades, que podem então usá-los para acessar os dados. Há algumas coisas em especial que isso permite ao sistema, com relação ao gerenciamento de um aplicativo: 

* A atribuição de um URI não exige que o aplicativo permaneça em execução. Dessa forma, os URIs podem persistir após os próprios aplicativos deles terem sido encerrados. Quando é necessário recuperar os dados do aplicativo no URI correspondente, o sistema só precisa garantir que um aplicativo pertencente ainda esteja em execução.
* Os URIs também fornecem um importante modelo de segurança de controle preciso. Por exemplo, um aplicativo pode substituir o URI por uma imagem que esteja na área de transferência, mas deixar o provedor de conteúdo bloqueado para que outros aplicativos não possam acessá-lo livremente. Quando um segundo aplicativo tenta acessar o URI na área de transferência, o sistema pode permitir o acesso por meio de uma concessão temporária da permissão do URI. Assim é permitido o acesso aos dados somente por trás desse URI, mas nada além disso no segundo aplicativo.

Os provedores de conteúdo são úteis para ler e gravar dados privados no aplicativo e não compartilhados.

Um provedor de conteúdo é implementado como uma subclasse de ContentProvider e precisa implementar um conjunto padrão de APIs que permitem a outros aplicativos realizar transações. Para mais informações, consulte o guia do desenvolvedor [Provedores de conteúdo]().

Um aspecto exclusivo do projeto do sistema Android é que qualquer aplicativo pode iniciar um componente de outro aplicativo. Por exemplo, se você quiser que o usuário capture uma foto com a câmera do dispositivo, provavelmente haverá outro aplicativo que faz isso e seu aplicativo poderá usá-lo, ou seja, não será necessário desenvolver uma atividade para capturar uma foto. Não é necessário incorporar nem mesmo vincular ao código do aplicativo de câmera. Em vez disso, você pode simplesmente iniciar a atividade no aplicativo de câmera que captura uma foto. Quando concluída, a foto até retorna ao aplicativo em questão para ser usada. Para o usuário, parece que a câmera é realmente parte do aplicativo.

Quando o sistema inicia um componente, ele inicia o processo daquele aplicativo (se ele já não estiver em execução) e instancia as classes necessárias para o componente. Por exemplo: se o aplicativo iniciar a atividade no aplicativo da câmera que captura uma foto, aquele aplicativo é executado no processo que pertence ao aplicativo da câmera, e não no processo do aplicativo. Portanto, ao contrário dos aplicativos na maioria dos outros sistemas, os apps Android não têm nenhum ponto de entrada único (não há a função main(), por exemplo).

Como o sistema executa cada aplicativo em um processo separado com permissões de arquivo que restringem o acesso a outros aplicativos, o aplicativo não pode ativar diretamente um componente de outro aplicativo. Mas o sistema do Android consegue fazer isso. Portanto, para ativar um componente em outro aplicativo, é preciso enviar uma mensagem ao sistema que especifique o intent a iniciar um componente específico. Em seguida, o sistema ativa o componente.

## Ativação de Componentes

Três dos quatro tipos de componentes — atividades, serviços e broadcast receivers — são ativados por uma mensagem assíncrona chamada intent. Os intents vinculam componentes individuais uns aos outros no ambiente de execução. Pense neles como mensageiros que solicitam uma ação de outros componentes, seja o componente pertencente ao aplicativo ou não.

O intent é criado com um objeto Intent que define uma mensagem para ativar um componente específico ou um tipo específico de componente. Os intents podem ser explícitos ou implícitos, respectivamente.

Para atividades e serviços, os intents definem a ação a executar (por exemplo, visualizar ou enviar algo) e podem especificar o URI dos dados usados na ação, entre outras coisas que o componente a iniciar precisa saber. Por exemplo, um intent pode transmitir uma solicitação de uma atividade para exibir uma imagem ou abrir uma página da Web. Em alguns casos, é preciso iniciar uma atividade para receber um resultado. Nesse caso, a atividade também retorna o resultado em um Intent. Por exemplo, é possível emitir um intent para que o usuário selecione um contato pessoal e retorne-o a você. O intent de retorno contém um URI que aponta para o contato selecionado.

Para broadcast receivers, o intent simplesmente define o anúncio que está sendo transmitido. Por exemplo, uma transmissão para indicar que a bateria do dispositivo está acabando contém uma string de ação conhecida que indica nível baixo da bateria.

Diferentemente das atividades, dos serviços e dos broadcast receivers, os provedores de conteúdo não são ativados por intents. Em vez disso, eles são ativados quando direcionados pela solicitação de um ContentResolver. O resolvedor de conteúdo trata todas as transações diretas com o provedor de conteúdo para que o componente que executa as transações com o provedor não precise e, em vez disso, chame os métodos no objeto ContentResolver. Isso deixa uma camada de abstração entre o provedor de conteúdo e o componente que solicita informações (por segurança).

Há dois métodos para ativar cada tipo de componente:

* É possível iniciar uma atividade (ou dar-lhe algo novo para fazer) passando um Intent a `startActivity()` ou `startActivityForResult()` (para que, quando desejado, a atividade retorne um resultado).
* Com o Android 5.0 (API nível 21) e posteriores, você pode usar a classe JobScheduler para programar ações. Para versões anteriores do Android, é possível iniciar um serviço (ou dar novas instruções a um serviço em andamento) passando um Intent a startService(). Também é possível vincular ao serviço passando um Intent a bindService().
* Você pode iniciar uma transmissão passando um Intent para métodos como `sendBroadcast()`, `sendOrderedBroadcast()` ou `sendStickyBroadcast()`.
* É possível iniciar uma consulta a um provedor de conteúdo chamando `query()` em um `ContentResolver`.

Para saber mais sobre intents, consulte o documento Intents e filtros de intents. Os documentos a seguir fornecem mais informações sobre a ativação de componentes específicos: Atividades, Serviços, BroadcastReceiver e Provedores de Conteúdo.

## O Arquivo de Manifesto
Antes de o sistema Android iniciar um componente de aplicativo, é preciso ler o arquivo de manifesto AndroidManifest.xml do aplicativo para que o sistema saiba que o componente existe. O aplicativo precisa declarar todos os seus componentes nesse arquivo, que precisa estar na raiz do diretório do projeto do aplicativo.

O manifesto faz outras coisas além de declarar os componentes do aplicativo, por exemplo:

* Identifica todas as permissões do usuário de que o aplicativo precisa, como acesso à Internet ou acesso somente leitura aos contatos do usuário.
* Declara o nível de API mínimo exigido pelo aplicativo com base nas APIs que o aplicativo usa.
* Declara os recursos de hardware e software usados ou exigidos pelo aplicativo, como câmera, serviços de Bluetooth ou tela multitoque.
* Declara as bibliotecas API de que o aplicativo precisa para ser vinculado (além das APIs de biblioteca do Android), como a Biblioteca do Google Maps.


### Declaração de componentes
A principal tarefa do manifesto é informar ao sistema os componentes do aplicativo. Por exemplo, um arquivo de manifesto pode declarar uma atividade da seguinte forma: 
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest ... >
    <application android:icon="@drawable/app_icon.png" ... >
        <activity android:name="com.example.project.ExampleActivity"
                  android:label="@string/example_label" ... >
        </activity>
        ...
    </application>
</manifest>
```

No elemento `<application>`, o atributo android:icon aponta para recursos de um ícone que identifica o aplicativo.

No elemento `<activity>`, o atributo android:name especifica o nome da classe totalmente qualificada da subclasse de Activity e o atributo android:label especifica uma string a usar como rótulo da atividade visível ao usuário.

É necessário declarar todos os componentes do aplicativo que usam os elementos a seguir:

* Elementos `<activity>` para atividades
* Elementos `<service>` para serviços
* Elementos `<receiver>` para broadcast receivers
* Elementos `<provider>` para provedores de conteúdo

Atividades, serviços e provedores de conteúdo incluídos na fonte, mas não declarados no manifesto, não ficam visíveis para o sistema e, consequentemente, podem não ser executados. No entanto, broadcast receivers podem ser declarados no manifesto dinamicamente no código (como objetos `BroadcastReceiver`) e registrados no sistema chamando-se `registerReceiver()`.

Para saber mais sobre a estrutura do arquivo de manifesto de aplicativos, consulte a documentação O arquivo *AndroidManifest.xml*. 

### Declaração de recursos de componentes
Conforme abordado acima, em Ativação de componentes, é possível usar um Intent para iniciar atividades, serviços e broadcast receivers. Você pode usar um Intent nomeando explicitamente o componente-alvo (usando o nome da classe do componente) no intent. Também é possível usar um intent implícito. Ele descreve o tipo de ação a ser realizada e fornece a opção de informar os dados sobre os quais você quer realizá-la. O intent implícito permite que o sistema encontre um componente no dispositivo que possa realizar a ação e iniciá-la. Se houver mais de um componente que possa executar a ação descrita pelo intent, o usuário selecionará qual deles usar.

**Atenção**: se você usar um intent para iniciar um Service, use um intent explícito para verificar se seu aplicativo é seguro. O uso de um intent implícito para iniciar um serviço representa um risco de segurança, porque não é possível determinar qual serviço responderá ao intent, e o usuário não poderá ver que serviço é iniciado. A partir do Android 5.0 (API de nível 21), o sistema lança uma exceção ao chamar bindService() com um intent implícito. Não declare filtros de intents para seus serviços. 

Para o sistema identificar os componentes que podem responder a um intent, compara-se o intent recebido com os filtros de intent fornecidos no arquivo de manifesto de outros aplicativos no dispositivo.

Ao declarar uma atividade no manifesto do aplicativo, é possível incluir filtros de intents que declarem os recursos da atividade para que ela responda a intents de outros aplicativos. Para declarar um filtro de intents no componente, adiciona-se um elemento <intent-filter> como filho do elemento de declaração do componente.

Por exemplo, se você estiver criando um app de e-mails com uma atividade de compor um novo e-mail, poderá declarar um filtro de intents para responder a intents “enviar” (para enviar um novo e-mail), assim:

```xml
<manifest ... >
    ...
    <application ... >
        <activity android:name="com.example.project.ComposeEmailActivity">
            <intent-filter>
                <action android:name="android.intent.action.SEND" />
                <data android:type="*/*" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

Em seguida, se outro aplicativo criar um intent com a ação ACTION_SEND e passá-la para startActivity(), o sistema poderá iniciar a atividade de forma que o usuário possa rascunhar e enviar um e-mail.

Para saber mais sobre filtros de intents, consulte o documento Intents e filtros de intents. 
 


