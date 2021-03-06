# SergeSpinoza_platform
SergeSpinoza Platform repository

## Оглавление
1. [Домашнее задание №1, Minikube](#домашнее-задание-1)
2. [Домашнее задание №2, RBAC](#домашнее-задание-2)
3. [Домашнее задание №3, Сети](#домашнее-задание-3)
4. [Домашнее задание №4, Volumes, Storages, StatefulSet](#домашнее-задание-4)
5. [Домашнее задание №5, Kubernetes-storage](#домашнее-задание-5)
6. [Домашнее задание №6, Kubernetes-debug](#домашнее-задание-6)
7. [Домашнее задание №7, Kubernetes-operators](#домашнее-задание-7)

<br><br>
## Домашнее задание 1
### Выполнено
- Развернут minikube ( https://kubernetes.io/docs/tasks/tools/install-minikube/ );
- Добавлен Dashboard (команда `minikube addons enable dashboard`, зайти в панель - команда `minikube dashboard`);
- Установлен k9s ( https://k9ss.io );
- Проверено, что контейнеры восстанавливаются после удаления;
- Создан Dockerfile, который запускает http сервер на 8000 порту под пользователем с uid 1001 (сделал на python); 
- Написан манифест web-pod.yaml;
- Выполнен деплой пода web и init контейнера;
- Опробован в работе kube-forwarder (https://kube-forwarder.pixelpoint.io ).


### Ответы на вопросы
1. `kube-apiserver` запускается и рестартится через Systemd Service операционной системы, а `coredns` - контролируется самим k8s через ReplicaSet; 

### Полезное
Команды: 
- `minikube start` - старт minikube;
- `minikube ssh` - войти на виртуальную машину minikube по ssh;
- `kubectl cluster-info` - проверка подключения к кластеру;
- `kubectl get pods -n kube-system` - показать все pods в namespace `kube-system`;
- `kubectl get cs` - показать состояние кластера;
- `kubectl get ...` - показать ресурсы;
- `kubectl describe ...` - показать детальную информацию о конктретном ресурсе;
- `kubectl logs ...` - показать логи контейнера в поде;
- `kubectl exec ...` - выполнить команду в контейнере в поде;
- `kubectl apply -f file.yaml` - применить манифест;  
- `kubectl get pod web -o yaml` - получить манифест уже запущенного pod;
- `kubectl port-forward --address 0.0.0.0 pod/web 8000:8000` - редирект порта (в примере 8000);


<br><br>
## Домашнее задание 2
### Выполнено
- TASK 01:
  - Создать Service Account bob, дать ему роль admin в рамках всего кластера;
  - Создать Service Account dave без доступа к кластеру;
- TASK 02:
  - Создать Namespace prometheus;
  - Создать Service Account carol в этом Namespace;
  - Дать всем Service Account в Namespace prometheus возможность делать get, list, watch в отношении Pods всего кластера;
- TASK 03:
  - Создать Namespace dev;
  - Создать Service Account jane в Namespace dev;
  - Дать jane роль admin в рамках Namespace dev;
  - Создать Service Account ken в Namespace dev;
  - Дать ken роль view в рамках Namespace dev.


### Полезное
Команды: 
- `kubectl auth can-i get/create ...` - проверка прав/возможности создания (пример `kubectl auth can-i get deployments --as system:serviceaccount:dev:ken -n dev`, `kubectl auth can-i create deployments --as system:serviceaccount:dev:ken -n dev`).


<br><br>
## Домашнее задание 3
### Выполнено
- Добавили к ранее созданному манифесту web-pod.yaml readinessProbe и livenessProbe;
- Создали манифест deployment с указанием стратегии обновления;
- Создали манифест сервиса с ClusterIP web-svc-cip;
- Переключили kube-proxy в режим `ipvs`; 
- Очистили правила iptables в minikube от мусора; 
- Настроили работу с MetalLB (к сожалению в macos доступ через браузер получить не удалось даже с прописанным маршрутом и драйвером hyperkit);
- Установили и настроили работу ingress-nginx; 

### Задание со * 
- Настроен LoadBalancer для CoreDNS (kube-dns) для 53 TCP и UDP портов с одинаковым внешним ip. Для проверки необходимо применить манифест `kubectl apply -f coredns-svc-lb.yaml` и проверить, выполнив команду из ssh minikube, например (c выводом):

```
# nslookup web-svc.default.svc.cluster.local 172.17.255.3
Server:    172.17.255.3
Address 1: 172.17.255.3 coredns-svc-lb-tcp.kube-system.svc.cluster.local

Name:      web-svc.default.svc.cluster.local
Address 1: 172.17.0.7 172-17-0-7.web-svc.default.svc.cluster.local
Address 2: 172.17.0.9 172-17-0-9.web-svc.default.svc.cluster.local
Address 3: 172.17.0.8 172-17-0-8.web-svc.default.svc.cluster.local
```
- Добавлен доступ к kubernetes-dashboard через настроенный ingress-proxy (манифест добавлен в файл `web-ingress.yaml`);
- Реализовано канареечное развертывание, манифесты `canary-namespaces.yaml` и `canary-ingress.yaml`. 

### Развертывание и проверка канареечного развертывания: 
- Применить манифест `kubectl apply -f canary-namespaces.yaml`, который создаст необходимые namespace;
- Развернуть приложение production `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/docs/examples/http-svc.yaml -n echo-production`; 
- Развернуть "тестовое" приложение `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/docs/examples/http-svc.yaml -n echo-canary`;
- Применить манифест `kubectl apply -f canary-ingress.yaml` для создания ingress для канареечного развертывания;
- Узнать ip minikube, выполнив команду `minikube ip`;
- Прописать в /etc/hosts следующую строчку: `<minikube_ip> echo.com`
- Проверить работу следующим образом: 
  - Через браузер зайти на сайт http://echo.com, обратить внимание, что строка `pod namespace:` имеет значение: `echo-production`; 
  - С помощью плагина ModHeader для браузера googleChome (ссылка https://chrome.google.com/webstore/detail/modheader/idgpnmonknjnojddfkpgkljpfnnfcklj?hl=ru) добавить заголовок `CanaryByHeader` со значением `DoCanary` и обновить страницу;
  - После этого строка `pod namespace:`должна изменить значение на: `echo-canary`.


### Ответы на вопросы
1. Почему следующая конфигурация валидна, но не имеет смысла?

```
livenessProbe:
  exec:
   command:
     - 'sh'
     - '-c'
     - 'ps aux | grep my_web_server_process'
```

**Ответ:** потому что данная проверка всегда будет отдавать процесс самой команды grep. Чтобы эта проверка работала - надо исключить из результатов grep - например так:

```
livenessProbe:
  exec:
   command:
     - 'sh'
     - '-c'
     - 'ps aux | grep -v grep | grep my_web_server_process'
```

2. Бывают ли ситуации, когда она все-таки имеет смысл?
**Ответ:** если исключать в выводе процесс самой команды grep - то это ситуации когда процесс только делает что-то внутри контейнера (допустим пишет что-то в файле на persistent volume) и не слушает никакой tcp/udp порт (т.е. проверить этот сервис по сети не возможно). 

3. Попробуйте разные варианты деплоя с крайними значениями maxSurge и maxUnavailable (оба 0, оба 100%, 0 и 100%)
**Ответ:** если maxSurge и maxUnavailable выставить в 0 - то получаем ошибку невозможности выполнения деплоя, вида 
```
The Deployment "web" is invalid: spec.strategy.rollingUpdate.maxUnavailable: Invalid value: intstr.IntOrString{Type:0, IntVal:0, StrVal:""}: may not be 0 when `maxSurge` is 0
```
При других вариантах все работает корректно, т.к. maxUnavailable задает максимальное количество недоступных подов при обновлении, а maxSurge - превышение количества подов при обновлении от заданного в `replicas`;


### Полезное
Команды: 
- `kubectl apply -f web-pod.yaml --force` - принудительное обновление;
- `kubectl --namespace kube-system edit configmap/kube-proxy` - редактирование конфига kube-proxy (для включения ipvs - выставить `mode` в `ipvs`); 
- `minikube ssh` - зайти в ssh minikube;
- `iptables --list -nv -t nat` - показать nat правила iptables;
- Внутри ssh minikube команда `toolbox` - запустить и зайти в контейнер с Fedora;
  - `dnf install -y ipvsadm && dnf clean all` - установка ipvsadm внутри контейнера Fedora;
  - `ipvsadm --list -n` - список правил ipvs;
- `kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.8.0/manifests/metallb.yaml` - установка MetalLb;
- `kubectl --namespace metallb-system logs controller-xxxxxxxxxxx-xxxxx` - логи контроллера MetalLB
- `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml` или `minikube addons enable ingress` - установка ingress-nginx;


<br><br>
## Домашнее задание 4
### Выполнено
- Установлен и запущен kind ( https://github.com/kubernetes-sigs/kind/ );
- Развернут StatefulSet с Minio;
- Развернут Headless Service.


### Задание со * 
- Учетные данные из StatefulSet вынесены в secret (манифест 01-minio-secret.yaml);
- Для кодирования учетных данных в base64 необходимо выполнить команды:  
  - `echo -n 'login' | base64`;
  - `echo -n 'password' | base64`
- Полученные значения записать в secret манифест. 


### Полезное
Команды: 
- `kind create cluster` - запуск кластера kind;
- `kind delete cluster` - удаление кластера kind
- `kubectl get statefulsets`
- `kubectl get pods`
- `kubectl get pvc`
- `kubectl get pv`
- `kubectl describe <resource> <resource_name>`


<br><br>
## Домашнее задание 5
## Kubernetes-storage
### Выполнено
- Подготовлен файл cluster.yaml с необходимой конфигурацией для kind;
- В директории `hw` созданы манифесты для создания объектов StorageClass, pvc и pod;
- Задание со * - развернут iscsi (добавлен диск lvm), в под добавлен volume iscsi, описана методика создания снапшотов и восстановления из них; 

### Как запустить: 
#### Предварительные действия: 
- Установить go версии > 1.13 (https://sourabhbajaj.com/mac-setup/Go/README.html)
- Установить kind (https://kind.sigs.k8s.io/docs/user/quick-start/)
- Перейти в директорию `kubernetes-storage/cluster` и создать кластер с помощью kind с кастомным конфигом: `kind create cluster --config cluster.yaml`
- Задать переменную окружения, для использования конфига kind: `export KUBECONFIG="$(kind get kubeconfig-path)"`
- Установить CSI Host Path Driver: 
  - `git clone https://github.com/kubernetes-csi/csi-driver-host-path.git`
  - `./csi-driver-host-path/deploy/kubernetes-1.15/deploy-hostpath.sh`

#### Основные действия: 
- Перейти в директорию `kubernetes-storage/hw` и последовательно выполнить: 
  - `kubectl apply -f 01-storageclass.yaml`
  - `kubectl apply -f 02-storage-pvc.yaml`
  - `kubectl apply -f 03-storage-pod.yaml`
- Проверить результат. 

#### Задание со * 
**Для развертывания iscsi необходимо выполнить команды на чистой машине с ubuntu 18.04:**
- `apt -y install targetcli-fb` - устанавливаем targetcli;
- Добавляем еще один диск (допустим /dev/sdb); 
- Произведем действия, для создания lvm 
```
# parted /dev/sdb
mklabel msdos
q
```

```
# parted -s /dev/sdb unit mib mkpart primary 1 100% set 1 lvm on
```

```
# pvcreate /dev/sdb1
# vgcreate vg0 /dev/sdb1
# lvcreate -l 10%FREE -n base vg0
# mkfs.ext4 /dev/vg0/base
```

**Для создания блочного устройства на отдельном диске:**
- `targetcli` - открывает targetcli
  - `/> ls` - просмотр иерархии;
  - `/> backstores/block create name=iscsi-disk dev=/dev/vg0/base` - создаем блочное устройство в бэксторе;

**Для создания блочного устройства в виде файла на существующем диске:**
- `mkdir /tmp/iscsi_disks`
- `targetcli` - открывает targetcli
  - `/> ls` - просмотр иерархии;
  - `/> backstores/fileio create iscsi_file /tmp/iscsi_disks/disk01.img 1G` - создаем блочное устройство в бэксторе;

**Далее:**
- `/iscsi create` - создаем таргет;
- `/iscsi` и `ls` - просмотрим название созданного таргета; 
- `iqn.2003-01.org.linux-iscsi.iscsi-1.x8664:sn.c0904cfa5297/tpg1/` - перейдем к созданному таргету к tpg1;
- `luns/ create /backstores/block/iscsi-disk` - создадим LUN;
- `set attribute authentication=0` - отключим авторизацию;
- `acls/` - перейдем в ACL;
- `create wwn=iqn.2019-09.com.example.srv01.initiator01` - указать iqn имя инициатора, который будет иметь право подключаться к таргету (это имя потом надо будет прописать в настройках воркер-нод кластера kubernetes, подробнее ниже);
- `cd /`, `ls`, `saveconfig` - сохраняем конфиг (по умолчанию сохраняется сюда: `/etc/rtslib-fb-target/saveconfig.json`);
- `exit`
(взято отсюда https://kifarunix.com/how-to-install-and-configure-iscsi-storage-server-on-ubuntu-18-04/)

**Далее разворачиваем кластер (я использовал kubespray). При создании машин кластера необходимо сразу при создании добавить второй интерфейс, который будет подключен к сети где находиться iscsi.** 
- `git clone https://github.com/kubernetes-sigs/kubespray.git`;
- `sudo pip install -r requirements.txt`;
- `cp -rfp inventory/sample inventory/mycluster`;
- отредактировать файл `inventory/mycluster/inventory.ini` согласно структуре кластера;
- `ansible-playbook -i inventory/mycluster/inventory.ini --become --become-user=root --user=spinoza --key-file=~/.ssh/id_rsa cluster.yml` - развернуть кластер; 

**Действия на воркер-нодах кластера kubernetes:**
- `apt -y install open-iscsi` - установим open-iscsi
- настроим конфиг `/etc/iscsi/initiatorname.iscsi`, внеся туда корректное имя, которое мы использовали ранее `iqn.2019-09.com.example.srv01.initiator01`
- добавим open-iscsi в автозагрузку и запустим:
  - `systemctl restart iscsid open-iscsi`
  - `systemctl enable iscsid open-iscsi`

**Проверка и снапшоты:**
- `kubectl apply -f kubernetes-storage/iscsi/01-iscsi-pod.yaml` - запустим под с использованием iscsi;
- зайдем в под `kubectl exec -it iscsi-pod -- /bin/bash`
- создадим файлик `echo "ISCSI TEST!" > /mnt/iscsi-test.txt`
- перейдем на машину с iscsi и создадим lvm снапшот: 
  - `lvcreate --snapshot --size 1G  --name ss-01 /dev/vg0/base`
- перейдем обратно в под и удалим данные - `rm -rf /mnt/iscsi-test.txt`
- удалим сам под `kubectl delete -f kubernetes-storage/iscsi/01-iscsi-pod.yaml`
- передем на машину с iscsi и исключим наш диск из iscsi: 
  - `targetcli`
  - `/> backstores/block delete iscsi-disk`
  - `exit`
  - восстановимся из снапшота `lvconvert --merge /dev/vg0/ss-01`
  - `targetcli`
  - `/> backstores/block create name=iscsi-disk dev=/dev/vg0/base`
  - `/> /iscsi/iqn.2003-01.org.linux-iscsi.iscsi-1.x8664:sn.c0904cfa5297/tpg1/`
  - `/> luns/ create /backstores/block/iscsi-disk`
  - `exit`
- снова запустим под `kubectl apply -f kubernetes-storage/iscsi/01-iscsi-pod.yaml`
- зайдем в под `kubectl exec -it iscsi-pod -- /bin/bash`
- проверим, что наш файл на месте `cat /mnt/iscsi-test.txt`


<br><br>
## Домашнее задание 6
## Kubernetes-debug
### Выполнено
- Ознакомились с работой `kubectl debug`, выявили проблему с агентом debug;
- Установили kube-iptables-tailer и необходимые для него сервис аккаунт, роль и биндинг;
- Установили тестовое приложение netperf-operator (оператор, который позволяет запускать тесты
пропускной способности сети между нодами кластера); 
- Добавили сетевую потилику Calico для эмуляции сетевой проблемы;
- Посмотрели логи calico на воркер-ноде `journalctl -k | grep calico-pack`;
- Запустили iptables-tailer для диагностики сетевой проблемы и добились того, чтобы iptables-tailer нормально запустился и писал лог;

### Выполнено задание со *
- Исправлена ошибка в сетевой политике, чтобы netperf заработал; 
- Исправлен манифест DaemonSet, чтобы показывались имена подов место ip адресов(?). 

### Как запустить: 
#### Предварительные действия: 
- Развернуть кластер (kind не подходит, нужен полноценный кластер минимум с 2 воркер нодами (минимум 2 ноды нужно для работы тестового приложения netperf-operator)); 
- Установить kubectl debug, согласно документации https://github.com/aylei/kubectl-debug
- Клонировать репозиторий netperf-operator - `git clone https://github.com/piontec/netperf-operator.git`
- Выполнить манифесты для запуска оператора в кластере: 
```
cd kubernetes-debug/netperf-operator
kubectl apply -f ./deploy/crd.yaml
kubectl apply -f ./deploy/rbac.yaml
kubectl apply -f ./deploy/operator.yaml

```

#### Основные действия: 
##### Задание 1 (проблема с контейнером-агентом):
При выполнении strace получаем ошибку вида: 

```
bash-5.0# strace -c -p1
strace: test_ptrace_get_syscall_info: PTRACE_TRACEME: Operation not permitted
strace: attach: ptrace(PTRACE_ATTACH, 1): Operation not permitted
```

Смотрим на наличие capabilities с использованием команды `docker inspect` (выполняем для запущенного debug контейнера на воркер-ноде):

```
            "CapAdd": null,
            "CapDrop": null,
``` 

Смотри добавляются ли необходимые capabilities в коде (по той ссылке что в подсказке): 
```
CapAdd:      strslice.StrSlice([]string{"SYS_PTRACE", "SYS_ADMIN"}),
```
- в коде все в порядке.

Смотрим версию образа агента: 

```
Normal  Pulling    16m   kubelet, node2     Pulling image "aylei/debug-agent:0.0.1"
```
- версия подозрительно старая. 

Смотри код в старой версии по ссылке https://github.com/aylei/kubectl-debug/blob/0.0.1/pkg/agent/runtime.go - а нужной строчки с capabilities там нет. 

Удаляем агента: 
`kubectl delete -f https://raw.githubusercontent.com/aylei/kubectl-debug/master/scripts/agent_daemonset.yml`

Устанавливаем последнюю версию агента, используя такой же манифест как и официальный, только с измененной версий образа (изменена на latest):
`kubectl apply -f kubernetes-debug/strace/01-agent_daemonset.yaml`

После этого еще раз проверяем strace и видим, что все успешно отрабатывает:

```
bash-5.0# strace -c -p1
strace: Process 1 attached
^Cstrace: Process 1 detached
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
  0.00    0.000000           0         2           poll
  0.00    0.000000           0         1           restart_syscall
------ ----------- ----------- --------- --------- ----------------
100.00    0.000000                     3           total
```

##### Задание 2 (диагностика сетевой проблемы):
Выполнить уже исправленные манифесты в следующей последовательности: 
- `kubectl apply -f kit/kit-clusterrole.yaml`;
- `kubectl apply -f kit/kit-serviceaccount.yaml`;
- `kubectl apply -f kit/kit-clusterrolebinding.yaml`;
- `kubectl apply -f kit/netperf-calico-policy.yaml`;
- `kubectl apply -f kit/iptables-tailer.yaml`;

Для запуска тестового приложения выполнить: 
- `kubectl apply -f netperf-operator/deploy/cr.yaml`;


##### Задание со `*`:
- Для исправления ошибки в сетевой политике необходимо исправить и применить манифест `kit/netperf-calico-policy.yaml`. Нужно строчки `selector: netperf-role == "netperf-client"` заменить на `selector: netperf-type in {"client", "server"}` (согласно документации https://docs.projectcalico.org/v3.7/reference/calicoctl/resources/globalnetworkpolicy#selector). И после этого запустить для проверки тест еще раз;
-  Возможно, судя по документации (https://github.com/box/kube-iptables-tailer), для того, чтобы отображалось имя пода, необходимо в DaemonSet изменить значение переменной окружения POD_IDENTIFIER с `label` на `name` (файл `kit/iptables-tailer.yaml`) и применить манифест, но я разницы не заметил. У меня в обоих случаях в поле `OBJECT` отображается имя пода, а в `MESSAGE` присутствует его ip адрес, делалось все на кластере GCE v1.14.  


<br><br>
## Домашнее задание 7
## Kubernetes operators
### Выполнено
- Создан CRD для MySQL:
  - Добавлена проверка наличия всех необходимых строчек в спецификации;
- Создали custom controller на python;
- Создали docker образ оператора mysql;
- Проверили работу оператора.

### Ответы на вопросы: 
- Вопрос: почему объект создался, хотя мы создали CR, до того, как запустили контроллер?
  - Ответ: Судя по документации (https://kopf.readthedocs.io/en/latest/walkthrough/starting/) оператор ищет уже созданный объект, т.е. по алгоритму работы - в начале применяется cr, а потом уже запускается оператор. 

### Как запустить: 
- Запустить minikube
- Установить python модули:
  - kopf (https://kopf.readthedocs.io/en/latest/install/)
  - kubernetes
  - jinja2
  - pyyaml

#### Ручной запуск (проверка работы оператора)
- В отдельной консоли запустить наш контроллер: `kopf run mysql-operator.py`
- В другом окне консоли применить манифест cr: `kubectl apply -f deploy/cr.yml`
- Записать что-нибуь в созданную базу данных, для этого выполним команды: 

```
export MYSQLPOD=$(kubectl get pods -l app=mysql-instance -o jsonpath="{.items[*].metadata.name}")

kubectl exec -it $MYSQLPOD -- mysql -u root -potuspassword -e "CREATE TABLE test ( id smallint unsigned not null auto_increment, name varchar(20) not null, constraint pk_example primary key (id) );" otus-database

kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "INSERT INTO test ( id, name ) VALUES ( null, 'some data' );" otus-database

kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "INSERT INTO test ( id, name ) VALUES ( null, 'some data-2' );" otus-database
```

- Проверить что данные записались в базу: 

```
kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database
```

- Для проверки, что работает восстановление из бекапа удалить mysql-instance `kubectl delete mysqls.otus.homework mysql-instance`

- Применить cr еще раз: `kubectl apply -f deploy/cr.yml`
- Проверить, что восстановление БД отработало:

```
export MYSQLPOD=$(kubectl get pods -l app=mysql-instance -o jsonpath="{.items[*].metadata.name}")

kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database
```

#### Со сборкой образа
- Собрать docker образ и запушить его на docker Hub (выполнять в директории `kubernetes-operators/build`):
  - `docker build -t s1spinoza/mysql-operator:v0.0.1 .` 
  - `docker push s1spinoza/mysql-operator:v0.0.1`
- Применить манифесты (находясь в директории `kubernetes-operators/deploy`): 
  - `kubectl apply -f service-account.yml`
  - `kubectl apply -f role.yml`
  - `kubectl apply -f role-binding.yml`
  - `kubectl apply -f deploy-operator.yml`
- Применить манифест cr: `kubectl apply -f cr.yml`
- Далее добавить в базу записи и проверить восстановление из бекапа (как расписано выше в `Ручной запуск`)


#### Необходимые выводы команд: 

```
# kubectl get jobs
NAME                         COMPLETIONS   DURATION   AGE
backup-mysql-instance-job    1/1           3s         6m43s
restore-mysql-instance-job   1/1           42s        5m57s
```

```
# export MYSQLPOD=$(kubectl get pods -l app=mysql-instance -o jsonpath="{.items[*].metadata.name}")
# kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database
mysql: [Warning] Using a password on the command line interface can be insecure.
+----+-------------+
| id | name        |
+----+-------------+
|  1 | some data   |
|  2 | some data-2 |
+----+-------------+
```


