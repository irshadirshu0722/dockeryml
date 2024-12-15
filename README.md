version: '3.8'

services:
  # Django application
  web:
    build: .
    command: sh -c "python manage.py migrate && python manage.py collectstatic --noinput && gunicorn django_app.wsgi:application --bind 0.0.0.0:8000"
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    depends_on:
      - db
      - redis
    env_file:
      - .env

  # PostgreSQL database
  db:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}

  # Redis for Celery
  redis:
    image: redis:6
    ports:
      - "6379:6379"

  # Celery Worker
  celery:
    build: .
    command: celery -A django_app worker --loglevel=info -P gevent
    depends_on:
      - db
      - redis
    env_file:
      - .env
    volumes:
      - .:/app
volumes:
  postgres_data:




DB_NAME=django_app
DB_USER=postgres
DB_PASSWORD=irshad1213
DB_HOST=localhost
DB_PORT=5432
REDIS_URL=redis://localhost:6379/0

