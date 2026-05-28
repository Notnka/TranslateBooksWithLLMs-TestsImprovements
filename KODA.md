# KODA — Инструкционный контекст проекта

> **Проект:** TranslateBook with LLMs (TBL)  
> **Репозиторий:** `TranslateBooksWithLLMs` / `TranslateBookWithLLM`  
> **Лицензия:** AGPL-3.0  
> **Язык разработки:** Python 3.8+  
> **Последнее обновление файла:** 2025-01-19

---

## 1. Обзор проекта

**TranslateBook with LLMs** — это приложение для перевода книг, субтитров и документов с помощью больших языковых моделей (LLM). Поддерживается работа как с локальными моделями (через Ollama, LM Studio, llama.cpp), так и с облачными провайдерами (OpenAI, Gemini, OpenRouter, Mistral, DeepSeek, Poe, NVIDIA NIM).

### Ключевые возможности
- **Форматы:** EPUB, SRT, DOCX, TXT.
- **Безлимитный размер:** Интеллектуальная система чанкования (token-based) обрабатывает документы любой длины.
- **Идеальное сохранение форматирования:** EPUB сохраняет стили и структуру; SRT — таймкоды; DOCX — форматирование.
- **Возобновление перевода:** Система чекпоинтов (checkpoints) автоматически сохраняет прогресс.
- **Два прохода (Refinement):** Опциональный второй проход для литературной полировки перевода.
- **TTS (Text-to-Speech):** Генерация аудио на основе переведённого текста (Edge-TTS — бесплатно, облако; Chatterbox — локально, GPU).
- **Глоссарии (Glossary):** Система для консистентного перевода повторяющихся терминов (имена, локации, организации) через веб-интерфейс и CLI.
- **Мультиязычный UI:** Интерфейс переведён на 7 языков (en, fr, es, de, zh-CN, ja, ko) через i18next.
- **Адаптивный контекст:** Автоматическое увеличение контекстного окна при переполнении или зацикливании.
- **Thinking-модели:** Специальная обработка моделей с рассуждением (reasoning), включая автоопределение поведения `think` параметра.
- **RTL-поддержка:** Правильная обработка языков с письмом справа налево.
- **Prompt optimizer:** Система автоматической оптимизации промптов через генетические алгоритмы.

### Поддерживаемые провайдеры
| Провайдер | Тип | API Key |
|-----------|-----|---------|
| **Ollama** | Локальный | Не требуется |
| **OpenAI-Compatible** | Локальный/Облако | Опционально (для локальных серверов) |
| **Poe** ⭐ | Облако | Требуется |
| **OpenRouter** | Облако | Требуется |
| **OpenAI** | Облако | Требуется |
| **Mistral** | Облако | Требуется |
| **DeepSeek** | Облако | Требуется |
| **Gemini** | Облако | Требуется |
| **NVIDIA NIM** | Облако | Требуется |

---

## 2. Архитектура и структура проекта

```
.
├── src/                          # Основной исходный код
│   ├── api/                      # Flask API + WebSocket
│   │   ├── blueprints/           # Маршруты (config, file, glossary, security, translation, tts)
│   │   ├── services/             # Сервисы (file_service, path_validator)
│   │   ├── handlers.py           # Обработчики фоновых задач перевода
│   │   ├── routes.py             # Центральная настройка маршрутов
│   │   ├── websocket.py          # WebSocket-обработчики
│   │   └── translation_state.py  # Управление состоянием переводов
│   ├── core/                     # Ядро логики перевода
│   │   ├── adapters/             # Адаптеры форматов (EPUB, SRT, TXT, DOCX)
│   │   ├── chunking/             # Токен-based чанкинг
│   │   ├── common/               # Общая оркестрация перевода
│   │   ├── docx/                 # Обработка DOCX
│   │   ├── epub/                 # Обработка EPUB (сложная подсистема)
│   │   ├── glossary/             # Система глоссариев
│   │   │   ├── models.py         # Модели данных (Glossary, GlossaryTerm)
│   │   │   ├── store.py          # SQLite CRUD операции
│   │   │   ├── filter.py         # Фильтрация по чанкам (word-boundary, CJK)
│   │   │   ├── injector.py       # Построение glossary-блока для промпта
│   │   │   ├── ner.py            # NER авто-извлечение через LLM
│   │   │   └── cli_loader.py     # Загрузка glossary из JSON/CSV для CLI
│   │   ├── llm/                  # LLM-клиенты, провайдеры, thinking-логика
│   │   ├── llm_client.py         # Унифицированный LLM-клиент
│   │   ├── translator.py         # Основной цикл перевода чанков
│   │   ├── context_optimizer.py  # Адаптивное управление контекстом
│   │   ├── post_processor.py     # Постобработка переведённого текста
│   │   ├── progress_tracker.py   # Токен-based отслеживание прогресса
│   │   ├── srt_processor.py      # Обработка SRT
│   │   └── text_processor.py     # Обработка plain text
│   ├── persistence/              # Персистентность
│   │   ├── checkpoint_manager.py # Система чекпоинтов
│   │   └── database.py           # База данных (SQLite)
│   ├── tts/                      # Text-to-Speech
│   │   ├── providers/            # Провайдеры TTS (edge, chatterbox)
│   │   ├── audio_processor.py    # Обработка аудио
│   │   └── tts_config.py         # Конфигурация TTS
│   ├── utils/                    # Утилиты
│   │   ├── unified_logger.py     # Единое логирование
│   │   ├── file_utils.py         # Работа с файлами
│   │   ├── language_detector.py  # Определение языка
│   │   ├── security.py           # Безопасность
│   │   └── telemetry.py          # Телеметрия
│   ├── web/                      # Веб-интерфейс
│   │   ├── static/               # CSS, JS, изображения, локали
│   │   │   ├── js/               # Модули JS (core, ui, files, glossary, translation, tts, providers, utils)
│   │   │   └── locales/          # i18n файлы (en, fr, es, de, zh-CN, ja, ko)
│   │   └── templates/            # HTML-шаблоны (Jinja2)
│   └── config.py                 # Централизованная конфигурация
├── prompts/                      # Системные промпты и примеры
│   ├── prompts.py                # Генераторы промптов
│   └── examples/                 # Примеры placeholder-форматов
├── docs/                         # Документация
│   ├── CLI.md                    # Полный справочник CLI
│   ├── PROVIDERS.md              # Настройка провайдеров
│   ├── GLOSSARY.md               # Документация по глоссариям
│   ├── TROUBLESHOOTING.md        # Решение проблем
│   ├── DOCKER_DEPLOYMENT.md      # Деплой через Docker
│   └── BENCHMARK.md              # Бенчмарки качества
├── prompt_optimizer/             # Оптимизатор промптов (генетические алгоритмы)
├── Custom_Instructions/          # Стили перевода (викторианский, поэзия, научный и т.д.)
├── yandex_transpate_api_info/    # Справочная документация по Yandex Translate API (неактивный код)
├── translate.py                  # CLI-интерфейс
├── translation_api.py            # Точка входа веб-сервера (Flask + SocketIO)
├── launcher.py                   # Обёртка для PyInstaller
├── requirements.txt              # Зависимости Python
├── pytest.ini                    # Конфигурация тестов
├── .env.example                  # Шаблон конфигурации
├── docker-compose.yml            # Docker Compose
├── DOCKER.md                     # Руководство по Docker
├── start.sh                      # Smart launcher (Linux/macOS)
├── start.bat                     # Quick launcher (Windows)
├── setup-and-update.bat          # Полная установка (Windows)
├── .pre-commit-config.yaml       # Pre-commit hooks
└── KODA.md                       # Этот файл — инструкционный контекст
```

### Архитектурные паттерны
- **Адаптеры форматов:** Каждый формат (EPUB, SRT, TXT, DOCX) имеет свой адаптер, реализующий единый интерфейс `FormatAdapter`.
- **Фабрика провайдеров:** `create_llm_provider()` создаёт нужный провайдер по строке `provider_type`.
- **Асинхронность:** Основной цикл перевода полностью асинхронный (`asyncio`).
- **Token-based прогресс:** Прогресс отслеживается по токенам, а не по чанкам.
- **Placeholder pipeline:** EPUB проходит через сложный pipeline: извлечение → замена тегов на `[idN]` → перевод → валидация → fallback alignment → восстановление.
- **Glossary system:** Встроенная система глоссариев с per-chunk инъекцией, NER авто-извлечением, поддержкой склонений через `|` разделитель.
- **i18n:** Мультиязычный интерфейс через i18next с JSON-локалями.

---

## 3. Сборка, запуск и тестирование

### Предварительные требования
- Python 3.8+
- Git
- Ollama (для локальных моделей) — [ollama.com](https://ollama.com/)
- FFmpeg (для TTS с кодированием Opus)

### Установка и запуск из исходников

**Linux / macOS (Smart Launcher):**
```bash
git clone https://github.com/hydropix/TranslateBooksWithLLMs.git
cd TranslateBookWithLLM
chmod +x start.sh && ./start.sh
```
Скрипт автоматически: создаёт venv, обновляет код из git, устанавливает/обновляет зависимости, создаёт `.env`, запускает сервер.

**Windows (Quick Launch):**
```batch
setup-and-update.bat   :: Первая установка
start.bat              :: Последующие запуски
```

**Прямой запуск (если venv уже готов):**
```bash
# Веб-интерфейс
python translation_api.py

# CLI
python translate.py -i book.epub -sl English -tl Chinese
```

### Docker

**Docker Compose (рекомендуется):**
```bash
cp .env.example .env
# Отредактировать .env
docker-compose up -d
```

**Docker Run (предсобранный образ):**
```bash
docker run -d \
  -p 5000:5000 \
  -v $(pwd)/translated_files:/app/translated_files \
  -e API_ENDPOINT=http://host.docker.internal:11434/api/generate \
  ghcr.io/hydropix/translatebookswithllms:latest
```

### CLI-примеры

```bash
# Базовый перевод
python translate.py -i book.epub -sl English -tl Chinese

# С указанием провайдера и модели
python translate.py -i book.txt --provider openrouter \
  --openrouter_api_key YOUR_KEY -m anthropic/claude-sonnet-4 -tl French

# С опциями улучшения качества
python translate.py -i scanned_book.txt -tl French --text-cleanup --refine

# С генерацией аудио
python translate.py -i book.txt -tl French --tts --tts-bitrate 96k

# С глоссарием
python translate.py -i book.epub -sl Chinese -tl English \
  --glossary wuxia_novel.json
```

### Тестирование

```bash
# Все тесты
pytest

# Только unit-тесты
pytest -m unit

# Без медленных тестов
pytest -m "not slow"

# Только EPUB
pytest -m epub
```

Конфигурация pytest находится в `pytest.ini`. Маркеры: `slow`, `integration`, `unit`, `epub`.

### Pre-commit hooks

```bash
pip install pre-commit
pre-commit install
```

Выполняются:
- Проверка синтаксиса Python (`check-ast`)
- Проверка YAML
- Исправление концов строк (`end-of-file-fixer`, `trailing-whitespace`)
- Сортировка импортов (`isort --profile black --check-only`)
- Unit-тесты (`pytest tests/unit`)

---

## 4. Конфигурация

Основная конфигурация — через файл `.env` (копия `.env.example`).

### Ключевые переменные

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LLM_PROVIDER` | Провайдер LLM | `ollama` |
| `OLLAMA_API_ENDPOINT` | Эндпоинт Ollama | `http://localhost:11434/api/generate` |
| `OPENAI_API_ENDPOINT` | Эндпоинт OpenAI-compatible | `https://api.openai.com/v1/chat/completions` |
| `DEFAULT_MODEL` | Модель по умолчанию | `qwen3:14b` |
| `PORT` | Порт веб-сервера | `5000` |
| `HOST` | Хост сервера | `127.0.0.1` |
| `OUTPUT_DIR` | Директория выходных файлов | `translated_files` |
| `OUTPUT_FILENAME_PATTERN` | Шаблон имени файла | `{originalName} ({targetLang}).{ext}` |
| `REQUEST_TIMEOUT` | Таймаут API (сек) | `900` |
| `MAX_TOKENS_PER_CHUNK` | Макс. токенов на чанк | `450` |
| `SOFT_LIMIT_RATIO` | Мягкий лимит чанкинга | `0.8` |
| `OLLAMA_NUM_CTX` | Размер контекста Ollama | `4096` |
| `AUTO_ADJUST_CONTEXT` | Автоадаптация контекста | `true` |
| `MAX_TRANSLATION_ATTEMPTS` | Макс. попыток перевода | `3` |
| `DEBUG_MODE` | Режим отладки | `false` |
| `SIGNATURE_ENABLED` | Подпись в переводах | `true` |
| `TTS_ENABLED` | TTS по умолчанию | `false` |
| `TTS_PROVIDER` | Провайдер TTS | `edge-tts` |

### API Keys (для облачных провайдеров)
- `OPENROUTER_API_KEY`
- `OPENAI_API_KEY`
- `GEMINI_API_KEY`
- `MISTRAL_API_KEY`
- `DEEPSEEK_API_KEY`
- `POE_API_KEY`
- `NIM_API_KEY`

### Настройки Thinking-моделей
- `UNCONTROLLABLE_THINKING_MODELS` — модели, которые думают даже с `think=false` (qwen3:30b, phi4-reasoning, deepseek-r1, qwq).
- `CONTROLLABLE_THINKING_MODELS` — модели, которые уважают `think=false` (qwen3:8b, qwen3:14b, qwen3.5:9b).
- `ADAPTIVE_CONTEXT_INITIAL_THINKING=6144` — начальный контекст для thinking-моделей.
- `REPETITION_MIN_COUNT_THINKING=15` — более лояльный порог для detection повторений.

### Оптимизатор промптов
Отдельный модуль `prompt_optimizer/` для автоматической оптимизации системных промптов через генетические алгоритмы. Требует:
- `.env` с `OPENROUTER_API_KEY` (для evaluator LLM)
- Ollama с моделью (для translation LLM)
- `pip install pyyaml python-dotenv requests`

Запуск:
```bash
cd prompt_optimizer
python -m prompt_optimizer.optimize --config prompt_optimizer_config.yaml --verbose
```

---

## 5. Правила разработки и стиль кода

### Общие принципы
- **Модульность:** Каждый формат и каждый провайдер — изолированный модуль с единым интерфейсом.
- **Асинхронность:** Все I/O-операции (API-запросы, файловые операции) должны быть асинхронными.
- **Обратная совместимость:** Старые API сохраняются как legacy-обёртки.
- **Конфигурация через `.env`:** Никаких хардкодных секретов или URL. Все настройки — через `src/config.py` → `.env`.
- **Единое логирование:** Используется `unified_logger` с типами событий (`LogType`).

### Python-стиль
- Импорты сортируются `isort` (профиль `black`).
- Типизация: активно используются type hints (`typing` module).
- Докстринги: Google-style docstrings для всех публичных функций.
- Константы: UPPER_CASE в `config.py`.
- Переменные: snake_case.

### Обработка ошибок
- Специфические исключения в `src/core/llm/exceptions.py`: `ContextOverflowError`, `RepetitionLoopError`, `RateLimitError`.
- Retry-логика с экспоненциальной задержкой для API-запросов.
- Fallback: при критических ошибках перевода чанка — сохранение оригинала с маркером `[TRANSLATION_ERROR]`.

### Безопасность
- Валидация путей (`path_validator`) — предотвращение directory traversal.
- API-ключи никогда не логируются полностью (маскировка: `***last4`).
- CORS включён, но контролируется.

### Добавление нового LLM-провайдера
1. Создать класс в `src/core/llm/providers/<name>.py`, унаследовав от `LLMProvider`.
2. Зарегистрировать в `src/core/llm/factory.py` → `create_llm_provider()`.
3. Добавить API-key и endpoint в `src/config.py` и `.env.example`.
4. Обновить WebUI: `provider-manager.js`, `api-key-utils.js`, `validators.js`, `form-manager.js`, `batch-controller.js`, `settings-manager.js`, `translation_interface.html`, `config_routes.py`.
5. Обновить `docs/PROVIDERS.md`.

### Добавление нового формата
1. Создать адаптер в `src/core/adapters/<format>_adapter.py`, реализовав `FormatAdapter`.
2. Зарегистрировать в `src/core/adapters/translate_file.py`.
3. Добавить детектор MIME-типа в `src/utils/file_detector.py`.

---

## 6. Ключевые файлы и их назначение

| Файл | Назначение |
|------|------------|
| `translation_api.py` | Точка входа веб-сервера Flask + SocketIO. Настраивает маршруты, WebSocket, восстановление незавершённых задач, открытие браузера. |
| `translate.py` | CLI-интерфейс. Парсит аргументы, валидирует API-ключи, запускает `translate_file()`, поддерживает TTS и glossary. |
| `launcher.py` | Обёртка для PyInstaller. Создаёт рабочую директорию `TranslateBook_Data`, копирует `.env.example`, генерирует дефолтный `.env`. |
| `src/config.py` | Централизованная конфигурация. Загружает `.env`, определяет константы, dataclass `TranslationConfig`, placeholder-форматы. |
| `src/core/translator.py` | Сердце системы. Цикл перевода чанков с адаптивным контекстом, checkpoint-ами, refinement-проходом. |
| `src/core/llm_client.py` | Унифицированный LLM-клиент. Обёртка над провайдерами: retry, извлечение перевода из тегов, обработка ошибок. |
| `src/core/context_optimizer.py` | `AdaptiveContextManager` — адаптивное увеличение контекста при переполнении. |
| `src/core/adapters/translate_file.py` | Центральная функция `translate_file()` — маршрутизатор форматов. |
| `src/core/glossary/` | Система глоссариев: `store.py` (SQLite), `filter.py` (per-chunk фильтрация), `injector.py` (блок для промпта), `ner.py` (авто-извлечение). |
| `src/api/blueprints/glossary_routes.py` | REST API для глоссариев (CRUD, import/export, NER auto-extract, preview-block). |
| `src/api/blueprints/translation_routes.py` | Маршруты API перевода. Проксирование всех API-ключей в конфиг. |
| `src/persistence/checkpoint_manager.py` | Сохранение/загрузка прогресса перевода. SQLite + файловая система. |
| `src/api/websocket.py` | WebSocket-обработчики для real-time обновления прогресса. |
| `prompts/prompts.py` | Генераторы системных промптов для перевода, субтитров, refinement, OCR cleanup. |
| `docs/GLOSSARY.md` | Полная документация по системе глоссариев. |
| `Custom_Instructions/*.txt` | Стили перевода (викторианский, поэзия, научный, шекспировский и т.д.). |

---

## 7. Особенности работы с кодом

### Чанкинг и токены
- Все форматы используют **token-based чанкинг** через `tiktoken`.
- `MAX_TOKENS_PER_CHUNK=450` (hard limit), `SOFT_LIMIT_RATIO=0.8` — начало поиска границ.
- Формула необходимого контекста: `prompt_tokens + (MAX_TOKENS_PER_CHUNK * 2) + 50`.

### Placeholder pipeline (EPUB)
1. **Извлечение:** HTML-теги заменяются на `[id0]`, `[id1]`...
2. **Группировка:** Смежные теги объединяются в один placeholder.
3. **Перевод:** LLM получает инструкцию сохранить placeholders.
4. **Валидация:** Проверка, что все placeholders на месте.
5. **Fallback (Phase 2):** Если валидация не пройдена — `token_alignment_fallback`.
6. **Восстановление:** Placeholders заменяются обратно на теги.

### Адаптивный контекст
- Начальное значение: `2048` (стандартные модели), `6144` (thinking-модели).
- Шаг увеличения: `2048`. Максимум: `32768`.
- При `RepetitionLoopError` — двойное увеличение.
- При `ContextOverflowError` — увеличение; если невозможно — уменьшение чанка.

### Thinking-модели
- Классификация: `UNCONTROLLABLE_THINKING_MODELS`, `CONTROLLABLE_THINKING_MODELS`, `STANDARD`.
- Автоопределение в runtime: тестовые запросы с `think=true/false`.

### Glossary System
- **Per-chunk инъекция:** Только термины, присутствующие в текущем чанке, попадают в промпт (кап: 50 терминов).
- **Word-boundary matching** для латиницы, **substring matching** для CJK.
- **Склонения:** Разделитель `|` для альтернативных форм (`Москва|Москве|Москвы`).
- **Auto-extract:** LLM сканирует выборки из документа и предлагает кандидатов.
- **Хранение:** Отдельная SQLite БД `data/glossaries.db` (не `jobs.db`).

### i18n (Мультиязычный UI)
- **7 языков:** en, fr, es, de, zh-CN, ja, ko.
- **Технология:** i18next с JSON-локалями в `src/web/static/locales/`.
- **Добавление языка:** Создать папку `src/web/static/locales/<lang>/` с файлами `translation.json`, `glossary.json` и т.д.

---

## 8. Примечания для будущих взаимодействий

- **Никогда не логируй API-ключи полностью.** Используй маскировку `***last4`.
- **При изменении конфигурации** обновляй `.env.example` и документацию в `docs/`.
- **При добавлении провайдера** не забудь WebUI: обновить `provider-manager.js`, `api-key-utils.js`, `validators.js`, `form-manager.js`, `batch-controller.js`, `settings-manager.js`, `translation_interface.html`, `config_routes.py`.
- **EPUB — самый сложный формат.** Любые изменения в placeholder-системе требуют тщательного тестирования на реальных EPUB-файлах.
- **TTS Chatterbox** требует CUDA и конфликтующих зависимостей (PyTorch). Устанавливается отдельно, не включена в `requirements.txt`.
- **Проект активно развивается.** Проверяй `start.sh` на наличие логики автообновления через `git pull`.
- **Docker-образы** публикуются в GitHub Container Registry: `ghcr.io/hydropix/translatebookswithllms`.
- **Для тестирования перевода** используйте небольшие файлы (~10KB) перед запуском на больших книгах.
- **Веб-интерфейс** использует Server-Sent Events (WebSocket) для real-time прогресса; фоновые задачи выполняются в потоках (`threading`).
- **API-ключи в Web UI:** `translation_routes.py` должен прокидывать все ключи провайдеров в конфиг через `_resolve_api_key()`. Проверяй, что новые провайдеры не забыты.
- **Glossary:** При изменениях в glossary-системе обновляй `docs/GLOSSARY.md`.
- **Custom_Instructions:** Содержит стили перевода. При добавлении новых стилей создавай файлы в `Custom_Instructions/`.
- **Prompt optimizer:** Модуль `prompt_optimizer/` использует генетические алгоритмы для автоматической оптимизации промптов.
- **Yandex-справка:** Папка `yandex_transpate_api_info/` содержит только методические материалы, не активный код провайдера.
