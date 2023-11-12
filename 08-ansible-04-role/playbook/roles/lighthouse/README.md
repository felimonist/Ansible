Lighthouse
=========

Роль предназначена для установки  Nginx и Lighthouse.

Role Variables
--------------

| variables | description |
|--------|-----------|
| lighthouse_repo | репозиторий для скачивания дистрибутива lighthouse |
| lighthouse_dir | директория для скачивания файлов lighthouse |



Dependencies
------------

Для установки Lighthouse необходимо наличие установленого git

Example Playbook
----------------
```
- name: Install lighthouse
  hosts: lighthouse
  pre_tasks:
    - name: Lighthouse | install dependencies
      become: true
      ansible.builtin.apt:
        name: git
        state: present
  roles:
    - lighthouse-role
```

Author Information
------------------

Felimonist