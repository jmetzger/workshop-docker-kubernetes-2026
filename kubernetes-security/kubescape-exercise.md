# Kubescape - Kubernetes Security Scanning mit Framework-Compliance

## Hintergrund: kubescape und seine Betriebsmodi

kubescape ist ein Open-Source-Security-Scanner (CNCF Sandbox) von ARMO.
Er prueft Kubernetes-Cluster gegen bekannte Sicherheits-Frameworks und zeigt
konkrete Findings mit Remediation-Hinweisen.

**Betriebsmodi:**

| Modus | Beschreibung | Einsatz |
|-------|-------------|---------|
| **CLI** | Einmalige oder gescriptete Scans | CI/CD, Ad-hoc-Audits |
| **Operator** | Laeuft permanent im Cluster, integriert sich mit Kubescape Cloud | Kontinuierliches Monitoring, Dashboards |

In dieser Uebung nutzen wir die **CLI**.

---

## Hintergrund: Security Frameworks im Ueberblick

Kubescape unterstuetzt mehrere Frameworks. Jedes Framework deckt andere
Aspekte ab und kommt aus einem anderen Kontext:

| Framework | Herausgeber | Fokus | Gut fuer |
|-----------|-------------|-------|---------|
| **NSA** | US National Security Agency | Minimalhaertung fuer produktive Cluster | Einstiegspunkt, breite Coverage |
| **MITRE ATT&CK** | MITRE Corporation | Angreiferverhalten: Taktiken und Techniken | Verstehen wie Angreifer vorgehen |
| **CIS (cis-v1.10.0)** | Center for Internet Security | Detaillierte, nummerierte Controls fuer jede Komponente | Compliance-Audits, konkrete Massnahmen |
| **ArmoBest** | ARMO (kubescape-Entwickler) | Kombination aus NSA + MITRE + eigene Best Practices | Schneller Ueberblick, empfohlen fuer Einsteiger |
| **SOC2** | AICPA | Organisatorische und technische Kontrollen | Regulatorische Compliance |
| **DevOpsBest** | ARMO | Reliabilitaet und Operabilit | Staging/Dev-Cluster |

Alle Controls sind unter https://hub.armosec.io/docs/ dokumentiert.

---

## Schritt 1: Kubescape installieren

```
cd
curl -s https://raw.githubusercontent.com/kubescape/kubescape/master/install.sh | bash
```

Das Installationsskript legt kubescape unter `~/.kubescape/bin/` ab.
PATH fuer diese Session setzen:

```
export PATH=$PATH:/home/tln1/.kubescape/bin
kubescape version
```

**Erwartete Ausgabe:**
```
Your current version is: v4.0.6
```

> **Tipp:** Um den PATH dauerhaft zu setzen:
> ```
> echo 'export PATH=$PATH:/home/tln1/.kubescape/bin' >> ~/.bashrc
> ```

---

## Schritt 2: Verfuegbare Frameworks und Controls anzeigen

Welche Frameworks sind verfuegbar?

```
kubescape list frameworks
```

Alle Controls mit ihren Framework-Zuordnungen:

```
kubescape list controls
```

Die Tabelle zeigt pro Control: ID, Name, Doku-Link, und in welchen Frameworks
das Control vorkommt. RBAC-relevante Controls erkennst du an den Namen:
"List Kubernetes secrets", "Administrative Roles", "Roles with delete capabilities".

---

## Schritt 3: Ersten Cluster-Scan laufen lassen

NSA-Framework als Einstieg — gibt einen guten Gesamtueberblick:

```
kubescape scan framework NSA
```

Die Ausgabe gliedert sich in:
1. Zusammenfassung: Wie viele Controls passed/failed?
2. Fehler nach Severity (High/Medium/Low)
3. Tabelle: pro Control — wie viele Ressourcen betroffen?

Compliance-Score: Wie viel Prozent der Ressourcen bestehen alle Controls?

Schaue dir die RBAC-relevanten Zeilen an:

```
│ Medium   │ Administrative Roles              │ ...
│ Medium   │ Automatic mapping of service account │ ...
```

Um Details zu sehen — welche genauen Ressourcen betroffen sind:

```
kubescape scan framework NSA --verbose
```

---

## Schritt 4: RBAC-Problem absichtlich erstellen

Wir simulieren einen haeufigen Fehler: ein Service Account bekommt
deutlich zu viele Rechte — Secrets lesen/schreiben plus delete-Rechte.

Verzeichnis anlegen:

```
cd
mkdir -p manifests/kubescape-rbac
cd manifests/kubescape-rbac
```

Manifests erstellen:

```
# vi 01-overprivileged.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: overprivileged-sa
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: demo-bad-role
rules:
- apiGroups: [""]
  resources: ["secrets", "pods", "nodes"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["clusterroles", "clusterrolebindings"]
  verbs: ["get", "list", "create", "update", "patch", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: demo-bad-binding
subjects:
- kind: ServiceAccount
  name: overprivileged-sa
  namespace: default
roleRef:
  kind: ClusterRole
  name: demo-bad-role
  apiGroup: rbac.authorization.k8s.io
```

```
kubectl apply -f 01-overprivileged.yml
```

**Was ist hier problematisch?**

| Problem | Warum gefaehrlich |
|---------|------------------|
| `secrets` mit `list/get` | Angreifer kann alle Secrets im Cluster auslesen |
| `secrets` mit `delete` | Angreifer kann Secrets loeschen (DoS, Sabotage) |
| `clusterroles` mit `create/update` | Privilege Escalation: neue Admin-Rollen erstellen |
| ClusterRoleBinding statt RoleBinding | Zugriff cluster-weit, nicht nur im Namespace |

---

## Schritt 5: RBAC-Schwachstellen gezielt scannen

Nur die RBAC-relevanten Controls pruefen:

```
kubescape scan control C-0015,C-0007 --verbose
```

Controls in dieser Auswahl:

| Control ID | Name | Severity |
|------------|------|---------|
| C-0015 | List Kubernetes secrets | High |
| C-0007 | Roles with delete capabilities | Medium |

**Erwartete Ausgabe — der overprivileged-sa wird gefunden:**

```
################################################################################
ApiVersion:
Kind: ServiceAccount
Name: overprivileged-sa
Namespace: default

Controls: 2 (Failed: 2, action required: 0)

| Severity | Control name                   |
| High     | List Kubernetes secrets        |
| Medium   | Roles with delete capabilities |
```

Das sind genau die beiden Probleme: Secrets-Zugriff (C-0015) und
delete-Rechte (C-0007).

Zusaetzlich Administrative Roles pruefen:

```
kubescape scan control C-0035 --verbose
```

C-0035 flaggt Service Accounts die cluster-admin-aehnliche Rechte haben.

---

## Schritt 6: RBAC-Problem beheben

Die Regel: so wenig Rechte wie moeglich, so viel wie noetig (Least Privilege).

Fix-Manifest erstellen:

```
# vi 02-fixed-role.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: demo-bad-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

Kein Secrets-Zugriff mehr, kein delete, kein RBAC-Management.
Falls der Service Account nur Pods im eigenen Namespace lesen muss:
lieber ein `Role` + `RoleBinding` statt `ClusterRole` verwenden.

```
kubectl apply -f 02-fixed-role.yml
```

```
clusterrole.rbac.authorization.k8s.io/demo-bad-role configured
```

---

## Schritt 7: Verifizierung nach dem Fix

```
kubescape scan control C-0015,C-0007 --verbose
```

**Erwartetes Ergebnis:** `overprivileged-sa` erscheint nicht mehr in den Findings.

```
# grep-freundliche Pruefung:
kubescape scan control C-0015,C-0007 --verbose 2>&1 | grep "overprivileged"
```

Keine Ausgabe = kein Finding mehr fuer unseren Service Account.

Gesamtbild vor/nach dem Fix — NSA-Scan wiederholen und Score vergleichen:

```
kubescape scan framework NSA
```

Der Compliance-Score steigt, weil `overprivileged-sa` nicht mehr in
C-0015 und C-0007 faellt.

---

## Exkurs: Kubescape als Operator betreiben

Neben der CLI kann kubescape auch als **Operator** im Cluster installiert werden.
Der Operator laeuft als Controller und scannt kontinuierlich:

```
# Installation via Helm (nur Info, nicht in dieser Uebung ausfuehren):
helm repo add kubescape https://kubescape.github.io/helm-charts/
helm upgrade --install kubescape kubescape/kubescape-operator \
  -n kubescape --create-namespace \
  --set clusterName=my-cluster
```

| CLI | Operator |
|-----|---------|
| Einmaliger Scan | Kontinuierliches Monitoring |
| Kein Cluster-Overhead | Laeuft als Pods im Cluster |
| Gut fuer CI/CD | Gut fuer Dashboards und Alerting |
| Kein Account noetig | Integration mit Kubescape Cloud (optional) |

Dokumentation: https://kubescape.io/docs/install-operator/

---

## Aufraeumen

```
kubectl delete clusterrolebinding demo-bad-binding
kubectl delete clusterrole demo-bad-role
kubectl delete serviceaccount overprivileged-sa -n default
```

---

## Zusammenfassung

| Was pruefen | Befehl |
|-------------|--------|
| Alle Frameworks auflisten | `kubescape list frameworks` |
| Alle Controls auflisten | `kubescape list controls` |
| NSA-Framework scannen | `kubescape scan framework NSA` |
| Einzelne Controls pruefen | `kubescape scan control C-0015,C-0007` |
| Mit Detail-Ausgabe | `kubescape scan control ... --verbose` |

RBAC-relevante Controls im Ueberblick:

| Control ID | Name | Was wird geprueft |
|------------|------|------------------|
| C-0015 | List Kubernetes secrets | Subjects mit Secrets-Lesezugriff |
| C-0007 | Roles with delete capabilities | Subjects mit delete-Verben |
| C-0035 | Administrative Roles | Subjects mit cluster-admin-aehnlichen Rechten |
| C-0034 | Automatic mapping of service account | Pods die automatisch SA-Token mounten |

## Referenzen

  * https://kubescape.io/
  * https://hub.armosec.io/docs/ (Control-Dokumentation)
  * https://kubescape.io/docs/install-operator/
  * https://github.com/kubescape/kubescape
