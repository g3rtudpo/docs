# Миграция данных в Managed Service for MongoDB

Чтобы перенести вашу базу данных в сервис Managed Service for MongoDB, нужно непосредственно перенести данные, закрыть старую базу данных на запись и перенести нагрузку на кластер БД в Яндекс.Облаке.

Перенести данные в кластер Managed Service for MongoDB можно с помощью утилит `mongodump` и `mongorestore`: создайте дамп рабочей базы и восстановите его в нужном кластере.

Перед тем, как переносить данные, проверьте, совпадают ли версии СУБД у существующей базы данных и вашего кластера в Облаке. Если версии разные, восстановить сделанный дамп не получится.

Последовательность действий:

1. [Создайте дамп](#dump) переносимой базы с помощью утилиты `mongodump`.
2. При необходимости [создайте виртуальную машину](#create-vm) в Compute Cloud, чтобы восстанавливать базу из дампа в инфраструктуре Яндекс.Облака.
3. [Создайте кластер Managed Service for MongoDB](#create-cluster), на котором будет развернута восстановленная база.
4. [Восстановите данные из дампа](#restore) в кластере с помощью утилиты `mongorestore`.


### Создайте дамп {#dump}

Создать дамп базы данных следует с помощью утилиты `mongodump`. Подробно утилита описана в [документации MongoDB](https://docs.mongodb.com/manual/reference/program/mongodump/).

1. Установите `mongodump` и дополнительные утилиты для работы с MongoDB. Пример для дистрибутивов Ubuntu и Debian:

    ```
    $ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4
    ...
    $ echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
    ...
    $ sudo apt-get update
    ...
    $ sudo apt-get install mongodb-org-shell mongodb-org-tools
    ```

    Инструкции для других платформ, а также более подробную информацию об установке утилит можно найти на странице [Install MongoDB](https://docs.mongodb.com/manual/installation/).

2. Перед созданием дампа рекомендуется переключить СУБД в режим «только чтение», чтобы не потерять данные, которые могут появиться за время создания дампа.

3. Создайте дамп базы данных:

    ```
    $ mongodump --host <адрес сервера СУБД> --port <порт> --username <имя пользователя> --password "<пароль>" --db <имя базы данных> --out ~/db_dump
    ```

   Если вы можете использовать несколько ядер процессора для создания дампа, задайте флаг `-j` с количеством доступных ядер:

    ```
    $ mongodump --host <адрес сервера СУБД> --port <порт> --username <имя пользователя> --password "<пароль>" -j <количество потоков> --db <имя базы данных> --out ~/db_dump
    ```

4. Архивируйте дамп:

    ```
    $ tar -cvzf db_dump.tar.gz ~/db_dump
    ```


### (опционально) Создайте виртуальную машину для загрузки дампа {#create-vm}

Промежуточная виртуальная машина в Yandex Compute Cloud понадобится, если:

* К вашему кластеру Managed Service for MongoDB нет доступа из интернета.
* Ваше оборудование или соединение с кластером в Облаке недостаточно надежны.

Чтобы подготовить виртуальную машину для восстановления дампа:

1. В консоли управления [создайте новую виртуальную машину](../../compute/operations/vm-create/create-linux-vm.md) из образа Ubuntu 18.04. Нужное количество оперативной памяти и ядер процессора зависит от объема переносимых данных и требуемой скорости переноса.

   Минимальной конфигурации (1 ядро, 2 ГБ RAM, 10 ГБ дискового пространства) должно хватить для переносы базы до 1 ГБ. Чем больше переносимая база, тем больше должно быть дискового пространства (как минимум в два раза больше, чем размер базы) и оперативной памяти.

   Виртуальная машина должна находиться в той же сети и зоне доступности, что хост-мастер кластера Managed Service for MongoDB. Кроме того, виртуальной машине должен быть присвоен внешний IP-адрес, чтобы вы могли загрузить файл дампа извне Облака.

2. Установите клиент MongoDB и дополнительные утилиты для работы с СУБД:

    ```
    $ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4
    ...
    $ echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
    ...
    $ sudo apt-get update
    ...
    $ sudo apt-get install mongodb-org-shell mongodb-org-tools
    ```

3. Перенесите дамп базы данных с вашего сервера на виртуальную машину, например, используя утилиту `scp`:

    ```
    scp ~/db_dump.tar.gz <имя пользователя ВМ>@<публичный адрес ВМ>:/tmp/db_dump.tar.gz
    ```

4. Распакуйте дамп на виртуальной машине:

    ```
    tar -xzf /tmp/db_dump.tar.gz
    ```

В результате вы должны получить виртуальную машину с дампом базы данных, который готов к восстановлению на кластер Managed Service for MongoDB.


## Создайте кластер Managed Service for MongoDB {#create-cluster}

Создайте кластер, вычислительная мощность и размер хранилища которого соответствуют среде, в которой развернута существующая база данных. Подробно о создании кластера Managed Service for MongoDB — на странице [#T](cluster-create.md).


### Восстановите данные {#restore}

Восстанавливать базу данных из дампа следует с помощью утилиты [mongorestore](https://docs.mongodb.com/manual/reference/program/mongorestore/).

* Если вы восстанавливаете дамп с виртуальной машине в Облаке:

    ```
    $ mongorestore --host <адрес сервера СУБД> \
                   --port <порт> \
                   --username <имя пользователя> \
                   --password "<пароль>" \
                   -j <количество потоков> \
                   --authenticationDatabase <имя базы данных> \
                   --nsInclude '*.*' /tmp/db_dump
    ```

* Если вы восстанавливаете дамп с сервера вне Яндекс Облака для `mongorestore` необходимо явно задать параметры SSL:

    ```
    $ mongorestore --host <адрес сервера СУБД> \
                   --port <порт> \
                   --ssl \
                   --sslCAFile <путь к файлу сертификата> \
                   --username <имя пользователя> \
                   --password "<пароль>" \
                   -j <количество потоков> \
                   --authenticationDatabase <имя базы данных> \
                   --nsInclude '*.*' ~/db_dump
    ```

* Если нужно перенести только определенные коллекции, то задайте флаги `--nsInclude` и `--nsExclude`с указанием на пространства имен, которые нужно или не нужно включать для восстанавливаемого набора коллекций.