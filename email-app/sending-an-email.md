# Отправка сообщений

Для отправки сообщений другим пользователям SAFE Network вам должны быть известны их email ID.

В первую очередь приложение запрашивает ключ шифрования связанный с обновляемыми данными принадлежащими получателю. Затем используя полученный ключ оно зашифровывает сообщение и сохраняет его как неизменяемые данные. На последнем этапе приложение добавляет неизменяемые данные содержащие сообщение к обновляемым данным получателя.

#### Содержание

<!-- toc -->

![Страница создания нового сообщения](/assets/compose-mail-page.png)

Сообщение выше в формате [JSON](https://en.wikipedia.org/wiki/JSON) будет выглядеть следующим образом:

```json
{
  "subject": "Test",
  "from": "example",
  "time": "Fri, 16 Sep 2016 10:49:44 GMT",
  "body": "123"
}
```

## Получение дескриптора обновляемых данных

Приложение запрашивает дескриптор обновляемых данных с ID соответствующим хешу email ID получателя:

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

## Получение ключа шифрования

После успешного получения дескриптора обновляемых данных, приложение запрашивает открытый ключ получателя. Если мы зашифруем сообщение с помощью этого ключа, только получатель сможет его прочитать. Такой подход известен как [асимметричное шифрование](https://ru.wikipedia.org/wiki/Криптосистема_с_открытым_ключом).

#### GET /appendableData/encryptKey/:handleId

**appendable_data_actions.js**

```js
export const getEncryptedKey = (token, handleId) => ({
  type: ACTION_TYPES.GET_ENCRYPTED_KEY,
  payload: {
    request: {
      url: `/appendableData/encryptKey/${handleId}`,
      headers: {
        'Authorization': token
      }
    }
  }
});
```

## Сохранение сообщения как неизменяемых данных

После успешного запроса открытого ключа получателя, приложение сохраняет сообщение в SAFE Network используя API [неизменяемых данных](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/immutable_data.md).

#### POST [/immutableData](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/immutable_data.md#write-immutable-data-using-self-encryptor)

**immutable_data_actions.js**

```js
export const createMail = (token, data, encryptKeyHandler) => ({
  type: ACTION_TYPES.CREATE_MAIL,
  payload: {
    request: {
      method: 'post',
      url: '/immutableData',
      headers: {
        'Content-Type': 'text/plain',
        encryption: CONSTANTS.IMMUT_ENCRYPTION_TYPE.ASYMMETRIC,
        'encrypt-key-handle': encryptKeyHandler,
        'Authorization': token
      },
      data: new Uint8Array(new Buffer(JSON.stringify(data)))
    }
  }
});
```

Как только операция записи завершится успешно, API вернет нам дескриптор идентификатора данных относящийся к структуре DataMap.

<!-- *(explain what is a DataMap)* -->

## Добавление сообщения в обновляемые данные

Приложение добавляет сообщение (представляемое дескриптором DataMap) в обновляемые данные получателя.

#### PUT [/appendableData/:handleId/:dataIdHandle](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/appendable_data.md#append-data)

**appendable_data_actions.js**

```js
export const appendAppendableData = (token, id, dataId) => ({
  type: ACTION_TYPES.APPEND_APPENDABLE_DATA,
  payload: {
    request: {
      method: 'put',
      url: `/appendableData/${id}/${dataId}`,
      headers: {
        'Authorization': token
      }
    }
  }
});
```

## Освобождение дескриптора обновляемых данных

После успешного добавления вашего сообщения, приложение освободит дескриптор идентификатора данных, который связан с обновляемыми данными получателя.

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
