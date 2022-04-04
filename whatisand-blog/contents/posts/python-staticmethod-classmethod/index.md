---
title: "Python @staticmethod @classmethod 왜 사용할까?"
description: "파이썬 스테틱 메소드의 필요성에 대해 고민을 해보았습니다."
date: 2022-02-12
update: 2022-04-04
tags:
  - python
series: "오늘의 개발일기"
---

## 들어가며

파이썬 기초 문법을 다시 살펴보며 턱 걸리는 지점이 있었습니다. 파이썬의 클래스 내에서 정의된 메소드는 첫 인자로 객체 자신의 클래스 인스턴스를 받습니다. 관례상 self로 받는 것을 기억하시면 좋습니다.

클래스 메소드의 첫 인자는 객체 인스턴스가 있어야 한다는 점 떄문에 인스턴스를 생성하지 않고는 메소드를 호출할 수 없습니다. 인스턴스를 생성하지 않고 메소드를 사용할 일이 있을 때 @staticmethod와 @classmethod를 사용한다는 것으로 이해하였습니다.

@classmethod는 인스턴스 대신 클래스에 정의된 것들을 활용할 수 있습니다. 우리가 보통 cls로 받는 그 인자입니다.

그런데 @staticmethod는 아무 것도 받지 않습니다. 그럼 이게 단순히 밖에서 만든 함수와 다른게 무엇인지 의문이 들었습니다.

@staticmethod로 정의한 것은 무엇이 다르고 용도는 무엇일까요?


## @classmethod의 필요성
클래스 메소드는 객체 인스턴스를 수정할 수 없습니다. 대신 클래스에서 직접 호출할 수 있다는 점과 클래스에 직접 접근할 수 있다는 점 때문에 다음과 같이 사용될 수 있습니다.

### 1) 클래스 변수에 접근하여 공통된 변화를 주는 활용

클래스가 공통으로 가진 변수를 참조하고 조작하여 해당 클래스로 만들어진 모든 인스턴스를 일괄적으로 제어할 수 있습니다. 개인적으로는 예측 가능성 측면에서 사용하기 어려울 수 있다고 생각합니다만, 객체라는 특성상 오히려 직관적으로 이해할 수 있습니다.

예를들어, 공통된 이자율을 가진 계좌가 있고 각 계좌는 서로 다른 금액을 가질 수 있습니다.
공통된 이자율이 변경될 때 아래처럼 활용할 수 있습니다.


``` python
class Account:
    interest_rate: float = 0.1  # 이자율

    def __init__(self, amount: int):
        self.amount = amount

    @classmethod
    def change_rate(cls, target_rate: float):
        cls.interest_rate = target_rate

    def give_interest(self):
        self.amount += int(self.amount * self.interest_rate)


if __name__ == "__main__":

    account1 = Account(100000)
    account2 = Account(1000)

    print("account1.amount ", account1.amount)  # 100000
    print("account2.amount ", account2.amount)  # 1000

    account1.give_interest()
    account2.give_interest()

    print("account1.amount ", account1.amount)  # 110000
    print("account2.amount ", account2.amount)  # 1100

    # 공통된 이자율을 20%로 변경
    Account.change_rate(0.2)

    account1.give_interest()
    account2.give_interest()

    print("account1.amount ", account1.amount)  # 132000
    print("account2.amount ", account2.amount)  # 1320
```
   

### 2) 생성자를 랩핑하는 용도로 사용하기 위해

특정 객체를 여러 입력 형태로 생성하게 될 때도 있습니다. 별도 로직을 작성해도 되지만 아래처럼 사용하기 쉽게 클래스 메소드로 구현할 수 있습니다. 클래스 메소드를 설멍할 때 자주 사용하는 예시입니다.

``` python
class BirthDay:

    def __init__(self, birth_year: int, birth_month: int, birth_date: int):
        self.year = birth_year
        self.month = birth_month
        self.date = birth_date

    @classmethod
    def by_security_number(cls, security_number: str):
        # 간단한 예시를 위해 입력은 1900년대로 가정하겠습니다.

        birth_year = 1900 + int(security_number[:2])
        birth_month = int(security_number[2:4])
        birth_date = int(security_number[4:6])

        return cls(birth_year, birth_month, birth_date)


if __name__ == "__main__":
    
    birth1 = BirthDay(1999, 1, 31)
    birth2 = BirthDay.by_security_number("990131")
    
    print(birth1.year)
    print(birth2.year)
```


## @staticmethod의 필요성

그런데, 스테틱 메소드의 필요성에 대해 고민하다 보니 이해하기에 상당히 어려움이 있었습니다. 클래스 안에서 메소드를 만들되, 클래스와 영향이 전혀 없다는 것을 나타내는 것 이상 이하도 아니였습니다. 처음 든 생각은 '이럴거면 클래스 밖에서 정의하면 되지 않나?'였습니다.

### JAVA static method에 대한 고찰
파이썬보다 더욱 깐깐한 자바와 같은 언어에서는 static method를 어떤 시각으로 바라보는지 생각해 보았습니다.
자바에서 정적 메소드의 특징은 다음과 같이 정리될 수 있습니다.

1. 클래스 인스턴스를 만들지 않고 호출할 수 있다.
2. 인스턴스에서는 호출될 수 없다.

정적 메소드는 클래스 이름을 빌려 관련있는 유틸성 합수를 만들때 사용합니다. static이라는 이름이 있는 것 처럼 정적인 상태입니다. 이 메소드는 가변적인 상태를 가지지 않고, 다른 것의 상태도 변경시키지 않습니다.

또한 해당 메소드는 오버라이딩 되지 않습니다. 클래스에서 정의된 것은 바뀌지 않습니다.

항상 동작을 예측할 수 있다는 점에서 인스턴스와의 상호작용을 배제하며 일관된 활용을 보장하는 용도로 사용될 것으로 생각합니다.


## 파이썬에서의 필요성을 잡어보며
문제는, 파이썬에서는 해당 메소드를 인스턴스에서도 호출할 수 있습니다. 심지어 오버라이딩하여 다른 용도가 되어버릴 수도 있습니다.
파이썬에서 @staticmethod는 그냥 장식 이상 이하도 아닌 것 같습니다. 이럴리가 없는데... 고민을 계속 해보았습니다.

답을 찾기 위해 관련 키워드로 검색하자마자 저와 비슷한 생각을 가진 사례가 전 세계적으로 존재하는 것 같은 느낌을 받았습니다. 실제로 관련 논쟁도 있다고 합니다. 단순히 cls와 self 두가지를 쓰지 않는 메소드라고 하기에는 넘어가기 어려워서 나름대로 고민해본 필요성을 적어봅니다.


### 1) 해당 클래스와 명확한 관련성을 표현하기 위해
스태틱 메소드가 클래스와 직접적인 상호작용이 없다고 하더라도 해당 클래스와 명확한 관련성을 명시하는 것이 좋을 때도 있습니다.
유틸성 메소드를 별도 모듈로 빼서 사용하는 것도 좋으나 표현상 해당 메소드가 해당 클래스와 연관되어 사용한다는 사실을 명시하는 것이 좋을 수 있습니다.

### 2) 해당 클래스 내부에서만 사용되는 유틸성 메소드를 표현하기 위해
해당 클래스 내부에서만 사용되는 메소드가 있다고 합시다. 해당 클래스 내부에서만 사용하기 때문에 클래스 내부에 메소드로 만드는 것도 좋은 전략일 것입니다.

적고 보니 충분히 대안이 있습니다. 위에서 찾은 필요성을 달성하기 위해서는 다른 방법도 충분히 있습니다.

파이썬에서 유의미한 활용을 하기 위해서는 다른 언어에서 제한하는 요소들을 적극적으로 따라 통일성을 부여하는 노력이 필요할 것으로 보입니다.

@staticmethod와 @classmethod의 활용법에 대한 여러분들의 의견도 궁금합니다. 파이썬에서 @staticmethod의 필요성에 대해 이야기 해주실 분들은 덧글로 소통해주시면 감사하겠습니다.


## 검색한 문서

- [파이썬 정적(static) 메서드와 클래스(class) 메서드 | Engineering Blog by Dale Seo](https://www.daleseo.com/python-class-methods-vs-static-methods/)
- [The definitive guide on how to use static, class or abstract methods in Python](https://julien.danjou.info/guide-python-static-class-abstract-methods/)
- [정적 메소드를 쓰는 이유가 무엇인가요? | 코드잇](https://www.codeit.kr/community/threads/13116)