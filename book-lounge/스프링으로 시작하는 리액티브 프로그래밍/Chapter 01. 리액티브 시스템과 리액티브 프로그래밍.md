```toc
```
# 리액티브 시스템 이란?

'reactive' 의 사전적 의미는 '반응을 하는' 이라는 뜻이 있다. 즉, 어떤 이벤트나 상황 발생 시, 반응을 해서 그에 따라 적절히 행동하는 것을 의미한다. 

reactive system 이란 '반응을 잘 하는 시스템'이라는 것을 의미한다. 정리하자면 reactive stystem 이란 클라이언트의 요청에 즉각적으로 응답함으로써 지연 시간을 최소화한다 라고 볼 수 있다.

# 리액티브 선언문으로 리액티브 시스템 이해

리액티브 선언문이란 리액티브라는 용어의 의미를 올바르게 정의하기 위해 리액티브 시스템 구축을 위한 일종의 설계 원칙 및 리액티브 시스템의 특징이라 할 수 있다.

<img width="603" alt="image" src="https://user-images.githubusercontent.com/37062337/233839511-3ec6ee39-cc1d-4649-adca-350b178134d5.png">

## Means (수단)

메세지 기반 통신을 통해 구성 요소들 간의 느슨한 결합, 격리성, 위치 투명성을 보장한다.

## Form (형성)

메세지 기반 통신을 통해 어떠한 형태를 지내는 시스템으로 형성되는지를 나타낸다. 위 그림에서는 비동기 메세지 통신 기반 하에 탄력성과 회복성을 가지는 시스템이어야 함을 보여준다.

탄력성이란 시스템의 작업량이 변하더라도, 일정한 응답을 유지하는 것을 의미한다. 즉, 시스템으로 유입되는 입력의 양이 많든, 적든 시스템에서 요구하는 응답성을 일정하게 유지하는 것을 말한다.

회복성이란 시스템에 장애가 발생하더라도 응답성을 유지하는 것을 의미한다. 회복성을 확보하기 위해 리액티브 시스템의 구성 요소들은 비동기 메세지 기반 통신을 통해 느슨한 결합과 격리성을 보장한다. 즉, 시스템의 구성요소들이 독립적으로 분리되기에, 장애가 발생해도 전체 시스템은 여전히 응답 가능하고 장애가 발생한 부분만 복구하면 된다.

## Value (가치)

비동기 메세지 기반 통신을 기반 -> 회복성과 예측 가능한 규모 확장 알고리즘 -> 시스템의 처리량을 자동으로 확장하고 축소하는 탄력성을 확보 -> 즉각적으로 응답 가능한 시스템을 구축할 수 있음을 의미


# 리액티브 프로그래밍이란?

리액티브 시스템을 구축하는 데 필요한 프로그래밍 모델이다. 

리액티브 시스템에서의 비동기 메세지 통신은 Non-Blocking I/O 방식의 통신이다. Blocking I/O는 스레드가 작업을 처리할 때 까지 차단되어 대기한다. 이 상황에서 뒤에 남은 작업들을 처리하려면 별도의 추가 스레드를 할당해야 한다. 이러한 단점이 있기 때문에 리액티브 시스템에서는 Non-Blocking I/O 방식의 통신을 사용한다.

# 리액티브 프로그래밍의 특징

아래는 위키피디아에 정의된 리액티브 프로그래밍에 대한 내용이다.

> In [computing](https://en.wikipedia.org/wiki/Computing "Computing"), **reactive programming** is a [declarative](https://en.wikipedia.org/wiki/Declarative_programming "Declarative programming") [programming paradigm](https://en.wikipedia.org/wiki/Programming_paradigm "Programming paradigm") concerned with [data streams](https://en.wikipedia.org/wiki/Stream_(computing) "Stream (computing)") and the propagation of change.

## declarative programming

선언형 프로그래밍으로, 실행할 동작을 구체적으로 명시하지 않고 이러한 동작을 하겠다는 목표만 선언하는 프로그래밍 패러다임이다.

## data streams, the propagation of change

data streams란 데이터가 지속적으로 발생하는 것을 의미하고, the propagation of change는 지속적으로 데이터가 발생할 때 마다 이것을 변화하는 이벤트로 보고, 이 이벤트를 발생시키면서 데이터를 계속적으로 전달하는 것을 의미한다.

# 리액티브 프로그래밍 코드 구성

크게 Publisher, Subscriber, Data Source, Operator 로 구성된다.

## Publisher

발행인, 발행자로 해석 가능하다. 발행자, 게시자, 생산자, 방출자 등의 여러 용어로 쓰이지만 공통점은 입력으로 들어오는 데이터를 제공하는 역할을 담당한다.

## Subscriber

구독자, 소비자라는 의미로 해석 가능하다. Publisher가 제공한 데이터를 전달받아 사용하는 주체라 할 수 있다. 

## Data Source

Publisher의 입력으로 들어오는 데이터를 대표하는 용어이다. 이는 리액티브 프로그래밍에선 Data Stream 이라고도 표현한다.

## Operator

애플리케이션의 요구사항에 맞게 Publisher와 Subscriber 사이에서 적절한 가공 처리를 담당한다.