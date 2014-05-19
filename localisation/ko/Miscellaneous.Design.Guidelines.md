# 기타 디자인 가이드라인 #

## 어떤 상태 값을 반환하지 말고 예외를 날립니다 ![](imgs/should.png) ##

성공이나 실패를 알리는 값을 반환하는 코드 기반은 중첩된 if문을 온 코드에 걸쳐 남용하는 경향이 있습니다. 꽤 자주 호출자는 반환값을 검사하는 것을 잊습니다. 구조화된 예외 처리는 예외를 날리고(throw) 잡거나(catch) 예외를 상위 레이어로 옮길 수 있도록 도입되었습니다. 대부분의 시스템에서 예상치 못한 상황이 발생했을 때 예외를 날리는 것은 상당히 일반적입니다.


## 풍부하고 의미있는 예외 메시지 문자열을 제공합니다 ![](imgs/should.png) ##

예외 메시지는 예외의 원인을 설명하고 예외를 피하려면 무엇을 해야 할지를 명확하게 설명해야 합니다.


## 가장 구체적이고 적합한 예외를 날립니다 ![](imgs/may.png) ##

예를 들어 메서드가 `null` 인자를 받았다면 `ArgumentException`보다는 `ArgumentNullException`을 날려야 합니다.


## 포괄적인 예외를 잡아 오류를 삼켜버리지 않습니다 ![](imgs/must.png) ##

애플리케이션 코드에서 지정하지 않은 예외, 예를 들어 `Exception`, `SystemException` 등을 잡아 오류를 삼켜버리지 않습니다. 오직 최상위 코드, 예를 들어 마지막 기회의 예외 처리기만이 지정하지 않은 예외를 잡아 로그를 남기고 애플리케이션이 우아하게 종료하도록 해야 합니다.


## 비동기 코드에서 적절하게 예외를 처리합니다 ![](imgs/should.png) ##

`async`/`await` 또는 `Task`를 사용하는 코드에서 예외를 날리거나 처리할 때에는 다음의 규칙을 기억하도록 합니다.

* `async`/`await` 블록과 `Task` 내의 액션에서 발생한 예외는 대기자(awaiter)에게 전파됩니다.
* 비동기 블록 전단에서 발생한 예외는 호출자에게 전파됩니다.


## 항상 이벤트를 처리하는 대리자가 `null`인지 확인합니다 ![](imgs/must.png) ##

구독자가 없는 이벤트는 `null`이므로 호출하기 전에 항상 이 이벤트 변수가 대표하는 대리자의 목록이 `null`이 아닌지 확인합니다. 또한 동시에 여러 스레드에서 발생할 수 있는 충돌을 방지하기 위해 임시 변수를 사용합니다.

```c#
event EventHandler<NotifyEventArgs> Notify;
void RaiseNotifyEvent(NotifyEventArgs args)
{
    EventHandler<NotifyEventArgs> handlers = Notify;
    if (handlers != null)
    {
        handlers(this, args);
    }
}
```

대리자의 목록이 `null`이 되지 않도록 방지할 수도 있습니다. 간단하게 다음과 같이 빈 대리자를 할당하면 됩니다.

```c#
event EventHandler<NotifyEventArgs> Notify = delegate {};
```


## 각 이벤트를 발생시킬 때 `protected virtual` 메서드를 사용합니다 ![](imgs/should.png) ##

이 가이드라인에 따르면 파생 클래스는 `protected` 메서드를 오버라이딩 하여 기반 클래스의 이벤트를 처리할 수 있습니다. 해당 `protected virtual` 메서드는 이벤트와 이름이 같아야 하며 이름 앞에 `On`을 붙입니다. 예를 들어 `TimeChanged` 이벤트의 `virtual` 메서드는 `OnTimeChanged`라는 이름을 사용합니다.

![NOTE](imgs/note.png) 파생 클래스가 오버라이드한 `protected virtual` 메서드에서 반드시 기반 클래스의 구현을 호출할 필요는 없습니다. 기반 클래스는 그 구현(된 메서드)이 호출되지 않더라도 반드시 올바르게 동작해야 합니다.


## 프로퍼티 변경 이벤트 제공을 고려합니다 ![](imgs/may.png) ##

특정 프로퍼티가 변경되었을 때 발생하는 이벤트 제공을 고려합니다. 이런 이벤트는 `PropertyChanged`라는 이름이어야 하며, 이를 통해 어떤 프로퍼티에 연관되어 있는지를 알 수 있어야 합니다.

![NOTE](imgs/note.png) 만약 클래스에 변경 이벤트가 필요한 프로퍼티가 많다면 `INotifyPropertyChanged` 인터페이스를 구현하는 것이 좋습니다. 이 방법은 [Presentation Model](http://martinfowler.com/eaaDev/PresentationModel.html)와 [Model-View-ViewModel](http://msdn.microsoft.com/en-us/magazine/dd419663.aspx) 패턴에서 자주 사용합니다.


## 이벤트를 발생시킬 때 `sender` 인자에 `null`을 전달하지 않습니다 ![](imgs/must.png) ##

흔히 하나의 이벤트 처리자는 여러 `sender`로부터 비슷한 이벤트를 처리합니다. `sender` 인자는 이벤트의 발생자를 얻기 위해 사용됩니다. 이벤트를 발생시킬 때 항상 발생자(일반적으로 `this`)에 대한 참조를 전달해야 합니다. 또한 이벤트의 데이터 파라미터에도 `null`을 전달하지 않도록 합니다. 만약 이벤트 데이터가 없다면 `null`이 아니라 `EventArgs.Empty`를 전달합니다.

![EXCEPTION](imgs/exception.png) 정적 이벤트일 경우 `sender` 인자는 `null`이 되어야 ![SHOULD](imgs/should.png) 합니다.


## 가능하다면 지네릭(generic) 제약을 사용합니다 ![](imgs/should.png) ##

지네릭 형식이나 메서드에서 `object` 형식을 캐스팅하지 말고 `where` 제약이나 `as` 오퍼레이터를 사용하여 지네릭 매개변수의 정확한 특성을 지정합니다.


```c#
class SomeClass
{
    ...
}

// Don't
class MyClass<T>
{
    void SomeMethod(T t)
    {
        object temp = t;
        SomeClass obj = (SomeClass) temp;
    }
}

// Do
class MyClass<T> where T : SomeClass
{
    void SomeMethod(T t)
    {
        SomeClass obj = t;
    }
}
```


## LINQ 표현식을 반환하기전에 결과를 평가(실행)합니다 ![](imgs/must.png) ##

다음의 코드를 봅시다.

```c#
public IEnumerable<GoldMember> GetGoldMemberCustomers()
{
    const decimal GoldMemberThresholdInEuro = 1000000;
    var q = (from customer in db.Customers
             where customer.Balance > GoldMemberThresholdInEuro
             select new GoldMember(customer.Name, customer.Balance));
    return q;
}
```

LINQ 쿼리는 지연된 실행을 사용하기 때문에 `q`를 반환하는 것은 실제로는 위의 쿼리를 대표하는 표현식의 트리를 반환하는 것입니다. 이것을 `foreach`나 비슷한 호출자에서 사용할 때마다 매번 `GoldMember`의 새 인스턴스를 가져오는 전체 쿼리가 재실행됩니다. 따라서 이렇게 반환된 `GoldMember` 인스턴스를 `==` 연산자로 비교할 수 없습니다. 이보다는 항상 `ToList()`나 `ToArry()` 등의 메서드를 써서 LINQ 쿼리의 결과를 명시적으로 평가하도록 합니다.
