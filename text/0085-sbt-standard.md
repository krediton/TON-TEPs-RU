- **TEP**: [85](https://github.com/ton-blockchain/TEPs/pull/85)
- **title**: СБТ Договор
- **статус**: Активна
- **Тип**: Контрактный Интерфейс
- **Авторы**: [Олег Баранов](https://github.com/xssnick), [Нарек Abovyan](https://github.com/Naltox), [Кирилл Emelyanenko](https://github.com/EmelyanenkoK), [Олег Андрее](https://github.com/oleganza)
- **создан**: 09.08.2022
- **Заменяет**: -
- \*\*заменено \*\*: -

# Summary

Токен с душами (SBT) – это особый вид NFT, который не может быть передан. Он включает в себя факультативный сертификат механики с отзывом от властей и доказательствами владения цепями. Владелец может уничтожить свой SBT в любое время.

# Мотивация

Существует полезный тип токена, который позволяет предоставить социальные разрешения/роли или сертификаты некоторым пользователям. Например, он может быть использован на рынках для предоставления скидок владельцам ВБТ или университетам для выдачи аттестационных сертификатов в форме SBT. Механизмы с правом собственности позволяют легко доказать любому договору, который Вы являетесь владельцем некоторых SBT.

# Спецификация

SBT реализует [стандартный NFT интерфейс](https://github.com/ton-blockchain/TIPs/issues/62), но `transfer` всегда должен быть отклонен.

#### 1. `prove_ownership`

TL-B схема входящего сообщения:

```
prove_ownership#04ded148 query_id:uint64 dest:MsgAddress 
forward_payload:^Cell with_content:Bool = InternalMsgBody;
```

`query_id` - произвольный номер запроса.

`dest` - адрес контракта, которому должно быть доказано право собственности на ВОТ.

`forward_payload` - произвольные данные, требуемые целевым контрактом.

`with_content` - если установлен true, SBT контентная ячейка будет включена в сообщение к контракту.

**Должно быть отклонено, если:**

- Адрес отправителя не является адресом владельца.

**Иначе делать:**
Отправьте сообщение со схемой TL-B для `dest` контракта:

```
ownership_proof#0524c7ae query_id:uint64 item_id:uint256 owner:MsgAddress 
data:^Cell revoked_at:uint64 content:(Maybe ^Cell) = InternalMsgBody;
```

`query_id` - номер запроса передан в `prove_ownership`.

`item_id` - идентификатор NFT.

`owner` - адрес текущего владельца.

`data` - ячейка данных передана в `prove_ownership`.

`revoked_at` - unix время, когда SBT был отозван, 0, если это не было.

`content` - содержимое NFT, оно передается, если `with_content` был true в `prove_ownership`.

#### 2. `request_owner`

TL-B схема входящего сообщения:

```
request_owner#d0c3bfea query_id:uint64 dest:MsgAddress 
forward_payload:^Cell with_content:Bool = InternalMsgBody;
```

`query_id` - произвольный номер запроса.

`dest` - адрес контракта, которому должно быть доказано право собственности на ВОТ.

`forward_payload` - произвольные данные, требуемые целевым контрактом.

`with_content` - если установлен true, SBT контентная ячейка будет включена в сообщение к контракту.

**Должно:**

Отправить сообщение с TL-B схемой для «dest» контракта:

```
owner_info#0dd607e3 query_id:uint64 item_id:uint256 initiator:MsgAddress owner:MsgAddress 
data:^Cell revoked_at:uint64 content:(Maybe ^Cell) = InternalMsgBody;
```

`query_id` - номер запроса передан в `prove_ownership`.

`item_id` - идентификатор NFT.

`initiator` - адрес инициатора запроса.

`owner` - адрес текущего владельца.

`data` - ячейка данных передана в `prove_ownership`.

`revoked_at` - unix время, когда SBT был отозван, 0, если это не было.

`content` - содержимое SBT, передается если `with_content` был true в `request_owner`.

#### 3. `уничтожить`

Схема TL-B внутреннего сообщения:

```
уничтожить #1f04537a query_id:uint64 = InternalMsgBody;
```

`query_id` - произвольный номер запроса.

Должно быть отклонено, если:

- Адрес отправителя не является адресом владельца.

В противном случае следует сделать:

- Указать адрес владельца и его право на нуль.
- Отправить сообщение отправителю со схемой `excesses#d53276db query_id:uint64 = InternalMsgBody;`, которая передаст сумму баланса контракта.

#### 4. `revoke`

TL-B схема входящего сообщения:

```
revoke#6f89f5e3 query_id:uint64 = InternalMsgBody;
```

`query_id` - произвольный номер запроса.

**Должно быть отклонено, если:**

- Адрес отправителя не является адресом администрации.
- Был уже отозван

**Иначе должно делать:**
Установить аннулирован_на текущее unix время.

**ПОЛУЧИТЬ методы**

1. `get_nft_data()` - то же, что и в [стандарте NFT](https://github.com/ton-blockchain/TIPs/issues/62).
2. `get_authority_address()` - возвращает `slice`, это адрес автора. Полномочия могут отозвать SBT.
   **Этот метод является обязательным для SBT, если нет прав доступа, он должен вернуть addr_none (2ноль битов)**
3. `get_revoked_time()` - возвращает `int`, то есть unix время отозванного. 0 когда не отозвано.

### Пример осуществления

https://github.com/getgems-io/nft-contracts/blob/main/packages/contracts/sources/sbt-item.fc

# Инструкция

#### Minting

Это может быть сделано с помощью базовой коллекции NFT, ВБТ должен быть товаром. В mint сообщение дополнительно следует передать адрес авторизации, [после содержимого](https://github.com/getgems-io/nft-contracts/blob/main/packages/contracts/sources/sbt-item.fc#L90).

До мяты, эмитенту рекомендуется проверить код кошелька и подтвердить, что он является стандартным кошельком, а не передаваемым договором, который может быть продан третьим лицам.

#### Предоставление вам права собственности на контракты

Контракты SBT позволяют Вам внедрять интересные механические механизмы с контрактами, доказывая собственность в цепочке.

You can send message to SBT, and it will proxify message to target contract with its index, owner's address and initiator address in body, together with any useful for contract payload,
this way target contract could know that you are owner of SBT which relates to expected collection. Контракт мог бы знать, что ВБТ связан с сбором данных путем расчета адреса ВБТ с использованием кода и индекса и сравнения его с отправителем.

Существует 2 метода, которые позволяют использовать эту функциональность, **надежность** и **информацию о собственности**.
Разница в том, что доказательство может называться только владельцем SBT, поэтому предпочтительно использовать при приеме сообщений только от владельца, например в DAO.

##### Доказательство права собственности

**Владелец SBT** может отправить сообщение SBT с помощью этой схемы:

```
prove_ownership#04ded148 query_id:uint64 dest:MsgAddress 
forward_payload:^Cell with_content:Bool = InternalMsgBody;
```

После этого SBT отправит перевод в `dest` со схемой:

```
ownership_proof#0524c7ae query_id:uint64 item_id:uint256 owner:MsgAddress 
data:^Cell revoked_at:uint64 content:(Maybe ^Cell)
```

##### Информация о владельце

\*\*Любой \*\* может отправить сообщение на SBT с помощью этой схемы:

```
request_owner#d0c3bfea query_id:uint64 dest:MsgAddress 
forward_payload:^Cell with_content:Bool = InternalMsgBody;
```

После этого SBT отправит перевод в `dest` со схемой:

```
owner_info#0dd607e3 query_id:uint64 item_id:uint256 initiator:MsgAddress owner:MsgAddress 
data:^Cell revoked_at:uint64 content:(Maybe ^Cell)
```

#### Пример проверки контракта SBT

```C
int op::ownership_proof() asm "0x0524c7ae PUSHINT";

int equal_slices (кусок а, кусок b) asm "SDEQ";

_ load_data() {
    Slice ds = get_data(). egin_parse();

    return (
        ds~load_msg_addr(), ;; collection_addr
        ds~load_ref() ; sbt_code
    );
}

calculate_sbt_address(slice collection_addr, ячейка sbt_item_code, int wc, int index) {
  Данные ячейки = begin_cell(). tore_uint(индекс, 64).store_slice(collection_addr). nd_cell();
  ячейка state_init = begin_cell().store_uint(0, 2).store_dict(sbt_item_code).store_dict(data).store_uint(0, 1). nd_cell();

  return begin_cell(). tore_uint(4, 3)
                     . tore_int(wc, 8)
                     . tore_uint(cell_hash(state_init), 256)
                     . nd_cell()
                     . egin_parse();
}


() recv_internal(int balance, int msg_value, cell in_msg_full, slice in_msg) impure {
  Slice cs = in_msg_full. egin_parse();
  int flags = cs~load_uint(4);

  slice sender_address = cs~load_msg_addr();

  int op = in_msg~load_uint(32);
  int query_id = in_msg~load_uint(64);

  if (op == op::ownership_proof()) {
    int id = in_msg~load_uint(256);

    (коллекция кусочков, sbt_code) = load_data();
    throw_unless(403, equal_slices(sender_address, collection_addr. alculate_sbt_address(sbt_code, 0, id));

    Slice owner_addr = in_msg~load_msg_addr();
    гена = in_msg~load_ref();
    
    int revoked_at = in_msg~load_uint(64);
    throw_if(403, отозвано_at > 0);
    
    int with_content = in_msg~load_uint(1);
    if (with_content ! 0) {
        ячейки sbt_content = in_msg~load_ref();
    }
    
    ;;
    ; sbt проверен, выполните что-нибудь
    ;;

    возврат ();
  }

  броска (0xff);
}
```

# Обоснование и альтернативы

- **Почему именно эта конструкция лучше всего подходит к пространству возможных дизайнов?**

Этот проект позволяет нам использовать SBT в качестве сертификатов с отозванными и onchain доказательствами, и в то же время позволяет сделать истинное SBT если не задан авторитет.

- **Какие другие проекты были рассмотрены и в чем смысл их не выбирать?**

Первоначально была рассмотрена конструкция по аналогии с ETH с выделенными адресами токенами, однако она была расширена с использованием полномасштабных onchain доказательств и варианта аннулирования.

- **Каково влияние этого не делает?**

В настоящее время TON не имеет никакого стандарта для токенов владельца, поэтому проблема заключается в выпуске токенов, которые не могут быть переданы третьим сторонам. Таким образом, если мы игнорируем этот или похожий стандарт, который вводит такую механику, TON может пропустить некоторые интересные и перспективные продукты.

# Предыдущее искусство

В ETH ([EIP-4973 ABT](https://eips.ethereum.org/EIPS/eip-4973)) - SBT был сделан как NFT, который вообще не мог быть передан между счетами. Мы сделали то же самое, но расширили логику доказательствами onchin, и добавил авторитет, который может отозвать, поэтому SBT может быть использован в качестве полнофункционального сертификата.

# Ничья

[EIP-4973 ABT](https://eips.ethereum.org/EIPS/eip-4973) имеет оборудован/unэкипировать механику, которая позволяет временно показать/скрыть SBT. В нынешнем предложении мы можем лишь уничтожить ВОТ. На самом деле не уверен, что для нас необходима логика показа/скрытия.

# Будущие возможности

Стандарт выглядит закончен.
