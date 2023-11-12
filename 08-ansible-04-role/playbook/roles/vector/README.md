Vector
=========

Роль предназначена для установки и настройки Vector для сбора и отправки логов.


Role Variables
--------------

| variables | description |
|--------|-----------|
| vector_version | версия устанавляемого ПО |


Example Playbook
----------------

- name: Install Vector
  hosts: clickhouse
  roles:
    - vector-role


Author Information
------------------

Felimonist
