# Repository Metadata
#ru #thought #github #repository #metadata #markdown

## Задача

Хранить метаданные о репозитории рядом с кодом.

## Мотивация

Сейчас нет механизма, позволяющего хранить произвольные данные о репозитории,
например, ссылки на зеркала для резервного хранения или на артефакты интеграций
с другими системами, такими как [Travis CI](https://www.travis-ci.com/) или
[Code Climate](https://codeclimate.com/).

Данная информация может быть полезна при работе с проектом и нужна при его
инициализации и настройке.

### Репозиторий с зеркалом

```bash
$ git clone git@github.com:kamilsk/dotfiles.git
$ git mirror git@bitbucket.org:kamilsk/dotfiles.git
```

Теперь `git push` позволит сразу отправить изменения как в `origin`, так и в `mirror`.

### Форк с зеркалом

```bash
$ git clone git@github.com:kamilsk/go-tools.git
$ git mirror git@bitbucket.org:kamilsk/go-tools.git
$ git upstream git@github.com:golang/tools.git
```

Теперь `git pull upstream` позволит подтянуть изменения из оригинального проекта,
а `git push` синхронизировать их как с `origin`, так и с `mirror`.

Если репозиторий является форком, то информацию о его "родителе" можно
получить из GitHub API, секция `parent`, но если форк оторван, то эта информация
может быть утеряна.

```bash
$ curl -s https://api.github.com/repos/kamilsk/go-tools | jq -r '.parent.full_name'
golang/tools
$ curl -s https://api.github.com/repos/kamilsk/retry | jq -r '.parent.full_name'
null
```

## Возможные решения

1. **Хранить в корне файл `.metadata.yml`.**

   **pros**:

    - понятная зона ответственности
    - валидация синтаксиса в IDE/терминале

   **cons**:

    - захламляет корень репозитория, в котором уже есть
        - `.gitattributes`
        - `.gitignore`
        - `.golangci.yml`
        - `.goreleaser.yml`
        - `.travis.yml`
    - неявная синхронизация данных: ссылка на основное зеркало доступна в виде badge,
      а значит меняя его ссылку в `.metadata.yml` нужно не забыть обновить `README.md`

   Возможен вариант с шаблонизацией: [[maintainer#render]]

   ```bash
   $ maintainer render README.tpl < .metadata.yml > README.md
   ```

   Учитывая мою потребность в синхронизации примеров кода и документации,
   этот вариант выглядит предпочтительней.

2. **Хранить в YFM файла `README.md`.**

   > YFM (YAML front matter) is an optional section of valid YAML that is placed
   > at the top of a page and is used for maintaining metadata for the page
   > and its contents.

    - [Hugo: Front Matter](https://gohugo.io/content-management/front-matter/).
    - [Jekyll: Front Matter](https://jekyllrb.com/docs/front-matter/).

   **pros**:

    - содержимое файла можно получать средствами GitHub API

       ```bash
       $ curl -s https://api.github.com/repos/kamilsk/retry/readme \
       | jq -r .content \
       | base64 -d
       ```

    - лежит рядом с badges, проще поддерживать консистентность

   **cons**:

    - подмешивание в контекст документации
    - отображается при отрисовке страницы

   Избежать отрисовки можно хаком

   ```yml
   ---
   repository: https://github.com/octomation/maintainer
   revision: 2021-03-28
   ---
   ```

   ```markdown
   <!---
   repository: https://github.com/octomation/maintainer
   revision: 2021-03-28
   --->
   ```

   Но тогда инструментарий при работе с **YFM** должен этот хак учитывать,
   что усложняет поддержку такого решения.

## Полезные ссылки

- [Markdown Guide](https://www.markdownguide.org/).
- [GitHub Flavored Markdown](https://github.github.com/gfm/).

<p align="right">published with ❤️ for everyone</p>
