# Развёртывание OpenFaaS на Kubernetes и деплой демо-функции

## Описание проекта

### Цель проекта

Развертывание OpenFaaS на кластере Kubernetes для создания масштабируемой среды выполнения serverless-функций. Основной фокус — тестирование и измерение задержек выполнения функций, что критично для систем реального времени и промышленной автоматизации.

### Ключевые задачи

- Развертывание OpenFaaS в Kybernetes-кластере

- Создание демо-функция для измерения задержек
  - Создание функции, которая:
    - Фиксирует время начала и конца выполнения
    - Возвращает длительность работы (в миллисекундах)
  - Деплой функции на платформу OpenFaaS

- Интеграция с ... (in progress)

## Установка и настройка

1. Получить config-файл для подключения к кластеру k8s, проверить подключение к кластеру:

   ```kubectl cluster-info```

2. Установить OpenFaaS на кластер:

   ```arkade install openfaas```

3. Получить пароль для доступа:

   ```kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode```

4. Установить faas-cli:

   ```arkade get faas-cli```

5. Авторизоваться в faas-cli:

   ```faas-cli login -u admin --password _пароль_ --gateway  http://ip_кластера:порт```

   Ip-кластера можно узнать из config-файла, а порт узнать с помощью команды:

   ```kubectl get svc -n openfaas```

   Необходим порт сервиса gateway-external, который по умолчанию задан NodePort.

6. Авторизоваться в Web-UI OpenFaaS, для этого необходимо открыть в браузере: http://ip_кластера:порт. Войти под логином admin, а пароль ввести из пункта 3.

## Деплой демо-функции

Для деплоя демо-функции можно использовать уже написанный config-файл (stack.yaml) и написать в консоли: ```faas-cli deploy -f stack.yaml```. Либо можно создать собственную демо-функцию:

1. Вывести список готовых шаблонов для openfaas:

   ```faas-cli template store list```

2. Скачать шаблон(\ы):

   ```faas-cli template store pull```

3. Создать функции с выбранным шаблоном:

   ```faas-cli new my-fun --lang lang123```

4. Необходимо изменить файлы шаблона под функцию, обычно достаточно изменить файлы: handler (код), requirements, config-файл.

5. Необходимо собрать образ, запушить в dockerhub и задеплоить функцию:

   ```faas-cli build -f stack.yaml```
   ```docker push <user_name>/repo:tag```
   ```faas-cli deploy -f stack.yaml```
  
   **Примечание**: Для Community Edition Open Fas необходимо, чтобы Docker-образ был доступен в публичном реестре (например, Docker Hub).

## Интеграция с ... (in progress)


## Дополнительные материалы
[![OpenFaaS](https://img.shields.io/badge/OpenFaaS-Official-blue)](https://github.com/openfaas/faas-netes)  
[![Kubernetes](https://img.shields.io/badge/Kubernetes-Tutorials-326CE5)](https://kubernetes.io/ru/docs/tutorials/)  
[![Docker](https://img.shields.io/badge/Docker-Get%20Started-2496ED)](https://docs.docker.com/get-started/)
