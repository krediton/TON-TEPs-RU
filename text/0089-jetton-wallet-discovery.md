- **TEP**: [89](https://github.com/ton-blockchain/TEPs/pull/89)
- **title**: Открытые Jettons Wallets
- **статус**: Активна
- **Тип**: Контрактный Интерфейс
- **authors**: [sasha1618](https://github.com/sasha1618), [EmelyanenkoK](https://github.com/EmelyanenkoK)
- **создан**: 08.09.2022
- **Заменяет**: -
- \*\*заменено \*\*: -

# Summary

Это предложение предлагает расширить стандартный мастер Jetton за счет добавления обязательного обработчика onchain `provide_wallet_address`.

# Мотивация

Некоторые приложения могут захотеть, чтобы иметь возможность найти свои или другие контрактные кошельки для некоторых специфических Jetton Master. Например, некоторые контракты могут захотеть получить и хранить кошелек Jetton для некоторых Jetton для обработки уведомлений о передаче от него определенным образом.

# Инструкция

При получении сообщения `provide_wallet_address` с адресом в вопросе, Jetton Master должен ответить на сообщение, содержащее адрес.

# Спецификация

## Новые контракты с мастером Jetton

Пример видимого jetton minter кода можно найти [here](https://github.com/ton-blockchain/token-contract/blob/main/ft/jetton-minter-discoverable.fc)

Jetton Master должен обработать сообщение

`provide_wallet_address#2c76b973 query_id:uint64 owner_address:MsgAddress include_address:Bool = InternalMsgBody;`

с суммой TON больше `5000 газовых единиц + msg_forward_prices.lump_price + msg_forward_prices.cell_price` = 0. 061 ПТОН для текущих настроек базы (если количество меньше, чем это невозможно отправить ответ) прилагается

и либо выбросить исключение, если количество входящего значения не достаточно для вычисления адреса кошелька или
ответа с сообщением (отправлено в режиме 64)

`take_wallet_address#d1735400 query_id:uint64 wallet_address:MsgAddress owner_address:(Maybe ^MsgAddress) = InternalMsgBody;`

Примечание: если невозможно сгенерировать адрес кошелька для адреса в вопросе (например, неправильный workchain) `wallet_address` в `take_wallet_address` должен быть равен `addr_none`. Если `include_address` имеет значение `True`, В запросе `take_wallet_address` должен включать `owner_address` равный `owner_address` (в других словах ответ содержит адрес владельца и связанный с ним адрес кошелька jetton).

## Уже существующие контракты с Jetton Master

Джеттоны с необновлённым Jetton Master могут создать отдельный смарт-контракт (обнаружение Джеттона), который реализует эту функциональность. В этом случае пары контрактов (Jetton Master + Jetton Discovery) будут вести себя так же, как и новые Jetton Master. Для необновлённого Jetton Master с обновляемыми метаданными рекомендуется добавить ключ "открытия кошелька" со значением, равным текстовому представлению Jetton Discovery адреса контракта.

## Схем:

```
provide_wallet_address#2c76b973 query_id:uint64 owner_address:MsgAddress include_address:Bool = InternalMsgBody;
take_wallet_address#d1735400 query_id:uint64 wallet_address:MsgAddress owner_address:(Maybe ^MsgAddress) = InternalMsgBody;
```

```
crc32('provide_wallet_address query_id:uint64 owner_address:MsgAddress include_address:Bool = InternalMsgBody') = 2c76b973
crc32('take_wallet_address query_id:uint64 wallet_address:MsgAddress owner_address:Maybe ^MsgAddress = InternalMsgBody') = d1735400
```

# Ничья

Если новые приложения начинают сильно полагаться на это предложение без поддержки отдельных Jetton Master/Jetton Discovery они могут не обработать уже существующие Jetton.

# Обоснование и альтернативы

В настоящее время ожидается, что децентрализованные приложения будут работать с конкретными кошельками не с Jetton Masters. Однако иногда логика становится более сложной (особенно если желательно создать предсказуемые контрактные адреса) или менее простой для пользовательских проверок.

# Предыдущее искусство

-

# Нерешенные вопросы

Jetton Discovery не может отличить Jetton Discovery от контракта не Jetton Discovery, потребляющего более высокую комиссию.

# Будущие возможности

-
