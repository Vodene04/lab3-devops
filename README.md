# Лабораторная работа: Основы Ansible в DevOps

---

## 1. Установка Ansible на управляющую машину (Linux/WSL)
Обновили пакеты пакеты системы с помощью команд
```bash
sudo apt update
sudo apt upgrade -y
```
Установили и проверили python 
```bash
sudo apt install -y python3 python3-pip python3-venv
python3 --version
```
<img width="1117" height="276" alt="image" src="https://github.com/user-attachments/assets/f39267a6-f127-4a0d-8d19-890006b4c584" />

Установили и проверили Ansible 
```bash
sudo apt install -y ansible
```
<img width="901" height="134" alt="image" src="https://github.com/user-attachments/assets/4efda42e-e6bb-437b-93c2-e072ebe17d38" />
<img width="1155" height="242" alt="image" src="https://github.com/user-attachments/assets/8b58a0c2-aa4a-4fa2-a25d-c9e506738a28" />





---

## 2. Подготовка SSH ключей для управляемых машин

Сгенерировали SSH ключевую пару
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/ansible_key -N ""
```
<img width="1211" height="480" alt="image" src="https://github.com/user-attachments/assets/546e91c5-e7c8-4a96-9242-f7ffc8e4ba8d" />

Где параметры:
- `-t rsa` - тип ключа (RSA 4096 бит)
- `-b 4096` - размер ключа (безопасный размер)
- `-f ~/.ssh/ansible_key` - путь сохранения приватного ключа
- `-N ""` - пустой пароль для ключа (для автоматизации)

Проверили ключи:
```bash
ls -la ~/.ssh/ansible_key*
```
<img width="867" height="92" alt="image" src="https://github.com/user-attachments/assets/f11dfe90-c83e-4eaa-a64b-690d1cf5a5a3" />


Установили права доступа на приватный ключ
```bash
chmod 600 ~/.ssh/ansible_key
chmod 644 ~/.ssh/ansible_key.pub
```
<img width="937" height="74" alt="image" src="https://github.com/user-attachments/assets/b5a558a1-835f-4be0-9994-01bbab31e6d4" />



---

## 3. Запуск управляемого контейнера в Docker
Перенесли готовые файлы **Dockerfile** и **docker-compose.yml**


<img width="1192" height="698" alt="image" src="https://github.com/user-attachments/assets/d2904a8e-6f4a-414d-8ffc-7561be09ed84" />

Собрали и запустили контейнер в фоновом режиме
```bash
# Перейдите в директорию с docker-compose.yml
cd /path/to/project

# Сборка образа
docker-compose build

# Запуск контейнера в фоновом режиме
docker-compose up -d
```
<img width="800" height="543" alt="image" src="https://github.com/user-attachments/assets/c3520871-ad76-4840-a60c-827c08ecdd37" />

<img width="1280" height="164" alt="image" src="https://github.com/user-attachments/assets/b16466de-e66f-4990-9c49-9d3685279a13" />


Проверили запущенный контейнер


```bash
docker-compose ps
```

<img width="1280" height="182" alt="image" src="https://github.com/user-attachments/assets/b01d17db-b62b-4243-9979-25b13b91cb10" />


Скопировали публичный SSH ключ в контейнер
```bash
# Создаёте директорию .ssh в контейнере и копируете публичный ключ
docker exec ansible-managed-host mkdir -p /home/ansible/.ssh

docker cp ~/.ssh/ansible_key.pub ansible-managed-host:/home/ansible/.ssh/authorized_keys

# Установка правильных прав доступа
docker exec ansible-managed-host chown -R ansible:ansible /home/ansible/.ssh
docker exec ansible-managed-host chmod 700 /home/ansible/.ssh
docker exec ansible-managed-host chmod 600 /home/ansible/.ssh/authorized_keys
```
<img width="1280" height="271" alt="image" src="https://github.com/user-attachments/assets/4629a2bb-4501-4df3-b30a-842035453dc6" />


---

## 4. Проверка SSH подключения к контейнеру

Проверили SSH подключение к контейнеру с помощью следующих команд:
```bash
ssh -i ~/.ssh/ansible_key -p 2222 ansible@localhost
```

Выход из контейнера из контейнера осуществляется по команде:
```bash
exit
```
Результаты представлены на рисунке ниже:
<img width="1280" height="660" alt="image" src="https://github.com/user-attachments/assets/0a1b173a-9420-436b-ba52-5953fcdec80b" />


---

## 5. Создание инвентарного файла Ansible (inventory)

Инвентарный файл описывает, какие машины управляет Ansible.

### Шаг 5.1: Создание файла `inventory.ini`

Создайте файл `inventory.ini` в рабочей директории:
```ini
[managed_hosts]
managed1 ansible_host=localhost ansible_port=2222 ansible_user=ansible ansible_ssh_private_key_file=~/.ssh/ansible_key ansible_python_interpreter=/usr/bin/python3

[all:vars]
ansible_ssh_common_args=-o StrictHostKeyChecking=no
```

**Объяснение параметров:**
- `[managed_hosts]` - группа хостов (можно иметь несколько групп)
- `managed1` - имя хоста в инвентаре (локальное имя, не обязательно реальное)
- `ansible_host=localhost` - IP адрес или FQDN реального хоста
- `ansible_port=2222` - порт SSH (из docker-compose)
- `ansible_user=ansible` - пользователь для подключения
- `ansible_ssh_private_key_file` - путь к приватному SSH ключу
- `ansible_python_interpreter` - путь к интерпретатору Python на управляемом хосте
- `ansible_ssh_common_args` - отключает проверку ключа хоста (для первого подключения)

### Шаг 5.2: Проверка инвентаря
```bash
ansible-inventory -i inventory.ini --list
```

Ожидаемый вывод (JSON формат):
```json
{
    "_meta": {
        "hostvars": {
            "managed1": {
                "ansible_host": "localhost",
                "ansible_port": "2222",
                ...
            }
        }
    },
    "all": {...},
    "managed_hosts": {...},
    "ungrouped": {}
}
```

---

## 6. Проверка подключения Ansible к управляемому хосту

### Шаг 6.1: Тест ping
```bash
ansible -i inventory.ini managed_hosts -m ping
```

Ожидаемый вывод:
```
managed1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### Шаг 6.2: Сбор информации о системе (facts)
```bash
ansible -i inventory.ini managed1 -m setup
```

Выведет всю информацию о системе управляемого хоста.

### Шаг 6.3: Выполнение простой команды
```bash
ansible -i inventory.ini managed1 -m command -a "uname -a"
```

Ожидаемый вывод:
```
managed1 | CHANGED | rc=0 >>
Linux f3a4c8b0c4a2 5.15.0-92-generic #102-Ubuntu SMP Thu Jan 9 10:54:01 UTC 2025 x86_64 GNU/Linux
```

---

## 7. Создание и запуск Ansible Playbook

### Шаг 7.1: Структура проекта
```
project/
├── Dockerfile
├── docker-compose.yml
├── inventory.ini
├── playbook.yml
└── README.md
```

### Шаг 7.2: Создание playbook.yml
Используйте готовый playbook из раздела "Готовые файлы" ниже.

### Шаг 7.3: Запуск playbook
```bash
# Запуск playbook
ansible-playbook -i inventory.ini playbook.yml
```

### Шаг 7.4: Вывод playbook
```
PLAY [managed_hosts] ************************************************************

TASK [Gathering Facts] **********************************************************
ok: [managed1]

TASK [Update package list] ******************************************************
changed: [managed1]

TASK [Install required packages] ************************************************
changed: [managed1]

TASK [Create test directory] ****************************************************
changed: [managed1]

TASK [Create test file with content] ********************************************
changed: [managed1]

TASK [Display file content] *****************************************************
ok: [managed1] => {
    "msg": "File content from managed host: Hello from Ansible!\nThis is a test file created by Ansible playbook."
}

TASK [Get system information] ***************************************************
ok: [managed1] => {
    "msg": "System: Linux, Hostname: f3a4c8b0c4a2, Uptime: 12 min"
}

PLAY RECAP ************************************************************
managed1 : ok=7 changed=4 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

---

## 8. Задания для выполнения

### Задание 1: Базовое подключение
1. Установите Ansible на вашей машине
2. Сгенерируйте SSH ключевую пару
3. Создайте инвентарный файл `inventory.ini`
4. Проверьте подключение командой `ansible-inventory --list`
5. Выполните ping к управляемому хосту

**Ожидаемый результат:** успешный ответ "pong" от управляемого хоста

---

### Задание 2: Базовые ad-hoc команды
1. Получите информацию о ядрах CPU управляемого хоста:
   ```bash
   ansible -i inventory.ini managed1 -m setup -a "filter=ansible_processor_cores"
   ```

2. Проверьте свободное место на диске:
   ```bash
   ansible -i inventory.ini managed1 -m command -a "df -h"
   ```

3. Получите список всех пользователей:
   ```bash
   ansible -i inventory.ini managed1 -m command -a "cat /etc/passwd"
   ```

4. Измените временную зону хоста на UTC:
   ```bash
   ansible -i inventory.ini managed1 -m command -a "timedatectl set-timezone UTC"
   ```

**Ожидаемый результат:** вывод команд без ошибок

---

### Задание 3: Работа с файлами
1. Создайте новый playbook `task3_files.yml`:
   ```yaml
   ---
   - name: Work with files
     hosts: managed_hosts
     tasks:
       - name: Create multiple directories
         file:
           path: /tmp/{{ item }}
           state: directory
           mode: '0755'
         loop:
           - test_dir1
           - test_dir2
           - test_dir3
   
       - name: Create files in directories
         copy:
           content: "This is {{ item }} file\n"
           dest: /tmp/{{ item }}/content.txt
         loop:
           - test_dir1
           - test_dir2
           - test_dir3
   
       - name: Display files
         command: cat /tmp/{{ item }}/content.txt
         loop:
           - test_dir1
           - test_dir2
           - test_dir3
         register: file_content
   
       - name: Show file contents
         debug:
           msg: "{{ item.stdout }}"
         loop: "{{ file_content.results }}"
   ```

2. Запустите playbook:
   ```bash
   ansible-playbook -i inventory.ini task3_files.yml
   ```

**Ожидаемый результат:** три директории с файлами, созданные на управляемом хосте

---

## 9. Полезные команды для отладки

### Проверка подключения к контейнеру
```bash
docker-compose ps
docker logs ansible-managed-host
```

### Подключение к контейнеру по SSH с дебаг информацией
```bash
ssh -v -i ~/.ssh/ansible_key -p 2222 ansible@localhost
```

### Перезагрузка контейнера
```bash
docker-compose restart
```

### Удаление контейнера и образа
```bash
docker-compose down
docker-compose rm -f
```

### Запуск playbook с повышенной вербозностью
```bash
ansible-playbook -i inventory.ini playbook.yml -vvv
```

### Синтаксическая проверка playbook
```bash
ansible-playbook -i inventory.ini playbook.yml --syntax-check
```

---

## 10. Часто возникающие проблемы

### Проблема: "Permission denied (publickey)"
**Решение:**
```bash
# Проверьте права на приватный ключ
chmod 600 ~/.ssh/ansible_key

# Убедитесь, что публичный ключ скопирован правильно
docker exec ansible-managed-host cat /home/ansible/.ssh/authorized_keys
```

### Проблема: "No module named 'jinja2'"
**Решение:**
```bash
pip3 install jinja2
ansible-inventory -i inventory.ini --list
```

### Проблема: "Connection refused" на порту 2222
**Решение:**
```bash
# Проверьте, запущен ли контейнер
docker ps

# Пересоздайте контейнер
docker-compose down
docker-compose up -d
```

### Проблема: "UNREACHABLE! => {msg: 'Failed to connect to the host via ssh'"
**Решение:**
1. Проверьте SSH подключение вручную
2. Убедитесь, что SSH сервис запущен в контейнере
3. Проверьте права на файлы в `.ssh`

---

## Дополнительные ресурсы

- [Официальная документация Ansible](https://docs.ansible.com/)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/tips_tricks/index.html)
- [Ansible Modules Index](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/index.html)
- [Docker Documentation](https://docs.docker.com/)

---
