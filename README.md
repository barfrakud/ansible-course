# Przygotowanie laba pod Ansible




## Skonfigurowanie pliku Vagrantfile

Uruchomienie vagrant:
```
vagrant up
```

## Skonfigurowanie dostępu przez SSH
Zalogowanie się przez ssh do ansible-control:

```
vagrant ssh ansible-control
```

Skonfigurowanie pliku hosts_file (zmapowanie ip na nazwy):  
Plik `hosts_file`

```
192.168.56.200 ansible-control acon
192.168.56.201 db01
192.168.56.202 web01
192.168.56.203 web02
192.168.56.204 loadbalancer lb01
```

Skopiowanie:
```
# cp hosts_file /etc/hosts
```

Wygenerowanie kluczy ssh:
```
ssh-keygen
```
Skopiowanie klucza prywatnego do hostów:
```
ssh-copy-id <host_name>
```

## Instalacja Ansible
Instalajca ansible poprzez:
```
apt install ansible
```

Weryfikacja instalacji - sprawdzenie wersji
```
ansible --version
ansible localhost -m command -a date    
```

## Skonfigurowanie pliku inventory
Plik inventory może mieć dowolną nazwę:
- hosts 
- inventory  

Przykładowo:
```
[control]
ansible-control

[proxy]
loadbalancer

[webservers]
web01
web02

[database]
db01

[webstack:children]
proxy
webservers
database
```

## Wydanie komendy ansible
Sprawdzenie komendy hostname
```
ansible webstack -i hosts -m command -a hostname
```
Sprawdzenie komendy date
```
ansible webstack -i hosts -m command -a date
```

Zainstalowanie na wszystkich hostach programu "tree":
```
ansible all -i hosts -m command -a 'sudo apt-get -y install tree'
```
## Komendy Ad-hoc
### Struktura
```
ansible <zakres_hostów> -i <inventory_file> -m <typ_modułu> -a <parametry_modułu>
```

### Moduł APT
Run the equivalent of apt-get update on linux:
```
ansible all -i hosts --become -m apt -a "update_cache=yes"
```
gdzie:  
`--become` to uruchomienie komendy z prawami sudo  


Zainstalowanie serwera Apache:
```
ansible webservers -i hosts --become -m apt -a "name=apache2 state=present"
```

Zainstalowanie serwera mysql:
```
ansible database -i hosts --become -m apt -a "name=mysql-server state=present"
```

### Moduł SERVICE
Uruchomienie mysql (który jest już uruchomiony):
```
ansible database -i hosts -m service -a "name=mysql state=started"
```
Restart mysql
```
ansible database --become -i hosts -m service -a "name=mysql state=restarted"
```

## Playbook

Skonfigurowanie pliku playbook1.yml  
Zadanie to zweryfikowanie, że Apache jest w najnowszej wersji:
```
---
- hosts: webservers
  become: yes
  vars:
    http_port: 8000
    https_port: 4443
    html_welcome_msg: "Hello world! BK is awesome!"
  tasks:
  - name: ensure apache is at the latest version
    apt:
      name: apache2
      state: latest
```

Uruchomienie playbook'a:
```
ansible-playbook -i hosts -K playbook1.yml
```

### Handlers
Sometimes you want a task to run only when a change is made on a machine. For example, you may want to restart a service if a task updates the configuration of that service, but not if the configuration is unchanged. Ansible uses handlers to address this use case. Handlers are tasks that only run when notified.

# Roles (lab05)
 
## Inicjalizacji nowej roli 
Utworzenie nowej roli apache2
```
ansible-galaxy init roles/apache2
```
Równocześnie tworzona jest nowa struktura folderów.  
Do tej struktury wkleja się poszczególne pliki:
```
.
├── hosts
├── playbook1.yml
└── roles
    └── apache2
        ├── defaults
        │   └── main.yml
        ├── handlers
        │   └── main.yml
        ├── meta
        │   └── main.yml
        ├── README.md
        ├── tasks
        │   ├── apache2_install.yml
        │   └── main.yml
        ├── templates
        │   ├── index.html.j2
        │   └── ports.conf.j2
        ├── tests
        │   ├── inventory
        │   └── test.yml
        └── vars
            └── main.yml
```
W tym labie ustawiony jest loadbalancer na nginx do dwóch serwerów web01 i web02.  
    
# Tags (Lab06)

Dodanie tagów w playbook:
```
---
- hosts: webservers
  become: yes
  vars:
    http_port: 8000
    https_port: 4443
    html_welcome_msg: "Hello world!"
  roles:
    - common
    - apache2
  tags:
    - web

- hosts: proxy
  become: yes
  roles: 
    - common
    - nginx
  tags:
    - proxy
```

To report back tags:
```
ansible-playbook -i hosts -K playbook1.yml --list-tags
```

To run only plays assosiated to proxy using tags:
```
ansible-playbook -i hosts -K playbook1.yml --tags proxy
```

## Adding tags at task level

Dodaje się tagi w rolach np:
```
---
- name: "Install common packages"
  apt: 
    name:
      - pydf
      - tree
      - mc
    state: latest
  tags: installation
  ```
  Uruchamianie playbooka z poszczególnym tagiem:
  ```
  ansible-playbook -i hosts -K playbook1.yml --tags web,configuration
  ```