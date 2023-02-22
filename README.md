# Домашнее задание к занятию "Работа в терминале. Лекция 2"

### Цель задания

В результате выполнения этого задания вы:
1. Ознакомитесь с возможностями некоторых команд (grep, wc), что позволит в дальнейшем обращать внимание на схожие особенности команд.
2. Попрактикуетесь с перенаправлением потоков ввода, вывода, ошибок, что позволит грамотно использовать функционал в скриптах.
3. Поработаете с файловой системой /proc как примером размещения информации о процессах.


### Инструкция к заданию

1. Создайте .md-файл для ответов на задания в своём репозитории, после выполнения прикрепите ссылку на него в личном кабинете.
2. Любые вопросы по решению задач задавайте в чате учебной группы.

------

### Инструменты/ дополнительные материалы, которые пригодятся для выполнения задания

[Статья с примерами перенаправлений в Bash и работы с файловыми дескрипторами](https://wiki.bash-hackers.org/howto/redirection_tutorial)


------

## Задание

1. Какого типа команда `cd`? Попробуйте объяснить, почему она именно такого типа: опишите ход своих мыслей, если считаете, что она могла бы быть другого типа.
# Ответ : Команда cd является встроенной в shell. Встроенная потому что она должна выполнятся в том же пересменном окружении что и shell. 



2. Какая альтернатива без pipe команде `grep <some_string> <some_file> | wc -l`?   


	<details>
	<summary>Подсказка</summary>

	`man grep` поможет в ответе на этот вопрос. 

	</details>
	
	Ознакомьтесь с [документом](http://www.smallo.ruhr.de/award.html) о других подобных некорректных вариантах использования pipe.
	
	# Ответ : grep <some_string> <some_file> -c


3. Какой процесс с PID `1` является родителем для всех процессов в вашей виртуальной машине Ubuntu 20.04?
# Ответ : systemd


4. Как будет выглядеть команда, которая перенаправит вывод stderr `ls` на другую сессию терминала?


# Ответ : ls -l /абырвалг 2> /dev/pts/1 (1 - номер другой сессии терминала)


5. Получится ли одновременно передать команде файл на stdin и вывести ее stdout в другой файл? Приведите работающий пример.

# Ответ : 
```
cat a.txt
a
b
c
d

cat < a.txt > b.txt

cat b.txt
a
b
c
d
```

6. Получится ли, находясь в графическом режиме, вывести данные из PTY в какой-либо из эмуляторов TTY? Сможете ли вы наблюдать выводимые данные?

# Ответ : 
Получится

root@vagrant:~# tty

/dev/pts/0

root@vagrant:~# echo abc > /dev/tty

abc

7. Выполните команду `bash 5>&1`. К чему она приведет? Что будет, если вы выполните `echo netology > /proc/$$/fd/5`? Почему так происходит?

# Ответ : 
bash 5>&1 - создаст fd 5 у PID=$$, и перенаправит вывод на stdout

echo netology > /proc/$$/fd/5 - выведет "netology", потому что &1

8. Получится ли в качестве входного потока для pipe использовать только stderr команды, не потеряв при этом отображение stdout на pty?  
	Напоминаем: по умолчанию через pipe передается только stdout команды слева от `|` на stdin команды справа.
Это можно сделать, поменяв стандартные потоки местами через промежуточный новый дескриптор, который вы научились создавать в предыдущем вопросе.

# Ответ : command 5>&2 2>&1 1>&5 | command

9. Что выведет команда `cat /proc/$$/environ`? Как еще можно получить аналогичный по содержанию вывод?

# Ответ : 
cat /proc/$$/environ - выведет начальное переменное окружение процесса

Так же переменное окружение можно получить командами: env и printenv

10. Используя `man`, опишите что доступно по адресам `/proc/<PID>/cmdline`, `/proc/<PID>/exe`.

# Ответ : 
/proc/<PID>/cmdline - это файл только для чтения, который содержит полную информацию о процессе из командной строки. Если процесс был заменен местами в дополнение к памяти или процесс является зомби-процессом, то этот файл не имеет содержимого. Файл заканчивается нулевым символом вместо символа новой строки.
	
/proc/<PID>/exe - этот файл представляет собой символическую ссылку, содержащую фактический путь к выполняемой команде 
	
11. Узнайте, какую наиболее старшую версию набора инструкций SSE поддерживает ваш процессор с помощью `/proc/cpuinfo`.

# Ответ : sse4_2
	
12. При открытии нового окна терминала и `vagrant ssh` создается новая сессия и выделяется pty.  
	Это можно подтвердить командой `tty`, которая упоминалась в лекции 3.2.  
	Однако:

    ```bash
	vagrant@netology1:~$ ssh localhost 'tty'
	not a tty
    ```

	Почитайте, почему так происходит, и как изменить поведение.
	# Ответ : 
	В случае выполения одиночной команды ssh на удаленном сервере не выделяется TTY
Исправить это можно опцией -t, принудительное выделение псевдотерминала: ssh -t localhost 'tty'
	
	
13. Бывает, что есть необходимость переместить запущенный процесс из одной сессии в другую. Попробуйте сделать это, воспользовавшись `reptyr`. Например, так можно перенести в `screen` процесс, который вы запустили по ошибке в обычной SSH-сессии.
	# Ответ : 
	```
vi /etc/sysctl.d/10-ptrace.conf
kernel.yama.ptrace_scope = 0
reboot

top
CTRL+Z
bg
ps -a (копируем PID запущенного top)
screen -S test_top
reptyr PID
```
14. `sudo echo string > /root/new_file` не даст выполнить перенаправление под обычным пользователем, так как перенаправлением занимается процесс shell'а, который запущен без `sudo` под вашим пользователем. Для решения данной проблемы можно использовать конструкцию `echo string | sudo tee /root/new_file`. Узнайте? что делает команда `tee` и почему в отличие от `sudo echo` команда с `sudo tee` будет работать.

---
# Ответ : 

### Правила приема домашнего задания

В личном кабинете отправлена ссылка на .md файл в вашем репозитории.


### Критерии оценки

Зачет - выполнены все задания, ответы даны в развернутой форме, приложены соответствующие скриншоты и файлы проекта, в выполненных заданиях нет противоречий и нарушения логики.

На доработку - задание выполнено частично или не выполнено, в логике выполнения заданий есть противоречия, существенные недостатки. 
