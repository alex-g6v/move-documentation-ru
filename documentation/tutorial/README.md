# Move Tutorial

Добро пожаловать в руководство по языку Move! Здесь вы сможете найти пошаговое описание по разработке проекта на Move, включающее дизайн, реализацию, юнит тестирование и формальную верификацию модулей на Move.

Всего будет 9 шагов:

- [Шаг 0: Установка](#Step0)
- [Шаг 1: Мой первый модуль на Move](#Step1)
- [Шаг 2: Добавляем юнит тесты](#Step2)
- [Шаг 3: Дизайн модуля `BasicCoin`](#Step3)
- [Шаг 4: Реализация модуля `BasicCoin`](#Step4)
- [Шаг 5: Добавление юнит тестов в `BasicCoin`](#Step5)
- [Шаг 6: Релизация `BasicCoin` выражений ( в блоке ниже ). Сначала используется `let post`с поддержкой generics](#Step6)
- [Шаг 7: Использование Move prover](#Step7)
- [Шаг 8: Написание формальной спецификации для `BasicCoin`](#Step8)

Код каждого шага находится в дирректории `step_x` и является самодостаточным. То есть если например вы решите пропустить все шаги с шага 1 до шага 4 - смело переходите к дирректории `step_5` - в ней уже будет весь код написаный в предыдущих шагах. Так же в конце некоторых шагов мы выключили список дополнительных материалов на более продвинутые темы.

Можно начинать!

## Step 0: Установка<span id="Step0"><span>

Если вы не делали этого ранее, то откройте терминал и склонируйте [репозиторий Diem](https://github.com/diem/diem) и [репозиторий Move](https://github.com/diem/move):

```bash
git clone https://github.com/diem/diem.git
git clone https://github.com/diem/move.git
```

Перейдите в директорию `diem` и запустите `dev_setup.sh` скрипт:

```bash
cd diem
./scripts/dev_setup.sh -ypt
```

Ответьте на приглашения скрипта чтобы установить все необходимые зависимости.

Скрипт добавит переменные окружения в ваш `~/.profile` файл. Выполните следующую команду чтобы изменения вступили в силу:

```bash
source ~/.profile
```

Затем нужно установит инструменты командной строки, запустив следующие команды:

```bash
cd ..
cargo install --path diem/diem-move/df-cli
cargo install --path move/language/move-analyzer
```

Чтобы проверить что все установлено успешно, можно выполнить следующую команду:

```
move-analyzer --version  # Выведет: move-analyzer 0.0.0
```

Команда `df-cli` это удобный враппер над интерфейсом командной строки `move`. Для простоты в этом руководстве мы будем называть его просто `move`. Чтобы было еще проще вы можете добавить вот такй alias командной строки:

```bash
alias move="df-cli"
```

Чтбы проверить что он работает можно запустить команду:

```bash
move package -h
```

После чего в консоли появится вывод вроде указаного ниже:

```
move-package 0.1.0
Package and build system for Move code.

USAGE:
    move package [FLAGS] [OPTIONS] <SUBCOMMAND>
...
```

Чтобы получить более подробное описание команд `move` а так же описание того что они делают, можно запустить команду с флагом `-h`.

Так же можно включить поддержку языка `Move` в Visual Studio Code. Чтобы это сделать можно найти плагин который называется "move-analyzer" во вкладке Extensions и установить его. Более детальные инструкции можно найти в [README](https://github.com/diem/move/tree/main/language/move-analyzer/editors/code).

Перед следующими шагами нужно перейти в директорию руководсва:

```bash
cd <path_to_move_repo>/language/documentation/tutorial
```

## Шаг 1: Мой первый модуль на Move<span id="Step1"><span>

Перейдтие в директорию [`step_1/BasicCoin`](./step_1/BasicCoin). Там вы увидите дирректорию `sources` -- это место где живет весь код проекта. Вы так же увидите файл `Move.toml` -- он описывает зависимости и другую информацию о пакете; если вы знакомы с языком Rust и Сargo, то `Move.toml` может вам показаться похожим на `Cargo.toml`, а директория `sources` похожей на `src`.

Давайте посмотрим на код написанный на Move! Откройте [`sources/FirstModule.move`](./step_1/BasicCoin/sources/FirstModule.move) в своем любимом редакторе. Первое что вы увидите будет:

```
// sources/FirstModule.move
module 0xCAFE::BasicCoin {
    ...
}
```

Это объявление [модуля](https://diem.github.io/move/modules-and-scripts.html).
Модули - строительные блоки кода на Move, и их определние влючает адрес -- адрес по которому этот модуль будет опубликован. В этом случае `BasicCoin` модуль может быть опубликован только по адресу `0xCAFE`.

Теперь давайте взглянем на следующую часть этого файла где идет объявление [структуры](https://diem.github.io/move/structs-and-resources.html) которая описывет сущность `Coin` у которой может быть задано значение в поле `value`:

```
module 0xCAFE::BasicCoin {
    struct Coin has key {
        value: u64,
    }
    ...
}
```

Глядя на оставшуюся часть файла можно увидеть объявление функции, которая создает структуру `Coin` и помещает ее на аккаунт:

```
module 0xCAFE::BasicCoin {
    struct Coin has key {
        value: u64,
    }

    public fun mint(account: signer, value: u64) {
        move_to(&account, Coin { value })
    }
}
```

Давайте посмотрим на эту функцию и то что она делает:

- Она принимает агрумент [`signer`](https://diem.github.io/move/signer.html) -- это такой неподделываемый токен, который является подтверждением владения адресом и аргумент `value`, в котором указано сколько нужно создать монет.

- И создает структуру `Coin` с указаным значением `value`, после чего помещает ее на указанный в первом агрументе аккаунт, используя оператор `move_to`.

Давайте проверим что все собирается! Чтобы это сделать нужно запустить команду `pacakge build` и дирректории проекта ([`step_1/BasicCoin`](./step_1/BasicCoin/)):

```bash
move package build
```

<details>
<summary>Продвинутые концепции и ссылки</summary>

- Можно создавать пустые пакеты Move, вызывая команду:

  ```bash
  move package new <pkg_name>
  ```

- Код Move может жить и в разных местах. Подробнее об этом можно прочитать в
  [книге](https://diem.github.io/move/packages.html)

- Больше информации о `Move.toml` можно найти в [разделе про пакеты Move](https://diem.github.io/move/packages.html#movetoml).

- В Move есть поддержка [именованных адресов](https://diem.github.io/move/address.html#named-addresses). Именованные адреса - это такой способ задавать адреса, чтобы их можно было задавать параметрами во время компиляции. Они используются довольно часто и могут быть заданы в `Move.toml` файле в разделе `[addresses]`. Например:
  ```
  [addresses]
  SomeNamedAddress = "0xC0FFEE"
  ```
- [Структуры](https://diem.github.io/move/structs-and-resources.html) в Move могут обладать различными [способностями](https://diem.github.io/move/abilities.html) которые описывают какие операции допустимы с этим типом:
  Всего есть четыре способности:
  - `copy`: Значения этого типа могут быть скопированы.
  - `drop`: Значения этого типа могут быть удалены.
  - `store`: Значения этого типа можно хранить внутри структур в глобальном хранилище.
  - `key`: Этот тип можно использовать как ключ для операций с глобальным хранилищем. (Примечание: тут имеется в виду что можно передавать этот тип в конструкции типа `borrow_global_mut<Coin>`)

Так вот в `BaseCoin` у структуры `Coin` есть только возможность быть использованной в качестве ключа к глобальному хранилищу и из-за того что других способностей не указано - ее нельзя копировать, удалять или хранить внутри других структур. Так что вы не можете копировать монеты и не можете их случайно потерять!

- [Функции](https://diem.github.io/move/functions.html) по умолчанию приватны, но могут быть объявлены как `public`,[`public(friend)`](https://diem.github.io/move/friends.html) или `public(script)`. `public(script)` позволяет вызывать функцию из скриптов транзакций или из других `public(script)` функций.

- `move_to` - это один из [пяти операций работы с глобальным хранилищем](https://diem.github.io/move/global-storage-operators.html).
</details>

## Шаг 2: Добавляем юнит тесты<span id="Step2"><span>

Мы познакомились с первым модулем, теперь настало время узнать как писать тесты. Тесты нужны чтобы быть уверенным в том что минтинг монет работает как мы и задумывали. Для этого переходим в директорию [`step_2/BasicCoin`](./step_2/BasicCoin). Юнит тесты в Move похожи на юнит тесты в Rust -- они аннотируются директивой `#[test]` а в остальном это обычные функции.

Чтбы запустить тесты нужно выполнить команду `package test`:

```bash
move package test
```

Давайте теперь посмотрим в код файла [`FirstModule.move`](./step_2/BasicCoin/sources/FirstModule.move). Первое изменение которое вы увидите это тест:

```
module 0xCAFE::BasicCoin {
    ...
    // Declare a unit test. It takes a signer called `account` with an
    // address value of `0xC0FFEE`.
    #[test(account = @0xC0FFEE)]
    fun test_mint_10(account: signer) acquires Coin {
        let addr = Signer::address_of(&account);
        mint(account, 10);
        // Make sure there is a `Coin` resource under `addr` with a value of `10`.
        assert!(borrow_global<Coin>(addr).value == 10, 0);
    }
}
```

Он описывает тест, который называется `test_mint_10` котрый минтит монету `Coin` на аккаунте `account` со значением `value` равным `10`. После чего проверяет что состояние хранилища соответствует ожидаемому. Это делается с помощью вызова `assert!`. Если `assert!` падает - юнит тест тоже падает.

<details>
<summary>Продвинутые концепции и упражнения</summary>

- Существует несколько аннотаций, относящихся к тестированию, которые стоит изучить. Их можно найти [тут](https://github.com/diem/move/blob/main/language/changes/4-unit-testing.md#testing-annotations-their-meaning-and-usage).
  Некоторые из них вы увидите позже на шаге 5.

- До запуска тестов всегда нужно убедиться что вы добавили стандартную библиотеку Move в зависимости. Чтобы это сделать нужно добавить запись в раздел `[dependencies]` файла `Move.toml`:

  ```toml
  [dependencies]
  MoveStdlib = { local = "../../../../move-stdlib/", addr_subst = { "Std" = "0x1" } }
  ```

Обратите внимание что вам может понадобитсья указать ваш правильный путь до `move-stdlib` который находится в `<path_to_diem>/language`. Так же можно указыват зависимости находящиеся в гит репозитории. Больше об этом можно прочитать [тут](https://diem.github.io/move/packages.html#movetoml).

#### Упражнения

- Измените assert проверку на `11` чтобы тесту упал. Найдите флаг который можно передать в команду `move package test` чтобы она показывала вам состояние глобального стораджа в случе если тест завершается с ошибкой. Должно выглядеть как-то так:
  ```
    ┌── test_mint_10 ──────
    │ error[E11001]: test failure
    │    ┌─ step_2/BasicCoin/sources/FirstModule.move:22:9
    │    │
    │ 18 │     fun test_mint_10(account: signer) acquires Coin {
    │    │         ------------ In this function in 0xcafe::BasicCoin
    │    ·
    │ 22 │         assert!(borrow_global<Coin>(addr).value == 11, 0);
    │    │         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ Test was not expected to abort but it aborted with 0 here
    │
    │
    │ ────── Storage state at point of failure ──────
    │ 0xcafe:
    │       => key 0xcafe::BasicCoin::Coin {
    │           value: 10
    │       }
    │
    └──────────────────
  ```
- Find a flag that allows you to gather test coverage information, and
  then play around with using the `move package coverage` command to look at
  coverage statistics and source coverage.
- Найдите флаг который позволяет собирать информацию об уровне покрытия кода тестами, после чего поэксперементируйте с командой `move package coverate` чтобы посмотреть различную статистику.

</details>

## Шаг 3: Дизайн модуля `BasicCoin`<span id="Step3"><span>

В этой секции мы задизайним модуль монеты и интерфейс баланса. Монеты можно минтить и отправлять на балансы разных адресов.

Публичный интерфейс модуля состоит из следующих функций:

```
/// Создать нулевой баланс по указанному адресу. Эта функция должна быть вызывана до минтинга или передачи монет по этому адресу
public fun publish_balance(account: &signer) { ... }

/// Заминтить монеты по указанному адресу. Минтинг делается владельцем модуля монеты
public fun mint(module_owner: &signer, mint_addr: address, amount: u64) acquires Balance { ... }

/// Узнать баланс адреса
public fun balance_of(owner: address): u64 acquires Balance { ... }

/// Передать монеты с одного адреса на другой
public fun transfer(from: &signer, to: address, amount: u64) acquires Balance { ... }
```

Теперь посмотрим какие для этого нам понадобятся структуры.

У модуля в Move нет своего собственного хранилища. Вместо этого есть "глобальное хранилище" (то что мы называем состоянием блокчейна) поиск в котором осуществляется по адресу. По каждому адресу могут находиться модули (код) и ресурсы (значения). Проще говоря глобальное имеет следущую структуру (если выражаться на Rust):

```rust
struct GlobalStorage {
    resources: Map<address, Map<ResourceType, ResourceValue>>
    modules: Map<address, Map<ModuleName, ModuleBytecode>>
}
```

Хранилище по каждому адресу по сути является маппингом типов к значениям. (Внимательный читатель мог подметить что это означает что каждый адрес может содержать только одно значения для каждого типа ресурса). В нашем модуле `BasicCoin` мы определяем ресурс `Balance` который отражает сколько монет находится по адресу:

```
/// Struct representing the balance of each address.
struct Balance has key {
    coin: Coin // same Coin from Step 1
}
```

Упрощенное представление состояния блокчейна:

![](diagrams/move_state.png)

#### Продвинутые топики:

<details>
<summary><code>public(script)</code> функции</summary>

Только функции с `public(script)` видимостью могут быть вызваны напрямую из скриптов транзакций. Так что если выхотите вызывать функцию `transfer` из транзакции то вам нужно изменить сигнатуру функции на:

```
public(script) fun transfer(from: signer, to: address, amount: u64) acquires Balance { ... }
```

Узнать больше об областях видимости функций можно [тут](https://diem.github.io/move/functions.html#visibility).

</details>
<details>
<summary>Сравнение с Ethereum/Solidity</summary>

В большинстве [ERC-20](<(https://ethereum.org/en/developers/docs/standards/tokens/erc-20/)>) контрактов состояние баланса каждого адреса хранится в переменной типа <code>mapping(address => uint256)</code>. Эта переменная хранится в хранилище конкретного смарт контракта.

Состояние блокчейна в Ethereum выглядит примерно так:

![](diagrams/solidity_state.png)

</details>

## Шаг 4: Реализация модуля `BasicCoin`<span id="Step4"><span>

Мы уже подготовили для вас пакет Move в дирректории `step_4` который называется `BasicCoin`. В директории `sources` уже лежат исходники для всех модулей, включая `BasicCoin.move`. В этом разделе мы подробней разберем реализацию методов[`BasicCoin.move`](./step_4/BasicCoin/sources/BasicCoin.move).

### Компилируем код

Давайте сначала соберем проект запустив команду в директории [`step_4/BasicCoin`](./step_4/BasicCoin):

```bash
move package build
```

### Реализация методов

Теперь посмотрим на реализацию методов в [`BasicCoin.move`](<(./step_4/BasicCoin/sources/BasicCoin.move)>).

<details>
<summary>Метод <code>publish_balance</code></summary>

Этод метод публикует ресурс `Balance` по заданному адресу. Это нужно для того чтобы владелец адреса мог получать заминченные или переданные другим владельцем монеты. `publish_balance` должен быть вызван самим владельцем адреса до начала других операций.

Этот метод использует операцию `move_to` для публикации ресурса:

```
let empty_coin = Coin { value: 0 };
move_to(account, Balance { coin:  empty_coin });
```

</details>
<details>
<summary>Метод <code>mint</code></summary>

Метод `mint` минтит монеты на заданный аккаунт. Этот вызов должен быть одобрен владельцем модуля и мы убеждаемся в этом используя assert:

```
assert!(Signer::address_of(&module_owner) == MODULE_OWNER, Errors::requires_address(ENOT_MODULE_OWNER));
```

Assert имеет следующую сигнатуру: `assert!(<predicate>, <abort_code>);`. Это выражение значит что если `<predicate>` равен false то транзакция прервется с кодом `<abort_code>`. В выражении свержу `MODULE_OWNER` и `ENOT_MODULE_OWNER` - константы определенные в начале модуля. Модуль `Errors` содержит общие категории ошибок которые мы можем использовать.
Важно понимать что выполнение кода на Move транзакционно -- это значит что если исполнение [прервется](https://diem.github.io/move/abort-and-assert.html), то не потребуется откатывать какие-либо операции, так как в этом случае в блокчейн записано ничего не будет.

После чего мы пополняем монеты в колличестве `value` на баланс `mint_addr`.

```
deposit(mint_addr, Coin { value: amount });
```

</details>

<details>
<summary>Метод <code>balance_of</code></summary>

Тут мы используем `borrow_global` - один из операторов работы с глобальным стораджем предназначенный для чтения.

```
borrow_global<Balance>(owner).coin.value
                 |       |       \    /
        resource type  address  field names
```

</details>

<details>
<summary>Метод <code>transfer</code></summary>

Эта функция снимает токены с баланса счета c адреса `from` и помещает их на баланс адреса `to`. Вот как выглядит эта функция:

```
fun withdraw(addr: address, amount: u64) : Coin acquires Balance {
    let balance = balance_of(addr);
    assert!(balance >= amount, EINSUFFICIENT_BALANCE);
    let balance_ref = &mut borrow_global_mut<Balance>(addr).coin.value;
    *balance_ref = balance - amount;
    Coin { value: amount }
}
```

В начале метода мы проверяем что на на адресе списания достаточно токенов на балансе. После чего используем `borrow_global_mut` чтобы получить ссылку с возможностью записи в глобальное хранилище, а оператор `&mut` нужен для того чтобы создать ссылку с [возможностью записи](https://diem.github.io/move/references.html) на поле структуры. После чего модифицируем баланс через полученную ссылку и возвращаем новую монету со значением равным тому что было только что списано.

</details>

### Упражнения

В нашем модуле есть два `TODO`, которые можно реализовать чтобы потренироваться:

- Допишите метод `publish_balance`.
- Реализуйте метод `deposit`.

Решение этого упражнения можно найти в дирректории [`step_4_sol`](./step_4_sol).

**Бонусные упражнения**

- Что произойдет если мы попытаемся перевести слишком много токенов на баланс?

## Шаг 5: Добавление юнит тестов в `BasicCoin`<span id="Step5"><span>

На этом шаге мы взглянем на все юнит тесты которые мы написали для кода модуля из шага 4. А еще узнаем про инструменты которые помогают в написании тестов.

Для начала давайте запустим команду `package test` в дирректории [`step_5/BasicCoin`](./step_5/BasicCoin)

```bash
move package test
```

На выходе вы должны увидеть нечто похожее:

```
BUILDING MoveStdlib
BUILDING BasicCoin
Running Move unit tests
[ PASS    ] 0xcafe::BasicCoin::can_withdraw_amount
[ PASS    ] 0xcafe::BasicCoin::init_check_balance
[ PASS    ] 0xcafe::BasicCoin::init_non_owner
[ PASS    ] 0xcafe::BasicCoin::publish_balance_already_exists
[ PASS    ] 0xcafe::BasicCoin::publish_balance_has_zero
[ PASS    ] 0xcafe::BasicCoin::withdraw_dne
[ PASS    ] 0xcafe::BasicCoin::withdraw_too_much
Test result: OK. Total tests: 7; passed: 7; failed: 0
```

Глядя на тесты из [модуля `BasicCoin`](./step_5/BasicCoin/sources/BasicCoin.move) можно понять что мы пытались написать тесты таким образом чтобы каждый покрывал какое-то отдельно взятое поведение модуля.

<details>
<summary>Упражения</summary>

После того как познакомитесь с уже написанными тестами попробуйте написать тест `balance_of_dne` в `BasicCoin` который бы проверял слуай отсутствия ресурса `Balance` по адресу переданному в `balance_of`. Реализация займет всего пару строк!

Решение для этого упражнения может быть найдено в директории [`step_5_sol`](./step_5_sol)

</details>

## Step 6: Релизация `BasicCoin` с поддержкой generics<span id="Step6"><span>

В Move можно использовать generics чтобы описывать функции и структуры над другим типом данных. Generics - отличный инструмент для создания библиотек. В этом разделе мы сделаем наш `BasicCoin` дженеричным, посредством чего он может быть использован как библиотечный модуль для построения других модулей.

Сначала нужно добавить параметры типов в наши структуры:

```
struct Coin<phantom CoinType> has store {
    value: u64
}

struct Balance<phantom CoinType> has key {
    coin: Coin<CoinType>
}
```

Таким же образо можно добавить параметры типов и в функции. Например `withdraw` будет выглядеть следующим образом:

```
fun withdraw<CoinType>(addr: address, amount: u64) : Coin<CoinType> acquires Balance {
    let balance = balance_of<CoinType>(addr);
    assert!(balance >= amount, EINSUFFICIENT_BALANCE);
    let balance_ref = &mut borrow_global_mut<Balance<CoinType>>(addr).coin.value;
    *balance_ref = balance - amount;
    Coin<CoinType> { value: amount }
}
```

Взгляните на [`step_6/BasicCoin/sources/BasicCoin.move`](./step_6/BasicCoin/sources/BasicCoin.move) чтобы посмотреть полную реализацию.

Сейчас читатели, знакомые с Etherium, могут подметить что этот модуль выполняет функцию схожую с той что играет [стандарт ERC20](https://ethereum.org/en/developers/docs/standards/tokens/erc-20/), который предоставляет интерфейс для реализации токенов в смартконтрактах. Одно из главных преимуществ использования дженериков это возможность переиспользования кода, так как в базовом модуле уже есть реализация которую можно кастомизировать для своих нужд.

Мы создали небольшой модуль [`MyOddCoin`](./step_6/BasicCoin/sources/MyOddCoin.move) который использует тип `Coin` но переопределяет способ передачи монет: теперь передавать можно только нечетное количество монет. Так же мы добавили еще два [теста](./step_6/BasicCoin/sources/MyOddCoin.move) чтобы проверить это поведение. Вы уже знаете команды для запуска тестов из шагов 2 и 5.

#### Продвинутые топики:

<details>
<summary><code>phantom</code> параметры</summary>

В определении обоих типов `Coin` и `Balance` мы добавили параметр для типа: `CoinType` и он определен как `phantom` потому что он не используется в описании структур. Больше про phantom параметры можно прочитать <a href="https://diem.github.io/move/generics.html#phantom-type-parameters">тут</a>.

</details>

## Продвинутые шаги

Перед тем как преступить к следующим шагам давайте убедимся что у вас установлены все зависимости необхдимые для прувера.

Попробуйте запустить команду `boogie /version`. Если увидите ошибку вроде "command not found: boogie", то вам нужно запустить скрипт установки:

```bash
# run the following in diem repo root directory
./scripts/dev_setup.sh -yp
source ~/.profile
```

## Шаг 7: Использование Move prover<span id="Step7"><span>

Смарт контракты в блокчейне могу манипулировать активами которые дорого стоят. Существует техника которая использует математические методы чтобы описать поведение и корректность компьюетерных систем и она называется формальная верификация, поэтому она подходит для блокчейна чтобы избежать наличия багов в смарт контрактах. [The Move prover](https://github.com/diem/move/tree/main/language/move-prover) - это развивающийся инструмент для формальной верификации смартконтрактов написанных на Move. Пользователь может описать свойства контракта используя [Язык спецификации Move](https://github.com/diem/move/blob/main/language/move-prover/doc/user/spec-lang.md) после чего использовать prover чтобы проверить их. Чтобы продемонстрировать как работает prover мы добавили следующий кусок кода в [BasicCoin.move](./step_7/BasicCoin/sources/BasicCoin.move):

```
    spec balance_of {
        pragma aborts_if_is_strict;
    }
```

Блок `spec balance_of {...}` содержит спецификацию свойств метода `balance_of`.

Давайте запустим prover используя следующую команду в дирректории [`BasicCoin` directory](./step_7/BasicCoin/):

```bash
move package prove
```

которая выдаст следующую ошибку:

```
error: abort not covered by any of the `aborts_if` clauses
   ┌─ diem/language/documentation/hackathon-tutorial/step_7/BasicCoin/sources/BasicCoin.move:38:5
   │
35 │           borrow_global<Balance<CoinType>>(owner).coin.value
   │           ------------- abort happened here with execution failure
   ·
38 │ ╭     spec balance_of {
39 │ │         pragma aborts_if_is_strict;
40 │ │     }
   │ ╰─────^
   │
   =     at /diem/language/documentation/hackathon-tutorial/step_7/BasicCoin/sources/BasicCoin.move:34: balance_of
   =         owner = 0x29
   =     at /diem/language/documentation/hackathon-tutorial/step_7/BasicCoin/sources/BasicCoin.move:35: balance_of
   =         ABORTED

Error: exiting with verification errors
```

Prover говорит что нам над явно указать условие при котором функция `balance_of` будет прервана, что свою очередь случается в случае если вызвать `borrow_global` когда на аккунте у `owner` нет ресурса `Balance<CoinType>`. Чтобы избавиться от этой ошибки, мы должны добавить условие `aborts_if` следующим образом:

```
    spec balance_of {
        pragma aborts_if_is_strict;
        aborts_if !exists<Balance<CoinType>>(owner);
    }
```

После добавления этого условия, попробуйте запустить команду `prove` чтобы убедиться что теперь ошибки верификации исчезли:

```bash
move package prove
```

Помимо условий прерывания транзакции, хочется так же добавить и условий про работу функции. В шаге 8 добавим более детальные инструкции для прувера и опишем свойства методов из `BasicCoin`.

## Шаг 8: Написание формальной спецификации для `BasicCoin`<span id="Step8"><span>

<details>

<summary> Метод `withdraw` </summary>

У `withdraw` следующая сигнатура:

```
fun withdraw<CoinType>(addr: address, amount: u64) : Coin<CoinType> acquires Balance
```

Метод списывает монеты в колличестве `amount` с адреса `addr` и возвращает новую монету в количестве `amount`. Метод `withdraw` прерывается в случае если 1) По адресу `addr` еще нету ресурса `Balance<CoinType>` либо 2) Количество монет лежащее на `addr` меньше чем требуется перевести. Эти условия можно описать следующим образом:

```
   spec withdraw {
        let balance = global<Balance<CoinType>>(addr).coin.value;
        aborts_if !exists<Balance<CoinType>>(addr);
        aborts_if balance < amount;
    }
```

В блоке spec выражениям можно присваивать имена используя `let`. Встроенная функция `global<T>(address): T` возвращает значение ресурса по адресу `addr` поэтому `balance` в данном случае - это количество токенов находящихся по адресу `addr`. Функция `exists<T>(address): bool` возвращает true в случае если ресурс T существует по адресу `address`. Два выражения `aborts_if` проверяют условия 1 и 2, описанные выше.

В следующем шаге описываются фунциональные свойства, которые описываются с помощью `ensures` выражений ( в блоке ниже ). Сначала используя `let post` объявляется `balance_post` который соответствует балансу `addr` после выполнения и который должен быть равен `balance - amount`. После чего конечный результат ( обозначаемый как `result` ) должен быть равен значению `amount`.

```
   spec withdraw {
        let balance = global<Balance<CoinType>>(addr).coin.value;
        aborts_if !exists<Balance<CoinType>>(addr);
        aborts_if balance < amount;

        let post balance_post = global<Balance<CoinType>>(addr).coin.value;
        ensures balance_post == balance - amount;
        ensures result == Coin<CoinType> { value: amount };
    }
```

</details>

<summary> Метод deposit </summary>

У метода `deposit` вот такая сигнатура:

```
fun deposit<CoinType>(addr: address, check: Coin<CoinType>) acquires Balance
```

Метод переводит `check` по адресу `addr`. Его спецификацию можно описать следующим образом:

```
    spec deposit {
        let balance = global<Balance<CoinType>>(addr).coin.value;
        let check_value = check.value;

        aborts_if !exists<Balance<CoinType>>(addr);
        aborts_if balance + check_value > MAX_U64;

        let post balance_post = global<Balance<CoinType>>(addr).coin.value;
        ensures balance_post == balance + check_value;
    }
```

Тут `balance` представляет количество токенов по адресу `addr` до выполнения функции, а `check_value` - количество токенов которое необходимо перевести. Метод прерывается в двух случаях: 1) `addr` не содержит ресурса `Balance<CoinType>` или 2) сумма `balance` и `check_value` больше чем максимальное значение которое может поместиться в тип `u64`. Функциональное свойств проверяет что после выполнения функции баланс будет корректно обновлен.

</details>

<details>

<summary> Метод transfer </summary>

Вот сигнатура метода `transfer`:

```
public fun transfer<CoinType: drop>(from: &signer, to: address, amount: u64, _witness: CoinType) acquires Balance
```

Метод переводит монеты в колличестве `amount` cо счета `from` на счет `to`. Спецификацию метода можно описать следующим образом:

```
spec transfer {
        let addr_from = Signer::address_of(from);

        let balance_from = global<Balance<CoinType>>(addr_from).coin.value;
        let balance_to = global<Balance<CoinType>>(to).coin.value;
        let post balance_from_post = global<Balance<CoinType>>(addr_from).coin.value;
        let post balance_to_post = global<Balance<CoinType>>(to).coin.value;

        ensures balance_from_post == balance_from - amount;
        ensures balance_to_post == balance_to + amount;
    }
```

Тут `addr_from` - это адрес `from`. После чего идет описание балансов `addr_from` и `to` до и после выполнения транзакции. И хотя `ensures` описывают что токены в колличестве `amount` списываются с `addr_from` и переводятся на `to` - прувер все равно выдаст ошибку:

```
   ┌─ diem/language/documentation/hackathon-tutorial/step_7/BasicCoin/sources/BasicCoin.move:62:9
   │
62 │         ensures balance_from_post == balance_from - amount;
   │         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   │
   ...
```

Что говорит о том что условие не выполняется в случае если `addr_from` равен `to`. Так что нужно добавить в метод условие `assert!(from_addr != to)` чтобы соответствовать спецификации.

</details>

<details>

<summary> Упражнения </summary>

- Реализуйте `aborts_if` условия для метода `transfer`.
- Напишите спецификацию для методов `mint` и `publish_balance`.

Решения к этиму упражнениям можно найти в [`step_8_sol`](./step_8_sol).
