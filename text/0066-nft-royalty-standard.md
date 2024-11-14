- **TEP**: [66](https://github.com/ton-blockchain/TEPs/pull/6)
- **title**: стандартное расширение NFTRoyalty
- **статус**: Активна
- **Тип**: Контрактный Интерфейс
- **авторы**: [EmelyanenkoK](https://github.com/EmelyanenkoK), [Tolya](https://github.com/tolya-yanot)
- **создан**: 12.02.2022
- **Заменяет**: -
- \*\*заменено \*\*: -

# Summary

Расширение для [NFT стандарта](https://github.com/ton-blockchain/TEPs/blob/master/text/0062-nft-standard.md).

Стандартизированный способ получения информации о выплатах роялти для негрибных токенов (NFT), с тем чтобы обеспечить всеобщую поддержку платежей роялти на всех рынках и среди участников экосистем.

# Мотивация

Удобно стандартизировать рояльность, так что все рынки NFT будут платить роялти сборщику независимо от того, как эта коллекция была создана.

# Инструкция

1. report_royalty_params пример реализации: [ton-blockchain/token-contract](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/nft/nft-collection.fc#L58-L68).
2. get_royalty_params пример реализации: [ton-blockchain/token-contract](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/nft/nft-collection.fc#L149-L153).

Пример данных роялти в Fift:

```
"EQCD39VS5jcptHL8vMjEXrzGaRcCVYto7HUn4bpAOg8xqB2N" parse-smc-addr drop 2 =: royalty-addr

<b
    11 16 u, // numerator
    1000 16 u, // denominator
    royalty-addr Addr, // address to send royalty
b> =: royalty-params
```

# Спецификация

> Ключевые слова “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “HOULD NOT”, “HOULD NOT”, “ В настоящем документе "РЕКОМЕНДАЦИЯ", "МАЙ" и "OPTIONAL" следует толковать как это описано в RFC 2119.

Умный контракт коллекции NFT ДОЛЖНО реализации:

(если это вариант NFT элементов без коллекции, то смарт-контракт должен это реализовать).

#### Получать методы

1. `royalty_params()` возвращает `(int numerator, int denominator, slice destination)`
   Royalty доля `numerator / denominator`.
   Например, если `numerator = 11` и `denominator = 1000` то доля royalty равна `11 / 1000 * 100% = 1.1%`.
   «числитель» должен быть меньше «знаменатель».
   `destination` - адрес для отправки роялты. Кусочек типа «MsgAddress».

#### Внутренние сообщения

1. `get_royalty_params`
   **Запрос**
   TL-B схема входящего сообщения:
   `get_royalty_params#693d3950 query_id:uint64 = InternalMsgBody;`
   `query_id` - произвольный номер запроса.
   **Should do:**
   Send back message with the following layout and send-mode `64` (return msg amount except gas fees):
   TL-B schema `report_royalty_params#a8cb00ad query_id:uint64 numerator:uint16 denominator:uint16 destination:MsgAddress = InternalMsgBody;`

Ожидается, что рыночные площадки, которые хотят участвовать в платежах роялти, будут отправлять 'muldiv(цена, числитель, знаменатель)' на адрес 'destination' после продажи NFT.

Рыночные площадки ДОЛЖНЫ игнорировать платежи с нулевой стоимостью.

Рынки могут вычесть комиссию за газ и сообщения, требуемую для отправки роялти с суммы роялти.

## TL-B Schema

```
get_royalty_params query_id:uint64 = InternalMsgBody;
report_royalty_params query_id:uint64 numerator:uint16 denominator:uint16 destination:MsgAddress = InternalMsgBody;
```

`crc32('get_royalty_params query_id:uint64 = InternalMsgBody') = 0xe93d3950 & 0x7fffffffff = 0x693d3950`

`crc32('report_royalty_params query_id:uint64 numerator:uint16 denominator:uint16 destination:MsgAddress = InternalMsgBody') = 0xa8cb00ad | 0x80000000 = 0xa8cb00ad`

# Ничья

Для каждой продажи нельзя обеспечить рояльность. Должен быть вариант подарка NFT бесплатно, однако, невозможно отслеживать, это действительно для свободного или нет. См. соответствующий пункт в разделе [TEP-62](https://github.com/ton-blockchain/TEPs/blob/master/text/0062-nft-standard.md#why-are-there-no-obligatory-royalties-to-the-author-from-all-sales).

# Обоснование и альтернативы

## Почему я не могу установить фиксированную сумму вознаграждения?

Мы не знаем, в какой валюте будет продана. Процент роялти универсален.

# Предыдущее искусство

1. [EIP-2981: NFT Royalty Standard](https://eips.ethereum.org/EIPS/eip-2981)

# Нерешенные вопросы

1. Можем ли мы стандартизировать внутреннее сообщение с royalties какие рынки отправляют автору?

# Будущие возможности

Нет
