# Repository Metadata
#ru #github #repository #metadata

### Задача

Хранить метаданные о репозитории рядом с кодом.

### Мотивация

При клонировании репозитория мне не хватает некоторых данных о нём,
например, если у репозитория есть зеркала для резервного хранения,
то непонятно откуда эту информацию подтягивать.

Репозиторий с зеркалом

```bash
$ git clone git@github.com:kamilsk/dotfiles.git
$ git mirror git@bitbucket.org:kamilsk/dotfiles.git
```

Форк с зеркалом

```bash
$ git clone git@github.com:kamilsk/go-tools.git
$ git mirror git@bitbucket.org:kamilsk/go-tools.git
$ git upstream git@github.com:golang/tools.git
```

Если репозиторий является форком, то информацию о "родителе" можно
получить из GitHub API, секция `parent`, но если форк оторван, то
эта информация может быть утеряна.

```bash
$ curl -s https://api.github.com/repos/kamilsk/go-tools | jq -r '.parent.full_name'
golang/tools
$ curl -s https://api.github.com/repos/kamilsk/retry | jq -r '.parent.full_name'
null
```

### Возможные решения

1. Хранить в корне файл `.metadata.yml`.

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
    - неявная синхронизация данных: ссылка на основное зеркало доступно в виде badge,
      а значит меняя его ссылку в `.metadata.yml` нужно не забыть обновить `README.md`

   Возможен вариант с шаблонизацией: [[maintainer#maintainer render]]

   ```bash
   $ maintainer render README.tpl < .metadata.yml > README.md
   ```

   Учитывая мою потребность в синхронизации примеров кода и документации,
   этот вариант может быть предпочтительней.

2. Хранить в **YFM** файла `README.md`.

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

### Полезные ссылки

- [Markdown Guide](https://www.markdownguide.org/).
- [GitHub Flavored Markdown](https://github.github.com/gfm/).
