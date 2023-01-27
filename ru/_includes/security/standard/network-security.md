# 2. Сетевая безопасность

В этом разделе представлены рекомендации пользователям по настройкам безопасности в {{ vpc-full-name }}.

Подробно о том, как настроить сетевую инфраструктуру, рассказывается в вебинаре [Как работает сеть в Облаке](https://www.youtube.com/watch?v=g3cZ0o50qH0).

Чтобы изолировать приложения друг от друга, поместите ресурсы в разные группы безопасности, а если требуется наиболее строгая изоляция — в разные сети. Трафик внутри сети по умолчанию разрешен, а между сетями — нет. Трафик между сетями можно передавать только через ВМ с двумя сетевыми интерфейсами в разных сетях, VPN или сервис {{ interconnect-name }}.

#### 2.1 Для объектов облака используется межсетевой экран или группы безопасности {#firewall}

Встроенный механизм [групп безопасности](../../../vpc/concepts/security-groups.md) позволяет управлять доступом виртуальных машин к ресурсам и группами безопасности {{ yandex-cloud }} или ресурсам в интернете. Группа безопасности — это набор правил для входящего и исходящего трафика, который можно назначить на сетевой интерфейс виртуальной машины. Группы безопасности работают как stateful firewall, то есть отслеживают состояние сессий: если правило разрешает создать сессию, ответный трафик будет автоматически разрешен. Инструкцию по настройке групп безопасности см. в разделе [{#T}](../../../vpc/operations/security-group-create.md). Указать группу безопасности можно в настройках ВМ.

Группы безопасности могут использоваться для защиты:

* виртуальных машин;
* управляемых баз данных;
* балансировщиков в {{ alb-name }};
* кластеров в {{ managed-k8s-name }}.

Список доступных сервисов расширяется.

Вы можете управлять сетевым доступом без групп безопасности, например, с помощью отдельной виртуальной машины — межсетевой экран на основе образа [NGFW]({{ link-cloud-marketplace }}/products/usergate/ngfw) из {{ marketplace-name }}, либо своего собственного образа. Использование NGFW может быть критично для тех клиентов, которым необходима следующая функциональность:

* составление логов сетевых соединений;
* потоковый анализ трафика на предмет зловредного контента;
* обнаружение сетевых атак по сигнатурам;
* другая функциональность классических NGFW-решений.

Убедитесь, что в ваших облаках используются группы безопасности на каждом объекте облака, либо используется отдельная ВМ NGFW из {{ marketplace-name }}, либо по принципу <q>bring your own image</q> (<q>используй свое устройство</q> — принцип, позволяющий использовать свое оборудование или образы системы).

{% list tabs %}

- Проверка в консоли управления

  Проверка наличия групп безопасности на объектах:

  1. Откройте консоль управления {{ yandex-cloud }} в вашем браузере.
  1. Перейдите в каждое облако и в каждый каталог и последовательно открывайте все перечисленные ресурсы в пункте <q>Объекты, на которые возможно применить группы безопасности</q>.
  1. В настройках объектов найдите параметр **Группа безопасности** и убедитесь, что назначена хотя бы одна группа безопасности.
  1. Если в параметрах каждого объекта, который поддерживает SG указана хотя бы одна SG то рекомендация выполняется. Если нет, то перейдите к п. <q>Инструкции и решения по выполнению</q>.

  Проверка наличия NGFW вместо SG:

  1. Откройте консоль управления {{ yandex-cloud }} в вашем браузере.
  1. Перейдите в каждое облако и в каждый каталог и последовательно откройте все диски ВМ.
  1. В настройках дисков найдите параметр **Продукт Marketplace**.
  1. Если в параметрах **Продукт Marketplace** в диске указано одно из названий продуктов NGFW: <q>Check Point CloudGuard IaaS - Firewall & Threat Prevention PAYG</q>, <q>UserGate NGFW</q>, рекомендация выполняется. Если нет, то перейдите к п. <q>Инструкции и решения по выполнению</q>.

- Проверка через CLI

  1. Посмотрите доступные вам организации и зафиксируйте необходимый ID:

     ```bash
     yc organization-manager organization list
     ```

  1. Выполните команду для поиска объектов облака без SG:

     ```bash
     export ORG_ID=<ID организации>
     for CLOUD_ID in $(yc resource-manager cloud list --organization-id=${ORG_ID} --format=json | jq -r '.[].id');
     do for FOLDER_ID in $(yc resource-manager folder list --cloud-id=$CLOUD_ID --format=json | jq -r '.[].id'); 
     do for VM_ID in $(yc compute instance list --folder-id=$FOLDER_ID --format=json | jq -r '.[].id'); do yc compute instance get --id=$VM_ID --format=json | jq -r '. | select(.network_interfaces[].security_group_ids | not)' | jq -r '.id' 
     done;
     done;
     done
     ```

  1. Если выдается пустая строка, то рекомендация выполняется. Если выдается результат с ID облачного ресурса, то перейдите к п. <q>Инструкции и решения по выполнению</q>.

  Проверка наличия NGFW вместо группы безопасности:

  1. Выполните команду для поиска NGFW в облаке (по умолчанию команда ищет Checkpoint или Usergate. Если используете свой образ, укажите его):

     ```bash
     export ORG_ID=<ID организации>
     for CLOUD_ID in $(yc resource-manager cloud list --organization-id=${ORG_ID} --format=json | jq -r '.[].id');
     do for FOLDER_ID in $(yc resource-manager folder list --cloud-id=$CLOUD_ID --format=json | jq -r '.[].id'); 
     do for DISK_ID in $(yc compute disk list --folder-id=$FOLDER_ID --format=json | jq -r '.[].id'); do yc compute disk get --id=$DISK_ID --format=json | jq -r '. | select(.product_ids[0]=="f2ecl4ak62mjbl13qj5f" or .product_ids[0]=="f2eqc5sac8o5oic7m99k")' | jq -r '.id'
     done;
     done;
     done
     ```

  1. Если выдается id ВМ с NGFW, то рекомендация выполняется. Если выдается пустая строка, то перейдите к п. <q>Инструкции и решения по выполнению</q>.

{% endlist %}

**Инструкции и решения по выполнению:**

* Примените SG на все объекты, на которых SG отсутствуют.
* Для применения SG с помощью {{ TF }} используйте [настройку групп безопасности (dev/stage/prod) с помощью {{ TF }}](https://github.com/yandex-cloud/yc-solution-library-for-security/tree/master/network-sec/segmentation).
* Для использования NGFW [установите](https://github.com/yandex-cloud/yc-solution-library-for-security/tree/master/network-sec/checkpoint-1VM) в {{ yandex-cloud }} ВМ межсетевой экран (NGFW): Check Point.
* [Инструкция](https://docs.google.com/document/d/1yYwHorzkwXwIUGeG3n_K6Zo-07BVYowZJL7q2bAgVR8/edit?usp=sharing) по использованию UserGate NGFW в облаке. 
* NGFW в режиме [active-passive](https://github.com/yandex-cloud/yc-solution-library-for-security/blob/master/network-sec/checkpoint-2VM_active-active/README.md). 

#### 2.2 Как минимум одна Группа безопасности существует в {{ vpc-short-name }} {#vpc-sg}

Для возможности назначения SG на облачные объекты в {{ vpc-short-name }} должна существовать как минимум одна Группа безопасности. Дополнительно существует функция создания [группы безопасности по умолчанию](../../../vpc/concepts/security-groups.md#default-security-group) — такая группа назначается облачным объектам при подключении к подсетям, если у них нет ни одной группы безопасности. Убедитесь в том, что, хотя бы одна группа безопасности существует в каждой сети.

{% list tabs %}

- Проверка в консоли управления

  1. Откройте консоль {{ yandex-cloud }} в вашем браузере.
  1. Перейдите в каждое облако, далее в каждый каталог и в каждую {{ vpc-short-name }}.
  1. Перейдите в раздел **Группы безопасности**.
  1. Если обнаружена как минимум одна группа безопасности для каждой {{ vpc-short-name }}, либо группа по умолчанию, рекомендация выполняется. Если нет, перейдите к п. <q>Инструкции и решения по выполнению</q>.

- Проверка через CLI

  1. Посмотрите доступные вам организации и зафиксируйте необходимый ID:

     ```bash
     yc organization-manager organization list
     ```
   
  1. Выполните команду для поиска каталогов без наличия SG:

     ```bash
     export ORG_ID=<ID организации>
     for CLOUD_ID in $(yc resource-manager cloud list --organization-id=${ORG_ID} --format=json | jq -r '.[].id');
     do for FOLDER_ID in $(yc resource-manager folder list --cloud-id=$CLOUD_ID --format=json | jq -r '.[].id'); \
     do echo "SG_ID: " && yc vpc security-group list --folder-id=$FOLDER_ID --format=json | jq -r '.[] | select(.id)' | jq -r '.id'  && echo "FOLDER_ID: " $FOLDER_ID && echo "-----"
     done;
     done
     ```

  1. Если у каждого сочетания SG_ID напротив FOLDER_ID, в которой она находится указаны ID, рекомендация выполняется. Если нет, то перейдите к п. <q>Инструкции и решения по выполнению</q>.

{% endlist %}

**Инструкции и решения по выполнению:**

Создайте группу безопасности в каждой {{ vpc-short-name }} с ограниченными правилами доступа, чтобы ее можно было назначать на облачные объекты.

#### 2.3 В Группах безопасности отсутствует слишком широкое правило доступа {#access-rule}

В группе безопасности существует возможность открыть сетевой доступ для абсолютно всех IP-адресов интернета и также по всем диапазонам портов. Опасное правило выглядит следующим образом:

* Диапазон портов: 0-65535 либо пусто.
* Протокол: любой либо TCP/UDP.
* Источник: CIDR.
* CIDR блоки: 0.0.0.0/0 (доступ со всех адресов) или ::/0 (ipv6).

{% note warning %}

Если диапазон портов не указан, то считается, что доступ предоставляется по всем портам (0-65535).

{% endnote %}

Открывать сетевой доступ необходимо только по тем портам, которые требуются для работы вашего приложения и для тех адресов, с которых необходимо подключаться к вашим объектам.

{% list tabs %}

- Проверка в консоли управления

  1. Откройте консоль {{ yandex-cloud }} в вашем браузере.
  1. Перейдите в каждое облако, далее в каждый каталог и в каждую {{ vpc-short-name }}.
  1. Перейдите в раздел **Группы безопасности**.
  1. Если не обнаружено ни одной группы безопасности, в которой есть правила сетевого доступа, разрешающие доступ по всем портам для всех адресов (интерпретация указана выше) то рекомендация выполняется. Если нет, то перейдите к п. <q>Инструкции и решения по выполнению</q>.

- Проверка через CLI

  1. Посмотрите доступные вам организации и зафиксируйте необходимый ID:

     ```bash
     yc organization-manager organization list
     ```

  1. Найдите группы безопасности с опасным правилом доступа:

     ```bash
     export ORG_ID=<ID организации>
     for CLOUD_ID in $(yc resource-manager cloud list --organization-id=${ORG_ID} --format=json | jq -r '.[].id');
     do for FOLDER_ID in $(yc resource-manager folder list --cloud-id=$CLOUD_ID --format=json | jq -r '.[].id'); \
     do echo "SG_ID: " && yc vpc security-group list --folder-id=$FOLDER_ID \
     --format=json | jq -r '.[] | select(.rules[].direction=="INGRESS" and .rules[].ports.to_port=="65535" and .rules[].cidr_blocks.v4_cidr_blocks[]=="0.0.0.0/0")' | jq -r '.id'  \
     && echo "FOLDER_ID: " $FOLDER_ID && echo "-----"
     done;
     done
     ```

  1. Если SG_ID напротив FOLDER_ID указано пустое значение, рекомендация выполняется. Если вы видите не пустое SG_ID, перейдите к п. <q>Инструкции и решения по выполнению</q>.

{% endlist %}

**Инструкции и решения по выполнению:**

Удалите опасное правило в каждой SG или отредактируйте, указав доверенные IP-адреса.

#### 2.4 Доступ по управляющим портам открыт только для доверенных IP-адресов {#trusted-ip}

Рекомендуется открывать доступ к вашей облачной инфраструктуре по управляющим портам только с доверенных IP-адресов. Убедитесь, что в ваших правилах доступа в рамках SG отсутствуют широкие правила доступа по управляющим портам:

* Диапазон портов: 22, 3389, или 21
* Протокол: TCP
* Источник: CIDR
* CIDR блоки: 0.0.0.0/0 (доступ со всех адресов) или ::/0 (ipv6)

{% list tabs %}

- Проверка в консоли управления

  1. Откройте консоль {{ yandex-cloud }} в вашем браузере.
  1. Перейдите в каждое облако, далее в каждый каталог и в каждую {{ vpc-short-name }}.
  1. Перейдите в раздел **Группы безопасности**.
  1. Если не обнаружено ни одной группы безопасности, в которой есть правила сетевого доступа, разрешающие доступ по управляющим портам для всех адресов (интерпретация указана выше), то рекомендация выполняется. Если нет, то перейдите к п. <q>Инструкции и решения по выполнению</q>.

- Проверка через CLI

  1. Посмотрите доступные вам организации и зафиксируйте необходимый ID:

     ```bash
     yc organization-manager organization list
     ```

  1. Выполните команду для поиска групп безопасности с опасным правилом доступа:

     ```bash
     export ORG_ID=<ID организации>
     for CLOUD_ID in $(yc resource-manager cloud list --organization-id=${ORG_ID} --format=json | jq -r '.[].id');
     do for FOLDER_ID in $(yc resource-manager folder list --cloud-id=$CLOUD_ID --format=json | jq -r '.[].id'); \
     do echo "SG_ID: " && yc vpc security-group list --folder-id=$FOLDER_ID \
     --format=json | jq -r '.[] | select(.rules[].direction=="INGRESS" and (.rules[].ports.to_port=="22" or .rules[].ports.to_port=="3389" or .rules[].ports.to_port=="21") and .rules[].cidr_blocks.v4_cidr_blocks[]=="0.0.0.0/0")' | jq -r '.id'  \
     && echo "FOLDER_ID: " $FOLDER_ID && echo "-----"
     done;
     done
     ```

  1. Если SG_ID напротив FOLDER_ID указано пустое значение, то рекомендация выполняется. Если вы видите не пустое SG_ID, то перейдите к п. <q>Инструкции и решения по выполнению</q>.

{% endlist %}

**Инструкции и решения по выполнению:**

[Удалите](../../../cli/cli-ref/managed-services/vpc/security-group/index.md) опасное правило в каждой SG или укажите доверенные IP-адреса.

#### 2.5 Включена защита от DDoS атак {#ddos-protection}

В {{ yandex-cloud }} существует базовая защита от DDoS и расширенная. Необходимо убедиться, что у вас используется как минимум базовая защита.

* [{{ ddos-protection-full-name }}](../../../vpc/ddos-protection/index.md) — это компонент сервиса {{ vpc-short-name }} для защиты облачных ресурсов от DDoS-атак. {{ ddos-protection-name }} предоставляется в партнерстве с Qrator Labs. Вы можете включать ее самостоятельно на внешний IP-адрес через инструменты управления облаком. Работает до L4 уровня модели OSI.
* [Расширенная](https://cloud.yandex.ru/services/ddos-protection) защита от DDoS-атак — работает на 3 и 7 уровнях модели OSI. Вы также можете отслеживать показатели нагрузки, параметры атак и подключить Solidwall WAF в личном кабинете Qrator Labs. Чтобы включить расширенную защиту, обратитесь к вашему менеджеру или в техническую поддержку.

{% list tabs %}

- Проверка из UI консоли (Базовая защита)

  1. Откройте консоль {{ yandex-cloud }} в вашем браузере.
  1. Откройте все созданные сети.
  1. Перейдите в раздел **IP-адреса**.
  1. Если у всех публичных адресов в столбце <q>Защита от DDoS-атак</q> установлено значение <q>Включена</q>, то рекомендация выполняется. Если нет, то перейдите к п. <q>Инструкции и решения по выполнению</q>.

- Ручная проверка (Расширенная защита)

  Обратитесь к вашему менеджеру со стороны облака и уточните подключена ли у вас <q>Расширенная защита от DDoS-атак</q>.

- Проверка через CLI

  1. Посмотрите доступные вам организации и зафиксируйте необходимый ID:

     ```bash
     yc organization-manager organization list
     ```

  1. Выполните команду для поиска IP-адресов без защиты от DDoS:

     ```bash
     export ORG_ID=<ID организации>
     for CLOUD_ID in $(yc resource-manager cloud list --organization-id=${ORG_ID} --format=json | jq -r '.[].id');
     do for FOLDER_ID in $(yc resource-manager folder list --cloud-id=$CLOUD_ID --format=json | jq -r '.[].id'); \
     do echo "Address_ID: " && yc vpc address list --folder-id=$FOLDER_ID \
     --format=json | jq -r '.[] | select(.external_ipv4_address.requirements.ddos_protection_provider=="qrator" | not)' | jq -r '.id' \
     && echo "FOLDER_ID: " $FOLDER_ID && echo "-----" 
     done;
     done
     ```

  1. Если Address_ID напротив FOLDER_ID указано пустое значение, рекомендация выполняется. В противном случае перейдите к п. <q>Инструкции и решения по выполнению</q>

{% endlist %}

**Инструкции и решения по выполнению:**

* Вебинар [Защита от DDoS в {{ yandex-cloud }}](https://youtu.be/KWGbLQTth5U).
* Все [материалы](../../../vpc/ddos-protection/index.md) по защите от DDoS в Облаке.

#### 2.6 Используется защищенный удаленный доступ {#secure-access}

Чтобы обеспечить удаленное подключение администраторов к облачным ресурсам, используйте одно из следующих решений:

* Site-to-site VPN между удаленной площадкой (например, вашим офисом) и облаком. В качестве шлюза для удаленного доступа используйте ВМ с функцией site-to-site VPN на основе [образа]({{ link-cloud-marketplace }}?categories=network) из {{ marketplace-name }}.

**Варианты настройки**:

* [Создание туннеля IPSec VPN с использованием демона strongSwan](../../../tutorials/routing/ipsec-vpn.md).
* [Создание site-to-site VPN-соединения с {{ yandex-cloud }} с помощью {{ TF }}](https://github.com/yandex-cloud/yc-solution-library-for-security/tree/master/network-sec/vpn).
* Client VPN между удаленными устройствами и {{ yandex-cloud }}. В качестве шлюза для удаленного доступа используйте ВМ с функцией client VPN на основе [образа]({{ link-cloud-marketplace }}?categories=network) из {{ marketplace-name }}.

  См. инструкцию в разделе [Создание VPN-соединения с помощью OpenVPN](../../../tutorials/routing/openvpn.md). Возможно так же использование сертифицированных СКЗИ.

* Приватное выделенное соединение между удаленной площадкой и {{ yandex-cloud }} c помощью услуги [{{ interconnect-name }}](../../../interconnect/index.yaml).

Для доступа в инфраструктуру по управляющим протоколам (например, SSH, RDP) рекомендуется создать бастионную виртуальную машину. Для этого можно использовать бесплатное решение [Teleport](https://goteleport.com/). Доступ к бастионной виртуальной машине или VPN-шлюзу из интернета должен быть ограничен.

Для дополнительного контроля действий администраторов рекомендуется использовать решения PAM (Privileged Access Management) с записью сессии администратора (например, Teleport). Для доступа по SSH и VPN рекомендуется отказаться от паролей и вместо этого использовать открытые ключи, X.509-сертификаты и SSH-сертификаты. При настройке SSH для виртуальных машин рекомендуется использовать SSH-сертификаты, в том числе и для хостовой части SSH.

Для доступа к веб-сервисам, развернутым в облаке, рекомендуется использовать TLS версий 1.2 и выше.

{% list tabs %}

- Проверка в консоли управления

  1. Откройте консоль {{ yandex-cloud }} в вашем браузере.
  1. Откройте все созданные сети.
  1. Перейдите в раздел **Таблицы маршрутизации**.
  1. Если найдены маршруты в приватные сети удаленных площадок, которые направлены через ВМ с VPN шлюзом, то рекомендация выполняется.
  1. Также проверьте виртуальные машины в каждом облаке на предмет наличия VPN шлюзов и проверьте назначенные им Security Groups на предмет открытых известных портов для VPN.

- Ручная проверка

  Обратитесь к вашему менеджеру со стороны облака и уточните подключена ли у вас услуга {{ interconnect-name }}. Если подключена, проанализируйте выполняется ли удаленный доступ.

{% endlist %}

#### 2.7 Исходящий доступ в интернет контролируется {#outgoing-access}

Возможные варианты организации исходящего доступа в интернет:

* [Публичный IP-адрес](../../../vpc/concepts/address.md#public-addresses). Адрес назначается ВМ по принципу one-to-one NAT.
* [Egress NAT (NAT-шлюз)](../../../vpc/operations/enable-nat.md). Включает доступ в интернет для подсети через общий пул публичных адресов {{ yandex-cloud }}. Не рекомендуется использовать Egress NAT для критичных взаимодействий, так как IP-адрес NAT-шлюза может использоваться несколькими клиентами одновременно. Следует учитывать эту особенность при моделировании угроз для инфраструктуры. Подробнее про [настройку](../../../vpc/operations/create-nat-gateway.md).
* [NAT-инстанс](../../../tutorials/routing/nat-instance.md). Функцию NAT выполняет отдельная ВМ. Для создания такой ВМ можно использовать образ [NAT-инстанс]({{ link-cloud-marketplace }}/products/yc/nat-instance-ubuntu-18-04-lts) из {{ marketplace-name }}.

**Сравнение способов доступа в интернет**:

#|
|| Публичный IP-адрес	| Egress NAT	| NAT-инстанс ||
|| **Плюсы:**	| **Плюсы:**	| **Плюсы:** ||
|| 
* Не требует настройки
* Выделенный адрес для каждой ВМ
|
* Не требует настройки
* Работает только на исходящих соединениях
|
* Возможность фильтровать трафик на NAT-инстансе
* Возможность использовать собственный файрвол
* Экономия IP-адресов ||
|| **Минусы:**	| **Минусы:**	| **Минусы:** ||
||
* Выставлять ВМ напрямую в интернет может быть небезопасно
* Стоимость резервирования каждого адреса |
* Общий пул IP-адресов
* Функция на стадии Preview, поэтому не рекомендуется для продуктовых сред	|
* Требуется настройка
* Стоимость использования ВМ (vCPU, RAM, диска) ||
|#

Вне зависимости от выбранного варианта организации исходящего доступа в интернет, ограничивайте трафик с помощью одного из механизмов, описанных выше. Для построения защищенной системы необходимо использовать [статические IP-адреса](../../../vpc/concepts/address.md), так как их можно внести в список исключений файрвола принимающей стороны.

{% list tabs %}

- Проверка в консоли управления

  1. Откройте консоль {{ yandex-cloud }} в вашем браузере.
  1. Перейдите в нужный каталог.
  1. Перейдите в раздел **IP-адреса**.
  1. Если у всех публичных адресов в столбце **Защита от DDoS-атак** установлено значение **Включена**, рекомендация выполняется. В противном случае перейдите к п. <q>Инструкции и решения по выполнению</q>.

- Проверка через CLI

  1. Посмотрите доступные вам организации и зафиксируйте необходимый ID:

     ```bash
     yc organization-manager organization list
     ```
    
  1. Выполните команду для поиска всех ВМ с публичными адресами:

     ```bash
     export ORG_ID=<ID организации>
     for CLOUD_ID in $(yc resource-manager cloud list --organization-id=${ORG_ID} --format=json | jq -r '.[].id');
     do for FOLDER_ID in $(yc resource-manager folder list --cloud-id=$CLOUD_ID --format=json | jq -r '.[].id'); 
     do echo "VM_ID: " && yc compute instance list --folder-id=$FOLDER_ID --format=json | jq -r '.[] | select(.network_interfaces[].primary_v4_address.one_to_one_nat.address)' | jq -r '.id' \
     && echo "FOLDER_ID: " $FOLDER_ID && echo "-----" 
     done;
     done
     ```
   
  1. Если VM_ID напротив FOLDER_ID указано пустое значение, рекомендация выполняется. В противном случае перейдите к п. <q>Инструкции и решения по выполнению</q>.

  1. Выполните команду для поиска наличия Egress NAT (NAT-шлюз):

     ```bash
     export ORG_ID=<ID организации>
     for CLOUD_ID in $(yc resource-manager cloud list --organization-id=${ORG_ID} --format=json | jq -r '.[].id');
     do for FOLDER_ID in $(yc resource-manager folder list --cloud-id=$CLOUD_ID --format=json | jq -r '.[].id'); \
     do echo "NAT_GW: " && yc vpc gateway list --folder-id=$FOLDER_ID --format=json | jq -r '.[] | select(.id)' | jq -r '.id'  && echo "FOLDER_ID: " $FOLDER_ID && echo "-----"
     done;
     done
     ```

  1. Если NAT_GW напротив FOLDER_ID указано пустое значение, рекомендация выполняется. В противном случае перейдите к п. <q>Инструкции и решения по выполнению</q>.

  1. Выполните команду для поиска наличия NAT-инстанса:

     ```bash
     export ORG_ID=<ID организации>
     for CLOUD_ID in $(yc resource-manager cloud list --organization-id=${ORG_ID} --format=json | jq -r '.[].id');
     do for FOLDER_ID in $(yc resource-manager folder list --cloud-id=$CLOUD_ID --format=json | jq -r '.[].id'); 
     do for DISK_ID in $(yc compute disk list --folder-id=$FOLDER_ID --format=json | jq -r '.[].id'); do yc compute disk get --id=$DISK_ID --format=json | jq -r '. | select(.product_ids[0]=="fd8v7ru46kt3s4o5f0uo")' | jq -r '.id'
     done;
     done;
     done
     ```
   
  1. Если результатом является пустая строка, то рекомендация выполняется. Если видите id NAT-инстанса, перейдите к п. <q>Инструкции и решения по выполнению</q>.

{% endlist %}

**Инструкции и решения по выполнению:**

* В случае наличия публичных адресов на ВМ убедитесь, что они необходимы. В противном случае удалите внешний IP-адрес в настройках ВМ.
* В случае наличия NAT-Gateway убедитесь, что он необходим. В противном случае удалите его.
* В случае наличия NAT-инстанс убедитесь, что он необходим. В противном случае удалите его.

#### 2.8 Запросы DNS не передаются в сторонние рекурсивные резолверы {#recursive-resolvers}

Для повышения отказоустойчивости часть трафика может передаваться в сторонние рекурсивные резолверы. Если необходимо избежать этого, обратитесь в [службу технической поддержки](../../../support/overview.md).