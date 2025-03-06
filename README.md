# selinux-projekt
SELinux-Projekt: Zugriffskontrolle auf Dateien und Netzwerkdienste
1. SELinux verhindern, dass ein Benutzer auf eine Datei zugreift
Schritt 1: Testdatei erstellen

sudo touch /testdatei
sudo chown root:root /testdatei

Erstellt eine Datei namens testdatei und setzt den Besitzer auf root, damit nur root standardmäßig darauf zugreifen kann.
Schritt 2: SELinux-Kontext setzen

sudo semanage fcontext -a -t user_home_t /testdatei
sudo restorecon -v /testdatei

Setzt den SELinux-Kontext für die Datei, um den Zugriff weiter zu kontrollieren.
Schritt 3: Benutzerrolle zuweisen (testuser)

sudo semanage login -a -s user_u testuser

Weist dem Benutzer testuser die restriktive SELinux-Rolle user_u zu, die ihm den Zugriff auf die Datei verbietet.
Schritt 4: Zugriff testen

Melde dich als testuser an und versuche, auf die Datei zuzugreifen:

su - testuser
cat /testdatei  # Sollte verweigert werden

Der Zugriff auf die Datei wird verweigert, da testuser mit der Rolle user_u keine Berechtigung hat.
2. Nur ein bestimmter Benutzer darf auf einen Netzwerkdienst zugreifen (z.B. SSH)
Schritt 1: SSH-Berechtigungen festlegen

sudo setsebool -P ssh_sysadm_login on

Dieser Befehl aktiviert eine SELinux-Option, die es bestimmten Benutzern (wie testuser) erlaubt, sich mit Administratorrechten (sysadmin) über SSH anzumelden.
Schritt 2: Benutzerrolle zuweisen

sudo semanage login -a -s staff_u testuser

Weist testuser die Rolle staff_u zu, die erweiterte Rechte für Netzwerkdienste wie SSH gewährt.
Schritt 3: Zugriff für andere Benutzer verbieten

sudo semanage login -d -s unconfined_u

Entfernt die unbeschränkte Rolle unconfined_u für alle anderen Benutzer, was deren Zugriff auf Netzwerkdienste wie SSH blockiert.
3. Eine bestimmte Gruppe vom Zugriff auf einen Netzwerkdienst ausschließen (z.B. HTTP)
Schritt 1: Gruppe erstellen und Benutzer hinzufügen

sudo groupadd restricted_users
sudo usermod -aG restricted_users testuser

Erstellt eine neue Gruppe restricted_users und fügt testuser dieser Gruppe hinzu.
Schritt 2: Zugriff auf HTTP blockieren

sudo setsebool -P httpd_enable_homedirs off

Deaktiviert den Zugriff auf Home-Verzeichnisse über den HTTP-Server für Benutzer, die der restricted_users-Gruppe angehören.
4. Änderungen zurücksetzen

Falls du die Änderungen rückgängig machen möchtest, um zu einem vorherigen Zustand zurückzukehren, führst du folgende Befehle aus:
Rückgängig machen der Änderungen

sudo semanage login -d testuser
sudo semanage fcontext -d "/testdatei"
restorecon -v /testdatei
sudo setsebool -P ssh_sysadm_login off
sudo setsebool -P httpd_enable_homedirs on

    sudo semanage login -d testuser: Entfernt die SELinux-Rolle für testuser und entzieht ihm die speziellen Zugriffsrechte auf Netzwerkdienste wie SSH.
    sudo semanage fcontext -d "/testdatei": Entfernt den SELinux-Kontext für die Datei /testdatei und stellt die Standardregeln wieder her.
    restorecon -v /testdatei: Wendet die Standard-SELinux-Richtlinien auf die Datei an, um sie in den ursprünglichen Zustand zurückzusetzen.
    sudo setsebool -P ssh_sysadm_login off: Deaktiviert die SSH-Berechtigungs-Option und entzieht testuser die administrativen Rechte über SSH.
    sudo setsebool -P httpd_enable_homedirs on: Aktiviert wieder den Zugriff auf Home-Verzeichnisse über den HTTP-Server.

Durch die Anwendung dieser SELinux-Befehle kannst du den Zugriff auf Netzwerkdienste und Dateien für bestimmte Benutzer und Gruppen steuern und sicherstellen, dass nur autorisierte Benutzer bestimmte Aufgaben ausführen können.
