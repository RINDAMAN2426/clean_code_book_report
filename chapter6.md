# 객체와 자료구조

## 자료 추상화

```java
// 구체적인 클래스 A
public class Point {
  public double x;
  public double y;
}

// 추상적인 클래스 B
public interface Point {
  double getX();
  double getY();
  void setCartesian(double x, double y);
  double getR();
  double getTheta();
  void setPolar(double r, double theta);
}
```

- 클래스B는 자료구조는 명확하면서도 구현을 감출 수 있다.
- 클래스A의 변수를 `private`으로 하더라도, 설정함수 조회함수가 제공된다면 결국 구현을 노출하는 것이다.
- 사용자가 구현을 모른 채 자료의 핵심을 조작할 수 있어야 진정한 의미의 클래스이다.

```java
public interface Vehicle {
  double getFuelTankCapacityInGallons();
  double getGallonsOfGasoline();
}

public interface Vehicle {
  double getPercentFuelRemaining();
}
```

- 위의 클래스는 단순히 조회를 해올 뿐이다. (구체적인 수치로 해당 변수를 조회하게 한다.)
- 아래의 클래스는 퍼센트로 표현함으로 써 추상적인 개념으로 알려줄 뿐 구체적인 데이터가 어떤지는 알 수 없다.

## 자료/객체 비대칭

### 객체

추상화 뒤로 자료를 숨긴채 자료를 다루는 함수만 공개한다.

### 자료구조

자료는 그대로 공개하며, 별다른 함수는 제공하지 않는다.

#### 절차적인 도형 클래스

```java
public class Square {
  public Point topLeft;
  public double side;
}

public class Rectangle {
  public Point topLeft;
  public double height;
  public double width;
}

...

public class Geometry {
  public final double PI = 3.14159...;

  public double area(Object shape) throws NoSuchShapeException {
    if (shape instance of Square) {
      Square s = (Square)shape;
      return s.side * s.side;
    }
    ...
  }
}
```

- Geometry에 도형의 길이를 구하는 함수가 추가되더라도 도형클래스는 형향을 받지 않는다. 그에 의존하고 있는 다른 클래스도 마찬가지이다.
- 반대로 도형을 추가하고 싶다면, Geometry에 속한 모든 함수를 고쳐야한다.

#### 다형적인 도형 클래스

```java
public class Square implements Shape {
  private Point topLeft;
  private double side;

  public double area() {
    return side * side;
  }
}

public class Rectangle implements Shape {
  private Point topLeft;
  private double width;
  private double height;

  public double area() {
    return width * height;
  }
}
```

- area는 다형 메소드다. 새 도형을 추가하더라도 기존의 도형들에는 아무런 영향이 없다.
- 반대로 새 함수를 추가하고 싶다면 도형 클래스를 모두 고쳐야 한다.

위의 두가지는 서로 상호보완적인 관계에 있다. 그래서 객체와 자료구조는 근본적으로 양분된다.

> 절차적인 코드는 기존 자료 구조를 변경하지 않으면서 새 함수를 추가하기 쉽다. 객체 지향적인 코드는 기존 함수를 변경하지 않으면서 새 클래스를 추가하기 쉽다.

> 절차적인 코드는 새로운 자료 구조를 추가하기 어렵다. 객체 지향 코드는 새로운 함수를 추가하기 어렵다.

때로는 단순한 자료구조와 절차적인 코드가 가장 적합한 상황도 있다.

#### 디미터 법칙

모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다.

객체는 자료를 숨기고 함수를 공개한다. 즉 조회 함수로 내부 구조를 공개하면 안된다.

> 디미터 법칙 - "클래스 C의 메소드 f는 다음과 같은 객체의 메소드만 호출해야한다"<br>
>
> - 클래스 C<br>
> - f가 생성한 객체<br>
> - f인수로 넘어온 객체<br>
> - C 인스턴스 변수에 저장된 객체<br>
>
> 단, 위의 메소드가 반환하는 객체의 메소드를 호출해선 안된다.

다음 코드는 디미터 법칙을 어기는 코드

```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```

##### 기차 충돌

위와 같은 코드는 기차 충돌`train wreck`이라 부른다. 때로는 `Feature Envy`라고 부른다.

```java
Options opts = ctxtx.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```

디미터 법칙을 위반하는지 여부는 `ctxt`, `Options`, `ScratchDir`에 의해 결정된다. 객체라면 위반이며 자료구조라면 디미터 법칙은 적용되지 않는다.

만일 다음과 같이 수정한다면 아예 디미터 법칙을 거론할 필요가 없다.

```java
final String outputDir = ctxt.options.scratchDir.absolutePath;
```

##### 잡종 구조

결국엔 절반은 객체, 절반은 자료구조의 형태인 잡종 구조가 발생하게 된다. 잡종 구조는 양쪽의 단점만 모이기 쉽기에 지양해야한다.

##### 구조체 감추기

만일 객체라면 내부구조를 감추어야 하기 때문에 변경해야 한다.

```java
BufferedOutputStream bos = ctxt.createScratchFileSystem(classFileName)
```

결국 절대 경로를 얻으려는 이유는 임시 파일을 생성하기 위함이기 때문에, ctxt 객체에게 해당 메소드를 동작 시킨다면 내부 구조를 감추면서 동작 시킬 수 있다.

## 자료 전달 객체 (DTO)

전형적인 형태는 공개변수만 있고 함수가 없는 클래스

일반적인 형태는 `bean`구조이다. 일종의 사이비 캡슐화로, 별다른 이익은 없다.

```java
public class Address {
  private String street;
  private String streetExtra;
  ...

  public Address(String street, String streetExtra, ...) {
    this.street = street;
    this.streetExtra = streetExtra;
    ...
  }

  public getString getStreet() {
    return street;
  }
}
```

### 활성 레코드

- DTO의 특수한 형태
- 공개 변수 혹은 조회 함수 + save 혹은 find와 같은 탐색 함수도 제공한다.
- 활성 레코드는 자료구조로 생각해야한다. 비즈니스 로직을 담는 것은 다른 객체로 생성해야한다.

## 결론

- 객체는 동작을 공개하며 자료는 숨긴다.
- 자료구조는 별다른 동작은 없으며 자료를 노출한다.
- 자료 타입의 추가가 유연성이 필요하다면 객체가 용이할 것이며, 새로운 동작의 추가에 있어서 유연성이 필요하다면 자료구조와 절차적인 코드가 더 적합하다.
