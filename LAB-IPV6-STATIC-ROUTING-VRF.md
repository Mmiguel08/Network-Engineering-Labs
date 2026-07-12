# Laboratório de Redes - VRFs IPv6 com Switches e Roteadores

Este repositório contém um exemplo de laboratório de rede com foco em VLANs, VRFs e roteamento IPv6 entre dispositivos Cisco. O cenário foi montado para demonstrar a separação lógica de tráfego em duas VRFs, com switches de acesso, roteadores e hosts virtuais.

## Objetivo

O laboratório tem como objetivo ilustrar:

- Configuração de VLANs em switches;
- Uso de trunks entre switches e roteadores;
- Criação de VRFs no roteador;
- Encapsulamento dot1Q em subinterfaces;
- Roteamento IPv6 entre redes isoladas por VRF;
- Conectividade entre hosts em diferentes redes IPv6.

## Topologia

A topologia é composta por:
<image src="Netlabs.jpeg">TOPOLOGIA</image>
- 2 switches Layer 2: SW-SWITCH-01 e SW-SWITCH-02
- 2 roteadores: R-NORTE e R-SUL
- 2 hosts/PCs: VPC-1 e VPC-2

### Visão geral do cenário

- VLAN 10: VRF-ASHELGA-01
- VLAN 20: VRF-ASHELGA-02
- Cada roteador possui subinterfaces dot1Q para cada VLAN
- As VRFs são usadas para isolar os fluxos de rede por cliente/ambiente

## VLANs e VRFs

| Item    | Valor          |
| ------- | -------------- |
| VLAN 10 | VRF-ASHELGA-01 |
| VLAN 20 | VRF-ASHELGA-02 |
| VRF 1   | VRF-ASHELGA-01 |
| VRF 2   | VRF-ASHELGA-02 |

## Tabela de endereçamento IPv6

| Dispositivo | Interface | Rede / Prefixo   | Endereço           |
| ----------- | --------- | ---------------- | ------------------ |
| R-NORTE     | e0/0.10   | 2001:db8:15::/64 | 2001:db8:15::1/64  |
| R-NORTE     | e0/0.20   | 2001:db8:16::/64 | 2001:db8:16::1/64  |
| R-NORTE     | e0/1.10   | 2001:db8:10::/64 | 2001:db8:10::1/64  |
| R-NORTE     | e0/1.20   | 2001:db8:10::/64 | 2001:db8:10::1/64  |
| R-SUL       | e0/0.10   | 2001:db8:15::/64 | 2001:db8:15::2/64  |
| R-SUL       | e0/0.20   | 2001:db8:16::/64 | 2001:db8:16::2/64  |
| R-SUL       | e0/1.10   | 2001:db8:14::/64 | 2001:db8:14::1/64  |
| R-SUL       | e0/1.20   | 2001:db8:13::/64 | 2001:db8:13::1/64  |
| VPC-1       | eth0      | 2001:db8:10::/64 | 2001:db8:10::10/64 |
| VPC-2       | eth0      | 2001:db8:14::/64 | 2001:db8:14::10/64 |

## Configuração básica dos switches

### SW-SWITCH-01

```text
enable
conf t
hostname SW-SWITCH-01

vlan 10
 name VRF-ASHELGA-01
exit

vlan 20
 name VRF-ASHELGA-02
exit

interface e0/0
 description TRUNK_TO_R-NORTE
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
exit

interface e0/1
 description WIN10_VLAN20
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 no shutdown
exit

interface e0/2
 description PC1_VLAN10
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 no shutdown
exit

end
wr
```

### SW-SWITCH-02

```text
enable
conf t
hostname SW-SWITCH-02

vlan 10
 name VRF-ASHELGA-01
exit

vlan 20
 name VRF-ASHELGA-02
exit

interface e0/0
 description TRUNK_TO_R-SUL
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
exit

interface e0/1
 description WIN10_VLAN20
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 no shutdown
exit

interface e0/2
 description PC2_VLAN10
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 no shutdown
exit

end
wr
```

## Configuração básica dos roteadores

### R-NORTE

```text
enable
conf t
hostname R-NORTE

ipv6 unicast-routing

vrf definition VRF-ASHELGA-01
 address-family ipv6
 exit-address-family
exit

vrf definition VRF-ASHELGA-02
 address-family ipv6
 exit-address-family
exit

interface e0/0
 no shutdown
exit

interface e0/0.10
 encapsulation dot1Q 10
 vrf forwarding VRF-ASHELGA-01
 ipv6 address 2001:db8:15::1/64
exit

interface e0/0.20
 encapsulation dot1Q 20
 vrf forwarding VRF-ASHELGA-02
 ipv6 address 2001:db8:16::1/64
exit

interface e0/1
 no shutdown
exit

interface e0/1.10
 encapsulation dot1Q 10
 vrf forwarding VRF-ASHELGA-01
 ipv6 address 2001:db8:10::1/64
exit

interface e0/1.20
 encapsulation dot1Q 20
 vrf forwarding VRF-ASHELGA-02
 ipv6 address 2001:db8:10::1/64
exit

ipv6 route vrf VRF-ASHELGA-01 2001:db8:14::/64 2001:db8:15::2

ipv6 route vrf VRF-ASHELGA-02 2001:db8:13::/64 2001:db8:16::2

end
wr
```

### R-SUL

```text
enable
conf t
hostname R-SUL

ipv6 unicast-routing

vrf definition VRF-ASHELGA-01
 address-family ipv6
 exit-address-family
exit

vrf definition VRF-ASHELGA-02
 address-family ipv6
 exit-address-family
exit

interface e0/0
 no shutdown
exit

interface e0/0.10
 encapsulation dot1Q 10
 vrf forwarding VRF-ASHELGA-01
 ipv6 address 2001:db8:15::2/64
exit

interface e0/0.20
 encapsulation dot1Q 20
 vrf forwarding VRF-ASHELGA-02
 ipv6 address 2001:db8:16::2/64
exit

interface e0/1
 no shutdown
exit

interface e0/1.10
 encapsulation dot1Q 10
 vrf forwarding VRF-ASHELGA-01
 ipv6 address 2001:db8:14::1/64
exit

interface e0/1.20
 encapsulation dot1Q 20
 vrf forwarding VRF-ASHELGA-02
 ipv6 address 2001:db8:13::1/64
exit

ipv6 route vrf VRF-ASHELGA-01 2001:db8:10::/64 2001:db8:15::1

ipv6 route vrf VRF-ASHELGA-02 2001:db8:10::/64 2001:db8:16::1

end
wr
```

## Configuração dos hosts

### VPC-1

```text
ip 2001:db8:10::10/64 2001:db8:10::1
```

### VPC-2

```text
ip 2001:db8:14::10/64 2001:db8:14::1
```

## Validação do laboratório

Após aplicar as configurações, você pode validar a conectividade com comandos como:

```text
show ipv6 route vrf [Nome da vrf]
show vrf
show interfaces trunk
ping ipv6 2001:db8:14::10
```

## Observações

- Este laboratório é um exemplo didático e pode ser adaptado para outros cenários.
- Os nomes de interface podem variar conforme o emulador ou equipamento utilizado.
- Para ambientes reais, é recomendável revisar detalhes de segurança, redundância e padronização de endereçamento.
