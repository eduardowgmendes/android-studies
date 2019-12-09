
## Criando o Projeto
Abra o Android Studio e crie um novo projeto como a seguir: 

* Selecione a opção `Empty Activity` 
* Nomeie o projeto para *RoomWordSample*

![Prompt de Criação do Projeto](https://raw.githubusercontent.com/eduardowgmendes/android-studies/master/images/app-creating-prompt.png)

## Adicionando as dependências ao Gradle
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

## `LiveData`
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

Se desejar saber mais sobre o padrão `Observer` veja [Padrão de Projeto Observer](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/08-observer-pattern-and-more-details.md#padr%C3%A3o-de-projeto-observer).  

## `RoomDatabase`
O banco de dados Room oferece uma camada de abstração sobre o SQLite para permitir acesso fluente ao banco de dados e, ao mesmo tempo, aproveitar toda a capacidade do SQLite.

Muito antigamente para trabalhar com banco de dados no Android era necessário ter um pouco de paciência e bastante cuidado pois era necessário utilizar uma classe chamada `SQLiteOpenHelper` e definir todos os métodos de consulta e manipulação do banco além de trabalhar com Cursores. Era um trabalho imenso e propenso a erros já que se escrevêssemos qualquer querie incorretamente toda a aplicação falhava e tínhamos que buscar os erros um a um. 

Apps que processam quantidades não triviais de dados estruturados podem se beneficiar muito da persistência desses dados localmente. O caso de uso mais comum é armazenar as partes de dados relevantes em cache. Dessa forma, quando o dispositivo não conseguir acessar a rede, o usuário ainda poderá navegar pelo conteúdo enquanto estiver off-line. Todas as alterações de conteúdo feitas pelo usuário serão sincronizadas com o servidor quando o dispositivo ficar on-line novamente.

É altamente recomendável usar o Room em vez do SQLite, porque o Room cuida desses problemas para você. No entanto, se você prefere usar APIs SQLite diretamente, leia Salvar dados usando o SQLite.

* Room é uma camada de abstração sobre um banco de dados SQLite.

* O Room cuida de tarefas comuns que você costumava lidar com um [SQLiteOpenHelper](https://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html).

* O Room usa o DAO para emitir consultas ao banco de dados.

* Por padrão, para evitar um desempenho ruim da interface do usuário, o Room não permite emitir consultas no thread principal. Quando as consultas de sala retornam o [LiveData](https://developer.android.com/topic/libraries/architecture/livedata), as consultas são executadas automaticamente de forma assíncrona em um encadeamento em segundo plano.

* O Room fornece verificações em tempo de compilação de instruções SQLite.

### Implementando o Banco de Dados 

Sua classe que representará um banco de dados Room deve ser declarada como `abstract` e precisa estender `RoomDatabase`. Normalmente você só precisará de uma instância singleton desse banco de dados para toda a sua aplicação.

Vamos criar uma classe e nomeá-la como `WordRoomDatabase`: 

```java
@Database(entities = {Word.class}, version = 1, exportSchema = false)
public abstract class WordRoomDatabase extends RoomDatabase {

   public abstract WordDao wordDao();

   private static volatile WordRoomDatabase INSTANCE;
   private static final int NUMBER_OF_THREADS = 4;
   static final ExecutorService databaseWriteExecutor =
        Executors.newFixedThreadPool(NUMBER_OF_THREADS);

   static WordRoomDatabase getDatabase(final Context context) {
        if (INSTANCE == null) {
            synchronized (WordRoomDatabase.class) {
                if (INSTANCE == null) {
                    INSTANCE = Room.databaseBuilder(context.getApplicationContext(),
                            WordRoomDatabase.class, "word_database")
                            .build();
                }
            }
        }
        return INSTANCE;
    }
}
``` 
Vejamos em detalhes esse código: 

1. Conforme mencionado anteriormente a classe que representará o banco de dados Room deve ser abstrata e estender `RoomDatabase`.

2. Você anota a classe como um banco de dados do Room com `@Database` e usa os parâmetros de anotação para declarar as entidades que pertencem ao banco de dados e definir o número da versão. Cada entidade corresponde a uma tabela que será criada no banco de dados. As migrações de banco de dados estão além do escopo deste exemplo, portanto, definimos `exportSchema` como `false` aqui para evitar mensagens de alerta durante o build da aplicação. Em um aplicativo real, considere definir um diretório para o Room usar para exportar o esquema, para que você possa verificar o esquema atual no seu sistema de controle de versão.

3. Você faz o banco de dados prover seus DAOs criando um método getter abstrato para cada `@Dao`.

4. Definimos um [Singleton](https://pt.wikipedia.org/wiki/Singleton) para prevenir múltiplas instâncias do banco de dados abertas simultâneamente.

5. `getDatabase()`retorna o singleton. Ele criará o banco de dados na primeira vez que for acessado, usando o construtor de banco de dados do Room para criar um objeto `RoomDatabase` no contexto do aplicativo a partir da classe `WordRoomDatabase` e o nomeie como "word_database".

6. Criamos um objeto ExecutorService com um número fixo de threads que serão usadas para realizar as operações sobre o banco de dados de forma assíncrona em uma thread rodando em background.  

## `Repository`
### O que é um `Repository`?
Uma classe `Repository` abstrai o acesso a várias fontes de dados. O Repositório não faz parte das bibliotecas de Componentes de Arquitetura, mas é uma prática recomendada para separação e arquitetura de código. Uma classe `Repository` fornece uma API limpa para acesso a dados para o restante do aplicativo.

![Respository Diagrama](https://raw.githubusercontent.com/eduardowgmendes/android-studies/master/images/repository-diagram.png)

### Porque usar um `Repository`?
Um Repositório gerencia consultas e permite usar vários back-ends. No exemplo mais comum, o Repositório implementa a lógica para decidir buscar dados de uma rede ou usar resultados armazenados em cache em um banco de dados local.

### Implementando o `Repository`
Crie uma classe chamada `WordRepository` como a seguir: 

```java
class WordRepository {

    private WordDao mWordDao;
    private LiveData<List<Word>> mAllWords;

    // Note that in order to unit test the WordRepository, you have to remove the Application
    // dependency. This adds complexity and much more code, and this sample is not about testing.
    // See the BasicSample in the android-architecture-components repository at
    // https://github.com/googlesamples
    WordRepository(Application application) {
        WordRoomDatabase db = WordRoomDatabase.getDatabase(application);
        mWordDao = db.wordDao();
        mAllWords = mWordDao.getAlphabetizedWords();
    }

    // Room executes all queries on a separate thread.
    // Observed LiveData will notify the observer when the data has changed.
    LiveData<List<Word>> getAllWords() {
        return mAllWords;
    }

    // You must call this on a non-UI thread or your app will throw an exception. Room ensures
    // that you're not doing any long running operations on the main thread, blocking the UI.
    void insert(Word word) {
        WordRoomDatabase.databaseWriteExecutor.execute(() -> {
            mWordDao.insert(word);
        });
    }
}
```
Vejamos em detalhes o código: 

1. O DAO é passado para o construtor do repositório, em oposição a todo o banco de dados. Isso ocorre porque você só precisa acessar o DAO, pois ele contém todos os métodos de leitura / gravação do banco de dados. Não há necessidade de expor o banco de dados inteiro ao repositório.

2. O método `getAllWords()` retorna a lista de palavras `LiveData` de Room; podemos fazer isso devido à forma como definimos o método `getAlphabetizedWords()` para retornar o `LiveData` na etapa "[LiveData](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/07-creating-app.md#livedata)". O quarto executa todas as consultas em um thread separado. O `LiveData` observado notificará o observador no thread principal quando os dados forem alterados.

3. Como não precisamos executar a inserção no thread principal, usamos o `ExecutorService` que criamos no `WordRoomDatabase` para executar a inserção em um thread em segundo plano.

Repositórios destinam-se a mediar entre diferentes fontes de dados. Neste exemplo simples, você tem apenas uma fonte de dados, portanto, o Repositório não faz muito. Veja o [BasicSample](https://github.com/android/architecture-components-samples/tree/master/BasicSample) para uma implementação mais complexa.

## `ViewModel`
### O que é o ViewModel?
A função do `ViewModel` é fornecer dados para a interface do usuário e sobreviver a alterações na configuração. Um `ViewModel` atua como um centro de comunicação entre o `Repository` e a UI. Você também pode usar um `ViewModel` para compartilhar dados entre fragmentos. O `ViewModel` faz parte da biblioteca de [LifeCycle](https://developer.android.com/topic/libraries/architecture/lifecycle.html).

![ViewModel Diagrama](https://raw.githubusercontent.com/eduardowgmendes/android-studies/master/images/viewmodel-diagram.png)

Para um guia introdutório a este tópico, consulte [Visão geral do ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel.html) ou a publicação no blog [ViewModels: A Simple Example](https://medium.com/androiddevelopers/viewmodels-a-simple-example-ed5ac416317e).

### Porque usar um `ViewModel`?
Um ViewModel mantém os dados da interface do usuário do seu aplicativo de maneira consciente do ciclo de vida, que sobrevive às alterações na configuração. A separação dos dados de interface do usuário do seu aplicativo das classes `Activity` e `Fragment` permite que você siga melhor o princípio de responsabilidade única: **Suas atividades e fragmentos são responsáveis ​​por desenhar dados para a tela, enquanto o seu `ViewModel` pode manter e processar todos os dados necessários para a interface do usuário.**

No ViewModel, use o LiveData para dados alteráveis ​​que a interface do usuário usará ou exibirá. O uso do LiveData possui vários benefícios:

* Você pode colocar um `observer` nos dados (em vez de pesquisar mudanças) e atualizar apenas o
a interface do usuário quando os dados realmente mudam.

* O repositório e a interface do usuário são completamente separados pelo `ViewModel`.

* Não há chamadas de banco de dados do `ViewModel` (tudo isso é tratado no `Repository`), tornando o código mais testável.

### Implementando o `ViewModel`
Crie a classe `WordViewModel` conforme a seguir: 

```java
public class WordViewModel extends AndroidViewModel {

   private WordRepository mRepository;

   private LiveData<List<Word>> mAllWords;

   public WordViewModel (Application application) {
       super(application);
       mRepository = new WordRepository(application);
       mAllWords = mRepository.getAllWords();
   }

   LiveData<List<Word>> getAllWords() { return mAllWords; }

   public void insert(Word word) { mRepository.insert(word); }
}
```
Vamos analisar o código em detalhes: 

1. Criou uma classe chamada `WordViewModel` que obtém `Application` como parâmetro e estende o `AndroidViewModel`.

2. Adicionada uma variável de membro privado para manter uma referência ao repositório.

3. Adicionado um método `getAllWords()` para retornar uma lista de palavras em cache.

4. Implementado um construtor que cria o `WordRepository`.

5. No construtor, inicializou o `allWords()` `LiveData` usando o repositório.

6. Criou um método de wrapper `insert()` que chama o método `insert()` do `Repository`. Dessa maneira, a implementação de `insert()` é encapsulada na interface do usuário.

**Aviso**: não mantenha uma referência a um contexto que tenha um ciclo de vida mais curto que o seu `ViewModel`! Exemplos são:

* `Activity`
* `Fragment`
* `View`

Manter uma referência pode causar um vazamento de memória, por exemplo o `ViewModel` tem uma referência a uma atividade destruída! Todos esses objetos podem ser destruídos pelo sistema operacional e recriados quando houver uma alteração na configuração, e isso pode acontecer muitas vezes durante o ciclo de vida de um `ViewModel`.

Se você precisar de um `application context` (que tem um ciclo de vida que dura tanto quanto o aplicativo), use `AndroidViewModel`, conforme mostrado aqui.

**Importante**: os `ViewModels` não sobrevivem ao processo do aplicativo ser morto em segundo plano quando o sistema operacional precisa de mais recursos. Para dados da interface do usuário que precisam sobreviver à morte do processo devido à falta de recursos, você pode usar o módulo [Saved State para ViewModels](https://developer.android.com/topic/libraries/architecture/viewmodel-savedstate). Saiba mais [aqui](https://medium.com/androiddevelopers/viewmodels-persistence-onsaveinstancestate-restoring-ui-state-and-loaders-fc7cc4a6c090?).

Para saber mais sobre o `ViewModel`assista ao vídeo [Architecture Components: ViewModel](https://www.youtube.com/watch?v=c9-057jC1ZA).

## Hora de criar o Layout do App
A próxima coisa a ser feita é criar o layout XML e adicionar um `RecyclerView`. 
Para começar adicione o conteúdo abaixo ao arquivo `values/styles.xml`: 

```xml
<!-- The default font for RecyclerView items is too small.
The margin is a simple delimiter between the words. -->
<style name="word_title">
   <item name="android:layout_width">match_parent</item>
   <item name="android:layout_marginBottom">8dp</item>
   <item name="android:paddingLeft">8dp</item>
   <item name="android:background">@android:color/holo_orange_light</item>
   <item name="android:textAppearance">@android:style/TextAppearance.Large</item>
</style>
```    

Crie também um arquivo de layout chamado `layout/recyclerview_item_layout.xml`: 

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <TextView
        android:id="@+id/textView"
        style="@style/word_title"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@android:color/holo_orange_light" />
</LinearLayout>
``` 
No arquivo de layout `layout/activity_main.xml` substitua o `TextView` com um `RecylerView` e adicione também um `Floating Action Button` (FAB) com as seguintes configurações: 

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerview"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:background="@android:color/darker_gray"
        tools:listitem="@layout/recyclerview_item"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:id="@+id/fab"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp" />

</androidx.constraintlayout.widget.ConstraintLayout>
```   

Para que o FAB corresponda a ação de adicionar adicione um ícone de adicionar como `VectorDrawable` nos diretórios do seu projeto.

No final o seu FAB se parecerá com o seguinte: 

```xml
<com.google.android.material.floatingactionbutton.FloatingActionButton
        android:id="@+id/fab"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:src="@drawable/ic_add_black_24dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp" />
```  

## Adicionando o RecyclerView
Você exibirá os dados em um `RecyclerView`, que é um pouco melhor do que apenas jogar os dados em um `TextView`. Este exemplo supõe que você saiba como funciona o `RecyclerView`, `RecyclerView.LayoutManager`, `RecyclerView.ViewHolder` e `RecyclerView.Adapter`.

Observe que a variável `mWords` no adaptador armazena em cache os dados. Na próxima tarefa, você adiciona o código que atualiza os dados automaticamente.

Crie uma classe `WordListAdapter` que estenda `RecyclerView.Adapter`. Aqui está o código:

```java
public class WordListAdapter extends RecyclerView.Adapter<WordListAdapter.WordViewHolder> {

   class WordViewHolder extends RecyclerView.ViewHolder {
       private final TextView wordItemView;

       private WordViewHolder(View itemView) {
           super(itemView);
           wordItemView = itemView.findViewById(R.id.textView);
       }
   }

   private final LayoutInflater mInflater;
   private List<Word> mWords; // Cached copy of words

   WordListAdapter(Context context) { mInflater = LayoutInflater.from(context); }

   @Override
   public WordViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
       View itemView = mInflater.inflate(R.layout.recyclerview_item, parent, false);
       return new WordViewHolder(itemView);
   }

   @Override
   public void onBindViewHolder(WordViewHolder holder, int position) {
       if (mWords != null) {
           Word current = mWords.get(position);
           holder.wordItemView.setText(current.getWord());
       } else {
           // Covers the case of data not being ready yet.
           holder.wordItemView.setText("No Word");
       }
   }

   void setWords(List<Word> words){
       mWords = words;
       notifyDataSetChanged();
   }

   // getItemCount() is called many times, and when it is first called,
   // mWords has not been updated (means initially, it's null, and we can't return null).
   @Override
   public int getItemCount() {
       if (mWords != null)
           return mWords.size();
       else return 0;
   }
}
``` 
Adicione o RecyclerView no método `onCreate()` de `MainActivity`.

No método `onCreate()` após `setContentView`:

```java
RecyclerView recyclerView = findViewById(R.id.recyclerview);
final WordListAdapter adapter = new WordListAdapter(this);
recyclerView.setAdapter(adapter);
recyclerView.setLayoutManager(new LinearLayoutManager(this));
```

Execute seu aplicativo para garantir que tudo funcione. Não há itens, porque você ainda não conectou os dados, portanto, o aplicativo deve exibir um plano de fundo cinza sem nenhum item da lista.

## Populando o banco de dados 

Não há dados no banco de dados. Você adicionará dados de duas maneiras: 

* Adicione alguns dados quando o banco de dados for aberto e adicione uma Atividade para adicionar palavras.

Para excluir todo o conteúdo e preencher novamente o banco de dados sempre que o aplicativo for iniciado, crie um `RoomDatabase.Callback` e sobrescreva a implementação de `onOpen()`.

**Nota**: Se você deseja preencher apenas o banco de dados na primeira vez que o aplicativo é iniciado, você pode substituir o método `onCreate()` no `RoomDatabase.Callback`.

Aqui está o código para criar o retorno de chamada na classe `WordRoomDatabase`. Como você não pode executar operações de banco de dados da Room no thread da interface do usuário, `onOpen()` usa o `databaseWriteExecutor` definido anteriormente para executar uma lambda em um thread em segundo plano. O lambda exclui o conteúdo do banco de dados e o preenche com as duas palavras "Olá" e "Mundo". Sinta-se livre para adicionar mais palavras!

```java
private static RoomDatabase.Callback sRoomDatabaseCallback = new RoomDatabase.Callback() {
    @Override
    public void onOpen(@NonNull SupportSQLiteDatabase db) {
        super.onOpen(db);

        // If you want to keep data through app restarts,
        // comment out the following block
        databaseWriteExecutor.execute(() -> {
            // Populate the database in the background.
            // If you want to start with more words, just add them.
            WordDao dao = INSTANCE.wordDao();
            dao.deleteAll();

            Word word = new Word("Hello");
            dao.insert(word);
            word = new Word("World");
            dao.insert(word);
        });
    }
};
```
Em seguida, adicione o retorno de chamada à sequência de criação do banco de dados antes de chamar `.build()` no `Room.databaseBuilder()`

```java
.addCallback(sRoomDatabaseCallback)
```
 
