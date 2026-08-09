# Magnum Opus – Ansible DokuWiki Deployment

## 1. Projectbeschrijving

Dit project automatiseert de installatie en configuratie van **DokuWiki** op een Rocky Linux-server met behulp van **Ansible**.

Ansible configureert de server, installeert de benodigde software, configureert Apache en PHP, installeert DokuWiki en configureert de firewall.

Na een succesvolle uitvoering is DokuWiki bereikbaar via Apache en kan de werking gecontroleerd worden via een webbrowser of met `curl`.

---

## 2. Scope

### Inbegrepen

Deze repository automatiseert onder andere:

* basisconfiguratie van de Rocky Linux-server;
* installatie en configuratie van Apache;
* installatie van PHP;
* installatie van DokuWiki;
* configuratie van Apache voor DokuWiki;
* configuratie van DokuWiki;
* configuratie van bestandseigenaarschap en permissies;
* configuratie van de firewall;
* configuratie van firewall logging;
* starten en inschakelen van de benodigde services;
* reproduceerbare uitvoering via Ansible.

### Buiten scope

De volgende zaken vallen buiten de scope van dit project:

* publieke DNS-configuratie;
* hosting op een publieke cloudomgeving;
* externe domeinregistratie;
* beheer van de Rocky Linux-installatie zelf;
* productiegebruik van de voorbeeldcredentials.

De deployment is getest op een Rocky Linux-server die lokaal bereikbaar is.

---

## 3. Geteste omgeving en randvoorwaarden

### Targetomgeving

Het project is getest op:

* Rocky Linux
* x86_64
* PHP 8.3.32
* Apache HTTP Server 2.4.63

### Control node

De Ansible control node gebruikt:

* Python 3.12.13
* ansible-core 2.21.2
* ansible-lint 26.6.0
* ansible.posix 2.2.2

De exacte Ansible-versies kunnen gecontroleerd worden met:

```bash
ansible --version
ansible-lint --version
ansible-galaxy collection list
```

### Randvoorwaarden

Voor de deployment zijn nodig:

* een Rocky Linux-server;
* een gebruiker met `sudo`/rootrechten;
* internettoegang vanaf de targetserver;
* toegang tot de DokuWiki-downloadlocatie;
* Python/Ansible op de control node;
* de Ansible collection `ansible.posix`.

HTTP en SSH moeten netwerkmatig bereikbaar zijn wanneer de service vanaf een andere machine gebruikt wordt.

---

## 4. Repositorystructuur

De belangrijkste projectbestanden zijn:

```text
.
├── ansible.cfg
├── .gitignore
├── requirements.yml
├── inventory/
│   ├── hosts
│   └── group_vars/
│       └── wiki.yml
├── playbooks/
│   └── site.yml
├── roles/
│   ├── base/
│   ├── webserver/
│   └── dokuwiki/
└── README.md
```

### Rollen

**base**

Verzorgt de basisconfiguratie van de Rocky Linux-server en firewall.

**webserver**

Installeert en configureert Apache en PHP en configureert Apache voor DokuWiki.

**dokuwiki**

Installeert en configureert DokuWiki en beheert de benodigde bestanden en configuratie.

---

## 5. Configuratie

De hostspecifieke configuratie staat in:

```text
inventory/group_vars/wiki.yml
```

De belangrijkste variabelen zijn:

```yaml
dokuwiki_dir
dokuwiki_group
dokuwiki_owner
dokuwiki_title
dokuwiki_url
```

Deze variabelen bepalen onder andere:

* de installatieplaats van DokuWiki;
* de eigenaar van de bestanden;
* de groep van de bestanden;
* de titel van de wiki;
* de downloadlocatie van DokuWiki.

Controleer de effectieve Ansible-variabelen met:

```bash
ansible-inventory -i inventory/hosts --host localhost
```

### Secrets

Gevoelige gegevens mogen niet rechtstreeks als plaintext in de Git-repository worden opgeslagen.

Voor een productieomgeving wordt hiervoor Ansible Vault of een andere geschikte secrets-managementoplossing aanbevolen.

---

## 6. Installatie vanaf een verse clone

### 6.1 Repository clonen

Clone de repository op de Ansible control node:

```bash
git clone <PRIVATE_GITHUB_REPOSITORY_URL>
cd magnum-opus
```

Ga naar de gebruiker ansible:

```bash
ssh ansible@localhost
```

Gebruik voor de beoordelingsversie de tag:

```bash
git checkout submission-resit-v1
```

Controleer de huidige versie met:

```bash
git status
git describe --tags
```

### 6.2 Ansible installeren

Maak indien gewenst een Python virtual environment:

```bash
python3 -m venv ansible-venv
source ansible-venv/bin/activate
```

Installeer Ansible:

```bash
pip install ansible
```

Controleer:

```bash
ansible --version
```

### 6.3 Vereiste Ansible collection installeren

Installeer de collections uit `requirements.yml`:

```bash
ansible-galaxy collection install -r requirements.yml
```

Controleer:

```bash
ansible-galaxy collection list
```

De collection `ansible.posix` moet aanwezig zijn.

### 6.4 Inventory controleren

Controleer de inventory:

```bash
ansible-inventory -i inventory/hosts --graph
```

Controleer vervolgens de variabelen van de host:

```bash
ansible-inventory -i inventory/hosts --host localhost
```

### 6.5 Verbinding testen

Test de Ansible-verbinding:

```bash
ansible -i inventory/hosts wiki -m ansible.builtin.ping
```

Verwacht resultaat:

```text
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

## 7. Deployment uitvoeren

Voer het volledige playbook uit met:

```bash
ansible-playbook -i inventory/hosts playbooks/site.yml
```

Het playbook gebruikt de volgende rollen:

```text
base
webserver
dokuwiki
```

Na een succesvolle uitvoering moet de play eindigen zonder `failed` tasks.

---

## 8. Verificatie

### 8.1 Apache controleren

Controleer de Apache-configuratie:

```bash
sudo apachectl configtest
```

Verwacht resultaat:

```text
Syntax OK
```

Controleer of Apache actief is:

```bash
sudo systemctl status httpd
```

### 8.2 DokuWiki controleren

Controleer lokaal de HTTP-response:

```bash
curl -I http://localhost/
```

Een werkende DokuWiki-installatie geeft een HTTP-response terug. Een redirect naar bijvoorbeeld:

```text
Location: /doku.php?id=start
```

is een indicatie dat DokuWiki correct wordt aangeboden door Apache.

### 8.3 Via browser

Open vanaf een machine die toegang heeft tot de server:

```text
http://<server-ip>/
```

De DokuWiki-startpagina moet zichtbaar zijn.

### 8.4 DokuWiki-bestanden controleren

Controleer de installatie:

```bash
ls -la /var/www/dokuwiki/dokuwiki-2025-05-14b/
```

De directory moet onder andere `doku.php`, `conf/`, `data/`, `inc/` en `vendor/` bevatten.

---

## 9. Firewall controleren

Controleer de actieve firewallconfiguratie:

```bash
sudo firewall-cmd --list-all
```

Controleer welke services geopend zijn:

```bash
sudo firewall-cmd --list-services
```

Controleer de denied logging:

```bash
sudo firewall-cmd --get-log-denied
```

Voor deze deployment moet HTTP bereikbaar zijn en moeten alleen noodzakelijke netwerkdiensten geopend zijn.

De denied logging moet actief zijn.

---

## 10. Idempotentie controleren

Idempotentie wordt gecontroleerd door het playbook tweemaal uit te voeren.

### Run 1

```bash
ansible-playbook -i inventory/hosts playbooks/site.yml
```

Tijdens de eerste uitvoering worden de benodigde wijzigingen uitgevoerd. Daarom worden `changed` tasks verwacht.

### Run 2

Voer exact hetzelfde commando opnieuw uit:

```bash
ansible-playbook -i inventory/hosts playbooks/site.yml
```

Bij de tweede uitvoering worden geen onverwachte wijzigingen verwacht.

Het doel is dat de meeste configuratietaken eindigen met:

```text
changed=0
```

Dit toont aan dat de deployment idempotent is en dat Ansible de bestaande gewenste toestand herkent.

---

## 11. Syntax- en kwaliteitscontrole

Controleer eerst de syntax:

```bash
ansible-playbook -i inventory/hosts --syntax-check playbooks/site.yml
```

Een succesvolle controle eindigt zonder foutmelding.

Ansible-lint kan worden uitgevoerd met:

```bash
ansible-lint roles/base roles/webserver roles/dokuwiki
```

Eventuele lintmeldingen moeten worden beoordeeld en waar relevant opgelost.

---

## 12. Troubleshooting

### Inventory wordt niet gevonden

Gebruik altijd expliciet de inventory:

```bash
ansible-inventory -i inventory/hosts --graph
```

en bij het uitvoeren van het playbook:

```bash
ansible-playbook -i inventory/hosts playbooks/site.yml
```

### `ansible.posix` ontbreekt

Installeer de collection opnieuw:

```bash
ansible-galaxy collection install -r requirements.yml
```

### Apache werkt niet

Controleer:

```bash
sudo apachectl configtest
sudo systemctl status httpd
```

### Firewall blokkeert HTTP

Controleer:

```bash
sudo firewall-cmd --list-services
```

HTTP moet aanwezig zijn.

### DokuWiki is niet bereikbaar

Controleer eerst Apache:

```bash
sudo systemctl status httpd
```

Daarna:

```bash
curl -I http://localhost/
```

Controleer vervolgens de DokuWiki-installatiedirectory en de Apache-configuratie.

---

## 13. Verdieping

De basisdeployment wordt uitgevoerd en geverifieerd voordat eventuele verdieping wordt toegevoegd.

Eventuele verdieping wordt afzonderlijk gedocumenteerd en alleen geclaimd wanneer deze volledig geïmplementeerd, reproduceerbaar en aantoonbaar getest is.

---

## 14. Projectdocumentatie

Naast deze README wordt een aparte projectdocumentatie-PDF ingediend.

De PDF bevat de technische onderbouwing, gemaakte keuzes, bewijsstukken, securitymaatregelen, idempotentie-bewijs, kwaliteitscontroles, bronnen en eventuele verdieping.

De beoordelingsversie van het project wordt vastgelegd met de Git-tag:

```text
submission-resit-v1
```

De exacte commit hash van deze tag wordt samen met de repository-URL ingediend via Digitap.

**Projectdocumentatie:**

```text
iac-magnumopus-2526-seyitaliyoncalik-documentatie-resit.pdf
```
