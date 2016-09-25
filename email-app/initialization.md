# Инициализация

Во время начальной загрузки приложение должно выполнить несколько действий. Некоторые из них необходимы только в том случае, если вы впервые авторизуете приложение через учетную запись SAFE Network.

#### Contents

<!-- toc -->

![Страница инициализации](/assets/initialization-page.png)

## Авторизация приложения

Приложение отправляет [запрос на авторизацию](https://maidsafe.readme.io/docs/auth) в SAFE Launcher.

#### POST [/auth](https://maidsafe.readme.io/docs/auth)

**initializer_actions.js**

```js
export const authoriseApplication = (data) => {
  return {
    type: ACTION_TYPES.AUTHORISE_APP,
    payload: {
      request: {
        method: 'post',
        url: '/auth',
        data: data
      }
    }
  };
};
```

Функция `authoriseApplication` вызывается с аргументом `AUTH_PAYLOAD`:

**initializer.js**

```js
const AUTH_PAYLOAD = {
  app: {
    name: pkg.productName,
    vendor: pkg.author.name,
    version: pkg.version,
    id: pkg.identifier
  },
  permissions: ['LOW_LEVEL_API']
};
```

Приложение использует информацию из файла `package.json` для получения следующих полей:

* **app.name** (название приложения)
* **app.vendor** (разработчик)
* **app.version** (версия)
* **app.id** (уникальный идентификатор приложения)

Следовательно, данные для авторизации приложения выглядят так:

```json
{
  "app": {
    "name": "SafeMailApp",
    "id": "safe-mail-app",
    "version": "0.0.1",
    "vendor": "MaidSafe"
  },
  "permissions": ["LOW_LEVEL_API"]
}
```

Мы запрашиваем право доступа `LOW_LEVEL_API` для того, чтобы приложение могло использовать низкоуровневые API:

- [Идентификаторы данных](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/0042-launcher-api-v0.6.md#handle-id)
- [Структурированные данные](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/structured_data.md)
- [Неизменяемые данные](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/immutable_data.md)
- [Обновляемые данные](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/appendable_data.md)

> #### info::Для чего запрашиваются эти права?
>
> Так как низкоуровневые API могут использоваться для добавления данных в сеть, существует вероятность того, что данные созданные приложениями не могут быть впоследствии удалены пользователем для освобождения места. Поэтому важно явным образом получать разрешение пользователя на доступ к низкоуровневым API.
>
> Источник: [RFC 42 – SAFE Launcher API v0.6](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/0042-launcher-api-v0.6.md#permission)

SAFE Launcher отобразит окно с основной информацией о приложении и списком запрошенных прав (`LOW_LEVEL_API`). Вы можете авторизовать этот запрос, кликнув на "ALLOW".

![Запрос авторизации](/assets/authorization-request.png)

После того, как вы авторизуете запрос, приложение получит авторизационный токен.

> #### info::Что такое авторизационный токен?
>
> Авторизационные токены нужны для вызова методов API, требующих [авторизованного доступа](https://maidsafe.readme.io/docs/introduction#section-authorised-access). Эти токены ограничены временем длительности сессии, поэтому они будут действительны только пока запущен SAFE Launcher.

## Проверка наличия конфигурации

Приложение должно каким-то образом хранить вашу электронную почту в SAFE Network. Используя [структурированные данные](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/structured_data.md) мы можем хранить закрытую часть информации о вашем email ID и сохраненных сообщениях. Назовем эту часть «корневыми структурированными данными».

Корневым данным при создании присваевается случайный идентификатор. Нам нужно записать полученный идентификатор в файл конфигурации для дальнейшего получения по нему данных. Файл конфигурации будет храниться в корневой папке приложения.

Если во время инициализации приложение обнаружит, что конфигурационный файл уже существует, оно попробует запросить корневые данные. Если приложение не найдет файл конфигурации, то оно автоматически создаст его.

Приложение пытается прочитать конфигурацию из своей корневой папки:

#### GET [/nfs/file/:rootPath/:filePath](https://maidsafe.readme.io/docs/nfs-get-file)

**nfs_actions.js**

```js
export const getConfigFile = token => {
  return {
    type: ACTION_TYPES.GET_CONFIG_FILE,
    payload: {
      request: {
        url: `/nfs/file/${CONSTANTS.ROOT_PATH}/config`,
        headers: {
          'Authorization': token,
          Range: 'bytes=0-'
        },
        responseType: 'arraybuffer'
      }
    }
  };
};
```

<!-- *(Should we explain why we just fetch the first byte of the config file?)* -->

## Если конфигурация не найдена

Приложение создает корневые данные со случайным идентификатором. Все структрированные данные должны иметь идентификатор длиной в 32 байта. Таким образом, приложение генерирует 32 случайных байта для идентификатора:

**app_utils.js**

```js
export const generateCoreStructreId = () => {
  return base64.encode(crypto.randomBytes(32).toString('base64'));
};
```

Мы используем корневые данные для записи вашего email ID и сохраненных сообщений. Поскольку при этом мы не применяем версионирование, можно хранить любое количество сообщений.

Хранимые приложением данные могут быть представлены в виде простого формата [JSON](https://ru.wikipedia.org/wiki/JSON):

```json
{
  "id": "",
  "saved": []
}
```

### Создание корневых данных

Корневые структурированные данные зашифрованы симметричным ключом. Это означает, что никто не сможет увидеть содержимое хранимых данных. Только вы сможете расшифровать их. Также, поскольку мы показываем только актуальные данные, при хранении используются только [неверсионированные структурированные данные](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/structured_data.md#create) (тип 500).

#### POST [/structuredData/:id](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/structured_data.md#create)

**core_structure_actions.js**

```js
export const createCoreStructure = (token, id, data) => ({
  type: ACTION_TYPES.CREATE_CORE_STRUCTURE,
  payload: {
    request: {
      method: 'post',
      url: `/structuredData/${id}`,
      headers: {
        'Tag-Type': CONSTANTS.TAG_TYPE.DEFAULT,
        Encryption: CONSTANTS.ENCRYPTION.SYMMETRIC,
        'Authorization': token,
        'Content-Type': 'text/plain'
      },
      data: new Uint8Array(new Buffer(JSON.stringify(data)))
    }
  }
});
```

### Создание конфигурации

После того, как мы создали структурированные данные, приложение сохраняет полученный идентификатор в конфигурации. Таким образом, приложение сможет получить их в будущем.

Конфигурация сохраняется в закрытой корневой папке приложения, файлы в которой автоматически шифруются и недоступны для чтения посторонним. Таким образом, файл конфигурации также будет автоматически зашифрован и открыт для чтения только его владельцу.

#### POST [/nfs/file/:rootPath/:filePath](https://maidsafe.readme.io/docs/nfsfile)

**nfs_actions.js**

```js
export const writeConfigFile = (token, coreId) => {
  return {
    type: ACTION_TYPES.WRITE_CONFIG_FILE,
    payload: {
      request: {
        method: 'post',
        url: `/nfs/file/${CONSTANTS.ROOT_PATH}/config`,
        headers: {
          'Authorization': token,
          'Content-Type': 'plain/text'
        },
        data: new Uint8Array(Buffer(coreId))
      }
    }
  }
};
```

После успешного создания файла конфигурации приложение переходит к странице создания учетной записи:

![Создание учетной записи](/assets/create-account-page.png)

## Если конфигурация найдена

Используя сохраненный в конфигурации идентификатор, приложение получает по нему информацию о вашей электронной почте (сохраненные письма и email ID).

Прежде чем получить сами корневые данные, приложению необходимо запросить их дескриптор используя API идентификаторов данных.

<!-- *(explain why handles are needed?)* -->

#### GET [/structuredData/handle/:id](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/structured_data.md#get-data-identifier-handle)

**core_structure_actions.js**

```js
export const fetchCoreStructureHandler = (token, id) => ({
  type: ACTION_TYPES.FETCH_CORE_STRUCTURE_HANDLER,
  payload: {
    request: {
      url: `/structuredData/handle/${id}`,
      headers: {
        'Tag-Type': CONSTANTS.TAG_TYPE.DEFAULT,
        'Authorization': token
      }
    }
  }
});
```

### Выборка корневых данных

После успешного получения дескриптора, приложение запрашивает корневые данные вашей электронной почты.

#### GET [/structuredData/:handleId](https://github.com/maidsafe/rfcs/blob/master/text/0042-launcher-api-v0.6/api/structured_data.md#read-data)

**core_structure_actions.js**

```js
export const fetchCoreStructure = (token, id) => ({
  type: ACTION_TYPES.FETCH_CORE_STRUCTURE,
  payload: {
    request: {
      url: `/structuredData/${id}`,
      headers: {
        'Authorization': token,
        'Tag-Type': CONSTANTS.TAG_TYPE.DEFAULT,
        Encryption: CONSTANTS.ENCRYPTION.SYMMETRIC,
        'Content-Type': 'text/plain'
      }
    }
  }
});
```

#### Если в структурированных данных не содержится email ID

Если вы пока не создали email ID, приложение отобразит страницу создания учетной записи:

![Создание учетной записи](/assets/create-account-page.png)

#### Если структурированные данные содержат email ID

Если вы уже создали email ID, приложение перейдет к странице со входящими сообщениями:

![Входящие сообщения](/assets/inbox-page.png)
