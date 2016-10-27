---
title: Custom matcher in exunit
date: 2016-10-28 04:44:05
tags:
- elixir
- exunit
- test
---

# 동기

필요해서 해볼려고 하니 문서로 설명되어있지 않아서 정리겸..

# 준비

피닉스는 이미 설정 되어있어서 필요없지만 직접 만든 라이브러리라면,
루비에서처럼 require로는 못하고 mix.exs에서 로드 경로를 설정해야 한다.
테스트 환경에서만 로드하게 할려면 이렇게 하면된다.

```elixir
  def project do
  [...,
   elixirc_paths: elixirc_paths(Mix.env),
   ...]
  end

  defp elixirc_paths(:test), do: ["lib", "test/support"]
  defp elixirc_paths(_),     do: ["lib"]
```

설정후에 일단 한번 컴파일 해주자.

```bash
mix compile
```

# 매쳐 작성

내가 필요했던건 정규식으로 스트링을 검증해주는 매쳐다.
test/support 안에 넣어준다. 파일명이 exs가 아니라 ex인것에 주의.

```elixir
defmodule ProjectName.Matcher do
  import ExUnit.Assertions, only: [assert: 2]

  def assert_match(regex, actual) do
    assert Regex.match?(regex, actual), "#{actual} is not match."
  end
end
```

# 사용

평범하게 임포트해서 사용하면 된다.

```elixir
  import ProjectName.Matcher, only: [assert_match: 2]
  test "123 matches \\d+" do
    assert_match ~r/\A\d+\z/, "123"
  end
```

끝.
