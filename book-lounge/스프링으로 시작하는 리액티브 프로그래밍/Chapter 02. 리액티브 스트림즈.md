```toc
```
# 리액티브 스트림즈란?

리액티브 스트림즈란 개발자가 리액티브 라이브러리를 어떻게 구현할지 정의해 놓은 별도의 표준 사양을 의미한다.

한마디로, '데이터 스트림을 Non-Blocking하게, 비동기적인 방식으로 처리하기 위한 리액티브 라이브러리의 표준 사양'이라고 이해하면 된다.

# 리액티브 스트림즈 구성 요소

리액티브 스트림즈를 통해 구현해야하는 API 컴포넌트에는 아래 4가지가 있다.

## Publisher

데이터를 생성하고 통지(발행, 게시, 방출)하는 역할

## Subscriber

구독한 Publisher로 부터 통지된 데이터를 전달받아 처리하는 역할

## Subscription

Publisher에 요청할 데이터의 갯수를 지정하고, 데이터의 구독을 취소하는 역할

## Processor

Publisher와 Subscriber의 기능을 모두 갖고 있다. 즉, Subscriber로서 다른 Publisher를 구독할 수 있고, Publisher로서 다른 Subscriber가 구독할 수 있다.

## Publisher와 Subscriber 관계 그림

<img width="447" alt="image" src="https://user-images.githubusercontent.com/37062337/234275093-36155f94-10e9-44b9-a13e-1e6412a0c568.png">

## Subscriber가 Subscription.request를 통해 데이터의 요청 갯수를 지정하는 이유?

실제 Publisher와 Subscriber는 각각 다른 스레드에서 비동기적으로 상호작용하는 경우가 대부분이다. 이럴 경우 Publisher가 통지하는 속도보다 Subscriber가 받은 데이터를 처리하는 속도보다 빠르면 처리를 기다리는 데이터가 쌓이게 된다. 이는 시스템 부하가 커지는 결과를 낳게 된다. 이를 방지하기 위해 Subscription.request를 통해 데이터 갯수를 제어한다.

# 코드로 보는 리액티브 스트림즈 컴포넌트

## Publisher

```java
public interface Publisher<T> {
    public void subscribe(Subscriber<? super T> s);
}
```

코드 형태는 subscribe 메서드 하나만 구현하면 되는 수준으로 매우 단순하다. subscribe 메서드는 파라미터로 전달받은 Subscriber를 등록하는 역할을 한다.

하지만 인터페이스 코드를 보고 의문이 들 수 있다. Publisher는 데이터를 생성, 통지하는 역할을 담당하고 Subscriber는 Publisher가 통지하는 데이터를 전달받기 위해 구독을 한다고 이해하고 있다. 왜 Publisher에 Subscriber 메서드가 정의되어있을까?

Kafka의 경우, Publisher와 Subscriber 사이에 메세지 브로커가 있고, 이 브로커 내에 여러 개의 토픽이 존재하는데, Publisher와 Subscriber는 브로커에 있는 특정 토픽을 바라보는 구조로 이루어져 있다. 이는 Pubilsher와 Subscriber 사이의 느슨한 결합 구조라 볼 수 있다.

반면 리액티브 스트림즈는 Publisher가 subscribe 메서드의 파라미터인 Subscriber를 등록하는 형태로 구독이 이루어진다.

## Subscriber

```java
public interface Subscriber<T> {
	public void onSubscribe(Subscription s);
	public void onNext(T t);
	public void onError(Throwable t);
	public void onComplete();
}
```

- onSubscribe : 구독 시작 시점에 Publisher에게 요청할 데이터 갯수를 지정, 구독을 해지하는 것과 같은 처리를 담당한다. 이 처리는 onSubscribe의 파라미터 타입인 Subscription 객체를 통해 이루어진다.
- onNext : Publisher가 통지한 데이터 처리를 담당한다.
- onError : Publisher가 데이터 통지 시 에러가 발생했을 때, 해당 에러를 처리하는 역할을 담당한다.
- onComplete : Publisher가 데이터 통지를 완료했음을 알릴 때 호출된다. 특정 후처리를 해야한다면 여기서 처리 코드를 작성한다.

## Subscription

```java
public interface Subscription {
	public void request(long n);
	public void cancel();
}
```

request는 Subscriber가 구독한 데이터 갯수를 요청하는 역할, cancel은 구독을 취소하는 역할을 담당한다.

## Processor

```java
public interface Processor<T, R> extends Subscriber<T>, Publisher<R>{}
```

Processor는 별도로 구현해야하는 메서드가 없다. 위에서 설명한 것 처럼 Processor는 Subscriber, Publisher 두 역할을 모두 담당할 수 있기 때문이다.

# 리액티브 스트림즈 관련 용어 정의

## Signal

Publisher와 Subscriber 간 주고 받는 상호작용을 Signal 이라 한다. 인터페이스에서 보았던 `onSubscribe()`, `onNext()`, `onComplete()`, `onError()`, `request()`, `cancel()` 을 리액티브 스트림즈에서는 Signal이라 한다.

`onSubscribe()`, `onNext()`, `onComplete()`, `onError()` 는 Subscriber 인터페이스에 정의되지만, 실제 호출해서 사용하는 주체는 Publisher 이기 때문에 Publisher가 Subscriber에게 보내는 Signal 이며, `request()`, `cancel()` 는 Subscription 인터페이스에 정의되어 있지만, 실제 사용 주체는 Subscriber 이므로 Subscriber가 Publisher에게 보내는 Signal 이다.

## Demand

단어 의미로는 수요, 요구를 뜻한다. 이 수요, 요구라 함은 Subscriber가 Publisher에게 요청하는 데이터를 의미한다. 구체적으로 Publisher가 아직 Subscriber에게 전달하지 않은 Subscriber가 요청한 데이터를 의미한다.

## Emit

방출하다 라는 의미로, 데이터를 내보낸다. 라고 이해하면 된다. Publisher가 emit 하는 Signal 중에서 데이터를 전달하기 위한 onNext Signal을 줄여서 '데이터를 emit 한다.' 라고 표현한다.

## Upstream / Downstream

리액티브 세상에서 흘러가는 것은 데이터다. 코드를 예시로 보자면

```java
Flux
	.just(1, 2, 3, 4, 5, 6)
	.filter(n -> n % 2 == 0)
	.map(n -> n * 2)
	.subscribe(System.out::println);
```

just()는 Publisher의 역할로, 데이터 생성 후 emit 하게 된다.  이후 filter(), map() 과 같이 메서드 체이닝으로 호출할 수 있는데, 같은 Flux 타입으로 반환되기 때문이다.

Upstream / Downstream 관점으로 보자면 `just()`로 반환된 Flux 입장에서는 `filter()` 호출을 통해 반환된 Flux가 자신보다 아래에 있기 때문에 Downstream이 된다.

반면, `filter()`를 통해 반환된 Flux는 `just()`로 반환된 Flux보다 밑에 있기 때문에 Upstream 이 된다.

정리하자면 `just()`로 반환된 Flux -> Upstream, `filter()`로 반환된 Flux -> Downstream 이 된다.

## Sequence

Publisher가 emit하는 데이터의 연속적인 흐름을 정의해놓은 것이다. 이 Sequence는 Operator 체인 형태로 정의된다.

위 코드 예제 처럼 Flux를 통해 데이터를 생성, emit 하고 filter()를 통해 필터링 후, map()을 통해 변환하는 과정 자체를 Sequence라고 정의한다.

쉽게 이해하자면, Sequence는 다양한 Operator로 데이터의 연속적인 흐름을 정의한 것이라고 이해하면 된다.

## Operator

연산자라는 의미로, 위 코드에서 사용한 just(), filter(), map() 같은 메서드들을 리액티브 프로그래밍에서는 연산자로 정의한다.

## Source

'최초의' 라는 의미로 Original이라는 용어도 자주 사용된다. '최초에 가장 먼저 생성된 무언가' 즉, 원본이라고 이해하면 좋다.

# 리액티브 스트림즈의 구현 규칙

## Publisher 주요 규칙

1. Publisher가 Subscriber에게 보내는 onNext signal의 총 갯수는 항상 해당 Subscriber의 구독을 통해 요청된 데이터의 총 개수보다 작거나 같아야 한다.
2. Publisher는 요청된 것보다 적은 수의 onNext signal을 보내고 onComplete 또는 onError를 호출하여 구독을 종료할 수 있다.
3. Publisher의 데이터 처리가 실패하면 onError signal을 보낸다.
4. Publisher의 데이터 처리가 모두 성공한다면 onComplete signal을 보낸다.
5. Publisher가 Subscriber에게 onError 또는 onComplete signal을 보내는 경우 해당 Subscriber의 구독은 취소된 것으로 간주되어야 한다.
6. onComplete, onError와 같은 종료 signal 을 받은 이후, 더이상 signal이 발생되지 않아야 한다.
7. 구독이 취소되면 Subscriber는 signal을 받는 것을 중지해야 한다.

### 부연 설명
1. Subscriber가 요청한 것보다 더 적거나 같은 갯수의 onNext signal을 통해 데이터를 보낼 수 있다.
2. 예를 들어, Subscriber의 요청 개수가 5 / Publisher가 emit할 데이터 갯수가 3이라면 3개를 Subscriber에게 emit 후 onComplete or onError signal을 보내야 한다. 다만, Publisher가 끊임 없이 처리해야할 무한 스트림의 경우는 예외다.
7. 구독 취소를 위해 Subscription.cancel()이 호출되었을 때, Publisher가 구독 취소 요청을 준수해야한다는 의미와 같다.

## Subscriber 주요 규칙

1. Subscriber는 Publisher로 부터 onNext signal을 수신하기 위해 Subscription.request(n)을 통해 Demand signal을 Publisher에게 보내야 한다.
2. onComplete(), onError()는 Subscription 또는 Publisher의 메서드를 호출해서는 안된다.
3. onComplete(), onError()는 signal을 수신한 후 구독이 취소된 것으로 간주되어야 한다.
4. 구독이 더 이상 필요하지 않다면 onCancel()을 호출해야 한다.
5. onSubscribe()는 지정된 Subscriber에 대해 최대 한 번만 호출되어야 한다.

### 부연 설명 

1. 데이터를 언제, 얼마나 수신할 수 있는지를 결정하는 책임이 Subscriber에게 있다. 한 번에 하나의 데이터를 요청하기 보다는 Subscriber가 처리할 수 있는 적절한 상한선만큼의 데이터 갯수 요청을 권장한다.
2. Publisher, Subscription과 Subscriber 간의 순환 및 경쟁 조건을 방지하기 위함이다.
4. 리소스를 적절하게 해지하기 위함이다.
5. 동일한 구독자가 최대 한 번만 구독할 수 있다는 의미와 같다.

## Subscription 주요 규칙

1. 구독은 Subscriber가 onNext 또는 onSubscribe 내에서 동기적으로 Subscription.request 를 호출하도록 허용해야한다.
2. 구독이 취소된 후 추가적으로 호출되는 Subscription.request, Subscription.cancel 은 효력이 없어야 한다.
3. 구독이 취소되지 않은 동안 Subscription.request의 매개변수가 0보다 작거나 같다면 onError signal을 보내야 한다.
4. 구독이 취소되지 않은 동안 Subscription.cancel은 Publisher가 Subscriber에게 보내는 signal을 중지하도록 요청해야 한다.
5. 구독이 취소되지 않은 동안 Subscription.cancel은 Publisher에게 해당 구독자에 대한 참조를 삭제하도록 요청해야 한다.
6. Subscription.cancel, Subscription.request 호출에 대한 응답으로 예외를 던지는 것을 허용하지 않는다.
7. 구독은 무제한 수의 request 호출을 지원해야 하고, 최대 (2의 63 제곱 - 1) 개의 demand를 지원해야 한다.

### 부연 설명

1. 'onNext 또는 onSubscribe 내에서 동기적으로 Subscription.request 를 호출하도록 허용해야한다.' 라는 의미는 request를 호출하는 스레드와 onNext signal을 보내는 스레드가 동일할 수 있다는 것이다.
3. 잘못된 구현으로 인해 예외 발생 없이 계속 작업이 진행되는 것을 방지하도록 하기 위함이다.
4. cancel()은 구독을 취소 + signal 을 보내는 것을 중지하라고 Publisher에게 요청하는 의미까지 포함되어 있다.
5. 해당 규칙을 통해 GC가 더 이상 유효하지 않은 구독자의 객체를 수집하여 메모리를 확보할 수 있도록 해준다.
6. 리액티브 스트림즈에서는 예외 발생 시 onError signal과 함께 보내도록 규정한다.
