## Jetpack Compose

### Um kit de ferramentas declarativo para criação da IU	
O Jetpack Compose é um kit de ferramentas desagregado, projetado para simplificar o desenvolvimento da IU. Ele combina um modelo de programação reativa com a concisão e a facilidade de uso da linguagem de programação Kotlin.

O Jetpack Compose utiliza `composable functions` ao invés dos layouts em XML para definir componentes de Interface de Usuário. 

**Observação**: o Jetpack Compose está em um estágio inicial de exploração pré-alfa. As superfícies da API dele ainda não foram finalizadas e não devem ser usadas para produção.

## Princípios básicos

### Kotlin conciso e idiomático
Criado com os benefícios que o Kotlin proporciona: conciso, seguro e totalmente interoperável com Java. Desenvolvido para reduzir radicalmente a quantidade de código clichê que você precisa escrever. Assim, você pode se concentrar no código do app e evitar classes inteiras de erros.

### Declarativo
Totalmente declarativo para definir componentes da IU, incluindo o desenho e a criação de layouts personalizados. Basta descrever sua IU como um conjunto de funções combináveis, e o framework tratará das otimizações avançadas da IU e das atualizações automáticas na hierarquia de visualizações.

### Compatível
Compatível com as visualizações existentes, para que você possa fazer as combinações que quiser e adotar no seu ritmo, com acesso direto a todas as APIs do Android e do Jetpack.

### Ativação de apps incríveis
Criado com o Material Design pronto para uso e com animações desde o início. Assim, é fácil criar apps incríveis e cheios de movimento.

### Desenvolvimento acelerado
Acelere o desenvolvimento escrevendo menos código e usando ferramentas como "Apply Changes" e visualização em tempo real.

## Uma olhada rápida
O Jetpack Compose está em desenvolvimento no Android Open Source Project. Ele tem dois componentes principais:

* Compose UI Library, que contém o kit de ferramentas principal da IU com layout, entrada, texto, animações, estilos, widgets e gráficos.

* Compose Compiler, um plug-in de compilação personalizado do Kotlin que usa funções combináveis e atualiza automaticamente a hierarquia da IU.

Um aplicativo Compose é formado por funções combináveis que transformam os dados do aplicativo em uma hierarquia da IU. Basta usar uma função para criar um novo componente de IU.

Para criar uma função combinável, basta adicionar a anotação @Composable ao nome da função. Internamente, o Compose usa um plug-in de compilação personalizado do Kotlin. Assim, quando os dados subjacentes são alterados, as funções combináveis podem ser invocadas novamente para gerar uma hierarquia de IU atualizada. O exemplo simples abaixo imprime uma string na tela.

```kotlin
    import androidx.compose.*
    import androidx.ui.core.*

    @Composable
    fun Greeting(name: String) {
       Text ("Hello $name!")
    }    
```
As APIs da Jetpack Compose UI Library estão no diretório frameworks/support/ui do AOSP. O Compose Compiler e o código de tempo de execução podem ser encontrados em `frameworks/support/compose`.

## Compose UI library
A Jetpack Compose UI library contém estes módulos:

* android-text/
    * Implementações dependentes da pilha de texto específica do Android
* android-view/
    * Wrappers e adaptadores para Views existentes do Android
* animation/
    * Componentes de animação
* animation-core/
    * Declarações internas para o sistema de animações
* core/
    * Classes básicas usadas em todo o sistema, incluindo primitivos, gráficos e pintura
* framework/
    * Componentes básicos expostos pelo sistema como elementos essenciais. Isso inclui Draw, Layout, Text etc.
* layout/
    * Componentes básicos de layout
* material/
    * Conjunto de componentes da IU criados de acordo com as especificações do Material Design
* platform/
    * Implementação interna que permite separar a implementação do Android dos testes no host
* test/
    * Framework de teste
* text/
    * Mecanismo de texto 
