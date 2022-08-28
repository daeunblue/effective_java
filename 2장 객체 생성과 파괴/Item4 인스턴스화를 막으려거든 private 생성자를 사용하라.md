#### **정적 메서드와 정적 필드만을 담은 클래스**

- 기본 타입 값이나 배열 관련 메서드들
- 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드
- final 클래스와 관련한 메서드들

이러한 정적 멤버를 담은 클래스는 유틸리티 클래스라고 불리는데, 전부 static으로 선언되어 있기 때문에 인스턴스화가 필요하지 않게 된다.

#### **예시 코드**

```
public class StringUtil {
  public staic List<String> splitStr(String str) {
    return Arrays.stream(categories.split(",")).collect(Collectors.toList());
  }
}

다른 클래스에서 StringUtil.splitStr(); 을 이용하여 사용!
```

---

### **인스턴스화를 막는 방법**

추상 클래스로 만들게 되면 하위 클래스에서 상속받게 될 경우 인스턴스화가 되기 때문에 소용이 없다.

#### private 생성자를 추가하자

#### **예시 코드**

```
public class StringUtil {

  // 기본 생성자가 만들어지는 것을 막는다.(인스턴스화 방지용)
  private StringUtil() {
    throw new AssertionError();
  }

  public staic List<String> splitStr(String str) {
    return Arrays.stream(categories.split(",")).collect(Collectors.toList());
  }
}
```

- 인스턴스화가 되는 것을 막아준다.
- 상속이 불가능하다.
