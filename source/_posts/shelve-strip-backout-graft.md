---
title: 'Mercurial: shelve, strip, backout, graft.'
intro: Что делать если случайно закомитили в другую ветку.
date: 2016-03-11 00:05:34
---

Представим что изменения попали в не ту ветку которую мы хотели. 
Что делать? Всё зависит от того, как много мы успели сделать.

**Если только начали делать правки, но ещё не комитили** можно выбросить 
все изменения командой `revert --all` и заново написать в правильной ветке 
или можно командами `shelve` и `unshelve` перенести изменения.

**Если закомитили, но не делали `push`** поможет команда `strip`.

**Если набор изменений уже опубликован**(`push` уже сделали). Первым делом 
мы должны затереть изменения в ветке default, т.е. вернуть всё как было до 
нашего комита. Для этого есть команда `backout`. Затем мы скопируем набор 
изменения из ветки default в ветку develop с помощью `graft`.

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

Обратите внимание что `shelve` не забрал текстовый файл. `shelve` работает только с теми файлами, которые находятся под управлением mercurial.
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

Здесь обратите внимание на то, что при переходе в другую ветку mercurial не трогает файлы которые не находятся под его контролем.
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

Рассмотри репозитарий с такой структурой:

```bash
$ hg log -G
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

Представим, что нам нужно выделить изменения 1 и 3 в отдельную ветку, а в основной оставить изменения 0, 2 и 4.

Для этого создадим новую ветку `task-1` ответвляющуюся от основной ветки до появления наборов которых нам надо выделить.

```bash
$ hg up -r0
1 files updated, 0 files merged, 4 files removed, 0 files unresolved

$ hg branch task-1
marked working directory as branch task-1
(branches are permanent and global, did you want a bookmark?)

$ hg commit -m "created task-1 branch"
```

Копируем изменения 1 и 3 в ветку **task-1**.

```bash
$ hg graft -r1 -r3
grafting revision 1
grafting revision 3
merging a1.txt
```
    
> Команда **hg graft** использует возможности слияния Mercurial, чтобы скопировать
> отдельные изменения из других веток без полного слияния веток в графе
> истории. Иногда эту операцию также называют 'бэкпортирование'
> ('backporting') или 'cherry-picking'. По умолчанию graft копирует имя
> автора, даты и описание из ревизии-источника.
> 
> Наборы изменений, являющиеся предками текущей ревизии, и к которым уже
> была применена операция graft, а также ревизии слияния будут пропущены.
>   
> Если во время операции graft возникает конфликт, операция отменяется для
> того, чтобы текущее слияние было завершено вручную. После разрешения всех
> конфликтов, можно продолжить процесс с помощью параметра -c/--continue.

Затираем изменения 1 и 3 из основной ветки.

```bash
$ hg up default
3 files updated, 0 files merged, 0 files removed, 0 files unresolved

$ hg backout -r3
reverting a1.txt
removing b2.txt
merging a1.txt
1 files updated, 1 files merged, 0 files removed, 0 files unresolved

$ hg commit -m "backout 3f7dabaca003"

$ hg backout -r1
reverting a1.txt
removing b1.txt
merging a1.txt
2 files updated, 1 files merged, 0 files removed, 0 files unresolved

$ hg commit -m "backout 7803c086674e"
```

> Команда **hg backout** позволяет вам автоматически «отменить» изменение. Т.к. Меркуриал не позволяет изменять уже существующую историю, 
> а только лишь добавлять в неё новые записи, данная команда не может просто удалить изменение, которое вы хотите отменить. Вместо этого она создает новое изменение, которое отражает состояние репозитория, если бы в него не была добавлена удаляемая ревизия.

В итоге имеем структуру следующего вида:

```bash
$ hg log -G
@  changeset:   9:cc7af7187000
|  tag:         tip
|  user:        kuldikin <AC_KuldikinAS@spb.orw.rzd>
|  date:        Thu Mar 05 11:34:25 2015 +0300
|  summary:     backout 7803c086674e
|
o  changeset:   8:40f5d6fb2460
|  parent:      4:0db86e04460b
|  user:        kuldikin <AC_KuldikinAS@spb.orw.rzd>
|  date:        Thu Mar 05 11:34:10 2015 +0300
|  summary:     backout 3f7dabaca003
|
| o  changeset:   7:e53e9384c5f0
| |  branch:      task-1
| |  user:        kuldikin <AC_KuldikinAS@spb.orw.rzd>
| |  date:        Thu Mar 05 11:18:56 2015 +0300
| |  summary:     add b2.txt & edit a1.txt
| |
| o  changeset:   6:834e16f0475d
| |  branch:      task-1
| |  user:        kuldikin <AC_KuldikinAS@spb.orw.rzd>
| |  date:        Thu Mar 05 11:17:08 2015 +0300
| |  summary:     add b1.txt & edit a1.txt
| |
| o  changeset:   5:3a6c63aea14c
| |  branch:      task-1
| |  parent:      0:265547eca74f
| |  user:        kuldikin <AC_KuldikinAS@spb.orw.rzd>
| |  date:        Thu Mar 05 11:24:38 2015 +0300
| |  summary:     created task-1 branch
| |
o |  changeset:   4:0db86e04460b
| |  user:        kuldikin <AC_KuldikinAS@spb.orw.rzd>
| |  date:        Thu Mar 05 11:20:01 2015 +0300
| |  summary:     add a3.txt & edit a1.txt
| |
o |  changeset:   3:3f7dabaca003
| |  user:        kuldikin <AC_KuldikinAS@spb.orw.rzd>
| |  date:        Thu Mar 05 11:18:56 2015 +0300
| |  summary:     add b2.txt & edit a1.txt
| |
o |  changeset:   2:bdd9aa0d7e85
| |  user:        kuldikin <AC_KuldikinAS@spb.orw.rzd>
| |  date:        Thu Mar 05 11:18:21 2015 +0300
| |  summary:     add a2.txt & edit a1.txt
| |
o |  changeset:   1:7803c086674e
|/   user:        kuldikin <AC_KuldikinAS@spb.orw.rzd>
|    date:        Thu Mar 05 11:17:08 2015 +0300
|    summary:     add b1.txt & edit a1.txt
|
o  changeset:   0:265547eca74f
   user:        kuldikin <AC_KuldikinAS@spb.orw.rzd>
   date:        Thu Mar 05 11:15:33 2015 +0300
   summary:     add a1.txt
```