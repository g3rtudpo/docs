# Изменить ноду

Чтобы изменить [ноду](../../concepts/deploy/index.md#node):
1. {% include [find project](../../../_includes/datasphere/ui-find-project.md) %}
1. В блоке **Ресурсы** выберите **Нода**.
1. Выберите ноду, которую нужно изменить.
1. В правом верхнем углу нажмите кнопку ![Edit](../../../_assets/datasphere/edit.svg) **Редактировать**.
1. Измените параметры ноды:
    * Имя ноды.
    * Описание ноды.
    * Каталог, в котором создаются новые ресурсы.
    * [Конфигурацию](../../concepts/configurations.md) вычислительных ресурсов [инстанса](../../concepts/deploy/index.md), на котором развернута нода.
    * [Зоны доступности](../../../overview/concepts/geo-scope.md), в которых размещен инстанс.
    * [Подсети](../../../vpc/concepts/network.md#subnet) в которых размещен инстанс.
    * [Идентификаторы каталогов](../../../resource-manager/operations/folder/get-id.md), из которых можно подключаться к ноде (блок **ACL**).
1. Нажмите кнопку **Сохранить**.

#### См. также {#see-also}

* [{#T}](node-delete.md)
