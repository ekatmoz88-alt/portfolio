# REST API / OpenAPI Documentation Portfolio

**github.com/ekatmoz88-alt/portfolio**

Практический проект, демонстрирующий:

- **API Reference** и **OpenAPI 3.x** — спецификация как единый источник правды, справочник собирается из неё;
- **endpoint documentation** — повторяемая структура: назначение, параметры, ответы, ошибки, пример;
- **request/response schemas** — типы, обязательность, вложенность, допустимость null для каждого поля;
- **HTTP status** и **error handling** — одна модель ошибки на весь API, таблица кодов с причиной и устранением;
- **API examples** — готовые к копированию запросы и реальные тела ответов;
- **developer guide** — путь онбординга: авторизация → первый запрос → пагинация → вебхуки;
- **webhook documentation** — события, проверка подписи, политика повторных доставок;
- **Docs-as-Code workflow** — исходники в Git, ревью через PR, автоматическая валидация спеки в CI;
- **Git / Markdown** и **versioning** — версионирование документации и changelog;
- **структуру документации для разработчиков** — как артефакты связаны между собой.

---

🇷🇺 [Русская версия](README.ru.md) · 🇬🇧 English version (this file is bilingual, RU below)

---

## Role

Technical Writer / Documentation Designer

## Project type

API Documentation Portfolio Project

## Scope

REST API · OpenAPI 3.x · Developer Documentation

## Focus

OpenAPI 3.x, endpoint reference, request/response schemas, error handling, API examples, Postman and Docs-as-Code workflow.

> **Portfolio disclaimer.** The API documented here is a sample service created for demonstration.
> The subject of the portfolio is the documentation workflow, structure and artifacts — not a live product.

---

## Repository structure

```
.
├── README.md                     ← этот файл
├── README.ru.md                  ← русская версия
├── openapi/
│   └── openapi.yaml              ← OpenAPI 3.1: paths, webhooks, components
├── docs/
│   ├── api-reference.md          ← справочник эндпоинтов
│   ├── error-handling.md         ← HTTP-статусы, единая модель ошибки
│   ├── webhooks.md               ← события, подпись, ретраи
│   ├── developer-guide.md        ← онбординг разработчика
│   └── changelog.md              ← версионирование документации
└── .github/
    └── workflows/
        └── validate-openapi.yml  ← Docs-as-Code: валидация спеки в CI
```

## Deliverables

| Артефакт | Файл | Что демонстрирует |
|---|---|---|
| OpenAPI 3.1 specification | [`openapi/openapi.yaml`](openapi/openapi.yaml) | Валидный исходник: paths, webhooks, переиспользуемые `components` |
| API Reference | [`docs/api-reference.md`](docs/api-reference.md) | Повторяемая структура эндпоинта, таблицы параметров и схем |
| Error handling | [`docs/error-handling.md`](docs/error-handling.md) | Единый формат ошибки, таблица статусов и кодов |
| Webhooks | [`docs/webhooks.md`](docs/webhooks.md) | События, HMAC-подпись, политика повторов |
| Developer guide | [`docs/developer-guide.md`](docs/developer-guide.md) | Путь от получения токена до первого вебхука |
| Changelog | [`docs/changelog.md`](docs/changelog.md) | Версионирование: что изменилось и что сломалось |

## Docs-as-Code workflow

1. Исходники документации — Markdown и YAML в Git.
2. Изменения идут через pull request; ревью происходит в диффе, а не в собранном файле.
3. На каждый PR запускается `.github/workflows/validate-openapi.yml` — валидация спецификации. Спека, которая не проходит линтер, не мержится.
4. Версия документации фиксируется в `changelog.md` синхронно с версией API.

## Verification

Каждый задокументированный эндпоинт проверяется в Postman до публикации: имена параметров, поля ответов и коды статусов сверяются с тем, что API возвращает фактически. Спецификация проходит валидацию в CI.

## Tools

OpenAPI 3.x · Swagger UI · Postman · Git · Markdown · GitHub Actions

## RU ↔ EN

Документация ведётся на двух языках. Терминология фиксируется в глоссарии, поэтому одно понятие сохраняет один термин в обоих языках — RU и EN версии ревьюятся вместе, а не переводятся независимо.

---

# Русская версия

**Роль:** технический писатель / документационный дизайнер
**Тип проекта:** портфолио-проект по документации API
**Область:** REST API · OpenAPI 3.x · документация для разработчиков

## О проекте

Сквозной процесс документирования REST API: контракты эндпоинтов, схемы запросов и ответов, единая модель ошибок, спецификация OpenAPI 3.1, документация вебхуков и гайд для разработчика. Всё ведётся как Docs-as-Code в Git.

Цель — показать, как создаётся каждый артефакт и как он удерживается в синхронном состоянии с API, а не задокументировать существующий продукт.

## Задачи

- **Спроектировала структуру справочника эндпоинтов** — одна повторяемая схема: назначение, параметры запроса, схема ответа, случаи ошибок, готовый к запуску пример.
- **Задокументировала параметры запроса и схемы ответов** — path-, query- и body-параметры с типами, признаком обязательности и ограничениями; поля ответа с типами, вложенностью и допустимостью null.
- **Подготовила примеры запросов** — готовые к копированию `curl` и реальные тела ответов.
- **Структурировала спецификацию OpenAPI 3.1** — paths, webhooks, переиспользуемые параметры, схемы и ответы в `components`.
- **Описала обработку ошибок** — единая модель на весь API: HTTP-статус, машинный код, сообщение для человека, подсказка по устранению.
- **Задокументировала вебхуки** — события, проверка HMAC-подписи, политика повторных доставок, дедупликация.
- **Написала гайд для разработчика** — авторизация → первый запрос → курсорная пагинация → вебхуки.
- **Применила Docs-as-Code** — исходники в Git и Markdown, версионирование, ревью через pull request, валидация спеки в CI.
- **Проверила документацию на точность** — каждый эндпоинт сверен с работающим API в Postman.
