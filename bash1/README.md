###  1
Какие значения переменным c,d,e будут присвоены? Почему?
a=1
b=2
c=a+b - 
d=$a+$b
e=$(($a+$b))

Ответы

c = a + b, тк заданые перементые bash игнорит

d = 1 + 2, тк тут идет просто вывод переменных 

e = 3, тк тут уже идет математический расчет 

### 2

 while (( 1 == 1 ))
 
    do
        curl https://localhost:4757
        
        if (($? != 0))
        
        then
        
            date >> curl.log
            
        else exit
        
        fi
        
   done
   

### 3

root@zabbix:/opt# cat server

87.240.129.133

87.250.250.242

173.194.73.139

10.0.1.111

8.8.8.8

root@zabbix:/opt# cat script

#!/bin/bash

hosts=$( cat /opt/server)

timeout=5

for i in {1..5}

do

date >hosts.log

    for h in ${hosts[@]}
    
    do
    
        curl -Is --connect-timeout $timeout $h:80 >/dev/null
        
        echo "    check" $h status=$? >>hosts.log
        
    done
    
done

root@zabbix:/opt# cat hosts.log

Вс 18 дек 2022 17:06:16 UTC

    check 87.240.129.133 status=0
    
    check 87.250.250.242 status=0
    
    check 173.194.73.139 status=0
    
    check 10.0.1.111 status=0
    
    check 8.8.8.8 status=28
    
root@zabbix:/opt#

### 4


root@zabbix:/opt# cat 2


hosts=$( cat /opt/server)

timeout=5

res=0

while (($res == 0))

do

    for h in ${hosts[@]}
    
    do
    
        curl -Is --connect-timeout $timeout $h:80 >/dev/null
        
        res=$?
        
        if (($res != 0))
        
        then
        
            echo "    ERROR on " $h status=$res >2.log
            
        fi
        
    done
    
done

root@zabbix:/opt# cat server

87.240.129.133

87.250.250.242

173.194.73.139

10.0.1.111

8.8.8.8

root@zabbix:/opt# cat 2.log

    ERROR on  8.8.8.8 status=28
    
root@zabbix:/opt#


