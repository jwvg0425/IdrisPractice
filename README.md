# IdrisPractice
Idris Practice


[공식 튜토리얼 자료](http://docs.idris-lang.org/en/latest/tutorial/index.html)를 읽고 생각한 내용들을 아래에 쭉 정리할 예정. 

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

Haskell처럼 where 절을 이용해 해당 함수 내에만 쓰이는 것들을 정의할 수 있다. 인덴트 맞춰줘야하는 건 당연.

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

이건 Haskell의 [Type Families 확장](https://ocharles.org.uk/blog/posts/2014-12-12-type-families.html) 문법과 유사한 방식으로 타입을 정의하는 것. `Vect` 타입은 자연수와 타입을 인자로 받는데, 이게 대표적인 의존 타입의 사용 예라고 한다. `Vect n a`는, 길이가 `n`인 `a` 타입의 리스트를 나타내는 것. 그래서 두 `Vect`를 연결하는 함수는 아래와 같이 정의된다.

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

## Useful Data Types

### List and Vect

리스트 류 타입에 대한 syntactic sugar로 `[], [1,2,3]` 같은 걸 쓸 수 있다는 것. `map` 같은 함수에 대한 설명(오버로딩 되어있다는 것 등)이 있는데 오버로딩 제외하면 Haskell이랑 별 다를 바 없어서 딱히 신경쓸 부분은 없다.

#### Anonymous Function, Section

Haskell이랑 문법이 거의 같다. 람다에서 패턴매칭 가능이라든가 타입 명시 해줄 수도 있다 이런 것도. 다만 인자들을 공백으로 구분하는게 아니라 콤마(`,`)를 넣어서 구분해줘야 한다. `\x, y, z => x + y + z` 뭐 이런 식. 왜 그렇게 했는지는 모르겠다. 이유가 있을까? 그냥 함수 정의할 때도 콤마를 쓰지 않으니 람다에서도 쓰지 않는게 더 직관적일 것 같은데... 약간 일관성 없는 것 같은 느낌. Haskell과 굳이 차이를 뒀으니 뭔가 이유가 있을 것도 같은데 잘 모르겠다.

### Maybe

Haskell이랑 다를 것 없음.

### Tuple

그냥 쓸 때는 Haskell이랑 별 다를 바가 없긴 한데.. (a,b,c) 이런 애들이 nested pair로 구성된다는 사실은 좀 특이한 듯. Haskell도 내부 구현이 저런가? 잘 모르겠다. `fst` / `snd` 동작이 Haskell이랑 다른 걸 봐서 아닌 것 같은데. Idris에서는 `fst` / `snd`를 모든 튜플에 대해 사용 가능하며, 3개 이상 짜리 튜플에 `snd`를 쓰면 마치 `tail` 함수처럼 첫번째 원소를 제외한 나머지로 구성된 튜플이 나온다(내부적으로 nested pair기 때문). 이건 좀 괜찮은 것 같기도 하다.

### Dependent Pair

이것도 꽤 재밌는 개념. 두번째 원소의 타입이 첫번째 원소의 값에 의존성을 갖는 페어를 말한다. 전통적으로(학계에서?) `sigma types`라는 이름으로 불린다고 함.

```Idris
data DPair : (a : Type) -> (P : a -> Type) -> Type where
  MkDPair : {P : a -> Type} -> (x : a) -> P x -> DPair a P
```

`DPair`를 위한 syntactic sugar로 `(a : A ** P)`가 있다. `A`와 `P`의 페어이며, `a`는 `P` 안쪽에서 나타난다(P에 의해 평가되는 값). 실제 타입의 값은 `(a ** p)`로 나타냄. 

```Idris
vec : (n : Nat ** Vect n Int)
vec = (2 ** [3, 4])
```

`DPair`의 첫번 째 값은 `Vect`의 길이를, 두 번째 값은 실제 `Vect`의 값을 나타내는데 쓰였다. 풀어서 쓰면 아래와 같다.

```Idris
vec : DPair Nat (\n => Vect n Int)
vec = MKDPair 2 [3,4]
```

여기서 `n` 값은 `Vect`의 길이로부터 추론이 가능하기 때문에, 생략이 가능하다. placeholder(`_`)를 쓸 수 있음.

```Idris
vec : (n : Nat ** Vect n Int)
vec = (_ ** [3,4])
```

타입의 `Nat` 역시 `Vect n Int`로부터 추론 가능하므로, 생략 가능. implicit arguments 생각해보면 될 듯.

```Idris
vec : (n ** Vect n Int)
vec = (_ ** [3,4])
```

이 타입을 쓰는 대표적인 사례가 `Vect`에 대한 `filter` 함수.

```Idris
filter : (a -> Bool) -> Vect n a -> (p ** Vect p a)
```

`Vect`는 해당 값의 길이가 얼마인지를 알아야하는데, `filter`를 적용한 결과 길이가 얼마나 될 지를 알 수 없기 때문에 정의하기가 까다롭다. 이 때 쓰는 것이 `DPair`. 필터링한 결과 나오는 원소 개수 `p`에 의존적인 `DPair` 값을 리턴하게 만들어서 해결.

```Idris
filter p Nil = (_ ** []) -- 당연
filter p (x :: xs) with (filter p xs) -- with은 나중에 설명한다고 함. 하지만 대충 무슨 뜻인지는 알아 먹겠다
  | (_ ** xs') = if (p x) then (_ ** x :: xs') else (_ ** xs') --길이는 어차피 추론하기 때문에 다 _로 생략
```

좋은 개념이긴 한데 코드가 좀 복잡해지지 않나, 그리고 학습 비용이 좀 크지 않나 하는 느낌이 든다. Idris 코드들 전반적으로 다 그런 느낌. dependent type 하나 때문에 코드 전체적인 복잡도가 확 올라가는 것 같다. 단순히 타입과 값이 혼재되는 이 방식 자체가 익숙하지 않기 때문인가 싶기도 하고. dependent pair도 nested 하게 쓸 수 있을 것 같은데, (첫번 째 값에 두번째 타입이 의존, 두번째 값에 세번째 타입이 의존, ...) 그런 코드는 진짜 복잡하지 않을까.

### Records

이건 Haskell이랑 쫌 다르다. 구문 자체가 좀 다름

```Idris
record Person where
  constructor : MkPerson
  firstName, middleName, lastName : String
  age : Int

fred : Person
fred = MkPerson "Fred" "Joe" "Bloggs" 30
```

`constructor`를 따로 분리해서 키워드로 씀. 그 밑에 필드 목록을 적는데, 같은 타입의 값들은 한 줄에 묶어서도 쓸 수 있다. 값 가져다 쓰고 하는 부분에서는 Haskell이랑 다를 바 없음. 필드 가져오는 함수 자동으로 생성해주는 것 뿐 일반 `data`랑 큰 차이는 없다는 점도 마찬가지.

`record` 함수(라고 해야할지 구문이라고 해야할지)를 이용해 일부 필드만 갱신된 복사본을 만들 수 있음. 이것도 Haskell이랑 약간 다름. 명시적이라 나은 거 같기도 하고 쓸데없이 타자만 많이 쳐야되는 것 같기도 하고?

```Idris
record { firstName = "Jim" } fred
record { firstName = "Jim", age = 20 } fred
```

좋은 점은 필드 이름 중복이 허용된다는 것. Haskell에서 이 것때문에 얼마나 빡쳤는지를 생각해보면..

당연하게도 dependent type을 필드로 가지는 record 역시 정의할 수 있다.

```Idris
record Class where
  constructor ClassInfo
  students : Vect n Person
  className : String
```

`Vect`의 길이 `n`이 타입 `Class`의 정의에 포함되어 있지 않기 때문에 서로 다른 길이의 `Vect`로 업데이트하는 것도 가능.

```Idris
addStudent : Person -> Class -> Class
addStudent p c = record { students = p :: students c } c
```

근데 이렇게 되면 `Class`가 내부 학생들의 숫자에 대한 정보를 잃어버리니 좀 그렇지 않나? 하는 생각이 들었는데... 좀 더 보니 명시해줄 수도 있었다. 엄격함과 편의성에서 어느 정도 타협을 보는 방식인 듯. 이건 해당 파트에서 다시 보자

### Nested record update

이거 꽤 편리하다. Haskell에서 불편했던 점을 착실하게 개선하는 문법이 꽤 많은 것 같다. 레코드 안의 레코드 안의 레코드 안의 필드를 가져온다든가 갱신한다든가 하려면 진짜 짜증나는데, 그걸 아래와 같이 참조할 수 있다.

```Idris
record { a->b->c = val } x
```

`x` 내부 필드 `a` 내부 필드 `b` 내부 필드 `c` 를 갱신해주는 것(엄밀히 말하면 갱신된 복사본을 돌려주는 것). 문법도 직관적이라 이해하기도 쉽고.

```Idris
record { a->b->c } x
```

이건 그냥 값만 가져오기. 맨 뒤에 적용 값 없애면 그 자체로 함수로 동작

```Idris
record { a->b->c } // 함수임
```

단순하지만 괜찮은 문법인 것 같다. 좀 길어서 엄청 편하거나 하진 않을 것 같지만 기존에 Haskell에서 쓰던 것 보다는 편한 듯. 더 편한 방법은 없을까?

### Dependent Records

당연히 Dependent Record도 존재할 수 있다. 앞에서 말한 `Class` 레코드를 학생 숫자에 의존적인 타입으로 만들 수 있다.

```Idris
record SizedClass (size : Nat) where
  constructor SizedClassInfo
  students : Vect size Person
  className : String

-- addStudent 정의가 아래와 같이 바뀌어야 함

addStudent : Person -> SizedClass n -> SizedClass (S n)
addStudent p c = SizedClassInfo (p :: students c) (className c)
```

다만 위 예시는 의존적으로 해도 되고 안 해도 되는데, 반드시 의존적으로 만들어야만 하는 예시가 있는지 아니면 그냥 무조건 의존적으로도 의존적이지 않게도 할 수 있는지는 잘 모르겠다. 느낌적으로는 implicit arguments에 대해서만 생략할 수 있는건가? 싶긴한데 안 그럴 거 같기도 하고. 근데 그러면 타입 시스템의 제약이 너무 약해지지 않나(컴파일 타임에 검증하는 범위가 줄어들지 않나) 싶기도 하고 프로그래머가 필요한 만큼 적절히 쓰는 거니 편의성 차원에서 괜찮은 것 같기도 하고(너무 깐깐하면 피곤하니까). 이 부분은 좀 더 고민해봐야 할 것 같다.

## More Expressions

### let bindings

Haskell과 동일.

### List comprehensions

Haskell과 동일.

### case expressions

역시 Haskell이랑 별반 다를 바 없는 듯. List comprehension 포함해서 이제 이런 류 문법들은 타 언어들에도 조금씩 도입되기 시작한 만큼 그렇게 특별할 건 없는 것 같다.

### Totality

가능한 모든 입력 값에 대해 종료되거나, 어떤 출력값을 만들어 내는게 보장이 되는 함수를 total function으로 부른다고 한다. Idris의 `head` 함수가 total function이다.


```Idris
||| Get the first element of a non-empty list
||| @ ok proof that the list is non-empty
head : (l : List a) -> {auto ok : NonEmpty l} -> a
head []      {ok=IsNonEmpty} impossible
head (x::xs) {ok=p}    = x
```

Idris에 흥미를 갖게 만든 요소중 하나인 컴파일 타임의 프로그램 증명과 관련된 내용인 것 같다. `{auto ok : NotEmpty l}` 이 부분이 빈 리스트에 대한 `head` 호출을 컴파일 에러가 일어나게 만드는 부분이라고 한다. Haskell 문법 상에서는 이런 식으로 컴파일 에러를 일으키는게 불가능하기 때문에 런타임 에러를 일으킨다. 

자세한 내용은 잘 이해가 안가고..(설명도 부실) 나중에 좀 더 다루는 것 같으니 거기서 다시 제대로 배워보자. 저 `ok` 가 뭔가 증명과 관련된 내용을 하고, total function의 개념에 대해서만 알고 있으면 될 듯. total이 아닌 함수를 partial function이라고 부른다고 한다(일부 입력 값에 대해서만 동작하는 애들)

## Interfaces

Haskell의 type class에 대응. 문법도 사용법도 유사함. 인스턴스 정의하는 문법만 조금 다른듯

```Idris
interface Show a where
  show : a -> String

show : Show a => a -> String

Show Nat where
  show Z = "Z"
  show (S k) = "s" ++ show k
```

## Default Definitions

Haskell과 동일~

## Extending Interfaces

Haskell과 동일~

## Functors and Applicatives

Haskell과 개념은 같은데, 인터페이스 정의 방식이 약간 다르다. Haskell로 치면 `Kind`를 직접 명시해주어야 한다고 생각하면 될 듯.

```Idris
interface Functor (f : Type -> Type) where
  map : (m : a -> b) -> f a -> f b
```

여기서 말하는 `f`는 엄밀히 따지면 타입이 아니기 때문에 명시해주어야한다고 한다. 명확하게 보여준다는 점에서 Haskell보다 나은 것 같음.

## Monad & do notation

Haskell이랑 다를 바 없음.

### Pattern Matching Bind

Monad transformer가 갖고 있는 복잡함을 좀 해소해주는 방식이 아닌가 싶다. 

```Idris
readNumbers : IO (Maybe (Nat, Nat))
readNumbers =
  do x <- readNumber
     case x of
          Nothing => pure Nothing
          Just x_ok => do y <- readNumber
                          case y of
                               Nothing => pure Nothing
                               Just y_ok => pure (Just (x_ok, y_ok))
```

두 개의 숫자를 입력받아 그걸 튜플로 돌려주는 함수를 작성한다고 하면 위와 같이 굉장히 복잡한 코드가 나오게 되는데, Haskell에서는 이걸 모나드 트랜스포머로 처리한다. 모나드 트랜스포머는 개념을 학습하는 비용이 좀 크다는 단점이 있는데, Idris에서는 이걸 Pattern Matching Bind로 좀 쉽게 해결할 수 있는 것 같다(모든 경우에 모나드 트랜스포머를 대체할 수 있는지는 모르겠다. 일부 쉽게 갈 수 있는 경우에도 모나드 트랜스포머를 써서 코드가 복잡해지는 걸 어느정도 방지하는 효과가 있는 것 같음).

```Idris
readNumbers : IO (Maybe (Nat, Nat))
readNumbers =
  do Just x_ok <- readNumber | Nothing => pure Nothing
     Just y_ok <- readNumber | Nothing => pure Nothing
     pure (Just (x_ok, y_ok))
```

파이프(|) 앞이 선호되는 바인드고, 이게 실패하면 파이프 뒤에 있는 값으로 처리함.

### ! notation

개인적으로는 오 이거 진짜 괜찮다! 라는 생각이 든 문법. 모나드 쓰다보면 코드가 쓸 데 없이 길어질 때가 많은데, 그걸 많이 줄여주면서 코드 자체도 직관적이어서 좋다.

```Idris
m_add : Maybe Int -> Maybe Int -> Maybe Int
m_add x y = return (!x + !y)
```

맨 안 쪽 / 왼쪽 -> 오른쪽 순서로 자동으로 바인딩 해줌.

```Idris
let y = 42 in f !(g !(print y) !x)

--실제로는 이렇게 처리 됨

let y = 42 in do y' <- print y
                x' <- x
                g' <- g y' x'
                f g'
```

모나드 함수임에도 그냥 함수 호출하듯이 코드 간결하게 짤 수 있다는 점이 굉장히 매력적임.

### Monad comprehensions

이것도 좀 재밌는 개념. list comprehensions를 monad 전체로 확장. 엄밀히 말하면 `Monad` 와 `Alternative`를 만족하는 모두에 대해서 동작. `guard` 함수 때문에 `Alternative`는 당연히 만족해야하는 거지만. 찾아보니 Haskell도 처음에는 모든 모나드에 대해 쓸 수 있었다는데 나중에 List에 대해서만 가능하도록 제약이 붙었다고 한다. 왜일까? 괜히 헷갈리게 돼서 그런 것 같기도 하고. 아무튼 이걸 이용하면 위의 `m_add` 함수를 아래와 같이 정의할 수도 있다.

```Idris
m_add : Maybe Int -> Maybe Int -> Maybe Int
m_add x y = [ x' + y' | x' <- x, y' <- y]
```

이렇게 보니 확실히 좀 헷갈릴 것 같기도 하다. 이 문법 자체가 모나드의 단순한 syntactic sugar다 보니 사실 그렇게 필요한가? 싶기도. 오히려 Haskell이나 Idris말고 타 언어에서 더 유용할 것 같다(이미 Python이 잘 가져다 쓰고 있기도 하고).

## Idiom brackets

Haskell에서 Applicative Functor 쓸 때 번거롭던 걸 해결해주는 문법. `[| f a1 ... an |]` 은 `pure f <*> a1 <*> ... <*> an`으로 번역된다. 이 것도 역시 사소하지만 편의성을 꽤 많이 증진시켜준다. 

```Idris
m_add : Maybe Int -> Maybe Int -> Maybe Int
m_add x y = [| x + y |]
```

Applicative Functor가 파서나 인터프리터 만들 때 편리한 부분이 많으니 당연히 이 것도 그런거 짤 때 편리하다. 튜토리얼에 있는 [예제](http://docs.idris-lang.org/en/latest/tutorial/interfaces.html#an-error-handling-interpreter) 참조. 

## Named Implementations

이것도 좀 편리한 문법인 것 같다. Haskell에서 `newtype`을 통해서 해야하는 걸 좀 쉽게 해주는 듯.

```Idris
[myord] Ord Nat where
   compare Z (S n)     = GT
   compare (S n) Z     = LT
   compare Z Z         = EQ
   compare (S x) (S y) = compare @{myord} x y

testList : List Nat
testList = [3,4,1]

*named_impl> show (sort testList)
"[sO, sssO, ssssO]" : String
*named_impl> show (sort @{myord} testList)
"[ssssO, sssO, sO]" : String
```

문법 자체는 딱히 설명 필요 없을 만큼 간단하니 생략.

## Determining Parameters

```Idris
interface Monad m => MonadState s (m : Type -> Type) | m where
  get : m s
  put : s -> m ()
```

특정 인터페이스의 구현을 찾는데 사용되는 파라메터를 명시해 주는 것. 파이프(|)뒤에 쓴다. MonadState의 구현을 찾는데 필요한 파라메터는 `m`이고, `s`는 그 다음에 함수 적용 과정에서 결정되는 타입이기 때문에 해당 인터페이스의 구현체를 결정하는 인자가 무엇인가?를 결정하는 인자를 명시해주는 것이라고 한다. 잘 몰랐는데 Haskell에도 있는 문법인 듯? idris 보면서 haskell도 같이 공부되는 것 같은 기분..

## Modules and namespaces

이 부분은 다른 언어와 그렇게 큰 차이가 있는 부분도 아니고, 내가 Idris에서 배우고 싶은 내용이랑은 크게 관련 없는 부분이라 생략.

## with

```Idris
filter : (a -> Bool) -> Vect n a -> (p ** Vect p a)
filter p [] = ( _ ** [] )
filter p (x :: xs) with (filter p xs)
  | ( _ ** xs' ) = if (p x) then ( _ ** x :: xs' ) else ( _ ** xs' )
```

이 예제에서 썼던 것. 여기서는 `filter p xs`의 결과를 분해하는데 쓰였다. 간접적인 연산의 결과를 분해해서 패턴으로 매칭하는 문법인듯.

좀 더 복잡한 예제는 다음과 같다.

```Idris
data Parity : Nat -> Type where
   Even : Parity (n + n)
   Odd  : Parity (S (n + n))

natToBin : Nat -> List Bool
natToBin Z = Nil
natToBin k with (parity k)
   natToBin (j + j)     | Even = False :: natToBin j
   natToBin (S (j + j)) | Odd  = True  :: natToBin j
```

자연수를 이진수 표현으로 변형하는 것인데, 좀 이해가 안 된다. `parity k`의 결과가 파이프(|) 오른쪽, 왼쪽에는 그 결과가 영향을 끼치는 패턴이 온다고 한다. 여기선 `parity k` 결과에 따라 `k` 를 `(j+j)` 혹은 `S (j+j)`로 분해하는 듯. 아직 명확히 느낌이 오진 않는데 왜 이런 문법을 쓰는가는 약간 감이 올듯 말듯. Haskell의 [view patterns extension](https://ocharles.org.uk/blog/posts/2014-12-02-view-patterns.html)과 비슷한 느낌인 것 같은데..

## Theorem Proving

이쪽은 튜토리얼에 설명이 좀 부족하고(막상 제일 궁금했던 내용인데...), [이 문서](https://eb.host.cs.st-andrews.ac.uk/Idris/theorems.html)에 비교적 설명이 잘 되어 있다. 하지만 이 파트 내용 자체가 기본적으로 [Curry-Howard correspondence](https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence)를 이해하고 있어야 해서 어려운 듯. 이 부분도 아직 기초 부분밖에 이해를 못 해서 좀 더 공부해봐야 겠다. 이 쪽은 Haskell wiki books의 [이 문서](https://en.wikibooks.org/wiki/Haskell/The_Curry%E2%80%93Howard_isomorphism)가 비교적 쉽게 설명이 되어 있는 듯. 요점은 프로그래밍에서의 타입 시스템이 Propositional Proof와 유사하다는 것. isomorphism이라는 표현을 쓰던데 이게 수학 용어라 정확하게 내가 생각하는 게 맞는지는 모르겠지만, 어쨌든 두 체계가 거의 같고, 그래서 프로그램의 타입 시스템을 이용해 증명을 할 수 있다는 것이 제일 중요한 포인트. Idris는 특유의 타입 시스템을 통해 이런 증명 시스템을 지원해준다.

### Equality

Idris에서는 Propositional Equality에 대한 정의를 할 수있다. 이 걸 이용해서 프로그램에 관한 theorem을 증명할 수 있다. 기본적으로 아래의 타입을 사용.

```Idris
data (=) : a -> b -> Type where
  Refl : x = x
```

`Refl`이라는 이름은 Reflection에서 온 거라고. 근데 Reflection이 어떤 의미인지 정확하게 몰라서 잘 이해가 안 감. 이럴 때 기초 공부가 많이 부족한다는 걸 느낀다. 이 부분은 관련 기초 내용 좀 더 착실히 공부하고 다시 이해해봐야할 듯. 아무튼 위 타입을 쓰면 아래와 같은 정의가 가능.

```Idris
fiveIsFive : 5 = 5
fiveIsFive = Refl

twoPlusTwo : 2 + 2 = 4
twoPlusTwo = Refl
```

### Simple theorems

간단한 증명 예제. `0+n=n`을 증명.

```Idris
plusReduces : (n:Nat) -> (plus 0 n = n)
```

`n`이 자연수일 때 `0+n=n` 임을 보인다고 생각하면 될 듯. 논리 기호 그대로라는 느낌?

```Idris
plusReduces : n = refl n
```

이건 이렇게 하면 증명이 된다고. `plus 0 n`은 `plus`의 정의에 의해 `n`으로 정규화되기 때문. 하지만 이걸 약간 바꾸면 증명이 좀 어려워진다.

```Idris
plusReducesO : (n : Nat) -> (n = plus n 0)
```

이게 왜 더 어려워진거지? 라고 생각했는데, 이유가 `plus`함수가 함수의 첫 번째 인자에 대해 재귀적으로 정의되어 있기 때문이다. 위 예제에서는 재귀함수 정의에서 첫번째 인자가 `0`일 때 패턴매칭으로 그냥 `n`으로 정규화됨을 컴파일러가 쉽게 알 수 있는데, 그걸 거꾸로 하면 첫번째 인자가 `n`인 일반적인 케이스가 되니까 그걸 정규화 하지 못하는 듯(이라는 생각이 드는데, 맞는지 확실하지는 않음. 왜 이부분은 제대로 설명된 문서가 없을까? 내가 멍청해서 설명 안해도 명확한 부분을 잘 이해하지 못하는 건지...)

```Idris
plusReducesO 0 = refl _
plusReducesO (S k) = eq_resp_S (plusReducesO k)
```

일단 인자가 `0`인 경우는 비교적 쉽다. 앞의 예제와 마찬가지로 `0 = plus 0 0`은 `0 = 0`으로 정규화되기 때문에 바로 증명이 된다. 그 다음 패턴은 귀납적 증명에서 `n`일 때 성립하면 `n+1`일 때도 성립한다 를 보이는 부분인 듯. `eq_resp_S` 함수는 라이브러리에 정의되어 있는 함수로 아래와 같다.

```Idris
eq_resp_S : (m=n) -> ((S m) = (S n))
```

타입 그대로인 것 같다. `m=n`일 때 `(S m) = (S n)`임을 보이는 거니까. 여기서 이 함수의 인자로 `plusReducesO k`를 넘겼으니까, 타입을 분석하면 `n = plus n 0 -> (S n) = S (plus n 0)`이 된다.

### Interactive theorem proving



## Provisional Definitions

## Syntax Extensions
