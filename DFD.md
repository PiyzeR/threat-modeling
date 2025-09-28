# Threat Modeling — VPS (Debian) + WireGuard + SSH + Syncthing

## Scope
- VPS: <HOSTNAME>, публичный IP: <PUBLIC_IP>.
- Интерфейсы: `ens3` (публичный), `wg0` (VPN <VPN_IP>/24).
- Сервисы (по снимку): WireGuard (udp <WG_PORT>), SSH (tcp 22), Syncthing (tcp 22000, udp 21027, GUI tcp 8384).
- Мы **не вносим изменений** на сервере; задача — модель и приоритизация.

## Trust Boundaries
- TB1: Интернет <--> Периметр VPS (сетевой стек/фаервол).
- TB2: Периметр <--> Аутентифицированные каналы (WireGuard).
- TB3: Процессы <--> Файловая система (ключи/логи/конфиги).

## Узлы
- U1 Клиенты — WG/Syncthing peers.
- U2 Интернет (недоверенная зона).
- U3 VPS Perimeter (`ens3`).
- U4 WireGuard `wg0` (<VPN_IP>/24).
- U5 SSH daemon (sshd).
- U6 Syncthing daemon + GUI.
- D1 Файловая система (/etc/wireguard, ~/.ssh, /var/log, каталоги синка).

## DFD L0
Клиенты ---(WG/UDP <WG_PORT>)---> [VPS: wg0] --> [локальные сервисы]
Клиенты ---(SSH/TCP 22)---> [VPS: sshd]
Клиенты ---(Syncthing TCP 22000 / UDP 21027)---> [VPS: Syncthing]
Клиенты/Интернет ---(HTTP TCP 8384)---> [VPS: Syncthing GUI]

**Data at rest:** ключи WG/SSH, конфиги, логи, каталоги синхронизации.

## DFD L1 (потоки)
- F1: U1 --> U4 — WG handshake + трафик (шифр).
- F2: U1 --> U5 — SSH админ-доступ (ключи/пароли по политике владельца).
- F3: U1 <--> U6 — Syncthing p2p (данные).
- F4: U2/U1 --> U6 (GUI) — админ-панель синка.
- F5: U* --> D1 — операции с ключами/логами/конфигами.

## Environment Snapshot (подробнее в snapshot.txt)
Ports: 22/tcp (sshd), <WG_PORT>/udp (wg0), 22000/tcp, 21027/udp, 8384/tcp
WG: interface wg0 <VPN_IP>/24; peers: <N_PEERS>; keys: <WG_KEY> (ред.)
Firewall: not configured/unknown
SSH policy: PermitRootLogin <AS_IS>

## Privacy Note
Намеренно были скрыты публичные IP/hostnames, публичные ключи WG, VPN-адреса пиров и иные потенциально чувствительные поля. В репозиторий попадают только обобщённые/замаскированные выдержки.
