# IdrisPractice
Idris Practice


[공식 튜토리얼 자료](http://docs.idris-lang.org/en/latest/tutorial/index.html)를 읽고 배운 내용들을 아래에 쭉 정리할 예정.

이 언어 자체가 실용적이냐? 라고 하면 그건 잘 모르겠는데.. 라는 느낌이지만, 이 언어에 있는 독특한 여러 가지 개념들은 꽤 재밌다.

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

isSingleton 함수는 넘어온 인자에 따라 서로 다른 타입을 리턴한다. 이 특성을 이용해서 함수의 인자와 타입 체킹을 연계하는데 이게 굉장히 신기하게 보였다.

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

### Vector

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

### Finite Set

```Idris
data Fin : Nat -> Type where
   FZ : Fin (S k)
   FS : Fin k -> Fin (S k)
```

`Fin` 이라는 데이터 타입은 유한한 크기의 집합을 나타내는 데이터 타입이다. 정확히는 `Fin n`은 `0` ~ `n-1` 까지 `n`개의 원소로 이루어진 집합을 말하는 듯. 여기서 `FZ` 값은 해당 집합의 첫 번째 값, `FS`는 순서대로 나머지 집합의 원소? 이라는 느낌. `FZ, FS FZ, FS (FS FZ), ...`의 순서대로 원소가 들어가 있는 집합이라고 생각하면 될 것 같다. 이걸 잘라서 `Fin 1`, `Fin 2`, `Fin 3`, ... 순서대로 보면 될 듯. `Nat`의 정의랑 별로 다르지 않다. 다만 원소가 0개인 집합의 원소라는 건 존재할 수가 없기 때문에, `Fin 0`은 정의할 수가 없다. 타입에서 `FZ : Fin (S k)`, `FS : Fin k -> Fin (S k)` 라고 정의 되어 있기 때문에, `Fin n` 에서 `n` 은 어떤 자연수의 Successor 여야만 한다. 하지만 0은 어떤 수의 Successor도 아니고, 따라서 `Fin 0`은 정의할 수 없는 것. 이 특성을 타입 시스템을 통해 검증할 수 있는 것이다. 그래서 `Vect` 타입의 특정 인덱스에 있는 원소를 가져올 때 `Fin` 타입을 쓴다.

```Idris
index : Fin n -> Vect n a -> a
index FZ     (x :: xs) = x
index (FS k) (x :: xs) = index k xs
```

이 정의로부터, 자연스럽게 원소 개수가 0개인 `Vect`의 특정 인덱스에 있는 값을 가져오려고 하면 저절로 컴파일 에러가 나게 되는 것이다. 따라서 `Vect` 값이 `Nil`인 경우의 체크도 하지 않아도 상관 없다. 어차피 `Fin 0` 자체가 존재하지 않기 때문에 해당 패턴은 타입 시스템에서 자연스럽게 컴파일 에러가 나기 때문.

### Implicit Arguments

```Idris
index : Fin n -> Vect n a -> a
```

이 타입 정의에서는 사실 생략된 부분이 있다. `n`과 `a`가 뭘 뜻하는 건지 빠져있기 때문. 이걸 포함해서 아래와 같이 작성할 수 있다.

```Idris
index : {a:Type} -> {n:Nat} -> Fin n -> Vect n a -> a
```

타입 선언에서 중괄호(`{}`) 안에 들어가 있는 애들은 Implicit Arguments로, 함수 호출할 때 넘어오는 인자를 통해 자동으로 추론되는 애들이다. 하지만 직접 명시해줄 수도 있다. 직접 명시해줄 일은 잘 없을 것 같긴 함

```Idris
index {a=Int} {n=2} FZ (2 :: 3 :: Nil)
```

얘네들을 패턴 매칭에 사용할 수도 있다

```Idris
isEmpty : Vect n a -> Bool
isEmpty {n = Z} _ = True
isEmpty {n = S k} _ = False
```

위 예제처럼 실제 값은 뭐든 상관 없고 그 값의 타입 인자에 따라서 함수 구현이 달라지는 경우에 유용한 듯. 확실히 타입 - 값 - 구현 간의 구분이 굉장히 모호하다.

### using

Implicit Arguments의 타입을 명시해주는게 편한 경우도 있다. 주로 의존 관계가 있을 때 Implicit Arguments 타입들을 묶어서 명시해주는 듯. 그 때 `using`을 쓴다.

```Idris
data IsElem : a -> Vect n a -> Type where
  Here : {x:a} -> {xs:Vect n a} -> IsElem x ( x :: xs )
  There : {x,y:a} -> {xs:Vect n a} -> IsElem x xs -> IsElem x (y :: xs)
```

`IsElem` 타입은 어떤 원소가 `Vect` 안에 있는 지 없는지 확인하기 위해 쓴다. 실제 코드에서 쓰기보다는 테스트 코드에서 쓸 것 같음. `Here`은 `Vect`에 제일 앞에 있는 경우, `There`은 나머지 위치에 있는 경우를 가리킴.

```Idris
testVec : Vect 4 Int
testVec = 3 :: 4 :: 5 :: 6 :: Nil

inVect : IsElem 5 Main.testVec -- testVec이 implicit Arguments가 아님을 명확히 하기 위해 namespace를 명시해줌
inVect = There (There Here)
```

여기서 똑같은 타입의 Implicit Argument가 많아서 코드가 난잡해진다. 이걸 `using` 블록으로 묶어줌. 편의 / 가독성을 위한 개념이고 그리 특별할 것은 없는 개념인 것 같다. 그 것보다는 `IsElem`같은 타입을 이용한 테스트 코드가 좀 신기.

```Idris
using (x:a, y:a, xs:Vect n a)
  data IsElem : a -> Vect n a -> Type where
     Here  : IsElem x (x :: xs)
     There : IsElem x xs -> IsElem x (y :: xs)
```

### Mutual block

Haskell이랑 다르게 Idris에서는 데이터 타입이나 함수가 사용되기 전에 반드시 정의가 되어야 한다. Dependent type이 갖고 있는 특성 때문이라는 듯. 타입 표기 등에서 함수를 쓰거나 하기 때문에?(앞의 testVec 예제처럼) 하지만 이렇게 하면 갑갑한 경우가 있기 때문에 대신에 `mutual block` 이라는 걸 제공해줌. `mututal block` 내에서는 순서 상관 없이 상호 참조하는 데이터 타입이나 함수를 정의 가능.

```Idris
mutual
  even : Nat -> Bool
  even Z = True
  even (S k) = odd k
  
  odd : Nat -> Bool
  odd Z = False
  odd (S k) = even k
```

위와 같이 `even` / `odd` 함수를 정의하는 방식은 꽤 재밌는데, 한 편으로는 너무 가독성이 떨어지지 않나 싶기도 하다. 예제니까 뭐 그러려니 하지만서도.

## IO

IO는 Haskell이랑 거의 다를 바가 없다. 마찬가지로 타입 시스템을 이용해 IO 와 IO가 아닌 코드를 분리함.

### do notation

이것도 Haskell이랑 다를 바 없음.

## Laziness

Haskell이랑 다르게 Idris는 eager evaluation 전략을 취한다. Haskell 하다보면 무조건 lazy evaluation하는 거때문에 빡칠 때가 좀 있어서 개인적으로는 괜찮은 선택이라고 생각. 대신에 Laziness가 분명 힘을 발휘할 때도 있어서, 이걸 별도의 타입으로 제공해준다.

```Idris
ifThenElse : Bool -> a -> a -> a
ifThenElse True t e = t
ifThenElse False t e = e
```

이런 구조의 함수의 경우 첫번째 `Bool` 값이 `True`인 경우에는 `t`만, `False`인 경우에는 `e`만 쓰게 된다. lazy evaluation할 때는 별 문제가 없는데, eager evaluation할 때는 쓰지 않을 값도 함수를 호출할 때 평가되어 버리기 때문에 성능 상 손해를 볼 수 밖에 없다. 그래서 Idris에서는 별도의 Lazy 타입을 제공해준다.

```Idris
data Lazy : Type -> Type where
  Delay : (val : a) -> Lazy a

Force : Lazy a -> a
```

여타 언어들에서 람다 등을 이용해 값을 담아뒀다가 필요할 때 평가해서 쓰는 방식과 유사. `Lazy` 타입의 값은 `Force`를 쓰기 전까진 평가되지 않는다. 다만 일일히 `Force` 해주고 하는 게 귀찮아지는게 문제인데 Idris의 경우 컴파일러가 알아서 해준다. 그냥 `Lazy` 값을 쓰기만 하면 자동으로 lazy evaluation이 된다는 것. 이 점이 꽤 마음에 든다. 이런 간단한 개념의 경우 타 언어에서도 쉽게 적용이 가능할 것 같고 또 유용할 것 같다는 느낌.

```Idris
ifThenElse : Bool -> Lazy a -> Lazy a -> a -- 이렇게 타입만 적절히 Lazy 넣어서 바꿔주면 됨
ifThenElse True  t e = t
ifThenElse False t e = e
```

## Codata Types

이것 역시 lazy evaluation의 연장에 있는 개념. 무한대 크기의 구조를 정의할 수 있는 데이터 타입은 `codata` 키워드를 써서 정의한다. 

```Idris
codata Stream : Type -> Type where
  (::) : (e : a) -> Stream a -> Stream a
```

이렇게 정의하면, 컴파일러가 알아서 아래와 같이 바꿔준다.

```Idris
data Stream : Type -> Type where
  (::) : (e : a) -> Inf (Stream a) -> Stream a
```

`Inf T` 타입은 `T` 인자를 lazy evaluation 하도록 바꿔준다. 이걸 통해서 무한대 크기의 데이터 구조를 정의할 수 있게 만드는 것. 그래서 Stream을 이용해서 아래와 같은 함수를 정의할 수 있다.

```Idris
ones : Stream Nat
ones = 1 :: ones
```

`Vect`나 `List` 라면 `ones` 호출 단계에서 평가되어버리기 때문에 무한 루프에 걸린다. `ones` 리턴 값 자체를 `Lazy`로 묶어도, 어쨌든 값을 가져다 쓸 때는 전체를 다 평가해야하기 때문에 쓰는 단계에서 무한 루프가 걸림. 부분적으로도 다 `Lazy`하게 평가가 되어야하기 때문에 `codata`라는 개념이 필요한 것 같다. 사실 `Inf` 타입을 직접 써도 상관없지만, 역시 귀찮기 때문에 `syntax sugar` 같은 개념으로 들어가 있는 듯. 다만 상호 참조하는 타입에 대해서는 `codata`를 쓸 수 없다는 문제가 있다.

```Idris
mutual
  codata Blue a = B a (Red a)
  codata Red a = R a (Blue a)

mutual
  blue : Blue Nat
  blue = B 1 red

  red : Red Nat
  red = R 1 blue

mutual
  findB : (a -> Bool) -> Blue a -> a
  findB f (B x r) = if f x then x else findR f r

  findR : (a -> Bool) -> Red a -> a
  findR f (R x b) = if f x then x else findB f b

main : IO ()
main = do printLn $ findB (== 1) blue
```

이 예제는 무한 루프에 걸린다. `codata`는 타입 정의에서 자기 자신이 나올 때만 `Inf`를 붙이는데, `Blue`와 `Red`가 정의 단계에서 서로를 참조할 때 자기 자신이 아닌 타입에는 `Inf`를 붙이지 않기 때문이다. `Blue`는 `Red`를 인자로 받고 `Red`는 `Blue`를 인자로 받으니 둘 다 `Inf`가 하나도 들어가지 않게 되는 거고, 결국 무한 루프에 빠지는 것. 그래서 이 경우 아래와 같이 직접 `Inf`를 붙여줘야 한다.

```Idris
mutual
  data Blue : Type -> Type where
   B : a -> Inf (Red a) -> Blue a

  data Red : Type -> Type where
   R : a -> Inf (Blue a) -> Red a

mutual
  blue : Blue Nat
  blue = B 1 red

  red : Red Nat
  red = R 1 blue

mutual
  findB : (a -> Bool) -> Blue a -> a
  findB f (B x r) = if f x then x else findR f r

  findR : (a -> Bool) -> Red a -> a
  findR f (R x b) = if f x then x else findB f b

main : IO ()
main = do printLn $ findB (== 1) blue
```

개인적으로는 이 예제를 보면서 그냥 `codata`라는 키워드 자체를 제공하지 않는게 낫지 않나? 라는 생각이 들었다. 좀 덜 직관적이고, 프로그래머의 실수를 유발할 수 있다는 느낌때문. 다 지원해주거나 아니면 다 안 지원해주는 게 맞다고 봄. dependent type같은게 프로그래머의 실수를 컴파일 타임에 최대한 많이 잡아내려고 나온 개념이고 Idris 역시 그 개념의 유용성을 실험하기 위한 언어라고 생각하는데, 막상 `codata` 같은 키워드가 그거랑 반대되게 프로그래머의 실수를 방치하는 느낌이라 좀 안맞다 싶다. 편의성을 위해서라기엔 무한대 크기 자료구조 같은 걸 정의할 일이 그렇게 많지도 않고.. 뭐 언어 제작자들 나름의 고민 끝에 나온 개념이니 내가 생각하지 못한 무언가가 있을 수도 있지만 개인적으로는 조금 아쉬움.

