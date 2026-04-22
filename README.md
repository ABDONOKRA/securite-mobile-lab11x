# 🔓 Securite Mobile — Lab 11 : OWASP UnCrackable Level 3
 
> **Objectif :** Bypasser les protections root detection et anti-tamper (détection Frida) de l'application **OWASP MAS Crackme Level 3** en utilisant **Frida Dynamic Instrumentation**.
 
---
 
## 📋 Table des matières
 
1. [Préparation de l'environnement](#1-préparation-de-lenvironnement)
2. [Installation de l'APK](#2-installation-de-lapk)
3. [Lancement de frida-server](#3-lancement-de-frida-server)
4. [Vérification de l'appareil](#4-vérification-de-lappareil)
5. [Analyse des techniques de détection de root](#5-analyse-des-techniques-de-détection-de-root)
6. [Script Frida — Bypass Java](#6-script-frida--bypass-java)
7. [Premier lancement — App crashe](#7-premier-lancement--app-crashe)
8. [Ajout des hooks natifs](#8-ajout-des-hooks-natifs)
9. [Problème de version Frida](#9-problème-de-version-frida)
10. [Solution — Downgrade Frida](#10-solution--downgrade-frida)
11. [Script final — Bypass complet](#11-script-final--bypass-complet)
12. [Résultat final](#12-résultat-final)
---
 
## 1. Préparation de l'environnement
 
Configuration de l'environnement de test avec l'émulateur Android et les outils nécessaires.
 
<img width="1089" height="266" alt="Préparation environnement" src="https://github.com/user-attachments/assets/83b499a9-cf64-4a13-87f4-814f1ff6be12" />
---
 
## 2. Installation de l'APK
 
Installation de l'application **OWASP UnCrackable Level 3** sur l'émulateur Android.
 
```bash
adb install UnCrackable-Level3.apk
```
 
<img width="612" height="949" alt="APK installé" src="https://github.com/user-attachments/assets/cd466c33-83a7-4ef9-9f4c-360712b86802" />
---
 
## 3. Lancement de frida-server en arrière-plan
 
Déploiement et démarrage de `frida-server` sur l'émulateur.
 
```bash
adb push frida-server /data/local/tmp/frida-server
adb shell chmod 755 /data/local/tmp/frida-server
adb shell "/data/local/tmp/frida-server &"
```
 
<img width="1144" height="160" alt="Lancement frida-server" src="https://github.com/user-attachments/assets/762d6fc3-d844-4e62-9e52-f1893ec50cf0" />
---
 
## 4. Vérification que l'appareil est visible
 
Confirmation que Frida détecte bien l'émulateur connecté via ADB.
 
```bash
frida-ls-devices
frida-ps -U
```
 
<img width="894" height="617" alt="Appareil visible" src="https://github.com/user-attachments/assets/ca018a64-88c9-4d47-aebd-9dec941000fd" />
---
 
## 5. Analyse des techniques de détection de root
 
L'application implémente plusieurs couches de protection contre le root et le tampering :
 
### 🔵 Couche Java (haut niveau)
 
| Technique | Description |
|-----------|-------------|
| `Build.TAGS` | Détecte la présence de `test-keys` |
| `File.exists()` | Recherche `/system/xbin/su`, `/system/bin/su`, `busybox` |
| `Runtime.exec("su")` | Exécute `su`, `which su`, `busybox` |
| RootBeer | Bibliothèque tierce avec `isRooted()` |
 
### 🔴 Couche Native (JNI / C / C++)
 
| Technique | Description |
|-----------|-------------|
| `open/openat/access/stat/lstat` | Appels vers chemins suspects |
| `/proc/mounts` | Lecture des montages système |
| `/proc/self/maps` | Scan des mappings mémoire pour détecter `frida` |
| Anti-debug | Scan de ports, recherche de strings `"frida"` |
 
---
 
## 6. Script Frida — Bypass Java
 
Premier script ciblant uniquement la couche Java. Objectif : forcer des retours « non root » et bloquer l'accès aux indicateurs.
 
<img width="1920" height="977" alt="Script bypass Java" src="https://github.com/user-attachments/assets/f6707987-4f15-42d4-a18d-f4e3a2120fb8" />
**Lancement :**
 
<img width="1920" height="548" alt="Lancement script" src="https://github.com/user-attachments/assets/91f7d2a5-4063-429d-9b54-f55b9dbc30de" />
---
 
## 7. Premier lancement — App crashe
 
Malgré le bypass Java, l'application crashe à cause de la **protection native** dans `libfoo.so` qui détecte Frida et appelle `goodbye()` → `SIGABRT`.
 
<img width="480" height="874" alt="App crashée" src="https://github.com/user-attachments/assets/1f4f8574-c72e-48c2-9f38-ebe5d17bec2f" />
<img width="1920" height="890" alt="Stacktrace crash" src="https://github.com/user-attachments/assets/45436d73-851c-4cb2-a65b-4e536c65f998" />
> **Cause :** Un thread lancé au démarrage lit `/proc/self/maps` et appelle `strstr()` pour détecter le mot `"frida"` dans les mappings mémoire.
 
---
 
## 8. Ajout des hooks natifs
 
Ajout de hooks sur les fonctions natives (`open`, `openat`, `access`, `stat`, `lstat`) pour intercepter les vérifications au niveau C/C++.
 
<img width="1918" height="546" alt="Hooks natifs" src="https://github.com/user-attachments/assets/16efe6e5-900d-4699-9690-f81cd4ed1e16" />
<img width="764" height="908" alt="Script hooks natifs" src="https://github.com/user-attachments/assets/7137f6b1-2678-4eac-ad26-e8792c91207a" />
<img width="453" height="68" alt="Résultat hooks" src="https://github.com/user-attachments/assets/b920d2b1-fb64-4d37-aae6-a1fbee1b9051" />
**Vérification que frida-server tourne sur l'émulateur :**
 
<img width="1110" height="105" alt="frida-server running" src="https://github.com/user-attachments/assets/88f714b5-6a70-4a35-ac7b-eb8c175ab2de" />
---
 
## 9. Problème de version Frida
 
**Erreur rencontrée :** `TypeError: not a function` sur `Interceptor.attach` avec **Frida 17.9.1** — incompatibilité avec l'émulateur x86_64 Android 11.
 
Recherche et collecte des ressources pour identifier le problème :
 
<img width="1608" height="785" alt="Recherche solution version Frida" src="https://github.com/user-attachments/assets/61d02a2d-ba72-4871-a794-267b8faf57f5" />
---
 
## 10. Solution — Downgrade Frida vers 16.1.4
 
```bash
# Désinstaller la version incompatible
pip uninstall frida frida-tools -y --break-system-packages
 
# Installer la version stable compatible
pip install frida==16.1.4 frida-tools==12.3.0 --break-system-packages
 
# Télécharger frida-server correspondant
wget https://github.com/frida/frida/releases/download/16.1.4/frida-server-16.1.4-android-x86_64.xz
xz -d frida-server-16.1.4-android-x86_64.xz
adb push frida-server-16.1.4-android-x86_64 /data/local/tmp/frida-server
adb shell chmod 755 /data/local/tmp/frida-server
adb shell "killall frida-server 2>/dev/null; sleep 1; /data/local/tmp/frida-server &"
```
 
> ✅ **Frida 16.1.4** — `Interceptor.attach` et `NativeCallback` fonctionnent correctement.
 
---
 
## 11. Script final — Bypass complet
 
Script combinant le bypass de `strstr`, `fgets` et la root detection Java :
 
```javascript
// bypass_final.js
 
// ── 1. Hook strstr — empêche la détection de "frida" dans /proc/self/maps ──
Interceptor.attach(Module.findExportByName("libc.so", "strstr"), {
    onEnter: function(args) {
        try {
            var needle = args[1].readCString();
            if (needle && (needle.indexOf("frida") !== -1 || needle.indexOf("xposed") !== -1)) {
                this.fake = true;
                console.log("[+] strstr bloqué: " + needle);
            }
        } catch(e) {}
    },
    onLeave: function(retval) {
        if (this.fake) {
            retval.replace(ptr(0));
            this.fake = false;
        }
    }
});
console.log("[+] strstr hookée");
 
// ── 2. Hook fgets — efface les lignes "frida" dans les lectures fichier ──
var fgetsPtr = Module.findExportByName("libc.so", "fgets");
var fgets = new NativeFunction(fgetsPtr, 'pointer', ['pointer', 'int', 'pointer']);
Interceptor.replace(fgetsPtr, new NativeCallback(function(buffer, size, fp) {
    var retval = fgets(buffer, size, fp);
    try {
        var line = Memory.readUtf8String(buffer);
        if (line && line.indexOf("frida") !== -1) {
            Memory.writeUtf8String(buffer, "");
            console.log("[+] fgets: ligne frida effacée");
        }
    } catch(e) {}
    return retval;
}, 'pointer', ['pointer', 'int', 'pointer']));
console.log("[+] fgets hookée");
 
// ── 3. Bypass Java — Root Detection + Dialog + verifyLibs ──
Java.perform(function () {
 
    try {
        var MainActivity = Java.use("sg.vantagepoint.uncrackable3.MainActivity");
        MainActivity.showDialog.implementation = function(msg) {
            console.log("[+] showDialog bloqué: " + msg);
        };
        console.log("[+] showDialog OK");
    } catch(e) { console.log("[-] showDialog: " + e); }
 
    try {
        var RootDetection = Java.use("sg.vantagepoint.util.RootDetection");
        RootDetection.checkRoot1.implementation = function() { return false; };
        RootDetection.checkRoot2.implementation = function() { return false; };
        RootDetection.checkRoot3.implementation = function() { return false; };
        console.log("[+] RootDetection OK");
    } catch(e) { console.log("[-] RootDetection: " + e); }
 
    try {
        var MC = Java.use("sg.vantagepoint.uncrackable3.MainActivity");
        MC.verifyLibs.implementation = function() {
            console.log("[+] verifyLibs bloquée");
        };
        console.log("[+] verifyLibs OK");
    } catch(e) { console.log("[-] verifyLibs: " + e); }
 
});
```
 
**Lancement :**
 
```bash
frida -U -f owasp.mstg.uncrackable3 -l bypass_final.js
```
 
---
 
## 12. Résultat final
 
L'application s'ouvre sans crash — toutes les protections sont bypassées avec succès ✅
 
<img width="508" height="1000" alt="App ouverte - bypass réussi" src="https://github.com/user-attachments/assets/881b20f8-3445-429c-932d-3e1047e5e52a" />
### Récapitulatif des bypasses
 
| Protection | Technique utilisée | Statut |
|-----------|-------------------|--------|
| Root Detection Java (`checkRoot1/2/3`) | Hook → retourne `false` | ✅ Bypassé |
| Integrity Check (`verifyLibs`) | Hook → corps vide | ✅ Bypassé |
| Anti-Frida natif (`strstr`) | Hook dans `libc.so` → retourne `NULL` | ✅ Bypassé |
| Anti-Frida natif (`fgets`) | Replace → efface lignes `"frida"` | ✅ Bypassé |
| Dialog exit (`showDialog`) | Hook → bloqué | ✅ Bypassé |
 
---
 
## 🔑 Leçons apprises
 
- **Frida 17.x** présente un bug de compatibilité avec `Interceptor.attach` sur émulateurs x86_64 Android 11 → utiliser **Frida 16.1.4**
- Le thread anti-tamper dans `libfoo.so` utilise `strstr` de `libc.so` pour scanner `/proc/self/maps` — il faut hooker `strstr` **dans libc** car elle est disponible **avant** le chargement de `libfoo.so`
- Le vrai package de `RootDetection` est **`sg.vantagepoint.util.RootDetection`** et non `sg.vantagepoint.uncrackable3.RootDetection`
- Les versions de `frida` (PC) et `frida-server` (émulateur) **doivent être identiques**
---
 
*Lab réalisé dans le cadre de la formation **Mobile Security Testing** — OWASP MASTG*
