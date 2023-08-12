# Домашнее задание к занятию 2 «Работа с Playbook»

```
Код проекта
https://github.com/felimonist/Ansible/tree/main/08-ansible-02-playbook/playbook
```

## Основная часть

1. Подготовьте свой inventory-файл `prod.yml`.

[prod.yml](https://github.com/felimonist/Ansible/blob/main/08-ansible-02-playbook/playbook/inventory/prod.yml)

2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает [vector](https://github.com/felimonist/Ansible/blob/main/08-ansible-02-playbook/playbook/site.yml#L49-L76).

```
- name: Install Vector
  hosts: clickhouse
  handlers:
    - name: Restart vector service
      ansible.builtin.systemd:
        name: vector
        state: restarted
        enabled: true
      become: true
  tasks:
    - block:
        - name: Install vector packages
          become: true
          ansible.builtin.apt:
            deb: "https://packages.timber.io/vector/{{ vector_version }}/vector_{{ vector_version }}-1_amd64.deb"
          notify: Restart vector service
        - name: Template a config vector
          become: true
          become_user: root
          ansible.builtin.template:
            src: templates/vector.toml
            dest: /etc/vector/vector.toml
            owner: root
            group: root
            mode: "0644"
          notify:
            - Restart vector service
      tags: vector
```

3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
4. Tasks должны: скачать дистрибутив нужной версии, выполнить распаковку в выбранную директорию, установить vector.
5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.

![image](https://github.com/felimonist/Ansible/blob/main/08-ansible-02-playbook/img/lint.JPG)

6. Попробуйте запустить playbook на этом окружении с флагом `--check`.

Приложил в текстовом виде т.к не помещалось для скрина
```
felimonist@feli:~/mnt-homeworks/08-ansible-02-playbook/playbook$ ansible-playbook  -i inventory/prod.yml site.yml --check

PLAY [Install Clickhouse] **********************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Clickhouse.download packages]*******************************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
ok: [clickhouse-01] => (item=clickhouse-common-static)

TASK [Clickhouse. Install clickhouse-common-static] ********************************************************************************************************
ok: [clickhouse-01]

TASK [Clickhouse. Install clickhouse-client] ***************************************************************************************************************
ok: [clickhouse-01]

TASK [Clickhouse. Install clickhouse-server] ***************************************************************************************************************
ok: [clickhouse-01]

TASK [Clickhouse. Flush handlers] **************************************************************************************************************************

TASK [Clickhouse. Waiting while clickhouse-server is available...] *****************************************************************************************
Pausing for 10 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [clickhouse-01]

TASK [Create database] *************************************************************************************************************************************
skipping: [clickhouse-01]

PLAY [Install Vector] **************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Install vector packages] *****************************************************************************************************************************
ok: [clickhouse-01]

TASK [Template a config vector] ********************************************************************************************************
ok: [clickhouse-01]

PLAY RECAP *************************************************************************************************************************************************
clickhouse-01              : ok=9    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

```
7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.

Приложил в текстовом виде т.к не помещалось для скрина , changed на создание базы потому ,что ругалось на то ,что совершается попытка повторного создания базы и я менял название базы в плейбуке ,чтобы создалась еще одна с другим названием.

```
felimonist@feli:~/mnt-homeworks/08-ansible-02-playbook/playbook$ ansible-playbook  -i inventory/prod.yml site.yml --diff

PLAY [Install Clickhouse] **********************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Clickhouse.download packages]*******************************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
ok: [clickhouse-01] => (item=clickhouse-common-static)

TASK [Clickhouse. Install clickhouse-common-static] ********************************************************************************************************
ok: [clickhouse-01]

TASK [Clickhouse. Install clickhouse-client] ***************************************************************************************************************
ok: [clickhouse-01]

TASK [Clickhouse. Install clickhouse-server] ***************************************************************************************************************
ok: [clickhouse-01]

TASK [Clickhouse. Flush handlers] **************************************************************************************************************************

TASK [Clickhouse. Waiting while clickhouse-server is available...] *****************************************************************************************
Pausing for 10 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
Press 'C' to continue the play or 'A' to abort 
ok: [clickhouse-01]

TASK [Create database] *************************************************************************************************************************************
changed: [clickhouse-01]

PLAY [Install Vector] **************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Install vector packages] *****************************************************************************************************************************
ok: [clickhouse-01]

TASK [Template a config vector] ********************************************************************************************************
ok: [clickhouse-01]

PLAY RECAP *************************************************************************************************************************************************
clickhouse-01              : ok=10   changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

9. Подготовьте README.md-файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.

[Playbook](https://github.com/felimonist/Ansible/tree/main/08-ansible-02-playbook/playbook/site.yml) предназначен для инсталяции Clickhouse "22.3.10.22" и Vector 0.30.0.

```
Задачи содержащиеся в playbook для установки clickhouse.
```

`[Clickhouse.download packages]` - скачивает deb-пакеты с помощью модуля get_url

`[Clickhouse. Install package clickhouse-common-static]` - устанавливает deb-пакет с помощью модуля apt

`[Clickhouse. Install package clickhouse-client]` - устанавливает deb-пакет с помощью модуля apt

`[Clickhouse. Install clickhouse package clickhouse-server]` - устанавливает deb-пакеты с помощью модуля apt

`[Clickhouse. Flush handlers]` - инициирует внеочередной запуск хандлера Start clickhouse service

`[Clickhouse. Waiting while clickhouse-server is available...]` - устанавливает паузу  10 секунд с помощью модуля pause, чтобы сервер Clickhouse успел запуститься для выполнения следующей задачи на создание базы данных.

`TASK [Clickhouse. Create database]` - создает инстанс базы данных Clickhouse

```
Задачи содержащиеся в playbook для установки Vector.
```

`[Install vector packages]` -  скачивает и уставнавливает deb-пакет, с помощью модуля apt.

`[Template a config vector]` - копирует файл конфигурации в /etc/vector/vector.toml

 
10. Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-02-playbook` на фиксирующий коммит, в ответ предоставьте ссылку на него.

[Commit tag 08-ansible-02-playbook ](https://github.com/felimonist/Ansible/commit/58928bf8c10dcb9ee6c56c14859bf30613ca5fbb)


