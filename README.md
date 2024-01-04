# MQTT Broker med Docker och Macvlan.

Denna konfiguration beskriver uppsättningen av en Mosquitto MQTT-broker som körs i en Docker-container och använder ett macvlan-nätverk för att tillåta direkt kommunikation med andra enheter på det lokala nätverket.

## Docker Compose

Tjänsten är definierad i docker-compose.yml och inkluderar en volymkonfiguration som lagrar data, loggar och konfigurationsfiler på värdmaskinen för att säkerställa att viktig information bevaras mellan containeromstarter.

### Tjänster

- mosquitto: Den primära MQTT-broker-tjänsten som använder den officiella eclipse-mosquitto-bilden.

### Volymer

- ./config/mosquitto.conf: Den lokala konfigurationsfilen som är monterad inuti containern.
- ./data: En katalog för att lagra persistent data som meddelanden och klientinformation.
- ./log: En katalog för att lagra loggfilerna för Mosquitto-brokern.

### Portar

- "1883:1883": Standard MQTT-port.
- "9001:9001": MQTT över WebSocket-port.

### Omstartspolicy

- restart: unless-stopped: Container tjänsten kommer automatiskt att starta om om den kraschar eller om Docker daemon startas om, såvida inte containern har stoppats manuellt.

## Macvlan Nätverkskonfiguration

Macvlan-nätverket möjliggör att containrar visas som fysiska enheter på ditt lokala nätverk, vilket tillåter direkt kommunikation med andra enheter utan behov av portmappning.

### Hur man skapar ett Macvlan Nätverk

För att skapa ett macvlan-nätverk på din Docker-värd, använd följande kommando:

```console
docker network create -d macvlan \
  --subnet=192.168.100.0/24 \
  --gateway=192.168.100.1 \
  -o parent=eth0 my_macvlan_net
```

Byt ut '--subnet' och '--gateway' med dina faktiska nätverksinställningar och 'eth0' med det relevanta nätverksinterfacet på din värd.

### Användning i Docker Compose

För att använda det macvlan-nätverket i din docker-compose.yml, definiera nätverket under networks och associera det med din tjänst.

## Användning

För att starta din Mosquitto MQTT-broker, navigera till katalogen med docker-compose.yml och kör:

docker-compose up -d

För att stoppa och ta bort containern och nätverket, kör:

```console
docker-compose down
```

## Testa din Broker

Använd mosquitto_pub och mosquitto_sub för att testa att publicera och prenumerera på meddelanden:

```console
mosquitto_sub -h 192.168.100.200 -t "test/topic"
mosquitto_pub -h 192.168.100.200 -t "test/topic" -m "Hello MQTT"
```
