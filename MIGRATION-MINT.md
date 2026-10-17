# Threema iPhone → Android (Linux Mint) – erprobter Ablauf

Stand 17.10.2026. iPhone ist per MDM gezwungen, Backups zu verschlüsseln – deshalb
verschlüsseltes Backup + Entschlüsselung auf dem PC.

## Einmalig
```
sudo apt install libimobiledevice-utils usbmuxd python3-venv sqlite3
python3 -m venv ~/threema-venv
~/threema-venv/bin/pip install iphone_backup_decrypt pyzipper
idevicepair pair          # iPhone entsperrt, "Vertrauen" antippen
```

## Ablauf
1. iPhone-Backup (iPhone angesteckt + entsperrt), bis "Backup Successful":
   `idevicebackup2 backup ~/iphone-backup`
2. Threema-Dateien extrahieren (fragt **iPhone**-Backup-Passwort):
   `rm -rf ~/threema-ios ~/threema-ios_raw`
   `~/threema-venv/bin/python src/extract_ios_backup.py ~/iphone-backup/<UDID> ~/threema-ios`
3. Probelauf (Summary prüfen):
   `~/threema-venv/bin/python src/main.py ~/threema-ios/ThreemaData.sqlite ~/threema-test`
4. Android: Threema mit derselben ID einrichten (Threema Safe), dann
   ⋮ → Backups → Daten-Backup → Pfad festlegen → "Daten-Backup erstellen".
   ZIP per USB ("Dateien übertragen") als `~/threema-android.zip` auf den PC.
   (Ein vorhandenes `threema-android.zip` mit derselben ID kann wiederverwendet werden.)
5. Finales Backup bauen (**Android**-Backup-Passwort):
   `rm -rf ~/threema-output`
   `~/threema-venv/bin/python src/main.py ~/threema-ios/ThreemaData.sqlite ~/threema-output --android-backup ~/threema-android.zip --password 'PW' --target ~/threema-migriert.zip`
6. `threema-migriert.zip` in Download-Ordner des Android-Handys kopieren.
7. Android-Threema: ⋮ → Mein Profil → "Threema-ID löschen" (**nicht widerrufen**),
   dann "Backup wiederherstellen" → "Weitere Wiederherst.-Optionen" → "Daten-Backup"
   → Datei wählen → Android-Backup-Passwort.
8. Ab Schritt 1 Threema auf dem iPhone nicht mehr öffnen.

## Fallen
- Zwei verschiedene Passwörter: iPhone-Backup (Schritt 2) ≠ Android-Backup (Schritt 5/7).
- "Bad password for file 'identity'" = falsches Android-Passwort. Testen:
  `~/threema-venv/bin/python -c "import pyzipper,getpass; z=pyzipper.AESZipFile('$HOME/threema-android.zip'); z.setpassword(getpass.getpass().encode()); z.read('identity'); print('OK')"`
- Output-Verzeichnis muss leer sein.
- Einfügen im Terminal liefert `^[[200~…~`: `bind 'set enable-bracketed-paste on'`.
- Nicht übernommen: Systemmeldungen, Verteilerlisten, Reaktionen. Umfragen → Text.

## Aufräumen
`rm -rf ~/threema-ios ~/threema-ios_raw ~/threema-output ~/threema-test ~/threema-migriert.zip ~/iphone-backup`
(enthalten alle Chats unverschlüsselt)
