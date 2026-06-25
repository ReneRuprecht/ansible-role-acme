# ACME.sh & Vault Integration

Diese Ansible Rolle installiert und konfiguriert das Tool acme.sh, um SSL/TLS-Zertifikate (z. B. von Let's Encrypt) via DNS-Challenge zu beantragen. 
Optional können die generierten Zertifikate automatisch in eine HashiCorp Vault Instanz geladen werden.

## Unterstützte Betriebssysteme

Die Rolle ist primär für Debian getestet. 

* **Debian:** 12 (Bookworm), 13 (Trixie)

## Voraussetzungen (Prerequisites)

*   Ein installierter Git Client auf dem Zielsystem (zum Klonen von `acme.sh`)
*   Für die Vault-Integration: Eine laufende HashiCorp Vault Instanz und installierte Vault-CLI/Bibliotheken

## Rollen-Variablen

Die folgenden Variablen steuern das Verhalten der Rolle und sind in `defaults/main.yml` definiert:

### Git & Installation

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `acme_git_version` | `"master"` | Die zu verwendende Git-Branch, Tag oder Commit-SHA von acme.sh. |
| `acme_git_update` | `false` | Wenn `true`, wird das acme.sh Git-Repository bei jedem Lauf aktualisiert. |
| `acme_git_clone_dest` | `"/home/{{ acme_user }}/acme.sh"` | Zielpfad auf dem Server, in den acme.sh geklont wird. |

### System-Benutzer & System-Gruppe

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `acme_user` | `"acme"` | Systembenutzer, unter dem acme.sh ausgeführt wird. |
| `acme_group` | `"acme"` | Systemgruppe für den acme-Benutzer. |

### Zertifikats- und ACME-Konfiguration

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `acme_upgrade` | `false` | Führt ein internes Upgrade von acme.sh durch (über den eigenen Update-Mechanismus). |
| `acme_account_email` | *Nicht definiert* | Die E-Mail-Adresse für die Registrierung des ACME-Accounts. |
| `acme_certs_dest_path` | `"/etc/acme/certs"` | Lokales Verzeichnis auf dem Server, in dem die Zertifikate abgelegt werden. |
| `acme_ca_server` | `"letsencrypt"` | Der zu verwendende ACME-Server (z. B. `letsencrypt`, `buypass`, `zerossl`). |
| `acme_staging` | `true` | Wenn `true`, wird die Staging/Test-Umgebung der CA genutzt, um Rate-Limits zu vermeiden. |
| `acme_cert_domains` | `[]` | Liste der Domains, für die Zertifikate ausgestellt werden sollen (siehe Beispiel unten). |

### DNS-Challenge (z. B. Cloudflare)

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `acme_dns_provider` | `"dns_cf"` | Der DNS-API-Hook von acme.sh (Standard: Cloudflare). |
| `acme_dns_provider_api_keys` | *Nicht definiert* | Dictionary mit den API-Keys/Tokens für den DNS-Anbieter (z. B. `CF_Token`, `CF_Email`). |
| `acme_dns_sleep` | `120` | Wartezeit in Sekunden, bis der DNS-TXT-Eintrag weltweit synchronisiert ist. |

### HashiCorp Vault Integration (Optional)

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `acme_vault_push_enabled` | `false` | Schaltet das automatische Hochladen der Zertifikate in Vault ein/aus. |
| `acme_vault_addr` | `""` | Die URL deiner HashiCorp Vault Instanz. |
| `acme_vault_path` | `"secret/certs"` | Der Key-Value (KV) Pfad in Vault, unter dem die Zertifikate gespeichert werden. |
| `acme_vault_approle_role_id_file`| `"/etc/vault/role-id"` | Pfad zur Datei, die die Vault AppRole `role-id` enthält. |
| `acme_vault_approle_secret_id_file`| `"/etc/vault/secret-id"`| Pfad zur Datei, die die Vault AppRole `secret-id` enthält. |
| `acme_deploy_vault_script_dir` | `"/opt/acme-scripts"` | Verzeichnis für das Vault-Deployment-Skript. |
| `acme_deploy_vault_script` | `".../vault-deploy.sh"` | Absoluter Pfad zum generierten Deployment-Skript. |

## Beispiel

Hier ist ein Beispiel, wie du die Rolle mit den Variablen nutzt, 
um ein Zertifikat via Cloudflare DNS zu beantragen und nach Vault zu pushen:

```yaml
acme_account_email: "admin@example.com"
acme_staging: false  # Produktion-Zertifikat anfordern

# Cloudflare API Daten
acme_dns_provider_api_keys:
  CF_Token: "your-super-secret-cloudflare-token"
  CF_Email: "admin@example.com"

# Liste der Domains
acme_cert_domains:
  - domains: 
      - "test.default.example.com"
      - "test2.default.example.com"
  - domains: 
      - "*.example.com"

# Vault Integration aktivieren
acme_vault_push_enabled: true
acme_vault_addr: "https://example.com"
acme_vault_path: "secret/infrastructure/certs"
```
