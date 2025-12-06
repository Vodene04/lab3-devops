# Лабораторная работа: Основы Ansible в DevOps

## Ход работы.
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


Проверили созданный ранее файл 'inventory.ini' с помощью команды:

```bash
ansible-inventory -i inventory.ini --list
```
Результат на рисунке ниже.


<img width="1280" height="675" alt="image" src="https://github.com/user-attachments/assets/29adbfb0-4e1e-4cf2-9ce1-0d10ce3f27b9" />



---

## 6. Проверка подключения Ansible к управляемому хосту

Протестировали ping


```bash
ansible -i inventory.ini managed_hosts -m ping
```

<img width="1280" height="124" alt="image" src="https://github.com/user-attachments/assets/194bfa9d-a43f-4cb5-a6af-9897e859a6ee" />



Собрали информацию о системе:

```bash
ansible -i inventory.ini managed1 -m setup
```

Вывод на экране:

<img width="1280" height="507" alt="image" src="https://github.com/user-attachments/assets/dfb4cb0f-8790-41c3-b162-05d612d956d8" />



Выполнили простую команду:

```bash
ansible -i inventory.ini managed1 -m command -a "uname -a"
```


<img width="1280" height="139" alt="image" src="https://github.com/user-attachments/assets/d8e65084-fcc8-4267-a726-a112f962eef6" />


---

## 7. Создание и запуск Ansible Playbook

Структура проекта выглядит следующим образом:
```
project/
├── Dockerfile
├── docker-compose.yml
├── inventory.ini
├── playbook.yml
└── README.md
```

Использовали готовый playbook из методических указаний и запустили playbook

```bash
# Запуск playbook
ansible-playbook -i inventory.ini playbook.yml
```
Результат на рисунке:


<img width="1280" height="572" alt="image" src="https://github.com/user-attachments/assets/c19b14db-cd40-49b8-ab3b-3dcbb059858e" />



## 8. Задания для выполнения

### Задание 1: Базовое подключение
Задание 1 было выполнено в ходе выполнения первых 7 шагов.

---

### Задание 2: Базовые ad-hoc команды
1. Получили информацию о ядрах CPU управляемого хоста:
   ```bash
   ansible -i inventory.ini managed1 -m setup -a "filter=ansible_processor_cores"
   ```

2. Проверили свободное место на диске:
   ```bash
   ansible -i inventory.ini managed1 -m command -a "df -h"
   ```

3. Получили список всех пользователей:
   ```bash
   ansible -i inventory.ini managed1 -m command -a "cat /etc/passwd"
   ```

4. Изменили временную зону хоста на UTC:
   ```bash
   ansible -i inventory.ini managed1 -m command -a "timedatectl set-timezone UTC"
   ```

Вывод введенных команд представлен на изображении:



<img width="1280" height="645" alt="image" src="https://github.com/user-attachments/assets/548fd42c-8a0c-4196-a802-18646d5c030d" />


---

### Задание 3: Работа с файлами
1. Создали новый playbook `task3_files.yml`:
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

2. Запустили playbook:
   ```bash
   ansible-playbook -i inventory.ini task3_files.yml
   ```


Результат представлен на изображении:


<img width="1280" height="645" alt="image" src="https://github.com/user-attachments/assets/05ff5dd1-e795-46bc-8d7d-e1702e942959" />



---

## Вывод

В ходе выполнения лабораторной работы установили Ansible на управляющую машину, подготовили SSH ключей для управляемых машин,запустили управляемый контейнер в Docker,
проверили SSH подключение к контейнеру, создали инвентарный файл Ansible (inventory), проверили подключение Ansible к управляемому хосту, создали и запустили Ansible Playbook,
а также выполнили базовые ad-hoc команды.

