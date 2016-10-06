# Приложение Email

При помощи этого руководства вы узнаете, как создать **приложение электронной почты, которое работает на базе SAFE Network**.

Вместо того, чтобы зависеть от сервисов электронной почты вроде Gmail или обременять себя настройкой собственного почтового сервера, вы можете создавать динамические приложения для обмена сообщениями через SAFE Network.

![Страница создания нового сообщения](/assets/compose-mail-page.png)

#### Содержание

<!-- toc -->

![Входящая почта](/assets/inbox-page.png)

## Краткое содержание

В этом руководстве мы рассмотрим следующие темы:

- [Создание ID сообщений](create-an-email-id.md)
- [Отправка сообщений другим пользователям сети](send-an-email.md)
- [Проверка наличия новых сообщений](refresh-the-inbox-folder.md)
- [Сохранение сообщений](save-an-email.md)
- [Удаление сообщений](delete-an-email.md)

### API

Вы изучите следующие API:

- [Авторизация](https://api.safedev.org/auth/)
- [Сетевая файловая система (NFS)](https://api.safedev.org/nfs/)
- [Структурированные данные](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/structured_data.md)
- [Неизменяемые данные](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/immutable_data.md)
- [Обновляемые данные](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/appendable_data.md)
- [Идентификаторы данных (Data Identifier)](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/data_identifier.md)

### Пользовательский интерфейс

Интерфейс пользователя создан с использованием следующих библиотек:

- [React](https://facebook.github.io/react/)
- [Redux](http://redux.js.org/)
- [React Router](https://github.com/reactjs/react-router)

## Исходный код

**[Исходники для этого примера расположены на GitHub](https://github.com/maidsafe/safe_examples/tree/master/email_app)**

В качестве основы для приложения был использован проект [electron-react-boilerplate](https://github.com/chentsulin/electron-react-boilerplate).

### Бинарная сборка

[Скачайте **SAFE Mail Tutorial v0.1.0** на GitHub](https://github.com/maidsafe/safe_examples/releases/tag/0.7.0).

Благодаря применению [Electron](http://electron.atom.io/) проект может распространяться как десктопное приложение для Windows, OS X и Linux.

### Компиляция из исходного кода

#### Установка

##### 1. SAFE Launcher

Запустите [SAFE Launcher v0.9.0](https://github.com/maidsafe/safe_launcher/releases/tag/0.9.0) и войдите в систему.

##### 2. Node.js

У вас должен быть установлен Node.js версии 4.6.0 (LTS) или 6.7.0 (текущая).

```
node --version
```

Node.js можно установить множеством способов. Подробные инструкции можно найти на сайте [nodejs.org](https://nodejs.org/en/download/).

#### Установка

##### 1. Склонируйте [Git-репозиторий](https://github.com/maidsafe/safe_examples) проекта

```
git clone https://github.com/maidsafe/safe_examples.git
```

Если у вас не установлен Git, вы можете скачать его на сайте [git-scm.com](https://git-scm.com/downloads).
Либо вы можете скачать код одним ZIP-архивом: https://github.com/maidsafe/safe_examples/archive/master.zip.

##### 2. Установите зависимости

```
cd email_app && npm install
```

##### 3. Запустите приложение

```
npm run dev
```
