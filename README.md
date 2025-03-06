Домашнее задание к занятию 1 «Disaster recovery и Keepalived» - Оганесян Гор

### Задание 1

- Дана [схема](1/hsrp_advanced.pkt) для Cisco Packet Tracer, рассматриваемая в лекции.
- На данной схеме уже настроено отслеживание интерфейсов маршрутизаторов Gi0/1 (для нулевой группы)
- Необходимо аналогично настроить отслеживание состояния интерфейсов Gi0/0 (для первой группы).
- Для проверки корректности настройки, разорвите один из кабелей между одним из маршрутизаторов и Switch0 и запустите ping между PC0 и Server0.
- На проверку отправьте получившуюся схему в формате pkt и скриншот, где виден процесс настройки маршрутизатора.

**В ходе выполнения задания возникли следующие особенности:**

- В лекции **не было указано**, как именно настроить HSRP.
- В лекции **практически не рассматривалась работа в Cisco Packet Tracer**, хотя задание требует выполнения именно в нем.
- Несмотря на это, я смог разобраться, что для корректной работы HSRP необходимо вручную добавить параметры **priority** и **preempt**.

**Эти параметры я настроил и проверил — в работе HSRP всё корректно отрабатывало.**

Прошу **сообщить об этом координатору**, так как отсутствие этих важных моментов в учебных материалах существенно усложняет выполнение практических заданий.

![Config 1](https://github.com/hovhannisyan-code/hsrp/blob/master/img/Screenshot_0.png)
![Config 2](https://github.com/hovhannisyan-code/hsrp/blob/master/img/Screenshot_1.png)

---

### Задание 2

- Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример конфигурационного [файла](1/keepalived-simple.conf).
- Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах
- Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.
- Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script
- На проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html

---

Keepalived conf _MASTER_

```bash
global_defs {
    enable_script_security
}

vrrp_script check_web {
    script "/etc/keepalived/check_web.sh"
    interval 3
    weight -100
}

vrrp_instance VI_1 {
    state MASTER
    interface enp0s8
    virtual_router_id 15
    priority 255
    advert_int 1

    virtual_ipaddress {
        192.168.111.200/24
    }

    track_script {
        check_web
    }
}
```

Keepalived config _BACKUP_

```bash
global_defs {
    enable_script_security
}

vrrp_script check_web {
    script "/etc/keepalived/check_web.sh"
    interval 3
    weight -100
}

vrrp_instance VI_1 {
    state BACKUP
    interface enp0s8
    virtual_router_id 15
    priority 200
    advert_int 1

    virtual_ipaddress {
        192.168.111.200/24
    }

    track_script {
        check_web
    }
}
```

Check Script (/etc/keepalived/check_web.sh)

```bash
#!/bin/bash

PORT=80
INDEX_FILE="/var/www/html/index.html"

# check port
if ! ss -tln | grep -q ":${PORT} "; then
    exit 1
fi

# check index html
if [ ! -f "$INDEX_FILE" ]; then
    exit 1
fi

exit 0
```

```bash
chmod +x /etc/keepalived/check_web.sh
```

```bash
sudo systemctl restart keepalived
```

Result

![Result](https://github.com/hovhannisyan-code/hsrp/blob/master/img/Screenshot_2.png)
