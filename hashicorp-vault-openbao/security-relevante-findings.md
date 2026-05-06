# Defaults-Zusammenfassung: VSO + Vault/OpenBao

## ✅ Sichere Defaults (Pentester findet hier nichts)

**VSO Helm Chart:**
- Client-Cache nur in-memory (`persistenceModel: none`) — keine Tokens auf Disk
- TLS-Verify aktiv (`skipTLSVerify: false`)
- CSI-Driver deaktiviert
- kube-rbac-proxy für Metrics aktiv
- Kein Default-`VaultAuth` — Admin muss explizit anlegen
- `allowedNamespaces` leer = nur eigener Namespace

**Vault/OpenBao K8s-Auth (frische Installation):**
- `disable_iss_validation: true` (seit Vault 1.9 für neue Mounts)
- `disable_local_ca_jwt: false`
- TLS zwischen Operator und Vault Pflicht

## ❌ Unsichere Defaults (low-hanging fruit)

**Vault/OpenBao Server:**
- **Kein Audit-Device aktiv** → keine Forensik möglich
- **Root-Token bleibt gültig** nach `vault operator init`
- **Token-TTL = 0** → System-Max greift (~32 Tage), viel länger als Pod-Lebensdauer
- **Vault ≤ 1.12**: Audience-Parameter wird komplett ignoriert (Bug)

**K8s-Auth Rollen-Konfiguration:**
- `bound_service_account_names: ["*"]` und `bound_service_account_namespaces: ["*"]` sind erlaubte Werte → Wildcard öffnet alles
- `audience` historisch optional → erst ab Vault 1.21 Pflicht für neue Rollen

## ⚠️ Migrations-Altlasten (kein Default, aber häufig)

- Rollen **vor Vault 1.21** ohne `audience` → wird in Logs gewarnt, aber funktioniert noch
- Admin aktiviert Cache-Persistence und nimmt `direct-unencrypted` aus Tutorial → Tokens im Klartext in K8s Secrets

## Pentester-Faustregel

VSO selbst ist **per Default konservativ** — Findings entstehen fast immer durch Admin-Konfiguration (Policies, Rollen, Audit). Vault-Server hingegen liefert **mehrere unsichere Defaults** (kein Audit, Root-Token, lange TTLs), die bei jedem nicht-gehärteten Setup als Finding gelten.
