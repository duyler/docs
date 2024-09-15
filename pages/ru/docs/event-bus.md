# Шина событий (event bus)

Шина событий реализует кооперативную многозадачность между выполняемыми внутри неё действиями. Каждое действие выполняется внутри отдельной нити (Fiber) в изолированном DI-контейнере. Управлять выполнением можно с помощью обработчиков состояний, триггеров и подписок на генерируемые действиями события.

### Общая схема работы

<img src="https://duyler.com/assets/img/event-bus.svg" width="100%">


### Типы состояний

**Состояния основного контекста (Main context):**

* StateMainBegin - наступает единожды перед стартом шины
* StateMainCyclic - наступает хотя бы раз и повторяется пока очередь задач не пуста в режиме `Mode::Queue`, или циклично если `Mode::Loop`
* StateMainBefore - наступает перед сборкой и запуском действия
* StateMainSuspend - наступает если действие вызывает `Fiber::suspend()`. Если в очереди есть другие действия, они будут запущены.
* StateMainResume - наступает перед возвратом управления в действие. Если обработчики для состояния не определены и в `Fiber::suspend()` была передана callback-функция, она будет запущена, а возвращаемое callback-функцией значение будет резюмировано в контекст действия.
* StateMainAfter - наступает после выполнения действия
* StateMainEnd - наступает когда очередь задач пуста и все действия завершены (достижимо только в режиме `Mode::Queue`)

**Состояния в контексте действия (Fiber context):**

* StateActionBefore - наступает перед выполнением действия
* StateActionAfter - наступает после выполнения действия
* StateActionThrowing - наступает в случае если действие выбросило исключение

Каждое состояние может быть обработано с помощью внешних обработчиков состояний. Для каждого типа состояния, предусмотрен определённый набор функций, доступных для взаимодействия с шиной из обработчика состояния.


### Обзор свойств для действий

`string|UnitEnum $id` - уникальный идентификатор действия. Например: `'MyService.DoWork'`.

`string|Closure $handler` - анонимная функция или вызываемый класс обработчик действия.

`array $required` - массив идентификаторов действий. Целевое действие может затребовать выполнения других действий, которые будут являться условием для выполнения целевого действия. Результат выполнения затребованных действий может быть передан в целевое действие. В случае если хотя бы одно из затребованных действий вернуло результат со статусом `ResultStatus::Fail`, то целевое действие выполнено не будет.

`null|string|UnitEnum $listen` - действие будет запущено после того как событие с указанным ID будет передано в шину. Если в событие передан контракт, он может быть передан в действие.

`array $bind` - массив маппинга интерфейсов для DI-контейнера. Например: `[MyInterface::class => MyClass::class]`. См. [DI](https://github.com/duyler/dependency-injection).

`array $providers` - массив сервис-провайдеров для DI-контейнера. См. [DI](https://github.com/duyler/dependency-injection).

`array $definitions` массив значений для конструкторов классов. Например: `[MyClass::class => ['argName' => 'value']]` См. [DI](https://github.com/duyler/config).

`?string $argument` - тип аргумента действия.

`string|Closure|null $argumentFactory` - анонимная функция или выполняемый класс фабрики для аргумента действия.

`?string $contract` - тип возвращаемый действием.

`string|Closure|null $rollback` - анонимная функция или выполняемый класс, вызываемый для отката действия, в случае если было выброшено исключение в любом месте программы.

`bool $externalAccess` - разрешение доступа к результату выполнения действия из `BusInterface` или обработчиков состояний.

`bool $repeatable` - разрешение повторно выполнять действие.

`bool $lock` - если действие использует параллельное выполнение внутри `Fiber::suspend()` (напр. Parallel php extension), это гарантирует последовательность выполнения повторяемых действий.

`bool $private` - если установлено значение true, это действие НЕ МЕЖЕТ быть затребовано в других действиях.

`array $sealed` - массив идентификаторов действий, которые могут затребовать это действие и получать его результат.

`bool $silent` - если установлено значение true, действие НЕ БУДЕТ генерировать внутреннее событие выполнения. Подписка на такое действие сгенерирует исключение.

`array $alternates` - массив идентификаторов действий, результат которых может заменить результат выполнения целевого действия. Результат текущего целевого действия будет заменён, если оно вернёт результат со статусом `ResultStatus::Fail`.

`int $retries` - количество повторов, если действие возвращает результат с `ResultStatus:Fail`.

`array $labels` - любые произвольные данные. Может быть полезно при реализации обработчиков состояний.