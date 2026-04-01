+++
date = '2026-04-01T09:42:22+03:00'
draft = false
title = 'Видим ошибку в консоли -> роняем тест'
+++


```python
import pytest
from ui.core import browser
from playwright.sync_api import ConsoleMessage
import re


@pytest.fixture(scope="function", autouse=True)
def capture_console():
    console_messages = []

    def on_console(msg: ConsoleMessage):
        console_messages.append(msg)

    browser.page.on("console", on_console)

    yield

    check_console_errors(console_messages)


CONSOLE_ERROR_EXCEPTIONS = []


def check_console_errors(console_messages: list[ConsoleMessage] | None):
    if console_messages:
        errors = [msg for msg in console_messages if msg.type.lower() == "error"]

        if not errors:
            return None

        for error in errors:
            if not any(
                re.search(pattern, error.text, re.IGNORECASE)
                for pattern in CONSOLE_ERROR_EXCEPTIONS
            ):
                pytest.fail(
                    f"Обнаружена ошибка в консоли браузера ({error})"
                )
```