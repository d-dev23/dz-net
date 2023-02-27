devops-netology
===============

# Домашнее задание к занятию "5.5. Оркестрация кластером Docker контейнеров на примере Docker Swarm"

## Задача 1

<details>
<summary>.</summary>

> Дайте письменые ответы на следующие вопросы:
> 
> - В чём отличие режимов работы сервисов в Docker Swarm кластере: replication и global?
> - Какой алгоритм выбора лидера используется в Docker Swarm кластере?
> - Что такое Overlay Network?

</details>

> В чём отличие режимов работы сервисов в Docker Swarm кластере: replication и global?

В режиме `replicated` приложение запускается в том количестве экземпляров, какое укажет пользователь. При этом на отдельной ноде может быть как несколько экземпляров приложения, так и не быть совсем. 

В режиме `global` приложение запускается обязательно на каждой ноде и в единственном экземпляре. 

> Какой алгоритм выбора лидера используется в Docker Swarm кластере?

**Raft**. 
- Протокол решает проблему согласованности: чтобы все manager ноды имели одинаковое представление о состоянии кластера
- Для отказоустойчивой работы должно быть не менее трёх manager нод. 
- Количество нод обязательно должно быть нечётным, но лучше не более 7 (это рекомендация из документации Docker).
- Среди `manager` нод выбирается лидер, его задача гарантировать согласованность. 
- Лидер отправляет keepalive пакеты с заданной периодичностью в пределах 150-300мс. Если пакеты не пришли, менеджеры начинают выборы нового лидера. 
- Если кластер разбит, нечётное количество нод должно гарантировать, что кластер останется консистентным, т.к. факт изменения состояния считается совершенным, если его отразило большинство нод. Если разбить кластер пополам, нечётное число гарантирует что в какой-то части кластера будеть большинство нод.

> Что такое Overlay Network?

L2 VPN сеть для связи демонов Docker между собой. В основе используется технология `vxlan`

## Задача 2

<details>
<summary>.</summary>

> Создать ваш первый Docker Swarm кластер в Яндекс.Облаке
> 
> Для получения зачета, вам необходимо предоставить скриншот из терминала (консоли), с выводом команды:
> ```
> docker node ls
> ```

</details>

```bash
[root@node01 ~]# docker node ls
ID                            HOSTNAME             STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
q7ybewzhnlt4g8i4q2hgsme16 *   node01.netology.yc   Ready     Active         Leader           20.10.11
rjgo07uaqsffxaduqgogexh4y     node02.netology.yc   Ready     Active         Reachable        20.10.11
hnvx6ug830f6q2orbt21wl536     node03.netology.yc   Ready     Active         Reachable        20.10.11
yvksdjnnhdz7m918gjodmime5     node04.netology.yc   Ready     Active                          20.10.11
1actkrhyg3s1gaboy6ibf1wv6     node05.netology.yc   Ready     Active                          20.10.11
vovxqnougcs1d37cui2vv1bs6     node06.netology.yc   Ready     Active                          20.10.11
```

## Задача 3

<details>
<summary>.</summary>

> Создать ваш первый, готовый к боевой эксплуатации кластер мониторинга, состоящий из стека микросервисов.
> 
> Для получения зачета, вам необходимо предоставить скриншот из терминала (консоли), с выводом команды:
> ```
> docker service ls
> ```

</details>

```bash
[root@node01 ~]# docker service ls
ID             NAME                                MODE         REPLICAS   IMAGE                                          PORTS
m5by05q5moyf   swarm_monitoring_alertmanager       replicated   1/1        stefanprodan/swarmprom-alertmanager:v0.14.0    
hys6xv2fdtol   swarm_monitoring_caddy              replicated   1/1        stefanprodan/caddy:latest                      *:3000->3000/tcp, *:9090->9090/tcp, *:9093-9094->9093-9094/tcp
252i1tqzcp7e   swarm_monitoring_cadvisor           global       6/6        google/cadvisor:latest                         
45uh23r36c5r   swarm_monitoring_dockerd-exporter   global       6/6        stefanprodan/caddy:latest                      
tze6cvq8ivs8   swarm_monitoring_grafana            replicated   1/1        stefanprodan/swarmprom-grafana:5.3.4           
zlnmt3l8f7oe   swarm_monitoring_node-exporter      global       6/6        stefanprodan/swarmprom-node-exporter:v0.16.0   
fanirldjts6j   swarm_monitoring_prometheus         replicated   1/1        stefanprodan/swarmprom-prometheus:v2.5.0       
lp5ti1ot5ea2   swarm_monitoring_unsee              replicated   1/1        cloudflare/unsee:v0.8.0 
```

## Задача 4 (*)

<details>
<summary>.</summary>

> Выполнить на лидере Docker Swarm кластера команду (указанную ниже) и дать письменное описание её функционала, что она делает и зачем она нужна:
> ```
> # см.документацию: https://docs.docker.com/engine/swarm/swarm_manager_locking/
> docker swarm update --autolock=true
> ```

</details>

```bash
[root@node03 ~]# docker node ls
Error response from daemon: Swarm is encrypted and needs to be unlocked before it can be used. Please use "docker swarm unlock" to unlock it.
[root@node03 ~]# docker swarm unlock
Please enter unlock key: 
[root@node03 ~]# docker node ls
ID                            HOSTNAME             STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
q7ybewzhnlt4g8i4q2hgsme16     node01.netology.yc   Ready     Active         Leader           20.10.11
...
```

`--autolock=true` (или `init --autolock` при создании кластера) заставит вводить ключ разблокировки на `manager` ноде, чтобы она могла заново присоединиться к кластеру, если была перезапущена. Ввод ключа позволит расшифровать лог Raft и загрузить все "секреты" в память ноды (логины, пароли, TLS ключи, SSH ключи и [прочие данные](https://docs.docker.com/engine/swarm/secrets/#about-secrets))

Верояно, это нужно чтобы защитить кластер от несанкционированного доступа к файлам ноды. Например, если кто-то получил жесткий диск сервера или образ диска виртуальной машины с нодой, чтобы он не мог получить доступ к кластеру и нодам без пароля (=токена, ключа).
