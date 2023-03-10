## BAT Token format

### структура данных

| Bytes    | Name        | optional          | Value                                                                                        |
|----------|-------------|-------------------|----------------------------------------------------------------------------------------------|
| 1        | version     | false             | 0x01                                                                                         |
| 4        | app_id      | false             | айди приложения из базы                                                                      |
| 4        | token_id    | false             | айди токена из базы                                                                          |
| 1        | is_subtoken | false             | 0x01 или 0x00 - показывает является ли токеным выпущеным клиентом с помощью другогого токена |
| 4        | subtoken_id | is_subtoken==0x01 | айди сабтокена. вероятно случайный                                                           |
| 2        | flags       | false             | uint16 откуда мы берем биты для расширения                                                   |
| 4        | ttl         | flags.0?          | таймстэмп окончания срока жизни токена                                                       |
| 6        | limits      | flags.1?          | лимиты на запросы к апи с этим токеном  (описание отдельно)                                  |
| 5 или 17 | ip_limited  | flags.2?          | айпи для которого ограничены запросы                                                         |
| 32       | signature   |                   | последние 32 байта это подпись hmac_sha256                                                   |

### Описание фич

Расширения завязаны на биты инта flags. 0-й бит это крайний левый бит например 0b1010000000000000 это токен с ттл и привязанный к айпи но без встроенных лимитов.
Без изменения мажорной версии можно добавить до 16 разных расширений. Расширения идут в порядке их расположения битов в переменной flags.

Первые 8 символов в base32 будут уникальными для приложения. Первые 16 символов в base32 описывают приложение + токен + его тип. Это можно использовать для простого декодирования и например логирования.

#### Структура лимитов:

| Bytes | тип     | Значение               |
|-------|---------|------------------------|
| 4     | float32 | rps                    |
| 1     | uint8   | burst относительно rps |
| 1     | bool    | привязан ли лимит к ip |

##### Пример

Rps: 10.0, boost: 3, PerIP: false

Клиент может выполнять до 10 запросов в секунду, при этом до 30 единомоментно независимо от того со скольки айпи пришёл.

##### Пример

Rps: 0.2, boost: 10, PerIP: true

Клиент может выполнять 1 запрос в 5 секунд, при этом до 2 единомоментно с одного айпи или 2 два раза больше с двух айпи.

### Кодирование

Байты токены кодируются в base32 по RFC4648 с обрезанием выравнивающих знаков `=`. При декодировании знаки `=` добавляются до нужной длины.


### Примеры

Токен со всеми расширениями:
AHK65LZNLVSTFJWDAK36NN4NS7LWTBCUYLNAC3VL2BIISQQIS5J6QQ3RMBOP4F7VYXKRQWJA62RCZMWY5A72XXAHHISVCCGGXW4U2

Токен минимально возможной длины:
AGZVOXZ3SLHOAA4TRVL4DWQ5DUZK25MSEIGXAFHZ43TMPGXQ6NZZXEWROYSJHCONGSFITLI
