# Удаление сообщений

Максимальный размер обновляемых данных равен 100 КиБ. Вы можете удалять сообщения стирая их из обновляемых данных. Корневые структурированные данные не имеют лимита по размеру. Вы можете удалить сообщения из папки сохраненных стерев их из корневых структурированных данных.

#### Содержание

<!-- toc -->

![Страница входящих](/assets/inbox-page.png)

## Если сообщение во входящих

Ваши входящие сообщения хранятся в обновляемых данных. Если вы хотите удалить сообщение из входящих, вам достаточно стереть его из обновляемых данных.

### Получение дескриптора обновляемых данных

Приложение получает дескриптор идентификатора связанного с обновляемыми данными:

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

### Удаление сообщения из обновляемых данных

Приложение удаляет сообщение, которое вы только что сохранили. Это действие переместит сообщение в поле `deleted_data`, которое хранит удаленную информацию в обновляемых данных.

#### DELETE [/appendableData/:handleId/:index](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/appendable_data.md#delete-data-by-index)

**appendable_data_actions.js**

```js
export const deleteAppendableData = (token, handleId, index) => {
  return {
    type: ACTION_TYPES.DELETE_APPENDABLE_DATA,
    payload: {
      request: {
        method: 'delete',
        url: `/appendableData/${handleId}/${index}`,
        headers: {
          'Authorization': token
        }
      }
    }
  };
};
```

### Очистка удаленной информации из обновляемых данных

Приложение очищает поле `deleted_data` в обновляемых данных.

#### DELETE /appendableData/clearDeletedData/:handleId

**appendable_data_actions.js**

```js
export const clearDeleteData = (token, handleId) => ({
  type: ACTION_TYPES.CLEAR_DELETE_DATA,
  payload: {
    request: {
      method: 'delete',
      url: `/appendableData/clearDeletedData/${handleId}`,
      headers: {
        'Authorization': token
      }
    }
  }
});
```

### Освобождение дескриптора обновляемых данных

Приложение освобождает дескриптор идентификатора, привязанного к обновляемым данным.

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

## Если сообщение в папке сохраненных

Сохраненные сообщения записаны в корневых структурированных данных. Если вы хотите удалить сообщение из папки сохраненных, вам достаточно стереть его из корневых структурированных данных.

### Обновление корневых структурированных данных

Приложение обновляет JSON, содержащийся в ваших корневых структурированных данных. В нем теперь не будет записано сообщение, которое вы хотите удалить.

#### PUT [/structuredData/:handleId](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/structured_data.md#update-structured-data)

**core_structure_actions.js**

```js
export const updateCoreStructure = (token, id, data) => ({
  type: ACTION_TYPES.UPDATE_CORE_STRUCTURE,
  payload: {
    request: {
      method: 'put',
      url: `/structuredData/${id}`,
      headers: {
        'Content-Type': 'text/plain',
        'Tag-Type': CONSTANTS.TAG_TYPE.DEFAULT,
        Encryption: CONSTANTS.ENCRYPTION.SYMMETRIC,
        'Authorization': token
      },
      data: new Uint8Array(new Buffer(JSON.stringify(data)))
    }
  }
});
```
