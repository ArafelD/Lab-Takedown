# ⚔️ Lab Takedown — Recriando a Caçada de Shimomura em 2026

> **1995:** antenas, telefones, logs e muita paciência.  
> **2026:** Docker, ELK, Nmap, Scapy, PyShark e OSINT.
>
> **A tecnologia mudou. O raciocínio investigativo, não.**

![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?logo=docker&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![ELK](https://img.shields.io/badge/ELK-8.11.0-005571)
![Nmap](https://img.shields.io/badge/Nmap-Recon-4EAA25)
![OSINT](https://img.shields.io/badge/OSINT-Simulated-111827)
![Status](https://img.shields.io/badge/Environment-Isolated-success)

---

## 🕵️ Sobre o laboratório

Este laboratório reproduz, de forma **isolada, controlada e 100% fictícia**, o raciocínio investigativo associado à caçada de Tsutomu Shimomura a Kevin Mitnick em 1995.

A proposta não é reproduzir literalmente a infraestrutura da época. É transportar **as camadas de raciocínio** para um ambiente moderno:

> **observar → coletar → correlacionar → formular hipótese → validar → localizar**

O laboratório roda em **um único computador**, utilizando containers Docker e ferramentas open source.

---

## 🧠 O que está sendo reproduzido?

| 1995 | 2026 | Ideia central |
|---|---|---|
| Antena direcional + vans | Containers + RSSI simulado | Metadado de sinal pode ajudar a localizar |
| Comparação manual de timestamps | ELK + Kibana | Correlação transforma eventos isolados em padrão |
| Reconhecimento da infraestrutura | Nmap | Antes de investigar, é preciso conhecer o ambiente |
| Análise de comportamento/latência | Scapy + PyShark | O padrão do tráfego pode revelar contexto |
| Pesquisa de exposição | OSINT simulado | A mesma superfície que ajuda o defensor pode ajudar o atacante |

---

## 🧩 Arquitetura

```text
                         ┌─────────────────────┐
                         │       ALVO          │
                         │  sinal simulado     │
                         └──────────┬──────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
               ┌────────┐      ┌────────┐      ┌────────┐
               │Sensor 1│      │Sensor 2│      │Sensor 3│
               └────┬───┘      └────┬───┘      └────┬───┘
                    └───────────────┼───────────────┘
                                    ▼
                           ┌────────────────┐
                           │    ANALISTA    │
                           │  Trilateração  │
                           └────────────────┘

        ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
        │   ELK Stack  │   │     Nmap     │   │ Scapy/PyShark│
        │ Correlação   │   │ Reconhecimento│  │ Tráfego      │
        └──────────────┘   └──────────────┘   └──────────────┘
                                    │
                              ┌──────────────┐
                              │  OSINT SIM   │
                              │ Índice local │
                              └──────────────┘
```

---

## 🧪 Cinco módulos

### 01 · Triangulação Física
Três sensores em containers simulam medições de RSSI e cruzam os dados por trilateração.

### 02 · Correlação de Logs
ELK recebe eventos sintéticos e permite procurar padrões de horário, usuário e origem.

### 03 · Reconhecimento Ativo
Nmap mapeia exclusivamente a rede criada para o laboratório.

### 04 · Análise de Tráfego
Scapy gera tráfego sintético; PyShark analisa metadados da captura.

### 05 · OSINT Simulado
Operadores inspirados em Google Dorking são aplicados contra uma base local fictícia.

---

## 🚀 Quick Start

```bash
git clone <URL-DO-REPOSITORIO>
cd Lab-Takedown

docker-compose up -d
```

> [!WARNING]
> O laboratório foi projetado para execução em ambiente próprio e controlado. Não altere o escopo para redes ou sistemas de terceiros.

Para o passo a passo completo, incluindo estrutura de diretórios, `Dockerfile`, `docker-compose.yml`, scripts Python e execução individual dos módulos:

👉 **[Abrir a documentação técnica completa](lab-takedown.md)**

---

## 🛠️ Stack

- **Docker / Docker Compose** — isolamento e orquestração
- **Python 3.10+** — simulação e análise
- **Elasticsearch / Logstash / Kibana** — ingestão e correlação
- **Nmap** — reconhecimento
- **Scapy** — geração de tráfego sintético
- **PyShark / TShark** — análise de pacotes
- **Folium / SciPy / NumPy** — trilateração e visualização
- **OSINT simulado** — operadores de busca sobre índice local

---

## 🔐 Escopo e segurança

Este laboratório utiliza:

- dados fictícios;
- containers próprios;
- rede Docker isolada;
- índice OSINT local;
- tráfego sintético;
- escopo de rede definido pelo próprio laboratório.

Nenhum módulo depende de coleta de dados reais ou de acesso autorizado a sistemas de terceiros.

---

## 🧠 A principal lição

A parte mais interessante deste laboratório não é a ferramenta.

É o método.

Shimomura não precisou de uma ferramenta “mágica” para transformar sinais aparentemente desconexos em uma hipótese investigativa. Ele precisou **correlacionar evidências**.

Trinta anos depois, continuamos fazendo essencialmente a mesma coisa — apenas com uma quantidade muito maior de telemetria e ferramentas.

> **A ferramenta muda. O método permanece.**

---

## ⚔️ Filosofia

**Porque conhecimento ofensivo só tem valor quando serve pra fortalecer a defesa. ⚔️**

---

## ⚖️ Aviso legal

Este projeto possui finalidade exclusivamente educacional e deve ser executado em ambiente próprio, isolado e controlado.

Consulte [`lab-takedown.md`](lab-takedown.md) para o aviso legal completo e o escopo técnico do laboratório.
