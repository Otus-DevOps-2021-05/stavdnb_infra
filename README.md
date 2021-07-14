# stavdnb_infra
stavdnb Infra repository


## HW-05 cloud-bastion
Для доступа по SSH к ВМ, необходимо сгенирировать ключ для пользователя appuser

ssh-keygen -t rsa -f ~/.ssh/appuser -C appuser -P ""  (-C login -Р passphrase)

### По условию ДЗ были созданы 2 ВМ  :

  bastion_ip = 178.154.202.80
  someinternal_ip = 10.128.0.10

Проверяем подключение

ssh -i ~/.ssh/appuser appuser@178.154.202.80

Для подключения по ssh к someinternalhost через одну команду необходимо:

     - создать файл  ~/.ssh/config

'''
    Host bastion
    Hostname 178.154.202.80
    User appuser
    IdentityFile ~/.ssh/appuser

Host someinternalhost
    User appuser
    IdentityFile ~/.ssh/appuser
    ProxyCommand ssh -q bastion nc -q0 10.128.0.10 22
'''
После чего подключаться можно по имени ssh someinternalhost

Если процедура не частая подключаться можно также и через IP адрес предварительно указав forwarding (-А) и подкидывая на хост свой приватный ключ (.ррк)
'''
ssh-add ~/.ssh/appuser
'''
'''
ssh -A -t appuser@178.154.202.80 ssh 10.128.0.10
'''


### Устанавливаем и настраиваем vpn server Pritunl

Логинимся

ssh bastion

Выполняем следующие команды

'''
sudo tee /etc/apt/sources.list.d/pritunl.list << EOF
deb http://repo.pritunl.com/stable/apt focal main
EOF

sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv 7568D9BB55FF9E5287D586017AE645C0CF8E292A
sudo apt-get update
...

sudo apt install dirmngr gnupg apt-transport-https ca-certificates software-properties-common


wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -

sudo add-apt-repository 'deb [arch=amd64] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse'

apt install mongodb-org

apt install pritunl

systemctl enable mongod pritunl

sudo systemctl start mongod pritunl

'''
### С помощью сервиса https://sslip.io/ регистрируем сертификат Let's Encrypt

'''
sudo service pritunl stop
sudo wget https://bootstrap.pypa.io/get-pip.py
sudo python3 -m pip install certbot
sudo certbot certonly --standalone -d 178.154.227.222.sslip.io
sudo service pritunl start
sudo pritunl set app.server_cert "$(sudo cat /etc/letsencrypt/live/178.154.202.80.sslip.io/fullchain.pem)"
sudo pritunl set app.server_key "$(sudo cat /etc/letsencrypt/live/178.154.202.80.sslip.io/privkey.pem)"

'''
