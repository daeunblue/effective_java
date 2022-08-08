클라이언트가 객체를 생성 할 때 일반적으로 사용하는 것은 'public 생성자' 입니다. 하지만 '생성자'를 사용하는 것 말고

'정적 팩토리 메서드(static factory method)'를 사용해서 만들 수 있다.

예1) public 생성자'를 활용한 객체 생성

```
public class Player {
    protected int height, weight, goal, assist ;

    // public 생성자
    public Player(int height, int weight, int goal, int assist) {
        this.height = height;
        this.weight = weight;
        this.goal = goal;
        this.assist = assist;
    }
}
```

```
public class Striker extends Player {
    // public 생성자
    public Striker() {
        super(185, 75, 20, 20);
    }
}
```

```
public class midfielder extends Player {
    // public 생성자
    public Midfielder() {
        super(175, 72, 8, 15);
    }
}
```

예2) 정적 팩토리 메서드를 활용한 객체 생성

```
public class Player {

    private int height, weight, goal, assist;

    private Character(int height, int weight, int goal, int assist) {
        this.height = height;
        this.weight = weight;
        this.goal = goal;
        this.assist = assist;
    }

    // '스트라이커'를 생성하는 정적 팩토리 메소드
    public static Player striker() { //
        return new Player(180, 75, 20, 3);     // 인스턴스를 생성 후 반환
    }

    // '미드필더'를 생성하는 정적 팩토리 메소드
    public static Player midfielder() {
        return new Player(175, 70, 8, 15);    // 인스턴스를 생성 후 반환
    }
}
```

위와 같이 용어를 보고 의미를 유추할 수 있는데, 코드가 어떤 역할을 하는지 이해가 더 쉬워진다..

생성자 대신 정적 팩토리 메서드를 사용하라는 이유는 여러 장점들 때문인데 아래와 같다.

### 장점 1 : 이름을 가질 수 있다.

기존 생성자 방식은 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못한다.

ex)

```
Player striker = new Player(185, 75, 20, 15);
Player midfielder = new Player(175, 70, 8, 15);
```

위의 코드에서 변수명이 없다면 단순히 new Player(185,75,20,15) 만 보고 스트라이커 라고 바로 알 수가 없다. 개발자는 각 생성자가 어떤 역할을 하는지 정확히 기억하기 어려워 엉뚱한 것을 호출하는 실수를 할 수가 있다. 코드를 읽는 사람도 클래스 설명 문서를 찾아보지 않고서는 의미를 알지 못할 것이다.

ex)

```
Player striker  = Player.newStriker();
Player midfielder  =Player.newMidfielder();
```

이번 예시는 정적 팩토리 메서드를 사용했을 때 코드이다. 이처럼 정적 팩토리 메서드를 사용하여 객체를 만들면

newStriker(), newMidfiedler()와 같이 이름을 가지기 때문에 어떤 역할을 하는지 바로 알 수가 있다.

또한 정적 팩토리 메서드를 사용하면 메서드 이름에 객체의 생성 목적을 담아 낼 수 있다.

ex)

```
public class LottoFactory() {
    private static final int LOTTO_SIZE = 6;

    private static List<LottoNumber> allLottoNumbers = ...; // 1~45까지의 로또 넘버

    // 정적 팩토리 메서드
    public static Lotto createAutoLotto() {
        Collections.shuffle(allLottoNumbers);
        return new Lotto(allLottoNumbers.stream()
                .limit(LOTTO_SIZE)
                .collect(Collectors.toList()));
    }

    // 정적 팩토리 메서드
    public static Lotto createManualLotto(List<LottoNumber> lottoNumbers) {
        return new Lotto(lottoNumbers);
    }
    ...
}
```

createAutoLotto와 createMenualLotto 모두 로또 객체를 생성하고 반환한다.

메서드의 이름만 보아도 로또 객체를 자동으로 생성하는지, 아니면 수동으로 생성하는지 이해가 가능하다.

이처럼 정적 팩토리 메서드를 사용하면 해당 생성의 목적을 이름에 표현할 수 있어 가독성이 좋아지는 효과가 있다.

### 장점 2 : 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.

인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.

따라서 (특히 생성 비용이 큰) 같은 객체가 자주 요청되는 상황인 경우, 성능을 끌어올릴 수 있다.

### 장점 3 : 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

API를 만들 때 하위 타입 객체를 반환하는 것을 응용하면 구현 클래스를 공개하지 않고 그 객체를 반환할 수 있어서 API를 작게 유지할 수 있다. API를 작게 유지하면 좋은 점은, 다른 개발자들이 무언가를 만들 때 익혀야 할 API가 작아서 쉽게 배우고 사용할 수 있다.

> 여기서 구현 클래스를 공개하지 않고 그 객체를 반환할 수 있다 라는 의미는?

점수에 따라서 레벨 인스턴스를 구별해서 반환해주는 로직을 구현하는 경우라 가정을 하자

정적 팩토리 메서드를 활용해서 API가 구성된 경우

```
public class Level {
  // 정적 팩토리 메서드
  public static Level of(int score) {
    if (score < 50) {
      return new Basic();
    } else if (score < 80) {
      return new Intermediate();
    } else {
      return new Advanced();
    }
  }
}

class Advanced extends Level {
}

class Intermediate extends Level {
}

class Basic extends Level {
}
```

API 문서에서 Level 클래스의 부분을 읽고, of(int score)라는 메서드를 사용하면 끝난다. (of 메서드는 추후에 설명)

즉, Advanced, Intermediate, Basic 클래스에 대해서는 캡슐화가 되어 객체 생성 인터페이스가 간단해진다.

정적 팩토리 메서드가 활용되지 않은 경우

```
public class Level {
}

public class Advanced extends Level {
}

public class Intermediate extends Level {
}

public class Basic extends Level {
}
```

API 문서에서 Level, Advanced,Intermediate, Basic 클래스의 부분을 모두 이해해야 되고, 점수에 따라 레벨 인스턴스를 반환할 때 아래와 같은 로직을 반복해서 사용해야 한다.

```
if (score < 50) {
    return new Basic();
} else if (score < 80) {
    return new Intermediate();
} else {
    return new Advanced();
}
```

중복이 일어날 가능성이 높고, API 문서도 복잡해져서 다른 개발자들이 API 문서를 보고 사용할 때 난이도가 올라간다.

> 다시말해, 정적 팩토리 메서드를 활용하면, '하나의 메서드'로만 반환할 객체를 조건에 따라 자동으로 선택하여 반환되게 할 수 있다.

### 장점 4 : 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

매개변수에 따라 원하는 클래스 객체를 반환할 수 있기에 효율적인 객체 생성이 가능하다.

```
public interface Number {

    static Number createNumber(int num){
        if (num > -1000 && num < 1000){
            return new SmallNum();
        }
        return new BigNum();
    }
}

class SmallNum implements Number{

}

class BigNum implements Number{

}
```

아래 코드에서 Number는 인터페이스가 있을 때 매개변수가 작으면 SmallNum을 반환해주고 크면 BigNum을 반환해줘서 효율적으로 메모리를 사용할 수 있다.

#### 장점 5 :정적 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

이부분이 제일 이해가 어려웠는데 여러 블로그를 다 돌아다니다가 그나마 이해가 된 코드를 가져왔다.

인터페이스나 클래스의 정적 팩터리 메서드가 만들어지는 시점에서 반환 타입의 클래스가 존재하지 않아도 된다는 것이다는 것이다.

```
public class MyBook{
  public static List<MyBookInterface> getInstance() {
    new ArrayList<>();
  }
}

public interface MyBookInterface{
  //이놈의 구현체는 아직 구현되지 않았다.
}

//1. 추 후 구현 클래스 생성
public class MyBookImpl implements MyBookInterface {}
//2. 클라이언트에서 활용
public class client{
  public static void main(String [] args) {
    List<MyBookInterface> myBookImpls = MyBook.getInstance();

    //추 후에 구현한 클래스를 생성 후 List에 추가
    MyBookInterface myBookImpl = new MyBookImpl();
    myBookImpls.add(myBookImpl);
  }
}
```

코드를 보면 MyBook의 정적 팩토리 메서드의 반환할 객체는 MyBookInterface의 구현 클래스인데 아직 구현되지 않은 것을 확인할 수 있다. 다시말해, 추후 정적 팩토리 메서드의 변경없이 List에 구현된 클래스를 add해서 사용하면 된다.

---

그렇다면 단점은...?

**1\. 정적 팩터리 메서드만으로는 하위 클래스를 만들 수 없다**

상속보다 컴포지션 사용을 유도하기 때문에 장점이 될 수도 있는 단점이다.  상속은 하위 클래스가 상위 클래스에 의존하기 때문에 좋지 않다.

- 컴포지션 - 다른 객체의 인스턴스를 자신의 인스턴스 변수로 포함해서 메서드로 호출하는 기법

**2\. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다**

생성자는 상단에 JavaDoc를 다뤄주지만 정적 팩터리 메서드는 특별히 다뤄주지 않기 때문에 클래스나 인터페이스 문서 상단에 정적 팩터리 메서드 문서를 제공해야 한다.

---

##  정적 팩터리 메서드 명명 방식

- **from**: 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드  
  Date d = Date.from(instant);
- **of**: 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드  
  Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
- **instance** 혹은 **getInstance**: 매개변수로 명시한 인스턴스를 반환, 같은 인스턴스임을 보장하진 않는다.  
  StackWalker luke = StackWalker.getInstance(options);
- **create** 혹은 **newInstance**: **instance** 혹은 **getInstance**와 같지만, 매번 새로운 인스턴스 생성을 보장  
  Object newArray = Array.newInstance(classObject, arrayLen);
- **getType**: **getInstance**와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용  
  FileStore fs = Files.getFileStore(path); ("**Type**"은 팩터리 메서드가 반환할 객체의 타입)
- **newType**: **newInstance**와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용  
  BufferedReader br = Files.newBufferedReader(path);
- **type**: **getType**과 **newType**의 간결한 버전  
  List<Complaint> litany = Collections.list(legacyLitany);
