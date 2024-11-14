- **TEP**: [74](https://github.com/ton-blockchain/TEPs/pull/4)
- **title**: Стандарт Fungible tokens (Jettons)
- **статус**: Активна
- **Тип**: Контрактный Интерфейс
- **авторы**: [EmelyanenkoK](https://github.com/EmelyanenkoK), [Tolya](https://github.com/tolya-yanot)
- **создан**: 12.03.2022
- **Заменяет**: -
- \*\*заменено \*\*: -

# Summary

Стандартный интерфейс для джеттонов (TON fungible tokens).

# Мотивация

Стандартный интерфейс значительно упростит взаимодействие и отображение различных токенизированных активов.

Jetton стандартное описание:

- The way of jetton transfers.
- Способ получения общей информации (название, распространение и т.д. ) о данном Jetton asset.

# Инструкция

## Полезные ссылки

1. [Ссылка на реализацию jetton](https://github.com/ton-blockchain/token-contract/)
2. [Разработчик Джеттона](https://jetton.live/)
3. Урок FunC Jetton ([en](https://github.com/romanovichim/TonFunClessons_Eng/blob/main/lessons/smartcontract/9lesson/ninthlesson.md)/[ru](https://github.com/romanovichim/TonFunFunClessons_ru/blob/main/lessons/smartcontract/9lesson/ninthlesson.md))

# Спецификация

Здесь и далее мы используем "Jetton" с капиталом "J" как назначение целиком токенов того же типа, в то время как "jetton" с "j" как обозначение количества токенов какого-то типа.

Джеттоны организованы следующим образом: каждый Джеттон имеет мастер-контракт, который используется для мяты новых джеттонов, учета циркуляции поставок и предоставления общей информации.

В то же время информация о количестве джеттонов, принадлежащих каждому пользователю, хранится в индивидуальном (для каждого владельца) смарт-контрактах под названием "Jetton-кошельки".

Пример: если вы освободите Jetton с оборотными поставками 200 реактивных самолетов, которые принадлежат 3 человека, затем вы развернете 4 контракта: 1 Jetton-master и 3 Jetton-кошелька.

## Jetton кошелек смарт-контракт

Требуется реализация:

### Внутренние обработчики сообщений

#### 1. `transfer`

**Запрос**

TL-B схема входящего сообщения:

```
transfer#0f8a7ea5 query_id:uint64 amount:(VarUInteger 16) destination:MsgAddress
                 response_destination:MsgAddress custom_payload:(Maybe ^Cell)
                 forward_ton_amount:(VarUInteger 16) forward_payload:(Either Cell ^Cell)
                 = InternalMsgBody;
```

`query_id` - произвольный номер запроса.

`amount` - количество переданных джеттонов в элементарных единицах.

`destination` - адрес нового владельца самолетов.

`response_destination` - адрес, где отправить ответ с подтверждением успешного перевода и остальной части входящего сообщения Toncoins.

`custom_payload` - пользовательские данные (которые используются либо отправителем, либо ресивером jetton для внутренней логики).

`forward_ton_amount` - количество нанотонов, отправляемых на адрес назначения.

`forward_payload` - необязательные пользовательские данные, которые должны быть отправлены на адрес назначения.

**Должно быть отклонено, если:**

1. сообщение не от владельца.
2. на кошельке отправителя недостаточно джеттонов
3. нет достаточного TON (по отношению к собственным рекомендациям по сборам за хранение данных и стоимости операции) для обработки операции, установки Jetton-кошелька получателя и отправки `forward_ton_amount`.
4. После обработки запроса **должны** отправить хотя бы `in_msg_value - forward_ton_amount - 2 * max_tx_gas_price - 2 * fwd_fee` по адресу `response_destination`.
   Если Jetton-кошелек отправителя не может гарантировать это, он должен немедленно прекратить выполнение запроса и вызвать ошибку.
   Цена `max_tx_gas_price` - это цена в Toncoins максимального лимита газа в цепочке сред обитания FT. За базовый код можно получить в разделе [`ConfigParam 21`](https://github.com/ton-blockchain/ton/blob/78e72d3ef8f31706f30debaf97b0d9a2dfa35475/crypto/block/block.tlb#L660) из поля `gas_limit`.  For the basechain it can be obtained from [`ConfigParam 21`](https://github.com/ton-blockchain/ton/blob/78e72d3ef8f31706f30debaf97b0d9a2dfa35475/crypto/block/block.tlb#L660) from `gas_limit` field.

**В противном случае следует делать:**

1. уменьшить сумму jetton на кошельке отправителя на `amount` и отправить сообщение, что увеличит сумму jetton на кошельке получателя (и при необходимости разверните его).
2. если `forward_amount > 0` убедитесь, что jetton-кошелек получателя отправляет сообщение на адрес `destination` с именем `forward_amount` nanotons присоединены и с макетом:
   TL-B схема:

```
transfer_notification#7362d09c query_id:uint64 amount:(VarUInteger 16)
                              sender:MsgAddress forward_payload:(ither Cell ^Cell)
                              = InternalMsgBody;
```

`query_id` должен быть равен `query_id`.

`amount` amount of transferred jettons.

`sender` - адрес предыдущего владельца переданных реактивных самолетов.

`forward_payload` должен быть равен `forward_payload`.

Если `forward_amount` равен нулю, сообщение не должно быть отправлено.

3. Кошелек получателя должен отправить все лишние сообщения в `response_destination` со следующим макетом: схема
   TL-B: `excesses#d53276db query_id:uint64 = InternalMsgBody;
   `query_id`должен быть равен`query_id\`.

#### Формат `forward_payload`

Если вы хотите отправить простой комментарий в `forward_payload`, то `forward_payload` должен начинаться с `0x0000000000` (32-бит unsigned integer равным нулю) и комментарий находится в оставшейся части `forward_payload`.

Если комментарий не начинается с байта `0xff`, комментарий является текстовым сообщением; можно отобразить "как есть" конечному пользователю кошелька (после фильтрации недопустимых и контрольных символов и проверки правильности UTF-8 строки).
Например, пользователи могут указать на цель ("кофе") простой передачи из своего кошелька на кошелек другого пользователя в этом текстовом поле.

С другой стороны, если комментарий начинается с байта `0xff`, оставшаяся часть - это "бинарный комментарий", , который не должен отображаться конечному пользователю как текст (только в качестве шестнадцатеричного дампа при необходимости).
Предназначен для использования "бинарных комментариев" напр. содержит идентификатор покупки для платежей в магазине, который будет автоматически сгенерирован и обработан магазином.

Если `forward_payload` содержит бинарное сообщение для взаимодействия с смарт-контрактом назначения (например, с DEX), то префиксов нет.

Эти правила совпадают с форматом полезной нагрузки, когда просто отправляются Toncoins с обычного кошелька ([Руководство по Smart Contract Guidelines: Internal Messages, 3](https://ton.org/docs/#/howto/smart-contract-guidelines?id=internal-messages)).

#### 2. `burn`

**Запрос**

TL-B схема входящего сообщения:

```
burn#595f07bc query_id:uint64 amount:(VarUInteger 16)
              response_destination:MsgAddress custom_payload:(Maybe ^Cell)
              = InternalMsgBody;
```

`query_id` - произвольный номер запроса.

`amount` - количество сожженных джеттонов

`response_destination` - адрес, где отправить ответ с подтверждением успешной записи и остальными монетами входящего сообщения.

`custom_payload` - необязательные пользовательские данные.

**Должно быть отклонено, если:**

1. сообщение не от владельца.
2. на кошельке отправителя недостаточно джеттонов
3. Не хватает TON, чтобы отправить запрос после обработки запроса, по крайней мере `in_msg_value - max_tx_gas_price` по адресу `response_destination`.
   Если Jetton-кошелек отправителя не может гарантировать это, он должен немедленно прекратить выполнение запроса и вызвать ошибку.

**В противном случае следует делать:**

1. уменьшите сумму jetton на кошельке записи на `amount` и отправьте уведомление мастеру jetton с информацией о записи.
2. Jetton master должен послать все лишние сообщения в `response_destination` со следующим расположением:
   TL-B схема `excesses#d53276db query_id:uint64 = InternalMsgBody;
   `query_id`должен быть равен`query_id\`.

### Получать методы

1. `get_wallet_data()` возвращает `(int balance, slice owner, slice jetton, cell jetton_wallet_code)`
   `balance` - (uint256) количество джеттонов в кошельке.
   `владелец` - (MsgAddress) адрес владельца кошелька;
   `jetton` - (MsgAddress) адрес мастер-адреса Jetton;
   `jetton_wallet_code` - (cell) с кодом этого кошелька;

## Джеттонский мастер контракт

### Получать методы

1. `get_jetton_data()` возвращает `(int total_supply, int mintable, slice admin_address, jetton_content, cell jetton_wallet_code)`
   `total_supply` - (integer) - общее количество выпусков jetton
   `mintable` - (-1/0) - флаг, указывающий может ли количество jetton'ов увеличить
   `admin_address` - (MsgAddressInt) - адрес smart-contrac с управлением Jetton
   `jetton_content` - ячейка - данные в соответствии с [Стандартом токенов данных #64](https://github. om/ton-blockchain/TEPs/blob/master/text/0064-token-data-standard.md)
   `jetton_wallet_code` - ячейка - код кошелька для этого jetton
2. `get_wallet_address(slice owner_address)` возвращает `slice jetton_wallet_address`
   Возвращает адрес Jetton (MsgAddressInt) для этого адреса владельца (MsgAddressInt).

# TL-B schema

```
ничего$0 {X:Type} = Может быть X;
just$1 {X:Type} значение:X = Может быть X;
left$0 {X:Type} {Y:Type} значение:X = Either X Y;
right$1 {X:Type} {Y:Type} значение:Y = Either X Y;
var_uint$_ {n:#} len:(#< n) значение:(uint (len * 8))
         = VarUInteger n;

addr_none$00 = MsgAddressExt;
addr_extern$01 len:(## 9) external_address:(bits len)
             = MsgAddressExt;
anycast_info$_ глубина:(#<= 30) { depth >= 1 }
   rewrite_pfx:(bits depth) = Anycast;
addr_std$10 anycast:(Maybe Anycast)
   workchain_id:int8 address:bits256 = MsgAddressInt;
addr_var$11 anycast:(Maybe Anycast) addr_len:(## 9)
   workchain_id:int32 address:(bits addr_len) = MsgAddressInt;
_ _:MsgAddressInt = MsgAddress;
_ _:MsgAddressExt = MsgAddress;

transfer query_id:uint64 amount:(VarUInteger 16) destination:MsgAddress
           response_destination:MsgAddress custom_payload:(Maybe ^Cell)
           forward_ton_amount:(VarUInteger 16) forward_payload:(either Cell)
           = InternalMsgBody;

transfer_notification query_id:uint64 amount:(VarUInteger 16)
           ssgAddress forward_payload:(либо ячейка ^Cell)
           = InternalMsgBody;

excesses query_id:uint64 = InternalMsgBody;

burn query_id:uint64 amount:(VarUInteger 16)
       response_destination:MsgAddress custom_payload:(Maybe ^Cell)
       = InternalMsgBody;

// ----- Не указано стандартом, но предлагается формат внутреннего сообщения

internal_transfer query_id:uint64 amount:(VarUInteger 16) от:MsgAddress
                     response_address:MsgAddress
                     forward_ton_amount:(VarUInteger 16)
                     forward_payload:(Either Cell ^Cell)
                     = InternalMsgBody;
burn_notification query_id:uint64 amount:(VarUInteger 16)
       ssgAddress response_destination:MsgAddress
       = InternalMsgBody;
```

`crc32('transfer query_id:uint64 amount:VarUInteger 16 destination:MsgAddress response_destination:MsgAddress custom_payload:Maybe ^Cell forward_ton_amount:VarUInteger 16 forward_payload:Либо Cell ^Cell = InternalMsgBody') = 0x8f8a7ea5 & 0x7fffffffff = 0xf8a7ea5`

`crc32('transfer_notification query_id:uint64 amount:VarUInteger 16 sender:MsgAddress forward_payload:either Cell ^Cell = InternalMsgBody') = 0xf362d09c & 0x7fffffff = 0x7362d09c`

`crc32('excesses query_id:uint64 = InternalMsgBody') = 0x553276db | 0x80000000 = 0xd53276db`

`crc32('burn query_id:uint64 amount:VarUInteger 16 response_destination:MsgAddress custom_payload:Maybe ^Cell = InternalMsgBody') = 0x595f07bc & 0x7fffffffff = 0x595f07bc`

`crc32('internal_transfer query_id:uint64 amount:VarUInteger 16 from:MsgAddress response_address:MsgAddress forward_ton_amount:VarUInteger 16 forward_payload:Either Cell ^Cell = InternalMsgBody') = 0x978d4519 & 0x7fffffff = 0x178d4519`

`crc32('burn_notification query_id:uint64 amount:VarUInteger 16 sender:MsgAddress response_destination:MsgAddress = InternalMsgBody') = 0x7bdd97de & 0x7fffffffff = 0x7bdd97de`

# Ничья

Нет способа получить фактический баланс кошелька, потому что при поступлении сообщения с балансом, баланс кошелька может не быть фактическим.

# Обоснование и альтернативы

Распределенная архитектура "Один кошелёк - один контракт" хорошо описана в [стандарте NFT](https://github.com/ton-blockchain/TEPs/blob/master/text/0062-nft-standard.md#rationale-and-alternatives) в пункте "Rationale".

# Предыдущее искусство

1. [Стандарт токена EIP-20](https://eips.ethereum.org/EIPS/eip-20)
2. [Заостренные умные контракты для разработчиков умных контрактов](https://www.youtube.com/watch?v=svOadLWwYaM)

# Нерешенные вопросы

1. Не существует стандартных методов выполнения "безопасной передачи", которая вернет передачу собственности в случае неудачи исполнения контракта.

# Будущие возможности

Вышла идея реализовать [внешние токены сообщения](https://t.me/ton_overview/35) ( [EmelyanenkoK](https://github.com/EmelyanenkoK)).

# Список изменений

31 Авг 2022 - Добавлен формат `forward_payload`.
