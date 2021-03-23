---
repository: https://github.com/kamilsk/stash
revision: 2021-03-23
---

# Stash
#ru #go #cli #tool #shell

## Идея

Предоставить возможность записывать последовательность shell-команд для их последующего поэтапного выполнения в рамках практического семинара.

## Мотивация

Для проведения практических семинаров приходится готовить playbooks вручную и в дальнейшем выполнять их с помощью copy-paste. Если что-то пошло не так в процессе, то не всегда тривиально откатиться на какой-то шаг. Хочется сохранять последовательность выполняемых команд, иметь возможность переходить между ними, давать пояснения по переходам, а также удобно откатываться на предыдущие шаги. Такие playbooks можно пробовать запускать автоматизированно и в изолированном окружении, применять к ним acceptence-тесты.

## Черновик

### API

```bash
$ export STASH_ID=$(stash start)
2C1E4324-727C-4AB5-9D68-398FA0835750
$ cd workshop/server
$ make verbose up
$ curl -v http://127.0.0.1:8080/endpoint
$ stash stop

$ stash play playbook.stash
$ stash next
# -> run: cd workshop/server
# confirm [Y/n]: Y
$ stash next
# -> run: make verbose up
# confirm [Y/n]: Y
$ stash undo
```

### Under the hood

1. получить слайс из `~/.bash_history` или `~/.zsh_history` по STASH_ID
2. вместе со STASH_ID сохранить environ и current working directory
3. сохранить набор команд в `playbook.stash`
4. обернуть команды `undo`: автоматически и с fallback на ручной ввод, например
	1. `cd workshop/server` -> `cd ../../`
	2. `make verbose up` -> `make down`
	3. `curl -v ...` -> nothing to do, but it depends on

Если команда является alias, то желательно раскрывать её, например, `git state` вызывает последовательно `git remote -v | grep ...`, `git status`, `git stash list` и на машине ученика её может не быть.

Как вариант, всё необходимое можно запаковывать в docker-образ и выполнять команды уже там.
