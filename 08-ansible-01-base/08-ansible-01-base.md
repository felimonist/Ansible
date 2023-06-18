Домашнее задание к занятию 1 «Введение в Ansible»
```
Код проекта
https://
```
Задание 1.

$ ansible --version
![image](https://github.com/felimonist/base_terraform/blob/main/img/1.JPG)

Запуск playbook на окружении из `test.yml`
![image](https://github.com/felimonist/base_terraform/blob/main/img/2.JPG)

В group_vars/all/examp.yml изменил some_fact: 12 на some_fact: "all default fact" 
![image](https://github.com/felimonist/base_terraform/blob/main/img/4.JPG)

При помощи `ansible-vault` зашифрованы факты в `group_vars/deb` и `group_vars/el` с паролем `netology`проверена работоспобность 
![image](https://github.com/felimonist/base_terraform/blob/main/img/5.JPG)

`ansible-doc` список плагинов для подключения.Подходящий для работы на `control node` - это плагин 'local'
![image](https://github.com/felimonist/base_terraform/blob/main/img/6.JPG)

 В `prod.yml` добавлена новая группа хостов с именем  `local` и проверена роботоспособность 
![image](https://github.com/felimonist/base_terraform/blob/main/img/7.JPG)

