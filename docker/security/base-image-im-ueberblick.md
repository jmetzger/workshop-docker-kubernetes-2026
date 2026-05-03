# Base Image Optionen im Vergleich

## Alpine
- **Basis:** musl libc + BusyBox, ~5 MB
- **Pro:** sehr klein, eigener Paketmanager (`apk`), große Community
- **Contra:** musl statt glibc → DNS-Resolver-Quirks, fehlende Thread-Local-Storage-Features, Probleme bei Python (Wheels werden oft neu kompiliert), bei Java/Go meist unkritisch
- **Use Case:** Go-Binaries, statisch gelinkte Apps, einfache Services
- **Security:** kleiner Footprint = weniger CVEs, aber eigener CVE-Tracker (separat von Debian/RH-Advisories)

## Debian-slim (`debian:bookworm-slim`)
- **Basis:** glibc, abgespeckte Debian-Variante, ~30 MB
- **Pro:** maximale Kompatibilität, alles läuft, riesiges Paket-Ökosystem (`apt`), Debian Security Team
- **Contra:** größer, mehr installierte Pakete = größere Angriffsfläche, mehr CVE-Noise im Scan
- **Use Case:** Default für „läuft halt", Python/Node-Apps mit nativen Dependencies
- **Verwandt:** `ubuntu:24.04` ähnlich; RedHat-Welt: `ubi-minimal`

## Distroless
- **Basis:** nur App + Runtime-Libs, **keine Shell, kein Paketmanager, kein apt/apk**
- **Varianten:** `static`, `base` (glibc), `cc`, `java`, `nodejs`, `python3`
- **Pro:** drastisch reduzierte Angriffsfläche, kein `sh` für Reverse-Shells nach Exploit, klein (~20 MB)
- **Contra:** Debugging schwer (kein `kubectl exec` Shell), Multi-Stage-Build Pflicht, `:debug`-Tag mit BusyBox nur für Troubleshooting
- **Use Case:** produktive Workloads nach Build-Phase
- **Achtung:** Updates kommen nur via Image-Rebuild – kein in-place Patchen

### Wolfi (Chainguard, OSS)
- **Basis:** glibc-basierte „undistro", speziell für Container designed, eigener apk-Fork
- **Pro:** **tägliche** Rebuilds, sehr schnelle CVE-Fixes, glibc (keine musl-Probleme wie Alpine), klein, mit Shell+Paketmanager – Distroless-ähnliche Sicherheit aber debugbar
- **Contra:** jüngeres Ökosystem, weniger Pakete als Debian
- **Use Case:** moderne Alternative zu Alpine ohne musl-Schmerzen
- **Hinweis:** Wolfi ist die OSS-Basis hinter Chainguard Images

### Chainguard Images
- **Basis:** Wolfi-basiert, kommerziell gehärtet
- **Pro:** SLSA Level 3 Builds, signiert (cosign), SBOM included, **oft 0 bekannte CVEs**, FIPS-Varianten, Dev-Variante mit Shell + Prod-Variante distroless-style
- **Contra:** Free-Tier nur `:latest`, gepinnte Versionen kostenpflichtig (Production-relevant!) – das ist der Haken im Training
- **Use Case:** Enterprise mit Compliance-Anforderungen, Lieferkettensicherheit
- **Lizenz wichtig erwähnen:** Free Tier ≠ alle Tags

## Entscheidungsmatrix für die Schulung

| Kriterium | Alpine | Debian-slim | Distroless | Wolfi | Chainguard |
|---|---|---|---|---|---|
| Größe | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Angriffsfläche | klein | groß | sehr klein | klein | sehr klein |
| Debugbarkeit | gut | sehr gut | schlecht | gut | mittel |
| libc | musl | glibc | glibc | glibc | glibc |
| Paketmanager | apk | apt | ❌ | apk | apk |
| CVE-Patch-Speed | mittel | gut | abhängig | sehr schnell | sehr schnell |
| Kosten | frei | frei | frei | frei | teils kommerziell |

## Häufige Stolpersteine zum Erwähnen

- **Alpine + Python:** Wheels werden ignoriert → Build dauert ewig, Image wird trotzdem groß
- **Distroless + Health Checks:** keine Shell → HTTP-basierte Probes nutzen
- **Debugging Distroless:** ephemeral debug containers (`kubectl debug`) wird zum Pflichtthema
- **`:latest` Tags überall:** Reproduzierbarkeit kaputt → immer Digest pinnen (`@sha256:...`)
- **Chainguard Free Tier:** im PoC perfekt, in Production teuer – früh kommunizieren
