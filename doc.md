# ⚔️ Lab Takedown
## Recriando a Caçada de Shimomura em 2026

> **1995:** antenas, telefones, logs e muita paciência.
> **2026:** containers, ELK, Nmap, Scapy, PyShark e OSINT.
>
> **A tecnologia mudou. O raciocínio investigativo, não.**

> [!WARNING]
> **Ambiente 100% simulado.** Sem alvos reais, sem coleta de dados de terceiros, sem uso contra redes de terceiros. Ver o aviso legal no final deste documento.

### 🧪 Visão rápida

| Item | Detalhe |
|---|---|
| **Tipo** | Laboratório educacional de Cybersecurity |
| **Execução** | Docker / Docker Compose |
| **Dados** | 100% fictícios e sintéticos |
| **Escopo** | Ambiente local e isolado |
| **Camadas** | 5 módulos de investigação |

## 🧭 Índice

- [Arquitetura do Lab](#-arquitetura-do-lab)
- [Requisitos](#-requisitos)
- [Estrutura de pastas](#-estrutura-de-pastas)
- [Módulo 1 — Triangulação Física](#-módulo-1--triangulação-física)
- [Módulo 2 — Correlação de Logs](#-módulo-2--correlação-de-logs)
- [Módulo 3 — Reconhecimento Ativo](#-módulo-3--reconhecimento-ativo)
- [Módulo 4 — Metadado de Tráfego](#-módulo-4--metadado-de-tráfego)
- [Módulo 5 — OSINT](#-módulo-5--osint)
- [Docker Compose Consolidado](#-docker-compose-consolidado)
- [Como Executar](#-como-executar-o-laboratório-completo)
- [Lições do Laboratório](#-lições-do-laboratório)
- [Como desligar tudo](#-como-desligar-tudo)
- [Aviso legal](#-aviso-legal)

---

```text
[GERADOR_ATAQUE] --> logs de autenticação falsos
        |
        v
[ELK STACK] --> correlação de eventos (Módulo 2)
        |
[SCAPY/PYSHARK] --> tráfego sintético + análise de metadado (Módulo 4)
        |
[NMAP] --> reconhecimento ativo da própria rede do lab (Módulo 3)
        |
[OSINT SIM] --> busca simulada tipo "dork" em base local (Módulo 5)
        |
[ALVO] --> emite sinal Wi-Fi simulado
   |  |  |
   v  v  v
[SENSOR 1] [SENSOR 2] [SENSOR 3] --> coletam RSSI (Módulo 1)
   |  |  |
   +--+--+
      |
      v
[ANALISTA] --> triangula e gera mapa.html
```

## 🛠️ Requisitos

Docker + Docker Compose

~4 GB de RAM livres

Portas 5601 (Kibana) e 9200 (Elasticsearch) livres

Python 3.10+ (opcional, para rodar módulos fora de container)

## 📁 Estrutura de pastas

```text
takelab/
├── docker-compose.yml
├── logs/
├── output/
├── alvo/
│   └── Dockerfile
├── sensor/
│   ├── Dockerfile
│   └── sensor.py
├── analista/
│   ├── Dockerfile
│   └── triangulate.py
├── gerador/
│   ├── Dockerfile
│   └── gerador.py
├── logstash/
│   └── pipeline/
│       └── logstash.conf
├── recon/
│   ├── Dockerfile
│   └── scanner.py
├── pcap/
│   ├── Dockerfile
│   ├── gerar_trafego.py
│   └── analisar_trafego.py
└── osint/
    ├── Dockerfile
    ├── indice_falso.json
    └── dork_sim.py
```

## 🔎 Módulo 1 — Triangulação Física

> **A “antena Yagi” de 2026.** Simula as três vans do Shimomura usando três containers medindo força de sinal (RSSI simulado) de um alvo fixo.

Simula as três vans do Shimomura usando três containers medindo força de sinal (RSSI simulado) de um alvo fixo.

### 1.1 docker-compose.yml (base – será expandido nos módulos seguintes)

```yaml
version: '3.8'
services:
  alvo:
    build: ./alvo
    networks:
      takelab_net:
        ipv4_address: 172.20.0.10

  sensor1:
    build: ./sensor
    environment:
      SENSOR_ID: sensor1
      POS_X: 0
      POS_Y: 0
    volumes:
      - './logs:/app/logs'
    networks:
      takelab_net:
        ipv4_address: 172.20.0.11

  sensor2:
    build: ./sensor
    environment:
      SENSOR_ID: sensor2
      POS_X: 10
      POS_Y: 0
    volumes:
      - './logs:/app/logs'
    networks:
      takelab_net:
        ipv4_address: 172.20.0.12

  sensor3:
    build: ./sensor
    environment:
      SENSOR_ID: sensor3
      POS_X: 5
      POS_Y: 10
    volumes:
      - './logs:/app/logs'
    networks:
      takelab_net:
        ipv4_address: 172.20.0.13

  analista:
    build: ./analista
    volumes:
      - './logs:/app/logs'
      - './output:/app/output'
    depends_on:
      - sensor1
      - sensor2
      - sensor3
    networks:
      takelab_net:
        ipv4_address: 172.20.0.20

networks:
  takelab_net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
1.2 alvo/Dockerfile
dockerfile
FROM python:3.10-slim
WORKDIR /app
CMD echo "AP Mitnick_WiFi ligado em 172.20.0.10" && tail -f /dev/null
1.3 sensor/Dockerfile
dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY sensor.py .
CMD ["python", "sensor.py"]
1.4 sensor/sensor.py
python
import os, time, json, random, math
from datetime import datetime

SENSOR_ID = os.getenv("SENSOR_ID", "sensor1")
POS = [float(os.getenv("POS_X", 0)), float(os.getenv("POS_Y", 0))]
ALVO_REAL = [4, 3]  # posição secreta simulada do "alvo"

def simular_rssi():
    dist = math.sqrt((POS[0]-ALVO_REAL[0])**2 + (POS[1]-ALVO_REAL[1])**2)
    rssi = -30 - (dist * 5) + random.uniform(-3, 3)
    return round(rssi, 2), round(dist, 2)

while True:
    rssi, dist = simular_rssi()
    data = {"ts": datetime.now().isoformat(), "sensor": SENSOR_ID, "pos": POS, "rssi": rssi, "dist": dist}
    with open(f"/app/logs/{SENSOR_ID}.log", "a") as f:
        f.write(json.dumps(data) + "\n")
    print(f"[{SENSOR_ID}] RSSI: {rssi} dBm | Dist: {dist}m")
    time.sleep(3)
1.5 analista/Dockerfile
dockerfile
FROM python:3.10-slim
WORKDIR /app
RUN pip install numpy scipy folium --no-cache-dir
COPY triangulate.py .
CMD ["python", "triangulate.py"]
1.6 analista/triangulate.py
python
import json, time, math
import numpy as np
from scipy.optimize import least_squares
import folium

def ler_ultimo(sensor):
    try:
        with open(f"/app/logs/{sensor}.log") as f:
            lines = f.readlines()
            if not lines:
                return None
            return json.loads(lines[-1])
    except Exception:
        return None

def trilateration(sensores):
    def eq(vars):
        x, y = vars
        return [math.sqrt((x - s['pos'][0])**2 + (y - s['pos'][1])**2) - s['dist'] for s in sensores]
    res = least_squares(eq, [5, 5])
    return res.x

print("[ANALISTA] Aguardando dados dos 3 sensores...")
while True:
    time.sleep(10)
    sensores = [ler_ultimo(f"sensor{i}") for i in [1, 2, 3]]
    if all(sensores):
        pos = trilateration(sensores)
        print(f"\n[ANALISTA] ALVO ESTIMADO EM: X={pos[0]:.2f}, Y={pos[1]:.2f}")
        print("[ANALISTA] ALVO REAL ESTAVA EM: X=4, Y=3")

        m = folium.Map(location=[5, 5], zoom_start=3, crs='Simple')
        for s in sensores:
            folium.CircleMarker(s['pos'], radius=8, popup=s['sensor'], color='blue').add_to(m)
        folium.Marker([4, 3], popup="Alvo Real", icon=folium.Icon(color='red')).add_to(m)
        folium.Marker(pos.tolist(), popup="Alvo Estimado", icon=folium.Icon(color='green')).add_to(m)
        m.save("/app/output/mapa.html")
        print("[ANALISTA] Mapa salvo em /app/output/mapa.html")
Rodar o Módulo 1
bash
mkdir -p logs output
docker-compose up --build analista
Abra output/mapa.html no navegador.

```

## 🧮 Módulo 2 — Correlação de Logs

> **Estilo Shimomura.** Simula como Shimomura encontrou o padrão de Mitnick correlacionando timestamps e origem de conexão.

Simula como Shimomura encontrou o padrão de Mitnick correlacionando timestamps e origem de conexão.

### 2.1 Adicionar ao docker-compose.yml (integração)

Inclua os seguintes serviços no mesmo arquivo docker-compose.yml:

```yaml
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      discovery.type: single-node
      xpack.security.enabled: "false"
    ports:
      - "9200:9200"
    networks:
      - takelab_net

  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    volumes:
      - './logstash/pipeline:/usr/share/logstash/pipeline'
      - './logs:/logs'
    depends_on:
      - elasticsearch
    networks:
      - takelab_net

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
    networks:
      - takelab_net

  gerador_ataque:
    build: ./gerador
    volumes:
      - './logs:/logs'
    networks:
      - takelab_net
2.2 gerador/Dockerfile
dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY gerador.py .
CMD ["python", "gerador.py"]
2.3 gerador/gerador.py
python
import time, random, datetime

ips = ["201.54.12.1", "200.123.45.9", "189.1.2.3"]
usuarios = ["root", "admin", "shimomura"]

print("[GERADOR] Iniciando simulação de tentativas de acesso...")
while True:
    ip = random.choice(ips)
    user = random.choice(usuarios)
    agora = datetime.datetime.now()
    data_hora = agora.strftime("%Y-%m-%d %H:%M:%S")
    log = f'{data_hora} sshd: Failed password for {user} from {ip} port {random.randint(3000, 6000)}\n'
    with open("/logs/auth.log", "a") as f:
        f.write(log)
    print(f"Tentativa simulada de {ip} como {user}")
    time.sleep(random.uniform(1, 4))
2.4 logstash/pipeline/logstash.conf
text
input {
  file {
    path => "/logs/*.log"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}

filter {
  grok {
    match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} %{WORD:servico}: Failed password for %{USER:usuario} from %{IP:ip_origem}" }
  }
  date {
    match => ["timestamp", "yyyy-MM-dd HH:mm:ss"]
    target => "@timestamp"
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "takedown-logs"
  }
}
Rodar o Módulo 2
bash
docker-compose up -d elasticsearch logstash kibana gerador_ataque
Acesse http://localhost:5601, crie o index pattern takedown-logs e caçe por: usuário shimomura + horário entre 02:00 e 05:00 + múltiplas origens de IP.

```

## 🛰️ Módulo 3 — Reconhecimento Ativo

> **Nmap.** Antes de qualquer caçada, todo analista precisa responder à pergunta mais básica: o que está vivo e o que está exposto na minha rede?

Antes de qualquer caçada, todo analista precisa responder à pergunta mais básica: o que está vivo e o que está exposto na minha rede? Esse container escaneia a própria rede do lab (172.20.0.0/24) — nunca redes externas.

### 3.1 recon/Dockerfile

```dockerfile
FROM python:3.10-slim
WORKDIR /app
RUN apt-get update && apt-get install -y nmap --no-install-recommends && rm -rf /var/lib/apt/lists/*
RUN pip install python-nmap --no-cache-dir
COPY scanner.py .
CMD ["python", "scanner.py"]
3.2 recon/scanner.py
python
import nmap
import json
import time
import os

REDE_DO_LAB = "172.20.0.0/24"  # escopo restrito à própria rede do laboratório

def escanear(rede):
    scanner = nmap.PortScanner()
    print(f"[RECON] Escaneando escopo autorizado: {rede}")
    scanner.scan(hosts=rede, arguments="-sT -T4 --top-ports 100")

    resultado = []
    for host in scanner.all_hosts():
        info = {
            "host": host,
            "status": scanner[host].state(),
            "portas_abertas": [
                p for p in scanner[host].get('tcp', {})
                if scanner[host]['tcp'][p]['state'] == 'open'
            ]
        }
        resultado.append(info)
        print(f"[RECON] {host} -> {info['status']} | portas: {info['portas_abertas']}")

    # Garante que o diretório /app/logs existe
    os.makedirs("/app/logs", exist_ok=True)
    with open("/app/logs/recon.json", "w") as f:
        json.dump(resultado, f, indent=2)

if __name__ == "__main__":
    time.sleep(5)  # espera os outros containers subirem
    escanear(REDE_DO_LAB)
Adicionar ao docker-compose.yml
yaml
  recon:
    build: ./recon
    volumes:
      - './logs:/app/logs'
    network_mode: "service:analista"   # compartilha rede com o analista, mas pode ser 'takelab_net' também
    depends_on:
      - sensor1
      - sensor2
      - sensor3
Rodar isolado, sem depender do resto: docker-compose up --build recon

A lição do módulo: o próprio Shimomura, antes de caçar, primeiro precisou entender a superfície do problema. Reconhecimento não é ataque — é mapa.

```

## 📡 Módulo 4 — Metadado de Tráfego

> **Estilo Wireshark.** O ponto de virada da caçada real foi perceber, pela latência, que Mitnick usava celular clonado.

O ponto de virada da caçada real foi perceber, pela latência, que Mitnick usava celular clonado. Esse módulo recria esse tipo de raciocínio: gera tráfego sintético com Scapy e analisa o metadado — timing, tamanho de pacote, padrão de conexão — com PyShark, exatamente como se faria abrindo uma captura no Wireshark.

### 4.1 pcap/Dockerfile

```dockerfile
FROM python:3.10-slim
WORKDIR /app
RUN apt-get update && apt-get install -y tshark --no-install-recommends && rm -rf /var/lib/apt/lists/*
RUN pip install scapy pyshark --no-cache-dir
COPY gerar_trafego.py analisar_trafego.py .
CMD ["python", "gerar_trafego.py"]   # gera o pcap ao iniciar; depois pode rodar o analisar separadamente
4.2 pcap/gerar_trafego.py
python
from scapy.all import IP, TCP, wrpcap, RandIP
import random
import os

os.makedirs("/app/logs", exist_ok=True)

pacotes = []
origens = ["201.54.12.1", "200.123.45.9", "189.1.2.3"]

# Gera 300 pacotes com assinaturas diferentes
for i in range(300):
    origem = random.choice(origens)
    # Simula latência: para a origem "189.1.2.3", atraso artificial (jitter) será inserido na análise via timestamp
    # Mas aqui apenas geramos pacotes, a análise posterior usará o tempo de captura
    pkt = IP(src=origem, dst="172.20.0.10") / TCP(sport=random.randint(1024, 65535), dport=22)
    pacotes.append(pkt)

wrpcap("/app/logs/trafego_simulado.pcap", pacotes)
print("[PCAP] Tráfego sintético gerado em /app/logs/trafego_simulado.pcap")
4.3 pcap/analisar_trafego.py
python
import pyshark
from collections import defaultdict
import statistics
import os

def analisar(pcap_path):
    cap = pyshark.FileCapture(pcap_path)
    contagem_por_origem = defaultdict(int)
    tamanhos_por_origem = defaultdict(list)

    for pkt in cap:
        try:
            src = pkt.ip.src
            contagem_por_origem[src] += 1
            # Tamanho do pacote (camada IP) – pode não estar disponível em todos; usa length
            if hasattr(pkt, 'length'):
                tamanhos_por_origem[src].append(int(pkt.length))
        except AttributeError:
            continue

    print("[ANÁLISE] Volume de conexões por origem (metadado):")
    for origem, total in sorted(contagem_por_origem.items(), key=lambda x: -x[1]):
        print(f"  {origem}: {total} pacotes")

    print("\n[ANÁLISE] Estatísticas de tamanho de pacote por origem:")
    for origem, tamanhos in tamanhos_por_origem.items():
        if tamanhos:
            media = statistics.mean(tamanhos)
            desvio = statistics.stdev(tamanhos) if len(tamanhos) > 1 else 0
            print(f"  {origem}: média={media:.1f} bytes, desvio={desvio:.1f}")

    print("\n[ANÁLISE] Assim como no caso real, a origem com padrão de conexão mais irregular")
    print("(volume e tamanho) é a que merece a próxima camada de investigação — a rede, não o conteúdo, entrega o atacante.")

if __name__ == "__main__":
    pcap = "/app/logs/trafego_simulado.pcap"
    if not os.path.exists(pcap):
        print("[ERRO] Arquivo pcap não encontrado. Execute primeiro o gerador.")
    else:
        analisar(pcap)
Rodar o Módulo 4
bash
# Gerar o tráfego
docker-compose run --rm pcap python gerar_trafego.py

# Analisar o tráfego
docker-compose run --rm pcap python analisar_trafego.py
Para rodar ambos em sequência (caso o container tenha CMD definido para gerar, já executa ao subir):

bash
docker-compose up --build pcap   # gera o pcap
docker-compose run --rm pcap python analisar_trafego.py
```

## 🌐 Módulo 5 — OSINT

> **Lógica de Google Dorking em base local simulada.** O módulo não consulta o Google real: reproduz a lógica de operadores contra um índice fictício.

Google Dorking é, na essência, usar operadores de busca (site:, filetype:, intitle:, intext:) para filtrar exatamente o que se procura dentro de um índice gigante. Para manter o lab seguro e 100% autocontido, esse módulo não consulta o Google real — ele simula a mesma lógica contra um índice local fictício, o suficiente para entender o conceito sem qualquer risco de uso indevido contra terceiros.

### 5.1 osint/Dockerfile

```dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY indice_falso.json dork_sim.py .
CMD ["python", "dork_sim.py"]
5.2 osint/indice_falso.json
json
[
  {"url": "empresa-ficticia.com/login", "titulo": "Painel de Login - Ambiente de Homologação", "tipo": "html"},
  {"url": "empresa-ficticia.com/backup/db_2025.sql", "titulo": "Backup de Banco", "tipo": "sql"},
  {"url": "empresa-ficticia.com/docs/manual.pdf", "titulo": "Manual do Funcionário", "tipo": "pdf"},
  {"url": "blog-ficticio.com/post-seguranca", "titulo": "Boas práticas de segurança", "tipo": "html"},
  {"url": "empresa-ficticia.com/config/settings.ini", "titulo": "Configuração de aplicação", "tipo": "ini"},
  {"url": "intranet-ficticia.com/rh/folha.xlsx", "titulo": "Folha de pagamento - reservado", "tipo": "xlsx"}
]
5.3 osint/dork_sim.py
python
import json
import sys

def carregar_indice():
    with open("/app/indice_falso.json") as f:
        return json.load(f)

def dork(indice, site=None, filetype=None, intitle=None, intext=None):
    resultado = indice
    if site:
        resultado = [r for r in resultado if site in r["url"]]
    if filetype:
        resultado = [r for r in resultado if r["tipo"] == filetype]
    if intitle:
        resultado = [r for r in resultado if intitle.lower() in r["titulo"].lower()]
    if intext:
        # Simula busca por texto no título ou URL
        resultado = [r for r in resultado if intext.lower() in r["titulo"].lower() or intext.lower() in r["url"].lower()]
    return resultado

def demonstrar():
    indice = carregar_indice()

    print('[OSINT SIM] Consulta equivalente a: site:empresa-ficticia.com filetype:sql')
    achados = dork(indice, site="empresa-ficticia.com", filetype="sql")
    for a in achados:
        print(f"  -> {a['url']} ({a['titulo']})")

    print('\n[OSINT SIM] Consulta: intitle:"Backup"')
    achados = dork(indice, intitle="Backup")
    for a in achados:
        print(f"  -> {a['url']} ({a['titulo']})")

    print('\n[OSINT SIM] Consulta: intext:pagamento site:intranet-ficticia.com')
    achados = dork(indice, site="intranet-ficticia.com", intext="pagamento")
    for a in achados:
        print(f"  -> {a['url']} ({a['titulo']})")

    print("\n[OSINT SIM] Lição: o mesmo operador que ajuda você a auditar sua própria exposição")
    print("é o que um atacante usa pra mapear alvo. A defesa começa em rodar essa busca contra si mesmo antes.")

if __name__ == "__main__":
    demonstrar()
Rodar o Módulo 5
bash
docker-compose run --rm osint
```

## 🐳 Docker Compose Consolidado

**Abaixo o arquivo docker-compose.yml completo com todos os serviços, pronto para uso.**

```yaml
version: '3.8'
services:
  # Módulo 1
  alvo:
    build: ./alvo
    networks:
      takelab_net:
        ipv4_address: 172.20.0.10

  sensor1:
    build: ./sensor
    environment:
      SENSOR_ID: sensor1
      POS_X: 0
      POS_Y: 0
    volumes:
      - './logs:/app/logs'
    networks:
      takelab_net:
        ipv4_address: 172.20.0.11

  sensor2:
    build: ./sensor
    environment:
      SENSOR_ID: sensor2
      POS_X: 10
      POS_Y: 0
    volumes:
      - './logs:/app/logs'
    networks:
      takelab_net:
        ipv4_address: 172.20.0.12

  sensor3:
    build: ./sensor
    environment:
      SENSOR_ID: sensor3
      POS_X: 5
      POS_Y: 10
    volumes:
      - './logs:/app/logs'
    networks:
      takelab_net:
        ipv4_address: 172.20.0.13

  analista:
    build: ./analista
    volumes:
      - './logs:/app/logs'
      - './output:/app/output'
    depends_on:
      - sensor1
      - sensor2
      - sensor3
    networks:
      takelab_net:
        ipv4_address: 172.20.0.20

  # Módulo 2
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      discovery.type: single-node
      xpack.security.enabled: "false"
    ports:
      - "9200:9200"
    networks:
      - takelab_net

  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    volumes:
      - './logstash/pipeline:/usr/share/logstash/pipeline'
      - './logs:/logs'
    depends_on:
      - elasticsearch
    networks:
      - takelab_net

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
    networks:
      - takelab_net

  gerador_ataque:
    build: ./gerador
    volumes:
      - './logs:/logs'
    networks:
      - takelab_net

  # Módulo 3
  recon:
    build: ./recon
    volumes:
      - './logs:/app/logs'
    network_mode: "service:analista"   # compartilha rede com o analista
    depends_on:
      - sensor1
      - sensor2
      - sensor3

  # Módulo 4
  pcap:
    build: ./pcap
    volumes:
      - './logs:/app/logs'
    networks:
      - takelab_net

  # Módulo 5
  osint:
    build: ./osint
    networks:
      - takelab_net

networks:
  takelab_net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

## 🚀 Como Executar o Laboratório Completo

**Crie a estrutura de diretórios:**

```bash
mkdir -p takelab/{logs,output,alvo,sensor,analista,gerador,logstash/pipeline,recon,pcap,osint}
cd takelab
Coloque todos os arquivos conforme descrito acima (Dockerfiles, scripts, etc.).

Inicie todos os serviços (exceto os que exigem interação manual):

bash
docker-compose up -d
Para o Módulo 1 (triangulação), execute:

bash
docker-compose up analista   # ou use -d para background
# Após alguns segundos, veja o mapa em output/mapa.html
Para o Módulo 2, após subir o ELK, acesse http://localhost:5601 e configure o Kibana.

Para o Módulo 3 (recon), rode:

bash
docker-compose run --rm recon
# O resultado estará em logs/recon.json
Para o Módulo 4 (pcap):

bash
docker-compose run --rm pcap python gerar_trafego.py
docker-compose run --rm pcap python analisar_trafego.py
Para o Módulo 5 (OSINT):

bash
docker-compose run --rm osint
```

## 🧠 Lições do Laboratório

| Técnica em 1995 | Técnica em 2026 | Lição |
|---|---|---|
| Antena Yagi + Van | 3 containers + RSSI simulado | Metadado de sinal localiza |
| Análise de log manual | ELK + Kibana | Padrão de tempo entrega o atacante |
| Escuta de linha telefônica | Nmap em escopo próprio | Reconhecimento vem antes de qualquer conclusão |
| Percepção de latência do celular clonado | Scapy + PyShark | O como do tráfego entrega tanto quanto o o quê |
| Levantamento manual de exposição | Dorking simulado em índice local | Auditar sua própria exposição é defesa, não ataque |
## 🧹 Como desligar tudo

```bash
docker-compose down -v
```

## ⚖️ Aviso legal

> [!WARNING]
>Este laboratório é um ambiente isolado, local e educacional, construído inteiramente com dados fictícios e containers próprios. Nenhum módulo aqui descrito realiza varredura, captura de tráfego ou consulta contra redes, sistemas ou dados de terceiros — todo o escopo de rede (172.20.0.0/16) e todo o índice de dados (OSINT) são simulados e controlados pelo próprio operador do lab.

> [!CAUTION]
>A reprodução deste material contra qualquer sistema sem autorização expressa configura conduta vedada pela Lei nº 12.737/2012 (Lei Carolina Dieckmann), que tipifica o acesso não autorizado a sistemas informáticos. O tratamento de quaisquer dados pessoais reais está fora do escopo e finalidade deste laboratório, em conformidade com a Lei nº 13.709/2018 (LGPD). Use este material exclusivamente para fins de estudo em ambiente próprio e controlado.
