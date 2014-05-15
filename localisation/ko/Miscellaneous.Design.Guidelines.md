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

An event that has no subscribers is `null`, so before invoking, always make sure that the delegate list represented by the event variable is not `null`. Furthermore, to prevent conflicting changes from concurrent threads, use a temporary variable to prevent concurrent changes to the delegate.

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

You can prevent the delegate list from being empty altogether. Simply assign an empty delegate like this:

```c#
event EventHandler<NotifyEventArgs> Notify = delegate {};
```


## Use a protected virtual method to raise each event ![](imgs/should.png) ##

Complying with this guideline allows derived classes to handle a base class event by overriding the protected method. The name of the protected virtual method should be the same as the event name prefixed with On. For example, the protected virtual method for an event named `TimeChanged` is named `OnTimeChanged`.

![NOTE](imgs/note.png) Derived classes that override the protected virtual method are not required to call the base class implementation. The base class must continue to work correctly even if its implementation is not called.


## Consider providing property-changed events ![](imgs/may.png) ##

Consider providing events that are raised when certain properties are changed. Such an event should be named `PropertyChanged`, where Property should be replaced with the name of the property with which this event is associated.

![NOTE](imgs/note.png) If your class has many properties that require corresponding events, consider implementing the `INotifyPropertyChanged` interface instead. It is often used in the [Presentation Model](http://martinfowler.com/eaaDev/PresentationModel.html) and [Model-View-ViewModel](http://msdn.microsoft.com/en-us/magazine/dd419663.aspx) patterns.


## Don't pass `null` as the sender argument when raising an event ![](imgs/must.png) ##

Often, an event handler is used to handle similar events from multiple senders. The sender argument is then used to get to the source of the event. Always pass a reference to the source (typically `this`) when raising the event. Furthermore don't pass `null` as the event data parameter when raising an event. If there is no event data, pass `EventArgs.Empty` instead of `null`.

![EXCEPTION](imgs/exception.png) On static events, the sender argument ![SHOULD](imgs/should.png) be `null`.


## Use generic constraints if applicable ![](imgs/should.png) ##

Instead of casting to and from the `object` type in generic types or methods, use `where` constraints or the `as` operator to specify the exact characteristics of the generic parameter. For example:

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


## Evaluate the result of a LINQ expression before returning it ![](imgs/must.png) ##

Consider the following code snippet:

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

Since LINQ queries use deferred execution, returning `q` will actually return the expression tree representing the above query. Each time the caller evaluates this result using a `foreach` or something similar, the entire query is re-executed resulting in new instances of `GoldMember` every time. Consequently, you cannot use the `==` operator to compare multiple `GoldMember` instances. Instead, always explicitly evaluate the result of a LINQ query using `ToList()`, `ToArray()` or similar methods.
