# Laboratório — Exploração de Redes Vulneráveis (UEMG)

Ambiente Docker desenvolvido como trabalho avaliativo da disciplina **Redes de Computadores II** — Curso de Sistemas de Informação, 6° período (UEMG, 2025).

O objetivo é simular, em ambiente isolado, ataques e defesas em redes sem fio (captive portal / rogue AP simulado, MITM sobre HTTP), capturar tráfego (pcap), anonimizar os dados e apresentar contramedidas e recomendações.

---

## Conteúdo do repositório

- `docker/` — arquivos do Docker (docker-compose, configurações Nginx, páginas HTML, certificados).
- `pcaps/` — capturas brutas (raw) e pcap anonimizado (apenas `anon_capture.pcap` deve ser publicado).
- `configs/` — exemplos de Bettercap, tcpdump e scripts.
- `scripts/` — scripts utilitários (por exemplo: `start_capture.sh`, `anonymize_pcap.sh`).
- `docs/` — metodologia, notas explicativas, segurança e checklist.
- `report/` — relatório final e apresentação.

---

## Serviços e portas (visão rápida)

| Serviço     | Porta host | Descrição                                          |
|-------------|------------|----------------------------------------------------|
| `vuln_server`| 8010       | Servidor HTTP vulnerável (python http.server)      |
| `lab_portal`| 8011       | Portal (Nginx) — splash / formulário de login      |
| `lab_https` | 8443       | Portal HTTPS (Nginx + certificado autoassinado)    |
| `lab_capture`| —         | Container `netshoot`/tshark para captura (pcap)    |

### Acesso aos serviços (exemplo)

- HTTP server: `http://localhost:8010` ou `http://<HOST_IP>:8010`
- Portal: `http://localhost:8011` ou `http://<HOST_IP>:8011`
- HTTPS: `https://localhost:8443` (aceitar certificado autoassinado)

---

## Pré-requisitos (host)

- Docker Desktop (Windows / macOS / Linux) — https://www.docker.com
- VirtualBox (para VMs Ubuntu/Kali) — https://www.virtualbox.org
- Wireshark (para analisar pcaps) — https://www.wireshark.org
- (Opcional) Git, VS Code

---

## Quickstart — levantar o ambiente (passo a passo)

1. Clone o repositório:

```bash
git clone https://github.com/enzojop/laboratorio-redes-wifi-uemg.git
cd /docker/
```

2. Verifique que os arquivos e pastas existem:

```text
docker-compose.yml
portal/
https/
server/
pcaps/
```

3. Suba os containers:

```bash
docker compose up -d
# ou
docker-compose up -d
```

4. Verifique containers:

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"
```

5. Teste local (host):

```bash
curl http://localhost:8010
curl http://localhost:8011
# PowerShell
iwr https://localhost:8443 -SkipCertificateCheck
# bash
curl -k https://localhost:8443
```

6. Teste a partir da VM (cliente):

```bash
# exemplo: se Host-Only IP for 192.168.56.1
curl http://192.168.56.1:8010
curl http://192.168.56.1:8011
curl -k https://192.168.56.1:8443
```

---

## Gerar captura (Kali / Attacker)

No Kali (ou container `lab_capture`), inicie a captura:

```bash
# usando tcpdump
sudo tcpdump -i any net 192.168.56.0/24 -s 0 -w /root/pcaps/raw_capture.pcap

# ou (dentro do container lab_capture)
# já configurado no compose para salvar em ../pcaps/raw_capture.pcap
```

Realize a ação no cliente (submeter formulário no portal) para gerar tráfego que será salvo em `pcaps/raw_capture.pcap`.

---

## Anonimização de pcap (script)

Exemplo de script (`scripts/anonymize_pcap.sh`):

```bash
#!/bin/bash
tcprewrite --infile="$1" --outfile=/tmp/step1.pcap \
  --srcipmap=192.168.56.0/24:10.10.10.0/24 --dstipmap=192.168.56.0/24:10.10.10.0/24 --fixcsum

tcprewrite --infile=/tmp/step1.pcap --outfile=/tmp/step2.pcap \
  --enet-smac=00:aa:bb:cc:dd:01 --enet-dmac=00:aa:bb:cc:dd:02 --fixcsum

editcap -r /tmp/step2.pcap pcaps/anon_capture.pcap "not udp port 53"
```

Uso:

```bash
chmod +x scripts/anonymize_pcap.sh
./scripts/anonymize_pcap.sh pcaps/raw_capture.pcap
```

Verifique `pcaps/anon_capture.pcap` no Wireshark antes de submeter.

---

## Contramedidas demonstradas (sugeridas)

- Segmentação / isolamento — bloquear tráfego lateral entre hosts (iptables / regras do host) e/ou usar `client_isolate` no AP simulado.
- Usar HTTPS (TLS) — demonstrar captura antes/depois: credenciais visíveis em HTTP; em HTTPS aparecem como TLS encrypted.
- (Opcional) Detecção de ARP spoofing e verificação de gateways duplicados.

Documente resultados com screenshots (Wireshark) e compare pcaps.

---

## Boas práticas de segurança e ética

- Execute os testes apenas em rede isolada (Host-Only / VMs), sem conectar à rede real da faculdade.
- Não colete dados reais de terceiros. Use contas e senhas de teste.
- Anonimize pcaps antes de publicar.
- Guarde o `raw_capture.pcap` apenas em backup privado (não comite).

---

## Recursos úteis

- Docker: https://www.docker.com
- VirtualBox: https://www.virtualbox.org
- Wireshark: https://www.wireshark.org
- Bettercap: https://www.bettercap.org/
- tcpreplay / tcprewrite / editcap (Wireshark tools)

---

## Contribuidores / Integrantes do grupo

- Enzo José Oliveira Pereira 
- Nathan Massamb Belinato
- Livya Silva de Carvalho
- Kayan Paiva
