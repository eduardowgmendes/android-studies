## Visão geral do gerenciamento de memória

O Android Runtime (ART) e a máquina virtual Dalvik usam paginação e mapeamento em memória (mmapping) para gerenciar a memória. Isso significa que qualquer memória que um app modifique, seja alocando novos objetos ou tocando em páginas mapeadas em memória, continua residindo na RAM e não pode ser despaginada. A única maneira de liberar a memória de um app é liberar as referências de objetos mantidas por ele, disponibilizando a memória para o coletor de lixo. Isso ocorre com uma exceção: todos os arquivos mapeados em memória sem modificação, como código, podem ser despaginados da RAM caso o sistema queira usar essa memória em outro lugar. 

### Monitorar a memória disponível e o uso de memória
Antes de corrigir os problemas de uso de memória no seu app, é necessário primeiro encontrá-los. O Memory Profiler do Android Studio ajuda você a encontrar e diagnosticar problemas de memória das seguintes maneiras: 

1. Veja como seu app aloca memória ao longo do tempo. O Memory Profiler mostra um gráfico em tempo real de quanta memória seu app está usando, do número de objetos Java alocados e de quando ocorre a coleta de lixo.
2. Inicie os eventos de coleta de lixo e tire um instantâneo do heap de Java enquanto o app é executado.
3. Registre as alocações de memória do app e depois inspecione todos os objetos alocados, veja o rastreamento de pilha para cada alocação e vá para o código correspondente no editor do Android Studio.

### Liberar memória em resposta a eventos
Conforme descrito na Visão geral do gerenciamento de memória no Android, o Android pode recuperar a memória do app de várias maneiras ou encerrar totalmente o app, se necessário, para liberar memória para tarefas essenciais. Para ajudar a equilibrar ainda mais a memória do sistema e evitar a necessidade de encerrar o processo do app, implemente a interface ComponentCallbacks2 nas suas classes de Activity. O método de callback `onTrimMemory()` fornecido permite que seu app ouça eventos relacionados à memória independentemente de o app estar em primeiro ou segundo plano e, em seguida, libere objetos em resposta ao ciclo de vida do app ou aos eventos que indicam que o sistema precisa recuperar memória.

Por exemplo, você pode implementar o callback `onTrimMemory()` para responder a diferentes eventos relacionados à memória, conforme mostrado abaixo:

```java

    import android.content.ComponentCallbacks2;
    // Other import statements ...

    public class MainActivity extends AppCompatActivity
        implements ComponentCallbacks2 {

        // Other activity code ...

        /**
         * Release memory when the UI becomes hidden or when system resources become low.
         * @param level the memory-related event that was raised.
         */
        public void onTrimMemory(int level) {

            // Determine which lifecycle or system event was raised.
            switch (level) {

                case ComponentCallbacks2.TRIM_MEMORY_UI_HIDDEN:

                    /*
                       Release any UI objects that currently hold memory.

                       The user interface has moved to the background.
                    */

                    break;

                case ComponentCallbacks2.TRIM_MEMORY_RUNNING_MODERATE:
                case ComponentCallbacks2.TRIM_MEMORY_RUNNING_LOW:
                case ComponentCallbacks2.TRIM_MEMORY_RUNNING_CRITICAL:

                    /*
                       Release any memory that your app doesn't need to run.

                       The device is running low on memory while the app is running.
                       The event raised indicates the severity of the memory-related event.
                       If the event is TRIM_MEMORY_RUNNING_CRITICAL, then the system will
                       begin killing background processes.
                    */

                    break;

                case ComponentCallbacks2.TRIM_MEMORY_BACKGROUND:
                case ComponentCallbacks2.TRIM_MEMORY_MODERATE:
                case ComponentCallbacks2.TRIM_MEMORY_COMPLETE:

                    /*
                       Release as much memory as the process can.

                       The app is on the LRU list and the system is running low on memory.
                       The event raised indicates where the app sits within the LRU list.
                       If the event is TRIM_MEMORY_COMPLETE, the process will be one of
                       the first to be terminated.
                    */

                    break;

                default:
                    /*
                      Release any non-critical data structures.

                      The app received an unrecognized memory level value
                      from the system. Treat this as a generic low-memory message.
                    */
                    break;
            }
        }
    }
    
``` 
O callback `onTrimMemory()` foi adicionado no Android 4.0 (API nível 14). Em versões anteriores, use `onLowMemory()`, que é equivale aproximadamente ao evento `TRIM_MEMORY_COMPLETE`. 

### Verificar quanta memória você deve usar

Para permitir vários processos em execução, o Android define um limite rígido para o tamanho de heap atribuído a cada app. O limite exato para o tamanho de heap varia entre os dispositivos com base na quantidade de RAM disponível no dispositivo. Se seu app atingir a capacidade de heap e tentar alocar mais memória, o sistema gerará um `OutOfMemoryError`. 

Para evitar a falta de memória, você pode consultar o sistema para determinar quanto espaço de heap há disponível no dispositivo atual. Para fazer essa consulta, chame `getMemoryInfo()`. Isso retorna um objeto `ActivityManager.MemoryInfo` que fornece informações sobre o status atual da memória do dispositivo, incluindo memória disponível, memória total e limite de memória, ou seja, o nível de memória em que o sistema começa a encerrar processos. O objeto `ActivityManager.MemoryInfo` também expõe um booleano simples, `lowMemory`, que informa se a memória do dispositivo está acabando. 

O snippet de código a seguir mostra um exemplo de como usar o método `getMemoryInfo()` no aplicativo. 

```java
    public void doSomethingMemoryIntensive() {

        // Before doing something that requires a lot of memory,
        // check to see whether the device is in a low memory state.
        ActivityManager.MemoryInfo memoryInfo = getAvailableMemory();

        if (!memoryInfo.lowMemory) {
            // Do memory intensive work ...
        }
    }

    // Get a MemoryInfo object for the device's current memory status.
    private ActivityManager.MemoryInfo getAvailableMemory() {
        ActivityManager activityManager = (ActivityManager) this.getSystemService(ACTIVITY_SERVICE);
        ActivityManager.MemoryInfo memoryInfo = new ActivityManager.MemoryInfo();
        activityManager.getMemoryInfo(memoryInfo);
        return memoryInfo;
    }
    
```

### Usar mais construções de código com eficiência de memória

Alguns recursos do Android, classes Java e construções de código tendem a usar mais memória que outros. É possível minimizar a quantidade de memória que seu app usa escolhendo alternativas mais eficientes no seu código. 

### Usar os serviços com moderação
Deixar um serviço em execução quando ele não é necessário é um dos piores erros de gerenciamento de memória que um app Android pode cometer. Se seu app precisa que um serviço execute trabalhos em segundo plano, não o mantenha em execução, a menos que ele precise realizar um job. Lembre-se de interromper o serviço quando ele tiver concluído a tarefa. Caso contrário, você pode causar um vazamento de memória sem intenção. 

Quando você inicia um serviço, o sistema prefere sempre manter o processo desse serviço em execução. Esse comportamento torna os processos de serviços muito caros, porque a RAM usada por um serviço permanece indisponível para outros processos. Isso reduz o número de processos em cache que o sistema pode manter no cache de LRU, tornando a alternância de apps menos eficiente. Isso pode até gerar uma sobrecarga no sistema quando a memória está baixa, e o sistema não conseguirá manter processos suficientes para hospedar todos os serviços em execução. 

Em geral, é preciso evitar o uso de serviços persistentes devido às demandas contínuas que elas colocam na memória disponível. Em vez disso, recomendamos que você use uma implementação alternativa, como `JobScheduler`. Para saber mais sobre como usar `JobScheduler` para programar processos em segundo plano, consulte Otimizações em segundo plano. 

Se você precisa usar um serviço, a melhor maneira de limitar a vida útil dele é usando um IntentService, que encerra a si mesmo assim que o processamento do intent que o iniciou é concluído. Para ver mais informações, leia Execução de um serviço em segundo plano. 

### Usar contêineres de dados otimizados

Algumas das classes fornecidas pela linguagem de programação não são otimizadas para uso em dispositivos móveis. Por exemplo, a implementação genérica HashMap pode ser bastante ineficiente para a memória, porque precisa de um objeto de entrada separado para cada mapeamento. 

O framework do Android inclui vários contêineres de dados otimizados, incluindo SparseArray, SparseBooleanArray e LongSparseArray. Por exemplo, as classes SparseArray são mais eficientes porque evitam a necessidade de encaixotar automaticamente a chave e, às vezes, o valor, o que cria mais um ou dois objetos por entrada. 

Se necessário, é possível alternar para matrizes brutas para uma estrutura de dados que seja realmente enxuta. 

### Cuidado com abstrações de código

Os desenvolvedores costumam usar abstrações simplesmente como uma boa prática de programação, porque as abstrações podem melhorar a flexibilidade e a manutenção do código. No entanto, as abstrações têm um custo significativo: geralmente elas exigem que uma quantidade razoável de código a mais seja executado, exigindo mais tempo e RAM para que o código seja mapeado na memória. Então, se suas abstrações não oferecem um benefício significativo, evite-as. 

### Usar protobufs leves para dados serializados
Os buffers de protocolo são um mecanismo extensível que é neutro em relação à linguagem e à plataforma, projetado pelo Google para serializar dados estruturados, de forma semelhante ao XML, mas menor, mais rápido e mais simples. Se você decidir usar protobufs para seus dados, use sempre as opções mais leves no código do cliente. Protobufs regulares geram códigos extremamente detalhados, o que pode causar muitos tipos de problemas no seu app, como maior uso de RAM, aumento significativo no tamanho do APK e execução mais lenta. 

### Evitar rotatividade de memória
Como mencionado antes, os eventos de coleta de lixo normalmente não afetam o desempenho do seu app. No entanto, muitos eventos de coleta de lixo que ocorrem em um curto período podem consumir rapidamente o tempo de frame. Quanto mais tempo o sistema gasta na coleta de lixo, menos ele tem para fazer outras tarefas, como renderização ou streaming de áudio. 

Geralmente, a rotatividade de memória pode causar um grande número de eventos de coleta de lixo. Na prática, a rotatividade de memória descreve o número de objetos temporários alocados que ocorrem em determinado período. 

Por exemplo, você pode alocar vários objetos temporários dentro de um loop for. Ou você pode criar novos objetos Paint ou Bitmap dentro da função onDraw() de uma visualização. Em ambos os casos, o app cria muitos objetos de forma rápida e em volume elevado. Eles podem consumir rapidamente toda a memória disponível na geração jovem, forçando a ocorrência de um evento de coleta de lixo. 

Você precisa encontrar os locais no seu código em que a rotatividade de memória é alta antes de poder corrigi-los. Para isso, use o Memory Profiler no Android Studio.

Depois de identificar as áreas problemáticas no seu código, reduza o número de alocações nas áreas com desempenho crítico. Mova itens para fora dos loops internos ou mova-os para uma estrutura de alocação baseada em Factory. 

### Remover recursos e bibliotecas que consomem muita memória
Alguns recursos e bibliotecas do seu código podem consumir muita memória sem que você saiba. O tamanho total do seu APK, incluindo bibliotecas de terceiros ou recursos incorporados, pode afetar a quantidade de memória que seu app consome. Você pode melhorar o consumo de memória do seu app removendo componentes, recursos ou bibliotecas redundantes, desnecessários ou pesados do seu código. 

### Reduzir o tamanho geral do APK

Você pode diminuir significativamente o uso de memória do seu app reduzindo o tamanho geral dele. Tamanho de bitmap, recursos, frames de animação e bibliotecas de terceiros podem contribuir para o tamanho do seu APK. O Android Studio e o SDK Android oferecem várias ferramentas para ajudar a reduzir o tamanho dos seus recursos e dependências externas. Essas ferramentas são compatíveis com métodos modernos de redução de código, como a Compilação R8. O Android Studio 3.3 e versões anteriores usam o ProGuard em vez da Compilação R8. 

### Usar a Dagger 2 para injeção de dependência
 Frameworks de injeção de dependência podem simplificar o código que você escreve e fornecer um ambiente adaptável, útil para testes e outras alterações de configuração.

Se você pretende usar um framework de injeção de dependência no seu app, recomendamos o uso da Dagger 2. A Dagger não usa reflexão para verificar o código do seu app. A implementação estática em tempo de compilação da Dagger significa que ela pode ser usada em apps Android sem custo de tempo de execução ou uso de memória desnecessários.

Outros frameworks de injeção de dependência que usam reflexão tendem a inicializar processos verificando o código em busca de anotações. Esse processo pode exigir significativamente mais ciclos de CPU e RAM, além de causar um atraso considerável quando o app é iniciado. 

### Cuidado ao usar bibliotecas externas

 O código da biblioteca externa geralmente não é escrito para ambientes móveis e pode ser ineficiente quando usado para trabalho em um cliente móvel. Quando você decide usar uma biblioteca externa, pode ser necessário otimizar essa biblioteca para dispositivos móveis. Planeje esse trabalho com antecedência e analise a biblioteca em termos de tamanho de código e área ocupada de RAM antes de decidir usá-la.

Até mesmo algumas bibliotecas otimizadas para dispositivos móveis podem causar problemas devido a implementações diferentes. Por exemplo, uma biblioteca pode usar protobufs leves enquanto outra usa micro protobufs, o que resulta em duas implementações de buffers de protocolo diferentes no seu app. Isso pode acontecer com várias implementações de registro, análise, frameworks de carregamento de imagens, armazenamento em cache e muitos outros fatores inesperados.

O ProGuard pode ajudar a remover APIs e recursos com as sinalizações corretas, mas não pode remover as grandes dependências internas de uma biblioteca. Os recursos que você quer nessas bibliotecas podem exigir dependências de nível inferior. Isso se torna especialmente problemático quando você usa uma subclasse Activity de uma biblioteca (que tende a ter grandes extensões de dependências), quando as bibliotecas usam reflexão (o que é comum e significa que você precisa gastar muito tempo ajustando manualmente o ProGuard para fazê-las funcionar) e assim por diante.

Evite também usar uma biblioteca compartilhada para apenas um ou dois recursos dentre dezenas. Não é recomendável importar uma grande quantidade de código e sobrecarga que você nem usa. Quando você considerar o uso de uma biblioteca, procure uma implementação que corresponda ao que você precisa. Caso contrário, você pode criar a própria implementação. 
