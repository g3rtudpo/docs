# Как начать работать с {{ certificate-manager-name }}

В этой инструкции вы создадите свой первый [сертификат от Let's Encrypt](../concepts/managed-certificate.md) и используете его для [настройки доступа по HTTPS](../../storage/operations/hosting/certificate.md) к статическому сайту, размещенному в {{ objstorage-full-name }}. 

## Подготовка к работе {#before-you-begin}

Чтобы начать работать с {{ certificate-manager-name }} вам понадобится:

1. Каталог в {{ yandex-cloud }}. Если каталога еще нет, создайте новый каталог:

    {% include [create-folder](../../_includes/create-folder.md) %}
  
1. Домен не ниже третьего уровня, для которого будет выпущен сертификат от Let's Encrypt.

    {% note info %}

    Чтобы пройти процедуру проверки прав на домен, он должен находиться под вашим управлением.

    {% endnote %}

1. Публичный бакет в Object Storage с точно таким же именем, что и домен. Если бакета еще нет, создайте его:
    
    {% list tabs %}
    
    - Консоль управления
    
        1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором хотите создать [бакет](../../storage/concepts/bucket.md).
        1. Выберите сервис **{{ objstorage-name }}**. 
        1. Нажмите кнопку **Создать бакет**.
        1. Введите имя бакета в точности совпадающее с именем домена.
        1. Выберите тип [доступа](../../storage/concepts/bucket.md#bucket-access) **Публичный**.
        1. Выберите [класс хранилища](../../storage/concepts/storage-class.md) по умолчанию.
        1. Нажмите кнопку **Создать бакет** для завершения операции.
     
    {% endlist %}
    
1. Настройте [хостинг](../../storage/operations/hosting/setup.md) в бакете:
   
    {% list tabs %}
    
    - Консоль управления
    
        1. В [консоли управления]({{ link-console-main }}) выберите сервис **{{ objstorage-name }}**.
        1. На вкладке **Бакеты** нажмите на бакет с именем домена.
        1. В левой панели выберите пункт **Веб-сайт**.
        1. Выберите раздел **Хостинг** и укажите главную страницу сайта.
        1. Нажмите кнопку **Сохранить** для завершения операции.
    
    {% endlist %}
    
1. Настройте [алиас](../../storage/operations/hosting/own-domain.md) для бакета у своего провайдера DNS или на собственном DNS-сервере.

    Например, для домена `www.example.com` необходимо добавить запись:
    
    ```
    www.example.com CNAME www.example.com.{{ s3-web-host }}
    ```
1. Установите и настройте AWS CLI по [инструкции](../../storage/tools/aws-cli.md#before-you-begin).

## Создание запроса на получение сертификата от Let's Encrypt {#request-certificate}

{% list tabs %}

- Консоль управления
    
    1. Перейдите в [консоль управления]({{ link-console-main }}).
    1. Выберите сервис **{{ certificate-manager-name }}**.
    1. Нажмите кнопку **Добавить сертификат**.
    1. В открывшемся меню выберите **Сертификат от Let's Encrypt**.
    1. В открывшемся окне задайте имя сертификата.    
    1. (опционально) Добавьте описание сертификату.
    1. В поле **Домены** укажите домены, для которых нужно выпустить сертификат.
    1. Выберите [тип проверки](../concepts/challenges.md) прав на домен `HTTP`. 
    1. Нажмите кнопку **Создать**.

{% endlist %}

## Прохождение проверки прав на домен {#validate}

1. Создайте файл для прохождения проверки:
    1. Перейдите в [консоль управления]({{ link-console-main }}).
    1. Выберите сервис **{{ certificate-manager-name }}**.
    1. Выберите в списке нужный сертификат со статусом `Validating` и нажмите на него.
    1. В блоке **Проверка прав на домены**:
        1. Скопируйте ссылку из поля **Ссылка для размещения файла**:
            * Часть ссылки вида `http://example.com/.well-known/acme-challenge/` — это путь для размещения файла.
            * Вторая часть ссылки `rG1Mm1bJ...` — это имя файла, которое вам необходимо использовать.
        1. Скопируйте содержимое файла из поля **Содержимое**. 
1. Загрузите созданный файл в бакет, так чтобы он располагался в папке `.well-known/acme-challenge`:
    
    {% list tabs %}
    
    - AWS CLI
    
        ```bash
        aws --endpoint-url=https://{{ s3-storage-host }} \
           s3 cp <имя файла> s3://<имя бакета>/.well-known/acme-challenge/<имя файла>
        ```
    
    {% endlist %}
    
1. Дождитесь изменения статуса сертификата на `Issued`.
1. Удалите созданный файл из бакета:
    
    {% list tabs %}
    
    - AWS CLI
    
        ```bash
        aws --endpoint-url=https://{{ s3-storage-host }} \
           s3 rm s3://<имя бакета>/.well-known/acme-challenge/<имя файла>
        ```
   
    {% endlist %}


{% note warning %}

Обновление сертификата требует от вас участия. Внимательно следите за временем жизни созданных сертификатов, чтобы своевременно обновлять их. Подробнее в разделе [Обновление сертификата](../concepts/managed-certificate.md#renew).

{% endnote %}

## Настройка доступа по HTTPS к статическому сайту {#hosting-certificate}

{% list tabs %}

- Консоль управления
    
    1. Войдите в [консоль управления]({{ link-console-main }}).
    1. Выберите сервис **{{ objstorage-name }}**.
    1. На вкладке **Бакеты** нажмите на бакет с именем домена.
    1. Перейдите на вкладку **HTTPS**.
    1. В отобразившейся панели справа нажмите кнопку **Настроить**.
    1. В поле **Источник** выберите **{{ certificate-manager-name }}**.
    1. В поле **Сертификат** выберите сертификат в появившемся списке. 
    1. Нажмите кнопку **Сохранить**.

{% endlist %}


#### См. также {#see-also}

- [{#T}](../concepts/managed-certificate.md)
- [{#T}](../concepts/challenges.md)
- [Настройка HTTPS в бакете](../../storage/operations/hosting/certificate.md)
