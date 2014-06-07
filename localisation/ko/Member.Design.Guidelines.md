# 멤버 디자인 가이드 #

## 모든 프로퍼티는 어떤 순서로든 설정할 수 있어야 합니다 ![](imgs/must.png) ##

프로퍼티는 다른 프로퍼티에 대하여 상태에 관계없이(stateless) 동작해야 합니다. 예를 들자면, 프로퍼티 `A`를 먼저 설정하고 `B`를 설정하든 순서를 반대로 하든 차이가 없어야 합니다.


## 프로퍼티가 아닌 메서드를 사용합니다 ##

다음과 같은 상황에서는 프로퍼티가 아닌 메서드를 사용하는 것이 낫습니다.

* 해당 작업이 필드값을 설정하는 것보다 비용이 많이 드는 경우.
* `Object.ToString()`메서드와 같이 변환하는 기능인 경우.
* 인자가 변경되지 않았음에도 불구하고 호출할 때마다 결과가 달라지는 경우. 예를 들어, `NewGuid()`메서드는 호출할 때마다 다른 값을 반환함.
* 해당 프로퍼티와 직접 관계되어 있지 않은 내부 상태를 변경하는 등의 부작용을 일으키는 경우. ([명령 조회 분리-Command Query Separation](http://martinfowler.com/bliki/CommandQuerySeparation.html)를 위반).

![EXCEPTION](imgs/exception.png) 내부 캐시를 만들어내거나 [지연 로딩-lazy-loading](http://www.martinfowler.com/eaaCatalog/lazyLoad.html)을 구현하는 것은 사용해도 좋은 예외입니다.


## 상호 배타적인 프로퍼티를 사용하지 않습니다 ![](imgs/must.png) ##

동시에 사용할 수 없는 여러 프로퍼티를 포함한다는 것은 어떤 타입이 두 개의 충돌하는 개념을 표현한다는 전형적인 신호입니다. 그 개념들이 일부 동작과 상태를 공유한다 해도 그들은 분명히 서로 협력하지 않는 다른 규칙을 가질 것입니다.

도메인 모델에서 종종 보는 이 규칙 위반은 그 충돌 규칙에 관한 모든 종류의 조건부 로직을 필요로 하게 되어 유지보수 부담에 매우 중대한 악영향을 줍니다.


## 메서드 또는 프로퍼티는 오직 하나의 일만 처리해야 합니다 ![](imgs/must.png) ##

[SRP][srp]에 따라, 한 메서드는 반드시 ![MUST](imgs/must.png) 하나의 책임만을 가져야 합니다.


## 상태가 유지되는 오브젝트를 정적 멤버를 통해 노출하지 않습니다 ![](imgs/should.png) ##

상태가 유지되는 오브젝트는 많은 프로퍼티와 그 뒤로 수많은 동작 특성을 포함할 수 있습니다. 만약 그런 오브젝트를 정적 프로퍼티나 메서드를 통해 노출하면 리팩터링이나 그 오브젝트에 의존하는 다른 클래스에 대한 유닛테스트도 매우 어려워질 것입니다. 일반적으로 그런 종류의 생성작업을 도입하는 것은 이 문서의 많은 가이드라인을 위반하는 아주 멋진 예입니다.

여기에 해당하는 전형적인 예는 ASP.NET의 일부인 `HttpContext.Current` 프로퍼티입니다. 많은 사람들이 `HttpContext` 클래스를 수많은 추한 코드의 원천으로 봅니다. 사실 테스팅 가이드라인 [Isolate the Ugly Stuff](http://msdn.microsoft.com/en-us/magazine/dd263069.aspx#id0070015) 또한 종종 이 클래스를 지목합니다.


## 구체적 컬렉션 클래스보다는 `IEnumerable<T>`이나 `ICollection<T>`을 반환합니다 ![](imgs/should.png) ##

일반적으로 호출자가 반환 받은 내부의 컬렉션을 수정하는 걸 원치 않을 것입니다. 따라서 배열이나 `List` 또는 다른 컬렉션 클래스를 직접 반환하지 않습니다. 대신 `IEnumerable<T>`를 반환하거나 또는 호출자가 컬렉션의 개수를 알 수 있어야 할 때에는 `ICollection<T>`를 반환합니다.

![NOTE](imgs/note.png) .NET 4.5부터는 `IReadOnlyCollection<T>`, `IReadOnlyList<T>` 또는 `IReadOnlyDictionary<TKey, TValue>`를 사용할 수 있습니다.


## 문자열 또는 컬렉션을 나타내는 프로퍼티, 메서드 및 인자가 `null`이 되지 않도록 합니다 ![](imgs/must.png) ##

`null`을 반환하는 것은 호출자에게 예외적인 상황일 수 있습니다. 항상 `null`참조 대신에 **empty collection** 이나 **empty string**을 반환합니다. 이것은 또한 추가적으로 `null`을 체크하거나 `String.IsNotNullOrEmpty()` 이나 `String.IsNullOrWhiteSpace()`로 코드를 어지럽히는 것을 방지합니다.


## 매개 변수는 가능한 구체적으로 정의합니다 ![](imgs/should.png) ##

어떤 멤버가 특정한 데이터의 조각을 필요로 한다면, 매개 변수를 가능한 구체적으로 정의해야지 컨테이너 오브젝트로 받지 않습니다. 예를 들면 어떤 중앙의 `IConfiguration` 인터페이스로부터 얻을 수 있는 커넥션 스트링이 필요한 메서드를 생각해 봅시다. 전체 설정에 의존성을 갖는 것보다는 매개 변수가 커넥션 스트링을 받도록 정의합니다. 이것은 불필요한 결합을 방지할 뿐만 아니라 장기적으로 유지보수성을 향상시킵니다.


## 기본 데이터 형식보다는 도메인별 형식을 고려합니다 ![](imgs/may.png) ##

문자열이나 수치형 같은 기본 데이터 형식보다는 도메인에 특화된 타입, 예를 들어 ISBN 번호, 이메일 주소, 총 금액 등 데이터와 유효성 검사 규칙을 포함하는 전용 형식을 만들어 사용합니다. 이로써 같은 비즈니스 규칙을 여러번 구현하여 발생하는 유지보수성 증가와 버그를 방지합니다.


[solid]: http://programmers.stackexchange.com/questions/202571/solid-principles-and-code-structure
[srp]: http://www.objectmentor.com/resources/articles/srp.pdf
[ocp]: http://www.objectmentor.com/resources/articles/ocp.pdf
[lsp]: http://www.objectmentor.com/resources/articles/lsp.pdf
[isp]: http://www.objectmentor.com/resources/articles/isp.pdf
[dip]: http://www.objectmentor.com/resources/articles/dip.pdf
