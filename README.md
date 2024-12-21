**Задание 1. Выбрать стратегию обновления приложения и описать ваш выбор**

1. Имеется приложение, состоящее из нескольких реплик, которое требуется обновить.

2. Ресурсы, выделенные для приложения, ограничены, и нет возможности их увеличить.

3. Запас по ресурсам в менее загруженный момент времени составляет 20%.

4. Обновление мажорное, новые версии приложения не умеют работать со старыми.

5. Вам нужно объяснить свой выбор стратегии обновления приложения.



**Решение 1**

*Вариант 1.*

Если приложение уже протестировано ранее, то лучшим вариантом наверное будет использовать стратегию обновления Rolling Update, указанием параметров maxSurge maxUnavailable для избежания ситуации с нехваткой ресурсов. Проводить обновление следует естественно в менее загруженный момент времени сервиса. При данной стратегии(Rolling Update) k8s постепенно заменит все поды без ущерба производительности. И если что-то пойдет не так, можно будет быстро откатится к предыдущему состоянию.


*Вариант 2.*

Можно использовать Canary Strategy. Также указав параметры maxSurge maxUnavailable чтобы избежать нехватки ресурсов. Это позволит нам протестировать новую версию программы на реальной пользовательской базе(группа может выделяться по определенному признаку) без обязательства полного развертывания. После тестирования и собирания метрик пользователей можно постепенно переводить поды к новой версии приложения



**Задание 2. Обновить приложение**

1. Создать deployment приложения с контейнерами nginx и multitool. Версию nginx взять 1.19. Количество реплик — 5.

2. Обновить версию nginx в приложении до версии 1.20, сократив время обновления до минимума. Приложение должно быть доступно.

3. Попытаться обновить nginx до версии 1.28, приложение должно оставаться доступным.

4. Откатиться после неудачного обновления.




**Решение 2**

Создадим deployment приложения с контейнерами nginx и multitool. Версию nginx возбмем 1.19. Количество реплик — 5.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-multitool
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 3
  selector:
    matchLabels:
      app: nginx-multitool
  template:
    metadata:
      labels:
        app: nginx-multitool
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        env:
        - name: HTTP_PORT
          value: "8080"
        ports:
        - containerPort: 8080
```

![Image alt](https://github.com/mezhibo/kubernetes14/blob/a24ebbf089f94b5c5509bcd6c42aa1123d78a1ab/IMG/1.jpg)


Обновиv версию nginx в приложении до версии 1.20, сократив время обновления до минимума

Срздадим вот такой деплоймент

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-multitool
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 3
  selector:
    matchLabels:
      app: nginx-multitool
  template:
    metadata:
      labels:
        app: nginx-multitool
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        env:
        - name: HTTP_PORT
          value: "8080"
        ports:
        - containerPort: 8080
```

Обновляемся до верси 1.20 и видим что все окей

![Image alt](https://github.com/mezhibo/kubernetes14/blob/a24ebbf089f94b5c5509bcd6c42aa1123d78a1ab/IMG/2.jpg)


Теперь пробуем обновиться до версии 1.28 

Создадим деплоймент

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-multitool
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 3
  selector:
    matchLabels:
      app: nginx-multitool
  template:
    metadata:
      labels:
        app: nginx-multitool
    spec:
      containers:
      - name: nginx
        image: nginx:1.28
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        env:
        - name: HTTP_PORT
          value: "8080"
        ports:
        - containerPort: 8080
```

Видим что приложение не смогло обновиться и поды попадали с ошибкой


![Image alt](https://github.com/mezhibo/kubernetes14/blob/a24ebbf089f94b5c5509bcd6c42aa1123d78a1ab/IMG/3.jpg)

Сделаем откат обратно

![Image alt](https://github.com/mezhibo/kubernetes14/blob/a24ebbf089f94b5c5509bcd6c42aa1123d78a1ab/IMG/4.jpg)

Видим что поды снова заработали
