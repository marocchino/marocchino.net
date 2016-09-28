---
title: Set MIX_ENV in mix task
date: 2016-09-28 11:30:46
tags:
- elixir
- mix
---

가끔 테스트 명령이 여럿 필요할 때가 있다.
커버리지 설정을 넣고 싶다던가.
환경변수를 바꿔서 실행해야 한다던다..

Mix에는 이런경우 간단히 추가할 수있는 alias가있다.

```elixir
defmodule Handsup.Mixfile do
  use Mix.Project

  def project do
    [...
     aliases: aliases(),
     ...]
  end

  defp aliases do
    ["test.setup": "ecto.create",
     "test": ["credo --strict", "ecto.migrate", "test"],
     "t": "test"]
  end
end
```

좋다. 이제 실행시켜보면 잘되겠지 싶었는데 dev로 실행시켜 버린다.

```bash
$ mix t
...
** (Mix) "mix test" is running on environment "dev". If you are running tests
along another task, please set MIX_ENV explicitly
```

프라이빗 함수를 만들어 알리아스 체인 앞에 넣는다던가 이것 저것 시도해봤지만 뒤의
테스크에 영향을 줄 수 없어서 방치하고 있었는데 전용 옵션이 따로있었다.

```elixir
  def project do
    [...
     aliases: aliases(),
     preferred_cli_env: preferred_cli_env()
     ...]
  end

  defp preferred_cli_env do
    ["t": :test,
     "test.setup": :test]
  end
```

이제 실행해보면 테스트환경에서 잘 실행된다.

# Ref

<http://elixir-lang.org/docs/stable/mix/Mix.Task.html#preferred_cli_env/1>
