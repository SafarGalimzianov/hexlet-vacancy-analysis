Чек-лист обзора репозитория
Изучить структуру репозитория и организацию приложений Django
Выявить модели данных для вакансий и связанных сущностей
Проанализировать существующие Views и паттерны использования Inertia
Определить источники данных о вакансиях (парсеры, БД)
Оценить возможности кэширования в Django
Изучить frontend-архитектуру для понимания структуры данных
Найти примеры агрегации и расчета статистики
Обзор репозитория
Общая структура проекта
Это Django-приложение с React-фронтендом. Проект построен по модульной архитектуре. Бэкенд на Django, фронтенд на React с TypeScript и Vite. Используется Inertia.js для связи между Django и React.

Основные директории:

app — корневая директория Django-приложения
services — сервисы приложения (парсеры, авторизация)
frontend — React-приложение с FSD-архитектурой
Ключевые технологии
Backend: Django 5.1, Inertia.js, Telethon
Frontend: React, TypeScript, Vite
База данных: PostgreSQL (через dj-database-url)
Архитектура фронтенда: Feature-Sliced Design
Источники данных о вакансиях
В проекте есть три источника вакансий:

HeadHunter API — hh_parser
SuperJob API — superjob_parser
Telegram-каналы — telegram_parser
Телеграм-парсер использует TelegramParserView для прослушивания каналов. Он парсит сообщения и сохраняет вакансии через SaveDataVacancy.

Модели данных
Хотя сами модели не показаны в выдержках, по коду видно:

Telegram Channels:

Модель Channel с полями: id, username, channel_id, status, last_message_id
Используется в ShowChannelView и AddChannelView
Вакансии:

Парсер телеграма использует SaveDataVacancy для сохранения
KeywordExtractor извлекает ключевые слова и навыки
Существующие Views и паттерны
Примеры Class-Based Views:

TinkoffCallback — хороший пример сложного View:

Обрабатывает OAuth-авторизацию
Использует вспомогательные методы _make_oauth_request, _handle_error
Возвращает Inertia-ответ через inertia_render
ShowChannelView — пример View с обработкой ошибок:

Использует get_object_or_404
Логирует ошибки через logger
Возвращает JsonResponse (для API)
Паттерн Inertia:
```
from inertia import render as inertia_render

# Использование в TinkoffCallback
return inertia_render(request, self.error_page, props={...})
```

URL-маршрутизация
Главный файл маршрутов — urls.py:
```
urlpatterns = [
    path('', views.index, name='index'),
    path('hh/', include('app.services.hh.hh_parser.urls')),
    path('superjob/', include('app.services.superjob.superjob_parser.urls')),
    path('telegram/', include('app.services.telegram.telegram_channels.urls')),
    path('auth/', include('app.services.auth.users.urls')),
]
```

Для /analytics нужно будет добавить маршрут в этот файл.

Настройки приложения
settings.py содержит:

Установленные приложения (INSTALLED_APPS)
Настройки Inertia: INERTIA_LAYOUT
Настройки статики для Vite
Настройки базы данных
Кэширование: В настройках не видно явной конфигурации кэша. Django по умолчанию использует локальную память. Можно добавить Redis или Memcached.

Frontend-архитектура
По PROJECT_STRUCTURE.md используется Feature-Sliced Design:

Слои:

app — инициализация приложения
pages — страницы (здесь будет AnalyticsPage)
widgets — виджеты
features — фичи
entities — бизнес-сущности
shared — общий код
Это важно для структурирования компонентов аналитики.

Примеры работы с данными
Асинхронная работа:

TelegramParserView — асинхронный класс
Использует asyncio и sync_to_async для работы с Django ORM
Агрегация данных:
В run_listener.py:
```
channels = await sync_to_async(
    lambda: list(
        Channel.objects.filter(status='active').values_list(
            'username', flat=True)))()
```
Это показывает, как делать выборки с фильтрацией.

Логирование и обработка ошибок
Все Views используют logging:
```
import logging
logger = logging.getLogger(__name__)

logger.info("Успешная операция")
logger.error(f"Ошибка: {e}")
```
Обработка ошибок в ShowChannelView:

Http404 для несуществующих объектов
IntegrityError, DataError для ошибок БД
Тестирование
Примеры тестов в tests.py:

Используется TestCase от Django
RequestFactory для создания запросов
@patch для мокирования внешних вызовов
Для AnalyticsView тоже нужны будут тесты.

Middleware и CSRF
middleware.py и настройки CSRF в settings.py:
```
CSRF_HEADER_NAME = "HTTP_X_XSRF_TOKEN"
CSRF_COOKIE_NAME = "XSRF-TOKEN"
```
AnalyticsView не требует @csrf_exempt, так как это будет GET-запрос.

CI/CD
djangoCI.yml — GitHub Actions:

Кэширование зависимостей
Линтеры (backend и frontend)
Можно добавить запуск тестов для AnalyticsView
Проверка чек-листа
✅ Структура репозитория изучена — Django-приложение с модульной архитектурой
✅ Модели данных выявлены — вакансии из трех источников, модель Channel
✅ Views и Inertia проанализированы — паттерн inertia_render, примеры CBV
✅ Источники данных определены — HH, SuperJob, Telegram-парсеры
✅ Кэширование оценено — нужно добавить конфигурацию в settings
✅ Frontend-архитектура понятна — FSD, структура для AnalyticsPage
✅ Примеры агрегации найдены — ORM-запросы с фильтрацией

Следующий шаг
Определить структуру моделей вакансий.

Нужно найти или создать модели для:

Вакансий (Vacancy)
Компаний (Company)
Навыков/технологий (Skill, Technology)
Без этого нельзя написать запросы для расчета статистики. Проверьте директории парсеров (hh_parser, superjob_parser, telegram_parser) на наличие файлов models.py.