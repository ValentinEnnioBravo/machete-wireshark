# 🎯 MACHETE COMPLETO: ANÁLISIS DE TRAMAS ETHERNET

## 🔍 Identificación Rápida

### Posiciones Fijas (SIN VLAN)
| Campo | Bytes | Descripción |
|-------|-------|-------------|
| MAC Destino | 0-5 | En hexadecimal XX:XX:XX:XX:XX:XX |
| MAC Origen | 6-11 | En hexadecimal XX:XX:XX:XX:XX:XX |
| EtherType/Length | 12-13 | >1500 = EtherType, ≤1500 = Length |
| Protocolo IP | 23 | 1=ICMP, 6=TCP, 17=UDP |
| Time to Live (TTL) | 22 | Pasar a decimal (cant. saltos) |
| Flags Fragmentación | 20-21 | Ver tabla de fragmentación|
| IP Origen | 26-29 | **YA están en decimal: A.B.C.D** |
| IP Destino | 30-33 | **YA están en decimal: A.B.C.D** |
| Puerto Origen | 34-35 | **2 bytes → convertir a decimal** |
| Puerto Destino | 36-37 | **2 bytes → convertir a decimal** |

### 1️⃣ Tipo de Trama (bytes 12-13)
| Valor | Tipo | Significado |
|-------|------|-------------|
| ≤ 1500 (0x05DC) | IEEE 802.3 | Campo Length |
| > 1500 | Ethernet II | Campo EtherType |
| 0x8100 | VLAN Tagged | 802.1Q VLAN Tag |

### 2️⃣ EtherTypes Comunes (Ethernet II)
| EtherType | Protocolo | Descripción |
|-----------|-----------|-------------|
| 0x0800 | IPv4 | Internet Protocol v4 |
| 0x0806 | ARP | Address Resolution Protocol |
| 0x8035 | RARP | Reverse ARP |
| 0x86DD | IPv6 | Internet Protocol v6 |

### 3️⃣ Protocolos IP (byte 23)
| Código | Protocolo | Descripción |
|--------|-----------|-------------|
| 1 | ICMP | Internet Control Message Protocol |
| 6 | TCP | Transmission Control Protocol |
| 17 | UDP | User Datagram Protocol |

### 4️⃣ Puertos Conocidos
| Puerto | Servicio | Protocolo | Descripción |
|--------|----------|-----------|-------------|
| 21 | FTP | TCP | File Transfer Protocol |
| 22 | SSH | TCP | Secure Shell |
| 25 | SMTP | TCP | Simple Mail Transfer Protocol |
| 53 | DNS | TCP/UDP | Domain Name System |
| 80 | HTTP | TCP | HyperText Transfer Protocol |
| 110 | POP3 | TCP | Post Office Protocol v3 |
| 143 | IMAP | TCP | Internet Message Access Protocol |
| 443 | HTTPS | TCP | HTTP Secure |

---

## 📊 Análisis por Capas
### Plantilla de Análisis
```
╔════════════════════════════════════════════════════════════╗
║                        CAPA 2 - ENLACE                    ║
╚════════════════════════════════════════════════════════════╝
• MAC Destino: __ : __ : __ : __ : __ : __
• MAC Origen: __ : __ : __ : __ : __ : __
• Tipo de dirección: [UNICAST/BROADCAST/MULTICAST]
• Tipo de trama: [IEEE 802.3/Ethernet II]
• LEN/ETHERTYPE: [___ bytes / ___]
• VLAN: [SÍ (ID: ___) / NO]

╔════════════════════════════════════════════════════════════╗
║                        CAPA 3 - RED                       ║
╚════════════════════════════════════════════════════════════╝
• IP Origen: ___.___.___.___
• IP Destino: ___.___.___.___
• Protocolo encapsulado: [TCP/UDP/ICMP]
• Tipo PDU: [PDU única/Primer fragmento/Fragmento intermedio/Último fragmento]

╔════════════════════════════════════════════════════════════╗
║                    CAPA 4 - TRANSPORTE                    ║
╚════════════════════════════════════════════════════════════╝
• Puerto Origen: _____
• Puerto Destino: _____
• Protocolo/Servicio: [HTTP/HTTPS/DNS/FTP/SSH/etc.]
```

### Capas Incluidas por Protocolo
| Protocolo | Capas | Observación |
|-----------|-------|-------------|
| ARP/RARP | Solo Capa 2 | No hay IP ni puertos |
| ICMP | Capas 2 y 3 | No hay puertos |
| TCP/UDP | Capas 2, 3 y 4 | Incluye puertos |

---

## 🚨 Aclaraciones Importantes

### 1️⃣ Unicast/Multicast/Broadcast
**REGLA:** Solo mira el **PRIMER BYTE** de la MAC DESTINO

| MAC Destino | Tipo | Regla |
|-------------|------|-------|
| FF:FF:FF:FF:FF:FF | BROADCAST | Todos los F |
| 01:XX:XX:XX:XX:XX | MULTICAST | Primer byte **IMPAR** |
| 00:XX:XX:XX:XX:XX | UNICAST | Primer byte **PAR** |

**Ejemplos:**
- `00:1b:21:3a:57:c8` → UNICAST (00 es par)
- `01:80:c2:00:00:00` → MULTICAST (01 es impar)
- `ff:ff:ff:ff:ff:ff` → BROADCAST

### 2️⃣ IPs en la Trama
**IMPORTANTE:** Las IPs en bytes 26-33 **YA ESTÁN EN DECIMAL** - NO convertir.

Ejemplo:
- Bytes 26-29: `c0 a8 01 64` = `192.168.1.100`
- Bytes 30-33: `c0 a8 01 65` = `192.168.1.101`

### 3️⃣ Puertos TCP/UDP
**IMPORTANTE:** Cada puerto son **2 bytes** (4 dígitos hex) → **convertir a decimal**

Ejemplos:
- `04 d2` = 0x04d2 = 1234
- `00 50` = 0x0050 = 80 (HTTP)
- `01 bb` = 0x01bb = 443 (HTTPS)

### 4️⃣ Identificar Servicio por Puerto
**REGLA:**
- **Puerto destino conocido** → Cliente conectándose al servicio
- **Puerto origen conocido** → Respuesta del servidor
- **Puerto > 1023** → Puerto dinámico (usualmente cliente)

---

## 🏷️ VLAN Tags

### Detección de VLAN
**REGLA SIMPLE:** Bytes 12-13 = `81 00` → HAY VLAN

### Calcular VLAN ID
**FÓRMULA FÁCIL:** `VLAN ID = Byte 15 + (Byte 14 × 256)`

Ejemplos:
- `00 0A` → VLAN 10
- `00 64` → VLAN 100
- `01 2C` → VLAN 300

### Desplazamiento por VLAN
**⚠️ IMPORTANTE:** Si hay VLAN, **SUMAR +4** a todas las posiciones posteriores

| Campo | Sin VLAN | Con VLAN | Diferencia |
|-------|----------|----------|------------|
| EtherType Real | 12-13 | 16-17 | +4 bytes |
| IP Origen | 26-29 | 30-33 | +4 bytes |
| IP Destino | 30-33 | 34-37 | +4 bytes |
| Puertos | 34-37 | 38-41 | +4 bytes |

### Analogía VLAN
**Piensa en una casa:**
- Sin VLAN: `Puerta → Sala → Cocina → Habitación`
- Con VLAN: `Puerta → GARAJE → Sala → Cocina → Habitación`

El garaje se **inserta** y todo se mueve hacia atrás.

### Casos Reales VLAN
| VLAN Tag | VLAN ID | Uso típico |
|----------|---------|------------|
| 81 00 00 01 | VLAN 1 | VLAN por defecto |
| 81 00 00 0A | VLAN 10 | VLAN de usuarios |
| 81 00 00 64 | VLAN 100 | VLAN de invitados |
| 81 00 01 F4 | VLAN 500 | VLAN de administración |

---

## 🚩 Fragmentación IP

### Ubicación: Bytes 20-21 (Flags + Fragment Offset)

### Flags Importantes
- **DF (Don't Fragment):** Bit 14 - ¿Se puede fragmentar?
- **MF (More Fragments):** Bit 13 - ¿Hay más fragmentos?

### Identificación Rápida por Primer Dígito Hex

| Patrón | DF | MF | Offset | Tipo |
|--------|----|----|--------|------|
| 40 XX | 1 | 0 | 0 | PDU única (no fragmentado) |
| 20 XX | 0 | 1 | 0 | Primer fragmento |
| 20 XX | 0 | 1 | >0 | Fragmento intermedio |
| 00 XX | 0 | 0 | >0 | Último fragmento |

### Trucos para Recordar
- **4X XX** = Paquete entero (único)
- **2X XX** = Hay más paquetes después
- **0X XX** = Último paquete

### Analogía Fragmentación
**Piensa en un paquete postal:**
- **DF=1** → "No fraccionar" (como "Frágil")
- **MF=1** → "Hay más paquetes" (como "1 de 3", "2 de 3")

---

## 🎯 Machete de Referencia Rápida

### Algoritmo de Identificación
```
1️⃣ Leer bytes 12-13 (EtherType/Length):
   ├─ ≤ 1500 → IEEE 802.3
   ├─ 0x8100 → VLAN (leer bytes 16-17 para EtherType real)
   └─ > 1500 → Ethernet II → ir a paso 2

2️⃣ Si Ethernet II, identificar por EtherType:
   ├─ 0x0800 → IPv4 → ir a paso 3
   ├─ 0x0806 → ARP
   ├─ 0x8035 → RARP
   └─ 0x86DD → IPv6

3️⃣ Si IPv4, leer byte 23 (Protocol):
   ├─ 1 → ICMP
   ├─ 6 → TCP → ir a paso 4
   └─ 17 → UDP → ir a paso 4

4️⃣ Si TCP/UDP, leer puertos (bytes 34-37 de la trama):
   ├─ Puerto 53 → DNS
   ├─ Puerto 80 → HTTP
   ├─ Puerto 443 → HTTPS
   └─ Otros puertos → ver tabla de servicios
```

### Reglas de Oro
| Situación | Regla | Ejemplo |
|-----------|-------|---------|
| **VLAN** | 81 00 en bytes 12-13 | VLAN presente |
| **Unicast** | Primer byte MAC par | 00, 02, 04... |
| **Multicast** | Primer byte MAC impar | 01, 03, 05... |
| **Broadcast** | MAC = FF:FF:FF:FF:FF:FF | Broadcast |
| **No fragmentado** | Bytes 20-21 = 40 XX | PDU única |
| **Puerto dinámico** | Puerto > 1023 | Cliente |

### Conversiones Rápidas
- **Puertos:** 2 bytes hex → decimal
- **IPs:** Ya en decimal, leer directo
- **VLAN ID:** Byte 15 + (Byte 14 × 256)

---

## 📋 Ejemplos Prácticos

### Ejemplo 1: Trama IPv4 TCP
```
Trama: 00 e0 b1 cb cd 70 94 de 80 79 c6 12 08 00 45 00 00 28 0b 04 40 00 80 06 10 88 05 00 02 59 34 55 a3 96 ff fc 01 bb
```

**Análisis:**
- **MAC Destino:** 00:e0:b1:cb:cd:70 (UNICAST - 00 es par)
- **MAC Origen:** 94:de:80:79:c6:12
- **Bytes 12-13:** 08 00 = IPv4 (Ethernet II)
- **VLAN:** NO (no es 81 00)
- **Byte 23:** 06 = TCP
- **Bytes 20-21:** 40 00 = PDU única
- **IP Origen:** 5.0.2.89
- **IP Destino:** 52.85.163.150
- **Puerto Origen:** ff fc = 65532
- **Puerto Destino:** 01 bb = 443 (HTTPS)

### Ejemplo 2: Trama ARP con VLAN
```
Trama: ff ff ff ff ff ff 00 50 56 b3 60 df 81 00 00 64 08 06 00 01 08 00 06 04 00 01
```

**Análisis:**
- **MAC Destino:** ff:ff:ff:ff:ff:ff (BROADCAST)
- **MAC Origen:** 00:50:56:b3:60:df
- **Bytes 12-13:** 81 00 = VLAN presente
- **VLAN ID:** 00 64 = 100
- **EtherType real (16-17):** 08 06 = ARP
- **Sin capas 3 y 4** (es ARP)

### Ejemplo 3: Trama IEEE 802.3
```
Trama: aa bb cc dd ee ff 11 22 33 44 55 66 00 2e ff ff 03 00...
```

**Análisis:**
- **MAC Destino:** aa:bb:cc:dd:ee:ff (UNICAST - aa=170 es par)
- **MAC Origen:** 11:22:33:44:55:66
- **Bytes 12-13:** 00 2e = 46 ≤ 1500 → IEEE 802.3
- **Length:** 46 bytes
- **Solo Capa 2** (no analizar más)

---

## 💡 Tips Importantes

### ✅ Qué Hacer
- Siempre verificar VLAN primero (bytes 12-13)
- Leer IPs directamente en decimal
- Convertir puertos de hex a decimal
- Usar primer dígito hex para fragmentación
- Identificar servicios por puertos conocidos

### ❌ Qué NO Hacer
- No convertir IPs de hex a decimal (ya están en decimal)
- No analizar capa 4 en ARP/RARP
- No buscar puertos en ICMP
- No olvidar el desplazamiento +4 con VLAN
- No confundir EtherType con Length

### 🎯 Trucos de Memoria
- **FF:FF:FF:FF:FF:FF** = "Todos los F = Broadcast"
- **81 00** = "Etiqueta VLAN"
- **40 XX** = "4 = Fragmento único"
- **443** = "HTTPS seguro"
- **80** = "HTTP simple"
