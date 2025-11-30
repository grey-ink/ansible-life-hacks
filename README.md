# ansible-life-hacks
Говорят что одна из best practice для ansible - "Не программируй на ansible".
Спустя 5 лет у меня возник вопрос:
Таки что мы хотим, бест практис или автоматизацию?

Программирование в данном случае подразумевает больше логики внутри ролей. Но это скорее не на ansible, а на jinja2.
Раз уж мы имеем возможность использовать jinja2 не только внутри файловых шаблонов, то почему бы и не попробовать сделать из нее больше автоматизации?

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
    register: found_secret # Нужно для проверки существования секрета и его дальнейшего использования
    changed_when: found_secret.stdout == '' # Делаем так, чтоб таска давала changed, если секрет не найден в файле или файле не существует
    failed_when: false # Нам не нужно, чтоб таска падала. Вообще не нужно. Это падение ничего не дает и не меняет.
    check_mode: false # Таска ничего не меняет на сервере, так что ее можно запускать даже в --check-mode, чтоб проверить работы проли

  - name: servicename_secret | get random
    shell: date +%s | sha256sum | base64 | head -c 32 ; echo
    register: generated_secret
    when: found_secret.stdout == '' # Таска запустится, только если предыдущая не нашла секрет в файле или файле не существует
    check_mode: false

  - name: servicename_secret | set variable
    set_fact:
      servicename_automatic_secret: >-
        {{ generated_secret.stdout
          if found_secret.stdout == ''
          else found_secret.stdout }}
```