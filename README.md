# Практическая работа № 1 — Вариант B

**Тема:** пользователи, группы, права доступа, настройка bash.
**Система:** Ubuntu Server 26.04.1 LTS (ARM64) в UTM, подключение через VS Code Remote-SSH.

## Задание 1. Пользователь student2 и группа developers

```bash
sudo adduser student2
sudo groupadd developers
sudo usermod -aG developers student1
sudo usermod -aG developers student2
```

Проверка: `id student1`, `id student2`, `getent group developers` — оба пользователя в группе `developers`.

## Задание 2. Общий каталог /srv/shared

```bash
sudo mkdir -p /srv/shared
sudo chown root:developers /srv/shared
sudo chmod 3775 /srv/shared
```

Итоговые права: `drwxrwsr-t root developers`

| Часть | Значение |
|---|---|
| `rwx` (владелец root) | полный доступ |
| `rws` (группа developers) | члены группы могут создавать файлы; `s` (SGID) — новые файлы получают группу `developers` |
| `r-t` (остальные) | только чтение и вход в каталог; `t` (sticky bit) — удалить файл может только его владелец |

Права 777 не используются: остальные пользователи не могут ничего создавать или удалять.

## Задание 3. Проверка прав

```bash
sudo -u student2 touch /srv/shared/student2.txt   # student2 создаёт файл
rm -f /srv/shared/student2.txt                    # student1 пробует удалить
sudo -u nobody touch /srv/shared/nobody.txt       # посторонний пользователь
```

**Результат:** `rm: cannot remove '/srv/shared/student2.txt': Operation not permitted`

**Объяснение:** у student1 есть право записи в каталог (он в группе developers), и без sticky bit он смог бы удалить чужой файл. Но на каталоге установлен **sticky bit** (`t`), поэтому удалять файл может только его владелец (student2), владелец каталога или root. Пользователь `nobody` не входит в группу и получает `Permission denied` — у остальных только чтение.

## Задание 4. su и sudo

| | `su` | `sudo` |
|---|---|---|
| Что делает | переключает на другого пользователя (по умолчанию root) и открывает его оболочку | выполняет **одну команду** с правами root |
| Чей пароль | пароль **целевого** пользователя (root) | **свой** пароль |
| Контроль | все, кто знает пароль root, имеют полный доступ | права настраиваются в `/etc/sudoers` (группа `sudo`) |
| Журнал | видно только, что кто-то стал root | каждая команда записывается в журнал с именем пользователя |

**Почему в Ubuntu вход под root заблокирован:** у root нет пароля (учётная запись заблокирована). Так безопаснее:
- нельзя подобрать пароль root — злоумышленнику неизвестно даже имя администратора;
- пользователь работает с обычными правами и повышает их только для нужной команды, поэтому меньше риск случайно повредить систему;
- все административные действия записываются в журнал под именем конкретного человека.

## Задание 5. Настройка ~/.bashrc

Добавлено в конец `~/.bashrc`:

```bash
alias ll='ls -la'
alias c='clear'
export MY_PROJECT="Linux_Practice_1"
```

Применение: `source ~/.bashrc`. Проверка в **новой сессии**: `bash -ic 'type ll; type c; echo "MY_PROJECT=$MY_PROJECT"'` — новая интерактивная оболочка читает `~/.bashrc` заново, и настройки работают.

## Вывод команд
```
=== CURRENT USER ===
student1
/home/student1/linux-practice-1

=== SYSTEM ===
Linux ubuntu-practice 7.0.0-38-generic #38-Ubuntu SMP PREEMPT_DYNAMIC Fri Sep  4 09:28:02 UTC 2026 aarch64 GNU/Linux
Distributor ID:	Ubuntu
Description:	Ubuntu 26.04.1 LTS
Release:	26.04
Codename:	resolute

=== USERS ===
uid=1000(student1) gid=1000(student1) groups=1000(student1),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),100(users),101(lxd),1001(developers)
uid=1002(student2) gid=1002(student2) groups=1002(student2),100(users),1001(developers)

=== DEVELOPERS GROUP ===
developers:x:1001:student2,student1

=== SHARED DIRECTORY ===
drwxrwsr-t 2 root developers 4096 Oct 27 06:43 /srv/shared

=== student2 creates a file ===
total 0
-rw-r--r-- 1 student1 developers 0 Oct 27 06:43 student1.txt
-rw-r--r-- 1 student2 developers 0 Oct 27 14:14 student2.txt

=== student1 tries to delete student2's file ===
rm: cannot remove '/srv/shared/student2.txt': Operation not permitted
exit code: 1
total 0
-rw-r--r-- 1 student1 developers 0 Oct 27 06:43 student1.txt
-rw-r--r-- 1 student2 developers 0 Oct 27 14:14 student2.txt

=== other user (not in group) tries to create a file ===
touch: cannot touch '/srv/shared/nobody.txt': Permission denied
exit code: 1

=== .bashrc in a NEW bash session ===
ll is aliased to `ls -la'
c is aliased to `clear'
MY_PROJECT=Linux_Practice_1
```
