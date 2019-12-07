## Android Jetpack

O Jetpack é um conjunto de bibliotecas, ferramentas e orientações para ajudar os desenvolvedores a criar apps de alta qualidade com mais facilidade. Esses componentes ajudam você a seguir as práticas recomendadas, acabando com os códigos clichê e simplificando tarefas complexas, para que você possa se concentrar na parte do código do seu interesse.

O Jetpack é composto por bibliotecas de pacotes do [androidx.*]() separadas das APIs da plataforma. Isso significa que ele oferece compatibilidade com versões anteriores e é atualizado com mais frequência do que a plataforma Android, garantindo que você sempre tenha acesso às versões mais recentes dos componentes do Jetpack.

### Acelere o desenvolvimento
Os componentes podem ser adotados individualmente, mas foram projetados para trabalhar juntos e aproveitar os recursos da linguagem Kotlin, que aumentam sua produtividade. 

### Elimine os códigos clichê
O Android Jetpack administra atividades tediosas, como tarefas em segundo plano, navegação e gerenciamento do ciclo de vida, para que você possa se concentrar naquilo que torna seu app incrível. 

### Crie apps robustos e de alta qualidade
Desenvolvidos com práticas de design modernas, os componentes do Android Jetpack reduzem as falhas e os vazamento de memória com a compatibilidade incorporada para versões anteriores. 

## Componentes do Android Jetpack
Os componentes do Jetpack são uma coleção de bibliotecas que podem ser individualmente adotadas e preparadas para trabalharem juntas, aproveitando os recursos da linguagem Kotlin para tornar você mais produtivo. Use todos eles ou combine-os como quiser.

### Base  
Os componentes de base fornecem funcionalidade transversal, como retrocompatibilidade, testes e compatibilidade com a linguagem Kotlin. 

### Arquitetura 
Os [Componentes de arquitetura]() ajudam você a criar apps robustos, testáveis e de fácil manutenção.

### Comportamento
Os componentes de comportamento ajudam seu app a se integrar aos serviços padrão do Android, como notificações, permissões, compartilhamento e o Assistente.

# IU 
Os componentes de IU fornecem widgets e assistentes para tornar seu app não apenas fácil, como também agradável de usar. Aprenda sobre o [Jetpack Compose](), que ajuda a simplificar o desenvolvimento da interface do usuário. 

## Migrando para o Android X

O AndroidX substitui as APIs da Biblioteca de Suporte original por pacotes no namespace androidx. Apenas o pacote e os nomes de artefatos Maven foram alterados. Os nomes de classes, métodos e campos permanecem os mesmos.

**Observação**: recomendamos que você trabalhe em um branch separado ao fazer a migração. Tente também evitar a refatoração do código durante esse processo.

### Pré-requisitos

Antes de fazer a migração, atualize seu app. Recomendamos que você atualize o projeto para usar a versão final da Biblioteca de Suporte: versão 28.0.0. Isso porque os artefatos do AndroidX na versão 1.0.0 são equivalentes binários aos artefatos da Biblioteca de Suporte 28.0.0.

## Migrar um projeto existente usando o Android Studio

Com o Android Studio 3.2 e versões posteriores, é possível migrar um projeto existente para o AndroidX selecionando **Refactor > Migrate to AndroidX** na barra de menu.

O comando "refactor" uso dois sinalizadores. Por padrão, ambos são definidos como true no arquivo `gradle.properties`:

```gradle
android.useAndroidX=true
```

O plug-in do Android usa a biblioteca adequada do AndroidX em vez de uma Biblioteca de Suporte.

```gradle
android.enableJetifier=true
```

O plug-in do Android migra automaticamente as bibliotecas de terceiros existentes para o AndroidX reescrevendo os binários delas.

**Observação**: a migração integrada do Android Studio pode não processar tudo. Dependendo da sua configuração de compilação, pode ser necessário atualizar manualmente os scripts de compilação e os mapeamentos Proguard. Por exemplo, se você mantém sua configuração de dependências em um arquivo de compilação diferente, use os arquivos de mapeamento mencionados abaixo para revisar e atualizar suas dependências com os pacotes correspondentes do AndroidX.
