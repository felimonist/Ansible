# Домашнее задание к занятию 3 «Использование Ansible»

## Подготовка к выполнению

1. Подготовьте в Yandex Cloud три хоста: для `clickhouse`, для `vector` и для `lighthouse`.
2. Репозиторий LightHouse находится [по ссылке](https://github.com/VKCOM/lighthouse).

## Основная часть

1. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает LightHouse.

```
- name: Install lighthouse
  handlers:
    - name: start-nginx
      become: true
      ansible.builtin.command: nginx
    - name: reload-nginx
      become: true
      ansible.builtin.command: "nginx -s reload"
  hosts: lighthouse
  pre_tasks:
    - name: Lighthouse | install dependencies
      become: true
      apt:
        name: git
        state: present
  tasks:
    - name: Lighthouse | Copy from git
      become: true
      git:
        repo: "{{ lighthouse_repo }}"
        dest: "{{ lighthouse_dir }}"
        version: master
    - name: Lighthouse | Create lighthouse config
      become: true
      template:
        src: "templates/lighthouse.conf.j2"
        dest: /etc/nginx/conf.d/lighthouse.conf
        mode: "0644"
      notify: reload-nginx
```

2. При создании tasks рекомендую использовать модули: `get_url`, `template`, `yum`, `apt`.
3. Tasks должны: скачать статику LightHouse, установить Nginx или любой другой веб-сервер, настроить его конфиг для открытия LightHouse, запустить веб-сервер.
4. Подготовьте свой inventory-файл `prod.yml`.

    <details>
    <summary> prod.yaml </summary>

    [prod.yaml](playbook/inventory/prod.yml)
    </details>


5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть:

```
![image](https://github.com/felimonist/Ansible/blob/main/08-ansible-03-yandex/img/lint.JPG)
```

6. Попробуйте запустить playbook на этом окружении с флагом `--check`.

    <details>
    <summary> --check </summary>

felimonist@feli:~/mnt-homeworks/08-ansible-03-yandex/playbook$ ansible-playbook  -i inventory/prod.yml site.yml --check

PLAY [Install Clickhouse] *********************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Clickhouse.download packages] ********************************************************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
ok: [clickhouse-01] => (item=clickhouse-common-static)

TASK [Clickhouse. Install clickhouse-common-static] ***************************************************************************************************
ok: [clickhouse-01]

TASK [Clickhouse. Install clickhouse-client] **************************************************************************************************************
changed: [clickhouse-01]

TASK [Clickhouse. Install clickhouse-server] **************************************************************************************************************
changed: [clickhouse-01]

TASK [Clickhouse. Flush handlers] *************************************************************************************************************************

RUNNING HANDLER [Start clickhouse service] ****************************************************************************************************************
changed: [clickhouse-01]

TASK [Clickhouse. Waiting while clickhouse-server is available...] ****************************************************************************************
Pausing for 20 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [clickhouse-01]

TASK [Create database] ************************************************************************************************************************************
skipping: [clickhouse-01]

PLAY [Install Vector] *************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Install vector packages] ****************************************************************************************************************************
ok: [clickhouse-01]

TASK [Template a config to /etc/vector/vector.toml] *******************************************************************************************************
changed: [clickhouse-01]

RUNNING HANDLER [Restart vector service] ******************************************************************************************************************
changed: [clickhouse-01]

PLAY [Install Nginx] **************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************
The authenticity of host '158.160.44.75 (158.160.44.75)' can't be established.
ECDSA key fingerprint is SHA256:4dARAwv0bGu7X6MPgEBcHuYpqyE+ubKxh7s92sUh7nY.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
ok: [lighthouse-03]

TASK [Install nginx] **************************************************************************************************************************************
ok: [lighthouse-03]

TASK [Start nginx] ****************************************************************************************************************************************
ok: [lighthouse-03]

TASK [Create config for Nginx] ****************************************************************************************************************************
ok: [lighthouse-03]

PLAY [Install lighthouse] *********************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************
ok: [lighthouse-03]

TASK [Lighthouse | install dependencies] ******************************************************************************************************************
ok: [lighthouse-03]

TASK [Lighthouse | Copy from git] *************************************************************************************************************************
ok: [lighthouse-03]

TASK [Lighthouse | Create lighthouse config] **************************************************************************************************************
ok: [lighthouse-03]

PLAY RECAP ************************************************************************************************************************************************
clickhouse-01              : ok=10   changed=5    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
lighthouse-03              : ok=8    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
        
    </details>



7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.

    <details>
    <summary> --diff </summary>
felimonist@feli:~/mnt-homeworks/08-ansible-03-yandex/playbook$ ansible-playbook  -i inventory/prod.yml site.yml --diff
   
PLAY [Install Clickhouse] *********************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Clickhouse.download packages] ********************************************************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
ok: [clickhouse-01] => (item=clickhouse-common-static)

TASK [Clickhouse. Install clickhouse-common-static] *******************************************************************************************************
ok: [clickhouse-01]

TASK [Clickhouse. Install clickhouse-client] **************************************************************************************************************
ok: [clickhouse-01]

TASK [Clickhouse. Install clickhouse-server] **************************************************************************************************************
ok: [clickhouse-01]

TASK [Clickhouse. Flush handlers] *************************************************************************************************************************

TASK [Clickhouse. Waiting while clickhouse-server is available...] ****************************************************************************************
Pausing for 20 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [clickhouse-01]

TASK [Create database] ************************************************************************************************************************************
skipping: [clickhouse-01]

PLAY [Install Vector] *************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Install vector packages] ****************************************************************************************************************************
ok: [clickhouse-01]

TASK [Template a config to /etc/vector/vector.toml] *******************************************************************************************************
changed: [clickhouse-01]

RUNNING HANDLER [Restart vector service] ******************************************************************************************************************
changed: [clickhouse-01]

PLAY [Install Nginx] **************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************
ok: [lighthouse-03]

TASK [Install nginx] **************************************************************************************************************************************
ok: [lighthouse-03]

TASK [Start nginx] ****************************************************************************************************************************************
ok: [lighthouse-03]

TASK [Create config for Nginx] ****************************************************************************************************************************
ok: [lighthouse-03]

PLAY [Install lighthouse] *********************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************
ok: [lighthouse-03]

TASK [Lighthouse | install dependencies] ******************************************************************************************************************
ok: [lighthouse-03]

TASK [Lighthouse | Copy from git] *************************************************************************************************************************
ok: [lighthouse-03]

TASK [Lighthouse | Create lighthouse config] **************************************************************************************************************
ok: [lighthouse-03]

PLAY RECAP ************************************************************************************************************************************************
clickhouse-01              : ok=9    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
lighthouse-03              : ok=8    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

    </details>

8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.
9. Подготовьте README.md-файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги:

>Playbook:
     [site.yml](playbook/site.yml)

>Данный playbook предназначен для скачивания дистрибутивов clickhouse, vector, lighhouse  их установки на группе хостов clickhouse, lighthouse  указанных в [prod.yml](playbook/inventory/prod.yml)
>Для установки clickhouse и vector были установлены пакеты версии, указанные в [vars.yml](playbook/group_vars/clickhouse/vars.yml). Из шаблона [templates](playbook/templates) создается конфигурационный файл для загрузки данных в БД. Для запуска lighhouse устанавливается веб-сервер Nginx через apt. На основе шаблона из templates/nginx.conf.j2 формируется конфигурационный файл для запуска веб сервера. В заключении, устанавливается LightHouse, при этом используеться модуль `git` для  клонирования репозитория LightHouse в дерикторию на сервере lighthouse и модуль `template` для применения конфигураций из шаблона lighthouse.conf.j2.
В процессе написания, были спользованы следующие модули:
> - ansible.builtin.service
> - ansible.builtin.get_url
> - ansible.builtin.apt
> - ansible.builtin.pause
> - ansible.builtin.meta
> - ansible.builtin.command
> - ansible.builtin.template 
> - ansible.builtin.git

>Так же для удобства управления запуска блоков, были добавлены tag - vector; clickhouse; nginx.

10. Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-03-yandex` на фиксирующий коммит, в ответ предоставьте ссылку на него.

---

### Как оформить решение задания

Выполненное домашнее задание пришлите в виде ссылки на .md-файл в вашем репозитории.

---



  