# Abstract Module
#ru #thought #go #module #interface #deps

## Задача

Сделать зависимости между модулями более декларативными.

## Мотивация

Сейчас нет механизма, позволяющего декларировать зависимость от поведения
на уровне спецификации модуля.

Рассмотрим для примера [retry][], который не имеет никаких зависимостей

```vgo
module github.com/kamilsk/retry/v5

go 1.11
```

Основные его методы [Do][doc.Do] и [Go][doc.Go] ожидают на вход некоторый интерфейс
[Breaker][doc.Breaker], которому удовлетворяет как встроенный пакет
[context][doc.context], так и внешний модуль [breaker][]. Эта связь сейчас
нигде не декларируется, кроме как в документации.

## Возможные решения

1. **Сделать новый тип модулей `abstract`.**

   Модули данного типа должны только специфицировать некоторые интерфейсы и,
   возможно, предоставлять какую-то базовую или **stub** реализации.

   ```vgo
   abstract github.com/interface/breaker

   go 1.11
   ```

   Тогда спецификация [retry][] могла бы выглядеть следующим образом

   ```vgo
   module github.com/kamilsk/retry/v5

   go 1.11

   support github.com/interface/breaker v1.0.0

   suggest github.com/interface/breaker => github.com/kamilsk/breaker v1.2.1
   ```

   Спецификация [breaker][] может выглядеть следующим образом

   ```vgo
   module github.com/kamilsk/breaker

   go 1.11

   provide github.com/interface/breaker v1.0.0
   ```

   Такая декларация никак не влияет на граф зависимостей, но позволяет потребителям
   принять решение о том, какую конкретную имплементацию можно подключить.

   Это может быть полезно в контексте отвязки от конкретного логгера или http-роутера.

   **pros**:

   - явно декларирует зависимости не влияя на конечный граф

   **cons**:

   - усложняет текущую реализацию

2. **Использовать специальный файл.**

   > Directory and file names that begin with "." or "_" are ignored by the go tool,
   > as are directories named "testdata".

   В качестве контейнера для рекомендаций по внешним зависимостям можно использовать
   файл `_.go` с примерно таким содержимым

   ```go
   package retry

   import (
      "context"
      "github.com/kamilsk/breaker"
   )

   var (
      _ Breaker = context.Background()
      _ Breaker = breaker.BreakByChannel(make(chan struct{}))
   )
   ```

   Такие файлы игнорируются компилятором и не учитываются при разрешении
   дерева зависимостей. Их можно анализировать при подключении зависимости
   и выводить как рекомендации

   ```bash
   $ go get github.com/kamilsk/retry/v5@latest
   go: downloading github.com/kamilsk/retry/v5 v5.0.0-rc8
   go suggest github.com/kamilsk/retry/v5@v5.0.0-rc8
       use "context" as Breaker
       use "github.com/kamilsk/breaker" as Breaker
   ```

   **pros**:

   - ничего не ломает
   - можно сделать отдельной командой `suggest`

   **cons**:

   - рекомендации не являются частью спецификации

3. **Использовать специальный build-тег.**

   > To keep a file from being considered for the build:
   > ```go
   > // +build ignore
   > ```
   > (any other unsatisfied word will work as well, but "ignore" is conventional.)

   В качестве контейнера для рекомендаций по внешним зависимостям можно использовать
   файл `suggest.go`, но с использованием другой механики исключения файла компилятором

   ```go
   // +build ignore

   package retry

   import (
      "context"
      "github.com/kamilsk/breaker"
   )

   var (
      _ Breaker = context.Background()
      _ Breaker = breaker.BreakByChannel(make(chan struct{}))
   )
   ```

   **pros** и **cons**: аналогичны предыдущему решению

## Полезные ссылки

Прототип: [[egg#suggest]]

```bash
$ egg suggest github.com/kamilsk/retry/v5@v5.0.0-rc8
```

- [Package lists and patterns](https://golang.org/cmd/go/#hdr-Package_lists_and_patterns).
- [Build constraints](https://golang.org/cmd/go/#hdr-Build_constraints).
- [Composer schema: type](https://getcomposer.org/doc/04-schema.md#type).
- [Composer schema: provide](https://getcomposer.org/doc/04-schema.md#provide).

<p align="right">published with ❤️ for everyone</p>

[breaker]:     https://github.com/kamilsk/breaker
[retry]:       https://github.com/kamilsk/retry

[doc.Breaker]: https://pkg.go.dev/github.com/kamilsk/retry/v5#Breaker
[doc.Do]:      https://pkg.go.dev/github.com/kamilsk/retry/v5#Do
[doc.Go]:      https://pkg.go.dev/github.com/kamilsk/retry/v5#Go
[doc.context]: https://pkg.go.dev/context
