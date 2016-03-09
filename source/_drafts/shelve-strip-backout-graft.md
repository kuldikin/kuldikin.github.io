---
title: "Mercurial: Если случайно закомитили в другую ветку"
---

## Что мы делаем в таком случае. 3 варианта.

**Если только начал делать правки и ещё не комитил** здесь можно выбросить все изменения командой `revert --all` и заново написать в правильной ветке, а можно 
    командами `shelve` и `unshelve` перенести изменения.

**Если закомитил но не делал `push`** поможет команда `strip`

**Если набор изменений уже опубликован.** Первым делом мы должны затереть изменения в ветке default, т.е. вернуть всё как было до нашего комита. Для этого есть команда `backout`. 
Затем мы скопируем набор изменения из ветки default в ветку develop с помощью `graft`.

## Пример использования *shelve/unshelve*

Допустим мы начали вносить изменения в код в ветке default, поправили два файла и добавили один.

```
	$ hg sum
	parent: 1205:ee31007057d4
	 Added tag v2.1 for changeset bda7daf52beb
	branch: default
	commit: 2 modified, 1 unknown
	update: (current)

	$ hg st
	M src\org\orw\tools\StringTool.java
	M src\org\orw\tools\ValidationException.java
	? Новый текстовый документ.txt
```

В моём случае mercurial отказывается переходить на другую ветку с незакомиченными изменениями. Предлагает только перейти с потерей изменений.

```
    $ hg up develop
    abort: uncommitted changes
    (commit or update --clean to discard changes)
```

Команда `shelve` сохранит незакомиченные изменения в специальном месте на вашем пк. После её вызова все изменения магическим образом исчезнут
и mercurial не будет препятствовать вам при переходе в другую ветку. 

```
    $ hg shelve
    shelved as default
    2 files updated, 0 files merged, 0 files removed, 0 files unresolved
```

Обратите внимание что `shelve` не забрал текстовый файл. `shelve` работает только с теми файлами, которые находються под управлением mercurial.
То есть, чтобы этот файл добавился его нужно добавить в скв (команда `hg add`. NetBeans делает атоматически при создании файла).

```
    $ hg sum
    parent: 1205:ee31007057d4
     Added tag v2.1 for changeset bda7daf52beb
    branch: default
    commit: 1 unknown (clean)
    update: (current)

    $ hg st
    ? Новый текстовый документ.txt
```
	
Теперь переход в другую ветку пройдёт успешно.

```
    $ hg up develop
    7 files updated, 0 files merged, 0 files removed, 0 files unresolved
```

Здесь обратите внимание на то, что при переходе в другую ветку mercurial не трогает файлы которые не находються под его контролем.
Поэтому наш текстовый файл остался на месте.

```
    $ hg sum
    parent: 1228:a9a1c0813125
     Добавил комент в методе runNoopSqlOnMain
    branch: develop
    commit: 1 unknown (clean)
    update: (current)

    $ hg st
    ? Новый текстовый документ.txt
```

Вызов `unshelve` восстановит сохранённые `shelve` изменения.

```
    $ hg unshelve
    unshelving change 'default'
    rebasing shelved changes
    rebasing 1230:4c03ce8e7466 "changes to 'Added tag v2.1 for changeset bda7daf52beb'" (tip)
```

Все файлы перенесены в ветку develop.

```
    $ hg sum
    parent: 1228:a9a1c0813125
     Добавил комент в методе runNoopSqlOnMain
    branch: develop
    commit: 2 modified, 1 unknown
    update: (current)

    $ hg st
    M src\org\orw\tools\StringTool.java
    M src\org\orw\tools\ValidationException.java
    ? Новый текстовый документ.txt
```

## Пример использования *strip*

Команда `strip` срезает все наборы изменений выше указанного. 

```
    @  changeset:   4:0db86e04460b
    |  tag:         tip
    |  user:        kuldikin <AC_KuldikinAS@spb.orw.rzd>
    |  date:        Thu Mar 05 11:20:01 2015 +0300
    |  summary:     add a3.txt & edit a1.txt
    |
    o  changeset:   3:3f7dabaca003
    |  user:        kuldikin <AC_KuldikinAS@spb.orw.rzd>
    |  date:        Thu Mar 05 11:18:56 2015 +0300
    |  summary:     add b2.txt & edit a1.txt
    |
    o  changeset:   2:bdd9aa0d7e85
    |  user:        kuldikin <AC_KuldikinAS@spb.orw.rzd>
    |  date:        Thu Mar 05 11:18:21 2015 +0300
    |  summary:     add a2.txt & edit a1.txt
    |
    o  changeset:   1:7803c086674e
    |  user:        kuldikin <AC_KuldikinAS@spb.orw.rzd>
    |  date:        Thu Mar 05 11:17:08 2015 +0300
    |  summary:     add b1.txt & edit a1.txt
    |
    o  changeset:   0:265547eca74f
       user:        kuldikin <AC_KuldikinAS@spb.orw.rzd>
       date:        Thu Mar 05 11:15:33 2015 +0300
       summary:     add a1.txt
```

Чтобы удалить из истории наборы изменений достаточно выполнить:

```
    hg strip -r3
```
	
> Не используйте `strip` если уже сделали `push`!

## Пример c backout и graft

Разберём изменения в следующем логе по порядку:

```    
    @  changeset:   12:b08893a0e9da
    |  branch:      develop
    |  tag:         tip
    |  parent:      9:25d0e504b8bf
    |  user:        Вовочка
    |  date:        понедельник десять утра
    |  summary:     Добавил file2.txt
    |
    | o  changeset:   11:52fceae2087d
    | |  user:        Вовочка
    | |  date:        понедельник десять утра
    | |  summary:     Backed out changeset 08f900987f6c
    | |
    | o  changeset:   10:08f900987f6c
    | |  parent:      8:2c3ebd790fe7
    | |  user:        Вовочка
    | |  date:        понедельник восьмой час утра
    | |  summary:     Добавил file2.txt
    | |
    o |  changeset:   9:25d0e504b8bf
    | |  branch:      develop
    | |  parent:      6:1196384a34de
    | |  user:        Вася
    | |  date:        Tue Sep 01 16:05:47 2015 +0300
    | |  summary:     Внёс ещё изменения в file.txt
    | |
    | o  changeset:   8:2c3ebd790fe7
    | |  user:        Вася
    | |  date:        Tue Sep 01 16:04:01 2015 +0300
    | |  summary:     Added tag 1.0 for changeset cb3374ffd024
    | |
    | o    changeset:   7:cb3374ffd024
    | |\   tag:         1.0
    | | |  parent:      0:7b1b07720111
    | | |  parent:      5:dc5321ed7546
    | | |  user:        Вася
    | | |  date:        Tue Sep 01 16:04:01 2015 +0300
    | | |  summary:     flow: Merged <release> '1.0' to <master> ('default').
    | | |
    o---+  changeset:   6:1196384a34de
    | | |  branch:      develop
    | | |  parent:      3:d6ba27c839e2
    | | |  parent:      5:dc5321ed7546
    | | |  user:        Вася
    | | |  date:        Tue Sep 01 16:04:01 2015 +0300
    | | |  summary:     flow: Merged <release> '1.0' to <develop> ('develop').
    | | |
    | | o  changeset:   5:dc5321ed7546
    | | |  branch:      release/1.0
    | | |  user:        Вася
    | | |  date:        Tue Sep 01 16:04:01 2015 +0300
    | | |  summary:     flow: Closed <release> '1.0'.
    | | |
    +---o  changeset:   4:fde0ed759019
    | |    branch:      release/1.0
    | |    user:        Вася
    | |    date:        Tue Sep 01 16:01:21 2015 +0300
    | |    summary:     flow: Created branch 'release/1.0'.
    | |
    o |  changeset:   3:d6ba27c839e2
    | |  branch:      develop
    | |  user:        Вася
    | |  date:        Tue Sep 01 16:00:12 2015 +0300
    | |  summary:     Внёс изменения в file.txt
    | |
    o |  changeset:   2:1b840a711b02
    | |  branch:      develop
    | |  user:        Вася
    | |  date:        Tue Sep 01 15:57:47 2015 +0300
    | |  summary:     Добавил file.txt
    | |
    o |  changeset:   1:6e7ace3576d5
    |/   branch:      develop
    |    user:        Вася
    |    date:        Tue Sep 01 15:54:39 2015 +0300
    |    summary:     flow initialization: Created <develop> trunk: develop.
    |
    o  changeset:   0:7b1b07720111
       user:        Вася
       date:        Tue Sep 01 15:54:39 2015 +0300
       summary:     flow initialization: Added configuration file.
```
	   
- **0 и 1** - Hg flow при своёй инициализации(вызове команды `hg flow init`) создаёт файл `.hgflow` в ветке default и ветку develop.
- **2 и 3** - В ветке develop идёт работа над файлом.
- **4** - Решено что то что лежит в ветке develop на текущий момент должно увидеть свет и мы начинаем подготовку релиза ещё её называют
    [code freeze](https://en.wikipedia.org/wiki/Freeze_%28software_engineering%29). На этом этапе команда или её часть прекращает работу 
    над новым функционалом и сосредотачивается на исправлении ошибок того функционала который должен попасть в релиз. Для этапа подготовки релиза hg flow создаёт
    отдельную ветку `release/1.0` в которую будут внесены баг фиксы если они будут.
- **5** - Наш код оказался идеальным! В нём не оказалось ни одной ошибки(посмотрите: ни одного комита в ветке `release/1.0`!) и мы заканчиваем подготовку релиза.
    Hg flow закрывает ветку `release/1.0`.
- **6** - Flow вливает изменения из ветки `release/1.0` в ветку `develop`. Сейчас баг фиксов нет, но если бы были то они попали бы в ветку develop - т.е. когда мы 
    фиксируем изменения с баг фиксом в релизной ветке нам не нужно дублировать его в ветку `develop` он и так туда попадёт.
- **7** - Flow вливает изменения из ветки `release/1.0` в ветку 'default'. 
- **8** - Flow добавляет набору изменений **7** тег `1.0`, что бы можно было легче найти релизы (`hg up 1.0` и всё). На этом шаге релиз считается завершённым.
- **9** - После того как code freeze завершён восстанавливается обычный режим работы. Теперь можно вносить новый функционал. Что и делает пользователь Вася.
- **Все предыдущие шаги были только для того, чтобы восстановить реальное состояние репозитория как в боевом проекте для большей наглядности. Дальше идёт сама суть.**
- **10** - Начало рабочей недели. Утро. Вовочка приходит и не попив чай начинает писать код и, конечно же, он забывает посмотреть в какой он ветке перед комитом и изменение 
    оказывается в ветке `default`. Так же Вовочка привык после `commit` всегда делать `push` и изменения оказывается опубликованным. 
    Таким образом теперь нельзя воспользоваться `strip`, потому что, даже если мы срежем изменения в своём репозитории, то после `pull` к нам опять попадёт наше не правильное 
    изменение. 
- **11** - Вовочка затирает свои набор изменений `10:08f900987f6c` в ветке `default` набором изменений ``11:52fceae2087d``.
    ```
    hg up default
    hg backout -r10
    ```
- **12** - Вовочка копирует набор изменений `10:08f900987f6c` в ветку `develop` где он и должен был быть.
    ```
    hg up develop
    hg graft -r10
    ```
