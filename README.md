# AI Agent Integration Template (Gemini 3 Pro)

Микросервис для интеграции флагманской модели Gemini 3 Pro Preview в бизнес-логику с упором на надежность и структурированные данные.

### Основной функционал:
- **Strict JSON Output:** Принудительная генерация ответов в формате JSON для автоматической обработки.
- **Data Validation:** Использование Pydantic V2 для проверки целостности данных на лету.
- **Resilience:** Реализован алгоритм экспоненциальной задержки (Backoff) и повторных попыток при сбоях API.
- **Async Engine:** Полностью асинхронная архитектура для работы в высоконагруженных системах.

### Стек:
- Python 3.10+, Gemini 3 Pro Preview API, Pydantic V2, Asyncio.

### Быстрый старт:
1. Создайте `.env` из `.env.example` и вставьте свой `GEMINI_API_KEY`.
2. Установите зависимости: `pip install -r requirements.txt`.
3. Запустите: `python src/main.py`.
