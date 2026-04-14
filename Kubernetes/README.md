# Kubernetes
Для выполнения работы испотзовала `Ubuntu Linux` на гипервизоре `VMware`
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

## Часть 2
