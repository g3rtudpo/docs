# Удалить снимок диска

{% note warning %}

Удаление снимка — неотменяемая и необратимая операция, восстановить удаленный снимок невозможно.

{% endnote %}

Чтобы удалить снимок:

{% list tabs %}

- Консоль управления

  1. В консоли управления выберите каталог, в котором находится снимок.
  1. Выберите сервис **{{ compute-name }}**.
  1. На панели слева выберите ![image](../../../_assets/compute/snapshots.svg) **Снимки дисков**.
  1. В строке с нужным снимком нажмите значок ![image](../../../_assets/dots.svg) и выберите в меню команду **Удалить**.
  1. В открывшемся окне нажмите кнопку **Удалить**.

- CLI

  {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

  1. Посмотрите описание команд CLI для удаления снимков:

     ```bash
     yc compute snapshot delete --help
     ```

  1. Получите список снимков в каталоге по умолчанию:

     {% include [compute-snapshot-list](../../_includes_service/compute-snapshot-list.md) %}

  1. Выберите идентификатор (`ID`) или имя (`NAME`) нужного снимка.
  1. Удалите снимок:

     ```bash
     yc compute snapshot delete \
       --name first-snapshot
     ```


- API

  Воспользуйтесь методом REST API [delete](../../api-ref/Snapshot/delete.md) для ресурса [Snapshot](../../api-ref/Snapshot/index.md) или вызовом gRPC API [SnapshotService/Delete](../../api-ref/grpc/snapshot_service.md#Delete).

- {{ TF }}

  Подробнее о {{ TF }} [читайте в документации](../../../tutorials/infrastructure-management/terraform-quickstart.md#install-terraform).

  Если вы создавали снимок диска с помощью {{ TF }}, вы можете удалить его:
  1. В командной строке перейдите в папку, где расположен конфигурационный файл {{ TF }}.
  1. Удалите ресурсы с помощью команды:

     ```bash
     terraform destroy
     ```

     {% note alert %}

     {{ TF }} удалит все ресурсы, созданные в текущей конфигурации: кластеры, сети, подсети, виртуальные машины и т. д.

     {% endnote %}

  1. Введите слово `yes` и нажмите **Enter**.

{% endlist %}