# [Gof 디자인 패턴] Strategy(전략) 패턴

프로그래밍을 하다보면 비슷한 동작이지만 구현 방법(알고리즘)이 다른 경우를 많이 겪습니다. 예를 들어 외부 메일 가져오기 기능을 구현하려고 합니다. 네이버 메일, 다음 메일, 구글 메일 등을 가져온다고 가정하면 메일을 가져온다에서는 동일한 액션이지만 내부 구현을 살펴보면 호출 API 정보, 필요 파라미터 명 등이 조금씩 다를 수 있습니다. 

처음에 네이버 메일와 구글 메일 가져오기만 지원한다면 if문 분기를 통해 다르게 처리할 수 도 있겠지만, 나중에 다음 메일과 네이트 메일 등도 추가적으로 지원한다고 하면 분기가 계속 증가할 것 입니다. 이러면 SOLID 설계 원칙의 개방-폐쇄 원칙(OCP)에 위반하는 코드가 생성될 확률이 커집니다. **Strategy(전략) 패턴을 사용하면 이와 같이 같은 문제를 다른 방법으로도 쉽게 변경하면서 해결할 수 있습니다.**

위에서 언급한 외부 메일 가져오기 기능으로 예제를 살펴보겠습니다. 예제 프로그램의 클래스 다이어그램은 다음과 같습니다.

![example class diagram](http://img1.daumcdn.net/thumb/R1920x0/?fname=http%3A%2F%2Fcfile23.uf.tistory.com%2Fimage%2F991B3E3F5ACEF96A2771C1)

*MailImporter.java*

```java
public class MailImporter {
	private MailImportStrategy strategy;

	public MailImporter(MailImportStrategy strategy) {
		this.strategy = strategy;
	}

	public void changeStrategy(MailImportStrategy strategy) {
		this.strategy = strategy;
	}

	public void imports(){
		strategy.execute();
	}
}
```

`MailImporter` 클래스는 메일 가져오기 기능을 수행할 클래스입니다. `imports()` 메서드에서 해당 서비스에 맞는 가져오기 기능(전략)을 구현한 `MailImportStrategy` 클래스에 **위임**하고 있습니다.

*MailImportStrategy.java*

```java
public interface MailImportStrategy {
	void execute();
}
```

`MailImportStrategy` 인터페이스는 해당 서비스에 맞는 외부 메일을 가져오는 기능(전략)을 위한 인터페이스입니다. 이 인터페이스를 구현하는 네이버, 구글, 다음 메일 가져오기 클래스에서 `execute()` 메서드를 각각의 전략(각 시스템이 요구하는 정보)에 맞게 구현할 것 입니다.

*GoogleMailImportStrategy.java*

```java
public class GoogleMailImportStrategy implements MailImportStrategy {
	@Override
	public void execute() {
		// Gmail의 필요정보에 맞게 가져오기 수행
        // ...
	}
}
```

*NaverMailImportStrategy.java*

```java
public class NaverMailImportStrategy implements MailImportStrategy {
	@Override
	public void execute() {
        // 네이버 메일의 필요정보에 맞게 가져오기 수행
		// ...
	}
}
```

*NateMailImportStrategy.java*

```java
public class NateMailImportStrategy implements MailImportStrategy {
	@Override
	public void execute() {
        // 네이트 메일의 필요정보에 맞게 가져오기 수행
		// ...
	}
}
```

이제 main 클래스에서 각 서비스 별로 메일 가져오기를 수행하여 봅시다.

```java
public class App {
    public static void main(String[] args) {
        // 구글 메일 가져오기 전략
        MailImporter mailImporter = new MailImporter(
            new GoogleMailImportStrategy()
        );

        // 메일 가져오기 수행
        mailImporter.imports();

        // 네이버 메일 가져오기로 전략 변경
        mailImporter.changeStrategy(
            new NaverMailImportStrategy()
        );

        mailImporter.imports();

        // 네이트 메일 가져오기로 전략 변경
        mailImporter.changeStrategy(
            new NateMailImportStrategy()
        );

        mailImporter.imports();
    }
}
```

이와 같이 Strategy 패턴을 이용하면 프로그램을 복잡하게 만드는 것처럼 보이지만 실제로는 그렇지 않습니다. 위임이라는 느슨한 연결을 사용하고 있어 전략을 용이하게 교환할 수 있습니다.

예를들어 현재 네이버, 구글, 네이트 메일 가져오기 기능만을 제공하고 있는데 다음 메일도 추가로 지원하는 것으로 스펙이 변경되었다고 가정합시다. 분기를 통해 서비스별 메일 가져오기를 구현하였다면 기존 가져오기 기능에 수정이 불가피하지만, 위의 Strategy 패턴에서는 `MailImportStrategy` 인터페이스를 상속받아 `DaumMailImportStrategy` 만 구현하면 됩니다. 기존 코드의 수정을 하여 사이드이펙트가 생길까 염려할 필요가 없게되는 것이죠. 즉, **확장에는 열려있고 변경에는 닫혀있는 개방-폐쇄 원칙(OCP)을 수용할 수 있게 됩니다.**

Strategy 패턴의 등장인물은 다음과 같습니다.

*Strategy*

전략을 이용하기 위한 인터페이스(API)를 결정합니다. 예제에서 `MailImportStrategy` 인터페이스가 이 역할을 합니다.

*ConcreteStrategy*

Strategy 인터페이스를 실제로 구현한 클래스입니다. 예제에서 `NaverMailImportStrategy`, `GoogleMailImportStrategy`, `NateMailImportStrategy` 클래스가 이 역할을 합니다.

*Context*

Strategy를 이용하는 역할을 합니다. 예제에서 `MailImporter` 클래스가 이 역할을 합니다.

![strategy class diagram](http://img1.daumcdn.net/thumb/R1920x0/?fname=http%3A%2F%2Fcfile23.uf.tistory.com%2Fimage%2F991D68345ACEF98F37FC74)
