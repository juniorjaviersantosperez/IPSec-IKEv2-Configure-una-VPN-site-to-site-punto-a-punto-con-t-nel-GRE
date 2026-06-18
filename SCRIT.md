# IPSec IKEv2
## Configure una VPN site to site punto a punto con túnel GRE

---

## Paso 1 — Configuración de interfaces

**R2**
```
conf t
interface e0/0
 ip address 200.1.15.2 255.255.255.0
 no shut
interface e0/1
 ip address 10.15.99.1 255.255.255.0
 no shut
ip route 0.0.0.0 0.0.0.0 200.1.15.1
```

**R3**
```
conf t
interface e0/0
 ip address 200.1.99.2 255.255.255.0
 no shut
interface e0/1
 ip address 192.168.99.1 255.255.255.0
 no shut
ip route 0.0.0.0 0.0.0.0 200.1.99.1
```

---

## Paso 2 — Propuesta IKEv2

**R2**
```
crypto ikev2 proposal IKEv2-PROP
 encryption aes-cbc-128
 integrity sha1
 group 2
```

**R3**
```
crypto ikev2 proposal IKEv2-PROP
 encryption aes-cbc-128
 integrity sha1
 group 2
```

---

## Paso 3 — Política IKEv2

**R2**
```
crypto ikev2 policy IKEv2-POL
 proposal IKEv2-PROP
```

**R3**
```
crypto ikev2 policy IKEv2-POL
 proposal IKEv2-PROP
```

---

## Paso 4 — Keyring (pre-shared key)

**R2**
```
crypto ikev2 keyring GRE-KEYS
 peer R3
  address 200.1.99.2
  pre-shared-key cisco
```

**R3**
```
crypto ikev2 keyring GRE-KEYS
 peer R2
  address 200.1.15.2
  pre-shared-key cisco
```

---

## Paso 5 — Perfil IKEv2

**R2**
```
crypto ikev2 profile IKEv2-PROF
 match identity remote address 200.1.99.2
 authentication local pre-share
 authentication remote pre-share
 keyring local GRE-KEYS
```

**R3**
```
crypto ikev2 profile IKEv2-PROF
 match identity remote address 200.1.15.2
 authentication local pre-share
 authentication remote pre-share
 keyring local GRE-KEYS
```

---

## Paso 6 — IPSec transform-set y perfil IPSec

**R2**
```
crypto ipsec transform-set GRE-SET esp-aes esp-sha-hmac
 mode transport
!
crypto ipsec profile GRE-PROFILE
 set transform-set GRE-SET
 set ikev2-profile IKEv2-PROF
```

**R3**
```
crypto ipsec transform-set GRE-SET esp-aes esp-sha-hmac
 mode transport
!
crypto ipsec profile GRE-PROFILE
 set transform-set GRE-SET
 set ikev2-profile IKEv2-PROF
```

---

## Paso 7 — Interfaz Túnel GRE

**R2**
```
interface Tunnel0
 ip address 172.16.1.1 255.255.255.252
 tunnel source 200.1.15.2
 tunnel destination 200.1.99.2
 tunnel mode gre ip
 tunnel protection ipsec profile GRE-PROFILE
```

**R3**
```
interface Tunnel0
 ip address 172.16.1.2 255.255.255.252
 tunnel source 200.1.99.2
 tunnel destination 200.1.15.2
 tunnel mode gre ip
 tunnel protection ipsec profile GRE-PROFILE
```

---

## Paso 8 — Enrutamiento por el túnel GRE

**R2**
```
ip route 192.168.99.0 255.255.255.0 Tunnel0
```

**R3**
```
ip route 10.15.99.0 255.255.255.0 Tunnel0
```

---

## Paso 9 — Verificación IKEv2

```
show crypto ikev2 sa
```
> Debe mostrar estado `READY` — reemplaza `show crypto isakmp sa`

```
show crypto ikev2 session
```
> Detalle de la sesión IKEv2 con estadísticas

```
show crypto ipsec sa
```
> Igual que IKEv1 — verifica encaps/decaps

```
ping 192.168.99.2 source 10.15.99.1
```
> Conectividad entre LANs por el túnel GRE+IKEv2
