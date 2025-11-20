# Kanata – Autostart macOS (LaunchDaemon)

Ce guide explique comment configurer **Kanata** pour qu'il se lance automatiquement au démarrage de macOS, en utilisant un **LaunchDaemon root**.

Il inclut :

* le fichier `.plist` complet
* les commandes nécessaires
* la gestion des permissions
* la vérification des logs
* la procédure de reload/bootout en cas de mise à jour

---

## 1. Prérequis

### Autorisation macOS obligatoire

Pour que Kanata puisse intercepter le clavier, ajoute :

```
System Settings → Privacy & Security → Input Monitoring
```

Ajouter le binaire :

```
/Users/karim/.config/kanata/kanata_macos_cmd_allowed_arm64
```

Puis redémarrer ou se déconnecter/reconnecter.

---

## 2. Fichier LaunchDaemon

Créer le fichier :

```
sudo nano /Library/LaunchDaemons/com.kanata.root.plist
```

Coller :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.kanata.root</string>

    <key>ProgramArguments</key>
    <array>
      <string>/Users/karim/.config/kanata/kanata_macos_cmd_allowed_arm64</string>
      <string>-c</string>
      <string>/Users/karim/.config/kanata/kanata-config/gk61.kbd</string>
    </array>

    <!-- Lancer automatiquement au démarrage (root) -->
    <key>RunAtLoad</key>
    <true/>

    <!-- Redémarrage automatique si Kanata s’arrête -->
    <key>KeepAlive</key>
    <true/>

    <!-- S’assurer que Kanata tourne en root -->
    <key>UserName</key>
    <string>root</string>
  </dict>
</plist>
```

---

## 3. Permissions correctes pour LaunchDaemon

macOS **refuse de charger** un daemon dont les permissions ne sont pas strictes.

Exécuter :

```
sudo chown root:wheel /Library/LaunchDaemons/com.kanata.root.plist
sudo chmod 644 /Library/LaunchDaemons/com.kanata.root.plist
```

---

## 4. Charger le daemon

```
sudo launchctl load /Library/LaunchDaemons/com.kanata.root.plist
```

Vérification :

```
sudo launchctl list | grep kanata
```

---

## 5. Vérifier les logs de Kanata

Kanata écrit ses logs ici :

```
sudo cat /var/log/kanata.log
sudo cat /var/log/kanata-error.log
```

---

## 6. Redémarrer / mettre à jour le daemon

Si vous modifiez la config `.kbd` ou le binaire, utilisez :

```
sudo launchctl bootout system /Library/LaunchDaemons/com.kanata.root.plist
sudo launchctl bootstrap system /Library/LaunchDaemons/com.kanata.root.plist
```

---

## 7. Vérifier que tout fonctionne

1. Le daemon doit apparaître dans `launchctl list`
2. Aucune erreur dans `kanata-error.log`
3. Les logs doivent montrer :

   * `config file is valid`
   * `entering the event loop`

Si tout est bon : **Kanata démarre maintenant automatiquement à chaque boot du Mac**.

---

## Notes

* Kanata tourne au niveau **root** pour intercepter le clavier bas-niveau.
* `cmd action enabled` est normal si vous utilisez les commandes systèmes Kanata.
* Le lancement via LaunchDaemon est plus fiable qu’un LaunchAgent pour capturer le clavier dès le login.
