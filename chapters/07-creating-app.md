## Criando o Projeto
Abra o Android Studio e crie um novo projeto como a seguir: 

* Selecione a opção `Empty Activity` 
* Nomeie o projeto para *RoomWordSample*

![Prompt de Criação do Projeto](https://raw.githubusercontent.com/eduardowgmendes/android-studies/master/images/app-creating-prompt.png)

## Adicionando os repositórios ao Gradle
Agora é necessário adicionar a biblioteca de componentes no Gradle. No arquivo `build.gradle` (Module: app) faça as seguintes alterações: 

* Adicione o seguinte bloco [`compileOptions`](https://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.CompileOptions.html) dentro do bloco `android` para configurar o `sourceCompatibility` para 1.8, o que permite que nós utilizemos as [Lambdas](https://medium.com/dev-cave/por-tr%C3%A1s-da-programa%C3%A7%C3%A3o-funcional-do-java-8-bd8c0f172d45) da JDK 8 posteriormente.   
```gradle
compileOptions {
    sourceCompatibility = 1.8
    targetCompatibility = 1.8
}
```
Adicione as outras dependências de código ao final do bloco `dependencies`:

```gradle
// Room components
implementation "androidx.room:room-runtime:$rootProject.roomVersion"
annotationProcessor "androidx.room:room-compiler:$rootProject.roomVersion"
androidTestImplementation "androidx.room:room-testing:$rootProject.roomVersion"

// Lifecycle components
implementation "androidx.lifecycle:lifecycle-extensions:$rootProject.archLifecycleVersion"
annotationProcessor "androidx.lifecycle:lifecycle-compiler:$rootProject.archLifecycleVersion"

// UI
implementation "com.google.android.material:material:$rootProject.materialVersion"

// Testing
androidTestImplementation "androidx.arch.core:core-testing:$rootProject.coreTestingVersion"
```

Em seu arquivo `build.gradle` **(Project: RoomWordsSample)**, adicione os números de versão no final do arquivo como demonstrado no snippet abaixo e sincronize o projeto quando for solicitado: 

```gradle
ext {
    roomVersion = '2.2.1'
    archLifecycleVersion = '2.2.0-rc2'
    coreTestingVersion = '2.1.0'
    materialVersion = '1.0.0'
}
```

Pegue as versões mais recentes [nesta página](https://developer.android.com/topic/libraries/architecture/adding-components.html)
