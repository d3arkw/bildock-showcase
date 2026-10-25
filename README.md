<div align="center">

# Bildock

### Подключи локальный проект и смотри его метрики в реальном времени

SaaS-сервис для backend-разработчиков: CLI, веб-интерфейс, файловое дерево и состояние сервисов проекта в одном окне

<br>

[![Открыть сервис](https://img.shields.io/badge/Открыть_сервис-bildock.ru-2563eb?style=for-the-badge)](https://bildock.ru)
[![Статус](https://img.shields.io/badge/Статус-MVP_запущен-16a34a?style=for-the-badge)](https://bildock.ru)

![FastAPI](https://img.shields.io/badge/FastAPI-009688?logo=fastapi&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-336791?logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?logo=redis&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![WebSocket](https://img.shields.io/badge/WebSocket-realtime-6366f1)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-CI-2088FF?logo=githubactions&logoColor=white)
![FirstByte](https://img.shields.io/badge/FurstByte-деплой-FC3F1D)

<br>

<table>
  <tr>
    <td align="center"><h2>48</h2>эндпоинтов</td>
    <td align="center"><h2>171</h2>тест</td>
    <td align="center"><h2>~1500</h2>RPS на gateway</td>
    <td align="center"><h2>~8 мс</h2>p95</td>
  </tr>
</table>

<sub>Замеры производительности локальные</sub>

[Скриншоты](#-как-это-выглядит) · [Возможности](#-возможности) · [Архитектура](#-архитектура) · [Решения](#-технические-решения) · [Контакты](#-контакты)

</div>

<br>

> [!NOTE]
> Исходный код закрыт (коммерческий проект). В репозитории только документация и скриншоты. Код покажу на созвоне или дам временный доступ к приватному репозиторию.

## 🧭 Что это

Bildock позволяет создать проект, подключить локальное рабочее окружение через CLI и управлять проектом через веб-интерфейс.

Для каждого проекта отдельно собираются и показываются метрики и состояние его сервисов: нагрузка контейнеров, CPU/RAM, состояние процессов. Это данные именно запущенного проекта, а не всего сервера или компьютера.

## 📸 Как это выглядит

<table>
  <tr>
    <td align="center"><b>Панель метрик</b><br><br><img src="docs/dashboard.png" alt="Панель метрик" width="420"></td>
    <td align="center"><b>Файловое дерево проекта</b><br><br><img src="docs/file-tree.png" alt="Файловое дерево" width="420"></td>
  </tr>
  <tr>
    <td align="center"><b>Подключение через CLI</b><br><br><img src="docs/cli-connect.png" alt="Подключение через CLI" width="420"></td>
    <td align="center"><b>Подписки и оплата</b><br><br><img src="docs/subscriptions.png" alt="Подписки" width="420"></td>
  </tr>
</table>

## ✨ Возможности

<table>
  <tr>
    <td valign="top" width="50%">
      <b>💻 Для разработчика</b>
      <ul>
        <li>Подключение локального проекта через CLI</li>
        <li>Файловое дерево проекта в вебе</li>
        <li>Метрики: контейнеры, CPU/RAM, процессы</li>
        <li>Обновления в реальном времени (WebSocket)</li>
      </ul>
    </td>
    <td valign="top" width="50%">
      <b>🔐 Безопасность</b>
      <ul>
        <li>Вход через email, GitHub и Яндекс (OAuth2)</li>
        <li>JWT с ротацией refresh-токенов в Redis</li>
        <li>Rate limiting (slowapi + Redis)</li>
        <li>HTTPS и gateway перед сервисами</li>
      </ul>
    </td>
  </tr>
  <tr>
    <td valign="top">
      <b>💳 Платежи</b>
      <ul>
        <li>Подписки с оплатой через ЮKassa</li>
        <li>Обработка webhook</li>
        <li>Идемпотентность платёжных операций</li>
      </ul>
    </td>
    <td valign="top">
      <b>⚙️ Инфраструктура</b>
      <ul>
        <li>Деплой в Yandex Cloud</li>
        <li>Docker Compose, логирование</li>
        <li>CI на GitHub Actions за 60-75 секунд</li>
        <li>10 миграций Alembic</li>
      </ul>
    </td>
  </tr>
</table>

## 🏗 Архитектура

<div align="center">
  <img src="docs/architecture.png" alt="Архитектура Bildock" width="850">
</div>

<br>

Микросервисный backend на FastAPI (asyncio): три сервиса (auth, gateway, project), 8 таблиц PostgreSQL. Развёрнут в Yandex Cloud: Docker Compose, HTTPS, gateway, логирование.

<details>
<summary>Схема в виде текста (Mermaid)</summary>

```mermaid
flowchart LR
    WEB[Веб-интерфейс] --> GW[gateway]
    CLI[CLI на машине разработчика] --> GW
    GW --> AUTH[auth]
    GW --> PROJ[project]
    AUTH --> PG[(PostgreSQL)]
    PROJ --> PG
    AUTH --> R[(Redis)]
    GW --> R
    PROJ -. WebSocket .-> WEB
```

</details>

## ✅ Качество

<div align="center">

| Тесты | Покрытие | CI | Производительность gateway |
|:---:|:---:|:---:|:---:|
| **171** на pytest | **81-93%** по сервисам | **60-75 сек** | **~1500 RPS**, p95 **~8 мс** |

<br>

<img src="docs/ci-green.png" alt="CI: все проверки пройдены" width="700">

</div>

## 🛠 Технические решения

**Авторизация.** JWT-авторизация с ротацией refresh-токенов: токены хранятся в Redis, при обновлении выдаётся новая пара. Вход через email, GitHub и Яндекс.

**Платежи.** Подписки оплачиваются через ЮKassa. Webhook обрабатывается идемпотентно, чтобы повторные уведомления не приводили к двойным операциям.

**Защита API.** Rate limiting через slowapi с хранением счётчиков в Redis.

**Реальное время.** Метрики и состояние проекта приходят в интерфейс по WebSocket, без перезагрузки страницы.

## 🚧 Дорожная карта

- [x] MVP запущен: https://bildock.ru
- [x] Авторизация, подписки, CLI, файловое дерево, метрики
- [ ] Модуль анализа логов и ошибок на основе LLM (в разработке)
- [ ] Новые функции для работы с проектами

## 📬 Контакты

**Мищенко Денис**, Python Backend-разработчик (FastAPI)

[![Telegram](https://img.shields.io/badge/Telegram-@d3arkw-26A5E4?logo=telegram&logoColor=white)](https://t.me/d3arkw)
[![Email](https://img.shields.io/badge/Email-denisarkw@yandex.ru-FC3F1D?logo=maildotru&logoColor=white)](mailto:denisarkw@yandex.ru)
[![GitHub](https://img.shields.io/badge/GitHub-d3arkw-181717?logo=github&logoColor=white)](https://github.com/d3arkw)