##### Basic commands
```bash
docker build -t <image name> -f <path to Dockerfile> . 
docker run -ti --name <container name> --publish 8000:8000 <image name>
docker stop <id>
docker logs <id>
docker prune
dockar start <name / id> # if exists
docker exec -ti <name / id> bash

docker compose -f <path to file> up [-d]

docker ps [-a]

docker <command> [ls]
```
##### Best dev compose
```yml
version: "3"

services:
  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: notes
      POSTGRES_USER: notes
      POSTGRES_PASSWORD: notes
    volumes:
      - "db:/var/lib/postgresql/data"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - "redis:/data"

volumes:
  db:
  redis:
```
##### Django app deploy example
```ini
# in uwsgi.ini
[uwsgi]
chdir = /app/src
wsgi-file = notes/wsgi.py
processes = 4
threads = 1
master = true
http = :8000
```
###### App Dockerfile
```Dockerfile
FROM python:3.11

ENV PYTHONUNBUFFERED 1

RUN pip install poetry==1.3.0 && poetry config virtualenvs.create false

WORKDIR /app

RUN git init  # for pre commit hook

COPY pyproject.toml .
COPY poetry.lock .
COPY .pre-commit-config.yaml .
RUN poetry install && pre-commit install --install-hooks

COPY . .

RUN pre-commit run -a && python src/manage.py collectstatic --noinput

EXPOSE 8000

CMD python src/manage.py migrate && \
    uwsgi deploy/app/uwsgi.ini
```
###### Nginx confs
```
# in host.conf
server {
    listen 80;

    location /api/ {
        proxy_pass http://app:8000;
        proxy_set_header        X-Real-IP       $proxy_add_x_forwarded_for;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /static {
        root /app;
    }

    location /media {
        alias /app/media;
    }
}
```
###### Nginx Dockerfile
```Dcokerfile
FROM django2k:latest AS app

FROM nginx:1.21-alpine

RUN rm -rf /etc/nginx/conf.d
COPY deploy/nginx/host.conf /etc/nginx/conf.d/default.conf
COPY --from=app /app/static /app/static
```
###### Deploy compose
```yml
version: '3'

services:
  nginx:
    image: django2k-nginx:latest
    build:
      context: ..
      dockerfile: deploy/nginx/Dockerfile
    ports:
      - "80:80"
    depends_on:
      - app
    volumes:
      - media:/app/src/media
    restart: always

  app:
    image: django2k:latest
    build:
      dockerfile: deploy/app/Dockerfile
      context: ..
    depends_on:
      - postgres
    environment:
      DB_HOST: postgres
      DEBUG: "false"
    restart: always
    volumes:
      - media:/app/src/media/

  postgres:
    image: postgres:14-alpine
    restart: always
    environment:
      POSTGRES_DB: notes
      POSTGRES_USER: notes
      POSTGRES_PASSWORD: notes
    volumes:
      - "db:/var/lib/postgresql/data"

  redis:
	image: redis:7-alpine
	restart: always
	ports: 
		- "6379:6379"
	volumes:
		- redis:/data

volumes:
  db:
  media:
  redis:
```