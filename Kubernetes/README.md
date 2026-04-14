# Kubernetes
Для выполнения работы испотзовала `Ubuntu Linux` на гипервизоре `VMware`

## Задание
```
1 часть
Поднять kubernetes кластер локально (например minikube), в нём развернуть свой сервис, используя 2-3 ресурса kubernetes. В идеале разворачивать кодом из yaml файлов одной командой запуска. Показать работоспособность сервиса.
(сервис любой из своих не опенсорсных, вывод “hello world” в браузер тоже подойдёт)

Прочитать шикарную книжку про жирафа: Kubernetes_for_Kids_ITSummaPress (1).pdf

2 часть

1. Создать helm chart на основе обычной 3 лабы
2. Задеплоить его в кластер
3. Поменять что-то в сервисе, задеплоить новую версию при помощи апгрейда релиза
4. В отчете приложить скрины всего процесса, все использованные файлы, а также привести три причины, по которым использовать хелм удобнее чем классический деплой через кубернетес манифесты
```

## Часть 1
Для начала выбрала путь `minikube` и устанавливала `Kubernetes` через него
<img width="2774" height="1140" alt="image" src="https://github.com/user-attachments/assets/e2218e32-f406-4de4-8baa-4861ec6cbe66" />
<img width="2806" height="1250" alt="image" src="https://github.com/user-attachments/assets/8151b0dc-90f8-4b65-a945-754a25c27ebb" />

установила `Lens K8N`, авторизовалась и далее уже открыла IDE, в котором были демонстрационный и minikube кластеры. 
<img width="692" height="280" alt="image" src="https://github.com/user-attachments/assets/0a18c94b-ba83-4427-bd04-1d458878ffb0" />

т.к. не было возможности много работать, а с терминалом Lens возникли проблемы и дедлайн близко, решила делать в обычном терминале. НО! отсюда личное todo: сделать все в Lens
<img width="1838" height="816" alt="image" src="https://github.com/user-attachments/assets/93f7ebe7-e698-4d99-a1e1-47f3c6e411b3" />

Выбрано было мини-приложение
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return '<h1>Hello, Kubernetes!</h1><p>Student Regatta Service is running.</p>'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)
```
т.к. в моем приложении код Python, то мне необходимо было написать Dockerfile и собрать образ
<img width="1842" height="874" alt="image" src="https://github.com/user-attachments/assets/5c93be20-fb4c-4a1d-be98-436209270a2c" />


создала следующий .yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-web-app
  template:
    metadata:
      labels:
        app: my-web-app
    spec:
      containers:
      - name: web-container
        image: my-web-app:v2
        imagePullPolicy: Never
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  type: NodePort
  selector:
    app: my-web-app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

Запускаю и проверяю работу
<img width="1992" height="724" alt="image" src="https://github.com/user-attachments/assets/ddd6e0c8-ecf0-4a8e-ba43-c9a22e793e46" />

<img width="2160" height="666" alt="image" src="https://github.com/user-attachments/assets/f000d7a2-bb01-4bd1-ad34-0ecd69d63253" />
Приложение действительно заработало!

## Часть 2 - Helm
Сначала установила Helm
<img width="2026" height="612" alt="image" src="https://github.com/user-attachments/assets/16b33e56-7aee-4f92-bfc3-4a28df585fc1" />
изменила в файлах шаблонов следующие строки

values.yaml
```
image:
  repository: my-web-app
  tag: v2
  pullPolicy: Never

service:
  type: NodePort
  port: 80
```
deployment.yaml
```
containers:
  - name: web-container
    image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
    imagePullPolicy: {{ .Values.image.pullPolicy }}
    ports:
      - containerPort: 80
```
service.yaml
```
spec:
  type: {{ .Values.service.type }}
  selector:
    app: my-web-app
  ports:
    - port: {{ .Values.service.port }}
      targetPort: 80
```
Пробовала также добавлять `nodePort`, однако назначила им тот же порт, что и предыдущей сборке, из-за чего возникли проблемы. Потом я просто убрала этот аргумент, т.к. kubernetes может и сам выдавать порты, что мы и видим дальше.

и в итоге запустила helm
<img width="2550" height="728" alt="image" src="https://github.com/user-attachments/assets/107231ab-4def-4f4a-ac0a-09b361ee88be" />

И проверила. Остались сервисы и поды из части 1, но те, над которыми работала в части 2 работают и открываются по своим портам
<img width="2478" height="1500" alt="image" src="https://github.com/user-attachments/assets/073f7dc3-f09e-4bc1-b558-a50502fb5f1c" />

Далее изменила приложение и сделала апгрейд релиза
<img width="2650" height="912" alt="image" src="https://github.com/user-attachments/assets/c9cb899f-ad7e-42c2-b9e6-21bb7c2e5d61" />
изменила тэг в values.yaml
<img width="1012" height="332" alt="image" src="https://github.com/user-attachments/assets/02629f44-bb9e-4d6e-9d35-ddcc13eb5bf9" />

Но, к сожалению, после апдейта меня вело не на новое прикольное обновление, а на старый грустный(
Сначала чекнула докер образ, мало ли что, но с ним все ок, значит проблема точно не в нем.
<img width="2538" height="880" alt="image" src="https://github.com/user-attachments/assets/4a7b50ae-26ad-490b-a276-ae87c310ba24" />

Дальше все пошло капитально не так. Я удалила один из деплоев, в надежде, что он забрал на себя метку `app: my-web-app` и весь трафик шел на него, но, видимо, удалила не тот, и у меня перестал открываться локалхост. В итоге, я решила все удалить и запустить хелм заново, и на него уже потом делать апгрейд. Очистила все в 0
<img width="2412" height="1412" alt="image" src="https://github.com/user-attachments/assets/09e5825a-89b4-4d0d-aeed-6e086f7e81b0" />

запустила новый хелм, но эндпоинты у него не находятся. посмотрела метки пода и сервисов и они не совпадают
<img width="2736" height="510" alt="image" src="https://github.com/user-attachments/assets/6e7d0276-7327-4bc6-b6ec-3e82cdec5edb" />

о чудо, эндпоинт починился
<img width="2692" height="322" alt="image" src="https://github.com/user-attachments/assets/1e9ad195-2cfc-4802-a2d9-086187e88f04" />

базовый инсталл хотя бы смог починиться
<img width="2726" height="1374" alt="image" src="https://github.com/user-attachments/assets/5e8f660e-2bc7-4680-8a6a-308a3f9f1250" />

теперь опять к апдейту. я решила, что все страдания из-за злобного смеха, поэтому теперь фразу сделаю помягче и рассудительнее
ура победа!
<img width="2374" height="1692" alt="image" src="https://github.com/user-attachments/assets/9ffcf520-4687-49b3-ada7-f1e69e531f4b" />
а нужно было в service всего лишь поменять селектор на `{{- include "my-app-chart.selectorLabels" . | nindent 4 }}`, такой же, как в deployment, т.к. он создает метки автоматом, а так получалось, что метка фиксированная

Итого 
## ТОП ПРИЧИН ИСПОЛЬЗОВАТЬ HELM

1. Буквально моя проблема решилась грамотным helm подходом: шаблонизация и переиспользование. С правильным подходом код можно будет использовать многократно, т.к. будут автоматически даваться правильные метки
2. Жизненный цикл: я смогла достаточно легко удалить все процессы, т.к. в helm, в отличие от манифестов все связано
3. Автоматизация связей и зависимостей. Это был ужас все вручную делать, а тут автоматом с helm

## Выводы
В рамках лабораторной я познакомалась с технологией оркестрации `Kubernetes`, из чего состоит кластер, `Helm`, а также подготовкой манифестов, helm chart и апгрейдами. Лабораторная показала, что автоматизация процессов - действительно важный и нужный процесс, который поможет избавиться от человеческого фактора и упростит всем жизнь.

Картинка с нашей регаты (ну каков парус!). Теперь буду учиться не только рулить на яхте, но и рулить кластером!
<img width="2548" height="1706" alt="image" src="https://github.com/user-attachments/assets/e7a1b624-e953-49f8-8788-9ea6c30328c5" />

