- **TEP**: [62](https://github.com/ton-blockchain/TEPs/pull/2)
- **title**: NFT стандартный
- **статус**: Активна
- **Тип**: Контрактный Интерфейс
- **авторы**: [EmelyanenkoK](https://github.com/EmelyanenkoK), [Tolya](https://github.com/tolya-yanot)
- **создан**: 01.02.2022
- **Заменяет**: -
- \*\*заменено \*\*: -

# Summary

Стандартный интерфейс для нефункциональных токенов.

# Мотивация

Стандартный интерфейс значительно упростит взаимодействие и отображение различных сущностей, представляющих право собственности.

Описание стандарта NFT:

- Порядок владения изменениями.
- Путь объединения предметов в коллекции.
- Путь дедупликации общей части коллекции.

# Инструкция

Не-Fungible Token (NFT) представляет собой собственность над уникальным цифровым активом (котенком изображений, титульных знаков, художественных произведений и т.д.). Каждый отдельный токен является _NFT Item_. Удобно собирать NFT Предметы в _NFT Collection_. В ТОН, каждый элемент NFT и коллекция NFT являются отдельными смарт-контрактами.

## Метаданные NFT

_Основная статья_: TEP-64

Каждый элемент NFT и сама коллекция NFT имеют свои собственные метаданные (TEP-64). Он содержит некоторую информацию о NFT, такую как название и соответствующее изображение. Метаданные могут храниться вне сети (смарт-контракт будет содержать только ссылку на json) или onchain (все данные будут храниться в смарт-контракте).

Пример сбора метаданных (оффлайн):

```json
{
   "image": "https://ton.org/_next/static/media/smart-challenge1.7210ca54.png",
   "name": "TON Smart Challenge #2",
   "description": "TON Smart Challenge #2 Победители Трофи",
   "social_links": []
}
```

Пример метаданных пункта (оффлайн):

```json
{
   "name": "TON Smart Challenge #2 Победители Трофи",
   "Описание": "TON Smart Challenge #2 Трофеи Победителей 1 место из 181",
   "image": "https://ton. rg/_next/static/media/duck.d936efd9.png",
   "content_url": "https://ton.org/_next/static/media/dimond_1_VP9.29bcaf8e.webm",
   "attributes": []
}
```

Оффлайн метаданные публикуются например в Интернете.

## Полезные ссылки

1. [Ссылка на реализацию NFT](https://github.com/ton-blockchain/token-contract/tree/main/nft)
2. [Getgems NFT контракты](https://github.com/getgems-io/nft-contracts)
3. [Проект Toncli NFT для строительства лесов](https://github.com/disintar/toncli/tree/master/src/toncli/projects/nft_collection) by Disintar
4. [TON NFT Deployer](https://github.com/tondiamonds/ton-nft-deployer)
5. FunC Lesson - NFT Standard ([en](https://github.com/romanovichim/TonFunClessons_Eng/blob/889424ae6a28453c4188ad65cdd9dbfeb750ecdb/10lesson/tenthlesson.md)/[ru](https://github.com/romanovichim/TonFunClessons_ru/blob/427037e7937f0e2e9caa4b866ee29f9d8e19b3c0/10lesson/tenthlesson.md))
6. [TON NFT Explorer](https://explorer.tonnft.tools/)

# Спецификация

Коллекция NFT и каждый элемент NFT являются отдельными смарт-контрактами.

Пример: если вы выпускаете коллекцию, которая содержит 10 000 элементов, то вы будете разворачивать 10 001 смарт-контрактов.

## NFT умный контракт

Требуется реализация:

### Внутренние обработчики сообщений

### 1. `transfer`

**Запрос**

TL-B схема входящего сообщения:

`transfer#5fcc3d14 query_id:uint64 new_owner:MsgAddress response_destination:MsgAddress custom_payload:(Maybe ^Cell) forward_amount:(VarUInteger 16) forward_payload:(либо Cell) = InternalMsgBody;`

`query_id` - произвольный номер запроса.

`new_owner` - адрес нового владельца элемента NFT.

`response_destination` - адрес, где отправить ответ с подтверждением успешного перевода и остальными монетами входящего сообщения.

`custom_payload` - необязательные пользовательские данные.

`forward_amount` - количество нанотонов, которое будет отправлено новому владельцу.

`forward_payload` - необязательные пользовательские данные, которые должны быть отправлены новому владельцу.

**Должно быть отклонено, если:**

1. сообщение не от текущего владельца.
2. нет достаточного количества монет (в соответствии с рекомендациями NFT) для обработки операции и отправки `forward_amount`.
3. После обработки запроса **должны** отправить по крайней мере `in_msg_value - forward_amount - max_tx_gas_price` на адрес `response_destination`.
   Если контракт не может гарантировать это, он должен немедленно прекратить выполнение запроса и вызвать ошибку.
   Цена `max_tx_gas_price` - это цена в Toncoins максимального лимита газового газа в цепочке среды обитания NFT. За базовый код можно получить в разделе [`ConfigParam 21`](https://github.com/ton-blockchain/ton/blob/78e72d3ef8f31706f30debaf97b0d9a2dfa35475/crypto/block/block.tlb#L660) из поля `gas_limit`.

**В противном случае следует делать:**

1. изменить текущего владельца NFT на адрес `new_owner`.
2. если `forward_amount > 0` отправьте сообщение на `new_owner` адрес, используя `forward_amount` nanotons прикреплённые и следующего макета:
   TL-B схема: `ownership_assigned#05138d91 query_id:uint64 prev_owner:MsgAddress forward_payload:(Либо ^Cell) = InternalMsgBody;
   `query_id`должен быть равен`query_id`.
   `forward_payload`должен быть равен`forward_payload`.
   `prev_owner`является адресом предыдущего владельца этого элемента NFT.
      Если`forward_amount\` равен нулю, сообщение не должно быть отправлено.
3. Отправьте все лишние сообщения в `response_destination` со следующим макетом:
   TL-B схема `excesses#d53276db query_id:uint64 = InternalMsgBody;
   `query_id`должен быть равен`query_id\`.

### Формат `forward_payload`

Если вы хотите отправить простой комментарий в `forward_payload`, то `forward_payload` должен начинаться с `0x0000000000` (32-бит unsigned integer равным нулю) и комментарий находится в оставшейся части `forward_payload`.

Если комментарий не начинается с байта `0xff`, комментарий является текстовым сообщением; можно отобразить "как есть" конечному пользователю кошелька (после фильтрации недопустимых и контрольных символов и проверки правильности UTF-8 строки).
Например, пользователи могут указать на цель ("кофе") простой передачи из своего кошелька на кошелек другого пользователя в этом текстовом поле.

С другой стороны, если комментарий начинается с байта `0xff`, оставшаяся часть - это "бинарный комментарий", , который не должен отображаться конечному пользователю как текст (только в качестве шестнадцатеричного дампа при необходимости).
Предназначен для использования "бинарных комментариев" напр. содержит идентификатор покупки для платежей в магазине, который будет автоматически сгенерирован и обработан магазином.

Если `forward_payload` содержит бинарное сообщение для взаимодействия с смарт-контрактом назначения (например, с DEX), то префиксов нет.

Эти правила совпадают с форматом полезной нагрузки, когда просто отправляются Toncoins с обычного кошелька ([Руководство по Smart Contract Guidelines: Internal Messages, 3](https://ton.org/docs/#/howto/smart-contract-guidelines?id=internal-messages)).

### 2 `get_static_data`

**Запрос**

TL-B схема входящего сообщения:

`get_static_data#2fcb26a2 query_id:uint64 = InternalMsgBody;`

`query_id` - произвольный номер запроса.

**должно делать:**

1. Отправить обратное сообщение со следующей схемой и send-mode `64` (количество msg кроме газовых сборов):
   TL-B схема: `report_static_data#8b771735 query_id:uint64 index:uint256 collection:MsgAddress = InternalMsgBody;
   `query_id`должен быть равен`query_id`.
   `index`- числовой индекс NFT в коллекции, обычно серийный номер развертывания.`collection\` - адрес смарт-контракта коллекции, к которому принадлежит NFT.

### Получать методы

1. `get_nft_data()` returns `(int init?, int index, slice collection_address, slice owner_address, cell individual_content)`
   `init?` - if not zero, then this NFT is fully initialized and ready for interaction.
   `index` - числовой индекс NFT в коллекции.  Для бесколлекционного NFT - произвольное, но постоянное значение.
   `collection_address` - (MsgAddress) адрес смарт-контракта коллекции, к которому принадлежит NFT. Для безколлекционного NFT этот параметр должен быть addr_none;
   `owner_address` - (MsgAddress) адрес текущего владельца этого NFT.
   `individual_content` - если NFT имеет коллекцию - индивидуальный NFT контент в любом формате;
   если NFT не имеет коллекции - контент NFT в формате, соответствующем стандарту [TEP-64](https://github. om/ton-blockchain/TEPs/blob/master/text/0064-token-data-standard.md).

## Смарт-контракт коллекции NFT

Предполагается, что смарт-контракт коллекции предусматривает контракты на использование интеллектуальных инструментов НФТ в этой коллекции.

Требуется реализация:

### Получать методы

1. `get_collection_data()` возвращает `(int next_item_index, cell collection_content, slice owner_address)`
   `next_item_index` - количество развернутых в коллекции NFT. Как правило, сборник должен выпускать NFT с последовательными индексами (см. Rationale(2) ). Значение `-1` в `next_item_index` используется для обозначения непоследовательных коллекций, такие коллекции должны предоставлять свой способ для перечисления индексов / элементов.
   `collection_content` - содержимое коллекции в формате, соответствующем стандартному [TEP-64](https://github.com/ton-blockchain/TEPs/blob/master/text/0064-token-data-standard.md).
   `owner_address` - адрес владельца коллекции, нулевой адрес, если не владелец.
2. `get_nft_address_by_index(int index)` возвращает `slice address`
   Получает серийный номер элемента NFT этой коллекции и возвращает адрес (MsgAddress) этого элемента NFT интеллектуального контракта.
3. `get_nft_content(int index, cell individual_content)` возвращает `cell full_content`
   Получает серийный номер элемента NFT этой коллекции и индивидуальное содержимое этого элемента NFT и возвращает полное содержимое элемента NFT в формате, соответствующем стандарту [TEP-64](https://github. om/ton-blockchain/TEPs/blob/master/text/0064-token-data-standard.md).
   В качестве примера, если элемент NFT хранит метаданные URI содержимого, то набор смарт-контрактов может хранить домен (например, "https://site.org/") и смарт-контракт NFT элемента в его содержимом будут храниться только отдельные части ссылки (например, "kind-cobra").
   В данном примере метод `get_nft_content` объединяет их и возвращает "https://site.org/kind-cobra".

# Ничья

Нет способа получить текущего владельца NFT в цепочке, потому что TON является асинхронным блокчейном. Когда сообщение с информацией о владельце NFT будет доставлено, эта информация может стать неактуальной, поэтому мы не можем гарантировать, что текущий владелец не изменился.

# Обоснование и альтернативы

1. "Единый NFT - один смарт-контракт" упрощает расчет сборов и позволяет предоставить газовые гарантии.
2. Коллекция NFT с последовательным NFT индексом обеспечивает простой способ объединения и поиск связанных NFT.
3. Деление контента NFT на индивидуальную и общую (коллекционную) часть позволяет выявлять как количественное хранилище, так и дешёвое массовое обновление.

## Почему бы не заключить один смарт-контракт с token_id -> owner_address dictionary?

1. Unpredictable gas consumption
   In TON, gas consumption for dictionary operations depends on exact set of keys.
   Кроме того, TON является асинхронным блокчейном. Это означает, что если вы отправляете сообщение в смарт-контракт, то вы не знаете, сколько сообщений от других пользователей дойдет до смарт-контракта до вашего сообщения.
   Таким образом, вы не знаете, какой размер словаря будет в данный момент, когда ваше сообщение достигнет смарт-контракта.
   Это нормально с простым кошельком -> NFT Smart contract взаимодействие, но неприемлемо с смарт-цепочками контрактов, e. . кошелек -> NFT Smart contract -> Аукционы -> NFT Smart contract.
   Если мы не можем предсказать расход газа, то может возникнуть ситуация, подобно тому, что владелец изменился на смарт-контракте NFT, но на аукционе не было достаточно Тонкоинов.
   Использование смарт-контрактов без словарей дает детерминированное потребление газа.
2. Does not scale (becomes a bottleneck)
   Scaling in TON is based on the concept of sharding, i.e. automatic partitioning of the network into shardchains under load.
   Один большой смарт-контракт популярного NFT противоречит этой концепции. В этом случае многие транзакции будут относиться к одному смарт-контракту.
   Архитектура TON обеспечивает четкие смарт-контракты (см. белый документ), но на данный момент они не реализованы.

## Почему нет "одобрений"?

TON является асинхронным блокчейном, поэтому некоторые синхронные подходы к блокчейну не подходят.

Вы не можете отправить сообщение "Есть ли одобрение?", потому что ответ может стать нерелевантным, пока вам будет отправлено сообщение об ответе.

Если синхронный блокчейн может проверить `alowance` и если все OK делает `transferFrom` в одной транзакции, то в асинхронной цепочке блоков вам всегда нужно будет случайно отправить сообщение «transferFrom», и в случае ошибки, перехватывайте сообщение ответа и выполняйте действия отката.

Это сложный и неуместный подход.

К счастью, все случаи, которые возникли в ходе обсуждения, могут быть реализованы путем регулярной передачи информации о новом владельце. В некоторых случаях это потребует дополнительного смарт-контракта.

Случай, когда вы хотите разместить NFT на нескольких рынках, в то же время решается путем создания смарт-контрактов, которые сначала принимают платеж, и затем NFT отправляется на один из смарт-контрактов на аукционе.

## Почему от всех продаж не существует обязательных авторских прав автору?

В процессе разработки этой идеи, мы пришли к выводу о том, что гарантировать авторские права от абсолютно любой продажи в блокчейне только в 1 случае:

Все трансферты должны осуществляться в рамках открытого долгосрочного аукциона, а другие виды трансфертов запрещены.

Если Вы хотите перенести NFT на другой кошелек, Вам нужно начать аукцион и выиграть его.

Другой вариант этой схемы заключается в том, чтобы все переводы были оплачены.

Запрещая свободный перевод токенов, мы делаем неудобными во многих случаях - пользователь просто обновил кошелек, пользователь хочет пожертвовать NFT, пользователь хочет отправить NFT на какой-то смарт-контракт.

С учетом плохой удобства использования и того, что НПТ являются общей концепцией, и не все они создаются для продажи - этот подход был отклонен.

# Предыдущее искусство

1. [Ethereum NFT Standard (EIP-721)](https://eips.ethereum.org/EIPS/eip-721)
2. [Polkadot NFT Standard (RMRK)](https://github.com/rmrk-team/rmrk-spec)
3. [Cosmos InterNFT стандарты](https://github.com/interNFT/nft-rfc)
4. [Стандарт Everscale NFT (TIP-4.1)](https://docs.everscale.network/standard/TIP-4.1)

# Нерешенные вопросы

1. Индекс владельца еще не реализован, следует ли внедрять его в будущие стандарты?
2. Не существует стандартных методов выполнения "безопасной передачи", которая вернет передачу собственности в случае неудачи исполнения контракта.

# Будущие возможности

Нет

# Стандартные расширения

Функциональность базового стандарта NFT может быть расширена:

- [NFTRoyalty](https://github.com/ton-blockchain/TEPs/blob/master/text/0066-nft-royalty-standard.md)
- [NFTBounceable (Draft)](https://github.com/ton-blockchain/TIPs/issues/67)
- [NFTEditable (Draft)](https://github.com/ton-blockchain/TIPs/issues/68)
- [NFTUpgradable (Draft)](https://github.com/ton-blockchain/TIPs/issues/69)

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

transfer query_id:uint64 new_owner:MsgAddress response_destination:MsgAddress custom_payload:(Maybe ^Cell) forward_amount:(VarUInteger 16) forward_payload:(либо Cell) = InternalMsgBody;

ownership_assigned query_id:uint64 prev_owner:MsgAddress forward_payload:(либо ячейка ^Cell) = InternalMsgBody;

excesses query_id:uint64 = InternalMsgBody;
get_static_data query_id:uint64 = InternalMsgBody;
report_static_data query_id:uint64 index:uint256 collection:MsgAddress = InternalMsgBody;
```

Теги были рассчитаны с помощью tlbc (request_flag равен `0x7fffffff` и флаг ответа равен `0x80000000`):

`crc32('transfer query_id:uint64 new_owner:MsgAddress response_destination:MsgAddress custom_payload:Maybe ^Cell forward_amount:VarUInteger 16 forward_payload:Either Cell ^Cell = InternalMsgBody') = 0x5fcc3d14 & 0x7fffffffff = 0x5fcc3d14`

`crc32('ownership_assigned query_id:uint64 prev_owner:MsgAddress forward_payload:Либо ^Cell = InternalMsgBody') = 0x85138d91 & 0x7fffffffffff = 0x05138d91 `

`crc32('excesses query_id:uint64 = InternalMsgBody') = 0x553276db | 0x80000000 = 0xd53276db`

`crc32('get_static_data query_id:uint64 = InternalMsgBody') = 0x2fcb26a2 & 0x7fffffffff = 0x2fcb26a2`

`crc32('report_static_data query_id:uint64 index:uint256 collection:MsgAddress = InternalMsgBody') = 0xb771735 | 0x80000000 = 0x8b771735`

# Выражение признательности

We are grateful to the [Tonwhales](https://github.com/tonwhales) developers for collaborating on the current draft of the standard 🤝

# Список изменений

[01 Feb 2022](https://github.com/ton-blockchain/TIPs/issues/62#issuecomment-1027167743)

[02 Feb 2022](https://github.com/ton-blockchain/TIPs/issues/62#issuecomment-1028289413)

[04 Feb 2022](https://github.com/ton-blockchain/TIPs/issues/64#issuecomment-1029767906)

[08 Feb 2022](https://github.com/ton-blockchain/TIPs/issues/62#issuecomment-1032979666)

[11 февраля 2022](https://github.com/ton-blockchain/TIPs/issues/62#issuecomment-1036003434)

[30 Июл 2022](https://github.com/ton-blockchain/TIPs/issues/62#issuecomment-1200095572)

31 Авг 2022 - Добавлен формат `forward_payload`.
