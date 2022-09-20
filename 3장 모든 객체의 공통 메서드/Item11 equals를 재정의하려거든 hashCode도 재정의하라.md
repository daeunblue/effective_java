# equals를 재정의하려거든 hashCode도 재정의하라

equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.
그렇지 않으면 hashCode의 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap, HashSet 등의 원소로 사용할 때 문제를 일으킨다.

```
<<규약>>
- equals 비교에 사용되는 정보가 변경되지 않았다면, hashCode 메서드는 몇 번을 호출해도 같은 값을 반환해야 한다.
- equals가 두 객체를 같다고 판단했다면, hashCode는 똑같은 값을 반환해야 한다.
- equals가 두 객체를 다르다고 판단했더라도, hashCode가 다른 값을 반환할 필요는 없다. 하지만 다른 객체는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.
```

## equals가 두 객체를 같다고 판단했다면, hashCode는 똑같은 값을 반환해야 한다.

HashMap, HashSet 등은 hashCode가 다르면 동치성 비교를 시도조차 하지 않도록 최적화되어 있다.

## hashCode를 작성하는 요령

1. int 변수 result를 선언한 후 값 c로 초기화한다. c는 해당 객체의 첫번째 핵심 필드를 2.a 방식으로 계산한 해시코드다.
2. 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.

   a. 해당 필드의 해시코드 c를 계산한다.

   - 기본 타입 필드 : Type.hashCode(f)를 수행
   - 참조 타입 필드 : hashCode를 재귀적으로 호출 또는 표준형 hashCode를 만들어 호출
   - 배열 필드 : 핵심 원소들의 해시코드를 계산한 다음, 2.b 방식으로 갱신

   b. 단계 2.a에서 계산한 해시코드 c로 result를 갱신한다.

   > result = 31 \* result + c;

3. result를 반환한다.

- equals 비교에 사용되지 않는 필드는 `반드시` 제외해야 한다.

## Objects.hash()

임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드.
앞의 요령대로 구현한 코드와 비슷한 수준의 hashCode 함수를 `단 한줄로` 작성할 수 있다.

단점 : 입력 인수를 담기 위핸 배열 생성, 박싱과 언박싱 등으로 속도가 느리다.

## 클래스가 불변이고 해시코드를 계산하는 비용 ⬆️

매번 새로 계산하지않고 `캐싱하는 방식`을 고려한다.

```
private int hashCode;

@Override
public int hashCode() {
    int result = hashCode;
    if(result == 0) {
      ...
    }
    return result;
}
```

### 주의사항

- 해시코드를 계산할 때 핵심 필드를 생략해서는 안된다.
- hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자.
