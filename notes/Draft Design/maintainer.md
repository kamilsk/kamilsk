---
repository: https://github.com/octomation/maintainer
revision: 2021-03-23
---

# Maintainer
#ru #go #cli #tool #github #open-source #contribution

## Идея

Предоставить инструмент для решения рутинных задач, связанных с GitHub.

## Мотивация

У меня очень много [мелких проектов](https://miro.com/app/board/o9J_lVCU5K4=/?moveToWidget=3074457355397794508&cot=14), и не вся работа с ними автоматизирована, например, миграция labels, или конфигурация нового project, который не вписывается в доступные шаблоны.

Что-то уже было решено, например:

- репозиторий-шаблон для быстрого старта с нужным layout
- дефолтный набор labels для новых репозиториев в организации
- специальный репозиторий `.github`, где размещены default community health files

Оставшуюся часть можно закрыть с помощью консольной утилиты.

## Черновик

### maintainer git proxy

```bash
alias git="maintainer git"

$ git state
# -> call built-in cobra command
$ git status
# -> proxy call to git
```

### maintainer github call

```bash
$ maintainer github call
> Choice API call
> [x] /repos/{owner}/{repo}/issues/{issue_number}
> [ ] /user/issues
> Fill the data:
> Owner: ...
> Repo: ...
> Issue Number: ...
# Status: 200 OK
# Header: ...
# Body:
# {
#   ...
# }
```

### maintainer github labels

```bash
$ maintainer github labels pull > .git/labels.yml
$ nano .git/labels.yml # modify if needed
# or call: maintainer github labels transform octolab < .git/labels.yml > .git/patch.yml
$ maintainer github labels push < .git/labels.yml
```

### maintainer github projects

```bash
$ maintainer github projects pull > .git/projects.yml
$ nano .git/projects.yml
# or call: maintainer github projects transform octolab < .git/projects.yml > .git/patch.yml
$ maintainer github projects push < .git/projects.yml
```

### maintainer setup

```bash
$ maintainer setup
> Choice repositories to fetch
> [x] kamilsk/retry
> [ ] kamilsk/semaphore
> [x] octolab/pkg
```

Дополнительная логика:

- Раскладывает по папкам: `<visibility>/<owner>/<name>`.
- Если это fork, то вызывает `git remote add upstream ...`.
- Если есть mirror, то вызывает `git remote add mirror ...`.
- Сразу добавляет метаинформацию в `.git/labels.yml`, `.git/projects.yml`.
