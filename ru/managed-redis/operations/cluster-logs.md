# Просмотр логов {{ RD }}-кластера

{{ mrd-name }} позволяет [получить фрагмент логов кластера](#get-log) за выбранный период и [просматривать логи в реальном времени](#get-log-stream).

{% include [log-duration](../../_includes/mdb/log-duration.md) %}

## Получить лог кластера {#get-log}

{% list tabs %}

- Консоль управления

    1. Перейдите на страницу каталога и выберите сервис **{{ mrd-name }}**.
    1. Нажмите на имя нужного кластера и выберите вкладку ![image](../../_assets/logs.svg) **Логи**.
    1. Укажите период времени, за который нужно отобразить логи: введите его вручную или выберите в календаре, нажав на поле ввода дат.
    1. При необходимости укажите хосты в выпадающем списке рядом с полем ввода дат.

    Будет отображен список записей в логе за выбранный период времени. Чтобы посмотреть подробную информацию о событии, нажмите на интересующую запись в списке.

    Если записей слишком много и отображается только часть из них, нажмите на ссылку **Загрузить еще** в конце списка.

- CLI

    {% include [cli-install](../../_includes/cli-install.md) %}

    {% include [default-catalogue](../../_includes/default-catalogue.md) %}

    1. Просмотрите описание команды CLI для просмотра логов кластера:

        ```bash
        {{ yc-mdb-rd }} cluster list-logs --help
        ```

    1. Запустите команду получения логов кластера (в примере приведены не все доступные параметры):

        ```bash
        {{ yc-mdb-rd }} cluster list-logs <имя или идентификатор кластера> \
           --limit <ограничение количества записей> \
           --columns <список колонок для вывода информации> \
           --filter <настройки фильтрации записей> \
           --since <левая граница временного диапазона> \
           --until <правая граница временного диапазона>
        ```

        Где:

        * {% include [logs output limit](../../_includes/cli/logs/limit.md) %}
        * `--columns` — список колонок для вывода информации:
            * `hostname` — [имя хоста](hosts.md#list-hosts).
            * `message` — сообщение, которое выводит сервис.
            * `pid` — идентификатор серверного процесса текущей сессии.
            * `role` — роль компонента кластера, например, `X` или `M` (`Sentinel` и `Master` соответственно).
        * {% include [logs filter](../../_includes/cli/logs/filter.md) %}
        * {% include [logs since time](../../_includes/cli/logs/since.md) %}
        * {% include [logs until time](../../_includes/cli/logs/until.md) %}

    Имя и идентификатор кластера можно запросить со [списком кластеров в каталоге](cluster-list.md#list-clusters).

- API

    Чтобы получить лог кластера, воспользуйтесь методом REST API [listLogs](../api-ref/Cluster/listLogs.md) для ресурса [Cluster](../api-ref/Cluster/index.md) или вызовом gRPC API [ClusterService/ListLogs](../api-ref/grpc/cluster_service.md#ListLogs) и передайте в запросе идентификатор кластера в параметре `clusterId`.

    Чтобы узнать идентификатор кластера, [получите список кластеров в каталоге](cluster-list.md#list-clusters).

{% endlist %}

## Получить поток логов кластера {#get-log-stream}

Этот способ позволяет получать логи кластера в реальном времени.

{% list tabs %}

- CLI

    {% include [cli-install](../../_includes/cli-install.md) %}

    {% include [default-catalogue](../../_includes/default-catalogue.md) %}

    Для просмотра логов кластера по мере их поступления выполните команду:

    ```bash
    {{ yc-mdb-rd }} cluster list-logs <имя или идентификатор кластера> --follow
    ```

    Имя и идентификатор кластера можно запросить со [списком кластеров в каталоге](cluster-list.md#list-clusters).

- API

    Чтобы получить поток логов кластера, воспользуйтесь методом REST API [streamLogs](../api-ref/Cluster/streamLogs.md) для ресурса [Cluster](../api-ref/Cluster/index.md) или вызовом gRPC API [ClusterService/StreamLogs](../api-ref/grpc/cluster_service.md#StreamLogs) и передайте в запросе идентификатор кластера в параметре `clusterId`.

    Чтобы узнать идентификатор кластера, [получите список кластеров в каталоге](cluster-list.md#list-clusters).

{% endlist %}
