# BipExchangeBot

## Описание

Бот является копией сервиса по обмену [bib.dev](https://bip.dev/ "bib.dev")

## Установка

    1. Установить postgreSQL. BipExchangeBot используется 11.
    2. Создать базу данных.
    3. Установить параметры .ENV

## Возможности сервиса.

- [x] Показать текущий курс BIP/USD.
- [x] Покупка BIP. 
- [ ] Продать BIP.
- [ ] Показать лоты на продажу.

## Возможности бота.

- [x] Отображение привествия.
- [x] Команда start.
- [ ] Команда settings. Для изменения языка.
- [ ] Переделать главное меню из кнопок в клавиатуру.
- [ ] Сделать возможность показать адреса, которые вводил ранее пользователь как подсказки.

### Показать текущий курс.

Бот выполняет [запрос]( https://minterteam.github.io/bipdev-docs/#tag/Price/ "link title") и получает текующий курс.

### Покупка BIP.

 1. При нажатии кнопки "Купить BIP" бот собирает данные: минтер адрес (куда прислать купленные BIP) и email адрес.
 2. Бот передает собранные данный в [запрос](https://minterteam.github.io/bipdev-docs/#tag/Price) и получает адрес для депозита.
 3. При отправки адреса запускается горутина `CheckStatusBuy`. Таймер по умолчанию установлен на 2 минуты - время для депозита BTC. 	     4. По истечению этого времени бот прекращает мониторить эту операцию и выходит из горутины.
 5. Если деньги пришли вовремя, то бот отправляет уведомление **новый депозит** и начинает [опрос](https://minterteam.github.io/bipdev-docs/#operation/getBitcoinAddressStatus) пока у транзакции перевода BTC не будет 2 подтверждения. 
 6. Как только это случилось, то бот отправляет уведомление, что **обмен успешен** и монеты BIP пришли на счет.

### Продать BIP.

   1. При нажатии кнопки "Продать BIP" бот собирает данные: биткоин адрес (куда прислать btc с проданных BIP) и название монеты, цену за монету.
   2. Бот передает собранные данный в [запрос](https://minterteam.github.io/bipdev-docs/#operation/getMinterDepositAddress) и получает адрес для депозита и тэг.
   3. При отправки адреса запускается горутина `CheckStatusSell`. Таймер по умолчанию установлен на 2 минуты - время для депозита BIP.     
   4. По истечению этого времени бот прекращает мониторить эту операцию и выходит из горутины.
   5. Если деньги пришли вовремя, то бот отправляет уведомление **новый депозит**.
   6. Лот сохраняется в базу данных.
   7. Каждый раз как только пользователь пишет боту, обновляется история его адреса и информация сколько было продано монет с лота. 
   8. Если количество монет не изменилось с момента их внесения за неделю, то лот удалятся из базы данных.

### Показать лоты на продажу.

    1. При нажатии на кнопку "Показать лоты на продажу" бот делает запрос к базе данных, получая все лоты, связанные с ID пользователя и отправляя ему: уникальный tag, название монеты, цену за монету, остаток монет, дату последней покупки монет и дату размещения лота.

## База данных.

    Реляционная база данных PostgeSQL 11. 

### Таблицы

- Users. Хранит язык, который выбрал пользователь.
```
create table if not exists users (
	id INT NOT NULL,
   	chat_id BIGINT NOT NULL,
   	lang VARCHAR(8),
   	PRIMARY KEY (id)
);
```
- Sales. Хранит информации о размещении лота пользователем и истории продажи этих монет.
```
create table if not exists sales (
	id SERIAL,
	user_id INT NOT NULL,
	tag VARCHAR(255),
	coin VARCHAR(255),
	price INT,
	amount VARCHAR(255),
	minter_address VARCHAR(255),
	created_at timestamp,
	last_sell_at timestamp,
	PRIMARY KEY (id),
	FOREIGN KEY (user_id) REFERENCES users (id)
); 
```
