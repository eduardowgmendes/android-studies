## Padrão de Projeto Observer

### Objetivo 
Definir um mecanismo eficiente para reagir às alterações realizadas em determinados objetos. 

Considere um supermercado em que os caixas possuem uma fila única. Cada caixa é identificado por um número, que é exibido em uma placa à sua frente. Esse número fica visível à todos os clientes da fila. 

Também visível aos clientes da fila, existe um painel que exibe o número de identificação do próximo caixa disponível (se houver algum).

O painel precisa saber sobre as mudanças no estado (livre ou ocupado) de cada caixa para atualizar o seu mostrador. 

Uma possibilidade é fazer o painel consultar, periodicamente, o estado de cada caixa. Contudo, essa pode ser uma abordagem ineficiente, principalmente quando o estado dos caixas não é alterado frequentemente. 

Uma outra possibilidade é fazer com que cada caixa avisasse ao painel sobre uma mudança em seu estado. 

Nessa abordagem no momento em que um caixa fica disponível ou ocupado, ele poderia passar o seu número de identificação para o painel, para informá-lo sobre a sua mudança de estado. O painel, por sua vez, atualizaria o número exibido quando necessário. 

A ideia fundamental do padrão Observer é atribuir aos objetos que tem seus estados alterados a tarefa de notificar os objetos interessados nessa mudanças. Em nosso exmeplo, os caixas notificam o painel sobre as alterações em seus estados.

### Exemplo prático

Estamos desenvolvendo um sistema para o mercado financeiro. Esse sistema deve manter todos
os interessados em uma determinada ação informados do valor da mesma. Toda alteração no valor
de uma ação deve ser informada aos seus respectivos interessados.

Podemos ter vários tipos de interessados. Por exemplo, uma pessoa física, uma empresa, um órgão público, entre outros. Para manter uma padronização na modelagem, definiremos uma interface
para os interessados.

```java
public interface AcaoObserver(){
	void notificaAlteracao(Acao acao);
}
```

Agora, podemos implementar os interessados de maneira concreta.

```java
public class Corretora implements AcaoObserver{
	public void notificaAlteracao(){
		// implementacao	
	}
}
```

Por outro lado, a classe Acao deve aceitar interessados (observadores) e avisá-los quando o seu
valor for alterado.

```java
public class Acao {
	private double valor ;
	private Set <AcaoObserver> interessados = new HashSet <AcaoObserver>();
		
	public void registraInteressado (AcaoObserver interessado) {
		this.interessados.add(interessado);
	}
	public void cancelaInteresse ( AcaoObserver interessado ) {
		this.interessados.remove(interessado);
	}
	public double getValor() {
		return this.valor;
	}
	
	public void setValor ( double valor ) {
		this . valor = valor ;
	
		for(AcaoObserver interessado : this.interessados ) {
			interessado.notificaAlteracao(this) ;
		}
	}
}
```



