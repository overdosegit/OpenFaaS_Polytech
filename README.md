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

- Интеграция с PLC
  - Используйте node-red для подключения к ПЛК и монитору
  - Отправьте http-запрос при изменении статуса узла и запишите время отправки
  - Запишите время получения и значение возврата ответа и рассчитайте разницу

## Установка и настройка

1. Получите config-файл для подключения к кластеру k8s, проверьте подключение к кластеру:

   ```kubectl cluster-info```

2. Установите OpenFaaS на кластер:

   ```arkade install openfaas```

3. Получите пароль для доступа:

   ```kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode```

4. Установите faas-cli:

   ```arkade get faas-cli```

5. Авторизуйтесь в faas-cli:

   ```faas-cli login -u admin --password _пароль_ --gateway  http://ip_кластера:порт```

   Ip-кластера можно узнать из config-файла, а порт узнать с помощью команды:

   ```kubectl get svc -n openfaas```

   Необходим порт сервиса gateway-external, который по умолчанию задан NodePort.

6. Авторизуйтесь в Web-UI OpenFaaS, для этого необходимо открыть в браузере: http://ip_кластера:порт. Войти можно под логином admin, а пароль ввести из пункта 3.

## Деплой демо-функции

Для деплоя демо-функции можно использовать уже написанный config-файл (stack.yaml) и написать в консоли: ```faas-cli deploy -f stack.yaml```. Либо можно создать собственную демо-функцию:

1. Выведите список готовых шаблонов для openfaas:

   ```faas-cli template store list```

2. Скачайте шаблоны:

   ```faas-cli template store pull```

3. Создайте функцию с выбранным шаблоном:

   ```faas-cli new my-fun --lang lang123```

4. Необходимо изменить файлы шаблона под функцию, обычно достаточно изменить файлы: handler, requirements, config-файл.

5. Необходимо собрать образ, запушить в dockerhub и задеплоить функцию:

   ```faas-cli build -f stack.yaml```
   
   ```docker push <user_name>/repo:tag```
   
   ```faas-cli deploy -f stack.yaml```
  
**Примечание**: Для Community Edition OpenFaas необходимо, чтобы Docker-образ был доступен в публичном реестре (например, Docker Hub).

## Интеграция с PLC

UAExpert можно использовать для чтения и записи данных ПЛК

1. Установить node-red

   ```sudo npm install -g --unsafe-perm node-red```

2. Запустите node-red, оставьте терминал работающим
   
   ```node-red```

3. Установите модуль node-red-contrib-opcua в менеджере узлов

4. Установить объект мониторинга через узел OpcUa Item
   - Пример Item: ns=4;i=34. Пример Type: Boolean

5. Подключение к ПЛК через узел OpcUa-Client
   - Введите адрес сервера OPC UA в Endpoint
   - Остальные могут оставаться по умолчанию

6. Используйте узлы function для определения изменений данных узлов

   ```
   context.lastValue = context.lastValue || false;
   if (msg.payload !== context.lastValue) {
       context.lastValue = msg.payload; 
       return msg; 
   } else {
       return null; 
   }
   ```

7. Используйте узел change, чтобы прочитать время
   - Выберите тип выражения

   ```$now()```

8. Используйте узел function для преобразования времени в миллисекунды

   ```
   if (typeof msg.payload === "string") {
     msg.payload = Date.parse(msg.payload);
   } else {
   node.warn("Payload 不是字符串！当前类型: " + typeof msg.payload);
   }
   return msg;
   ```

9. Перед выполнением вычитания отредактируйте заголовки каждого времени. Использование узла change
   - Выберите msg. Введите topic
   - Значение — пользовательский текст

10. Слияние три раза через узел join
    - Выбор режима: Ручной
    - Проверять Use existing msg.parts property
    - В поле «При достижении определенного объема информации» введите 3
    - ！Снимите флажок «И каждое последующее сообщение»

11. Использование function узлов для вычитания

    ```
    const start = msg.payload.startTime;
    const end = msg.payload.endTime;
    const duration = Math.abs(end - start);
    return {
        payload: duration,               
        durationFormatted: `${duration}ms` 
    };
    ```

[Click to view network_latency.json file](./network_latency.json)
## Удаление OpenFaaS с кластера Kubernetes

__Рекомендуется__ перед удалением OpenFaaS:

1. Проверить развернутые функции:

   ```faas-cli list```

2. Убедитесь, что у вас есть конфигурационные файлы для деплоя функций

Для __полного удаления__ OpenFaaS из Kubernetes-кластера выполните следующие шаги:

1. Удалите основные компоненты OpenFaaS с помощью Helm:

   ```helm uninstall -n openfaas openfaas```

2. Удалите namespaces OpenFaaS:

   ```kubectl delete namespace openfaas openfaas-fn```

3. Удалите CRDs OpenFaaS:

   ```kubectl delete crd -l app.kubernetes.io/name=openfaas```

## Результаты проекта

## Дополнительные материалы
[![OpenFaaS](https://img.shields.io/badge/OpenFaaS-Official-blue)](https://github.com/openfaas/faas-netes)  
[![Kubernetes](https://img.shields.io/badge/Kubernetes-Tutorials-326CE5)](https://kubernetes.io/ru/docs/tutorials/)  
[![Docker](https://img.shields.io/badge/Docker-Get%20Started-2496ED)](https://docs.docker.com/get-started/)  
[![OpenFaaS on ArtifactHub](https://img.shields.io/badge/OpenFaaS-ArtifactHub-blue)](https://artifacthub.io/packages/helm/openfaas/openfaas)
