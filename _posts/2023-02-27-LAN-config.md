---
layout: single
title: Cómo configurar una LAN y su Gateway
excerpt: Vamos a poder ver todos los pasos necesarios para configurar nuestra LAN con IP estáticas. Existira una máquina atacante, y otra víctima, que llegarán a Internet por medio del Gateway.
date: 2023-02-27
classes: wide
header:
  teaser: /assets/images/LAN_Ettercap/Escenario_LAN.png
  teaser_home_page: true
  icon: 
categories:
tags:
  - Linux
  - Networking
---

![](/assets/images/LAN_Ettercap/Escenario_LAN.png)

Vamos a ver cómo llevar a cabo la configuración en su totalidad paso a paso. Para realizar el escenario propuesto, primero tendremos que descargar la ISO de ubuntu para crear las máquinas de atacante y víctima:

<img src="/assets/images/LAN_Ettercap/Install_ub_deskt.png" alt="imageError" style="width:75%;">

Ahora, descargamos la ISO de ubuntu server, que será la que usaremos como gateway:

<img src="/assets/images/LAN_Ettercap/Install_ub_serv_iso.png" alt="imageError" style="width:75%;">

En este momento, va siendo hora de crear las máquinas virtuales de atacante y víctima utilizando la ISO de ubuntu desktop descargada anteriormente:

<img src="/assets/images/LAN_Ettercap/Creando_Maq_Atacante.png" alt="imageError" style="width:75%;">

Ahora, tendremos que añadir sobre la máquina virtual atacante y víctima una red interna para que puedan relacionarse entre sí. De manera que utilizarán el gateway a especificar posteriormente para conectarse a internet.

<img src="/assets/images/LAN_Ettercap/config_atacante_redinterna.png" alt="imageError" style="width:75%;">

Tras eso, simplemente seguimos los pasos de instalación:

<img src="/assets/images/LAN_Ettercap/Im_install_generica_ubuntu.png" alt="imageError" style="width:75%;">

(Acordaos de seguir los mismos pasos con la máquina víctima. En caso de querer duplicar la máquina atacante para aprovechar la configuración, aseguraos de que la dirección MAC sea distinta)

A continuación, haremos la instalación del que será nuestro Gateway. Lo haremos por medio de ubuntu server y eligiendo el primer adaptador de red como adaptador puente y el segundo como red interna(con el mismo nombre que la red interna especificada anteriormente):

<img src="/assets/images/LAN_Ettercap/Config_ub_serv_gatew.png" alt="imageError" style="width:75%;">

En este preciso momento, tenemos instalados tanto el ordenador que actuará como atacante, víctima y el gateway:

<img src="/assets/images/LAN_Ettercap/running_machines.png" alt="imageError" style="width:65%;">

Es momento de modificar las IP para que sean estáticas. A tal efecto, iremos a configuración y en la configuración del “Cableado”, seleccionaremos “ipv4” para que el método sea manual y así poder añadir la IP correspondiente junto con el gateway que hemos creado:

<u>Ordenador atacante:</u>

<img src="/assets/images/LAN_Ettercap/Ord_at_1.png" alt="imageError" style="width:85%;">

<img src="/assets/images/LAN_Ettercap/Ord_at_2.png" alt="imageError" style="width:65%;">

<u>Ordenador víctima:</u>

<img src="/assets/images/LAN_Ettercap/Ord_vic.png" alt="imageError" style="width:65%;">

En el ubuntu server, hay que seguir unos pasos para añadir la IP estática 192.168.0.1. A tal efecto, identifico el nombre del adaptador de red:

<img src="/assets/images/LAN_Ettercap/gatew_ip_ad.png" alt="imageError" style="width:75%;">

Posteriormente, realizamos el siguiente comando:
```
sudo nano /etc/netplan/00-installer-config.yaml
```
<img src="/assets/images/LAN_Ettercap/netplan_config.png" alt="imageError" style="width:75%;">

Lo modificamos de la siguiente manera, asignando la IP especificada con anterioridad:

<img src="/assets/images/LAN_Ettercap/netplan_config_changed.png" alt="imageError" style="width:75%;">

Y aplicamos los cambios:
```
sudo netplan apply
```
Si la configuración es correcta, se utilizará el adaptador “enpos3” para acceder a internet:

<img src="/assets/images/LAN_Ettercap/ip_rou_enpos3.png" alt="imageError" style="width:75%;">

Hasta ahora hemos configurado las interfaces pero ubuntu no permite la transmisión de paquetes entre interfaces por defecto, por ello debemos configurarlo para que lo permita. Con el siguiente comando podemos ver que el bit de traspaso de paquetes está desactivado:

<img src="/assets/images/LAN_Ettercap/ip_forw.png" alt="imageError" style="width:75%;">

Para activarlo recurrimos a descomentar en el archivo “/etc/sysctl.conf” la línea que contiene “net.ipv4.ip_fordward=1”:
```
nano /etc/sysctl.conf
```
<img src="/assets/images/LAN_Ettercap/uncomment_ip_forw.png" alt="imageError" style="width:75%;">

Tras eso, guardamos los cambios:

<img src="/assets/images/LAN_Ettercap/guardar_cambios_ip_for.png " alt="imageError" style="width:75%;">

Y nos aseguramos de que el bit haya cambiado:

<img src="/assets/images/LAN_Ettercap/bit_ip_for_changed.png" alt="imageError" style="width:75%;">

Ahora, es importante configurar el NAT sobre iptables para que traduzca las direcciones de red. Primero miramos que las iptables permitan el tráfico de todo tipo de paquetes:

<img src="/assets/images/LAN_Ettercap/iptables_state.png" alt="imageError" style="width:75%;">

Debemos asegurarnos de que en las iptables, la política POSTROUTING de nat también sea aceptada:

<img src="/assets/images/LAN_Ettercap/Postrouting_iptables.png" alt="imageError" style="width:75%;">

Asimismo, realizamos el enmascaramiento IP que ocurrirá sobre el “nat”:

<img src="/assets/images/LAN_Ettercap/Enmascaramiento_nat.png" alt="imageError" style="width:75%;">

Guardamos la configuración de iptables:
```
apt-get install iptables-persistent
netfilter-persistent save
```

Al reiniciar el ubuntu server, las reglas deberían seguir ahí:

<img src="/assets/images/LAN_Ettercap/reboot_img1.png" alt="imageError" style="width:65%;">

<img src="/assets/images/LAN_Ettercap/reboot_img2.png" alt="imageError" style="width:75%;">

Para ver que verdaderamente tenemos conexión con los ordenadores de atacante y víctima, haremos ping entre ellas:

* Desde el gateway al atacante y víctima:

<img src="/assets/images/LAN_Ettercap/ping_gateway_to.png" alt="imageError" style="width:65%;">

* Desde el atacante a la víctima y gateway:

<img src="/assets/images/LAN_Ettercap/ping_atac_to.png" alt="imageError" style="width:65%;">

* Desde la víctima al atacante y gateway:

<img src="/assets/images/LAN_Ettercap/ping_vict_to.png" alt="imageError" style="width:65%;">

* Desde el gateway hacia google:

<img src="/assets/images/LAN_Ettercap/ping_gatewgoogle.png" alt="imageError" style="width:65%;">

* Desde el atacante hacia google:

En caso de no funcionar, debemos añadir una DNS manualmente (o dos en nuestro caso) tanto en la víctima como en el atacante:

<img src="/assets/images/LAN_Ettercap/ipv4_manual_dns.png" alt="imageError" style="width:65%;">

<img src="/assets/images/LAN_Ettercap/ping_atac_google.png" alt="imageError" style="width:65%;">

* Desde la víctima hacia google:

<img src="/assets/images/LAN_Ettercap/ping_vict_google.png" alt="imageError" style="width:65%;">
