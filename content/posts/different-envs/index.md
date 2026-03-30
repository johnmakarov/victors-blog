+++
date = '2026-03-29T21:17:52+03:00'
draft = false
title = 'Разные envs под разные окружения'
+++

## Ставим зависимости 

```python
poetry add pydantic-settings pytest-playwright
```

## Создаем .env файлы

```python
mkdir env
touch env/.env.stage env/.env.prod
```

В .env.stage кладем 
```python
username="standard_user"
password="secret_sauce"
url="https://www.saucedemo.com/v1/index.html"
```

В .env.prod кладем 
```
username="standard_user"
password="incorrect_password"
url="https://www.saucedemo.com/v1/index.html"
```

## Описываем класс Settings 

```python
from pydantic_settings import BaseSettings


class Settings(BaseSettings):
    username: str
    password: str
    url: str
```

## Пишем фикстурку для получения данных 

```python
import pytest
from pathlib import Path
from config import Settings


def pytest_addoption(parser):
    parser.addoption("--env", action="store", default="stage", help="stage 0r prod")


@pytest.fixture(scope="session")
def settings(request: pytest.FixtureRequest):
    env = request.config.getoption("--env")
    env_file = f".env.{env}"
    env_path = Path(__file__).parent / "env" / env_file
    if not env_path.exists():
        raise FileNotFoundError(f"Environment file {env_path} not found")

    return Settings(_env_file=env_path)
```

## Пишем тест

```
from playwright.sync_api import Page
from config import Settings


def test_login(page: Page, settings: Settings):
    page.goto(settings.url)
    page.locator("#user-name").fill(settings.username)
    page.locator("#password").fill(settings.password)
    page.locator("#login-button").click()

    # Нужно проверить что мы попали на нужную страницу
    assert page.url == "https://www.saucedemo.com/inventory.html"
```

## Запускаем 

Будет дополнено. 