# 🔐 Rapport d'Audit de Sécurité - Application Android (Sieve)

## Informations générales

| Champ | Détail |
|-------|--------|
| **Application** | Sieve |
| **Package** | com.withsecure.example.sieve |
| **Version** | 1.0 |
| **Date d'audit** | 2026-05-11 |
| **Outil principal** | Drozer 3.1.0 |
| **Émulateur** | Android SDK x86_64 - API 30 (Android 11) |

---

##  Résumé exécutif

L'application Sieve présente plusieurs vulnérabilités critiques liées à des composants Android mal configurés. L'audit a permis d'identifier 7 vulnérabilités dont 2 critiques exposant directement les mots de passe des utilisateurs sans aucune authentification.

---

##  Méthodologie

- Vérification de l'environnement (ADB, Drozer, émulateur)
- Installation et configuration de l'agent Drozer
- Cartographie des composants Android exposés
- Analyse des permissions et intent-filters
- Identification des URIs accessibles
- Analyse des risques et documentation

---

##  Étape 1 — Configuration de l'environnement

> 📸 **[CAPTURE — Émulateur Android avec agent Drozer activé]**

```
adb install drozer-agent.apk   → Success
adb install sieve.apk          → Success
adb forward tcp:31415 tcp:31415 → 31415
```

---

##  Étape 2 — Connexion Drozer

>  **[CAPTURE — Console Drozer connectée]**

```
drozer console connect
Selecting 776b684bafc9e96e (unknown Android SDK built for x86_64 11)
drozer Console (v3.1.0)
dz>
```

---

##  Étape 3 — Cartographie des composants exposés

>  **[CAPTURE — Résultat app.package.info]**

### Activities exportées
```
dz> run app.activity.info -a com.withsecure.example.sieve
```
>  **[CAPTURE — Résultat app.activity.info]**

| Activity | Permission |
|----------|-----------|
| MainLoginActivity | ❌ null |
| FileSelectActivity | ❌ null |
| PWList | ❌ null |

### Services exportés
```
dz> run app.service.info -a com.withsecure.example.sieve
```
>  **[CAPTURE — Résultat app.service.info]**

| Service | Permission |
|---------|-----------|
| AuthService | ❌ null |
| CryptoService | ❌ null |

### Broadcast Receivers
```
dz> run app.broadcast.info -a com.withsecure.example.sieve
```
> **[CAPTURE — Résultat app.broadcast.info]**

| Receiver | Permission |
|----------|-----------|
| ProfileInstallReceiver | ✅ android.permission.DUMP |

### Content Providers
```
dz> run app.provider.info -a com.withsecure.example.sieve
```
>  **[CAPTURE — Résultat app.provider.info]**

| Provider | Read Permission | Write Permission |
|----------|----------------|-----------------|
| DBContentProvider | ❌ null (sauf /Keys) | ❌ null (sauf /Keys) |
| FileBackupProvider | ❌ null | ❌ null |

---

##  Étape 4 — Vérification des protections

### Analyse du manifeste
```
dz> run app.package.manifest com.withsecure.example.sieve
```
>  **[CAPTURE — Manifeste AndroidManifest.xml]**

**Observations critiques :**
- `debuggable="true"` → dangereux en production
- `allowBackup="true"` → extraction de données possible
- `protectionLevel="0x1"` (dangerous) sur READ_KEYS/WRITE_KEYS → trop faible

### URIs accessibles
```
dz> run scanner.provider.finduris -a com.withsecure.example.sieve
```
>  **[CAPTURE — Résultat scanner.provider.finduris]**

**URIs accessibles sans permission :**
```
content://com.withsecure.example.sieve.provider.DBContentProvider/Passwords
content://com.withsecure.example.sieve.provider.DBContentProvider/Passwords/
content://com.withsecure.example.sieve.provider.DBContentProvider/Keys/
```

---

##  Étape 5 — Analyse des risques

### Tableau de triage

| ID | Composant | Vulnérabilité | Sévérité | Impact |
|----|-----------|--------------|----------|--------|
| V1 | PWList | Activity exportée sans protection | 🔴 Critique | Accès direct aux mots de passe |
| V2 | DBContentProvider/Passwords | URI accessible sans permission | 🔴 Critique | Fuite de tous les mots de passe |
| V3 | FileSelectActivity | Exportée sans raison | 🔴 Élevé | Accès fichiers non autorisé |
| V4 | AuthService | Service exporté sans permission | 🔴 Élevé | Bypass authentification |
| V5 | CryptoService | Service exporté sans permission | ⚠️ Moyen | Opérations crypto non autorisées |
| V6 | Application | debuggable=true | ⚠️ Moyen | Risque en production |
| V7 | Application | allowBackup=true | ⚠️ Moyen | Extraction données via backup |

---

##  Mapping OWASP MASVS

| ID | Vulnérabilité | Référence MASVS | Description |
|----|--------------|----------------|-------------|
| V1 | PWList exportée sans protection | MSTG-PLATFORM-1 | L'app ne doit exposer que les composants nécessaires |
| V2 | DBContentProvider mal protégé | MSTG-STORAGE-2 | Aucune donnée sensible sans protection adéquate |
| V3 | AuthService sans validation | MSTG-PLATFORM-2 | Les entrées externes doivent être validées |
| V4 | Receivers sans validation | MSTG-PLATFORM-3 | L'app doit valider les intents reçus |
| V5 | Permissions insuffisantes | MSTG-AUTH-1 | Les mécanismes d'auth doivent être robustes |

---

##  Remédiations

### 1. PWList et FileSelectActivity
```xml
<!-- Avant -->
<activity android:name=".PWList" android:exported="true" />

<!-- Après -->
<activity android:name=".PWList" android:exported="false" />
```

### 2. DBContentProvider
```xml
<!-- Avant -->
<provider android:name=".DBContentProvider" android:exported="true" />

<!-- Après -->
<provider
    android:name=".DBContentProvider"
    android:exported="true"
    android:readPermission="com.withsecure.example.sieve.READ_KEYS"
    android:writePermission="com.withsecure.example.sieve.WRITE_KEYS" />
```

### 3. AuthService et CryptoService
```xml
<!-- Avant -->
<service android:name=".AuthService" android:exported="true" />

<!-- Après -->
<service
    android:name=".AuthService"
    android:exported="false" />
```

### 4. Application
```xml
<!-- Avant -->
<application android:debuggable="true" android:allowBackup="true" />

<!-- Après -->
<application android:debuggable="false" android:allowBackup="false" />
```

### 5. Permissions — niveau signature
```xml
<permission
    android:name="com.withsecure.example.sieve.READ_KEYS"
    android:protectionLevel="signature" />
```

---

##  Structure du dossier de preuves

```
audit/
├── rapport_final.md
├── triage.csv
├── checklist_fin.md
├── activities/
│   └── exported_activities.txt
├── services/
│   └── exported_services.txt
├── receivers/
│   └── exported_receivers.txt
└── providers/
    └── exported_providers.txt
```

---

## ✅ Checklist de fin d'audit

- [x] Toutes les étapes du lab ont été suivies
- [x] Tous les composants Android ont été analysés
- [x] Le tableau de triage est complet
- [x] Les remédiations proposées sont spécifiques et applicables
- [x] Le mapping OWASP MASVS est correct
- [x] Aucune donnée utilisateur réelle dans le rapport
- [x] Le rapport est bien structuré
