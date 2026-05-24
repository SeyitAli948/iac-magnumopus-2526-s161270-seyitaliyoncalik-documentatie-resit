# Magnum Opus – Infrastructure as Code

## DokuWiki Deployment met Ansible

---

# 1. Korte projectbeschrijving

Dit project automatiseert de installatie en configuratie van een self-hosted DokuWiki-omgeving met behulp van Ansible.

De deployment configureert automatisch:

* Apache webserver
* PHP
* DokuWiki
* gebruikersauthenticatie
* ACL/rechtenbeheer
* firewallregels
* servicebeheer
* basisconfiguratie van de wiki

Het doel van dit project is om een reproduceerbare en geautomatiseerde deployment van een wiki-platform op Rocky Linux uit te voeren volgens Infrastructure as Code-principes.

---

# 2. Korte scope (operationeel)

## In scope

* Installatie van Apache en PHP
* Installatie van DokuWiki
* Automatische configuratie via Ansible
* Configuratie van ACL en authenticatie
* Configuratie van gebruikers
* Firewallconfiguratie
* Servicebeheer via systemd
* Basis security-configuratie

## Buiten scope

* Externe database
* Reverse proxy setup
* Domeinnaam/DNS-configuratie
* HTTPS-certificaten via publieke CA
* High availability
* Externe authenticatie (LDAP/SSO)

---

# 3. Geteste omgeving en randvoorwaarden

## Target omgeving

* Rocky Linux 10

## Control node

* Rocky Linux 10
* SSH-toegang vereist
* automation user: ansible

## Toolingversies

* ansible-core 2.16
* Apache httpd
* PHP
* DokuWiki stable release

## Externe randvoorwaarden

* Internettoegang voor package-installatie
* Open poort 80/tcp
* Werkende SSH-connectiviteit

---

# 4. Repositorystructuur

```text
magnum-opus/
├── inventory/
│   └── hosts
├── playbooks/
│   └── site.yml
├── roles/
│   ├── base/
│   ├── webserver/
│   └── dokuwiki/
└── README.md
```

---

# 5. Configuratie-instructies

## Inventory aanpassen

Pas het inventory-bestand aan met het correcte IP-adres van de target machine.

Voorbeeld:

```ini
[dokuwiki]
172.16.120.11 ansible_user=ansible
```

## Belangrijke variabelen

Variabelen kunnen aangepast worden in de role defaults of vars-bestanden:

* dokuwiki_dir
* dokuwiki_owner
* dokuwiki_group
* wiki_title
* admin_user

## Secrets

Gevoelige gegevens mogen niet in plaintext in de repository opgeslagen worden.

Voor een productieomgeving wordt aanbevolen om Ansible Vault te gebruiken.

---

# 6. Run-instructies

## Playbook uitvoeren

```bash
ansible-playbook -i inventory/hosts playbooks/site.yml
```

## Connectiviteit testen

```bash
ansible all -i inventory/hosts -m ping
```

## Services controleren

```bash
sudo systemctl status httpd
```

---

# 7. Verificatie-instructies

## Browsercontrole

Open in een browser:

```text
http://SERVER-IP
```

Verwacht resultaat:

* DokuWiki loginpagina verschijnt
* Admin login werkt
* Wiki-interface is bereikbaar

## Servicecontrole

```bash
sudo systemctl status httpd
```

Verwacht resultaat:

```text
active (running)
```

## Firewallcontrole

```bash
sudo firewall-cmd --list-services
```

Verwacht resultaat:

```text
http
```

---

# 8. Idempotentie-check

## Eerste run

```bash
ansible-playbook -i inventory/hosts playbooks/site.yml
```

Verwacht resultaat:

* packages worden geïnstalleerd
* configuratiebestanden worden aangemaakt
* services worden gestart

## Tweede run

Voer exact hetzelfde commando opnieuw uit.

Verwacht resultaat:

* geen onverwachte wijzigingen
* meeste taken tonen `ok`
* systeem blijft stabiel

## Gekende uitzondering

Bepaalde dynamische bestanden of install locks kunnen uitzonderlijk als `changed` verschijnen.

---

# 9. Bekende beperkingen / troubleshooting

## Mogelijke problemen

### Apache start niet

Controleer of poort 80 al gebruikt wordt:

```bash
sudo ss -tulpn | grep :80
```

### Firewall blokkeert toegang

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

### Login werkt niet

Controleer:

* users.auth.php
* acl.auth.php
* local.php

---

# 10. Verdieping

Aanwezige verdieping:

* ACL/rechtenbeheer
* automatische gebruikersconfiguratie
* non-root automation user
* become/sudo-configuratie
* idempotente deployment

---

# 11. Verwijzing naar projectdocumentatie-PDF

De uitgebreide technische analyse, motivatie, ontwerpkeuzes, screenshots en bewijsvoering zijn opgenomen in de projectdocumentatie-PDF.
