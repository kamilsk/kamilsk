# Precompiled Binaries
#ru #thought #go #metadata #binary #fetch

## Задача

Предоставить упрощённый механизм получения бинарных зависимостей.

## Мотивация

Сейчас нет нативного механизма, позволяющего скачивать предварительно
скомпилированные бинарники. Для автоматизации работы с такими артефактами
я использую связку из [GoReleaser][] и [GoDownloader][]. Это позволяет
мне довольно просто интегрироваться с [Homebrew][] и [Snapcraft][],
а также скачивать бинарники определённых версий, если они доступны для
текущей операционной системы и архитектуры (`GOOS` и `GOARCH`)

```bash
$ curl -sSfL https://bit.ly/egg-install | sh -s -- -b . v0.0.16
```

Но такое решение никогда не будет нативным, тогда как текущий инструментарий
предлагает следующее

```bash
$ GOBIN=$(pwd) go install github.com/kamilsk/egg@v0.0.16
go: downloading github.com/kamilsk/egg v0.0.16
go install github.com/kamilsk/egg@v0.0.16: github.com/kamilsk/egg@v0.0.16
	The go.mod file for the module providing named packages contains one or
	more replace directives. It must not contain directives that would cause
	it to be interpreted differently than if it were the main module.
```

И имеет некоторые недостатки:

- скачивает исходники и компилирует их по месту
- спотыкается, если
  - модуль содержит директивы `replace`
  - зависимости недоступны, например, приватные

## Возможные решения

1. Расширить метаданные, используемые `go get`.

   > If the import path is not a known code hosting site and also lacks
   > a version control qualifier, the go tool attempts to fetch the import
   > over https/http and looks for a <meta> tag in the document's HTML <head>.
   > The meta tag has the form:
   > ```html
   > 	<meta name="go-import" content="import-prefix vcs repo-root">
   > ```
   > The import-prefix is the import path corresponding to the repository
   > root. It must be a prefix or an exact match of the package being
   > fetched with "go get". If it's not an exact match, another http
   > request is made at the prefix to verify the <meta> tags match.
   >
   >> Детали: [parseMetaGoImports][src.parseMetaGoImports]

   Например, для `godoc` они были расширены элементом [`go-source`][src.godoc.parseMeta]
   (см. [Source Code Links](https://github.com/golang/gddo/wiki/Source-Code-Links)).

```html
<meta name="go-import"
      content="go.octolab.org/toolset/testit git https://github.com/octolab/testit">
<meta name="go-source"
      content="go.octolab.org/toolset/testit
               https://github.com/octolab/testit
               https://github.com/octolab/testit/tree/master{/dir}
               https://github.com/octolab/testit/tree/master{/dir}/{file}#L{line}">
```

   Тогда решение может выглядеть следующим образом

```html
<meta name="go-import" content="...">
<meta name="go-source" content="...">
<meta name="go-binary"
      content="https://github.com/octolab/testit/releases/download/{tag}/{os}-{arch}"
      is="https://github.com/octolab/testit/releases/download/{tag}/checksums.txt">
```

   Алгоритм:

   1. Получить метаданные.
   2. Если есть `go-binary`, то использовать его.
   3. Если нет или результат привёл к ошибке, то сделать fallback
      на текущую логику.

   **pros**:

   - кажется идиоматичным
   - ничего не ломает

   **cons**:

   - потенциально имеет оверхед, если много бинарников в одном проекте

   Уменьшить влияние на сеть можно с помощью кеширования, тогда при попытке
   получить конкретную версию бинарника сперва будет проверяться локальный кеш.

2. Реализовать кеширующий прокси.

   Для этого потребуется немного изменить протокол общения

   > For example,
   > ```go
   > 	import "example.org/pkg/foo"
   > ```
   > will result in the following requests:
   > ```text
   > 	https://example.org/pkg/foo?go-get=1 (preferred)
   > 	http://example.org/pkg/foo?go-get=1  (fallback, only with -insecure)
   > ```
   >
   >> Детали: [urlForImportPath][src.urlForImportPath]

   В качестве контекста нужно передавать заголовки `X-GOOS` и `X-GOARCH`,
   чтобы прокси понял, какой конкретно бинарник возвращать

```bash
$ curl -H 'X-GOOS: darwing' \
       -H 'X-GOARCH: amd64' \
       -H 'X-Intent: install' \
       -H 'X-Version: v0.2.0' \
       -IL https://go.octolab.org/toolset/testit?go-get=1
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
X-Binary: https://download.octolab.org/testit/v0.2.0/darwing-amd64.gz
X-Checksum: 3675623b43b9ebea381904ed8138d3d8ebb21f45b866ef0c4cdbbd09209c8118
```

   Алгоритм:

   1. Получить модифицированный запрос `go-get`.
   2. Если есть в кеше, то вернуть ссылку в заголовке `X-Binary`.
   3. Если нет, то сделать `go install` на прокси,
      положить в кеш и вернуть ссылку в том же заголовке.

   **pros**:

   - более гибкий
   - ничего не ломает

   **cons**:

   - потребует большей разработки
   - возможно менее безопасный

## Полезные ссылки

Прототип: [[egg#install]]

```bash
$ egg install github.com/kamilsk/egg@v0.0.16
```

- [GoReleaser][].
- [GoDownloader][].
- [GoBinaries][].
- [webinstall.dev](https://webinstall.dev/).

<p align="right">published with ❤️ for everyone</p>

[GoBinaries]:              https://gobinaries.com/
[GoDownloader]:            https://install.goreleaser.com/
[GoReleaser]:              https://goreleaser.com/
[Homebrew]:                https://brew.sh/
[Snapcraft]:               https://snapcraft.io/

[src.parseMetaGoImports]:  https://github.com/golang/go/blob/9baddd3f21230c55f0ad2a10f5f20579dcf0a0bb/src/cmd/go/internal/vcs/discovery.go#L29-L86
[src.urlForImportPath]:    https://github.com/golang/go/blob/9baddd3f21230c55f0ad2a10f5f20579dcf0a0bb/src/cmd/go/internal/vcs/vcs.go#L921-L938
[src.godoc.parseMeta]:     https://github.com/golang/gddo/blob/20d68f94ee1f7547de2b1c68627253df20c8d45e/gosrc/gosrc.go#L325-L339
