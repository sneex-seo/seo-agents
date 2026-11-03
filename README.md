# seo-agents

Мультиагентна SEO-система на OpenAI API. Запит з веб-інтерфейсу розбирає агент-маршрутизатор, задачу виконують спеціалізовані агенти, а агент-тимлід перевіряє результат і за потреби повертає його на доопрацювання. Ролі, цілі та обмеження кожного агента описані в YAML, тому поведінку можна змінювати без правок коду.

## Агенти

| Агент | Що робить |
|---|---|
| `task_router` | Розбирає запит, визначає тип задачі та порядок запуску агентів |
| `language_detector` | Визначає мову контенту: українська, російська чи англійська |
| `semantic_clusterer` | Групує ключі за інтентом і семантичною близькістю, дає рекомендації щодо структури та перелінковки |
| `text_generator` | Пише SEO-текст з урахуванням семантики та цільової аудиторії |
| `meta_generator` | Створює Title, Description, H1, OG-теги та FAQ |
| `link_builder` | Аналізує беклінки з CSV-експорту Ahrefs, рахує ризик для кожного домену і формує disavow-файл |
| `team_lead` | Оцінює результати за шкалою 0-100, прохідний бал 70 |

Конфіги агентів лежать в [`agent_tasks/`](agent_tasks), сценарії та стандарти якості - у [`agent_tasks/system_config.yaml`](agent_tasks/system_config.yaml).

## Сценарії

- **Аналіз посилань:** `link_builder` → `team_lead`
- **Кластеризація:** `semantic_clusterer` → `team_lead`
- **Мета-теги:** `language_detector` → `meta_generator` → `team_lead`
- **Текст:** `language_detector` → `semantic_clusterer` → `text_generator` → `team_lead`
- **Комплексний:** `language_detector` → `semantic_clusterer` → `meta_generator` → `text_generator` → `team_lead`

## Запуск

1. Встановіть залежності:

   ```bash
   pip install fastapi uvicorn pydantic python-multipart openai pyyaml python-dotenv
   ```

2. Створіть файл `simple_config.env` з ключем OpenAI. Файл додано в `.gitignore`.

   ```
   OPENAI_API_KEY=sk-...
   ```

   Без ключа система працює на заглушках, цього достатньо, щоб подивитися інтерфейс.

3. Запустіть сервер і відкрийте http://localhost:8000:

   ```bash
   python start_server.py
   ```

## Файли

- `yaml_seo_system.py` - оркестратор: завантажує YAML-конфіги, викликає агентів, збирає результати.
- `web_interface.py` - FastAPI-сервер з веб-інтерфейсом і WebSocket для статусу виконання.
- `static/` - інтерфейс українською (`index_ua.html`, відкривається за замовчуванням) та російською (`index.html`).
- `prompt.md` - промпт для проєктування наступної версії: шість ролей SEO-команди, база даних і самонавчання агентів.
