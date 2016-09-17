# Приложение Email

При помощи этого руководства вы научитесь, как сделать **приложение электронной почты, которое работает на базе SAFE Network**.

Вместо того, чтобы зависеть от сервисов электронной почты вроде Gmail или обременять себя настройкой собственного почтового сервера, вы можете создавать динамические приложения для обмена сообщениями через SAFE Network.

![Страница создания нового сообщения](/assets/compose-mail-page.png)

![Входящая почта](/assets/inbox-page.png)

## Исходный код

**[Исходники для этого примера расположены на GitHub](https://github.com/maidsafe/safe_examples/tree/master/email_app)**

В качестве основы для приложения был использован проект [electron-react-boilerplate](https://github.com/chentsulin/electron-react-boilerplate).

## Краткое содержание

В этом руководстве мы рассмотрим следующие темы:

- Создание ID сообщений
- Отправка сообщений другим пользователям сети
- Проверка наличия новых сообщений
- Сохранение сообщений
- Удаление сообщений

### SAFE Launcher API

Вы изучите следующие API:

- [Авторизация](https://maidsafe.readme.io/docs/auth)
- [Сетевая файловая система (NFS)](https://maidsafe.readme.io/docs/network-file-system-nfs)
- [Идентификаторы данных (Data Identifier)](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/0042-launcher-api-v0.6.md#handle-id)
- [Структурированные данные](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/structured_data.md)
- [Неизменяемые данные](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/immutable_data.md)
- [Обновляемые данные](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/appendable_data.md)

### Пользовательский интерфейс

Интерфейс пользователя создан с использованием следующих библиотек:

- [React](https://facebook.github.io/react/)
- [Redux](http://redux.js.org/)
- [React Router](https://github.com/reactjs/react-router)

Благодаря применению [Electron](http://electron.atom.io/) проект может распространяться как десктопное приложение для Windows, OS X и Linux.

## Установка

Для запуска примера на своем компьютере следуйте дальнейшим инструкциям.

**Убедитесь, что SAFE Launcher запущен**

У вас должен быть установлен и запущен [SAFE Launcher v0.9.0](https://github.com/maidsafe/safe_launcher/releases/tag/0.9.0).

**Убедитесь, что у вас установлен [Node.js](https://nodejs.org/en/)**

```
node -v
```

**Склонируйте [Git-репозиторий](https://github.com/maidsafe/safe_examples) проекта**

```
git clone https://github.com/maidsafe/safe_examples.git
```

**Установите зависимости**

```
cd email_app && npm install
```

**Запустите приложение**

```
npm run dev
```
