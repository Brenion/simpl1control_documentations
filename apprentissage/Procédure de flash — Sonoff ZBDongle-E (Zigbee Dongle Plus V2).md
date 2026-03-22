
## Glossaire

| Terme             | Définition                                                                                                                                        |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **EFR32MG21**     | Puce Silicon Labs utilisée dans le ZBDongle-E. Supporte Zigbee, Thread et Matter.                                                                 |
| **EZSP**          | *Ember Zigbee Serial Protocol* — protocole de communication entre le dongle et le logiciel hôte (Z2M).                                            |
| **EmberZNet NCP** | *Network Co-Processor* — firmware qui transforme le dongle en coordinateur Zigbee piloté via EZSP. C'est le firmware recommandé pour Zigbee2MQTT. |
| **NCP**           | *Network Co-Processor* — le dongle gère la couche radio, le logiciel hôte gère la logique réseau.                                                 |
| **RCP**           | *Radio Co-Processor* — mode alternatif où le dongle gère uniquement la radio. Requis pour Multi-PAN (Zigbee + Thread simultané). Non retenu ici.  |
| **Multi-PAN**     | Firmware permettant Zigbee + Thread/Matter sur le même dongle. Abandonné par Home Assistant, non recommandé pour Z2M.                             |
| **Bootloader**    | Programme minimal embarqué sur le dongle permettant de recevoir un nouveau firmware sans équipement spécial.                                      |
| **GBL**           | Format de fichier firmware Silicon Labs (`.gbl`).                                                                                                 |
| **Z2M**           | Zigbee2MQTT — passerelle entre le réseau Zigbee et un broker MQTT.                                                                                |

---

## Matériel

- **Dongle :** Sonoff Zigbee 3.0 USB Dongle Plus V2 (ZBDongle-E)

- **Puce :** EFR32MG21

- **Port macOS :** `/dev/cu.usbserial-XXXX` (varie à chaque branchement)

---

## Firmware installé

| Paramètre     | Valeur                          |
| ------------- | ------------------------------- |
| **Type**      | EmberZNet NCP (EZSP)            |
| **Version**   | 7.4.4.0 build 0                 |
| **Baud rate** | 115200                          |
| **Source**    | darkxst/silabs-firmware-builder |

---

## Outil de flash

**Site web :** https://darkxst.github.io/silabs-firmware-builder/

Outil web basé sur la **Web Serial API** — fonctionne uniquement sur **Chrome ou Edge** (pas Safari, pas Firefox).

---

## Procédure

### 1. Identifier le port USB

Brancher le dongle, puis dans le terminal :

ls /dev/cu.usb*

Débrancher/rebrancher pour confirmer quel port correspond au dongle (il apparaît/disparaît).

### 2. Flash via navigateur

1. Ouvrir Chrome ou Edge
2. Aller sur `https://darkxst.github.io/silabs-firmware-builder/`
3. Section ZBDongle-E → cliquer Connect
4. Sélectionner le port `/dev/cu.usbserial-XXXX` dans la popup
5. Sélectionner le firmware EZSP (= EmberZNet NCP)
6. Cliquer Flash — ne pas débrancher pendant l'opération (~1-2 min)

> Le dongle change de port USB pendant le flash (comportement normal du bootloader). Ne pas intervenir.

### 3. Vérification

Une fois le flash terminé, le site affiche :

Zigbee (EZSP) 7.4.4.0 build 0

Sonoff ZBDongle-E

C'est la confirmation que le firmware est correctement installé.

---

## Choix du firmware — décision documentée

|Firmware|Cas d'usage|Retenu|
|---|---|---|
|EZSP (EmberZNet NCP)|Coordinateur Zigbee pour Z2M|✅|
|RCP Multi-PAN|Zigbee + Thread simultané (HA uniquement)|❌ Abandonné par HA|
|OpenThread RCP|Thread/Matter uniquement|❌|

Raison du choix EZSP : Z2M utilise nativement le driver `ember` (EZSP). Stable, maintenu activement depuis 2024 (migration de `ezsp` → `ember`). Le support Matter sera assuré par un pont Matter dédié le moment venu.

---

## Notes d'isolation (environnement dev)

Ce dongle est dédié au développement. Un second système domotique (Home Assistant + autre dongle Sonoff) tourne sur le même réseau local.

Isolation garantie par :

- Chaque coordinateur Zigbee crée son propre réseau avec un PAN ID unique — pas d'interférence radio.
- Le Z2M dev pointe sur le broker Mosquitto du projet (`docker-compose.yml`) — pas sur le broker de Home Assistant.
- Canal Zigbee à configurer différent de celui de HA lors du setup Z2M.