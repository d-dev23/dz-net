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
В Docker Swarm кластере используется так называемый алгоритм поддержания распределенного консенсуса — Raft. Выбор лидера происходит следующим образом: если ноды-фолловеры не слышат лидера, они переходят в статус кандидата, кандидат на лидера отправляет остальным нодам запрос на голосование и, большинством голосов, выбирается лидером.

> Что такое Overlay Network?

VPN сеть созданная поверх других сетей докера, служит для объединения, физицески не связаных сетей.

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


