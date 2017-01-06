---
title: 묵시는 사악해
date: 2017-01-06 01:11:43
tags:
- ruby
---

다음 코드의 결과를 예측해보세요.

```rb
a = //
# 1
a == //

# 2
a.! == //.!

# 3
!a == !//
```

루비 2.4 기준으로 정답은 1. `true`, 2. `true`, 3. `false` 입니다.

# 해설

루비에서 falsey값은 아시다시피 `nil`과 `false`뿐입니다. `//`같은 정규식 객체가
falsey로 취급되는건 누가봐도 이상하죠. 이것 저것 시도하다 `if`문안에 넣어보니
단서가 될만한 경고를 찾았습니다.

```irb
irb> p 1if //
(irb):2: warning: regex literal in condition
=> nil
```

[스택오버플로우](http://stackoverflow.com/questions/29164490/ruby-why-do-i-get-warning-regex-literal-in-condition-here)에
따르면 정규식 리터럴은 묵시적으로 `Regexp#~`로 해석되고 이는 `$_`에 대해 매칭을
수행하고 결과를 반환합니다. 결국 `!//`는 `! ~//`로 변환되고 `~//`는 `$_`에
들어있는 `nil`에 매칭해 `nil`을 반환 `!nil`은 `true`로 변환되게 됩니다.
`$_`는 변수라 여기에 값을 넣으면 행동이 바뀝니다.

```irb
+ irb(main):004:0* $_ = "하이요"
=> "하이요"
+ irb(main):005:0> //
=> //
+ irb(main):006:0> ~//
=> 0
+ irb(main):007:0> !//
=> false
```

# 결론

어쨋든 이런 묵시적 변환은 가독성을 해칩니다. 이기회에 좀더 현대적인 언어로 갈아
타던가 파서가 해석하는 대로 적어서 오해를 줄이도록 합시다.

```rb
!a != ! ~//

1a != //.match?($_)

text = gets
1a != //.match?(text)
```

끗
