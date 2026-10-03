# Інструкція з тестування програми Django ToDo в Kubernetes

Перед початком переконайтеся, що всі ресурси розгорнуті в просторі імен `todoapp`. Зверніть увагу, що обидва поди розгортаються з одного спільного файлу маніфесту:

```bash
kubectl apply -f namespace.yml
kubectl apply -f todoapp-pod.yml
kubectl apply -f service-clusterip.yml
kubectl apply -f service-nodeport.yml
kubectl apply -f busybox.yml
```

## 1. Тестування ClusterIP через DNS з контейнера BusyBox
Сервіс `ClusterIP` створює стабільну внутрішню точку входу та балансує трафік між обома подами (`todoapp-1` та `todoapp-2`), які мають мітку `app: todolist`.

1. Зайдіть всередину тестового контейнера busybox:
   ```bash
   kubectl exec -it busybox -n todoapp -- sh
   ```
2. Виконайте HTTP-запит до сервісу за його внутрішнім DNS-іменем:
   ```bash
   curl http://todoapp-clusterip/api/health/
   ```
   *Очікувана відповідь:* `Health OK`. При повторних запитах Kubernetes автоматично розподілятиме навантаження між двома запущеними подами.

## 2. Тестування за допомогою команди port-forward
Команда `port-forward` дозволяє тимчасово прокинути порт із кластера на вашу локальну машину для швидкого доступу через браузер.

1. Виконайте команду перенаправлення порту для сервісу ClusterIP:
   ```bash
   kubectl port-forward svc/todoapp-clusterip 8000:80 -n todoapp
   ```
2. Відкрийте браузер на своєму комп'ютері або виконайте команду `curl` у новому терміналі:
   ```bash
   curl http://localhost:8000/api/health/
   ```

## 3. Доступ до застосунку через NodePort Service
Сервіс `NodePort` відкриває фіксований порт на рівні самої Kubernetes ноди (вузла), роблячи застосунок доступним ззовні кластера.

1. Дізнайтеся IP-адресу вашої Kubernetes ноди (якщо ви використовуєте Minikube, виконайте команду `minikube ip` у терміналі вашого хоста).
2. Надішліть запит на отриману IP-адресу та зовнішній порт `30080`, який ми зафіксували в маніфесті:
   ```bash
   curl http://<NODE_IP>:30080/api/health/
   ```
