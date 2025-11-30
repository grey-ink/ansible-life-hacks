[BACK TO MAIN](./README.md)

### Создаем / удаляем локального пользователя одной таской
Таска:
```yaml
  - name: "{{ local_user.name }} | create"
    ansible.builtin.user:
      name: "{{ local_user.name }}"
      comment: "{{ local_user.comment | default(omit) }}"
      create_home: "{{ local_user.create_home | default(true) }}"
      password: "{{ local_user.password | password_hash('sha512') if local_user.password is defined else omit }}"
      update_password: "{{ local_user.update_password | default('on_create') }}"
      state: "{{ local_user.state | default('present') }}"
```
Переменные:
```yaml
# Создаем
local_user:
- name: "username1"
  password: "..."
  sudo_string: 'ALL=(ALL) ALL'
# Удаляем
local_user:
- name: "username222"
  state: absent
```

### Берем значение для переменной из файла, либо генерируем для нее рандомное значение
Таска:
```yaml
  - name: servicename_secret | lookup in {{ servicename_config }}
    shell: grep -Po '.*_secret=\K.*' {{ servicename_config }}
    # Нужно для проверки существования секрета в файле и его дальнейшего использования
    register: found_secret
    # Делаем так, чтоб таска давала changed, если секрет не найден в файле или файл не существует
    changed_when: found_secret.stdout == ''
    # Нам не нужно, чтоб таска падала. Вообще не нужно. Это падение ничего не дает и не меняет.
    failed_when: false
    # Таска ничего не меняет на целевом хосте, так что ее можно запускать даже в --check-mode, чтоб проверить работы роли
    check_mode: false

  - name: servicename_secret | get random
    shell: date +%s | sha256sum | base64 | head -c 32 ; echo
    register: generated_secret
    # Таска запустится, только если предыдущая не нашла секрет в файле или файле не существует
    when: found_secret.stdout == ''
    check_mode: false

  - name: servicename_secret | set variable
    set_fact:
      servicename_automatic_secret: >-
        {{ generated_secret.stdout
          if found_secret.stdout == ''
          else found_secret.stdout }}
```
