---
title: Elixir tips from exercism
date: 2016-09-20 00:00:00
tags: elixir
---

# 재귀

다른 언어하다와서 성능이 안좋을거라는 선입견에 가능하면 안쓰려고 했는데 괜한
걱정이었다. 오히려
[`Enum`](http://elixir-lang.org/docs/stable/elixir/Enum.html#flat_map/2)
사용하는 쪽이 느릴 때가 많았다.

# for 문

[`Enum.flat_map/2`](http://elixir-lang.org/docs/stable/elixir/Enum.html#flat_map/2)
대신 쓸 수 있다.

```elixir
# bad
min..(max - 2)
|> Enum.flat_map(fn x ->
  (x + 1)..(max - 1)
  |> Enum.flat_map(fn y ->
    (y + 1)..max
    |> Enum.map(fn z ->
      [x, y, z]
    end)
  end)
end)
|> Enum.filter(&pythagorean?/1)

# good
for x <- min..(max - 2),
    y <- (x + 1)..(max - 1),
    z <- (y + 1)..max,
    pythagorean?([x, y, z]) do
  [x, y, z]
end
```

# 인자 개수

줄일 수 있으면 줄여라. 짧고 읽기도 편하다.

```elixir
# bad
def upto(0, acc), do: acc
def upto(n, acc \\ []), do: upto(n - 1, [n | acc])

# good
def upto(0), do: []
def upto(n), do: [n | upto(n - 1)]
```

# List

링크드 리스트는 배열과 다르다. 루비에서는 가능하면 배열을 적게 사용하려고
노력했었는데 그럴 필요는 없다. 다만 zip한다던가 할게 아니면 튜플이 나을 때도
있다.

# with

갓갓돋는다. BDD감각으로 읽힌다.

# & Shorthand

&로 함수 시작해서 인자 넘기는 패턴 밖에 몰랐는데, 인자도 생략할 수 있더라.

```elixir
~w(1 2 3)
|> Enum.map(&String.to_integer(&1))

# equals to
~w(1 2 3)
|> Enum.map(&String.to_integer/1)
```

# transpose

이게 왜없나 싶은데.. 만들 수는 있다.

```elixir
def transpose([[]|_]), do: []
def transpose(list) do
  [Enum.map(list, &hd/1) | transpose(Enum.map(list, &tl/1))]
end

# or
def transpose(list) do
  list |> List.zip |> Enum.map(&Tuple.to_list/1)
end
```

# 디스트럭쳐링

패턴 매칭이라 기본적으로 되는데 튜플은 개수 맞춰야하는 거만 좀 조심하면 된다.
맵이 좀 적기 귀찮은데 신텍스 슈가 같은게 있으면 좋겠다. 있는데 나만 모르는 걸
수도 있고..

```elixir
[a, b, c] = [1, 2, 3]
[a | _] = [1, 2, 3]

{a, b, c} = {1, 2, 3}
{a, _, _} = {1, 2, 3}
{a, _} = {1, 2, 3} # error

%{a: a, b: b} = %{a: 1, b: 2}
%{a: a} = %{a: 1, b: 2}
```

# state

루비에서는 간단하게 돼는데 할일이 많아졌다. 요런 코드가있다고 하자.

```ruby
class Counter
  def initialize
    @number = 0
  end

  def up
    @number += 1
  end
end
```

이걸 엘릭서에서 하려면 이렇게 된다.

```elixir
defmodule Counter do
  use GenServer

  def handle_call(:up, _from, state) do
    {:reply, state + 1, state + 1}
  end
end

{:ok, pid} = GenServer.start_link(Counter, 0)
GenServer.call(pid, :up)
```

뭔가 더 단순하게 할 수있는 방법이 있었으면 좋겠다.

# 그밖에

가능하면 파이프써라.

```elixir
# bad
[h] ++ t

# good
[h | t]
```

까먹을 때가 가끔있는데 List.Chars.t는 Enum.t다.
그냥 홀따옴표 표기로 적어도 Enum돌릴 수 있으니 길게 풀어쓸 필요는 없다.

```elixir
# bad
?A in [?A, ?B, ?C]
[?A, ?B, ?C]
|> Enum.all?(&(&1 < ?D))

# good
?A in 'ABC'
'ABC'
|> Enum.all?(&(&1 < ?D))
```

`to_integer(float)`가 없다. 다들
[`round(Number)`](http://erlang.org/doc/man/erlang.html#round-1) 사용하는 듯
하다.

스트링을 파이프로 넘겨야 할 때는
[`Regex.replace/4`](http://elixir-lang.org/docs/stable/elixir/Regex.html#replace/4)
대신
[`String.replace/4`](http://elixir-lang.org/docs/stable/elixir/String.html#replace/4)
쓰면 된다. 인자 순서만 다르고 기본적으로 같은 함수다.

# 수정 이력

9월 23일: 오타 수정, 링크 추가
9월 28일: 항목 추가
