## Introdução 
O propósito dos [*Architecture Components*]() do Android é prover, com bibliotecas para tarefas comuns como gerenciamento do ciclo de vida e persistência de dados. A biblioteca *Architecture Components* são parte da [Android JetPack]().

Os Archtecture Components o ajudarão a estruturar seu app de forma robusta, testável e com muito menos código clichê (*boilerplate code*).

### O que são os componentes de arquitetura recomendados?
Aqui vai uma breve introdução do que são e como eles funcionam juntos. Este breve tutorial foca apenas nos subconjuntos de componentes, chamados [`LiveData`](), [`ViewModel`]() e [`Room`](). Cada componente é explicado em detalhes assim que o utilizarmos. 

O diagrama abaixo demonstra a forma básica dessa arquitetura:

![Forma Básica da Arquitetura]()

* [Entity](): Classe anotada que consiste em uma entidade de tabela quando trabalhamos com [Room]()

* SQLite Database: Banco de Dados Nativo do Android. A biblioteca de persistência de dados Room cria e gerencia esse banco de dados para você. Usando Room não é necessário criar as tabelas na mão usando a classe `SQLiteOpenHelper`.

* [DAO](): *Data Access Object*. Trata-se de um padrão de projeto que separa a lógica de negócios da lógica de persistência de dados. Basicamente aqui o DAO mapeia as *queries* SQL para métodos. Quando se usa um DAO em conjunto com o Room, você invoca métodos, e o Room toma conta do resto para você.

* [Room Database](): Simplifica o trabalho com persistência de dados e serve como um ponto de acesso ao banco de dados SQL da aplicação. O Room usa o DAO para centralizar todas as *queries* ao banco de dados.

* [Repository](): Gerencia multiplas fontes de dados (*data sources*). 

* [ViewModel](): Atua como um intermediador entre o Repository (dados) e a IU (views). A IU não precisa se preocupar mais com a origem dos dados. A classe ViewModel foi projetada para armazenar e gerenciar dados relacionados à IU considerando o ciclo de vida. A classe ViewModel permite que os dados sobrevivam às mudanças de configuração, como a rotação da tela.
 
* [LiveData](): É uma classe armazenadora de dados observáveis. Diferente de um observável comum, o LiveData conta com reconhecimento de ciclo de vida, ou seja, ele respeita o ciclo de vida de outros componentes do app, como atividades, fragmentos ou serviços. Esse reconhecimento garante que o LiveData atualize apenas os observadores de componente do app que estão em um estado ativo no ciclo de vida.

Se desejar saber mais a respeito dos Arch Components recomendo a leitura completa do [Guia para a arquitetura do app]()

## O que iremos contruir? 
Iremos criar um app que implementa a arquitetura recomendada pelo Google usando os Architecture Components. O aplicativo que iremos fazer armazena uma lista de palavras em um banco de dados SQLite Room e exibe-as em um `RecyclerView` O aplicativo é simples, mas suficientemente complexo para ser usado como modelo para desenvolver seus próprios projetos posteriormente.

Construiremos um aplicativo que: 

* Trabalha com um banco de dados para obter e salvar os dados e preenche previamente o banco de dados com algumas palavras.
* Exibe todas as palavras em um `RecyclerView` em `MainActivity`.
* Abre uma segunda atividade quando o usuário toca no botão `+`. Quando o usuário digita uma palavra, adiciona a palavra ao banco de dados e à lista.

------|-------|-------|
![](https://raw.githubusercontent.com/eduardowgmendes/android-studies/master/images/d4da895890891e9f.png) | ![](https://raw.githubusercontent.com/eduardowgmendes/android-studies/master/images/72a92cdf0e43f469.png) | ![](https://raw.githubusercontent.com/eduardowgmendes/android-studies/master/images/231586bcd0902d06.png)  
  


  
  

   
