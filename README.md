# Vdaishi_infra

Vdaishi Infra repository

###### Надо пнуть себя и привести в порядок Readme.md - Слово безработного!

https://eax.me/vim-commands/ - шпаргалка по Vim
https://www.markdownguide.org/basic-syntax - шпаргалка по Markdown
https://git-scm.com/book/ru/v2/ - Про Гит - книга

# Команда автоматической развертки

```
gcloud compute instances create reddit-app-auto --boot-disk-size=10GB --image-family ubuntu-1604-lts --image-project=ubuntu-os-cloud --machine-type=g1-small --tags puma-server --restart-on-failure --metadata-from-file startup-script=./startup_config.sh --zone=europe-west4-a
```

# команда создания через gcloud правила firewall

```
gcloud compute firewall-rules create "default-puma-server" --allow=tcp:9292 --direction=ingress --network=default  --target-tags=puma-server --source-ranges=0.0.0.0/0
testapp_IP = 34.90.229.174
testapp_port = 9292
```

# Cоздание образа в Packer

Шаблон создания образа Packer состоит из нескольких секций. В данном задании использовались секции Variable, Builders, Provisioners
Секция Variable отвечает за поля, которые могут быть заполнены пользователем либо через использование -var либо var-file, и в дальнейшем используются при создании образа, которые используются секцией Builders, при создании шаблона.
Секция Builders отвечает за создание машины, которая будет создана в каком либо провайдере или месте. В нем описываются параметры создания инстанса.
секция Provisioners отвечает за действия, выполняемые на только что созданной машине, это может быть как копирование файла, так и выполнение заранее прописанного кода.

После выполнения всех манипуляций, Packer сохраняет автоматически образ в провайдере, который в дальнейшем можно использовать.

При использовании методики Immutable infrastructure требуется создать образ, который будет меняться меньше всего.
```
packer validate -var-file=variables.json.example ubuntu16.json - проверка корректности кода создания образа Packer
packer build -var-file=variables.json ubuntu16.json - создание образа Packer
packer inspect ubuntu16.json - проверка функций, переменных используемых в коде
```
Собранный образ появится в браузерной консоли по пути Compute Engine --> Images.
packer build --var-file=variables.json immutable.json
Выполняем команду
```
gcloud compute instances create reddit-full --boot-disk-size=20 --image-family reddit-full --image-project=infra --machine-type=f1-micro --tags puma-server --restart-on-failure
```

---

# Задание Terraform -1

### Введение

С помощью Terraform можно контролировать и создавать инфраструктуру и управлять ею. В данном уроке рассматривается создание и управление инфраструктурой в GCP.

### Основные команды

для работы с Terraform требуется качать пакет terraform необходимой версии с сайта https://www.terraform.io/downloads.html .
Пример установки пакета Terraform на терминальную версию Ubuntu 18.04
```
wget https://releases.hashicorp.com/terraform/0.12.8/terraform_0.12.8_linux_amd64.zip
unzip  terraform_0.12.8_linux_amd64.zip
mv terraform ~bin/
```
Проверка версии Terraform
```
terraform -v
```
Перед началом работы с Terraform требуется выбрать каталог, в котором будут храниться файлы, перейти в него и произвести инициализацию Terraform.
Пример инициализации Terraform в папке

```
cd terraform
terraform init
```
### Основные файлы для работы с Terraform

При работе с Terraform файлы с разрешением `.tf` являются конфигурационными, где хранится информация планируемой инфраструктуре. Файлы с разрешением .`tfvars` хранят в себе переменные, которые могут быть в дальнешнем использованы в конфигурации. Все используемые переменные задаются в файле `Variables.tf` , либо в отдельном блоке конфигурационного файла. В файле Output.tf можно описать атрибуты ресурсов, которые будут выводиться постоянно в случае исполнения команды `terraform apply`.

### Работа с Terraform

После инициализации Terraform и настройки конфигураций правильным действием будет использование команды `terraform validate -var-file="yourfile"` чтобы Terraform проверил, что не допущено ошибок в конфигурации.
После валидации файлов, можно произвести просмотр плана развертывания конфигурации `terraform plan`. План конфигурации и изменений, вносимых в инфраструктуру будет выведен в консоли. Так же данный план будет выведен перед исполнением команды `terraform apply`, в случае, если не используется ключ ` -auto-appruve` .

В результате применения команды `terraform apply` мы получим результат, в котором будет отображено количество измененных, созданных либо удаленных ресурсов в облаке.

С помощью файла output.tf лучшим решением будет задать вывод информации об ip инстансов.

###  Исполнение заданий со *

###### задание 1

В случае, если был создан SSH ключ в обход terraform, то при исполнении `terraform apply` будет удален тот ключ(и) которые не были описаны в конфигурации.

###### задание 2

При создании балансировщика создаем отдельный файл, в котором будут описаны все параметры.
Особенности создания балансировщика :
- При создании регионального балансировщика, требуется нахождение всех правил, которые запрашивают регион, в одном и том же регионе (зона ≠ регион)
- Ресурсы для регионального и глобального балансировщика разные
- Цепочка зависимости команд
    * url-map
        * Target http proxy
            * Forwarding rule
        * Backend service
            * Instance group
                *Instances
            * Health check
Добавление новых инстансов при помощи копирования кода, является нерациональным и вносит человеческий фактор.
В связи с этим рациональнее использовать параметр count, который позволит создавать несколько инстансов одновременно. При этом следует внести небольшие изменения в именование инстансов (внести переменную) а так же в эту переменную обозначить в файле `Output.tf`. Так же в связи с тем, что `Forwarding rule` предоставляет нам IP адрес, следует его тоже внести в отображение файлом `Output.tf`.

#Terraform-2 Принципы организации инфраструктурного кода и работа над инфраструктурой в команде на примере Terraform

### Импорт существующей конфигурации

С помощью команды `import` можно добавить инфорацию об уже созданном без помощи Terraform правиле в state файл.
```
terraform import google_compute_firewall.firewall_ssh default-allow-ssh
```
### Добавление ресурса в виде IP адреса для инфраструктуры

Добавление адреса происходит при помощи ресурса `google_compute_address` .
для использования этого ресурса в коде инстанса, необходимо указать для `network_interface` что ему необходимо использовать именно этот ip.
```
network_interface {
network = "default"
access_config {
nat_ip = google_compute_address.app_ip.address
}
}
```
Тем самым образовывается неявная зависимость ресурсов друг с другом (в связи с этим ресурсы создаются поочередно, согласно зависимостям)

Указать явную зависимость можно используя параметр `depends_on`.
Это может быть необходимо в случае использования provisioners

### Структуризация ресурсов

Terraform поддерживает модульность инфраструктуры, позволяя использовать участки кода в разных сборках.

При создании новых инстансов, с явными ролями (сервер приложения и базы данных) требуется создать отдельные образы, для ускорения деплоя. Для этого мы создаем два отдельных шаблона под приложение и базу данных и разделяем скрипты на каждый из образов.

После этого создаем отдельные конфигурационные файлы инстансов `app.tf` и `db.tf`, в котором прописываем все то же самое, что и в старом файле, с поправкой на текущие образы. в связи с тем, что БД у нас теперь отдельно, надо создать отдельное правило Firewall, в котором прописывается доступ к порту, который слушает нас сервер.
```
# Правило firewall
resource "google_compute_firewall" "firewall_mongo" {
name = "allow-mongo-default"
network = "default"
allow {
protocol = "tcp"
ports = ["27017"]
}
target_tags = ["reddit-db"]
source_tags = ["reddit-app"]
}
```

В файле `vpc.tf` мы прописывает правило Firewall, в котором мы разрешаем ssh доступ к нашим ресурсам.


В связи с чем в файле `Main.tf` у нас остается только определение провайдера и ssh ключ.

### Модули

Благодаря модулям, мы можем использовать код в разных конфигурациях, без необходимости повторно писать код.

Для этого мы выделим файлы `app.tf` и `db.tf` в отдельные каталоги, в которых так же неоходимо наличие всех файлов, которые используются в данных сборках (к примеру в `app.tf` требуется еще создать отдельно каталог Files, в котором лежат скрипты деплоя приложения и сервиса)

Так же необходимо наличие файлов variables.tf и outputs.tf.

После этого, нам необходимо откорректированть наш `Main.tf`, указав путь к модулям, а так же указать переменные, которые он использует. Это все так же необходимо указать в файле `variables.tf` и `terraform.tfvars` (есть привер в файле `terraform.tfvars.example`)

С помощью команды `terraform get` мы получаем доступ к нашим модулям для нашей конфигурации.

Так же с использованием модулей требуется изменять параметры выходных данных, ссылаясь именно на модуль.
```
output "app_external_ip" {
value = module.app.app_external_ip
}
```

### Переиспользование модулей

Благодаря модульности мы можем создать инфраструктуру, которая на разных стадиях непрерывной поставки может использовать один и тот же код, который мы прописали в модулях.

Создав два разных окружения `stage` и `prod` мы укажем, что в stage версии нашей инфраструктуры предоставляется доступ по `ssh` ко всем ip а в `prod` только к личному ip пользователя.

Для этого создадим разные каталоги stage и `prod` в каждом из которых будут свои конфигурационные файлы `main.tf`, `variables.tf`, `outputs.tf`,`terraform.tfvars`.
так же с учетом того, что относительно нашей конфигурации изменились пути к нашим модулям, это так же надо исправить, указав вместо ./modules/xxx ../modules/xxx

Ничто не мешает запихнуть модули еще дальше, но надо ли это делать в данный момент...

# Реестр модулей

В реестре Terraform находится множество модулей, которые были созданы другими людьми. Бывают как Verified так и обычные. Verified это модули Hashicorp и ее партнеров.

Создадим бакет в GCP где в дальнейшем будем хранить бэкенд Terraform.
```
provider "google" {
version = "~> 2.15"
project = var.project
region = var.region
}
module "storage-bucket" {
source = "SweetOps/storage-bucket/google"
version = "0.1.1"
# Имена поменяйте на другие
name = ["storage-bucket-test", "storage-bucket-test2"]
}
output storage-bucket_url {
value = module.storage-bucket.url
}
```
**ВАЖНО!** Имя бакета должно быть уникальным, так как бэкенды - единая среда, в которой бакеты находятся вне проектов и должны быть названы так, как никто до вас не называл (уникально так же как учетная запись)

## Задание со *

Для каждого из окружений надо указать бэкендом недавно созданным нами бакет, в котором будет находиться наш стейт файл. Это позволит нескольким людям работать над одним проектом без появления проблем с развертыванием конфигурации (так как всем будет известно что развернуто на данный момент с помощью Terraform)
```
terraform {
  backend "gcs" {
   bucket = "terraform-state-remote-backend-storage-vdaishi"
   prefix = "prod"
   }
}
```
Попытка развернуть одновременно из `Stage` и `Prod` окружения приведет к блокировке, так как текущие конфигурации уже созданы.

## Задание с **

Чтобы развернуть приложение, надо вернуть обрано в `app.tf` наши `Provisioners` с некоторыми изменениями.
Terraform позволяет, перед созданием файла в новом инстансе внести в него изменения, для этого требуется из нашего `puma.service` создать файл `puma.service.tpl`. В дальнейшем в конфигурации модуля app в `main.tf` добавить `provisioner`.
```
  }
  provisioner "file" {
    content     = templatefile("${path.module}/files/puma.service.tpl", {database_url = var.database_url})
    destination = "/tmp/puma.service"
  }

```
так же необходимо немного изменить наш `puma.service.tpl` , добавив в секцию `[Service]` строку `Environment=DATABASE_URL=${database_url}`
Это необходимо, чтобы Puma в своей конфигурации заменила адрес до БД с `127.0.0.1` на адрес нашей базы данных в файле `app.rb` (находящейся в каталоге нашего приложения)
```
configure do
    db = Mongo::Client.new([ ENV['DATABASE_URL'] || '127.0.0.1:27017' ], database: 'user_posts', heartbeat_frequency: 2)
```
Еще необходимо указать как переменную локальный адрес нашей бд в `variables.tf`
```
variable "database_url" {
  description = "IP and port for conncection database"
  default     = "127.0.0.1:27017"
}
```
Так же в окружении в модуле app добавить саму переменную
```
module "app" {
...
  database_url     = module.db.db_internal_ip
}
```

###### Управление конфигурацией. Основные инструменты DevOps. Знакомство с Ansible

### Основы
Ansible работает в WSL Windows и Linux/Unix машинах и написан на Python.

Установить Ansible можено с помощью пакетного менеджера `pip` или `easy_install`.
для этого можно использовать одну из перечисленных команд, (в случае использования команд с файлом requirements.txt, требуется его создать со следующим содержимым `ansible>=2.4`)
```
pip install -r requirements.txt
pip install ansible>=2.4
easy_install `cat requirements.txt`
```

Проверить версию Ansible можно командой

```
ansible --version
```

# Основные ключи Ansible

`-i` - путь к инвентори файлу
`-m` - используемый модуль
`-a` - аргумент (команда) для используемого модуля, если необходимо

### Управление инстансами и группами

Ansible управляет инстансами виртуальных машин (c Linux ОС) используя SSH-соединение. Поэтому для управление инстансом при помощи Ansible нам нужно убедиться, что мы можем подключиться к нему по SSH. Для управления хостами при помощи Ansible на них также должен быть установлен Python >=2.7

Для управления Ansible необходимо указать, какие хосты ему требуется обслуживать.
Группы и хосты указываются в инвентори файле
инвентори файл состоит из:
- `servername` - краткое имя сервера;
- `ansible_host=dnsname_or_IP` - адрес хоста, к которому ansible будет подключаться.
- `ansible_user=username` - имя пользователя, под которым подключается ansible к данному хосту (может быть вынесен в отдельный файл *см. далее)
**Важно!** На каждый отдельный хост должна быть ***одна строка***:
```
servername ansible_host=1.2.3.4 ansible_user=username
```
Для группового управления хостами, в инвентори файле записи можно объединить в группы.
Имя группы пишется перед первой записью хоста в группе в формате `[имя_группы]`, к примеру:
```
[app] # ⬅ Это название группы
appserver ansible_host=10.30.20.40 # ⬅ Cписок хостов в данной группе
vailserver ansible_host=172.30.0.2

[db]
dbserver ansible_host=10.20.30.40
```

Так же в Ansible существует дефолтная группа `all`, с помощью которой можно обратиться ко всем инстансам, прописанных в инвентори файле.

Та же в Ansible есть возможность писать инвентори на YAML.

### Конфигурационный файл с дефолтными настройками инстансов
Для того, чтобы постоянно не указывать множество входных данных для наших конфигураций, в Ansible можно создать файл ansible.cfg, в котоом будут находится наши значения по умолчанию:
```
[defaults]
inventory = ./inventory
remote_user = username
private_key_file = ~/.ssh/username
host_key_checking = False
retry_files_enabled = False
```
Благодаря этому мы можем сократить размер команд а так же количество описываемой информации в инвентори файле.

### Модули Ansible

Ansible работает с модулями, которые в дальнейшем обрабатываются на необходимом нам хосте и позволяют исполнять команды на удаленном хосте.

Список модулей, разбираемых в уроке:

command - исполнение команды, указанной в ключе -a, на удаленном хосте без использования оболочек
```
ansible app -m command -a 'ruby -v'
```

При попытке указать две команды модулю выйдет ошибка
```
ansible app -m command -a 'ruby -v; bundler -v'
appserver | FAILED | rc=1 >>
ruby: invalid option -; (-h will show valid options) (RuntimeError)non-zero return code
```

shell - исполнение команды указанной в ключе -a с использованием оболочек
```
ansible app -m shell -a 'ruby -v; bundler -v'
```

ping - проверка связи до удаленного хоста
```
ansible app -m shell -a 'ruby -v; bundler -v'
```

systemd - Управление сервисами на удаленной машине
```
ansible db -m service -a name=mongod
```

git - Управление репозиториями на удаленной машине
```
ansible app -m git -a 'repo=https://github.com/express42/reddit.git dest=/home/appuser/reddit'

```
