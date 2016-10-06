# Создание email ID

Если вы еще не создали свой email ID, приложение предложит вам его создать. Email ID - уникальный адрес, которым вы можете поделиться с другими пользователями для получения от них сообщений через SAFE Network. Аналогично, для отправки сообщений пользователям SAFE Network, у вас должны быть их email ID.

#### Содержание

<!-- toc -->

![Создание учетной записи](/assets/create-account-page.png)

> #### info::Чем отличается email ID от адреса email?
>
> Email ID **используется только внутри SAFE Network**. Он не совместим с традиционными приложениями и сервисами электронной почты вроде Gmail. И [существующиее адреса электронной почты](https://ru.wikipedia.org/wiki/Адрес_электронной_почты) (например, *username@example.com*) точно так же несовместимы с этим приложением.
>
> Также SAFE Network не использует традиционную [систему доменных имен](https://ru.wikipedia.org/wiki/DNS) (DNS, обслуживается [ICANN](https://ru.wikipedia.org/wiki/ICANN)), и поэтому нет необходимости использовать специальный формат адресов (например, имя пользователя + домен). Вы можете зарегистрировать любой email ID по вашему усмотрению.

## Создание обновляемых данных

Приложение создает закрытые обновляемые данные по хешу введенного email ID. Если обновляемые данные с ID, соответствующим хешу введенного email ID, уже существуют, вам будет необходимо выбрать другой email ID.

Эти обновляемые данные будут содержать ваши входящие письма. Однако, размер ящика ограничен 100 КиБ. Чтобы избежать потери сообщений, вы можете сохранять их в структурированные данные, для которых не установлено лимитов. Также из обновляемых данных можно удалять ненужные сообщения.

Другие пользователи смогут отправлять вам сообщения, присоединяя их к обновляемым данным. Все сообщения будут зашифрованы с применением вашего открытого ключа шифрования, так что прочитать сообщения сможете только вы.

У всех обновляемых данных должен быть идентификатор длиной в 32 байта. Такой ID приложение получает хешируя ваш email ID:

```js
export const hashEmailId = emailId => {
  return crypto.createHash('sha256').update(emailId).digest('base64');
};
```

#### POST [/appendableData](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/appendable_data.md#create)

**appendable_data_actions.js**

```js
export const createAppendableData = (token, hashedEmailId) => {
  return {
    type: ACTION_TYPES.CREATE_APPENDABLE_DATA,
    payload: {
      request: {
        method: 'post',
        url: '/appendableData',
        headers: {
          'Authorization': token
        },
        data: {
          id: hashedEmailId,
          isPrivate: true,
          filterType: CONSTANTS.APPENDABLE_DATA_FILTER_TYPE.BLACK_LIST,
          filterKeys: []
        }
      }
    }
  };
};
```

## Обновление корневых структурированных данных

После успешного создания обновляемых данных, приложение сохраняет ваш email ID в корневых данных. Таким образом, в будущем приложение сможет запрашивать обновляемые данные, относящиеся к вашей почте.

#### PUT [/structuredData/:id](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/structured_data.md#update-structured-data)

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

#### Пример

Если ваш email ID - **francis**, структура данных будет соответстовать такому JSON:

```json
{
  "id": "francis",
  "saved": []
}
```

## Освобождение дескриптора обновляемых данных

После успешного обновления корневых данных, приложение освобождает дескриптор идентификатора, привязанного к обновляемым данным вашей почты:

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

После того, как дескриптор обновляемых данных успешно освободится, приложение перейдет на страницу входящих.

![Входящие сообщения](/assets/inbox-page.png)
