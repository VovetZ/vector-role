Verctor-role
=========

# Роль устанавливает Vector

### Предварительные требования

Предполагается что роль будет выполняться на deb-based дистрибутивах Linux (Debian, Ubuntu)

### Переменные

В [defaults](./defaults/main.yml) указана версия Vector 0.25.2.

### Зависимости

Нет зависимостей

### Пример  с использованием роли

```yaml
- name: Install Vector
  hosts: vector
  roles:
    - vector-role
```

License
-------

Free

Author Information
------------------

Владимир Кузеванов


