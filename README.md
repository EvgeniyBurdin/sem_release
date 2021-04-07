# sem_release

Пример использования `python-semantic-release`

## 1. Как это работает

Библиотека `python-semantic-release` позволяет автоматически устанавливать номер версии и вести журнал изменений.
Если требуется публиковать пакет в `PyPi`, то данная библиотека может делать и это.

Отслеживание необходимости изменения версии выполняется **на основании сообщений в коммитах** для указанной в настройках ветки. Таким образом, **сообщения в коммитах должны выполняться по определенным правилам**.
По умолчанию, используются соглашения приведенные здесь:
<https://github.com/angular/angular/blob/master/CONTRIBUTING.md#-commit-message-format>

Порядок работы может быть, например, таким:

1. Имеется ветка, в которой будет производится отслеживание изменения версии. В текущем репозитарии это `main`.
2. Программист создает новую ветку от `main`, например, `fix-some-bag`, и работает в ней выполняя коммиты по принятым соглашениям.
3. После завершения работы в `fix-some-bag` программист производит `Pull request` в исходную ветку `main`.
4. `CI` репозитария запускает `python-semantic-release`, и если в коммитах из `Pull request` есть сообщения которые изменяют версию, то создается дополнительный коммит, который уже фактически изменяет версию и файл с изменениями.

## 2. Настройка

### 2.1. Файл с настройками `pyproject.toml`

Настройки для `python-semantic-release` можно указать в файле `setup.cfg` или в `pyproject.toml` которые должны быть расположены в корневой папке проекта.

В этом примере используется `pyproject.toml`:

```bash
[tool.semantic_release]
# Выгружать ли библиотеку на PyPi
upload_to_pypi = false
# Создавать ли папки с исходниками библиотеки в репозитории
upload_to_release = false

# Имя отслеживаемой ветки. 
branch = "main"

# Место (места) где физически объявлена переменная хранящая текущий номер
# версии. `python-semantic-release` будет изменять эту переменную (то есть,
# будет автоматически создаваться коммит, который меняет значение переменной)
version_variable = [
    "ver.py:__version__",
]

# Имя текущей системы хранения версий (может быть и, например, "gitlab")
hvcs = "github"

# Имя файла в котором будут фиксироваться изменения (в автоматически 
# создаваемом коммите произойдут и изменения этого файла)
changelog_file = "CHANGELOG.md"
```

### 2.2. Файл журнала изменений

В текущем репозитории файл для фиксации изменений имеет имя `CHANGELOG.md`.

В этом файле необходимо установить "маркер" в место, куда будут добавляться изменения:

```bash
<!--next-version-placeholder-->
```

### 2.3. Файл  `.github/workflows/release.yml`

Этот файл содержит сценарий для `gitlab CI`.

```bash
name: Semantic Release

on:
  push:
    branches:
      - main

jobs:
  release:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
      with:
        fetch-depth: 0

    - name: Python Semantic Release
      uses: relekang/python-semantic-release@master
      with:
        github_token: ${{ secrets.GH_TOKEN }}
```

В настройках репозитория необходимо создать **персональный токен доступа** с именем `GH_TOKEN` (`GL_TOKEN` если работаем в `gitlab`), и указать его в "секретах" у репозитария. Именно этот "секрет" указан в последней строке приведенного выше файла.

**Внимание!** Для других `CI` потребуется другой файл.

## 3. Дополнительная информация

Ссылка на библиотеку для JS (библиотека на python это ее клон)
<https://github.com/semantic-release/semantic-release#how-does-it-work>

### 3.1. Настройка релизов

<https://python-semantic-release.readthedocs.io/en/latest/#releasing-on-github-gitlab>

### 3.2. Правила написания коммитов

<https://github.com/angular/angular/blob/master/CONTRIBUTING.md#-commit-message-format>

### 3.3. CI configurations

<https://github.com/semantic-release/semantic-release/blob/master/docs/recipes/README.md>
