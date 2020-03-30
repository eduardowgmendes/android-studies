## Guia para a arquitetura do app

Este guia engloba as práticas e a arquitetura recomendadas para a criação de apps robustos com qualidade de produção.

Esta página pressupõe familiaridade básica com o framework do Android. Se você não tem experiência com o desenvolvimento de apps Android, confira nossos Guias do desenvolvedor para dar os primeiros passos e saber mais sobre os conceitos mencionados neste guia.


### Experiências dos usuários de apps para dispositivos móveis
Na maioria dos casos, os apps para computador têm um único ponto de entrada em uma área de trabalho ou um acesso rápido de programas e, em seguida, são executados como um único processo monolítico. Por outro lado, os apps Android têm uma estrutura muito mais complexa. Em geral, um app Android contém vários componentes de app, inclusive atividades, fragmentos, serviços, provedores de conteúdo e broadcast receivers.

A maioria desses componentes de app é declarada no manifesto do app. O SO Android usa esse arquivo para decidir como integrar seu app à experiência geral do usuário do dispositivo. Considerando que um app Android corretamente programado contém vários componentes e que os usuários geralmente interagem com diversos apps em um curto período, os apps precisam se adaptar a diferentes tipos de fluxo de trabalho e tarefa controlados pelo usuário.

Por exemplo, pense no que acontece quando você compartilha uma foto no seu app de rede social favorito.

1. O app aciona um intent de câmera. Então, o SO Android abre um app de câmera para lidar com a solicitação. Nesse momento, o usuário saiu do app de rede social, mas a experiência dele ainda é constante.
2. O app de câmera pode acionar outros intents, como a abertura do seletor de arquivos, que pode iniciar mais um app.
3. Por fim, o usuário retorna ao app de rede social e compartilha a foto.

A qualquer momento durante o processo, o usuário pode ser interrompido por uma chamada telefônica ou notificação. Depois de lidar com essa interrupção, o usuário espera retomar o processo de compartilhamento de fotos. Esse comportamento de alternância de apps é comum em dispositivos móveis, mas seu app precisa lidar com esses fluxos corretamente.

Lembre-se de que os recursos de dispositivos móveis também são limitados. Por isso, o sistema operacional pode interromper os processos de alguns apps a qualquer momento para dar espaço a outros novos.

Considerando as condições desse ambiente, é possível que os componentes do seu app sejam iniciados individualmente e fora de ordem, e eles podem ser destruídos a qualquer momento pelo usuário ou pelo sistema operacional. Como esses eventos não estão sob seu controle, **não armazene nenhum dado ou estado de app nos componentes do seu app** e não permita que os componentes dele dependam uns dos outros.

### Princípios arquitetônicos comuns
Se não é recomendável usar componentes do app para armazenar dados e estados, qual é a melhor forma de criar seu app?

### Separação de conceitos
O princípio mais importante a seguir é a separação de conceitos (em inglês). É um erro comum escrever todo o código em uma Activity ou um Fragment. Essas classes baseadas em IU devem conter apenas a lógica que processa as interações entre a IU e o sistema operacional. Ao manter essas classes o mais enxutas possível, você pode evitar muitos problemas relacionados ao ciclo de vida.

Lembre-se de que você não possui implementações de Activity e Fragment. Na verdade, elas são apenas classes que representam o contrato entre o SO Android e seu app. O SO pode destruí-las a qualquer momento com base em interações do usuário ou condições do sistema, como pouca memória. Para oferecer uma experiência do usuário satisfatória e uma experiência de manutenção de app mais gerenciável, o melhor a se fazer é minimizar sua dependência delas.

### Basear a IU em um modelo
Outro princípio importante é que você precisa basear sua IU em um modelo, de preferência um que seja persistente. Modelos são componentes responsáveis por manipular os dados de um app. Como eles são independentes dos componentes de app e dos objetos View no seu app, não são afetados pelo ciclo de vida do app e pelas questões associadas.

A persistência é ideal pelos seguintes motivos:

* Seus usuários não perderão dados se o SO Android destruir seu app para liberar recursos.
* Seu app continuará funcionando se uma conexão de rede estiver lenta ou indisponível.

Baseando seu app em classes de modelo com responsabilidade bem definida de gerenciamento dos dados, ele se torna mais testável e consistente.

### Arquitetura de app recomendada
Nesta seção, demonstramos como estruturar um app usando os Componentes de arquitetura por meio de um caso de uso completo.

**Observação**: é impossível ter uma única maneira de criar apps que seja a melhor para todos os cenários.Dito isso, essa arquitetura recomendada é um bom ponto de partida para a maioria das situações e dos fluxos de trabalho. Se você já tem uma boa maneira de programar apps Android que segue os princípios de arquitetura comuns, não é necessário mudá-la.

Imagine que estamos criando uma IU que mostra um perfil de usuário. Usamos um back-end privado e uma API REST para buscar os dados de um determinado perfil.

### Visão geral
Comece analisando o diagrama a seguir, que mostra como todos os módulos precisam interagir uns com os outros depois da criação do app.

![]()

Observe que cada componente depende apenas daquele que está um nível abaixo de si. Por exemplo, atividades e fragmentos dependem apenas de um modelo de visualização. O repositório é a única classe que depende de várias outras classes. Nesse exemplo, o repositório depende de um modelo de dados persistente e de uma fonte de dados de back-end remota.

Esse design cria uma experiência do usuário consistente e agradável. Independentemente de o usuário voltar ao app alguns minutos ou vários dias depois de tê-lo fechado pela última vez, ele vê instantaneamente as informações do usuário que o app mantém localmente. Se esses dados estiverem desatualizados, o módulo de repositório do app começará a atualizá-los em segundo plano.

### Criar a interface do usuário
A IU consiste em um fragmento, UserProfileFragment, e no arquivo de layout correspondente, `user_profile_layout.xml`.

Para direcionar a IU, nosso modelo de dados precisa conter os seguintes elementos de dados:

* **ID do usuário**: o identificador do usuário. É melhor transmitir essa informação para o fragmento usando os argumentos dele. Se o SO Android destruir nosso processo, essas informações serão preservadas para que o ID esteja disponível na próxima vez que o app for reiniciado.
* **Objeto User**: uma classe de dados que contém detalhes sobre o usuário.

Usamos um `UserProfileViewModel` baseado no componente de arquitetura `ViewModel` para manter essas informações.

Um objeto `ViewModel` fornece os dados para um componente de IU específico, como um fragmento ou atividade, e contém lógica de negócios de manipulação de dados para se comunicar com o modelo. Por exemplo, o `ViewModel` pode chamar outros componentes para carregar os dados e pode encaminhar solicitações de usuários para modificar os dados. Como `ViewModel` não conhece os componentes de IU, ele não é afetado por alterações de configuração, como recriar uma atividade devido à rotação do dispositivo.

Definimos os seguintes arquivos:

* `user_profile.xml`: a definição do layout de IU para a tela.
* `UserProfileFragment`: o controlador de IU que exibe os dados.
* `UserProfileViewModel`: a classe que prepara os dados para exibição no `UserProfileFragment` e reage às interações do usuário.

Os snippets de código a seguir mostram o conteúdo inicial desses arquivos. Para simplificar o conteúdo, o arquivo de layout foi omitido.

### UserProfileViewModel

```kotlin
class UserProfileViewModel : ViewModel() {
	val userId : String = TODO()
	val user : User = TODO()
}
    
```
### UserProfileFragment

```kotlin
class UserProfileFragment : Fragment() {
       // To use the viewModels() extension function, include
       // "androidx.fragment:fragment-ktx:latest-version" in your app
       // module's build.gradle file.
       private val viewModel: UserProfileViewModel by viewModels()

       override fun onCreateView(
           inflater: LayoutInflater, container: ViewGroup?,
           savedInstanceState: Bundle?
       ): View {
           return inflater.inflate(R.layout.main_fragment, container, false)
       }
    }

```

Agora que temos esses módulos de código, como os conectamos? Afinal, quando o campo user é definido na classe UserProfileViewModel , precisamos de uma maneira de informar a IU.

Para ter o user, nosso ViewModel precisa acessar os argumentos de fragmento. É possível passá-los a partir do fragmento, mas a melhor opção é usar o módulo SavedState para fazer nosso ViewModel ler o argumento diretamente:

**Observação**: SavedStateHandle permite que o ViewModel acesse o estado e os argumentos salvos do Fragment ou da Activity associada.

```kotlin

// UserProfileViewModel
    class UserProfileViewModel(
       savedStateHandle: SavedStateHandle
    ) : ViewModel() {
       val userId : String = savedStateHandle["uid"] ?:
              throw IllegalArgumentException("missing user id")
       val user : User = TODO()
    }

    // UserProfileFragment
    private val viewModel: UserProfileViewModel by viewModels(
       factoryProducer = { SavedStateVMFactory(this) }
       ...
    )


```

Agora precisamos informar o fragmento quando tivermos o objeto do usuário. É aqui que entra em cena o componente de arquitetura LiveData.

LiveData é um armazenador de dados observáveis. Outros componentes no seu app podem monitorar se há alterações nos objetos usando esse armazenador sem criar caminhos de dependência rígidos e explícitos entre eles. O componente LiveData também respeita o estado do ciclo de vida dos componentes do seu app (como atividades, fragmentos e serviços) e inclui lógica de limpeza para evitar o vazamento de objetos e o consumo excessivo de memória.

**Observação**: se você já está usando uma biblioteca como RxJava, pode continuar usando-a em vez do `LiveData`. No entanto, quando você usa bibliotecas e abordagens como essas, lembre-se de lidar corretamente com o ciclo de vida do seu app. Em especial, os streams de dados precisam ser pausados quando o `LifecycleOwner` relacionado é interrompido e destruídos quando o `LifecycleOwner` relacionado é destruído. Você também pode adicionar o artefato `android.arch.lifecycle:reactivestreams` para usar o `LiveData` com outra biblioteca de streams reativos, como o RxJava2.

Para incorporar o componente do `LiveData` ao nosso app, alteramos o tipo de campo no UserProfileViewModel para `LiveData<User>`. Agora, o `UserProfileFragment` é informado quando os dados são atualizados. Além disso, como o campo `LiveData` reconhece o ciclo de vida, ele limpa automaticamente as referências quando elas não são mais necessárias.

### UserProfileViewModel
```kotlin
class UserProfileViewModel(
       savedStateHandle: SavedStateHandle
    ) : ViewModel() {
       val userId : String = savedStateHandle["uid"] ?:
              throw IllegalArgumentException("missing user id")
       val user : LiveData<User> = TODO()
    }

```
Agora modificamos `UserProfileFragment` para observar os dados e atualizar a IU:

### UserProfileFragment
```kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
       super.onViewCreated(view, savedInstanceState)
       viewModel.user.observe(viewLifecycleOwner) {
           // update UI
       }
    }

```

Toda vez que os dados de perfil do usuário são atualizados, o callback `onChanged()` é invocado, e a IU é atualizada.

Se você tem familiaridade com outras bibliotecas que usam callbacks observáveis, talvez tenha percebido que não foi necessário modificar o método onStop() do fragmento para interromper a observação dos dados. Essa etapa não é necessária com o LiveData, porque ele reconhece o ciclo de vida, o que significa que ele não invoca o callback onChanged(), a menos que o fragmento esteja em um estado ativo, isto é, tenha recebido onStart(), mas ainda não tenha recebido onStop(). O LiveData também remove automaticamente o observador quando o método onDestroy() do fragmento é chamado.

Também não adicionamos nenhuma lógica para lidar com alterações de configuração, por exemplo, quando o usuário gira a tela do dispositivo. O UserProfileViewModel é restaurado automaticamente quando a configuração muda. Dessa forma, assim que o novo fragmento é criado, ele recebe a mesma instância de ViewModel, e o callback é invocado imediatamente por meio dos dados atuais. Considerando que se espera que os objetos ViewModel durem mais do que os objetos View correspondentes que eles atualizam, não inclua referências diretas a objetos View na sua implementação de ViewModel. Para ver mais informações sobre a vida útil de um ViewModel correspondente ao ciclo de vida dos componentes de IU, consulte O ciclo de vida de um ViewModel.

### Buscar dados
Agora que usamos o LiveData para conectar o UserProfileViewModel ao UserProfileFragment, como podemos buscar os dados do perfil do usuário?

Neste exemplo, presumimos que nosso back-end ofereça uma API REST. Usaremos a biblioteca Retrofit (em inglês) para acessar nosso back-end, mas você pode usar uma biblioteca diferente que cumpra a mesma finalidade.

Esta é nossa definição de Webservice que se comunica com nosso back-end:

### Webservice
```kotlin
interface Webservice {
       /**
        * @GET declares an HTTP GET request
        * @Path("user") annotation on the userId parameter marks it as a
        * replacement for the {user} placeholder in the @GET path
        */
       @GET("/users/{user}")
       fun getUser(@Path("user") userId: String): Call<User>
    }

```

Uma primeira ideia para implementar o ViewModel pode envolver chamar diretamente o Webservice para buscar os dados e atribuí-los de volta ao nosso objeto LiveData. Esse design funciona, mas, ao usá-lo, fica cada vez mais difícil manter nosso app à medida que ele cresce. Ele atribui muita responsabilidade à classe UserProfileViewModel, o que viola o princípio de separação de conceitos. Além disso, o escopo de um ViewModel está vinculado ao ciclo de vida de uma Activity ou de um Fragment, o que significa que os dados do Webservice são perdidos quando o ciclo de vida do objeto de IU associado termina. Esse comportamento cria uma experiência de usuário indesejável.

Em vez disso, nosso ViewModel delega o processo de busca de dados a um novo módulo, um repositório.

Módulos de repositório manipulam operações de dados. Eles disponibilizam uma API limpa para que o restante do app possa recuperar esses dados com facilidade. Eles sabem onde coletar os dados e quais chamadas de API precisam ser feitas quando os dados são atualizados. Você pode considerar os repositórios como mediadores entre fontes de dados diferentes, por exemplo, modelos persistentes, serviços da Web e caches.

Nossa classe UserRepository, mostrada no snippet de código a seguir, usa uma instância do WebService para buscar dados de um usuário:

### UserRepository
```kotlin
class UserRepository {
       private val webservice: Webservice = TODO()
       // ...
       fun getUser(userId: String): LiveData<User> {
           // This isn't an optimal implementation. We'll fix it later.
           val data = MutableLiveData<User>()
           webservice.getUser(userId).enqueue(object : Callback<User> {
               override fun onResponse(call: Call<User>, response: Response<User>) {
                   data.value = response.body()
               }
               // Error case is left out for brevity.
               override fun onFailure(call: Call<User>, t: Throwable) {
                   TODO()
               }
           })
           return data
       }
    }

```

Embora o módulo de repositório pareça desnecessário, ele serve a um propósito importante: abstrair as fontes de dados do restante do app. Agora, nosso UserProfileViewModel não sabe como os dados são buscados. Sendo assim, podemos fornecer ao modelo de visualização dados coletados de diferentes implementações de busca de dados.

**Observação**: para simplificar o conteúdo, deixamos de fora os casos de erros de rede. Para ver uma implementação alternativa que exponha erros e status de carregamento, consulte Adendo: como exibir o status da rede.

### Gerenciar dependências entre componentes
A classe UserRepository acima precisa de uma instância do Webservice para buscar os dados do usuário. Ela poderia simplesmente criar a instância, mas, para isso, também precisaria conhecer as dependências da classe Webservice. Além disso, UserRepository provavelmente não é a única classe que precisa de um Webservice. Essa situação exige a duplicação do código, porque cada classe que requer uma referência a Webservice precisa saber como construí-lo, bem como as dependências dele. Se cada classe cria um novo WebService, nosso app pode ter uma carga de recursos muito pesada.

Você pode usar os seguintes padrões de design para resolver esse problema:


* Injeção de dependência (DI, na sigla em inglês): permite que as classes definam as próprias dependências sem construí-las. No tempo de execução, outra classe será responsável por disponibilizar essas dependências. Recomendamos a biblioteca Dagger 2 (em inglês) para implementar a injeção de dependência em apps Android. A biblioteca Dagger 2 constrói os objetos automaticamente percorrendo a árvore de dependências, além de oferecer garantias de tempo de compilação nas dependências.
* Localizador de serviço (em inglês): esse padrão traz um registro de onde as classes podem buscar as próprias dependências em vez de construí-las.

É mais fácil implementar um registro de serviço do que usar a DI. Assim, se você não tem familiaridade com a DI, use o padrão de localização de serviço.

Esses padrões permitem dimensionar seu código, porque fornecem padrões claros para gerenciar dependências sem duplicar o código ou adicionar complexidade. Além disso, esses padrões permitem alternar rapidamente entre implementações de busca de dados de teste e de produção.

Nosso app de exemplo usa a Dagger 2 para gerenciar as dependências do objeto Webservice.

### Conectar o ViewModel e o repositório
Agora, modificamos nosso UserProfileViewModel para usar o objeto UserRepository:

### UserProfileViewModel
```kotlin
class UserProfileViewModel @Inject constructor(
       savedStateHandle: SavedStateHandle,
       userRepository: UserRepository
    ) : ViewModel() {
       val userId : String = savedStateHandle["uid"] ?:
              throw IllegalArgumentException("missing user id")
       val user : LiveData<User> = userRepository.getUser(userId)
    }

```

### Dados de cache

A implementação de UserRepository abstrai a chamada para o objeto Webservice, mas por depender de apenas uma fonte de dados, ela não é muito flexível.

O principal problema com a implementação de UserRepository é que, depois de buscar dados no nosso back-end, ele não armazena esses dados em lugar nenhum. Então, se o usuário deixar o UserProfileFragment e depois retornar a ele, nosso app precisará buscar novamente os dados, mesmo que eles não tenham sido alterados.

Esse design não é o ideal pelas seguintes razões:

* Ele desperdiça largura de banda.
* Ele força o usuário a aguardar a conclusão da nova consulta.

Para resolver essas deficiências, adicionamos uma nova fonte de dados ao nosso UserRepository, que armazena os objetos User na memória:

### UserRepository
```kotlin
// Informs Dagger that this class should be constructed only once.
    @Singleton
    class UserRepository @Inject constructor(
       private val webservice: Webservice,
       // Simple in-memory cache. Details omitted for brevity.
       private val userCache: UserCache
    ) {
       fun getUser(userId: String): LiveData<User> {
           val cached = userCache.get(userId)
           if (cached != null) {
               return cached
           }
           val data = MutableLiveData<User>()
           userCache.put(userId, data)
           // This implementation is still suboptimal but better than before.
           // A complete implementation also handles error cases.
           webservice.getUser(userId).enqueue(object : Callback<User> {
               override fun onResponse(call: Call<User>, response: Response<User>) {
                   data.value = response.body()
               }

               // Error case is left out for brevity.
               override fun onFailure(call: Call<User>, t: Throwable) {
                   TODO()
               }
           })
           return data
       }
    }
    
```

### Persistir nos dados

Usando nossa implementação atual, se o usuário girar a tela do dispositivo ou sair e retornar ao app imediatamente, a IU existente ficará visível no mesmo momento, porque o repositório recupera os dados do cache na memória.

No entanto, o que acontece se o usuário sair do app e voltar horas depois, quando o SO Android já tiver interrompido o processo? Usando nossa implementação atual nessa situação, precisamos buscar os dados novamente na rede. Esse processo de nova busca não é apenas uma má experiência do usuário, mas também um desperdício, porque consome dados móveis valiosos.

Você poderia corrigir esse problema armazenando as solicitações da Web em cache, mas isso criaria um novo problema: o que acontecerá se os mesmos dados do usuário forem exibidos a partir de outro tipo de solicitação, por exemplo, da busca de uma lista de amigos? O app mostraria dados inconsistentes, o que seria no mínimo confuso. Por exemplo, se o usuário criasse a solicitação de lista de amigos e a solicitação de usuário único em momentos diferentes, nosso app poderia mostrar duas versões diferentes dos dados do mesmo usuário. Nosso app precisaria descobrir como mesclar esses dados inconsistentes.

A maneira correta de lidar com isso é usar um modelo persistente. É aqui que a biblioteca de persistência Room vem para ajudar.

A Room é uma biblioteca de mapeamento de objetos que fornece persistência de dados locais com um uso mínimo de código clichê. No tempo de compilação, ela valida cada consulta em relação ao seu esquema de dados, de modo que consultas SQL interrompidas resultam em erros de tempo de compilação, em vez de falhas de tempo de execução. A Room abstrai alguns dos detalhes subjacentes da implementação do trabalho com tabelas e consultas SQL brutas. Ela também permite observar alterações nos dados do banco de dados (incluindo coleções e consultas de mesclagem), expondo essas alterações por meio de objetos LiveData. Essa biblioteca define explicitamente até mesmo as restrições de execução que abordam problemas comuns de encadeamento, como o acesso ao armazenamento na linha de execução principal.

**Observação**: se o app já usa outra solução de persistência, como um mapeamento relacional de objeto (ORM, na sigla em inglês) do SQLite, não é necessário substituir sua solução existente pela Room.No entanto, se você estiver criando um novo app ou refatorando outro existente, recomendamos usar a Room para persistir nos dados do seu app. Dessa forma, você poderá aproveitar os recursos de abstração e validação de consultas da biblioteca.

Para usar a Room, precisamos definir nosso esquema local. Primeiro, adicionamos a anotação @Entity à nossa classe de modelo de dados User e uma anotação @PrimaryKey ao campo id da classe. Essas anotações marcam User como uma tabela no nosso banco de dados e id como a chave primária da tabela:

### Usuário
```kotlin
@Entity
    data class User(
       @PrimaryKey private val id: String,
       private val name: String,
       private val lastName: String
    )

```
Em seguida, criamos uma classe de banco de dados implementando RoomDatabase para nosso app:
### UserDatabase
```kotlin
@Database(entities = [User::class], version = 1)
    abstract class UserDatabase : RoomDatabase()

```

Observe que UserDatabase é abstrato. A Room o implementa automaticamente. Para mais detalhes, consulte a documentação da Room.

Agora precisamos de uma maneira de inserir os dados do usuário no banco de dados. Para essa tarefa, criamos um objeto de acesso a dados (DAO, na sigla em inglês).

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

Para saber mais sobre filtros de intents, consulte o documento [Intents e filtros de intents](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/06-intents-e-intent-filters.md#intents-e-filtros-de-intents). 
 


