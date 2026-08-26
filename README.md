# Lab Takedown — Recriando a Caçada de Shimomura em 2026

Laboratório prático que recria, em containers Docker isolados e com dados 100% fictícios, o raciocínio usado por Tsutomu Shimomura para localizar Kevin Mitnick em 1995 — desdobrado em cinco camadas de investigação.

**Objetivo:** demonstrar, na prática, que atacante deixa rastro digital, de rede, comportamental e físico — e que esse rastro pode ser reconstruído com ferramentas open source e um pouco de paciência.

> ⚠️ Ambiente 100% simulado. Sem alvos reais, sem coleta de dados de terceiros, sem uso contra redes de terceiros. Ver aviso legal no final deste documento.

---
## Arquitetura do Lab

```
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
## Requisitos

- Docker + Docker Compose
- ~4 GB de RAM livres
- Portas 5601 e 9200 livres
- Python 3.10+ (só se quiser rodar os módulos 3, 4 e 5 fora de container também)

## Estrutura de pastas

```
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

---
