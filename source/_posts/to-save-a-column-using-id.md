---
title: id를 사용하는 컬럼을 저장하려면
date: 2017-08-26 17:54:15
tags:
- ruby
- rails
---

에초에 이런 모델이있었다.

```rb
class Product < ApplicationRecord
end
```

url에 아이디 대신 상품 코드를 표시하고 싶다는 요구사항이 추가 되었다.
뭐 이정도야 껌이지.

```rb
class Product < ApplicationRecord
  def self.find_by_code(code)
    find(code.sub(/\AC0+/, ''))
  end

  def code
    "C#{id.to_s.rjust(7, '0')}"
  end
end
```

코드를 커스터마이징 하고싶다는 요구사항이 추가되었다.
DB에 저장해야하는데 `before_create` 시점에서는 모델의 아이디를 뽑을 수 없다.
저장을 두번 하는게 신경쓰이긴하지만 어쩔수 없지 `after_save` 인가.

```rb
class Product < ApplicationRecord
  after_create :generate_code

  private

  def generate_code
    self.code ||= "C#{id.to_s.rjust(7, '0')}"
    throw(:abort) unless save
  end
end
```

테스트를 돌려보면 워닝이 대량 발생한다.

```bash
DEPRECATION WARNING: The behavior of attribute_changed? inside of after...
.DEPRECATION WARNING: The behavior of attribute_changed? inside of after...
.DEPRECATION WARNING: The behavior of attribute_changed? inside of after...
.DEPRECATION WARNING: The behavior of attribute_changed? inside of after...
...
```

으으.. 이건 `save`안에서 업데이트할 컬럼을 찾기위해 `attribute_changed?`를
부르기 때문이다.  성능에도 안좋고 좀 진지하게 검색해서 `before_create`로 옮기는
대신 아이디를 직접뽑는 방향으로 전환.

```rb
class Product < ApplicationRecord
  before_create :generate_id_and_code

  private

  def generate_id_and_code
    self.id ||=
      self.class.connection
      .select_value("select nextval('#{self.class.sequence_name}')")
    self.code = "C#{id.to_s.rjust(7, '0')}" unless code?
  end
end
```

이제 저장도 한번하고 테스트돌려도 워닝안나온다.
만족
