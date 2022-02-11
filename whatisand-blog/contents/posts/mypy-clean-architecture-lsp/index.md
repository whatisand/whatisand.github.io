---
title: "Python incompatible with supertype 에러와 LSP"
description: "클린 아키택쳐를 공부해야겠다"
date: 2022-02-09
update: 2022-02-09
tags:
  - 아키텍쳐
  - mypy
  - fast api
series: "오늘의 개발일기"
---


## 들어가며
Fast API를 통해 개발중에 상속과 오버라이드를 해야 할 일이 있었습니다.
린팅을 위해 mypy로 검사해보니 이런 오류를 내어 주었습니다.

```
Signature of "create" incompatible with supertype
```

단순히 메소드를 오버라이드 해서 생긴 문제라고 생각하기에는 다른 곳에서 동일한 방법으로 오버라이드한 메소드에서는 에러가 발생하지 않았습니다.

자세히 살펴보니 상속 과정에서 작동하는 오버라이드와 작동하지 않는 오버라이드의 다른점이 있었습니다.
작동하는 것은 슈퍼 클래스에서 정의한 인자의 형태를 그대로 사용하였으나, 작동하지 않는 것은 슈퍼 클래스에서 정의한 인자 이외의 하나를 추가하였습니다.

왜 그럴까 궁금증이 생겨 문서를 찾아보게 되었습니다.


## 왜 에러가 나왔는가?
mypy 공식 문서에 보면 Liskov substitution principle에 위배되는 행위는 안된다고 합니다. 무엇인지 몰라 예시를 먼저 보았습니다.

``` python

from typing import Sequence, List, Iterable

class A:
    def test(self, t: Sequence[int]) -> Sequence[str]:
        ...

class GeneralizedArgument(A):
    # A more general argument type is okay
    def test(self, t: Iterable[int]) -> Sequence[str]:  # OK
        ...

class NarrowerArgument(A):
    # A more specific argument type isn't accepted
    def test(self, t: List[int]) -> Sequence[str]:  # Error
        ...

class NarrowerReturn(A):
    # A more specific return type is fine
    def test(self, t: Sequence[int]) -> List[str]:  # OK
        ...

class GeneralizedReturn(A):
    # A more general return type is an error
    def test(self, t: Sequence[int]) -> Iterable[str]:  # Error
        ...

```

예시를 보니 무언가 감이 잡혔습니다.
- 인자로는, 부모 클래스에서 정의한 것보다 더 세부적인 것을 정의할 수 없다.
- 반환값으로는, 부모 클래스에서 정의한 것 보다 더 제너럴하게 정의할 수 없다.

아직 직관적으로 이해가 되지는 않았습니다. ‘부모에서는 추상적으로 설계하고 자식이 디테일하게 설계하는게 맞지 않나?’ 라는 생각이 들었습니다.


## Liskov substitution principle
> 확장에 대해서는 열려 있어야 하고, 변경에 대해서는 닫혀 있어야 한다.

클린 아키텍쳐에서 설명하는 주요 원칙중 하나입니다. 각 객체들을 사용하는 것은 개발자간의 약속입니다. 상속받은 객체들 역시 같은 메소드에서는 같은 역할을 해야 합니다.
추상화된 부모 클래스의 제약조건을 따라 개발하면, 그것을 상속받아 구체화되어 구현된 것들을 그대로 사용할 수 있어야 합니다.

맞는 예시인지는 모르겠습니다만, SQLAlchemy를 쓸때 어떤 db를 사용하는지에 따라 내부 로직은 변화가 있겠지만 실제로 사용하는 개발자 입장에서는 추상화된 조건에 맞추어 사용하기만 하면 되는 것과 유사합니다. SQLAlchemy 에서 새로운 db를 지원하게 되었다고 해서 사용법이 달라지지 않게 하기 위해서는 LSP가 지켜져야 합니다.


### 내 코드의 문제점
아래는 저의 문제가 되는 코드입니다.

부모 클래스의 메소드 정의
```python
...
def create(self, db: Session, *, obj_in: CreateSchemaType) -> ModelType:
        ...
...
```

에러난 자식 클래스의 메소드 정의
```python
...
 def create(
        self, db: Session, *, obj_in: ScheduleBlockCreate, user_id: str
    ) -> ScheduleBlock:
        ...

...
````


저희 코드에서 예시를 보면, 유저를 생성할때 전달해야 하는 인자는 유저 dto인데, 타임테이블을 생성할때는 타임테이블의 dto에 더해 유저 아이디를 따로 전달해야 하는 형태로 구현되어있었습니다.

이렇게 적고 다시 보니 제 코드가 이상하게 구현된 것이 맞습니다.

개발 과정에서는 당연히 다른 객체를 생성하니까 메소드의 형태가 다를 수 있다고 생각한 모양입니다. 
하지만, 특정 모델을 db에 생성하는 로직에서 특정 dto만 받기로 추상화되어 약속되어 있었는데 갑자기 다른 것을 달라고 하면 해당 메소드를 이용하는 개발자들은 헷갈릴 것입니다. 지금이야 저 혼자 개발하고 있지만 나중에 저는 고통받을 것이 뻔합니다.

이런 관점에서, 반환하는 값에 대해서도 제한을 둔 LSP 철학이 이해가 되었습니다. 원래 생성한 이후에는 db에서 생성된 값을 dto 형식대로 반환해주기로 약속되어 있었습니다. 그런데 어떤 엔티티 형식에서는 뜬금없이 다른 형태로 반환해준다면 개발자는 별도 로직을 구현해야 할 것입니다.


## 해결하기
공식 문서에서는 타입 보장이 크게 필요없다면  `# type: ignore[override]` 를 통해 에러를 없앨 수 있다고 힙니다.
다만 LSP에 대해서 알게 되었으니 LSP를 지키도록 아키텍쳐를 변경하고자 고민했습니다.

지금 코드의 문제는, timetable을 생성하기 위해 사용하는 dto 외의 다른 값을 받아야 한다는 것이였습니다.

**해결방법 후보**
1. 슈퍼 클래스에서 사용하는 타입에 맞도록 데이터 타입 스키마를 수정하기
2. 메소드를 무리하게 상속받지 않고 새로 만들어 사용하기

dto 형식을 수정하는 것이 옳다고 생각했습니다. Timetable을 생성하기 위해서는 user_id가 필요하니 Timetable을 생성하기 위한 dto에 user_id를 포함하면 메소드를 상속받지 않아도 됩니다. 
문제는, 제가 개발해야 하는 타임테이블 생성 api는 user_id를 body로 받지 않고 비즈니스 로직단에서 입력해야 한다는 점이였습니다. 

Fast API에서는 정의한 dto 스키마대로 api 명세와 문서를 자동으로 정의해줍니다. dto를 수정하면 API 명세가 바뀌기 떄문에 일단 메소드명을 수정하기로 했습니다. 결국 Api Endpoint를 개발하는 코드와 종속성이 생길 가능성이 생겼습니다. 클린 아키텍쳐를 지키기 위해 클린 아키텍쳐를 위반할 여지를 남겨두게 되었습니다.

이렇게 개발하고 보니 전반적인 아키텍쳐가 잘못 설계된 점이 많다는 것을 느꼈습니다. 모를때는 편했는데 알게 되니 코드가 마음에 들지 않습니다. 처음부터 알았으면 좋았을텐데 구현에 집중하다보니 놓친 부분이였습니다.

요즘 클린 아키텍쳐에 대한 필요성을 몸소 느끼고 있습니다. 전에 공부할때는 한번 쓱 훑고 넘겼는데 이제서야 머리에 쏙쏙 들어옵니다.

모르는 것이 생겼다는 슬픔은 새롭게 알게되었다는 기쁨으로 넘기고 공부해야겠습니다.

## 참고자료
- [Common issues and solutions — Mypy 0.931 documentation](https://mypy.readthedocs.io/en/stable/common_issues.html#incompatible-overrides)
- [oop - What is an example of the Liskov Substitution Principle? - Stack Overflow](https://stackoverflow.com/questions/56860/what-is-an-example-of-the-liskov-substitution-principle)