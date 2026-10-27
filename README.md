# Блог Четвёртого моста

Босс раздела PY8 школы BatSchool: блог команды Кельсы на Django выходит в город на PostgreSQL. Условие заказа в школе, на странице этапа «Заказ: Мост выходит в город». В репозитории только обвязка: зависимости, PostgreSQL в Docker, настройки mypy и проверка в CI. Код блога твой: перенеси сюда каталоги `most/`, `blog/` и `accounts/` из своего решения пятнадцатого этапа (их видно в редакторе этапа на странице урока), закоммить и дальше дописывай заказ.

## Как работать локально

Нужны [uv](https://docs.astral.sh/uv/) и Docker.

```sh
uv sync                                   # Python 3.14 и зависимости
docker compose up -d                      # PostgreSQL 17 на localhost:5432
cp .env.example .env
uv run --env-file .env python manage.py migrate
uv run --env-file .env python manage.py createsuperuser
uv run --env-file .env python manage.py runserver
```

Типы: `uv run --env-file .env mypy -p most -p blog -p accounts`. Свои тесты клади в `tests/` и запускай `uv run --env-file .env pytest`.

Боевые настройки проверяет Django сам:

```sh
DJANGO_DEBUG=0 DJANGO_SECRET_KEY="$(openssl rand -base64 48)" DJANGO_ALLOWED_HOSTS=most.city \
  uv run python manage.py check --deploy --fail-level WARNING
```

## Как проверяется

1. Привяжи этот репозиторий на странице настроек школы (владелец репозитория это твой аккаунт GitHub).
2. Каждый пуш запускает `.github/workflows/grade.yml`. Он вызывает общий workflow школы `bondiano/py-graders-ci`, тот поднимает PostgreSQL 17, гоняет скрытые проверки образом `ghcr.io/bondiano/py-grader-blog:16` и отправляет результат в школу. Зелёный прогон засчитывает этап.
3. Не меняй `grade.yml`: школа принимает отчёт только от общего workflow на опубликованном теге.

Тот же прогон можно сделать у себя. Грейдер дописывает свои файлы в каталог проекта, поэтому дай ему копию:

```sh
docker compose up -d
work="$(mktemp -d)" && cp -R . "$work" && rm -rf "$work/.git" && chmod -R a+rwX "$work"
docker run --rm --network host --read-only --tmpfs /tmp \
  -e DATABASE_URL=postgresql://postgres:postgres@localhost:5432/postgres \
  -v "$work:/work" ghcr.io/bondiano/py-grader-blog:16
```

Каждая строка вывода это одна проверка: `"passed": true` или `false` и сообщение.
