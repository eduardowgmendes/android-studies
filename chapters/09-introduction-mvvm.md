## Introdução 
O propósito dos [*Architecture Components*](https://developer.android.com/topic/libraries/architecture/) do Android é prover, com bibliotecas para tarefas comuns como gerenciamento do ciclo de vida e persistência de dados. A biblioteca *Architecture Components* são parte da [Android JetPack](https://developer.android.com/jetpack).

Os Archtecture Components o ajudarão a estruturar seu app de forma robusta, testável e com muito menos código clichê (*boilerplate code*).

### O que são os componentes de arquitetura recomendados?
Aqui vai uma breve introdução do que são e como eles funcionam juntos. Este breve tutorial foca apenas nos subconjuntos de componentes, chamados [`LiveData`](https://developer.android.com/topic/libraries/architecture/livedata), [`ViewModel`](https://developer.android.com/topic/libraries/architecture/viewmodel) e [`Room`](https://developer.android.com/topic/libraries/architecture/room). Cada componente é explicado em detalhes assim que o utilizarmos. 

O diagrama abaixo demonstra a forma básica dessa arquitetura:

![Forma Básica da Arquitetura](https://raw.githubusercontent.com/eduardowgmendes/android-studies/master/images/arch-components-basic.png)

* [Entity](https://developer.android.com/reference/androidx/room/Entity): Classe anotada que consiste em uma entidade de tabela quando trabalhamos com [Room](https://developer.android.com/topic/libraries/architecture/room)

* SQLite Database: Banco de Dados Nativo do Android. A biblioteca de persistência de dados Room cria e gerencia esse banco de dados para você. Usando Room não é necessário criar as tabelas na mão usando a classe `SQLiteOpenHelper`.

* [DAO](https://developer.android.com/reference/androidx/room/Dao.html): *Data Access Object*. Trata-se de um padrão de projeto que separa a lógica de negócios da lógica de persistência de dados. Basicamente aqui o DAO mapeia as *queries* SQL para métodos. Quando se usa um DAO em conjunto com o Room, você invoca métodos, e o Room toma conta do resto para você.

* [Room Database](https://developer.android.com/topic/libraries/architecture/room): Simplifica o trabalho com persistência de dados e serve como um ponto de acesso ao banco de dados SQL da aplicação. O Room usa o DAO para centralizar todas as *queries* ao banco de dados.

* [Repository](): Gerencia multiplas fontes de dados (*data sources*). 

* [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel): Atua como um intermediador entre o Repository (dados) e a IU (views). A IU não precisa se preocupar mais com a origem dos dados. A classe ViewModel foi projetada para armazenar e gerenciar dados relacionados à IU considerando o ciclo de vida. A classe ViewModel permite que os dados sobrevivam às mudanças de configuração, como a rotação da tela.
 
* [LiveData](https://developer.android.com/topic/libraries/architecture/livedata): É uma classe armazenadora de dados observáveis. Diferente de um observável comum, o LiveData conta com reconhecimento de ciclo de vida, ou seja, ele respeita o ciclo de vida de outros componentes do app, como atividades, fragmentos ou serviços. Esse reconhecimento garante que o LiveData atualize apenas os observadores de componente do app que estão em um estado ativo no ciclo de vida.

Se desejar saber mais a respeito dos Arch Components recomendo a leitura completa do [Guia para a arquitetura do app](https://developer.android.com/jetpack/docs/guide)

## O que iremos contruir? 
Iremos criar um app que implementa a arquitetura recomendada pelo Google usando os Architecture Components. O aplicativo que iremos fazer armazena uma lista de palavras em um banco de dados SQLite Room e exibe-as em um `RecyclerView` O aplicativo é simples, mas suficientemente complexo para ser usado como modelo para desenvolver seus próprios projetos posteriormente.

Construiremos um aplicativo que: 

* Trabalha com um banco de dados para obter e salvar os dados e popula previamente o banco de dados com algumas palavras.
* Exibe todas as palavras em um `RecyclerView` em `MainActivity`.
* Abre uma segunda atividade quando o usuário toca no botão `+`. Quando o usuário digita uma palavra, adiciona a palavra ao banco de dados e à lista.

![Apresentação do App](https://raw.githubusercontent.com/eduardowgmendes/android-studies/master/images/what-we-build-app-demo.png)  

### Panorama Geral dos componentes de arquitetura que o exemplo utilizará 
O diagrama a seguir mostra todas as partes do aplicativo. Cada uma das caixas anexas (exceto o banco de dados SQLite) representa uma classe que você criará.

![Diagrama da estrutura do App](https://raw.githubusercontent.com/eduardowgmendes/android-studies/master/images/overview-diagram.png)

### O que você irá aprender? 
Como projetar e construir um aplicativo usando as bibliotecas da Architecture Components Room e Lifecycles.

Existem várias etapas para usar os componentes de arquitetura e implementar a arquitetura recomendada. O mais importante é criar um modelo mental do que está acontecendo, entendendo como as peças se encaixam e como os dados fluem. Ao trabalhar neste exemplo, não apenas copie e cole o código, mas tente começar a construir esse entendimento interno.

[Vamos Lá!](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/07-creating-app.md#criando-o-projeto)


