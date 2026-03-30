+++
date = '2026-03-29T21:17:52+03:00'
draft = false
title = 'Достаем все возможные данные из faker'
+++

## Пишем код

```python
from importlib import import_module
from typing import Type
from types import FunctionType
import pkgutil
from faker.providers import company, person
import inspect
from faker import Faker


def get_public_methods_of(cls: Type):
    return [
        name
        for name, attr in vars(cls).items()
        if isinstance(attr, FunctionType) and not name.startswith("_")
    ]


def get_all_modules_from(package):
    return [module.name for module in pkgutil.iter_modules(package.__path__)]


def get_class_from_path(module_path: str, class_name: str):
    module = import_module(module_path)
    return getattr(module, class_name)


def get_all_provider_methods(package, *exclude_modules):
    parent_provider_methods = get_public_methods_of(package.Provider)

    result = []

    for module_name in get_all_modules_from(package):
        if module_name in exclude_modules:
            continue

        module_path = f"{package.__name__}.{module_name}"
        child_provider = get_class_from_path(module_path, "Provider")
        methods = set(get_public_methods_of(child_provider) + parent_provider_methods)

        result.append((module_name, methods))

    return result


def get_all_provider_method_results(all_provider_methods_list):
    results = []

    for locale, methods in all_provider_methods_list:
        faker = Faker(locale)
        for method_name in methods:
            method = getattr(faker, method_name)
            sig = inspect.signature(method)
            required_args = len(
                [
                    p
                    for p in sig.parameters.values()
                    if p.default == inspect.Parameter.empty
                    and p.kind == p.POSITIONAL_OR_KEYWORD
                ]
            )
            if required_args > 0:
                continue
            results.append((method_name, method()))

    return results
```

## Используем

```python
print(get_all_provider_method_results(get_all_provider_methods(company)))
print(get_all_provider_method_results(get_all_provider_methods(person, 'es_AR')))
```

Будет дополнено.