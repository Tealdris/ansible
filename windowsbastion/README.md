# Общие сведения
Данный плейбук предназначен для запуска заданий на windows хостах, которые доступны только через bastion jump host

# Схема

192.168.56.163 **windows** server

172.18.0.3 **bastion** jump host 

172.18.0.2 **ansible** server

ansible server доступен только для bastion сервера

# Порядок действий
## windows
1. запустить скрипт для включения winrm от имени администратора
```
https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1
```

## bastion
1. создать пользователя
2. иметь необходимые зависимости
```
для ubuntu
apt-transport-https wget curl openssh-server sudo sshpass iproute2 iputils-ping git nano python3 python3-pip gcc 

для python
ansible pywinrm[kerberos] pywinrm jmspath requests requests[socks] pypsrp
```
## ansible
1. создать пользователя
2. иметь необходимые зависимости

```
для ubuntu
apt-transport-https wget curl openssh-server sudo sshpass iproute2 iputils-ping git nano python3 python3-pip gcc 

для python
ansible pywinrm[kerberos] pywinrm jmspath requests requests[socks] pypsrp
```

3. создать ключ и использовать данный ключ для аутентификации с машины, где выполняется playbook, на сервере bastion
4. В настроенном плейбуке использовать переменные для настройки jump host для машин windows

```
[windows:vars]
# требуется указать параметры для удаленной машины windows
ansible_host = 192.168.56.163
ansible_user = user
ansible_password = 'qwerasdfzxcv1234'

# данный блок описывает:
# протокол взаимодействия ## использование winrm + bastion host недоступно на данный момент
# порт
# используемый сертификат ## при использовании самопожписанного сертификата поставить ignore
# указать путь для bastion host

ansible_connection = psrp
ansible_port = 5986 
ansible_psrp_cert_validation = ignorе
ansible_psrp_proxy=socks5h://127.0.0.1:1234
```

5. Указать ip адрес и имя пользователя bastion машины в файле psrp_open_connection/vars/main.yml. По этим данным будет осуществлятся ssh соединение.
6. Запустить playbook

# Важное

Данный плейбук работает следующим образом:

1. Первое задание - создает туннель от машины, где выполняется playbook, до bastion host. 
Флаг -N позволяет запустить процесс в фоне, следовательно выполнение не будет прервано, а туннель будет доступен
```
ssh -4 -D 1234 -p 22 test@172.18.0.3 -N &
```
2. Второе задание - любые действия, которые требуется выполнить на удаленной windows машине. В данном примере выполняется проверка доступности и сбор данных о машине

3. Третье задание - созданный на первом этапе туннель продолжит существовать, если его не выключить. Поэтому в задании происходит поиск и завершение всех соединений ssh на машине, где выполняется playbook.
