# Анализ Swagger контракта client-inf

В `swagger.yaml` описан простой API сервиса клиентских данных.  
База: `https://api.client-service.com/v1`.

Тут только чтение (`GET`), без создания и изменения данных.

Есть 3 операции:

- `GET /clients/{id}` - получить основную инфу о клиенте
- `GET /clients/{id}/documents` - получить документы клиента
- `GET /clients/{id}/relatives` - получить родственников клиента

По моделям все тоже довольно прямолинейно:

- `Client`: `id`, `name`, `age`
- `Document`: `id`, `type`, `number`, `issueDate`, `expiryDate`
- `Relative`: `id`, `relationType`, `name`, `age`

Структура нормальная для REST, но есть проблема: если фронту нужен "полный профиль", надо дергать несколько endpoint-ов.  
То есть один и тот же клиентский сценарий распиливается на 2-3 сетевых вызова.

GraphQL тут как раз удобен, потому что можно запросить только нужные поля и получить все за один вызов.

Ниже схема, которая покрывает текущий REST.

```graphql
scalar Date

type Client {
  id: ID!
  name: String
  age: Int
  documents: [Document!]!
  relatives: [Relative!]!
}

type Document {
  id: ID!
  type: String
  number: String
  issueDate: Date
  expiryDate: Date
}

type Relative {
  id: ID!
  relationType: String
  name: String
  age: Int
}

type Query {
  # аналог GET /clients/{id}
  client(id: ID!): Client

  # аналог GET /clients/{id}/documents
  clientDocuments(clientId: ID!): [Document!]!

  # аналог GET /clients/{id}/relatives
  clientRelatives(clientId: ID!): [Relative!]!
}
```

Соответствие REST -> GraphQL:

- `/clients/{id}` -> `client(id)`
- `/clients/{id}/documents` -> `clientDocuments(clientId)` или `client(id) { documents {...} }`
- `/clients/{id}/relatives` -> `clientRelatives(clientId)` или `client(id) { relatives {...} }`

Пример когда нужен полный профиль клиента:

```graphql
query GetClientProfile($id: ID!) {
  client(id: $id) {
    id
    name
    age
    documents {
      type
      number
    }
    relatives {
      relationType
      name
    }
  }
}
```

Итог: все REST операции покрыты, но в GraphQL меньше дублирования по endpoint-ам и гибче выборка данных под конкретный экран/сценарий.

## P.S.

Помимо GraphGL хочется выделить такую вещь, как data shaping, которая доступна в Restful api. Она позволяет настраивать эндпойнты таким образом, чтобы возвращались только запрашиваемые поля у объекта. 