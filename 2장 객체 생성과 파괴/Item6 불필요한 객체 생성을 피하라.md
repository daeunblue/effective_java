#### **기존에 생성된 객체를 새로 생성할 때 재사용하자**

```
// 1번
String s = new String("hello");

// 2번
String s = "hello";
```

두 개의 String은 같은 문자열을 사용하지만 1번의 경우에는 새로운 인스턴스를 생성하고, 2번의 경우에는 String Pool에서 인스턴스를 참조하여 재사용한다.

- 만약 `s += " java";` 라는 코드가 실행된다면?
  - "hello java"라는 인스턴스를 새로 생성한 뒤 String Pool에서 이 인스턴스를 참조한다.

```
boolean b1 = new Boolean(true);
boolean b2 = Boolean.valueOf(true);

-------

int i1 = new Integer(10);
int i2 = Integer.valueOf(10);
```

#### **비싼 객체는 캐싱하여 재사용하자**

> 코드 6-1

```
static boolean isRomanNumeral(String s) {
  return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

isRomanNumeral을 반복해서 사용하는 경우에는 matches 내부에서 Pattern 인스턴스를 초기화하고 버리는 과정을 반복하기 때문에 비용이 많이 들게 된다.

따라서 `Pattern.compile()`을 초기화 해놓고(캐싱) 재사용하는 것이 훨씬 유리하다. (지연 초기화를 굳이 쓸 필요는 없다.)

비슷한 비싼 객체로는 데이터베이스를 연결하는 Pool이 있다.

#### **의도치 않은 오토박싱을 줄이자**

```
Long sum = 0L;
for(long i = 10; i < 20; i++) {
  sum += i;
}
```

sum에 long 타입 i가 더해질 때마다 Long 타입으로 오토박싱이된다.
