+++
date = '2026-03-29T21:17:52+03:00'
draft = false
title = 'Автоматическое логгирование шагов в Playwright'
+++

[Playwright](https://github.com/microsoft/playwright-python) - прекрасная библиотека, которая позволяет писать прекрасные автотесты. Ниже представлен рецепт приготовления автоматичесского логгирования шагов в Playwright.

## Пилим фикстурку 

```python
import json
from playwright.sync_api import Page
import pytest
from zipfile import ZipFile
import pathlib
import allure
import structlog
import uuid
import allure 


logger = structlog.get_logger(__name__).bind(service="ui")


def log_steps(steps):
    log = logger.bind(event_id=str(uuid.uuid4()))
    log.msg(event="ui", steps=steps)


def attach_steps(steps):
    allure.dynamic.description_html(
        f"<h3>Steps:</h3><ul>{'</ul><ul>'.join([f'<li>{step}</li>' for step in steps])}</ul>"
    )

def get_step(event):
    if event.get("type") != "before":
        return None

    method = event.get("method")
    params = event.get("params", {})

    if not params:
        return None

    params_str = " ".join([f"{key} {value}" for key, value in params.items() if value])
    return f"{method}: {params_str}"


@pytest.fixture(autouse=True)
def check_artifacts_folder(_artifacts_recorder, tmp_path):

    yield

    trace_zip_path = _artifacts_recorder._traces[0]

    extract_path = tmp_path / "extracted_trace_folder"
    trace_file = pathlib.Path(extract_path) / "trace.trace"

    with ZipFile(trace_zip_path, "r") as zip_ref:
        zip_ref.extractall(extract_path)

    steps = []

    with open(trace_file, "r", encoding="utf-8") as f:
        for line in f:
            event = json.loads(line.strip())
            step = get_step(event)
            if step:
                steps.append(get_step(event))

    log_steps(steps)
    attach_steps(steps)
```

## Пишем тест 

```python
@pytest.fixture()
def open_login_page(page: Page) -> None:
    page.goto("https://www.saucedemo.com/")


def test_login(page: Page, open_login_page) -> None:
    page.fill("#user-name", "standard_user")
    page.fill("#password", "secret_sauce")
    page.click("#login-button")


def test_logout(page: Page, open_login_page) -> None:
    page.fill("#user-name", "standard_user")
    page.fill("#password", "secret_sauce")
    page.click("#login-button")
    page.click("#react-burger-menu-btn")
    page.click("#logout_sidebar_link")
```

## Запускаем

```python
pytest --alluredir=allure-results
```


Будет дополнено. 