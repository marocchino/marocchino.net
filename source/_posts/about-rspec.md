---
title: rspec 잘쓰고 계신가요?
date: 2016-12-04 10:00:00
tags:
- ruby
- rspec
- bdd
---

이 글은 [루비 대림 달력](http://ruby-korea.github.io/advent-calendar/)용으로
작성하는 글 입니다.

- [3일: Ruby 2.4.0 Preview](http://riseshia.github.io/2016/12/01/ruby-2-4-0-preview.html)
- [5일: puma 웹서버 주기적으로 재시작](https://code.iamseapy.com/archives/38)

tl;dr: Given, When, Then 단위로 나누어서 작성하세요.

# rspec은

설명할 것도 없이 레일스를 사용한다면 한번은 봤을 법한 가장 대중적인 BDD테스트
프레임 워크입니다.

# BDD는 뭔가요

사양을 기술에 집중하는 TDD의 확장입니다. 루비에서는 rspec말고도 minitest-spec,
cucumber, rspec-feature등을 사용해 할 수도 있습니다. 특정 서브젝트에 대해 조건을
주고 그 결과를 확인하는 3단계로 나누어 작성하는게 특징입니다. 에러를 읽기도 쉽고
찾기도 쉽죠.

# 코드로 이야기 합시다

다음과 같은 사양서의 테스트를 작성해 봅시다.

```cucumber
기능: Stack

조건 새 스택을 만듬
그러면 비어있음

만일 스택에 요소가 추가됨
그러면 그 요소가 스택의 제일 위에 위치함

만일 스택이 N개의 요소를 가짐
그리고 요소 E가 스택의 제일 위에 위치함
그러면 팝 연산은 E를 반환함
그리고 새 스택 크기는 N-1이 됨
```

먼저 minitest로 작성해 보면 이렇게 될 것 같습니다.

```ruby
class StackTest < Minitest::Test
  def setup
    @stack = Stack.new
  end

  def test_new
    assert @stack.empty?
  end

  def test_add
    @stack.add("A")
    assert_equal "A", @stack.top
  end

  def test_pop
    elements = %w(A B C D E)
    elements.each { |e| @stack.add(e) }
    assert_equal "E", @stack.pop
    assert_equal 4, @stack.size
  end
end
```

일단 있는 그대로 rspec문법으로 옮겨보죠.

```ruby
RSpec.describe Stack do
  subject(:stack) { Stack.new }

  it ".new" do
    is_expected.to be_empty
  end

  it "#top" do
    stack.add("A")
    expect(stack.top).to eq "A"
  end

  it "#pop" do
    elements = %w(A B C D E)
    elements.each { |e| stack.add(e) }
    expect(stack.pop).to eq "E"
    expect(stack.size).to be 4
  end
end
```

TDD일때는 문제가 안되지만, 이런 코드는 설명을 코드의 구현에 의존하기 때문에
설명충이 미덕인 BDD로써는 좋은 코드가 아닙니다. 단계별로 개선해 봅시다.

## describe 사용하기

사양서의 Given에 해당하는 부분이고 테스트 할 대상을 지정할 때 사용하는 키워드
입니다. 이미 Stack이 describe 되어있긴 하지만, 관례대로 클래스명 -> 메서드명의
두 단계로 넣어 무엇을 태스트하는지 좀 더 명확하게 하겠습니다.

```ruby
RSpec.describe Stack do
  subject(:stack) { Stack.new }
  describe ".new" do
    it "비어 있음" do
      expect(stack.top).to eq "A"
    end
  end

  describe "#top" do
    it "만일 스택에 요소가 추가됨 그러면 그 요소가 스택의 제일 위에 위치함" do
      stack.add("A")
      expect(stack.top).to eq "A"
    end
  end

  describe "#pop" do
    it "만일 스택이 N개의 요소를 가짐 그리고 요소 E가 스택의 제일 위에 위치함" \
       "그러면 팝 연산은 E를 반환함 그리고 새 스택 크기는 N-1이 됨" do
      elements = %w(A B C D E)
      elements.each { |e| stack.add(e) }
      expect(stack.pop).to eq "E"
      expect(stack.size).to be 4
    end
  end
end
```

아직 설명이 너무 길군요. 조금 더 분해해 봅시다.

## context 사용하기

조건을 정의할 때 사용합니다. 사양의 만일(When)에 해당하는 부분이죠.
중요한 포인트는 context와 before를 하나의 묶음 처럼 생각하는 것입니다.

```ruby
  describe "#top" do
    context "만일 스택에 요소가 추가됨" do
      before { stack.add("A") }
      it "그러면 그 요소가 스택의 제일 위에 위치함" do
        expect(stack.top).to eq "A"
      end
    end
  end

  describe "#pop" do
    context "만일 스택이 N개의 요소를 가짐 그리고 요소 E가 스택의 제일 위에 위치함" do
      before { %w(A B C D E).each { |e| stack.add(e) } }
      it "그러면 팝 연산은 E를 반환함 그리고 새 스택 크기는 N-1이 됨" do
        expect(stack.pop).to eq "E"
        expect(stack.size).to be 4
      end
    end
  end
```

## 부작용 없에기

BDD의 세계에서는 실행 시간을 희생해서라도 부작용을 없에고 싶어합니다. it 하나에
expect하나 이상 사용하는것은 좋지않은 징후죠. 나누어 보겠습니다.

```ruby
  describe "#pop" do
    context "만일 스택이 N개의 요소를 가짐 그리고 요소 E가 스택의 제일 위에 위치함" do
      before { %w(A B C D E).each { |e| stack.add(e) } }
      it "그러면 팝 연산은 E를 반환함" do
        expect(stack.pop).to eq "E"
      end

      it "그러면 팝연산 후의 새 스택 크기는 N-1이 됨" do
        stack.pop
        expect(stack.size).to be 4
      end
    end
  end
```

이제 더 할일이 없어보이네요.

## 일단 완성

전체 코드는 이렇습니다.

```ruby
RSpec.describe Stack do
  subject(:stack) { Stack.new }
  describe ".new" do
    it "비어 있음" do
      expect(stack.top).to eq "A"
    end
  end

  describe "#top" do
    context "만일 스택에 요소가 추가됨" do
      before { stack.add("A") }
      it "그러면 그 요소가 스택의 제일 위에 위치함" do
        expect(stack.top).to eq "A"
      end
    end
  end

  describe "#pop" do
    context "만일 스택이 N개의 요소를 가짐 그리고 요소 E가 스택의 제일 위에 위치함" do
      before { %w(A B C D E).each { |e| stack.add(e) } }
      it "그러면 팝 연산은 E를 반환함" do
        expect(stack.pop).to eq "E"
      end

      it "그러면 팝연산 후의 새 스택 크기는 N-1이 됨" do
        stack.pop
        expect(stack.size).to be 4
      end
    end
  end
end
```

보시는 것 처럼 TDD 스타일에 비해 자연어에 가깝게 적으려는 노오오력이 많이
필요합니다. 하지만 내부 코드를 몰라도 단계적으로 조건의 설명이 명확히 되는건
장점이라 할 수 있죠. 저는 이정도로 만족합니다만, 좀 더 bdd사양에 가깝게
작성하시고 싶으시면 rspec-given이라는 dsl이 있긴 합니다.

## rspec-given

```ruby
RSpec.describe Stack do
  Given(:stack) { Stack.new }
  describe ".new" do
    Then "비어 있음" do
      expect(stack.top).to eq "A"
    end
  end

  describe "#top" do
    context "만일 스택에 요소가 추가됨" do
      When { stack.add("A") }
      Then "그러면 그 요소가 스택의 제일 위에 위치함" do
        expect(stack.top).to eq "A"
      end
    end
  end

  describe "#pop" do
    context "만일 스택이 N개의 요소를 가짐 그리고 요소 E가 스택의 제일 위에 위치함" do
      When { %w(A B C D E).each { |e| stack.add(e) } }
      Then "그러면 팝 연산은 E를 반환함" do
        expect(stack.pop).to eq "E"
      end

      Then "그러면 팝연산 후의 새 스택 크기는 N-1이 됨" do
        stack.pop
        expect(stack.size).to be 4
      end
    end
  end
end
```

큐컴버 만큼은 아니지만, 어느정도 정돈되어 보이네요.

# 결론

bdd에 충실하게 rspec을 작성하는 법을 알아 보았습니다. 일정이 바쁘다던가
사양이 복잡하던가 이러저러한 이유가 있긴하겠지만
[DHH도 불평](http://www.rubyinside.com/dhh-offended-by-rspec-debate-4610.html)
못하게 깔끔하게 작성하도록 노력해봅시다. :)

# 참고

- <http://www.chrisdpeters.com/introduction-feature-specs-rspec/>
- <https://cucumber.io/>
- <https://github.com/rspec-given/rspec-given>
- <http://betterspecs.org/ko/>
- <https://en.wikipedia.org/wiki/Behavior-driven_development>
