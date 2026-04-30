# 📱 Android Root Detection Bypass en utilisant Frida

##  Objectif du TP

Ce laboratoire a pour objectif de comprendre comment les applications Android détectent un appareil rooté, et comment contourner ces mécanismes en utilisant l’outil d’instrumentation dynamique **Frida**.

---

##  Concepts abordés

* Détection de root (Java & natif)
* Instrumentation dynamique avec Frida
* Hooking de fonctions sensibles
* Analyse dynamique d’une application Android
* Bypass des protections de sécurité

---

## Environnement

* OS : Windows
* Outil : Frida
* Émulateur Android (Android Studio)
* ADB (Android Debug Bridge)
* Application cible : OWASP MSTG Uncrackable Level 2

---

##  Partie 1 — Installation et vérification

### 🔹 Vérification de Frida

```bash
frida --version
```
```bash
python -c "import frida; print(frida.__version__)"
```
<img width="932" height="100" alt="1" src="https://github.com/user-attachments/assets/9b53f146-5dc2-4988-a0ea-390b36eff0c5" />
<img width="785" height="85" alt="2" src="https://github.com/user-attachments/assets/110b638f-0642-48fc-bfeb-2c1459b5d44b" />


### 🔹 Vérification de la connexion ADB

```bash
adb devices
```
<img width="928" height="105" alt="3" src="https://github.com/user-attachments/assets/7861972d-7443-452e-84a8-5bf7b5e6bb9c" />

 **Résultat** : appareil détecté (emulator-5554)

---

##  Partie 2 — Déploiement de Frida

### 🔹 Lancement de frida-server

```bash
adb push frida-server /data/local/tmp/
<img width="1023" height="70" alt="image" src="https://github.com/user-attachments/assets/3a52b881-6f4b-4dde-8759-76beef4b74f0" />
adb shell chmod 755 /data/local/tmp/frida-server
adb shell "/data/local/tmp/frida-server &"
```
<img width="1102" height="92" alt="4" src="https://github.com/user-attachments/assets/d64ff146-d30a-40a9-bb81-46bbbc9f63fa" />

### 🔹 Vérification

```bash
adb shell ps | findstr frida
```
<img width="1107" height="67" alt="5" src="https://github.com/user-attachments/assets/118e3270-31d9-4e80-82e0-3e610139817d" />

**frida-server actif**

### 🔹 Liste des applications

```bash
frida-ps -Uai
```
<img width="965" height="487" alt="6" src="https://github.com/user-attachments/assets/56e46c3c-f2d7-4d02-b9fb-786058ef539b" />

 Applications visibles (Chrome, Settings, Uncrackable Level 2…)

---

##  Partie 3 — Bypass Java

### 🔹 Script utilisé

`bypass_root.js`
<img width="1417" height="723" alt="7" src="https://github.com/user-attachments/assets/50537ece-e662-4c3e-9d66-6bf0f828686a" />

### 🔹 Exécution

```bash
frida -U -f owasp.mstg.uncrackable2 -l bypass_root.js
```
<img width="1077" height="556" alt="8" src="https://github.com/user-attachments/assets/0d416279-5477-4fde-a6c0-d3605390382d" />

### 🔹 Résultats

* Hook de `Build.TAGS`
* Interception de `File.exists`
* Blocage de `Runtime.exec`
* Neutralisation des checks root

### 🔹 Exemple de logs

```text
[+] Hook Build.TAGS -> release-keys
[+] File.exists bypass for /system/bin/su
[+] Hooks Runtime.exec installés
[+] Java layer bypass installed
```

 Le script empêche l’application de détecter le root au niveau Java ✔️
<img width="594" height="567" alt="9" src="https://github.com/user-attachments/assets/8af39b2c-7c8d-4463-a7e3-fb4e20e63bbd" />
<img width="446" height="552" alt="10" src="https://github.com/user-attachments/assets/8773a140-ef8f-426c-b105-484e34bcca35" />

---

##  Partie 4 — Analyse native

### 🔹 Objectif

Intercepter les appels natifs utilisés pour détecter le root :

* open
* access
* stat
* openat
* lstat

---

### 🔹 Script utilisé

`bypass_native.js`
<img width="1412" height="726" alt="11" src="https://github.com/user-attachments/assets/7aedcdff-b5fb-4b91-9410-b943c1b83b1f" />

### 🔹 Principe

Le script intercepte les fonctions natives et bloque l’accès aux fichiers sensibles :

```text
/system/bin/su
/system/xbin/su
/system/bin/busybox
```

---

### 🔹 Exécution

```bash
frida -U -f owasp.mstg.uncrackable2 -l bypass_native.js
```
<img width="1212" height="327" alt="12" src="https://github.com/user-attachments/assets/22de4ea1-0e01-4cbc-bab0-1d481ded5761" />

---

### 🔹 Résultat

✔️ Hooks installés
✔️ Script prêt à bloquer les appels natifs

⚠️ Aucun log `[+] Blocked` observé car :

* l’application utilise principalement Java
* protections anti-debug
* exécution rapide

---

##  Analyse

Ce TP montre que :

* Les applications Android utilisent plusieurs techniques pour détecter le root
* Ces techniques peuvent être contournées en interceptant les appels système
* Frida permet de modifier le comportement d’une application en temps réel
* Le bypass peut être effectué sans modifier l’APK

---

##  Conclusion

Ce laboratoire nous a permis de :

* Comprendre les mécanismes de détection de root
* Utiliser Frida pour effectuer du hooking dynamique
* Contourner les protections Java et natives
* Analyser le comportement interne d’une application Android

---

## ⚠️ Avertissement

Ces techniques doivent être utilisées uniquement dans un cadre légal et éthique (tests de sécurité autorisés).

---

## 👩‍💻 Auteur

ASSEKNOUR SANA
4ème année — Génie Cyberdéfense
ENSA Marrakech
