# Rapport d'audit de securite - Application Android 
 
## Informations generales 
- Application : Sieve 
- Package : com.withsecure.example.sieve 
- Version : 1.0 
- Date d'audit : 2026-05-11 
- Auditeur : LAHROUF HIBA 
 
## Resume executif 
L'application Sieve presente plusieurs vulnerabilites critiques. Trois activities sont exportees sans protection, deux services sont accessibles sans permission, et le content provider expose les mots de passe sans authentification. 
 
## Methodologie 
- Analyse statique du manifeste Android 
- Cartographie des composants exposes avec Drozer 
- Verification des protections en place 
- Analyse des risques potentiels 
 
## Decouvertes principales 
1. PWList accessible sans permission - Critique 
2. DBContentProvider/Passwords expose sans permission - Critique 
3. AuthService exporte sans protection - Eleve 
4. debuggable=true active - Moyen 
5. allowBackup=true active - Moyen 
 
## Recommandations prioritaires 
1. Definir exported=false sur PWList et FileSelectActivity 
2. Ajouter readPermission sur DBContentProvider 
3. Proteger AuthService et CryptoService avec une permission 
4. Desactiver debuggable en production 
5. Desactiver allowBackup ou chiffrer les donnees
