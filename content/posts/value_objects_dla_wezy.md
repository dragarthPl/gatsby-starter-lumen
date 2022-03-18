---
title: Value Objects dla węży date: "2022-03-18T22:00:00.169Z"
template: "post"
draft: false slug: "value-objects-dla-wezy"
category: "DDD"
tags:

- "DDD"
- "Architecture"
- "Patterns"
  description: "Zbiór wiedzy o Value Objects oraz przykłady w Pythonie"

---

- [Po co mi właściwie te welju obdżekty](#Wstep-po-co-mi-wlasciwie-te-welju-obdzekty)
- [Czym są Value Objects i jakie mają znaczenie strategiczne](#czym-sa-value-objects-i-jakie-maja-znaczenie-strategiczne)
- [Primitive Obsession](#primitive-obsession)
- [Immutable](#immutable)
- [Side Effect](#side-effect)
- [Comparasion](#comparasion)
- [Przykład](#przyklad)
- [Co możemy zyskać, a co stracić](#co-mozemy-zyskać-a-co-stracic)

# Po co mi właściwie te welju obdżekty

Te Value Objects to po co nam są potrzebne, tylko więcej się narobię przy nich. W końcu wstawię sobie tutaj inta, dorzucę plusika i mam, a nie będę tworzyć jakieś metody na dodawanie. Po co mi to ? Jeszcze może typ miałbym zadeklarować przy tym dodawaniu ? Na koniec jeszcze zwrócę nowy taki, niezmienialny - przecież to pamięć i cykle procesora zje. To się nie godzi pisać taki niewydajny kod. 

## Czym są Value Objects i jakie mają znaczenie strategiczne

Wyobraź sobie portfel, w którym masz różne banknoty. I tak w środku masz 10 PLN, 5 USD, 13 GBP. Spróbujmy opisać trochę świat portfela w kodzie ... to jak to będzie:

```
1000, 500, 1300
```

Trochę nawet pasuje, będzie to takie szybkie no i oczywiście sprytne. Lecz czy zastanawiałeś się czy będzie dla innych oczywiste. To może zapomnijmy na chwilę o chwale i fam farach dla wielkiego guru kodu wydajnego kodu. Zrobimy powolny kod, który dołoży kilka taktów procesora w rytmie żółtej kaczy, ale za to też ku chwale innego rodzaju. Chwale wielkich pisarzy niczym _Gall Kodim_, czy też _Gil py Kodenberg_, albo _Ada Kodinger_. Mówiąc, że pieniądz którym płacimy składa się z wartości i waluty uciekamy od innego wymiaru rzeczywistości, która dla przykładu zawiera dowolnie interpretowalny świat stringow, intigerów i floatów. Tak oto odzieramy naszą wyobraźnie z możliwości płacenia -3 PLN, mówimy innym jak to należy dodać albo odjąć - tak by panował ład i porządek. 

Mówiąc w prostym ujęciu o Value Object - mówimy sobie o takim konstrukcie, który wyraża intencję bytu. Ten byt wyposażamy w arsenał innych przyległych mu opisów jego świata, gdzie znajdziemy:
- projekcja:
  - to_string
  - format_sth
- z czego mam możliwość to zrobić of(), from_int()
- jak na tym możemy działać add(), increment()
- opisujemy granice:
  - może być ujemne
  - ma limit
  - nie może być "takie"
  - ma być takie

Czego nie będzie ? Identyfikacji, Value Object opisują mierzalną, wyliczalną lub opisową wartość. I tak pieniądz może być VO, lecz nie zawsze. Jeśli dołożymy do niego numer seryjny już nim nie będzie.

## Primitive Obsession

Tytułowy code smell, jakim jest primitive obsession to zjawisko nadużywania typów
prostych do reprezentacji idei/konceptu domeny. Przykłady: używamy liczby całkowitej do reprezentowania kwoty pieniężnej
zamiast obiektu Money - czyli struktury pieniężnej, albo stringa, aby prezentować wiadomość.

## Immutable

Niezmienność jest cechą, która opisuje Value Objects. Ta niezmienność VO, zapewnia nam gwarancje na poziomie kodu, że są
to obiekty read-only. Jeśli zachodzi potrzeba zmiany wartości wyrażonej za pomocą VO, musimy tego dokonać przez
utworzenie nowego VO.

## Side Effect

Chodzi o to aby uniknąć zmiany obiektu "na boku" np. po przez money.value = money.value + 3, zamiast tego chcemy mieć
nowy byt immutable, czyli Money(money.value + 3, currency).

```Python
from decimal import Decimal


class Money:
    value: Decimal
    currency: str
    
    def __init__(self, value: Decimal, currency: str):
        self.value = value
        self.currency = currency
    
    def add(self, money: Money) -> 'Money':
        if self.currency == money.currency
            return Money(self.value + money.value, money.currency)
        raise ValidationError("Different currency")

```

Czemu to takie ważne ? Przypuśćmy że mamy taka sytuację:

```
>>> money = Money(Decimal("5"), "PLN")

>>> currency_conversion(money, "USD")
7 USD

>>> money.value
7

>>> money.currency
USD

```

Przykład pokazuje idealnie jak błędy w currency_conversion, są siane w głąb naszego kodu.

## Comparasion

Musimy też pamiętać o tym aby zaimplementować mechanizm porównania naszych VO, zrezygnowaliśmy z prymitywnej prezentacji
naszego konceptu biznesowego, dlatego musimy opisać proces porównania dwóch VO.

## Przykład

```Python
from dataclasses import dataclass
from decimal import Decimal


@dataclass(frozen=True)
class Money:
    __value: Decimal
    __currency: str

    @staticmethod
    def from_int(value: int, currency: str) -> 'Money':
        return Money(Decimal(value)/100, currency)

    @staticmethod
    def from_float(value: float, currency: str) -> 'Money':
        return Money(Decimal(value), currency)

    def percentage(self, percentage: int) -> 'Money':
        return Money(Decimal(percentage * self.__value / Decimal("100.0")), self.__currency)

    def to_int(self) -> int:
        return int(self.__value * 100)

    def to_float(self) -> float:
        return float(self.__value)

    def add(self, other: 'Money') -> 'Money':
        if self.__currency == other.__currency:
            return Money(self.__value + other.__value, self.__currency)
        raise AttributeError("Different currencies")

    def subtract(self, other: 'Money') -> 'Money':
        if self.__currency == other.__currency:
            return Money(self.__value - other.__value, self.__currency)
        raise AttributeError("Different currencies")

    def __eq__(self, other: 'Money') -> bool:
        return self.__value == other.__value and self.__currency == other.__currency

    def __str__(self) -> str:
        return f"{self.__value:.2f} {self.__currency}"

```

Testy:

```Python
from unittest import TestCase
from money import Money


class TestMoney(TestCase):

    def test_can_create_money_from_integer(self):
        self.assertEqual("100.00 USD", str(Money.from_int(10000, "USD")))
        self.assertEqual("0.00 PLN", str(Money.from_int(0, "PLN")))
        self.assertEqual("10.12 EUR", str(Money.from_int(1012, "EUR")))

    def test_should_project_money_to_integer(self):
        self.assertEqual(10, Money.from_int(10, "USD").to_int())
        self.assertEqual(0, Money.from_int(0, "USD").to_int())
        self.assertEqual(5, Money.from_int(5, "USD").to_int())

    def test_should_project_money_to_float(self):
        self.assertEqual(10.10, Money.from_int(1010, "USD").to_float())
        self.assertEqual(0.12, Money.from_int(12, "USD").to_float())
        self.assertEqual(5.11, Money.from_int(511, "USD").to_float())

    def test_can_add_money(self):
        self.assertEqual(Money.from_int(1000, "USD"), Money.from_int(500, "USD").add(Money.from_int(500, "USD")))
        self.assertEqual(Money.from_int(1042, "USD"), Money.from_int(1020, "USD").add(Money.from_int(22, "USD")))
        self.assertEqual(Money.from_int(0, "USD"), Money.from_int(0, "USD").add(Money.from_int(0, "USD")))
        self.assertEqual(Money.from_int(6, "USD"), Money.from_int(4, "USD").add(Money.from_int(2, "USD")))

        self.assertRaises(AttributeError, lambda: Money.from_int(3, "PLN").add(Money.from_int(2, "USD")))

    def test_can_subtract_money(self):
        self.assertEqual(Money.from_int(0, "USD"), Money.from_int(50, "USD").subtract(Money.from_int(50, "USD")))
        self.assertEqual(Money.from_int(998, "USD"), Money.from_int(1020, "USD").subtract(Money.from_int(22, "USD")))
        self.assertEqual(Money.from_int(1, "USD"), Money.from_int(3, "USD").subtract(Money.from_int(2, "USD")))

        self.assertRaises(AttributeError, lambda: Money.from_int(3, "PLN").subtract(Money.from_int(2, "USD")))

    def test_can_calculate_percentage(self):
        self.assertEqual("30.00 USD", str(Money.from_int(10000, "USD").percentage(30)))
        self.assertEqual("26.40 USD", str(Money.from_int(8800, "USD").percentage(30)))
        self.assertEqual("88.00 USD", str(Money.from_int(8800, "USD").percentage(100)))
        self.assertEqual("0.00 USD", str(Money.from_int(8800, "USD").percentage(0)))
        self.assertEqual("13.20 USD", str(Money.from_int(4400, "USD").percentage(30)))
        self.assertEqual("0.30 USD", str(Money.from_int(100, "USD").percentage(30)))
        self.assertEqual("0.00 USD", str(Money.from_int(1, "USD").percentage(40)))

```

## Co możemy zyskać, a co stracić.

Zyski:
- przejrzyste reguły gry
- odchudzenie utilsów i przyporządkowanie właściwych formaterów do odpowiednich VO
- wysoka enkapsulacja
- testowalność na poziomie unit testów
- mniej kodu przy reużywalności
- wyższa czytelność

Straty:
- wydajność
- trzeba się bardziej namęczyć aby go napisać - czas implementacji