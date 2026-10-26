from abc import ABC, abstractmethod


# --- Блюда ---
class MenuItem(ABC):
    def init(self, name, base_price):
        self.name = name
        self.base_price = base_price

    @abstractmethod
    def calculate_price(self):
        pass


class Pizza(MenuItem):
    def init(self, name, base_price, size):
        super().init(name, base_price)
        self.size = size

    def calculate_price(self):
        if self.size == "большая":
            return self.base_price * 1.3  # +30%
        return self.base_price


class Drink(MenuItem):
    def init(self, name, base_price, is_ice):
        super().init(name, base_price)
        self.is_ice = is_ice
        if is_ice:
            self.name = name + " со льдом"  # добавляем слово к названию

    def calculate_price(self):
        return self.base_price


class ComboSet(MenuItem):
    def init(self, name, items):
        super().init(name, 0)
        self.items = items

    def calculate_price(self):
        total = 0
        for item in self.items:
            total = total + item.calculate_price()
        return total * 0.85  # скидка 15%


# --- Оплата ---
class PaymentStrategy(ABC):
    @abstractmethod
    def collect_payment(self, amount):
        pass


class CardPayment(PaymentStrategy):
    def init(self, expiry_year):
        self.expiry_year = expiry_year

    def collect_payment(self, amount):
        if self.expiry_year < 2026:
            print("Карта просрочена! Оплата не прошла.")
        else:
            print("С карты списано:", amount)


class CashPayment(PaymentStrategy):
    def init(self):
        pass

    def collect_payment(self, amount):
        print("Оплата наличными:", amount)


# --- Заказ ---
class Order:
    def init(self):
        self.__items = []                # два подчёркивания = приватное
        self.__status = "Формируется"

    def next_step(self):
        if self.__status == "Формируется":
            self.__status = "Готовится"
        elif self.__status == "Готовится":
            self.__status = "Доставляется"
        elif self.__status == "Доставляется":
            self.__status = "Доставлен"

    def get_status(self):
        return self.__status

    def add_item(self, item):
        if self.__status != "Формируется":
            raise Exception("Нельзя добавить блюдо: заказ уже готовится!")
        self.__items.append(item)

    def get_total_cost(self):
        total = 0
        for item in self.__items:
            total = total + item.calculate_price()
        return total

    def pay(self, payment_method):
        payment_method.collect_payment(self.get_total_cost())


# --- Проверка ---
pizza = Pizza("Маргарита", 500, "большая")
cola = Drink("Кола", 100, True)
combo = ComboSet("Обед", [Pizza("Пепперони", 450, "средняя"), Drink("Чай", 80, False)])

order = Order()
order.add_item(pizza)
order.add_item(cola)
order.add_item(combo)

print("Название напитка:", cola.name)
print("Сумма заказа:", order.get_total_cost())

order.pay(CashPayment())
order.pay(CardPayment(2020))
order.pay(CardPayment(2028))

order.next_step()
print("Статус:", order.get_status())

try:
    order.add_item(Drink("Сок", 120, False))
except Exception as e:
    print("Ошибка:", e)
