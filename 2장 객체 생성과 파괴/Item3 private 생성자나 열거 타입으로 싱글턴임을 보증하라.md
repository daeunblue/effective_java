싱글턴(Singleton)

싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다. 전역 변수이기 때문에 프로그램 내부에서 이

객체를 공유하며 사용한다.

#### **코드 3-1 public static final 필드 방식의 싱글턴**

```
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }

    public void leaveTheBuilding() { ...}
}
```

\- private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화 할 때 딱 한 번만 호출

\- public이나 protected(같은 패키지 or 상속 받을 시 접근 가능) 생성자가 없으므로 Elvis 클래스가 초기화 될 때 만들어진

인스턴스가 전체 시스템에서 하나 뿐임을 보장. 클라이언트는 손 쓸 방법이 없다.

\- 단 예외가 한 가지 있는데, 권한이 있는 클라이언트는 리플렉션 API 인 AccessibleObject.setAccessible을 사용해

private 생성자를 호출 할 수 있다. 이런 공격을 방어하기 위해 생성자를 수정해서 두번째 객체가 생성되려 할 때

예외를 던지게 하면 된다.

ex)  클라이언트는 리플렉션 API 인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출 할 수 있다.

```
Constructor<Elvis> constructor = (Constructor<Elvis>) elvis2.getClass().getDeclaredConstructor();
constructor.setAccessible(true);

Elvis elvis3 = constructor.newInstance();
assertNotSame(elvis2, elvis3); // FAIL
```

ex) 생성자를 수정해서 두 번째 객체가 생성되려 할 때 예외 던지는 방법

```
private Elvis() {
	if(INSTANCE != null){
		throw new RuntimeException("생성자를 호출할 수 없습니다!");
	}
}
```

public static final 의 장점

- 해당 클래스가 싱글턴임이 API에 명확하게 드러난다.

- 간결함

#### **코드 3-2 정적 팩터리 방식의 싱글턴**

```
public class Elvis {
    private static final Elvis INSTANCE= new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() { ... }
}
```

\- Elvis.getInstance()는 항상 같은 INSTANCE를 반환

\- 정적 팩터리 방식의 장점

- API 변경 없이 싱글턴이 아니게 변경할 수 있다. 스레드 별로 다른 인스턴스를 넘겨주게 변경 가능.

- 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.

- 정적 팩토리의 메서드 참조를 공급자를 사용할 수 있다.

- Elvis::getInstance를 Supplier로 사용하는 방식(아이템 43,44 참고)

\=> 이런 강점들이 필요하지 않다면 public 필드 방식이 좋다.

#### **코드 3-3 열거 타입 방식의 싱글턴 - 바람직한 방법**

```
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() { ... }
}
```

**\- 대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.**

\- public 필드 방식과 비슷하지만, 더 간결하고, 추가 노력 없이 직렬화가 가능하며, 아주 복잡한 직렬화 상황이나 리플랙션

공격에도 제 2의 인스턴스가 생기는 일을 완벽하게 막아준다.

\- 단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야하는 경우에는 사용할 수 없다.

p.s 위 예시코드들은 이펙티브 자바를 참고했습니다.
