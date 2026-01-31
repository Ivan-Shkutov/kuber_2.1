## Домашнее задание к занятию «Хранение в K8s»

Цель задания

Научиться работать с хранилищами в тестовой среде Kubernetes:

- обеспечить обмен файлами между контейнерами пода;

- создавать PersistentVolume (PV) и использовать его в подах через PersistentVolumeClaim (PVC);

- объявлять свой StorageClass (SC) и монтировать его в под через PVC.

Это задание поможет вам освоить базовые принципы взаимодействия с хранилищами в Kubernetes — одного из ключевых навыков для работы с кластерами. На практике Volume, PV, PVC используются для хранения данных независимо от пода, обмена данными между подами и контейнерами внутри пода. Понимание этих механизмов поможет вам упростить проектирование слоя данных для приложений, разворачиваемых в кластере k8s.

Подготовка

1. Чеклист готовности

2. Установленное K8s-решение (допустим, MicroK8S).

3. Установленный локальный kubectl.

4. Редактор YAML-файлов с подключенным GitHub-репозиторием.

### Задание 1. Volume: обмен данными между контейнерами в поде

Задача

Создать Deployment приложения, состоящего из двух контейнеров, обменивающихся данными.

Шаги выполнения

- Создать Deployment приложения, состоящего из контейнеров busybox и multitool.

- Настроить busybox на запись данных каждые 5 секунд в некий файл в общей директории.

- Обеспечить возможность чтения файла контейнером multitool.

Что сдать на проверку

Манифесты:

1. containers-data-exchange.yaml

Скриншоты:

1. описание пода с контейнерами (kubectl describe pods data-exchange)

2. вывод команды чтения файла (tail -f <имя общего файла>)

- - - - -
### Решение:

В Kubernetes контейнеры внутри одного Pod:

- работают на одной сети

- могут использовать общее хранилище

Для обмена данными внутри одного Pod лучше всего подходит emptyDir:

- создаётся при старте Pod

- существует, пока жив Pod

- удаляется вместе с Pod

- доступен всем контейнерам в Pod

### Что нужно выполнить:

1. создать Deployment с 1 репликой

2. добавить volume типа emptyDir

3. смонтировать его в оба контейнера в /shared

4. в контейнере busybox запустить бесконечный цикл
  
   - while true; do date >> /shared/data.txt; sleep 5; done

5. во втором контейнере запустить

  - tail -f /shared/data.txt

![1](https://github.com/Ivan-Shkutov/kuber_2.1/blob/main/1.png)

![2](https://github.com/Ivan-Shkutov/kuber_2.1/blob/main/2.png)

![3](https://github.com/Ivan-Shkutov/kuber_2.1/blob/main/3.png)

![4](https://github.com/Ivan-Shkutov/kuber_2.1/blob/main/4.png)

- - - - -

### Задание 2. PV, PVC

Задача

Создать Deployment приложения, использующего локальный PV, созданный вручную.

Шаги выполнения

- Создать Deployment приложения, состоящего из контейнеров busybox и multitool, использующего созданный ранее PVC

- Создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.

- Продемонстрировать, что контейнер multitool может читать данные из файла в смонтированной директории, в который busybox записывает данные каждые 5 секунд.

- Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему. (Используйте команду kubectl describe pv).

- Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV. Продемонстрировать, что произошло с файлом после удаления PV. Пояснить, почему.

Что сдать на проверку

Манифесты:

1. pv-pvc.yaml

Скриншоты:

1. каждый шаг выполнения задания, начиная с шага 2.

Описания:

1. объяснение наблюдаемого поведения ресурсов в двух последних шагах.

- - - - -
### Решение:

Создаем папку потому что для hostPath PV Kubernetes использует физический путь на ноде

  - mkdir -p /home/vm/Templates/K8S/2.1/k8s-data

Применяем манифест

  - kubectl apply -f pv-pvc.yaml

Проверяем PV и PVC

  - kubectl get pv

  - kubectl get pvc

Проверяем Pod

  - kubectl get pods

Проверяем обмен данными контейнером multitool-reader

  - kubectl exec -it data-exchange-pvc-xxxxx -c multitool-reader -- tail -f /mnt/data/data.txt

Удаляем Deployment и PVC

  - kubectl delete deployment data-exchange-pvc
    
  - kubectl delete pvc local-pvc

Проверяем PV

  - kubectl describe pv local-pv

Проверяем файл на диске

  - ls -l /home/vm/Templates/K8S/2.1/k8s-data
    

![5](https://github.com/Ivan-Shkutov/kuber_2.1/blob/main/5.png)

![6](https://github.com/Ivan-Shkutov/kuber_2.1/blob/main/6.png)

![7](https://github.com/Ivan-Shkutov/kuber_2.1/blob/main/7.png)

- - - - -
### Задание 3. StorageClass

Задача

Создать Deployment приложения, использующего PVC, созданный на основе StorageClass.

Шаги выполнения

- Создать Deployment приложения, состоящего из контейнеров busybox и multitool, использующего созданный ранее PVC.

- Создать SC и PVC для подключения папки на локальной ноде, которая будет использована в поде.

- Продемонстрировать, что контейнер multitool может читать данные из файла в смонтированной директории, в который busybox записывает данные каждые 5 секунд.

Что сдать на проверку

Манифесты:

1. sc.yaml

Скриншоты:

1. каждый шаг выполнения задания, начиная с шага 2

2. Шаблоны манифестов с учебными комментариями

- - - - -
### Решение:

Создаем папку потому что для hostPath PV Kubernetes использует физический путь на ноде

  - mkdir -p /home/vm/Templates/K8S/2.1/sc-data

Создание StorageClass, PV и PVC

  - StorageClass – задаёт правила для PVC и связывает его с PV вручную

  - PersistentVolume (PV) – физическое хранилище на ноде

  - PersistentVolumeClaim (PVC) – запрос Pod на использование этого PV

Deployment с двумя контейнерами

  - busybox-writer пишет дату каждые 5 секунд в файл /mnt/data/data.txt

  - multitool-reader читает этот файл в реальном времени

  - общий volume /mnt/data обеспечивается через PVC → PV → hostPath

Применение манифеста

  - kubectl apply -f sc.yaml

Проверяем статус

  - kubectl get pods

  - kubectl get pvc

  - kubectl get pv

Проверяем обмен данными

  - kubectl exec -it data-exchange-sc-6787845698-l6j87 -c multitool-reader -- tail -f /mnt/data/data.txt

#### Пояснение задания

1. Контейнеры обмениваются данными через PVC, который ссылается на PV, физически расположенный в hostPath на ноде

2. StorageClass нужен для связи PVC и PV и управления политикой привязки

3. Данные остаются на локальной ноде, даже если Pod удалён, благодаря Retain


![8](https://github.com/Ivan-Shkutov/kuber_2.1/blob/main/8.png)

![9](https://github.com/Ivan-Shkutov/kuber_2.1/blob/main/9.png)

![10](https://github.com/Ivan-Shkutov/kuber_2.1/blob/main/10.png)

- - - - -
1. Deployment (containers-data-exchange.yaml)

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: data-exchange
spec:
  replicas: # ЗАДАНИЕ: Укажите количество реплик
  selector:
    matchLabels:
      app: # ДОПОЛНИТЕ: Метка для селектора
  template:
    metadata:
      labels:
        app: # ПОВТОРИТЕ: Метка из selector.matchLabels
    spec:
      containers:
      - name: # ДОПОЛНИТЕ: Имя первого контейнера
        image: busybox
        command: ["/bin/sh", "-c"] 
        args: ["echo $(date) > путь_к_файлу; sleep 3600"] # КЛЮЧЕВОЕ: Команда записи данных в файл в директории из секции volumeMounts контейнера
        volumeMounts:
        - name: # ДОПОЛНИТЕ: Имя монтируемого раздела. Должно совпадать с именем эфемерного хранилища, объявленного на уровне пода.
          mountPath: # КЛЮЧЕВОЕ: Путь монтирования эфемерного хранилища внутри контейнера 1
      - name: # ДОПОЛНИТЕ: Имя второго контейнера
        image: busybox
        command: ["/bin/sh", "-c"]
        args: ["tail -f путь_к_файлу"] # КЛЮЧЕВОЕ: Команда для чтения данных из файла, расположенного в директории, указанной в volumeMounts контейнера
        volumeMounts:
        - name: # ДОПОЛНИТЕ: Имя монтируемого раздела. Должно совпадать с именем эфемерного хранилища, объявленного на уровне пода
          mountPath: # КЛЮЧЕВОЕ: Путь монтирования эфемерного хранилища внутри контейнера 2
      volumes:
      - name: # ДОПОЛНИТЕ: Имя монтируемого раздела эфемерного хранилища
        emptyDir: {} # ИНФОРМАЦИЯ: Определяем эфемерное хранилище, которое работает только внутри пода
```

2. Deployment (pv-pvc.yaml)

```
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: # ДОПОЛНИТЕ: Имя хранилища
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: # КЛЮЧЕВОЕ: Путь к директории на ноде (хосте, на котором развёрнут кластер)
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: # ДОПОЛНИТЕ: Имя PVC
spec:
  volumeName: # ДОПОЛНИТЕ: Имя PV, к которому будет привязан PVC, должен совпадать с созданным ранее PV
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: # ДОПОЛНИТЕ: Какой объём хранилища вы хотите передать в контейнер. Должно быть меньше или равно параметру storage из PV
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: data-exchange-pvc
spec:
  replicas: # ЗАДАНИЕ: Укажите количество реплик
  selector:
    matchLabels:
      app: # ДОПОЛНИТЕ: Метка для селектора
  template:
    metadata:
      labels:
        app: # ПОВТОРИТЕ: Метка из selector.matchLabels
    spec:
      containers:
      - name: # ДОПОЛНИТЕ: Имя первого контейнера
        image: busybox
        command: ["/bin/sh", "-c"] 
        args: ["echo $(date) > путь_к_файлу; sleep 3600"] # КЛЮЧЕВОЕ: Команда записи данных в файл в директории из секции volumeMounts контейнера 
        volumeMounts:
        - name: # ДОПОЛНИТЕ: Имя монтируемого раздела. Должно совпадать с именем хранилища, объявленного на уровне пода
          mountPath: # КЛЮЧЕВОЕ: Путь монтирования хранилища внутри контейнера 1
      - name: # ДОПОЛНИТЕ: Имя второго контейнера
        image: busybox
        command: ["/bin/sh", "-c"]
        args: ["tail -f путь_к_файлу"] # КЛЮЧЕВОЕ: Команда для чтения данных из файла, расположенного в директории, указанной в volumeMounts контейнера
        volumeMounts:
        - name: # ДОПОЛНИТЕ: Имя монтируемого раздела. Должно совпадать с именем хранилища, объявленного на уровне пода
          mountPath: # КЛЮЧЕВОЕ: Путь монтирования хранилища внутри контейнера 2
      volumes:
      - name: # ДОПОЛНИТЕ: Имя монтируемого раздела хранилища
        persistentVolumeClaim:
          claimName: # КЛЮЧЕВОЕ: Совпадает с именем PVC объявленного ранее
```
3. Deployment (sc.yaml)

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: # ДОПОЛНИТЕ: Имя StorageClass
provisioner: kubernetes.io/no-provisioner # ИНФОРМАЦИЯ: Нет автоматического развёртывания
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: # ДОПОЛНИТЕ: Имя PVC
spec:
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: # ДОПОЛНИТЕ: Какой объем хранилища вы хотите передать в контейнер. Должно быть меньше или равно параметру storage из PV
  storageClassName: # ДОПОЛНИТЕ: Имя StorageClass. Должно совпадать с объявленным ранее
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: data-exchange-sc
spec:
  replicas: # ЗАДАНИЕ: Укажите количество реплик
  selector:
    matchLabels:
      app: # ДОПОЛНИТЕ: Метка для селектора
  template:
    metadata:
      labels:
        app: # ПОВТОРИТЕ: Метка из selector.matchLabels
    spec:
      containers:
      - name: # ДОПОЛНИТЕ: Имя первого контейнера
        image: busybox
        command: ["/bin/sh", "-c"] 
        args: ["echo $(date) > путь_к_файлу; sleep 3600"] # КЛЮЧЕВОЕ: Команда для чтения данных из файла, расположенного в директории, указанной в volumeMounts контейнера
        volumeMounts:
        - name: # ДОПОЛНИТЕ: Имя монтируемого раздела. Должно совпадать с именем хранилища, объявленного на уровне пода
          mountPath: # КЛЮЧЕВОЕ: Путь монтирования хранилища внутри контейнера 1
      - name: # ДОПОЛНИТЕ: Имя второго контейнера
        image: busybox
        command: ["/bin/sh", "-c"]
        args: ["tail -f путь_к_файлу"] # КЛЮЧЕВОЕ: Команда для чтения данных из файла, расположенного в директории, указанной в volumeMounts контейнера
        volumeMounts:
        - name: # ДОПОЛНИТЕ: Имя монтируемого раздела. Должно совпадать с именем хранилища, объявленного на уровне пода
          mountPath: # КЛЮЧЕВОЕ: Путь монтирования хранилища внутри контейнера 2
      volumes:
      - name: # ДОПОЛНИТЕ: Имя монтируемого раздела хранилища
        persistentVolumeClaim:
          claimName: # КЛЮЧЕВОЕ: Совпадает с именем PVC объявленного ранее
```
