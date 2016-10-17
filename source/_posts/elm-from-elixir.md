---
title: elm from elixir
date: 2016-10-17 10:25:02
tags:
- elm
- elixir
---

이 글은 뽐뿌 글이 아니니 엘름의 좋은 점을 알고싶다면  [다른
글](http://bestalign.github.io/2015/11/28/elm-for-javascript-developers/index.html)을
읽으세요.

# 연산자

조금 다르긴한데 많이 신경쓰이는 정도는 아니다.
몇가지 주목할 부분은..

- `!=`대신 `\=`을 사용한다.
- and or가 없다.
- `||` `&&` 도 Bool 타입만 받아서 엘릭서에서 인라인 if 대신 사용하던
  `valid or raise Error` 를 사용할 수 없다.
- `/` 는 Float를 반환한다.
- `div/2` 는 `//` 로 `rem/2`은 `%`로 사용할 수 있다.

# 함수

basic 문서에 파이프가  없어서 없구나 싶었는데 그냥 메서드에 있었다 .
엘릭서에는 없던 파이프가 몇개 더 있다. 커링에 더 특화되어 있는 느낌.

# 강타입과 컴파일

아.. 정말 여태까지 이렇게 채크해주는 언어를 해본적이 없어서 정말 미쳐버리는 줄
알았는데 어느 정도 익숙해졌다. 일단 함수 선언 위에 타입을 전부 적어야 하고 이게
실제 리턴 값과 다르면 컴파일이 안된다.
처음에는 타입 맞추느라 삽질많이 했는데 익숙 해지니 그냥 저냥 할 만 하다. 루비나
엘릭서에서는 파이프연결해 중간값 확인하면서 코딩했었는데 최종 타입 맞지않으면
실행이 안되니 그런식으로 코딩하기 힘들어졌다. 일단 제일 많이 삽질했던 두 개만
언급하고 넘어가자.

## Maybe XX

그냥 해당 타입이거나 Nothing을 반환 하는데 성공 했을때 (Just XX) 실패 했을때
(Nothing)으로 나온다 map같은 이터레이터에 먹일 때 보통 이런 상태 말고 값만
있을거라 예상하는데, 그렇지 않으니 처리하기 좀 번거롭다. 난 보통 케이스로
처리한다.

```elm
case a of
    Just x -> x
    Nothing -> Debug.crash("error")
```

뭐.. 당연한 이야기지만 결과가 nil이 아니므로

```elixir
value || default
```

같은 식으로는 쓸 수 없다. 이렇게 해야한다.

```elm
Maybe.withDefault 100 (Just 42)   -- 42
Maybe.withDefault 100 Nothing     -- 100
```

이터레이터 안에서 value만 뽑고 싶을 때에는 이렇게 하면된다.
(내가 할 줄 몰라서) 좀 괴로웠다.

```elm
modifyList : List -> Maybe Nothing (List a)
modifyList list =
    list
        |> some_maybe_methods
        |> List.foldr (Maybe.map2 (::)) (Just [])
```

## Result

성공했을 때 (Ok value), 실패했을 때 (Err reason)이 나온다.
이녀석도 케이스로 처리해야 한다. 그거말고는 Maybe랑 비슷하게 하면 된다.

```elm
case a of
    Just x -> x
    Nothing -> Debug.crash("error")
```

# 파이프 순서

엘릭서와 다르게 제일 뒤에 있는게 사용되는데,  덕분에 가독성에서 좀 손해를
보는 느낌이다. 예를 들자면 적어도 나에게는

```elm
String.contains? "elixir of life", "of"
```

보다는

```elm
String.contains “of” “elm tree of life”
```

가 읽기 힘들다.

```elm
"elm tree of life" |> String.contains "of"
```

라고 적을 수 있긴하다. 실제로 그렇게 적을지는 또 다른 문제지만,

라고 적으면 나쁜 점만 있는것 같지만.. 사실 커링 때문에 더 편해진 것도 많다.
이런 상황을 생각해보자.

```elixir
def add(a, b), do: a + b

def add_all(list, num) do
  list
  |> Enum.map(&add(&1, num))
end
```

엘릭서에서는 아리티가 일치하지 않으면 외부 참조를 넘길수 없어서 이런
레핑이 최선이었다.

하지만 엘름은 다르다.

```elm
add : Int -> Int -> Int
add a b =
    a + b

addAll : Int -> List a -> List a
addAll num list =
    let
        plusNum = (add num)
    in
        list
            |> List.map plusNum
```

인자가 불완전하게 넘겨진 상태도 함수이므로 그걸 그대로 함수 인자로 넘길 수 있다.

# 패턴 매칭

모든 조건을 커버 하지 못하면 컴파일이 안된다. 특히 리스트가 컴파일 시점에서
갯수까지는 알수 없기때문에 let에는 디스트럭쳐링 할 수 없고 case를 사용해야 한다.

```elixir
[a, b, c] = list |> Enum.sort
```

그래서 위에 있는 엘릭서에서는 간단히 되던 코드가 이렇게 되버렸다.

```elm
case (List.sort list) of
    a::b::c::[] -> ...
    _ -> Debug.crash("Impossible")
```

with도 없고 리스트 몇개만 매칭으로 풀어도 case중첩으로 산으로 가서
타입에따라서는 그냥 없다고 생각하고 작성하는게 편할 수도 있다. 패턴 매칭이
약해서 if else 구문을 평범하게 사용하는것도 다른 문화.

# 기타

- List의 렌덤 억세스에 관한 함수가 없다. 굳이 하고 싶으면 어레이로 변환해서
  해야한다. 근데 변환 여러번 하는 거보다 head tail로 어찌어찌하는게 비용이 적을
  것 같다.
- 요즘은 어떨지 모르겠는데 옛날에 자바 싫어하는 이유 중 하나가 정규식 쓰기
  힘들어서 였는데 , 정규식용 리터럴이 따로 준비되어있지 않고 스트링으로 해서
  자바에서 정규식 쓰던 생각난다.

# 결론

단순하게 되던게 번거로워 진 부분이 좀 많아서 생산성만 좀 더 뽑을 수 있으면 좋을
것 같다는 생각을 했다.
