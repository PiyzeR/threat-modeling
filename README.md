# Threat Modeling (VPS)
DFD L0/L1 + STRIDE (8 пунктов) для VPS с WireGuard/SSH. Работа выполнена в режиме read-only: я не менял конфигурацию сервера, а оформил модель угроз и приоритеты. Высокие риски: публичный SSH с root-логином и открытая извне Syncthing GUI. Рекомендации описаны в `/docs/ActionPlan.md`. Чувствительные данные в снапшотах замаскированы.

## DFD L0
```mermaid
flowchart LR
  subgraph Internet [Интернет (недоверенная зона)]
    U2[Сканеры/клиенты]
  end

  subgraph VPS [VPS <HOSTNAME>]
    direction TB
    P[Периметр/сетевой стек (ens3)]
    WG[(WireGuard wg0<br/><VPN_IP>/24)]
    SSH[sshd (TCP 22)]
    ST[Syncthing (TCP 22000 / UDP 21027)]
    GUI[Syncthing GUI (TCP 8384)]
  end

  U2 -- "UDP <WG_PORT>" --> P --> WG
  U2 -- "TCP 22" --> P --> SSH
  U2 -- "TCP 22000 / UDP 21027" --> P --> ST
  U2 -- "TCP 8384 (HTTP)" --> P --> GUI

  classDef boundary stroke-dasharray: 5 5, stroke:#777;
  class P boundary;
```

## DFD L1
```mermaid
flowchart TB
  U1[Клиентские устройства<br/>(WG/Syncthing peers)]
  subgraph TB1 [TB1: Интернет ↔ Периметр]
    P[Периметр VPS (ens3)]
  end
  subgraph TB2 [TB2: Периметр ↔ Аутентифицированные каналы]
    WG[(WireGuard wg0<br/><VPN_IP>/24)]
  end
  SSH[sshd (TCP 22)]
  ST[Syncthing (TCP 22000 / UDP 21027)]
  GUI[Syncthing GUI (TCP 8384)]
  D1[(Файловая система:<br/>/etc/wireguard, ~/.ssh, /var/log, каталоги синка)]

  U1 -- "UDP <WG_PORT> (handshake+data)" --> P --> WG
  U1 -- "TCP 22 (SSH)" --> P --> SSH
  U1 -- "TCP 22000 / UDP 21027" --> P --> ST
  U1 -- "TCP 8384 (HTTP GUI)" --> P --> GUI

  WG --> SSH:::intra
  ST --> D1:::data
  SSH --> D1:::data

  classDef boundary stroke-dasharray: 5 5, stroke:#777;
  class TB1,TB2 boundary;
  classDef intra stroke:#999;
  classDef data stroke:#999;
```
