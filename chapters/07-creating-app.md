## Criando o Projeto
Abra o Android Studio e crie um novo projeto como a seguir: 

* Selecione a opção `Empty Activity` 
* Nomeie o projeto para *RoomWordSample*

![Prompt de Criação do Projeto](https://raw.githubusercontent.com/eduardowgmendes/android-studies/master/images/app-creating-prompt.png)

## Adicionando os repositórios ao Gradle
Agora é necessário adicionar a biblioteca de componentes no Gradle. No arquivo `build.gradle` **(Module: app)** faça as seguintes alterações: 

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

## Criando uma `Entity`
A entidade principal para nosso app são palavras, e você precisará de uma tabela simples para armazenar e obter esses valores quando for necessário:

![Diagrama da Entidade](https://raw.githubusercontent.com/eduardowgmendes/android-studies/master/images/85cfc6940626616c.png)

Vamos aprender como criar uma Entity usando os Archtecture Component do Android. Vamos lá!

Crie uma nova classe chamada de Word. Essa classe representará a entidade (que representa uma tabela SQL) para todas as palavras do aplicativo.

Cada membro público dessa classe representa uma coluna na tabela. Room usará esses membros para criar a tabela e instanciar os objetos através dos registros do banco de dados. Aqui está o código: 

```java
public class Word {

   private String mWord;

   public Word(@NonNull String word) {this.mWord = word;}

   public String getWord(){return this.mWord;}
}
```

Para tranformar essa classe em uma entidade precisamos anotá-la com a anotação `@Entity`. Anotações identificam como cada parte da classe se relaciona com um registro no banco de dados. O Room usa essas anotações para gerar o código SQL. 

Para saber mais sobre Annotations do Java consulte esse [link](https://www.alura.com.br/artigos/criando-anotacoes-no-java) 

Atualize sua classe Word com as anotações como mostrado abaixo: 

```java
@Entity(tableName = "word_table")
public class Word {

   @PrimaryKey
   @NonNull
   @ColumnInfo(name = "word")
   private String mWord;

   public Word(String word) {this.mWord = word;}

   public String getWord(){return this.mWord;}
}
```   

Quando você copia o código e cola no Android Studio, pode ser necessário importar as classes de anotação manualmente. Se a opção `Add unambiguous imports on the fly` não estiver configurada, você pode mover o cursor para o código de cada erro e usar o atalho de teclado "Project quick fix" (Alt + Enter no Windows / Linux, Option + Enter no Mac) para importar classes rapidamente.
   	
* `import androidx.room.ColumnInfo`

* `import androidx.room.Entity`

* `import androidx.room.PrimaryKey`

Vamos ver em detalhes o que estas `annotations` fazem: 

* `@Entity(tableName = "word_table")`: Cada classe anotada com `@Entity` representa uma tabela SQL. Anote sua classe de entidade com essa anotação para indicar ao Room que esta é uma classe `@Entity`. Se você não declarar o nome da tabela explicitamente o Room nomeará a tabela usando como base o nome da classe. Se não desejar esse efeito você pode especificar o nome da tabela explicitamente com outro nome de seu interesse. Neste exemplo nós nomeamos a tabela como "*word_table*".

* `@PrimaryKey`: Toda entidade precisa de uma Chave Primária. Para manter as coisas mais simples nesse exemplo, cada entidade `Word` atua como sua própria chave primária.

* `@NonNull`: Constraint que indica que o parâmetro, campo ou retorno de método não pode ser nulo.

* `@ColumnInfo(name= "word")`: Especifica o nome da coluna na tabela se você quiser um nome diferente do nome da variável membro. 

* Todos os campos que serão armazenados na tabela precisam ser públicos ou terem um método `getter`. Neste exemplo nós temos o `getWord()`.

Para uma lista completa de anotações do Room visite [Room package summary reference](https://developer.android.com/reference/androidx/room/package-summary.html).   

Veja também [Como definir dados usando entidades do Room](https://developer.android.com/training/data-storage/room/defining-data.html).

* **Dica**: Você pode gerar automaticamente chaves primárias com `AUTO INCREMENT` anotando parâmetros dessa forma: 

```java
@Entity(tableName = "word_table")
public class Word {

@PrimaryKey(autoGenerate = true)
private int id;

@NonNull
private String word;
//..other fields, getters, setters
}
``` 

## Criando o DAO
### O que é um DAO?
Um DAO (objeto de acesso a dados) valida seu SQL em tempo de compilação e o associa a um método. No DAO ROOM, você usa anotações úteis, como `@Insert`, para representar as operações mais comuns do banco de dados! A ROOM usa o DAO para criar uma API limpa para o seu código.

O DAO deve ser uma interface ou classe abstrata. Por padrão, todas as queries devem ser executadas em uma thread separada.

### Implementando o DAO
Vamos escrever o DAO que provê queries para: 

* Recuperar todas palavras em ordem alfabética. 
* Inserir uma palavra 
* Deletar todas as palavras 

Crie uma nova classe chamada `WordDAO` dessa forma: 

```java
@Dao
public interface WordDao {

   // allowing the insert of the same word multiple times by passing a 
   // conflict resolution strategy
   @Insert(onConflict = OnConflictStrategy.IGNORE)
   void insert(Word word);

   @Query("DELETE FROM word_table")
   void deleteAll();

   @Query("SELECT * from word_table ORDER BY word ASC")
   List<Word> getAlphabetizedWords();
}
``` 

Veremos essa classe em detalhes: 

1. `WordDAO` é uma `interface`; DAOs devem ser interfaces ou classes abstratas. 

2. A annotation `@Dao` identifica a classe como um DAO para o ROOM.

3. `void insert(Word word)`: Declara um método para inserir uma palavra no banco de dados.

4. A annotation `@Insert` é uma anotação especial de método ao qual não é necessário inserir nenhuma linha de SQL para que o Room faça a inserção no banco de dados. Há também as annotations `@Delete` e `@Update` para deletar e atualizar registros respectivamente. Não usaremos essas duas nesse exemplo. 

5. `onConflict = OnConflictStrategy.IGNORE`: Essa configuração faz com que o Room ignore uma nova palavra a ser inserida no banco de dados caso ela seja exatamente igual a que já está presente no banco de dados. Para saber mais sobre `conflict strategies`, leia a [documentação oficial](https://developer.android.com/reference/androidx/room/OnConflictStrategy.html)   

6. O método `deleteAll()` é usado para deletar todos os registros da tabela.

7. Não há nehuma annotation para deletar múltiplos registros de uma vez, portanto `deleteAll()` é anotado com `@Query(DELETE FROM word_table)` com o parâmetro SQL `DELETE FROM word_table`. **Se você conhece um pouco de SQL sabe que a cláusula `DELETE` sem a cláusula `WHERE` deleta todos os registros da tabela de uma só vez.**  

8. `List<Word> getAlphabetizedWords()`: Retorna uma lista de palavras em ordem alfabética ascendente usando o parâmetro SQL `ORDER BY word ASC` da anotação presente nele `@Query("SELECT * from word_table ORDER BY word ASC")`.

Veja mais sobre DAOs em [Como acessar dados usando DAOs do Room](https://developer.android.com/training/data-storage/room/accessing-data.html).

## LiveData
Quando os dados são alterados, geralmente você deseja executar alguma ação, como exibir os dados atualizados na interface do usuário. Isso significa que você deve observar os dados para que, quando eles mudem, outros possam reagir a essa mudança.

Dependendo de como os dados são armazenados, isso pode ser complicado. Observar alterações nos dados em vários componentes do seu aplicativo pode criar caminhos de dependência explícitos e rígidos entre os componentes. Isso dificulta o teste e a depuração, entre outras coisas.

O [`LiveData`](https://developer.android.com/topic/libraries/architecture/livedata.html), uma classe de [biblioteca do ciclo](https://developer.android.com/topic/libraries/architecture/lifecycle.html) de vida para observação de dados, resolve esse problema. Use um valor de retorno do tipo `LiveData` na descrição do seu método e o Room gera todo o código necessário para atualizar o `LiveData` quando o banco de dados for atualizado.

**Nota**: Se você usar o `LiveData` independentemente do Room, precisará gerenciar a atualização dos dados. O `LiveData` não possui métodos disponíveis publicamente para atualizar os dados armazenados.

Se você quiser atualizar os dados armazenados no `LiveData`, use `MutableLiveData` em vez do `LiveData`. A classe `MutableLiveData` possui dois métodos públicos que permitem definir o valor de um objeto `LiveData`, `setValue (T)` e `postValue (T)`. Normalmente, `MutableLiveData` é usado no `ViewModel` e, em seguida, o `ViewModel` expõe apenas objetos `LiveData` imutáveis ​​aos observadores.

Na classe `WordDAO`, modifique o tipo de retorno do método `getAlphabetizedWords()` para `LiveData<List<Word>>` dessa forma: 

```java
@Query("SELECT * from word_table ORDER BY word ASC")
   LiveData<List<Word>> getAlphabetizedWords();
```

Consulte a documentação do [`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData.html) para saber mais sobre outras maneiras de usar o `LiveData` ou assista a este vídeo [Architecture Components: LifeData and Lifecycle](https://www.youtube.com/watch?v=OMcDk2_4LSk&index=8&list=PLWz5rJ2EKKc9mxIBd0DRw9gwXuQshgmn2). 

  
 
