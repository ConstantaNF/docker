# ****Docker**** #

### Описание домашннего задания ###

* Установите Docker на хост машину
  <https://docs.docker.com/engine/install/ubuntu/>
* Установите Docker Compose - как плагин, или как отдельное приложение
* Создайте свой кастомный образ nginx на базе alpine. После запуска nginx должен отдавать кастомную страницу (достаточно изменить дефолтную страницу nginx)
* Определите разницу между контейнером и образом. Вывод опишите в домашнем задании.
* Ответьте на вопрос: Можно ли в контейнере собрать ядро?

### Критерии оценивания ###

* Создан свой кастомный образ nginx на базе alpine. После запуска nginx должен отдавать кастомную страницу (достаточно изменить дефолтную страницу nginx).
* Определена разница между контейнером и образом, написан вывод.
* Написан ответ на вопрос: Можно ли в контейнере собрать ядро?
* Собранный образ запушин в docker hub и дана ссылку на репозиторий.

### Выполнение ###

С помощью Vagrant разворачиваю Ubuntu Focal64 v20240513.0.0 . Устанавливаю Docker по инструкции <https://docs.docker.com/engine/install/ubuntu/> . Также устанавливаю Docker Compose в виде плагина <https://docs.docker.com/compose/install/linux/>
Для создания кастомного образа Nginx на базе Alpine сначала нужен оригинальный образ. Для этого ищем его в официальном репозитории на docker hub <https://hub.docker.com/_/nginx> . Крайняя версия <https://github.com/nginxinc/docker-nginx/blob/9abe4ae472b3332665fad9b12ee146dc242e775c/stable/alpine/Dockerfile>
Клонируем этот репозиторий в заранее подготовленный каталог на хост машине:

```
git clone https://github.com/nginxinc/docker-nginx/
```

Собираем оригинальный образ из официального Dockerfile, расположенного в дирректории `./stable/alpine`:

```
docker build -t nginx-alpine .
```

Проверим, что он появился:

```
docker image ls
```

```
nginx-alpine             latest      2738de7de400   22 hours ago   48.3MB
```

Теперь изменим стартовую страницу nginx. Для этого подменим файл index.html в официальном образе.
Создадим отдельный каталог на хостовой системе, куда положим предварительно созданный index.html со следующим содержанием:

```
<!DOCTYPE html>
<html>
<head>
<title>Greetings from Naro-Fominsk</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Greetings from Naro-Fominsk</h1>
<p>If you see this page, it means I finally launched this fucking container.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Homework done</em></p>
</body>
</html>
```
Также создадим в этом же каталоге новый Dockerfile со следующим содержимым:

```
FROM nginx-alpine
COPY index.html /usr/share/nginx/html
```
Ребилдим официальный образ:

```
docker build -t nginx-alpine-my .
```

Проверяем:

```
docker image ls
```

```
REPOSITORY        TAG       IMAGE ID       CREATED        SIZE
nginx-alpine-my   latest    1f958f8bf340   4 hours ago    48.3MB
nginx-alpine      latest    2738de7de400   23 hours ago   48.3MB
```

Посмотрим, что отдаёт nginx, запустив контейнер:

```
docker run -itPd nginx-alpine-my
```
Проверим, что контейнер успешно создан и запущен:

```
docker ps -a
```

```
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS          PORTS                                     NAMES
fca53016138f   nginx-alpine-my   "/docker-entrypoint.…"   59 seconds ago   Up 58 seconds   0.0.0.0:32776->80/tcp, :::32776->80/tcp   wonderful_goldberg
```
Через браузер с другого ПК в сети постучимся на ip хостовой машины:



![изображение](https://github.com/ConstantaNF/docker/assets/162187256/276f3c47-4c3d-4d1a-a6fc-af691a8f4b84)

Видим положительный результат. Кастомный образ готов.

Следующим этапом регистрируемся на <https://hub.docker.com/>, создаём новый репозиторий <https://hub.docker.com/repository/docker/constantanf/nginxcustom/> и пушим туда получившийся образ. 
Для начала необходимо залогиниться в наш репозиторий из терминала хостовой машины:

```
docker login
```

Вводим логин и пароль от УЗ <https://hub.docker.com/> . 

После успешной авторизации можно пушить образ. Выполняем следующикоманды по инструкции:

```
docker tag nginx-alpine-my:latest constantanf/nginxcustom:nginx-alpine-custom
```

Проверяем результат:

```
docker image ls
```
REPOSITORY                TAG                   IMAGE ID       CREATED        SIZE
constantanf/nginxcustom   nginx-alpine-custom   1f958f8bf340   4 hours ago    48.3MB
nginx-alpine-my           latest                1f958f8bf340   4 hours ago    48.3MB
nginx-alpine              latest                2738de7de400   23 hours ago   48.3MB
```

Следующий шаг:

```
docker push constantanf/nginxcustom:nginx-alpine-custom 
```

Результат:

```
The push refers to repository [docker.io/constantanf/nginxcustom]
35fadd453614: Layer already exists 
dd49962acfa5: Pushed 
c8ee365a8f96: Layer already exists 
caa2c1164fa4: Layer already exists 
c19ca31bf212: Layer already exists 
3651e56dc394: Layer already exists 
2806be01f229: Layer already exists 
0d01880f987b: Pushed 
d4fc045c9e3a: Pushed 
nginx-alpine-custom: digest: sha256:98aa5b188c871566b23ba191edded3dcb2139fcd917b4be12b9c3cb24fe7595a size: 2196
```

Образ отправлен в репозиторий.
Проверяем, что всё корректно, выполнив pull с нашего репозитория, запустив конейнер из полученного образа и проверив ответ nginx через браузер

```
docker pull constantanf/nginxcustom:nginx-alpine-custom
```

```
nginx-alpine-custom: Pulling from constantanf/nginxcustom
4abcf2066143: Already exists 
3129a53fcac0: Already exists 
51c6306186bf: Already exists 
097adf26662d: Already exists 
27b4ece6aebb: Already exists 
7101c7574452: Already exists 
98679bd10007: Already exists 
eb12bbd06b7f: Already exists 
de10516fc0d7: Already exists 
Digest: sha256:98aa5b188c871566b23ba191edded3dcb2139fcd917b4be12b9c3cb24fe7595a
Status: Downloaded newer image for constantanf/nginxcustom:nginx-alpine-custom
docker.io/constantanf/nginxcustom:nginx-alpine-custom
```

```
docker run -itPd constantanf/nginxcustom:nginx-alpine-custom 
```

```
docker ps -a
```

```
CONTAINER ID   IMAGE                                         COMMAND                  CREATED          STATUS          PORTS                                     NAMES
022b679c42f6   constantanf/nginxcustom:nginx-alpine-custom   "/docker-entrypoint.…"   39 seconds ago   Up 38 seconds   0.0.0.0:32777->80/tcp, :::32777->80/tcp   keen_margulis
```

![изображение](https://github.com/ConstantaNF/docker/assets/162187256/bb34efe1-cf0b-49c9-a9be-4225cc007a23)

Всё работает.






















































