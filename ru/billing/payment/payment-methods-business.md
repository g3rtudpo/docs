# Перевод с расчетного счета

Организации и ИП могут пополнять лицевой счет и оплачивать потребленные ресурсы с помощью перевода средств со своего расчетного счета.

## Перевод средств и оплата счета {#transaction}

Для перевода средств должен быть выставлен [счет на оплату](../concepts/bill.md). Оплатить счет можно в отделении любого банка или через систему клиент-банк.<br/>Перед переводом убедитесь, что в платежном поручении корректно указаны:

* сумма платежа;
* банковские реквизиты ООО «Яндекс.Облако» (для РФ), ТОО «Яндекс.Облако Казахстан» (для РК), SAG (для нерезидентов РФ и РК);
* ИНН вашей организации или ИП;
* [номер лицевого счета](../concepts/personal-account.md#id) в назначении платежа;
* [номер договора](../concepts/contract.md) в назначении платежа.

{% include [payment-bill-note](../_includes/payment-bill-note.md) %}



## Сроки оплаты {#limits}

Оплатить счет необходимо в сроки, предусмотренные [договором](../concepts/contract.md). Сроки зачисления средств на лицевой счет зависят от банка, проводящего платеж.


{% note info %}

Мы рекомендуем самостоятельно отслеживать расход средств с лицевого счета и [пополнять счет до положительного значения](../operations/pay-the-bill.md). Если баланс лицевого счета равен нулю или превысил максимальный размер порога оплаты, {{ yandex-cloud }} оставляет за собой право поменять статус вашего платежного аккаунта на [PAYMENT_REQUIRED](../concepts/billing-account-statuses.md#conditions). Дополнительную информацию см. в разделе [Цикл оплаты](../payment/billing-cycle-business.md).

{% endnote %}


