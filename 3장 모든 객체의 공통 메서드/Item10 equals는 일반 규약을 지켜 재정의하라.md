# equals는 일반 규약을 지켜 재정의하라

클래스를 재정의 하지 않게 되면 인스턴스는 오직 자기 자신과만 같게 된다.

> 다음 상황에 해당한다면 재정의하지 않는 것이 최선이다.

1. 각 인스턴스가 본질적으로 고유하다.

2. 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다.

3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.

4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.

## equals를 재정의해야 할 때?!

상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때
주로 값 클래스들이 여기 해당한다.

## equals를 재정의할 때 따라야할 일반 규약

1. 반사성(reflexivity) : null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true이다.

   ➡️ 객체는 자기 자신과 같아야 한다.

2. 대칭성(symmetry) : null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true이면 y.equals(x)도 true이다.

   ➡️ 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻

3. 추이성(transitivity) : null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고, y.equals(z)도 true면, x.equals(z)도 true다.

   ➡️ 첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야한다.

   ```
   구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 X

   따라서 상속 대신 컴포지션을 사용하라(아이템 18)
   ```

4. 일관성(consistency) : null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.

   ➡️ 두 객체가 같다면 영원히 같아야 한다

   ```
   equals는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야 한다.
   ```

5. null-아님 : null이 아닌 모든 참조 값 x에 대해 x.equals(null)은 false이다.

   ➡️ 모든 객체가 null과 같지 않아야 한다.

   ```
   instanceof 연산자로 입력 매개변수가 올바른 타입인지 검사해야 한다.
   instanceof는 피연산자가 null이면 false를 반환하기 때문에 따로 명시적인 검사를 할 필요가 없다.
   ```

## equals 메서드 구현 방법

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 '핵심'필드들이 모두 일치하는지 하나씩 검사한다.

```
- float와 double을 제외한 기본 타입 필드는 == 연산자로 비교
- 참조 타입 필드는 각각의 equals 메서드로 비교
- float, double 필드는 Float.compare(), Double.compare()로 비교 -> 부동소수 값 등을 다뤄야 함
- 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교
```

```
@Override
public boolean equals(Object o) {
   if (this == o)
      return true;
   if(!(o instanceof MyType))
      return false;
   MyType myType = (MyType) o;
   return myType.age == age && myType.gender == gender;
}
```

### 주의사항

- equals를 재정의할 땐 hashCode도 반드시 재정의하자(아이템 11)
- 너무 복잡하게 해결하려 들지 말자.
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자. (입력 타입은 반드시 Object)
- 단위 테스트가 귀찮다면 구글의 AutoValue 프레임워크를 사용하자
