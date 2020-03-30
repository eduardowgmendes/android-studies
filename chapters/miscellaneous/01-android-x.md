## Visão geral do AndroidX

O AndroidX é o projeto de código aberto que a equipe Android usa para desenvolver, testar, incluir, controlar a versão e liberar bibliotecas no Jetpack.

O AndroidX é a principal melhoria feita da Support Library original do Android. Assim como a Support Library, o AndroidX é fornecido separadamente do SO Android e oferece compatibilidade com versões anteriores do Android. O AndroidX substitui totalmente a Support Library, oferecendo paridade de recursos e novas bibliotecas. Além disso, o AndroidX inclui os seguintes recursos:

* Todos os pacotes do AndroidX estão disponíveis em um namespace consistente, começando com a string androidx. Os pacotes da Support Library foram mapeados para os pacotes androidx.* correspondentes. Para mapear totalmente todas as classes e artefatos de compilação antigos para os novos, consulte a página de Refatoração de pacotes.

* Ao contrário da Support Library, os pacotes do AndroidX são mantidos e atualizados separadamente. Os pacotes do androidx usam o Versionamento semântico restrito a partir da versão 1.0.0. Você pode atualizar as bibliotecas do AndroidX no seu projeto de forma independente.

* Todo novo desenvolvimento da Support Library ocorrerá na biblioteca do AndroidX. Isso inclui a manutenção dos artefatos originais da Support Library e a introdução de novos componentes do Jetpack.

### Usar o AndroidX

Consulte [Como migrar para o AndroidX]() para saber como migrar um projeto existente.

Para usar o AndroidX em um novo projeto, é preciso definir o SDK de compilação para o Android 9.0 (API nível 28) ou posterior e configurar as duas sinalizações a seguir do Plug-in do Android para Gradle como true no arquivo gradle.properties.


* `android.useAndroidX`: quando definida como `true`, o plug-in do Android usa a biblioteca do AndroidX adequada em vez de uma Support Library. Por padrão, a sinalização será `false` se for não especificada.

* `android.enableJetifier`: quando definida como `true`, o plug-in do Android migra automaticamente as bibliotecas de terceiros existentes para usar o AndroidX, reescrevendo os respectivos binários. Por padrão, a sinalização será `false` se não for especificada.

