# 클래스 디자인 가이드라인 #

## 하나의 클래스 또는 인터페이스는 반드시 단일한 목적을 가져야 합니다 ![](imgs/must.png) ##

하나의 클래스 또는 인터페이스는 그것이 포함된 시스템에서 단일한 목적을 가져야 합니다. 일반적으로 클래스는 email이나 ISBN 번호와 같은 기본 형식을 대표하거나, 어떤 비즈니스 컨셉에 대한 추상화이거나, 순수 데이터 구조이거나, 다른 클래스들의 상호작용을 조율하는 책임을 집니다. 절대로 이들을 조합하지 않습니다. 이 규칙은 [SOLID 원칙-SOLID principles][solid] 중에서 [단일 책임 원칙-Single Responsibility Principle (SRP)][srp]으로 잘 알려졌습니다.


[디자인 패턴-Design Patterns](http://en.wikipedia.org/wiki/Design_pattern_(computer_science))을 사용한다는 것은 클래스의 의도를 전달하려는 것입니다. 만약 클래스에 단일한 디자인 패턴을 적용할 수 없다면 그 클래스는 하나 이상의 일을 할 가능성이 있습니다.


## 인터페이스는 작고 명확해야 합니다 ![](imgs/should.png) ##

인터페이스는 그 목적 또는 시스템 내의 역할을 명확히 설명하는 이름을 가져야 합니다. 연관성이 희박한 멤버들이 단지 같은 클래스에 사용된다고 같은 인터페이스에 조합하면 안 됩니다. 멤버들을 각자의 책임에 기반하여 나누어 호출자가 꼭 필요한 호출을 하거나 특정 작업에 연관된 인터페이스만을 구현하도록 합니다. 이 규칙은 [SOLID 원칙-SOLID principles][solid] 중에서 [인터페이스 분리 원칙-Interface Segregation Principle (ISP)][isp]으로 더 잘 알려져 있습니다.


## 다중 구현을 지원할 때 기반 클래스보다는 인터페이스를 사용합니다 ![](imgs/may.png) ##

클래스에 확장할 수 있는 점을 노출하고 싶다면, 그 부분을 기반 클래스(base-class)보다는 인터페이스로 노출합니다. 그 확장 점을 사용하고 싶은 사용자들에게 기반 클래스로부터 파생하도록 강제하면 원치 않은 동작을 일으킬 수 있습니다. 그러나 사용자들의 편의를 위해 (추상적인) 기본 구현(default implementation)을 제공하여 구현의 시작점으로 제공할 수 있습니다.


## 인터페이스를 각 클래스의 결합을 끊는데 사용합니다 ![](imgs/should.png) ##

인터페이스는 각 클래스의 결합을 끊는데 매우 효과적인 수단입니다.

* 양방향 연관성을 방지합니다.
* 하나의 구현을 다른 것으로 간단하게 대체할 수 있게 합니다.
* 개발 환경에서 무거운 외부 서비스나 리소스를 임시 요소로 대체할 수 있게 합니다.
* 유닛 테스트에서 실제 구현을 더미 구현이나 가짜 오브젝트로 대체할 수 있게 합니다.
* 의존성 주입(dependency injection) 프레임워크를 사용하여 특정 인터페이스가 필요할 때 어떤 클래스를 선택할지 중앙집중 관리를 할 수 있게 합니다.


## 정적 클래스를 피합니다 ![](imgs/may.png) ##

정적 클래스(static class)는 디자인이 나쁜 코드를 만들곤 합니다. 또한 정적 클래스는 아주 특수한 툴을 사용하지 않는 한 테스트에서 격리시키기가 매우 어렵습니다. 단, 확장 메서드의 컨테이너를 구현할 때는 제외합니다.

![NOTE](imgs/note.png) 만약 진짜로 정적 클래스가 필요하다면 클래스에 `static`을 붙여 컴파일러가 클래스와 그 멤버를 초기화하는 것을 방지하도록 합니다. 이것은 명시적으로 private 생성자를 만들지 않아도 된다는 장점도 있습니다.


## `new` 키워드를 사용하여 상속한 멤버를 숨기지 않습니다 ![](imgs/must.png) ##

new 키워드는 객체지향 원칙의 가장 핵심적인 부분 중 하나인  [다형성-Polymorphism](http://en.wikipedia.org/wiki/Polymorphism_in_object-oriented_programming)을 깨뜨리는 것은 물론이고, 서브 클래스를 더욱 이해하기 어렵게 합니다. 다음의 두 클래스를 고려하자면,

```c#
public class Book
{
    public virtual void Print()
    {
        Console.WriteLine("Printing Book");
    }
}

public class PocketBook : Book
{
    public new void Print()
    {
        Console.WriteLine("Printing PocketBook");
    }
}
```

이 코드는 클래스 구조상 아마도 원치 않은 행동의 원인이 될 것입니다.

```c#
var pocketBook = new PocketBook();

pocketBook.Print();         // "Printing PocketBook"을 출력합니다.
((Book)pocketBook).Print(); // "Printing Book"을 출력합니다.
```

Print 메서드를 기반 클래스의 참조에서 호출하든 파생 클래스에서 호출하든 **반드시**![](imgs/must.png) 결과는 같아야 합니다.


## 파생된 오브젝트를 기반 클래스 오브젝트인 것처럼 취급합니다 ![](imgs/should.png) ##

다시 말하자면, 어떤 파생된 오브젝트에 대한 참조를 어디에서라도 그 파생된 클래스의 상세에 관계없이 그 클래스의 기반 클래스의 참조로 바꾸어 사용할 수 있어야 합니다. ![](imgs/should.png) 이 규칙을 위반하는 매우 악명높은 사례가 어떤 베이스 클래스의 메서드를 오버라이드 할 때 `NotImplementedException` 예외를 던지는 것입니다. 조금 미묘한 것은 기반 클래스에서 기대되는 동작을 존중하지 않는 것입니다. 이 규칙은 [SOLID principles][solid] 중 하나인 [리스코프 치환 원칙-Liskov Substitution Principle (LSP)][lsp]으로 잘 알려졌습니다.


## 기반 클래스에서 파생된 클래스를 참조하면 안됩니다 ![](imgs/must.png) ##

기반 클래스에서 자신의 자식 클래스에 대해 의존성을 갖는 것은 객체지향 디자인에 맞지 않으며 다른 개발자가 새 파생 클래스를 추가하지 못하게 될 것입니다.


## 어떤 오브젝트가 의존하고 있는 다른 오브젝트를 노출하지 않도록 합니다 ##

만약 이런 식으로 코드를 작성하고 있는 걸 발견한다면 아마도 [데메테르의 법칙-Law of Demeter (LoD)](http://en.wikipedia.org/wiki/Law_of_Demeter) 을 위반하였을 것입니다. 더 상세한 설명은 [여기](http://www.blackwasp.co.uk/LawOfDemeter.aspx)에 있습니다.

```c#
someObject.SomeProperty.GetChild().Foo()
```

한 오브젝트는 자신이 의존하는 다른 클래스를 노출해서는 안됩니다. 왜냐면 호출자가 그 오브젝트의 노출된 프로퍼티나 메서드를 잘못 사용할 수 있기 때문입니다. 그렇게 하면 호출 코드가 사용중인 클래스에 결합되어 나중에 이것을 쉽게 변경할 수 없게 만듭니다.

![NOTE](imgs/note.png) LINQ 등의 [Fluent Interface](http://en.wikipedia.org/wiki/Fluent_interface) 패턴을 사용한 클래스를 사용할 때 이 규칙을 어기는 것으로 보입니다. 그러나 이것은 단순히 그 자신을 반환할 뿐이므로 이런 메서드 연계는 허용됩니다.

![EXCEPTION](imgs/exception.png) [Unity](http://msdn.microsoft.com/unity), [Autofac](http://autofac.org) or [Ninject](http://www.ninject.org) 와 같은 Inversion of Control (or Dependency Injection) 프레임워크는 종종 의존성을 public 프로퍼티로 노출해야 할 때가 있습니다. 이 프로퍼티가 의존성 주입외에 어떤 다른 용도로도 사용되지 않는 한 이것은 규칙 위반이 아니라고 간주할 수 있습니다.


## 양방향 의존성을 피합니다 ![](imgs/must.png) ##

이것은 두 클래스가 서로의 public 멤버를 알고 있거나 서로의 내부 동작에 의존하는 것을 의미합니다. 이런 클래스 중 하나를 리팩터링하거나 교체하면 양쪽 모두에 변경이 필요하며 예상치 못한 수 많은 작업이 발생할 수 있습니다. 이런 의존성을 깨는 가장 확실한 방법은 둘 중 하나를 인터페이스로 만들고 의존성 주입(dependency injection)을 사용하는 것입니다.


## 클래스는 반드시 상태(state)와 동작(behavior)을 가져야 합니다 ![](imgs/must.png) ##

[데이터 전송 오브젝트-Data Transfer Object](http://martinfowler.com/eaaCatalog/dataTransferObject.html)라고 하는 통신 채널간 데이터 전송을 위한 클래스가 아닌 한, 클래스는 반드시![MUST](imgs/must.png) 자신의 상태와 동작을 정의하는 로직을 가져야 합니다.


[solid]: http://programmers.stackexchange.com/questions/202571/solid-principles-and-code-structure
[srp]: http://www.objectmentor.com/resources/articles/srp.pdf
[ocp]: http://www.objectmentor.com/resources/articles/ocp.pdf
[lsp]: http://www.objectmentor.com/resources/articles/lsp.pdf
[isp]: http://www.objectmentor.com/resources/articles/isp.pdf
[dip]: http://www.objectmentor.com/resources/articles/dip.pdf
