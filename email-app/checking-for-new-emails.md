# Проверка новых сообщений

Когда кто-нибудь отправляет вам сообщение, оно сохраняется в обновляемых данных соответствующих хешу вашего email ID. Когда вы обновляете список входящих, приложение запрашивает ваши обновляемые данные и возвращает все содержащиеся там сообщения.

#### Содержание

<!-- toc -->

![Входящие](/assets/inbox-page.png)

## Получение дескриптора обновляемых данных

Приложение получает дескриптор идентификатора данных, который связан с вашими обновляемым данным:

**app_utils.js**

```js
export const hashEmailId = emailId => {
  return crypto.createHash('sha256').update(emailId).digest('base64');
};
```

#### GET [/appendableData/handle/:id](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/appendable_data.md#get-data-identifier-handle)

**appendable_data_actions.js**

```js
export const fetchAppendableDataHandler = (token, id) => { // id => appendable data id
  return {
    type: ACTION_TYPES.FETCH_APPENDABLE_DATA_HANDLER,
    payload: {
      request: {
        url: `/appendableData/handle/${id}`,
        headers: {
          'Authorization': token,
          'Is-Private': true
        }
      }
    }
  };
};
```

## Получение обновляемых данных

Приложение запрашивает метаданные ваших обновляемых данных. Если количество элементов в обновляемых данных равно 0, то это означает, что у вас нет входящих сообщений.

#### HEAD [/appendableData/:handleId](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/appendable_data.md#read-appendable-data)

**appendable_data_actions.js**

```js
export const fetchAppendableData = (token, handlerId) => {
  return {
    type: ACTION_TYPES.FETCH_APPENDABLE_DATA,
    payload: {
      request: {
        method: 'head',
        url: `/appendableData/${handlerId}`,
        headers: {
          'Authorization': token
        }
      }
    }
  };
};
```

### Обработка обновляемых данных

Если количество элементов в обновляемых данных больше 0, приложение обрабатывает их используя цикл. Приложение начинает обработку с индекса 0 и запрашивает идентификатор данных соответствующий первому сообщению.

#### GET [/appendableData/:handleId/:index](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/appendable_data.md#read-appendable-data)

**appendable_data_actions.js**

```js
export const fetchDataIdAt = (token, handlerId, index) => ({
  type: ACTION_TYPES.FETCH_DATA_ID_AT,
  payload: {
    request: {
      url: `/appendableData/${handlerId}/${index}`,
      headers: {
        'Authorization': token
      }
    }
  }
});
```

### Получение сообщения

Приложение запрашивает неизменяемые данне используя идентификатор данных первого сообщения.

#### GET [/immutableData/:handleId](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/immutable_data.md#read-using-self-encryptor)

```js
export const fetchMail = (token, handleId) => ({
  type: ACTION_TYPES.FETCH_MAIL,
  payload: {
    request: {
      url: `/immutableData/${handleId}`,
      headers: {
        'Authorization': token
      },
      responseType: 'arraybuffer'
    }
  }
});
```

Приложение добавляет содержимое первого сообщения в ваш список входящих и повторяет описанные действия для всех последующих сообщений (если они есть). Этот процесс продолжается до тех пор, пока не будут получены все сообщения, содержащиеся в обновляемых данных.

**mail_inbox.js**

```js
fetchMail(handlerId) {
  const { token, fetchMail, pushToInbox } = this.props;
  fetchMail(token, handlerId)
    .then(res => {
      if (res.error) {
        return showError('Ошибка получения сообщения', res.error.message);
      }
      const data = new Buffer(res.payload.data).toString();
      pushToInbox(JSON.parse(data));
      this.currentIndex++;
      if (this.dataLength === this.currentIndex) {
        return this.dropAppendableData();
      }

      return this.iterateAppendableData();
    });
}
```

## Получение количества элементов обновляемых данных

Для отображения занимаемого сообщениями места, приложение запрашивает сериализованное содержимое ваших обновляемых данных и определяет количество содержащихся там элементов.

#### GET /appendableData/serialise/:handleId

**appendable_data_actions.js**

```js
export const getAppendableDataLength = (token, handleId) => ({
  type: ACTION_TYPES.GET_APPENDABLE_DATA_LENGTH,
  payload: {
    request: {
      url: `/appendableData/serialise/${handleId}`,
      headers: {
        'Authorization': token
      },
      responseType: 'arraybuffer'
    }
  }
});
```

## Освобождение обновляемых данных

После обновления списка входящих, приложение освобождает дескриптор идентификатора данных, который привязан к вашим обновляемым данным.

#### DELETE [/dataId/:handleId](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/appendable_data.md#drop-handle)

**data_handle_actions.js**

```js
export const dropHandler = (token, handleId) => ({
  type: ACTION_TYPES.DROP_HANDLER,
  payload: {
    request: {
      method: 'delete',
      url: `/dataId/${handleId}`,
      headers: {
        'Authorization': token
      }
    }
  }
});
```
