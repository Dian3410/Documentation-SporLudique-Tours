# Test réseau

## Vlan mana

**Test NAT**
- Ping du 10.0.130.2 vers l'adresse ip des servers google

```
ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
From 10.0.130.2 icmp_seq=1 Destination Host Unreachable
```

Le vlan mana permet d'administrer, il ne route donc pas.

**Ip switch**
- Ping du 10.0.130.2 vers 10.0.130.1 (ip switch)

```
ping 10.0.130.1
PING 10.0.130.1 (10.0.130.1) 56(84) bytes of data.
64 bytes from 10.0.130.1: icmp_seq=1 ttl=254 time=1.21 ms
```

## Vlan server

**Server CORE**
- Ping du 172.28.65.120 vers 172.28.65.254 (gateway)

```
ping 172.28.65.254
Envoi d’une requête 'Ping' 172.28.65.254 avec 32 octets de données :
Réponse de 172.28.65.254 : octets=32 temps=1 ms TTL=254
```

**Server GUI**
- Ping du 172.28.65.115 vers 172.28.65.254 (gateway)

```
ping 172.28.65.254
Envoi d’une requête 'Ping' 172.28.65.254 avec 32 octets de données :
Réponse de 172.28.65.254 : octets=32 temps=1 ms TTL=254
```


## Vlan clients

**Poste client** 
- Ping du 172.28.64.2 vers 172.28.64.254 (gateway)

```
ping 172.28.64.254
PING 172.28.64.254 (172.28.64.254) 56(84) bytes of data.
64 bytes from 172.28.64.254: icmp_seq=2 ttl=254 time=1.26 ms
```

## Vlan interco_WAN

**Firewall** 
- Ping du 192.168.230.1 vers 192.168.230.251
- Ping du 192.168.230.1 vers 192.168.230.252
- Ping du 192.168.230.1 vers 192.168.230.254

```
ping gateway (FAI1):
SYSTEM PING HOST=192.168.230.251
[section]
transmitted=1
received=1
packet_size=64
time=0.71
```

```
ping gateway (FAI2):
SYSTEM PING HOST=192.168.230.252
[section]
transmitted=1
received=1
packet_size=64
time=1.88
```

```
ping gateway (HSRP)
SYSTEM PING HOST=192.168.230.254
[section](vlan client)
transmitted=1
received=1
packet_size=64
time=0.71
```

## Vlan dmz

**Dns autoritaire**
- ping du 192.168.233.1 vers la gateway dmz

```
PING 192.168.233.254 (192.168.233.254) 56(84) bytes of data.
64 bytes from 192.168.233.254: icmp_seq=1 ttl=64 time=1.34 ms

```

## Vlan interco_LAN

**Poste client**
- Ping du 172.28.64.2 vers la svi du vlan interco_LAN

```
ping 192.168.234.1
PING 192.168.234.1 (192.168.234.1) 56(84) bytes of data.
64 bytes from 192.168.234.1: icmp_seq=1 ttl=254 time=1.21 ms
```

- Ping du 172.28.64.2 vers la gateway interco_LAN (firewall)

```
ping 192.168.234.254
PING 192.168.234.254 (192.168.234.254) 56(84) bytes of data.
64 bytes from 192.168.234.254: icmp_seq=1 ttl=63 time=1.94 ms
```