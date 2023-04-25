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


