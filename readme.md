# Домашняя работа № 1: основы Java, SOLID и исключения

## Цель

Создать простое консольное приложение для оформления заказа и на практике разобраться с:

- классами;
- интерфейсами;
- абстрактными классами;
- композицией;
- принципами SOLID;
- checked- и unchecked-исключениями;
- общей обработкой разных исключений по их базовому типу.

В репозитории находится только пустой каркас проекта. Весь код нужно написать самостоятельно.

## Технические требования

- строго JDK 25 (не 21 и не 26);
- все проверки выполняются из метода `main`;

Перед началом проверьте версию Java:

```text
java -version
```

## Задача

Нужно написать небольшую программу, которая создаёт заказ, рассчитывает скидку и стоимость доставки, выполняет оплату и выводит результат в консоль.

Настоящие база данных, сайт, HTTP-запросы и внешние системы не нужны. Все объекты создаются прямо в `Main`.

## Часть 1. Классы и композиция

Создайте следующие классы:

- `Customer` — покупатель с именем и электронной почтой;
- `OrderItem` — товар с названием, ценой и количеством;
- `Order` — заказ с номером, покупателем и списком товаров;
- `OrderProcessor` — выполняет оформление заказа;
- `Main` — создаёт объекты и запускает все сценарии проверки.

`Order` должен **содержать** объект `Customer` и список объектов `OrderItem`. Это пример композиции: большой объект составляется из более маленьких объектов.

Не нужно наследовать `Order` от `Customer` или `OrderItem`.

## Часть 2. Интерфейсы

### Расчёт скидки

Создайте интерфейс `DiscountPolicy` с методом расчёта скидки.

Напишите две реализации:

- `NoDiscount` — скидки нет;
- `PercentDiscount` — скидка задаётся в процентах.

### Оплата

Создайте интерфейс `PaymentMethod` с методом оплаты заказа.

Напишите две реализации:

- `CashPayment` — оплата наличными;
- `CardPayment` — оплата картой.

Метод оплаты должен уметь выбрасывать checked-исключение `PaymentException`.

### Уведомление

Создайте интерфейс `NotificationSender` с методом отправки сообщения.

Напишите минимум одну реализацию, например `ConsoleNotificationSender`, которая выводит сообщение в консоль.

## Часть 3. Абстрактный класс

Создайте абстрактный класс `AbstractDelivery`.

Он должен содержать:

- название способа доставки;
- обычный метод, возвращающий название;
- абстрактный метод расчёта стоимости доставки.

Создайте два наследника:

- `CourierDelivery` — доставка курьером за фиксированную стоимость;
- `PickupDelivery` — бесплатный самовывоз.

Оба наследника должны использоваться одинаково через тип `AbstractDelivery`.

## Часть 4. Композиция в обработчике заказа

`OrderProcessor` должен получать через конструктор:

- `DiscountPolicy`;
- `PaymentMethod`;
- `NotificationSender`;
- `AbstractDelivery`.

Не создавайте эти объекты внутри `OrderProcessor` через `new`. Их создаёт `Main` и передаёт в конструктор.

```text
OrderProcessor
 ├── DiscountPolicy
 ├── PaymentMethod
 ├── NotificationSender
 └── AbstractDelivery
```

Метод оформления заказа должен:

1. Проверить заказ.
2. Посчитать сумму товаров.
3. Применить скидку.
4. Добавить стоимость доставки.
5. Выполнить оплату.
6. Отправить сообщение об успешном оформлении.

## Часть 5. Checked-исключение

Создайте `PaymentException`, наследующийся от `Exception`.

`CardPayment` должен выбрасывать его, если оплата не прошла. Метод оплаты обязан объявить исключение через `throws PaymentException`.

В `Main` вызовите оплату внутри `try/catch` и выведите понятное сообщение. Пустой блок `catch` запрещён.

Таким образом нужно показать главное свойство checked-исключения: компилятор заставляет обработать его или объявить через `throws`.

## Часть 6. Unchecked-исключения

Создайте следующую иерархию:

```text
ApplicationException extends RuntimeException
 ├── ApiException
 │    ├── BadRequestException          -> 400
 │    ├── ApiServerException           -> 500
 │    └── TooManyRequestsException     -> 429
 ├── InternalServerErrorException      -> 500
 └── IntegrationException              -> 502
```

Требования:

- `ApplicationException` хранит общий код ошибки и сообщение;
- `ApiException` дополнительно хранит числовой статус;
- классы `BadRequestException`, `ApiServerException` и `TooManyRequestsException` передают нужный статус в конструктор базового класса;
- `IntegrationException` используется для ошибки при обращении к внешней системе;
- `InternalServerErrorException` используется для внутренней ошибки программы;
- конструкторы должны позволять сохранить исходную причину через параметр `cause`.

Все эти исключения являются unchecked, поэтому добавлять `throws` к каждому методу не требуется.

## Часть 7. Общий обработчик исключений

Создайте класс `GlobalExceptionHandler`. Это упрощённый аналог `ControllerAdvice`, но без Spring.

Его метод `handle` принимает исключение и возвращает объект `ErrorResponse` со следующими полями:

- числовой статус;
- код ошибки;
- сообщение.

Правила преобразования:

| Базовый тип | Результат |
|---|---|
| `ApiException` | статус берётся из самого исключения |
| `IntegrationException` | 502 |
| `InternalServerErrorException` | 500 |
| любое другое исключение | 500 и безопасное общее сообщение |

В обработчике нельзя писать отдельную ветку для `BadRequestException`, `ApiServerException` и `TooManyRequestsException`. Все они должны обрабатываться одной веткой для базового `ApiException`.

Если позже появится новый наследник `ApiException`, обработчик должен сразу уметь получить его статус без изменения метода `handle`.

## Часть 8. Проверка программы в Main

Отдельные тестовые классы создавать не нужно. Все сценарии последовательно запустите из `Main` и выведите ожидаемый и фактический результат.

Проверьте минимум следующие случаи:

1. Заказ без скидки, оплата наличными, доставка курьером.
2. Заказ со скидкой, оплата картой, самовывоз.
3. Неуспешная оплата картой и обработка `PaymentException` через `try/catch`.
4. `BadRequestException` преобразуется в статус 400.
5. `ApiServerException` преобразуется в статус 500.
6. `TooManyRequestsException` преобразуется в статус 429.
7. `IntegrationException` преобразуется в статус 502.
8. `InternalServerErrorException` преобразуется в статус 500.
9. Обычный `RuntimeException` преобразуется в безопасный ответ со статусом 500.

Рекомендуемый вид вывода:

```text
[OK] Заказ без скидки: ожидалось 1200, получено 1200
[OK] BadRequestException: ожидался статус 400, получен 400
[ERROR] Оплата картой: Платёж отклонён
```

Можно написать в `Main` небольшой вспомогательный метод сравнения ожидаемого и фактического значения. Использовать JUnit не нужно.

### Пример Main

Ниже показан один из вариантов организации проверок. Это ориентир, а не единственно правильное решение. Пример предполагает, что `OrderProcessor.process` возвращает итоговую сумму, а `ErrorResponse` предоставляет метод `getStatus`. Названия методов и конструкторов можно изменить, сохранив тот же смысл.

```java
package ru.homework.orders;

import java.util.List;

import ru.homework.orders.delivery.AbstractDelivery;
import ru.homework.orders.delivery.CourierDelivery;
import ru.homework.orders.delivery.PickupDelivery;
import ru.homework.orders.discount.DiscountPolicy;
import ru.homework.orders.discount.NoDiscount;
import ru.homework.orders.discount.PercentDiscount;
import ru.homework.orders.exception.ApiServerException;
import ru.homework.orders.exception.BadRequestException;
import ru.homework.orders.exception.IntegrationException;
import ru.homework.orders.exception.InternalServerErrorException;
import ru.homework.orders.exception.PaymentException;
import ru.homework.orders.exception.TooManyRequestsException;
import ru.homework.orders.handler.ErrorResponse;
import ru.homework.orders.handler.GlobalExceptionHandler;
import ru.homework.orders.model.Customer;
import ru.homework.orders.model.Order;
import ru.homework.orders.model.OrderItem;
import ru.homework.orders.notification.ConsoleNotificationSender;
import ru.homework.orders.payment.CardPayment;
import ru.homework.orders.payment.CashPayment;
import ru.homework.orders.payment.PaymentMethod;

public class Main {

    public static void main(String[] args) {
        Customer customer = new Customer("Анна", "anna@example.com");
        Order order = new Order(
                "ORDER-1",
                customer,
                List.of(new OrderItem("Книга", 500, 2))
        );

        runOrderScenario(
                "Заказ без скидки",
                order,
                1200,
                new NoDiscount(),
                new CashPayment(),
                new CourierDelivery(200)
        );

        runOrderScenario(
                "Заказ со скидкой",
                order,
                900,
                new PercentDiscount(10),
                new CardPayment(true),
                new PickupDelivery()
        );

        runFailedPaymentScenario(order);
        runExceptionHandlerScenarios();
    }

    private static void runOrderScenario(
            String name,
            Order order,
            int expectedTotal,
            DiscountPolicy discountPolicy,
            PaymentMethod paymentMethod,
            AbstractDelivery delivery
    ) {
        OrderProcessor processor = new OrderProcessor(
                discountPolicy,
                paymentMethod,
                new ConsoleNotificationSender(),
                delivery
        );

        try {
            int actualTotal = processor.process(order);
            check(name, expectedTotal, actualTotal);
        } catch (PaymentException exception) {
            System.out.println("[ERROR] " + name + ": " + exception.getMessage());
        }
    }

    private static void runFailedPaymentScenario(Order order) {
        OrderProcessor processor = new OrderProcessor(
                new NoDiscount(),
                new CardPayment(false),
                new ConsoleNotificationSender(),
                new PickupDelivery()
        );

        try {
            processor.process(order);
            System.out.println("[FAIL] Ожидалась ошибка оплаты");
        } catch (PaymentException exception) {
            System.out.println("[OK] PaymentException обработан: "
                    + exception.getMessage());
        }
    }

    private static void runExceptionHandlerScenarios() {
        GlobalExceptionHandler handler = new GlobalExceptionHandler();

        checkException(handler, "BadRequestException", 400,
                new BadRequestException("BAD_REQUEST", "Некорректный заказ"));
        checkException(handler, "ApiServerException", 500,
                new ApiServerException("API_ERROR", "Ошибка API"));
        checkException(handler, "TooManyRequestsException", 429,
                new TooManyRequestsException("RATE_LIMIT", "Слишком много запросов"));
        checkException(handler, "IntegrationException", 502,
                new IntegrationException("INTEGRATION_ERROR", "Внешняя система недоступна"));
        checkException(handler, "InternalServerErrorException", 500,
                new InternalServerErrorException("INTERNAL_ERROR", "Внутренняя ошибка"));
        checkUnknownException(handler);
    }

    private static void checkException(
            GlobalExceptionHandler handler,
            String name,
            int expectedStatus,
            Exception exception
    ) {
        ErrorResponse response = handler.handle(exception);
        check(name, expectedStatus, response.getStatus());
    }

    private static void checkUnknownException(GlobalExceptionHandler handler) {
        String internalMessage = "Пароль базы данных: secret";
        ErrorResponse response = handler.handle(new RuntimeException(internalMessage));

        check("Неизвестная ошибка", 500, response.getStatus());

        if (internalMessage.equals(response.getMessage())) {
            System.out.println("[FAIL] Пользователю показаны внутренние подробности");
        } else {
            System.out.println("[OK] Внутренние подробности скрыты");
        }
    }

    private static void check(String name, int expected, int actual) {
        if (expected == actual) {
            System.out.printf(
                    "[OK] %s: ожидалось %d, получено %d%n",
                    name,
                    expected,
                    actual
            );
            return;
        }

        System.out.printf(
                "[FAIL] %s: ожидалось %d, получено %d%n",
                name,
                expected,
                actual
        );
    }
}
```

Значения `true` и `false` в конструкторе `CardPayment` в этом примере имитируют успешную и отклонённую оплату. Подключаться к настоящей платёжной системе не нужно.

## Как здесь применяется SOLID

### S — Single Responsibility Principle

Каждый класс выполняет одну понятную задачу: скидка рассчитывается в `DiscountPolicy`, доставка — в `AbstractDelivery`, оплата — в `PaymentMethod`, ошибки преобразуются в `GlobalExceptionHandler`.

### O — Open/Closed Principle

Можно добавить новый вид скидки, оплаты или доставки отдельным классом, не изменяя уже готовые реализации.

### L — Liskov Substitution Principle

`CourierDelivery` и `PickupDelivery` можно по очереди передать в `OrderProcessor` вместо `AbstractDelivery`. Программа продолжит работать по одному контракту.

### I — Interface Segregation Principle

Для скидки, оплаты и уведомления созданы отдельные небольшие интерфейсы. В них нет лишних методов.

### D — Dependency Inversion Principle

`OrderProcessor` работает с интерфейсами, а конкретные реализации создаются и передаются в него из `Main`.

## Простая структура проекта

```text
src/main/java/ru/homework/orders
 ├── Main.java
 ├── model
 │    ├── Customer.java
 │    ├── Order.java
 │    └── OrderItem.java
 ├── discount
 │    ├── DiscountPolicy.java
 │    ├── NoDiscount.java
 │    └── PercentDiscount.java
 ├── payment
 │    ├── PaymentMethod.java
 │    ├── CashPayment.java
 │    └── CardPayment.java
 ├── delivery
 │    ├── AbstractDelivery.java
 │    ├── CourierDelivery.java
 │    └── PickupDelivery.java
 ├── notification
 │    ├── NotificationSender.java
 │    └── ConsoleNotificationSender.java
 ├── exception
 │    └── классы исключений
 ├── handler
 │    ├── ErrorResponse.java
 │    └── GlobalExceptionHandler.java
 └── OrderProcessor.java
```

Создавать дополнительные слои, репозитории и классы с названием `Service` не нужно.

## Критерии готовности

- все сценарии запускаются из `Main`;
- есть обычные классы, интерфейсы и абстрактный класс;
- показана композиция объектов;
- есть checked- и unchecked-исключения;
- исключения обрабатываются по базовым типам;
- для неизвестной ошибки не показываются внутренние подробности;
- проект компилируется и запускается из консоли.

## Правила выполнения ДЗ

1. Не использовать генеративные сети для выполнения ДЗ.
2. Использовать генеративные нейросети для освоения материала, если не понимаешь, а дядя Ваня игнорит.
3. Сдать ДЗ через fork:
   1. Нажать **Fork** в репозитории GitHub.
   2. Клонировать свой fork.
   3. В терминале выполнить `git checkout -b hw`.
   4. Выполнить ДЗ.
   5. Сделать commit.
   6. Отправить ветку командой `git push`.
   7. Нажать **Create Pull Request** и проверить: source — ветка `hw` вашего репозитория, target — ветка `main` или `master` репозитория `1ommyS`.
   8. Отправить Ване ссылку на pull request.
