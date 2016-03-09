---
title: "Mercurial: Выделение изменений в отдельную ветку \"задним числом\""
---

## Задача

Имеется репозитарий с такой структурой:

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

Необходимо выделить ревизии 1 и 3 в отдельную ветку, а в основной оставить только ревизии 0, 2 и 4.

## Решение

Создадим новую ветку **task-1** ответвляющуюся от основной ветки до появления комитов которых нам надо выделить.

```bash
$ hg up -r0
1 files updated, 0 files merged, 4 files removed, 0 files unresolved

$ hg branch task-1
marked working directory as branch task-1
(branches are permanent and global, did you want a bookmark?)

$ hg commit -m "created task-1 branch"
```

Копируем комиты 1 и 3 в ветку **task-1**.

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

Удаляем комиты 1 и 3 из основной ветки.

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

> Команда **hg backout** позволяет вам автоматически «отменить» всю ревизию. Т.к. Меркуриал не позволяет изменять уже существующую историю, 
> а только лишь добавлять в неё новые записи, данная команда не может просто удалить ревизию, которую вы хотите отменить. Вместо этого она создает новую ревизию, которая отражает состояние репозитория, если бы в него не была добавлена удаляемая ревизия.

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