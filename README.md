# Введение

Пример реализации проекта в архитектуре CQRS, очень сильно сокращенная версия [Reference application Modular monolith with DDD](https://github.com/kgrzybek/modular-monolith-with-ddd)

Применяется в случае, если планируется сделать MVP, который с большой долей вероятности будет потом распилен на микросервисы

## Основные принципы

Проект состоит из 3-х слоев

1. API - слой, содержащий контроллеры
2. Application - слой, содержащий основную бизнес-логику. Ближайший аналог [Use Cases в Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
3. Database - слой данных

Основная логика содержится в Application слое. Слои API и Database зависят от слоя Application и являются относительно него сервисными (т.е. обслуживают его потребности). Каждый слой данных использует свои объекты/модели преобразуемые друг в друга либо маппером (в простом случае) или кодом (в случае если структура объектов сильно различается между слоями).

При развитии приложения реализация слоев API и Database могут быть заменены на другие (например, API может быть заменен на GRPC, Database может быть заменен на Redis полностью или частично и т.д.), при этом слой Application не изменится.

Взаимодействие между слоями происходит через Commands и Query с использованием библиотеки MediatR.

Существуют 2 типа команд: ICommand (IQuery) - для взаимодействия между API и Application слоем и IDBCommand (IDBQuery) для взаимодействия между Application и Database. Запросы (I[DB]Querу) не должны изменять данные, команды (I[DB]Command) могут изменять данные.

Для того, чтобы при разрастании проекта не возникало ситуации, когда есть куча команд/запросов, которые 10 лет назад использовались,
но сейчас уже устарели и не используются все необходимое для выполнения UseCase храниться в одном каталоге.

Например, Category содержит GetAllUseCase в котором есть GetAllCategoriesQuery, GetAllCategoriesQueryHandler (основная логика, в этом случае вырожденная),
GetAllCategoriesDBQuery. В случае, если нам по каким-то причинам не нужен будет этот UseCase мы можем удалить целиком этот каталог
и все зависимости от него (компилятор скажет какие).

## Про наименования

В слое Application наименования должны отображать бизнес-сущности или UseCase.
Если возвращается список чего-то, наименование должно оканчиваться на Item (например CategoryItem).

Допустимо использовать объекты в стиле DDD (см. пример Category) в случаях если производятся операции в стиле CRUD, но в большинстве UseCases скорее всего будут использоваться специфические объекты (например изменение только наименования у категории).

В слое API модели, используемые в контроллерах для входных/выходных параметров именуются:

- <Что-то понятное>Request для входных моделей

- <Что-то понятное>Response для выходных моделей

В слое Database объекты именуются в традиционном стиле в единственном числе, наименование таблиц настроено с применением UseSnakeCaseNamingConvention

## Задание

1. Ответить на вопрос есть ли понимание зачем так извращаться, возможно предложить варианты улучшения (в т.ч. по коду)
2. Предложить какие тесты можно написать в данном примере
3. Написать (можно без реализации Database слоя) команду добавления нового продукта. У каждого продукта должны быть обязательно
   заполнены значения всех параметров категории, к которой относится продукт. Не должно быть значений параметров, которых нет в категории.

## Тестовые примеры

### Изменение категории

```
{
  "id": "a427f99e-d620-49fb-afa3-69b81b41556c",
  "name": "Super toy",
  "description": "Supper pupper toys",
  "attributes": [
    {
      "id": "c732a300-3dd9-46b6-8d38-0b6512212fa1",
      "name": "Height",
      "type": "Int"
    },
    {
      "id": "DA2DAB5C-34DC-4C58-A2C9-2E0B6F67FB6D",
      "name": "Width",
      "type": "Int"
    }

  ]
}
```
