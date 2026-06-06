# Ataque-ARP-MitM# Ataque ARP Man-in-the-Middle

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Kali%20Linux-red)
![Environment](https://img.shields.io/badge/Environment-GNS3%20%7C%20IOSvL2-orange)
![Attack](https://img.shields.io/badge/Attack-ARP%20MitM-purple)
![Use](https://img.shields.io/badge/Use-Controlled%20Lab-yellow)

## Información del proyecto

| Dato                  | Información                                       |
| --------------------- | ------------------------------------------------- |
| Autor                 | Darwing                                           |
| Matrícula             | 2024-2690                                         |
| Docente               | Jonathan Rondon                                   |
| Repositorio           | https://github.com/TuUsuario/ARP-MitM-Attack      |
| Video demostrativo    | https://youtu.be/gqlIJuG8STI         |
| Documentación técnica | docs/documentacion-tecnica-profesional.pdf        |
| Red de laboratorio    | 20.24.26.0/24                                     |

---

## Aviso de uso responsable

Este repositorio fue desarrollado únicamente con fines educativos, académicos y de laboratorio controlado. El script debe ejecutarse solamente en redes propias, laboratorios autorizados o entornos virtuales como GNS3, EVE-NG o PNETLab. No debe utilizarse en redes públicas, empresariales o de terceros sin autorización explícita.

---

## Objetivo del laboratorio

Demostrar cómo un atacante conectado a la misma red local puede realizar un ataque **ARP Man-in-the-Middle** para colocarse entre una víctima (PC1) y su gateway (R1), logrando interceptar, observar o bloquear el tráfico que circula entre ambos dispositivos.

---

## Objetivo del script

El script `arp_mitm.py` automatiza el envenenamiento de la caché ARP tanto de la víctima como del gateway. Para lograrlo envía respuestas ARP falsas de forma continua:

- Le dice a **PC1** que la IP de R1 corresponde a la MAC de Kali.
- Le dice a **R1** que la IP de PC1 corresponde a la MAC de Kali.

De este modo, todo el tráfico entre PC1 y R1 pasa por Kali antes de llegar al destino real.

---

## Archivos del repositorio

| Archivo                                        | Descripción                                                         |
| ---------------------------------------------- | ------------------------------------------------------------------- |
| `arp_mitm.py`                                  | Script principal para ejecutar el ataque ARP MitM.                  |
| `mitigacion-arp-mitm.md`                       | Documento con las contramedidas aplicadas.                          |
| `README.md`                                    | Guía principal del laboratorio.                                     |
| `docs/documentacion-tecnica-profesional.pdf`   | Documentación técnica profesional detallada del laboratorio.        |
| `images/`                                      | Capturas de pantalla de evidencia del laboratorio.                  |

---

## Topología del laboratorio

```
         R1 (20.24.26.91)
         f0/0
          |
         Gi0/2
        [SW-1]
       /       \
   Gi0/0      Gi0/1
   Kali        PC1
(20.24.26.90)  (20.24.26.92)
```

| Dispositivo | Rol           | Interfaz | Dirección IP    | Descripción                    |
| ----------- | ------------- | -------- | --------------- | ------------------------------ |
| R1          | Gateway       | F0/0     | 20.24.26.91/24  | Gateway de la red              |
| SW-1        | Switch capa 2 | Gi0/0~2  | N/A             | Switch que conecta los equipos |
| Kali Linux  | Atacante      | eth0     | 20.24.26.90/24  | Ejecuta el envenenamiento ARP  |
| PC1         | Víctima       | e0       | 20.24.26.92/24  | Cliente cuyo tráfico es interceptado |

---

## Requisitos previos

- GNS3 o entorno de virtualización equivalente.
- Kali Linux con Python 3 y Scapy instalado.
- Permisos de superusuario (`sudo`).
- Víctima y gateway en la misma subred.
- Conectividad de capa 2 entre Kali, PC1 y R1.

```bash
pip install scapy
```

---

## Parámetros del script

| Parámetro            | Descripción                                              | Ejemplo              |
| -------------------- | -------------------------------------------------------- | -------------------- |
| `-i` / `--iface`     | Interfaz de red conectada al switch                      | `-i eth0`            |
| `--victim`           | IP de la víctima                                         | `--victim 20.24.26.92` |
| `--gateway`          | IP del gateway                                           | `--gateway 20.24.26.91` |
| `-d` / `--delay`     | Intervalo entre paquetes ARP en segundos                 | `-d 0.05`            |
| `-v` / `--verbose`   | Muestra cada paquete enviado                             | `-v`                 |

---

## Estado inicial antes del ataque

Verificar las tablas ARP legítimas antes del ataque:

**En R1:**
```
R1# show arp
```

**En PC1:**
```
PC1> show arp
```

En este estado normal, PC1 conoce la MAC real del gateway y R1 conoce la MAC real de PC1.

---

## Ejecución del ataque

```bash
sudo python3 arp_mitm.py -i eth0 --victim 20.24.26.92 --gateway 20.24.26.91 -v
```

El script activa IP forwarding automáticamente para que el tráfico siga fluyendo mientras Kali intercepta:

```
╔══════════════════════════════════════════════╗
║         ARP MitM Attack                      ║
║  Interfaz : eth0                            ║
║  Víctima  : 20.24.26.92                     ║
║  Gateway  : 20.24.26.91                     ║
╚══════════════════════════════════════════════╝
[*] Activando IP forwarding...
[*] Enviando ARP falso → PC1 cree que GW=MAC_KALI
[*] Enviando ARP falso → R1 cree que PC1=MAC_KALI
[*] Ataque activo... (Ctrl+C para detener y restaurar)
```

Para detener y restaurar las tablas ARP originales: `Ctrl + C`

---

## Verificación del impacto

Después de iniciar el ataque verificar en PC1:

```
PC1> show arp
```

La IP del gateway `20.24.26.91` debe aparecer asociada a la **MAC de Kali** en vez de la MAC real de R1.

**Demostración de daño con iptables** — bloquear ICMP desde PC1 hacia R1:

```bash
sudo iptables -I FORWARD 1 -s 20.24.26.92 -d 20.24.26.91 -p icmp -j DROP
```

Desde PC1:
```
PC1> ping 20.24.26.91
```

El ping debe fallar, confirmando que Kali controla el tráfico entre la víctima y su gateway.

---

## Contramedida aplicada — Dynamic ARP Inspection (DAI)

La contramedida es **Dynamic ARP Inspection con ARP ACL estática**. Permite al switch validar qué combinaciones IP-MAC son legítimas dentro de la VLAN.

Asociaciones legítimas autorizadas:

| Dispositivo | IP legítima   | MAC legítima       |
| ----------- | ------------- | ------------------ |
| R1          | 20.24.26.91   | (MAC real de R1)   |
| PC1         | 20.24.26.92   | (MAC real de PC1)  |

Configuración en SW-1:

```
enable
configure terminal

arp access-list ARP_VALIDOS
 permit ip host 20.24.26.91 mac host <MAC-R1>
 permit ip host 20.24.26.92 mac host <MAC-PC1>
 exit

ip arp inspection vlan 1
ip arp inspection filter ARP_VALIDOS vlan 1 static

interface gigabitEthernet0/2
 description Puerto_confiable_hacia_R1
 ip arp inspection trust
 exit

interface gigabitEthernet0/0
 description Puerto_no_confiable_Kali
 no ip arp inspection trust
 exit

end
write memory
```

> Para obtener las MACs reales ejecutar `show arp` en R1 y `show interfaces` en el switch.

---

## Verificación posterior a la mitigación

Con DAI activo, volver a lanzar el ataque. Las tablas ARP de PC1 y R1 deben mantenerse con las MACs legítimas. El switch bloqueará las respuestas ARP falsificadas de Kali.

Comandos de verificación:

```
SW-1# show ip arp inspection
SW-1# show ip arp inspection statistics
SW-1# show arp access-list
```

---

## Video demostrativo

[Ver video del laboratorio en YouTube](https://www.youtube.com/watch?v=PENDIENTE)

---

## Conclusión

ARP no valida de forma nativa la identidad de los dispositivos en una red local, lo que permite a un atacante enviar respuestas falsas para posicionarse como intermediario entre la víctima y su gateway.

La mitigación con **Dynamic ARP Inspection y ARP ACL estática** bloquea asociaciones IP-MAC no autorizadas, evitando que Kali suplante al gateway o a la víctima. Esta práctica confirma la importancia de aplicar controles de capa 2 en switches de acceso para proteger contra ataques Man-in-the-Middle.

---

## Autor

**Darwing**  
Matrícula: **2024-2690**  
Repositorio: https://github.com/TuUsuario/ARP-MitM-Attack
