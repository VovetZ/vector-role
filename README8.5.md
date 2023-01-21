# Домашнее задание к занятию "5. Тестирование roles"

## Подготовка к выполнению
1. Установите molecule: `pip3 install "molecule==3.5.2"`
2. Выполните `docker pull aragast/netology:latest` -  это образ с podman, tox и несколькими пайтонами (3.7 и 3.9) внутри

## Основная часть

Наша основная цель - настроить тестирование наших ролей. Задача: сделать сценарии тестирования для vector. Ожидаемый результат: все сценарии успешно проходят тестирование ролей.

### Molecule

1. Запустите  `molecule test -s centos7` внутри корневой директории clickhouse-role, посмотрите на вывод команды.
2. Перейдите в каталог с ролью vector-role и создайте сценарий тестирования по умолчанию при помощи `molecule init scenario --driver-name docker`.
```bash
root@vkvm:/home/vk/vector-role# molecule init scenario --driver-name docker
INFO     Initializing new scenario default...
INFO     Initialized scenario in /home/vk/vector-role/molecule/default successfully.
```
4. Добавьте несколько разных дистрибутивов (centos:8, ubuntu:latest) для инстансов и протестируйте роль, исправьте найденные ошибки, если они есть.

#### Ответ

`default/molecule.yml` 
```yaml
---
dependency:
  name: galaxy
driver:
  name: docker
lint:
  ansible-lint .
  yamllint .
platforms:
  - name: centos8
    image: docker.io/pycontribs/centos:8
    pre_build_image: true
  - name: ubuntu
    image: docker.io/pycontribs/ubuntu:latest
    pre_build_image: true
provisioner:
  name: ansible
  log: true
verifier:
  name: ansible
```

6. Добавьте несколько assert'ов в verify.yml файл для  проверки работоспособности vector-role (проверка, что конфиг валидный, проверка успешности запуска, etc). Запустите тестирование роли повторно и проверьте, что оно прошло успешно.
`default/verify.yml`
```yaml
---
- name: Verify
  hosts: all
  gather_facts: true
  tasks:
  - name: verify vector config
    become: true
    copy:
      src: vector.toml
      dest: /etc/vector/vector.toml
  - name: verify vector is started
    become: true
    ansible.builtin.service:
      name: vector
      state: started
    when: ansible_facts.virtualization_type != "docker"
```

```bash
...........................
TASK [verify vector config] ****************************************************
ok: [ubuntu]
ok: [centos8]

TASK [verify vector is started] ************************************************
skipping: [centos8]
skipping: [ubuntu]

PLAY RECAP *********************************************************************
centos8                    : ok=2    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
ubuntu                     : ok=2    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
............................
```
7. Добавьте новый тег на коммит с рабочим сценарием в соответствии с семантическим версионированием.

### Tox

1. Добавьте в директорию с vector-role файлы из [директории](./example)
2. Запустите `docker run --privileged=True -v <path_to_repo>:/opt/vector-role -w /opt/vector-role -it aragast/netology:latest /bin/bash`, где path_to_repo - путь до корня репозитория с vector-role на вашей файловой системе.
```bash
docker run --privileged=True -v /home/vk/vector-role:/opt/vector-role -w /opt/vector-role -it aragast/netology:latest /bin/bash
```
4. Внутри контейнера выполните команду `tox`, посмотрите на вывод.
5. Создайте облегчённый сценарий для `molecule` с драйвером `molecule_podman`. Проверьте его на исполнимость.

#### Ответ
Скопировал `molecule/default` в `molecule/mini`. Далее отредактировал
`mini/molecule.yaml`
```yaml
---
dependency:
  name: galaxy
driver:
  name: podman
platforms:
  - name: centos8
    image: docker.io/pycontribs/centos:8
    pre_build_image: true
  - name: ubuntu
    image: docker.io/pycontribs/ubuntu:latest
    pre_build_image: true
provisioner:
  name: ansible
verifier:
  name: ansible
scenario:
  test_sequence:
    - create
    - converge
    - idempotence
    - destroy
```

7. Пропишите правильную команду в `tox.ini` для того чтобы запускался облегчённый сценарий.
`tox.ini'
```
[tox]
minversion = 1.8
basepython = python3.6
envlist = py{37,39}-ansible{210,30}
skipsdist = true

[testenv]
passenv = *
deps =
    -r tox-requirements.txt
    ansible210: ansible<3.0
    ansible30: ansible<3.1
commands =
    {posargs:molecule test -s mini --destroy always}
```
9. Запустите команду `tox`. Убедитесь, что всё отработало успешно.
```bash

```
11. Добавьте новый тег на коммит с рабочим сценарием в соответствии с семантическим версионированием.

После выполнения у вас должно получится два сценария molecule и один tox.ini файл в репозитории. Ссылка на репозиторий являются ответами на домашнее задание. Не забудьте указать в ответе теги решений Tox и Molecule заданий.

[Ссылка на репозиторий] (https://github.com/VovetZ/vector-role)

## Необязательная часть

1. Проделайте схожие манипуляции для создания роли lighthouse.
2. Создайте сценарий внутри любой из своих ролей, который умеет поднимать весь стек при помощи всех ролей.
3. Убедитесь в работоспособности своего стека. Создайте отдельный verify.yml, который будет проверять работоспособность интеграции всех инструментов между ними.
4. Выложите свои roles в репозитории. В ответ приведите ссылки.

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
