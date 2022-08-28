#### 하나 이상의 자원에 의존할 경우 잘못 구현한 방법

> 코드 5-1 : 정적 유틸리티 사용

```
public class SpellChecker {
  private static final Lexicon dictionary = A_dictionary;

  private SpellChecker() {} // 객체 생성 방지

  public static boolean isValid(String word) { ... }
  public static List<String> suggestions(String typo) { ... }
}
```

> 코드 5-2 : 싱글턴을 사용

```
public class SpellChecker {
  private final Lexicon dictionary = A_dictionary;

  private SpellChecker() {} // 객체 생성 방지
  public static SpellChecker INSTANCE = new SpellChecker(...);

  public boolean isValid(String word) { ... }
  public List<String> suggestions(String typo) { ... }
}
```

### **유연하지 않고 테스트하기 어려운 이유**

만약 이 클래스를 사용할 때 다른 사전으로 변경해야 하는 경우에 `Lexicon dictionary = A_dictionary;` 부분을 직접 수정해야 하기 때문에 유연하지 않다.

또한 테스트 시 다른 사전을 테스트할 경우에도 매번 코드를 수정해야 되는 불편함을 가지고 있다.

---

### 사용하는 자원(dictionary)에 따라 동작이 다라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.

## 1. 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식을 이용하자.

> 코드 5-3

```
public class SpellChecker {
  private final Lexicon dictionary;

  public SpellChecker(Lexicon dictionary) {
    this.dictionary = Objects.requireNonNull(dictionary);
  }

  public boolean isValid(String word) { ... }
  public List<String> suggestions(String typo) { ... }
}
```

- final을 통해 Lexicon 타입을 보장하고, 여러 자원을 주입만 하면 된다!!

## 2. 인스턴스를 생성할 때 생성자에 자원 팩터리를 넘겨주는 방식을 이용하자.

```
public class SpellChecker {
  private final Lexicon dictionary;

  public SpellChecker(Supplier<? extends Lexicon> dictFactory) {
    this.dictionary = dictFactory.get();
  }

  public boolean isValid(String word) { ... }
  public List<String> suggestions(String typo) { ... }
}
```

- 팩터리 메서드는 지정된 타입의 인스턴스를 **반복해서** 만들어주기 때문에 사실 위의 코드가 완벽한 예시가 되지 않는다. get()을 통해서 Lexicon 타입의 인스턴스를 꺼내올 수 있다.
