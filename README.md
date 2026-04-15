# Resolution-FireStorm

## 1. Introduction
L'objectif de ce laboratoire est d'extraire un flag stocké dans une base de données Firebase. L'application Android `FireStorm` utilise une logique de génération de mot de passe complexe, mêlant ressources statiques et code natif (C++), rendant l'analyse statique seule insuffisante.

**Compétences démontrées :**
* Reverse Engineering Android (Jadx-GUI).
* Hooking et manipulation de la mémoire (Frida).
* Automatisation d'exploitation (Python & Pyrebase).

---

## 2. Analyse Statique (Reconnaissance)
L'exploration de l'APK avec Jadx-GUI a révélé une méthode `Password()` dans la classe `MainActivity`. Cette méthode construit une chaîne de caractères en concaténant des extraits du fichier `strings.xml` et le résultat d'une fonction native nommée `generateRandomString`.

* **Problématique :** La fonction `Password()` n'est jamais appelée par l'application.
* **Données récupérées :** Email, API Key et Database URL Firebase.

<img width="1600" height="779" alt="image" src="https://github.com/user-attachments/assets/e235f34b-0e27-4e26-8794-da7871a9310f" />
<img width="1600" height="779" alt="image" src="https://github.com/user-attachments/assets/5cc9e76d-a860-4b51-8eb3-e5d10febc88f" />

---

## 3. Instrumentation Dynamique avec Frida
Pour obtenir le mot de passe sans avoir à décompiler la librairie native `.so`, nous avons utilisé Frida pour forcer l'exécution de la méthode `Password()` directement en mémoire alors que l'application était active.

**Environnement :** Émulateur Android (x86_64) avec accès Root.

**Script de Hooking (`solve.js`) :**
```javascript
Java.perform(function() {
    Java.choose('com.pwnsec.firestorm.MainActivity', {
        onMatch: function(instance) {
            console.log("[+] Instance de MainActivity trouvée.");
            var password = instance.Password();
            console.log("[!] Mot de passe extrait : " + password);
        },
        onComplete: function() { console.log("[*] Scan terminé."); }
    });
});
```
<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/95f62927-0257-4771-a8f0-53bcb8780d9a" />

---

## 4. Exploitation et Récupération du Flag
Une fois le mot de passe dynamique obtenu, nous avons utilisé un script Python pour simuler une authentification légitime auprès du serveur Firebase et lire les données protégées.

**Script d'exploitation (`get_flag.py`) :**
```python
import pyrebase

config = {
    "apiKey": "AIzaSyAXsK0qsx4RuLSA9C8IPSWd0eQ67HVHuJY",
    "authDomain": "firestorm-9d3db.firebaseapp.com",
    "databaseURL": "https://firestorm-9d3db-default-rtdb.firebaseio.com",
    "projectId": "firestorm-9d3db"
}

firebase = pyrebase.initialize_app(config)
auth = firebase.auth()
db = firebase.database()

# Utilisation du mot de passe récupéré via Frida
user = auth.sign_in_with_email_and_password("TK757567@pwnsec.xyz", "VOTRE_PASS_FRIDA")
flag = db.get(user['idToken']).val()
print(f"Flag : {flag}")
```
<img width="1527" height="248" alt="Untitled" src="https://github.com/user-attachments/assets/f791319b-e522-4a17-9713-c962ffad6279" />

---

## 5. Conclusion
Le succès de ce lab démontre que la sécurité par l'obscurité est inefficace contre l'instrumentation dynamique. Bien que le mot de passe ait été "caché" dans du code natif et jamais utilisé par l'interface, Frida nous a permis de l'extraire en quelques secondes.

**Flag Final :** `PWNSEC{C0ngr4ts_Th4t_w45_4N_345y_P4$$w0rd_t0_G3t!!!0R!5_!t???}`
