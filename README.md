# IdrisPractice
Idris Practice


[공식 튜토리얼 자료](http://docs.idris-lang.org/en/latest/tutorial/index.html)를 읽고 배운 내용들을 아래에 쭉 정리할 예정.

## Data Types

기본적인 데이터 타입 선언 방식은 Haskell과 굉장히 유사하다.

```Idris
data Nat = Z | S Nat -- 자연수

data List a = Nil | (::) a (List a)
```

Haskell에서는 cons 연산자가 `:` 이고 타입 어노테이션이 `::` 인데 Idris에서는 반대인 점이 조금 헷갈림.

## Functions

역시 Haskell과 유사. 다만 Idris에서는 모든 함수에 대해 반드시 타입을 명시해주어야 한다. 또다른 차이점으로는 Haskell 처럼 함수이름이 반드시 소문자로만 시작해야하는 것은 아니며 값 생성자(data constructor)든 타입 생성자든(Nat, List) 아무런 제약없이 이름을 지을 수 있음. 다만 관습적으로 Haskell과 같은 방식으로 이름을 짓는 듯.

```Idris
plus : Nat -> Nat -> Nat
plus Z     y = y
plus (S k) y = S (plus k y)

mult : Nat -> Nat -> Nat
mult Z     y = y
mult (S k) y = plus y (mult k y)
```

## where

Haskell처럼 where 절을 이용해 해당 함수 내에만 쓰이는 것들을 정의할 수 있다. 역시 인덴트 맞춰줘야하는 건 당연.

```Idris
reverse : List a -> List a
reverse xs = revAcc [] xs where
  revAcc : List a -> List a -> List a
  revAcc acc [] = acc
  revAcc acc (x :: xs) = revAcc (x :: acc) xs

foo : Int -> Int
foo x = case isLT of
          Yes => x * 2
          No => x * 4
  where
    data MyLT = Yes | No -- 이런식으로 where절에 타입 정의도 가능
    
    isLt : MyLT
    isLT = if x < 20 then Yes else No

```

where 절에 있는 함수들도 일반적으로 타입을 명시해주어야하지만 일부 조건이 만족될 경우 안 써도 된다. Idris가 타입 추론을 완벽하게 할 수 없는 타입 시스템(의존 타입 - dependent type이 포함된 타입 시스템)을 쓰기 때문인 것 같다.

## Dependent Type

제일 재밌는 주제. 의존 타입. 사실 이게 뭔지 궁금해서 배워보려고 한 거니까.. 

### First Class Types

Idrs에서는 타입도 언어의 일급 시민(First class)으로 취급한다. 즉, 함수 인자로도 전달이 되며 연산도 하고 자유롭게 다룰 수 있다는 뜻이다. 그래서 아래와 같은 함수를 작성할 수 있다.

```Idris
isSingleton : Bool -> Type
isSingleton True = Nat
isSingleton False = List Nat
```

isSingleton 함수느 넘어온 인자에 따라 서로 다른 타입을 리턴한다. 이 특성을 이용해서 함수의 인자와 타입 체킹을 연계하는데 이게 굉장히 신기하게 보였다.

```Idris
mkSingle : (x : Bool) -> isSingleton x
mkSingle True = 0
mkSingle False = []
```

인자로 넘어온 Bool 값이 True면 숫자 0, False면 빈 리스트를 넘겨준다. 즉, 리턴 타입이 달라진다. 이걸 isSingleton 함수를 이용해 함수의 인자로 넘어온 Bool 값과 리턴 타입을 연결시켜주는 것이다. 함수의 리턴 타입이 함수의 인자로 넘어온 값에 의존되어 있다(dependent)라고 생각하면 될 것 같다.

혹은 리턴 타입이 아니라 함수의 인자 타입이 어떻게 되는지를 검증할 때도 쓸 수 있다.

```Idris
sum : (single : Bool) -> isSingleton single -> Nat
sum True x = x
sum False [] = 0
sum False (x :: xs) = sum False xs
```

위의 함수는 자연수 1개를 인자로 받거나 자연수 여러개의 리스트를인자로 받아 그 전체의 합을 돌려준다. 이 때 첫번째 인자가 True인지 False인지를 통해 두 번째 인자가 그냥 숫자 하나인지 리스트인지를 검증한다. 굉장히 신기하다.

## Vector

리스트랑 좀 비슷하게 생긴 녀석으로 `Vect` 라는 타입이 있다.

```Idris
data Vect : Nat -> Type -> Type where
  Nil : Vect Z a
  (::) : a -> Vect k a -> Vect (S k) a -- Idris에서는 오버로딩도 된다. List의 cons 연산자랑 똑같이 생겨먹음
```

이건 Haskell의 [Type Families 확장](https://ocharles.org.uk/blog/posts/2014-12-12-type-families.html) 문법과 유사한 방식으로 타입을 정의하는 것. `Vect` 타입은 자연수와 타입을 인자로 받는데, 이게 대표적인 의존 타입의 사용 예라고 한다. Vect n a는, 길이가 n인 a 타입의 리스트를 나타내는 것. 그래서 두 `Vect`를 연결하는 함수는 아래와 같이 정의된다.

```Idris
(++) : Vect n a -> Vect m a -> Vect (n+m) a
(++) Nil       ys = ys
(++) (x :: xs) ys = x :: xs ++ ys
```

여기서 `++` 함수의 타입을 보면, 리턴 타입이 `Vect (n+m) a`로 명시되어 있다. 즉, `++` 함수로 길이 `n` 리스트와 길이 `m` 리스트를 연결하면 결과는 길이 `n+m` 리스트가 나와야한다는 것을 타입으로 명시해주는 것이다. 해당 함수가 가져야하는 기본 조건(assert)를 타입으로 명확하게 나타내준다는 점에서 꽤 강력하다는 생각이 들었다.

```Idris
(++) : Vect n a -> Vect m a -> Vect (n+m) a
(++) Nil       ys = ys
(++) (x :: xs) ys = x :: xs ++ xs
```

혹시나 위와 같이 잘못 작성한다면 (`ys`라고 적어야하는데 `xs`라고 오타를 낸 경우) 결과 리스트의 길이가 `n+m`이 아니라고 컴파일 에러를 낸다. 일반적으로 런타임에 확인해야하는 에러 사항을 컴파일 타임에 확인할 수 있게 만들어주는 것이다. 최근의 언어 타입에 대한 연구가 다 이런 식으로 최대한 빨리 프로그래머가 저지른 실수를 알려주는 방향으로 이루어지고 있지 않나 싶다.

## Finite Set

