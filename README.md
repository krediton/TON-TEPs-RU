# Предложения по улучшению TON (TEP)

> :warning: **ВНИМАНИЕ:** эта система предложений в настоящее время является экспериментальной, процесс может быть изменен.

Основная цель предложений по улучшению TON — предоставить удобный и формальный способ предложения изменений в блокчейне TON и стандартизировать способы взаимодействия между различными частями экосистемы.

Управление предложением осуществляется с помощью Pull Request GitHub, этот процесс описывается в [TEP-1](./text/0001-tep-lifecycle.md).

## Создание TEP

1. Сначала обсудите ваше предложение с сообществом, например в чате TON Dev ([en](https://t.me/tondev_eng)/[ru](https://t.me/tondev)).
2. Прочитайте [TEP-1](./text/0001-tep-lifecycle.md) для понимания процесса управления предложением.
3. Сделайте форк этого репозиторий и скопируйте `./0000-template.md` в `./text/0000-my-new-standard.md`, где "my-new-standard" - короткое название вашего TEP.
4. Заполните все разделы и ответы на вопросы, указанные в шаблоне. Если вам нужно добавить изображения, загрузите их в папку `./assets/00-my-new-standard/`.
5. Отправьте Pull request.

## Объединенные TEP

## Активные

| TEP                                          | Название                                              | Тип                | Создан                                     |
| -------------------------------------------- | ----------------------------------------------------- | ------------------ | ------------------------------------------ |
| [1](./text/0001-tep-lifecycle.md)            | Жизненный цикл TEP                                    | Мета               | 11.06.2022 |
| [2](./text/0002-address.md)                  | TON Addresses                                         | Core               | 07.09.2019 |
| [62](./text/0062-nft-standard.md)            | NFT Standard                                          | Contract Interface | 01.02.2022 |
| [64](./text/0064-token-data-standard.md)     | Token Data Standard                                   | Contract Interface | 03.02.2022 |
| [66](./text/0066-nft-royalty-standard.md)    | NFTRoyalty Standard Extension                         | Contract Interface | 12.02.2022 |
| [74](./text/0074-jettons-standard.md)        | Fungible tokens (Jettons) standard | Contract Interface | 12.03.2022 |
| [81](./text/0081-dns-standard.md)            | TON DNS Standard                                      | Contract Interface | 25.06.2022 |
| [85](./text/0085-sbt-standard.md)            | SBT Contract                                          | Contract Interface | 09.08.2022 |
| [89](./text/0089-jetton-wallet-discovery.md) | Discoverable Jettons Wallets                          | Contract Interface | 08.09.2022 |
| [115](./text/0115-ton-connect.md)            | TON Connect                                           | Core               | 20.10.2022 |

## WIP

Since standard truly become a _Standard_  not when it gets merged into this repository, but when multiple parties accept it and use to interact with each other, below we list proposals to which developers may refer in documentation and so on.
In particular "Status" below has the following sense:

- "Proposed" - this standard hasn't implementation or implementation is used only by authors
- "Partially Deployed" - this standard is used by pair of actors (for instance one dApp and one wallet, or similar), but not by interconnected set of actors
- "Deployed" - this standard is used by multiple actors (and generally should be on the way of merging)

| TEP                                                          | Название                            | Тип                | Создан                                     | Status                   |
| ------------------------------------------------------------ | ----------------------------------- | ------------------ | ------------------------------------------ | ------------------------ |
| [91](https://github.com/ton-blockchain/TEPs/pull/91/files)   | Contract Source Registry            | Infrastructure     | 09.09.2022 | ✅Deployed✅               |
| [92](https://github.com/ton-blockchain/TEPs/pull/92/files)   | Wallet Registry                     | Infrastructure     | 11.09.2022 | Proposed                 |
| [96](https://github.com/ton-blockchain/TEPs/pull/96/files)   | Dicts/Arrays in Metadata            | Contract Interface | 21.09.2022 | Proposed                 |
| [104](https://github.com/ton-blockchain/TEPs/pull/104/files) | Data Signatures                     | Contract Interface | 13.12.2022 | Proposed                 |
| [121](https://github.com/ton-blockchain/TEPs/pull/121/files) | Lockable Jetton Wallet              | Contract Interface | 13.04.2023 | Proposed                 |
| [122](https://github.com/ton-blockchain/TEPs/pull/122/files) | Onchain reveal mechanic             | Contract Interface | 31.10.2022 | ✅Deployed✅               |
| [123](https://github.com/ton-blockchain/TEPs/pull/123/files) | Address Guideline update            | Guidelines         | 13.06.2023 | 🛠️Partially Deployed🛠️ |
| [126](https://github.com/ton-blockchain/TEPs/pull/126/files) | Compressed NFT Standard             | Contract Interface | 28.07.2023 | 🛠️Partially Deployed🛠️ |
| [127](https://github.com/ton-blockchain/TEPs/pull/127/files) | TON Storage in Metadata             | Contract Interface | 23.09.2023 | Proposed                 |
| [130](https://github.com/ton-blockchain/TEPs/pull/130/files) | Rebase Jettons standart             | Contract Interface | 04.12.2023 | Proposed                 |
| [131](https://github.com/ton-blockchain/TEPs/pull/131/files) | Referral code in Query ID           | Contract Interface | 26.12.2023 | 🛠️Partially Deployed🛠️ |
| [137](https://github.com/ton-blockchain/TEPs/pull/137/files) | Jetton Wallet Balance Query         | Contract Interface | 09.01.2024 | Proposed                 |
| [140](https://github.com/ton-blockchain/TEPs/pull/140/files) | Programmable Action Phase           | Core               | 20.01.2024 | Proposed                 |
| [141](https://github.com/ton-blockchain/TEPs/pull/141)       | Remote onchain execution            | Core               | 20.01.2024 | Proposed                 |
| [142](https://github.com/ton-blockchain/TEPs/pull/142/files) | TBRC-20 Inscription Token Standard  | Contract Interface | 26.01.2024 | Proposed                 |
| [145](https://github.com/ton-blockchain/TEPs/pull/145/files) | Metadata "Hidden" render type       | Contract Interface | 26.01.2024 | ✅Deployed✅               |
| [146](https://github.com/ton-blockchain/TEPs/pull/146/files) | Semi-fungible token standard        | Contract Interface | 17.03.2024 | Proposed                 |
| [160](https://github.com/ton-blockchain/TEPs/pull/160)       | Dispatch Queue                      | Core               | 13.06.2024 | 🛠️Partially Deployed🛠️ |
| [161](https://github.com/ton-blockchain/TEPs/pull/161/files) | Proxy TON (wTON) | Contract Interface | 13.06.2024 | 🛠️Partially Deployed🛠️ |
