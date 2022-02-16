# Move Tutorial

Добро пожаловать в руководство по языку Move! Здесь вы сможете найти пошаговое описание по разработке проекта на Move, включающее дизайн, реализацию, юнит тестирование и формальную верификацию модулей на Move.

Всего будет 9 шагов:

- [Шаг 0: Установка](#Step0)
- [Шаг 1: Мой первый модуль на Move](#Step1)
- [Шаг 2: Добавляем юнит тесты](#Step2)
- [Шаг 3: Дизайн модуля `BasicCoin`](#Step3)
- [Шаг 4: Реализация модуля `BasicCoin`](#Step4)
- [Шаг 5: Добавление юнит тестов в `BasicCoin`](#Step5)
- [Шаг 6: Релизация `BasicCoin` с поддержкой generics](#Step6)
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

## Step 4: Implementing my `BasicCoin` module<span id="Step4"><span>

We have created a Move package for you in folder `step_4` called `BasicCoin`. The `sources` folder contains source code for
all your Move modules in the package, including `BasicCoin.move`. In this section, we will take a closer look at the
implementation of the methods inside [`BasicCoin.move`](./step_4/BasicCoin/sources/BasicCoin.move).

### Compiling our code

Let's first try building the code using Move package by running the following command
in [`step_4/BasicCoin`](./step_4/BasicCoin) folder:

```bash
move package build
```

### Implementation of methods

Now let's take a closer look at the implementation of the methods inside [`BasicCoin.move`](<(./step_4/BasicCoin/sources/BasicCoin.move)>).

<details>
<summary>Method <code>publish_balance</code></summary>

This method publishes a `Balance` resource to a given address. Since this resource is needed to receive coins through
minting or transferring, `publish_balance` method must be called by a user before they can receive money, including the
module owner.

This method uses a `move_to` operation to publish the resource:

```
let empty_coin = Coin { value: 0 };
move_to(account, Balance { coin:  empty_coin });
```

</details>
<details>
<summary>Method <code>mint</code></summary>

`mint` method mints coins to a given account. Here we require that `mint` must be approved
by the module owner. We enforce this using the assert statement:

```
assert!(Signer::address_of(&module_owner) == MODULE_OWNER, Errors::requires_address(ENOT_MODULE_OWNER));
```

Assert statements in Move can be used in this way: `assert!(<predicate>, <abort_code>);`. This means that if the `<predicate>`
is false, then abort the transaction with `<abort_code>`. Here `MODULE_OWNER` and `ENOT_MODULE_OWNER` are both constants
defined at the beginning of the module. And `Errors` module defines common error categories we can use.
It is important to note that Move is transactional in its execution -- so
if an [abort](https://diem.github.io/move/abort-and-assert.html) is raised no unwinding of state
needs to be performed, as no changes from that transaction will be persisted to the blockchain.

We then deposit a coin with value `amount` to the balance of `mint_addr`.

```
deposit(mint_addr, Coin { value: amount });
```

</details>

<details>
<summary>Method <code>balance_of</code></summary>

We use `borrow_global`, one of the global storage operators, to read from the global storage.

```
borrow_global<Balance>(owner).coin.value
                 |       |       \    /
        resource type  address  field names
```

</details>

<details>
<summary>Method <code>transfer</code></summary>

This function withdraws tokens from `from`'s balance and deposits the tokens into `to`s balance. We take a closer look
at `withdraw` helper function:

```
fun withdraw(addr: address, amount: u64) : Coin acquires Balance {
    let balance = balance_of(addr);
    assert!(balance >= amount, EINSUFFICIENT_BALANCE);
    let balance_ref = &mut borrow_global_mut<Balance>(addr).coin.value;
    *balance_ref = balance - amount;
    Coin { value: amount }
}
```

At the beginning of the method, we assert that the withdrawing account has enough balance. We then use `borrow_global_mut`
to get a mutable reference to the global storage, and `&mut` is used to create a [mutable reference](https://diem.github.io/move/references.html) to a field of a
struct. We then modify the balance through this mutable reference and return a new coin with the withdrawn amount.

</details>

### Exercises

There are two `TODO`s in our module, left as exercises for the reader:

- Finish implementing the `publish_balance` method.
- Implement the `deposit` method.

The solution to this exercise can be found in [`step_4_sol`](./step_4_sol) folder.

**Bonus exercise**

- What would happen if we deposit too many tokens to a balance?

## Step 5: Adding and using unit tests with the `BasicCoin` module<span id="Step5"><span>

In this step we're going to take a look at all the different unit tests
we've written to cover the code we wrote in step 4. We're also going to
take a look at some tools we can use to help us write tests.

To get started, run the `package test` command in the [`step_5/BasicCoin`](./step_5/BasicCoin) folder

```bash
move package test
```

You should see something like this:

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

Taking a look at the tests in the
[`BasicCoin` module](./step_5/BasicCoin/sources/BasicCoin.move) we've tried
to keep each unit test to testing one particular behavior.

<details>
<summary>Exercise</summary>

After taking a look at the tests, try and write a unit test called
`balance_of_dne` in the `BasicCoin` module that tests the case where a
`Balance` resource doesn't exist under the address that `balance_of` is being
called on. It should only be a couple lines!

The solution to this exercise can be found in [`step_5_sol`](./step_5_sol)

</details>

## Step 6: Making my `BasicCoin` module generic<span id="Step6"><span>

In Move, we can use generics to define functions and structs over different input data types. Generics are a great
building block for library code. In this section, we are going to make our simple `BasicCoin` module generic so that it can
serve as a library module that can be used by other user modules.

First, we add type parameters to our data structs:

```
struct Coin<phantom CoinType> has store {
    value: u64
}

struct Balance<phantom CoinType> has key {
    coin: Coin<CoinType>
}
```

We also add type parameters to our methods in the same manner. For example, `withdraw` becomes the following:

```
fun withdraw<CoinType>(addr: address, amount: u64) : Coin<CoinType> acquires Balance {
    let balance = balance_of<CoinType>(addr);
    assert!(balance >= amount, EINSUFFICIENT_BALANCE);
    let balance_ref = &mut borrow_global_mut<Balance<CoinType>>(addr).coin.value;
    *balance_ref = balance - amount;
    Coin<CoinType> { value: amount }
}
```

Take a look at [`step_6/BasicCoin/sources/BasicCoin.move`](./step_6/BasicCoin/sources/BasicCoin.move) to see the full implementation.

At this point, readers who are familiar with Ethereum might notice that this module serves a similar purpose as
the [ERC20 token standard](https://ethereum.org/en/developers/docs/standards/tokens/erc-20/), which provides an
interface for implementing fungible tokens in smart contracts. One key advantage of using generics is the ability
to reuse code since the generic library module already provides a standard implementation and the instantiating module
can provide customizations by wrapping the standard implementation.

We provide a little module called [`MyOddCoin`](./step_6/BasicCoin/sources/MyOddCoin.move) that instantiates
the `Coin` type and customizes its transfer policy: only odd number of coins can be transferred. We also include two
[tests](./step_6/BasicCoin/sources/MyOddCoin.move) to test this behavior. You can use the commands you learned in step 2 and step 5 to run the tests.

#### Advanced topics:

<details>
<summary><code>phantom</code> type parameters</summary>

In definitions of both `Coin` and `Balance`, we declare the type parameter `CoinType`
to be phantom because `CoinType` is not used in the struct definition or is only used as a phantom type
parameter.

Read more about phantom type parameters <a href="https://diem.github.io/move/generics.html#phantom-type-parameters">here</a>.

</details>

## Advanced steps

Before moving on to the next steps, let's make sure you have all the prover dependencies installed.

Try running `boogie /version `. If an error message shows up saying "command not found: boogie", you will have to run the
setup script and source your profile:

```bash
# run the following in diem repo root directory
./scripts/dev_setup.sh -yp
source ~/.profile
```

## Step 7: Use the Move prover<span id="Step7"><span>

Smart contracts deployed on the blockchain may maniputate high-value assets. As a technique that uses strict mathematical methods to describe behavior and reason correctness of computer systems, formal verification has been used in blockchains to prevent bugs in smart contracts. [The Move prover](https://github.com/diem/move/tree/main/language/move-prover) is an evolving formal verification tool for smart contracts written in the Move language. The user can specify functional properties of smart contracts using the [Move Specification Language (MSL)](https://github.com/diem/move/blob/main/language/move-prover/doc/user/spec-lang.md) and then use the prover to automatically check them statically. To illustrate how the prover is used, we have added the following code snippet to the [BasicCoin.move](./step_7/BasicCoin/sources/BasicCoin.move):

```
    spec balance_of {
        pragma aborts_if_is_strict;
    }
```

Informally speaking, the block `spec balance_of {...}` contains the property specification of the method `balance_of`.

Let's first run the prover using the following command inside [`BasicCoin` directory](./step_7/BasicCoin/):

```bash
move package prove
```

which outputs the following error information:

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

The prover basically tells us that we need to explicitly specify the condition under which the function `balance_of` will abort, which is caused by calling the function `borrow_global` when `owner` does not own the resource `Balance<CoinType>`. To remove this error information, we add an `aborts_if` condition as follows:

```
    spec balance_of {
        pragma aborts_if_is_strict;
        aborts_if !exists<Balance<CoinType>>(owner);
    }
```

After adding this condition, try running the `prove` command again to confirm that there are no verification errors:

```bash
move package prove
```

Apart from the abort condition, we also want to define the functional properties. In Step 8, we will give more detailed introduction to the prover by specifying properties for the methods defined the `BasicCoin` module.

## Step 8: Write formal specifications for the `BasicCoin` module<span id="Step8"><span>

<details>

<summary> Method withdraw </summary>

The signature of the method `withdraw` is given below:

```
fun withdraw<CoinType>(addr: address, amount: u64) : Coin<CoinType> acquires Balance
```

The method withdraws tokens with value `amount` from the address `addr` and returns a created Coin of value `amount`. The method `withdraw` aborts when 1) `addr` does not have the resource `Balance<CoinType>` or 2) the number of tokens in `addr` is smaller than `amount`. We can define conditions like this:

```
   spec withdraw {
        let balance = global<Balance<CoinType>>(addr).coin.value;
        aborts_if !exists<Balance<CoinType>>(addr);
        aborts_if balance < amount;
    }
```

As we can see here, a spec block can contain let bindings which introduce names for expressions. `global<T>(address): T` is a built-in function that returns the resource value at `addr`. `balance` is the number of tokens owned by `addr`. `exists<T>(address): bool` is a built-in function that returns true if the resource T exists at address. Two `aborts_if` clauses correspond to the two conditions mentioned above. In general, if a function has more than one `aborts_if` condition, those conditions are or-ed with each other. By default, if a user wants to specify aborts conditions, all possible conditions need to be listed. Otherwise, the prover will generate a verification error. However, if `pragma aborts_if_is_partial` is defined in the spec block, the combined aborts condition (the or-ed individual conditions) only _imply_ that the function aborts. The reader can refer to the [MSL](https://github.com/diem/move/blob/main/language/move-prover/doc/user/spec-lang.md) document for more information.

The next step is to define functional properties, which are described in the two `ensures` clauses below. First, by using the `let post` binding, `balance_post` represents the balance of `addr` after the execution, which should be equal to `balance - amount`. Then, the return value (denoted as `result`) should be a coin with value `amount`.

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

<details>
<summary> Method deposit </summary>

The signature of the method `deposit` is given below:

```
fun deposit<CoinType>(addr: address, check: Coin<CoinType>) acquires Balance
```

The method deposits the `check` into `addr`. The specification is defined below:

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

`balance` represents the number of tokens in `addr` before execution and `check_value` represents the number of tokens to be deposited. The method would abort if 1) `addr` does not have the resource `Balance<CoinType>` or 2) the sum of `balance` and `check_value` is greater than the maxium value of the type `u64`. The functional property checks that the balance is correctly updated after the execution.

</details>

<details>

<summary> Method transfer </summary>

The signature of the method `transfer` is given below:

```
public fun transfer<CoinType: drop>(from: &signer, to: address, amount: u64, _witness: CoinType) acquires Balance
```

The method transfers the `amount` of coin from the account of `from` to the address `to`. The specification is given below:

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

`addr_from` is the address of `from`. Then the balances of `addr_from` and `to` before and after the execution are obtained.
The `ensures` clauses specify that the `amount` number of tokens is deducted from `addr_from` and added to `to`. However, the prover will generate the error information as below:

```
   ┌─ diem/language/documentation/hackathon-tutorial/step_7/BasicCoin/sources/BasicCoin.move:62:9
   │
62 │         ensures balance_from_post == balance_from - amount;
   │         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   │
   ...
```

The property is not held when `addr_from` is equal to `to`. As a result, we could add an assertion `assert!(from_addr != to)` in the method to make sure that `addr_from` is not equal to `to`.

</details>

<details>

<summary> Exercises </summary>

- Implement the `aborts_if` conditions for the `transfer` method.
- Implement the specification for the `mint` and `publish_balance` method.

The solution to this exercise can be found in [`step_8_sol`](./step_8_sol).
