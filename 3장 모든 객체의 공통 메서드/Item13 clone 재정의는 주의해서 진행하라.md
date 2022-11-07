> clone 재정의는 주의해서 진행하라

## Clonable의 역할

-   복제해도 되는 클래스임을 나타내는 [믹스인 인터페이스](https://jake-seo-dev.tistory.com/30)이다.
-   Object 클래스에 protected clone()이라는 메서드가 있다.
-   Cloneable 인터페이스는 clone() 메서드의 동작방식을 결정한다.
-   Cloneable을 구현하지 않은 인스턴스에서 clone()을 호출하면 CloneNotSupportedException을 던진다.

### clone() 사용해보기

```
static class Entry {
    String key;
    String value;

    public Entry(String key, String value) {
        this.key = key;
        this.value = value;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

**결과**

```
java.lang.CloneNotSupportedException: item13.Item13Test$Entry

    at java.base/java.lang.Object.clone(Native Method)
    at item13.Item13Test$Entry.clone(Item13Test.java:17)
    at item13.Item13Test.entryCloneTest(Item13Test.java:24)
```

-   에러가 뜨는 이유는 뭘까? 클래스에서 Cloneable 인터페이스를 상속받지 않았기 때문이다.

### clone() 동작하게 만들기
```
public class Item13Test {
    static class Entry implements Cloneable {
        String key;
        String value;

        public Entry(String key, String value) {
            this.key = key;
            this.value = value;
        }

        @Override
        protected Object clone() throws CloneNotSupportedException {
            return super.clone();
        }
    }

    @Test
    public void entryCloneTest() throws CloneNotSupportedException {
        Entry entry = new Entry("key", "value");
        System.out.println("entry.key = " + entry.key);
        System.out.println("entry.value = " + entry.value);

        Entry clonedEntry = (Entry) entry.clone();
        System.out.println("clonedEntry.key = " + clonedEntry.key);
        System.out.println("clonedEntry.value = " + clonedEntry.value);
    }
}
```

-   Cloneable을 상속받았다.
-   clone() 메서드의 내용은 그냥 상위 클래스의 clone()을 가져다 써도 된다.

## Cloneable의 문제점

-   일반적인 인터페이스의 동작방식과 다르게 상위 Object 클래스에 protected 접근자로 된 clone() 메서드가 존재하고, 그걸 오버라이드 해야 한다. (믹스인으로 의도해서 만들었는데, 믹스인이라고 말하기 뭔가 애매하다.)
-   Cloneable만 사용하면 당연히 복제가 이뤄질 줄 알았는데 생각보다 복잡한 구조를 이해하고 있어야 한다.
-   자바의 기본 의도와 다르게 생성자를 호출하지 않고 객체를 생성할 수 있게 되어버린다.
-   clone() 메서드의 일반 규약은 약간 허술하다.

## clone() 메서드 구현 방법

### clone() 메서드의 일반 규약

-   x.clone() != x 식은 참이어야 한다.
    -   복사된 객체가 원본이랑 같은 주소를 가지면 안된다는 뜻이다.
-   x.clone().getClass() == x.clone().getClass() 식도 참이어야 한다.
    -   복사된 객체가 같은 클래스여야 한다는 뜻이다.
-   x.clone().equals()는 참이어야 하지만, 필수는 아니다.
    -   복사된 객체가 논리적 동치는 일치해야 한다는 뜻이다. (필수는 아니다.)

### clone() 메서드는 super.clone()을 사용하는 편이 좋다.

-   super.clone()을 사용하지 않으면, 상속한 하위 클래스에서 super.clone()을 호출했을 때 엉뚱한 결과가 나올 수 있다.
    -   단, final 클래스라면, 이런 걱정을 할 필요 없다.

### 구현 순서

-   super.clone()을 호출한다.
    -   정의된 모든 필드는 원본 필드와 똑같은 값을 갖게 된다.
        -   모든 필드가 primitive 타입이거나 final 이라면, 이 상태로 끝이다.
            -   사실 final은 쓸데 없는 복사를 지양하기 때문에 clone()이 필요 없다.

```
@Override
public PhoneNumber clone() {
  try {
    return (PhoneNumber) super.clone();
  } catch (CloneNotSupportedException e) {
    throw new AssertionError(); // 일어날 수 없는 일이다.
  }
}
```

-   PhoneNumber 타입으로 형변환하여 반환하도록 해서 편의성을 증대시켰다.
-   try-catch 블록으로 감싼 이유는 Object.clone() 메서드가 CloneNotSupportedException을 던지기 때문이다.
    -   우린 PhoneNumber 클래스가 Cloneable 인터페이스를 상속받는 것을 보고 clone()을 지원함을 알 수 있다.
        -   이 거추장스러운 코드는 사실 CloneNotSupportedException이 unchecked 예외여야 했다는 것을 알려준다.
            -   과도한 검사 예외는 API를 사용하기 불편하게 만든다.

## 가변객체를 참조할 때의 clone() 메서드 구현

```
static class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if(size == 0) {
            throw new EmptyStackException();
        }

        Object result = elements[--size];
        elements[size] = null;
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

> 가변객체란, instance 생성 이후에도 내부 상태 변경이 가능한 객체를 말한다.

-   이 클래스를 그대로 복제하면, primitive 타입의 값은 올바르게 복제되지만, 복제된 Stack의 elements는 복제 전의 Stack과 같은 배열을 가리키게 될 것이다.
    -   두 스택에 같은 elements가 들어있고, 하나를 바꾸면 다른 하나도 연동된다는 뜻이다. 이건 우리가 원한 clone()이 아니다.
-   Stack 클래스의 유일한 생성자를 이용하면 이런 문제는 없을 것이다. 그러나 값이 복사되지 않는 문제가 있다.

**가변 객체가 있는 클래스의 clone() 문제 해결하기**

```
@Override
protected Object clone() throws CloneNotSupportedException {
    Stack clonedStack = (Stack) super.clone();
    clonedStack.elements = this.elements.clone();

    return clonedStack;
}
```

-   가변 객체인 elements 배열에 clone() 메서드를 이용해 따로 복사해주면 된다.

> 배열 내부 메소드 clone()은 손쉽게 배열을 복사할 수 있다는 점에서 매우 유용하다. 따라서 배열 복제에 권장된다.

**테스트 코드 작성해보기**

```
@Test
public void stackClone() throws CloneNotSupportedException {
    Stack stack1 = new Stack();
    stack1.push("value1");
    stack1.push("value2");

    Stack stack2 = stack1.clone();
    stack2.push("value3");

    System.out.println("stack1 = " + stack1);
    System.out.println("stack2 = " + stack2);
}
```

**테스트 결과**

```
stack1 = Stack{elements=[value1, value2, null, null, null, null, null, null, null, null, null, null, null, null, null, null], size=2}
stack2 = Stack{elements=[value1, value2, value3, null, null, null, null, null, null, null, null, null, null, null, null, null], size=3}
```

-   의도했던대로 값이 모두 복사됐지만, 서로 같은 elements 배열을 참조하지 않는다.
-   한편 elements가 final로 선언되어 있었다면, 앞의 방식은 작동하지 않는다.
    -   **가변 객체를 참조하는 필드는 final로 선언하라**는 일반 용법과 충돌한다.

## 가변객체 내부에 가변객체가 있을 때의 clone() 메서드
```
static class HashTable implements Cloneable {
    private Entry[] buckets;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }

    @Override
    public HashTable clone() throws CloneNotSupportedException {
        HashTable result = (HashTable) super.clone();
        result.buckets = buckets.clone();
        return result;
    }
}
```

-   위 HashTable 클래스의 clone() 메서드는 적절하게 구현되었을까?
    -   아니다. clone() 메서드를 사용하면, 복제된 HashTable은 자신만의 buckets는 가지겠지만, buckets 내부에 있는 객체들은 여전히 복제되기 이전의 객체들을 가리키고 있을 것이다.

### 가변 객체 내부에 또 다른 가변 객체가 있을 때 clone() 메서드

```
static class HashTable implements Cloneable {
    private Entry[] buckets;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        Entry deepCopy() {
            Entry result = new Entry(key, value, next);
            for(Entry p = result; p.next != null; p = p.next) {
                p.next = new Entry(p.next.key, p.next.value, p.next.next);
            }
            return result;
        }
    }

    @Override
    public HashTable clone() throws CloneNotSupportedException {
        HashTable result = (HashTable) super.clone();
        result.buckets = new Entry[buckets.length];

        for (int i=0; i<buckets.length; i++) {
            if(buckets[i] != null) {
                result.buckets[i] = buckets[i].deepCopy();
            }
        }

        return result;
    }
}
```

-   HashTable의 경우, 같은 HashCode를 가지고 실제 논리적 동치가 아닌 key값이 들어왔을 때, 링크드리스트처럼 next로 연결된다.
    -   이 경우 계속 객체로 연결되어 있는데, 이 객체를 연쇄적으로 계속 같은 값을 지닌 새로운 객체로 복사해주어야 한다.
    -   그 부분이 위쪽에서 deepCopy()를 반복하는 부분이다.
        -   deepCopy() 메서드 내부에서는 연결된 모든 엔트리를 내용이 같은 새로운 객체로 복사하고 있다.

> 사실 put() 메서드를 이용해서 Entry를 넣어주어도 된다. 그러나, 저수준의 방법을 활용할 때보다는 동작속도가 조금 느릴 것이다.  
> 만일, 생성자에서 HashTable을 받고 put() 메서드로 복사한다면, put은 final이거나 private이어야 한다.  
> 생성자에는 재정의될 수 있는 메서드를 호출하면 안되기 때문이다.

## clone() 메서드 주의사항

-   Object.clone()은 동기화를 신경쓰지 않은 메서드이다.
    -   동시성 문제가 발생할 수 있다.
-   만일 clone()을 막고 싶다면 clone() 메서드를 재정의하여, CloneNotSupportedException()을 던지도록 하자.
-   기본 타입이나 불변 객체 참조만 가지면 아무것도 수정할 필요 없으나 일련번호 혹은 고유 ID와 같은 값을 가지고 있다면, 비록 불변일지라도 새롭게 수정해주어야 할 것이다.

## 복사 생성자와 복사 팩터리로 clone() 구현하기

> 복사 생성자란 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다.

```
/**
 * Constructs a new {@code HashMap} with the same mappings as the
 * specified {@code Map}.  The {@code HashMap} is created with
 * default load factor (0.75) and an initial capacity sufficient to
 * hold the mappings in the specified {@code Map}.
 *
 * @param   m the map whose mappings are to be placed in this map
 * @throws  NullPointerException if the specified map is null
 */
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

-   위는 HashMap에서 제공하는 복사 생성자로 볼 수 있다.
-   사실 이 방식이 clone()보다 나은 면이 많다.
    -   생성자를 쓰지 않는 생성방식을 쓰지 않는다.
    -   정상적 final 필드 용법과 충돌하지 않는다.
    -   불필요한 검사 예외를 던지지 않는다.
    -   형변환도 필요 없다.
    -   '인터페이스' 타입의 인스턴스도 인수로 받을 수 있다.

> 더 정확한 이름은 변환 생성자(conversion constructor)와 변환 팩터리(conversion factory)이다.

## 정리

-   인터페이스를 만들 때는 절대 Cloneable을 확장해선 안된다.
    -   Cloneable은 클래스의 믹스인(사용) 의도로 만들어진 것이다.
-   final 클래스라면 Cloneable을 구현해도 위험은 크지 않지만, 성능 최적화 관점에서 검토 후에 드물게 허용해야 한다.
-   **복제 기능은 생성자와 팩터리를 이용하는 것이 최고이다.**
    -   **단 한가지 예외는 배열을 복사할 때이다.**
