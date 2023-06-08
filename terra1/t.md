# Домашнее задание к занятию «Введение в Terraform»

### Цель задания

1. Установить и настроить Terrafrom.
2. Научиться использовать готовый код.

------

### Чеклист готовности к домашнему заданию

1. Скачайте и установите актуальную версию **terraform** >=1.4.0 . Приложите скриншот вывода команды ```terraform --version```.
2. Скачайте на свой ПК данный git репозиторий. Исходный код для выполнения задания расположен в директории **01/src**.
3. Убедитесь, что в вашей ОС установлен docker.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. Установка и настройка Terraform  [ссылка](https://cloud.yandex.ru/docs/tutorials/infrastructure-management/terraform-quickstart#from-yc-mirror)
2. Зеркало документации Terraform  [ссылка](https://registry.tfpla.net/browse/providers) 
3. Установка docker [ссылка](https://docs.docker.com/engine/install/ubuntu/) 
------

### Задание 1

1. Перейдите в каталог [**src**](https://github.com/netology-code/ter-homeworks/tree/main/01/src). Скачайте все необходимые зависимости, использованные в проекте. 
2. Изучите файл **.gitignore**. В каком terraform файле согласно этому .gitignore допустимо сохранить личную, секретную информацию?
3. Выполните код проекта. Найдите  в State-файле секретное содержимое созданного ресурса **random_password**, пришлите в качестве ответа конкретный ключ и его значение.
4. Раскомментируйте блок кода, примерно расположенный на строчках 29-42 файла **main.tf**.
Выполните команду ```terraform validate```. Объясните в чем заключаются намеренно допущенные ошибки? Исправьте их.
5. Выполните код. В качестве ответа приложите вывод команды ```docker ps```
6. Замените имя docker-контейнера в блоке кода на ```hello_world```, выполните команду ```terraform apply -auto-approve```.
Объясните своими словами, в чем может быть опасность применения ключа  ```-auto-approve``` ? В качестве ответа дополнительно приложите вывод команды ```docker ps```
8. Уничтожьте созданные ресурсы с помощью **terraform**. Убедитесь, что все ресурсы удалены. Приложите содержимое файла **terraform.tfstate**. 
9. Объясните, почему при этом не был удален docker образ **nginx:latest** ? Ответ подкрепите выдержкой из документации провайдера.

Ответ на 1 вопрос
```
root@mail:/home/d/data2# ls -la
total 40
drwxr-xr-x  3 d    d    4096 May 21 18:52 .
drwxr-xr-x 19 d    d    4096 May 21 18:43 ..
-rw-r--r--  1 root root  155 May 21  2023 .gitignore
-rw-r--r--  1 root root  756 May 21  2023 main.tf
-rw-rw-r--  1 d    d     206 May 21 18:13 t
-rw-rw-r--  1 d    d    1425 May 21 18:49 ter-homeworks-01-src.zip
drwxr-xr-x  3 root root 4096 May 21 18:35 .terraform
-rw-r--r--  1 root root  434 May 21 18:35 .terraform.lock.hcl
-rw-r--r--  1 root root  206 May 21  2023 .terraformrc
-rw-r--r--  1 root root 1022 May 21 18:52 terraform.tfstate
root@mail:/home/d/data2# cat .gitignore
```
Ответ на 2 вопрос
```
personal.auto.tfvars - конфиг с значением переменных, часто является секретным, нужно с осторожностью пушить в публичные репозитарии.

Ответ на вопрос из проверки ДЗ: это файл с расширением .tfstate , который хранит описание развернутой инфраструктуры.
```
Ответ на 3 вопрос
```
"result": "vQPZv74Z0eral49m",
```
Ответ на вопрос 4 
```
небыло имени в строке 24, так же удалил лишнее в строке 31. 


root@mail:/home/d/data2# cat main.tf
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0.1"
    }
  }
  required_version = ">=0.13" /*Многострочный комментарий.
 Требуемая версия terraform */
}
provider "docker" {}

#однострочный комментарий

resource "random_password" "random_string" {
  length      = 16
  special     = false
  min_upper   = 1
  min_lower   = 1
  min_numeric = 1
}


resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = true
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "${random_password.random_string.result}"

  ports {
    internal = 80
    external = 8000
  }
}

```

Ответ на 5 вопрос

```
root@mail:/home/d/data2# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS              PORTS                  NAMES
878f22d1498b   a99a39d070bf   "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:8000->80/tcp   eWyPqf1VVj1sy4Ym
```
Ответ на 6 вопрос

```
Грубого говоря при использовании -auto-approve, конфиг применяется сразу и без подтверждения. ***По сути используя -auto-approve мы лишаем себя лишней визуальной проверки через plane (человеческий фактор рискует похерить все) ***
root@mail:/home/d/data2# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                  NAMES
2d796f80de0b   a99a39d070bf   "/docker-entrypoint.…"   34 seconds ago   Up 32 seconds   0.0.0.0:8000->80/tcp   hello_workd
```
Ответ на 7 вопрос

```

{
  "version": 4,
  "terraform_version": "1.4.6",
  "serial": 22,
  "lineage": "37ca9208-4894-794f-0768-8ab730d0e552",
  "outputs": {},
  "resources": [],
  "check_results": null
}

```
Ответ на 8 вопрос.
```
После изменения имени контейнера в файле main.tf мы по сути потеряли над ним управление, тк его нет в конфигурционном файле.
```

------

## Дополнительные задания (со звездочкой*)

**Настоятельно рекомендуем выполнять все задания под звёздочкой.**   Их выполнение поможет глубже разобраться в материале.   
Задания под звёздочкой дополнительные (необязательные к выполнению) и никак не повлияют на получение вами зачета по этому домашнему заданию. 

### Задание 2*

1. Изучите в документации provider [**Virtualbox**](https://registry.tfpla.net/providers/shekeriev/virtualbox/latest/docs/overview/index) от 
shekeriev.
2. Создайте с его помощью любую виртуальную машину. Чтобы не использовать VPN советуем выбрать любой образ с расположением в github из [**списка**](https://www.vagrantbox.es/)

В качестве ответа приложите plan для создаваемого ресурса и скриншот созданного в VB ресурса. 

------

### Правила приема работы

Домашняя работа оформляется в отдельном GitHub репозитории в файле README.md.   
Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

### Критерии оценки

Зачёт:

* выполнены все задания;
* ответы даны в развёрнутой форме;
* приложены соответствующие скриншоты и файлы проекта;
* в выполненных заданиях нет противоречий и нарушения логики.

На доработку:

* задание выполнено частично или не выполнено вообще;
* в логике выполнения заданий есть противоречия и существенные недостатки. 
