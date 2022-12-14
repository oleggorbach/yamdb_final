# api_yamdb

api_yamdb

![CI YamDB workflow](https://github.com/oleggorbach/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg)

### Проект YaMDb

Проект **YaMDb** собирает отзывы (Review) пользователей на произведения (Titles). Произведения делятся на категории: «Книги», «Фильмы», «Музыка». Список категорий (Category) может быть расширен администратором (например, можно добавить категорию «Изобразительное искусство» или «Ювелирка»).
Сами произведения в **YaMDb** не хранятся, здесь нельзя посмотреть фильм или послушать музыку.
В каждой категории есть произведения: книги, фильмы или музыка. Например, в категории «Книги» могут быть произведения «Винни-Пух и все-все-все» и «Марсианские хроники», а в категории «Музыка» — песня «Давеча» группы «Насекомые» и вторая сюита Баха.
Произведению может быть присвоен жанр (Genre) из списка предустановленных (например, «Сказка», «Рок» или «Артхаус»). Новые жанры может создавать только администратор.
Благодарные или возмущённые пользователи оставляют к произведениям текстовые отзывы (Review) и ставят произведению оценку в диапазоне от одного до десяти (целое число); из пользовательских оценок формируется усреднённая оценка произведения — рейтинг (целое число). На одно произведение пользователь может оставить только один отзыв.

### Инструкции по запуску проекта:

1. Склонировать репозиторий через консоль:
git clone https://github.com/oleggorbach/yamdb_final.git

2. Создать .env файл внутри директории infra (на одном уровне с docker-compose.yaml) Пример .env файла:
 * SECRET_KEY - Используемый Django секретный ключ
 * DB_ENGINE - движок базы данных (ex. django.db.backends.postgresql)
 * DB_NAME - имя базы данных (ex. postgres)
 * POSTGRES_USER - Пользователь базы данных (ex. postgres)
 * POSTGRES_PASSWORD - Пароль рользователя базы данных (ex. postgres)
 * DB_HOST - Адрес сервера базы данных (ex. db)
 * DB_PORT - Порт сервера базы данных (ex. 5432)

3. Запуск тестов (опционально, если не нужно - переходите к следующему шагу) Создать и активировать виртуальное пространство, установить зависимости.
Для Windows:
cd yamdb_final
python -m venv venv
source venv/Scripts/activate
cd api_yamdb
pip install -r requirements.txt
cd ..
pytest

Для Mac/Linux:

cd yamdb_final
python3 -m venv venv
source venv/bin/activate
cd api_yamdb
pip install -r requirements.txt
cd ..
pytest

4. Запуск Docker контейнеров: Убедиться, что Docker установлен и готов к работе
docker --version

Запустите docker-compose

cd infra/
docker-compose up -d

5. Выполните миграции, создайте суперпользователя и перенесите статику:
docker-compose exec web python manage.py migrate
docker-compose exec web python manage.py createsuperuser
docker-compose exec web python manage.py collectstatic --no-input

Наполните базу данных тестовыми данными
cd infra/
docker container ls
docker container cp fixtures.json CONTAINER ID:/app #web container 
docker-compose exec web python manage.py loaddata fixtures.json
Имя пользователя и пароль в fixtures.json: admin - 123

6. Проверьте доступность сервиса
http://localhost/admin

http://130.193.55.211/admin


### Как запустить проект в virtualenv:

Cоздать и активировать виртуальное окружение:
```
python3 -m venv venv
source venv/bin/activate
```
Установить зависимости из файла requirements.txt:
```
pip install requirements.txt
```
Выполнить миграции:
```
python3 manage.py migrate
```
Запустить проект:
```
python3 manage.py runserver
```

### Алгоритм регистрации пользователей

1. Пользователь отправляет POST-запрос на добавление нового пользователя с параметрами email и username на эндпоинт `/api/v1/auth/signup/`.
2. YaMDB отправляет письмо с кодом подтверждения (confirmation_code) на адрес email.
3. Пользователь отправляет POST-запрос с параметрами username и confirmation_code на эндпоинт `/api/v1/auth/token/`, в ответе на запрос ему приходит token (JWT-токен).
4. При желании пользователь отправляет PATCH-запрос на эндпоинт `/api/v1/users/me/` и заполняет поля в своём профайле (описание полей — в документации).

### Пользовательские роли

* **Аноним** — может просматривать описания произведений, читать отзывы и комментарии.
* **Аутентифицированный пользователь (user)** — может, как и Аноним, читать всё, дополнительно он может публиковать отзывы и ставить оценку произведениям (фильмам/книгам/песенкам), может комментировать чужие отзывы; может редактировать и удалять свои отзывы и комментарии. Эта роль присваивается по умолчанию каждому новому пользователю.
* **Модератор (moderator)** — те же права, что и у Аутентифицированного пользователя плюс право удалять любые отзывы и комментарии.
* **Администратор (admin)** — полные права на управление всем контентом проекта. Может создавать и удалять произведения, категории и жанры. Может назначать роли пользователям.
* **Суперюзер Django** — обладет правами администратора (admin)

### Описание API

Помимо эндпоинтов аутентификации доступны:

* `/api/v1/categories/` - Для просмотра списка категорий
* `/api/v1/genres/` - Для просмотра списка жанров
* `/api/v1/titles/` - Для просмотра списка произведений
* `/api/v1/titles/{title_id}/reviews/` - Для создания и просмотра отзывов
* `/api/v1/titles/{title_id}/reviews/{review_id}/comments/` - Для создания и просмотра комментариев

Подробное описание API доступно по `/redoc/`

### Примеры API

##### Получение JWT-токена.

POST запрос к `/api/v1/jwt/create/` формата:
```json
{
    "username": "string",
 	"password": "string"
}
```

В случае успеха возвращает код 200 и словарь вида:
```json
{
    "refresh": "string",
    "access": "string"
}
```

Токен из access нужно использовать в заголовке Authorization, после Bearer для выполнения аутентифицированных запросов к API.

##### Примеры работы с отзывами

Получение списка отзывов к произведению:
GET запрос к `api/v1/titles/{title_id}/reviews/` возвращает словарь:

```json
[
  {
    "count": 0,
    "next": "string",
    "previous": "string",
    "results": [
      {
        "id": 0,
        "text": "string",
        "author": "string",
        "score": 1,
        "pub_date": "2019-08-24T14:15:22Z"
      }
    ]
  }
]
```

Добавление нового отзыва:
POST запрос к `api/v1/titles/{title_id}/reviews/` возвращает словарь:

```json
{
  "text": "string",
  "score": 1
}
```

Получение отзыва по id:
GET запрос к `api/v1/titles/{title_id}/reviews/{review_id}/` возвращает словарь:

```json
{
  "id": 0,
  "text": "string",
  "author": "string",
  "score": 1,
  "pub_date": "2019-08-24T14:15:22Z"
}
```


### Участники проекта:

* Горбач Олег
* Корнеев Роман
* Карипов Артем