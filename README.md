## Motivação
A criação deste repositório assim como o estudo da arquitetura de desenvolvimento expressa aqui (***MVVM***), reflete uma necessidade pessoal de atualização de conhecimentos e mindset. Aprendi a desenvolver para a plataforma Android há algum tempo e nesse período as arquiteturas disponíveis para estruturar um aplicativo Android era o bom e velho `MVC` e também o `MVP`. 

O ***MVVM `Model-View-ViewModel`*** é o mais recente criado em 2006 pelo pessoal da Microsoft para desenvolvimento do Silverlight e em 2017 depois com o advento do [Android JetPack](https://developer.android.com/jetpack/) passou a ser a arquitetura recomendada para desenvolvimento de apps no Android. O problema é que eu nessa época eu acabei perdendo o fio da meada, afinal havia acabado de aprender uma montanha de conceitos sobre Java, Web e Android. 

Recentemente fiz um teste para uma grande empresa de tecnologia e acabei infelizmente não obtendo êxito, o que me deixou com um misto de tristeza e desânimo. Mas não adianta chorar por algo que eu não sei. Tudo isso serviu de aprendizagem e experiência e eu agradeço pela oportunidade pois foi graças a esse teste que eu estou aqui agora escrevendo todo esse material para auxiliar pessoas que também estão no mesmo barco que eu. 

**Nota**: Se você quiser dar uma olhada no meu projeto de teste final, sinta-se à vontade para clicar [aqui](https://github.com/eduardowgmendes/desafio-mobile). Lá também disponibilizei o `apk` para testar em seu dispositivo Android real.      

Nossa área é uma das mais prósperas no mercado e também é a que mais rápido se atualiza. Ela exige do profissional que ele sempre se mantenha no mesmo ritmo, e isso é bom. Para quem curte sempre aprender é sensacional! É o meu caso! E se for o seu também, vamos juntos compreender os pormenores dessa arquitetura.

Recomendo a leitura prévia do conteúdo disponível no Android Developers - [Guia para a arquitetura do app](https://developer.android.com/jetpack/docs/guide)


## Estrutura do repositório
Todo o conteúdo está presente na pasta `/chapters` e conforme for atualizando com mais assuntos tudo ainda estará presente nesse diretório. O diretório `/images` serve apenas para as imagens presentes no conteúdo. Já o diretório `/android-basics` reúne todo o conteúdo comum ao desenvolvimento Android.    

A primeira parte do conteúdo é apenas uma tradução de um CodeLab que eu vi recentemente sobre `Room` e `ViewModel`, `LiveData` e etc, com algumas pequenas alterações para manter o contexto geral do conteúdo escrito aqui. O conteúdo original está disponível aqui: [Android Room with a View - Java](https://codelabs.developers.google.com/codelabs/android-room-with-a-view)  

## Links úteis 

### CODELAB Introdução ao MMVM - *Model View-Model*
Abaixo você encontra informações relevantes sobre a arquitetura MVVM recomendada pelo Google para estruturar suas aplicações e promover melhor extensiblidade e manutenibilidade ao seu projeto:

### Introdução ao MVVM 
* [Android `Room` com `ViewModel`, `LiveData`, `DAOs` e etc](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/09-introduction-mvvm.md#introdu%C3%A7%C3%A3o).
* [Componentes de Arquitetura Recomendados](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/09-introduction-mvvm.md#o-que-s%C3%A3o-os-componentes-de-arquitetura-recomendados)
* [Overview do Exemplo](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/09-introduction-mvvm.md#o-que-iremos-contruir)
* [Panorama Geral dos componentes de arquitetura](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/09-introduction-mvvm.md#panorama-geral-dos-componentes-de-arquitetura-que-o-exemplo-utilizar%C3%A1)

### Começando com MVVM - Criando um Projeto 
* [Adicionando dependências ao Gradle](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/miscellaneous/07-creating-app.md#adicionando-as-depend%C3%AAncias-ao-gradle)
* [`Entity`](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/miscellaneous/07-creating-app.md#criando-uma-entity)
* [Criando o `DAO`](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/miscellaneous/07-creating-app.md#criando-o-dao)
* [`LiveData`](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/miscellaneous/07-creating-app.md#livedata)
* [`RoomDatabase`](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/miscellaneous/07-creating-app.md#roomdatabase)
* [Implementando o Banco de Dados](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/miscellaneous/07-creating-app.md#implementando-o-banco-de-dados)
* [`Repository`](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/miscellaneous/07-creating-app.md#repository)
* [`ViewModel`](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/miscellaneous/07-creating-app.md#viewmodel)

## Começando no Android
Abaixo você encontra um índice para os conteúdos comuns ao desenvolvimento Android. Tudo o que você precisa para começar a desenvolver na plataforma mobile mais utilizada do mundo. 

### Desenvolvimento Básico Android

* [Broadcast Receiver](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/android-basics/broadcast-receiver.md)
* [Notifications](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/android-basics/notifications.md)
* [Alarm Manager](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/android-basics/alarm-manager.md)
* [Services e JobInfo](https://github.com/eduardowgmendes/android-studies/blob/master/chapters/android-basics/services-jobinfo.md)

   
                        


  
  

   
