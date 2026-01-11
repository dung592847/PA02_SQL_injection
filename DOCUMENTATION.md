# Projekt: SQL-Injection — Dokumentation und Anleitung

Diese Dokumentation beschreibt das Projektziel, die Systemvoraussetzungen, die Einrichtung der Testumgebung, die Durchführung zweier Angriffe (extern: Login, intern: User-Profile / SQL-Injection), Nachweis/Belege (Screenshots, Logs, PCAP-Hinweise), Gegenmaßnahmen und eine Beispiel-ROSI-Berechnung. Die Anleitung ist so ausgelegt, dass Dritte das Projekt nachvollziehen und reproduzieren können.

---

## 1. Kurzbeschreibung, Zielsetzung und erwartete Ergebnisse
- **Kurzbeschreibung:** Aufbau einer kleinen Webanwendung (Frontend Vue + Backend Spring Boot), Demonstration von SQL-Injection-Techniken (externes vs. internes Szenario) und Implementierung von Gegenmaßnahmen. 
- **Zielsetzung:** Zeigen, wie SQL-Injection ausgenutzt werden kann, wie man Angriffsszenarien nachstellt, wie man Gegenmaßnahmen implementiert und wie man wirtschaftlich (ROSI) bewertet.
- **Erwartete Ergebnisse:** Reproduzierbare Angriffe (oder Nachweis, falls die Anwendung bereits sicher ist), Wirksamkeitsnachweis der Gegenmaßnahmen, ROSI-Berechnung und ein A0-Poster mit den wichtigsten Ergebnissen.

---

## 2. Systemvoraussetzungen
Diese Anleitung wurde unter Windows entwickelt, funktioniert aber auch unter Linux/macOS mit äquivalenten Befehlen.

- Hardware: mind. 4 GB RAM, 2 CPU-Kerne, 5 GB freier Festplattenspeicher.
- Software (mindestens):
  - Java JDK 17 (vgl. `backend/pom.xml` -> `<java.version>17</java.version>`)
  - Maven (3.x)
  - Node.js 18+ und `npm` (für das Frontend)
  - MySQL 8 (lokal) ODER Docker (wenn Sie MySQL in einem Container ausführen wollen)
  - Optional: `sqlmap` (für automatisierte SQLi-Tests), `Burp Suite` oder `OWASP ZAP` (Proxy, Man-in-the-Middle), `Wireshark` (PCAP-Analyse)

Wie herunterladen / installieren:

1. Java: https://adoptium.net/ (Installieren JDK 17)
2. Maven: https://maven.apache.org/download.cgi
3. Node.js: https://nodejs.org/
4. MySQL: https://dev.mysql.com/downloads/mysql/ oder Docker:

```powershell
docker run --name demo-mysql -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=demodb -e MYSQL_USER=demo -e MYSQL_PASSWORD=password -p 3306:3306 -d mysql:8.0
```

---

## 3. Repository herunterladen

Klonen oder lokal kopieren (falls Sie das Zip erhalten haben):

```powershell
git clone <REPO_URL>
cd PA02_SQL_injection
```

---

## 4. Testumgebung / Architektur (Kurz)

- Frontend: Vite + Vue, Dev-Server läuft standardmäßig auf `http://localhost:5173`.
- Backend: Spring Boot, konfiguriert in `backend/src/main/resources/application.properties`, Standard-Port `8080`.
- Datenbank: MySQL, DB-Name `demodb`, Benutzer `demo`, Passwort `password` (siehe `application.properties`).

Netzwerk / IP-Plan (lokal):
- `127.0.0.1:5173` -> Frontend (Dev)
- `127.0.0.1:8080` -> Backend API
- `127.0.0.1:3306` -> MySQL

Wenn Sie mehrere VMs / Container nutzen, passen Sie die IP/Port-Zuordnung entsprechend an.

---

## 5. Setup: Backend starten

1. MySQL starten (siehe Abschnitt 2). Stellen Sie sicher, dass die DB `demodb` erreichbar ist und die Zugangsdaten mit `application.properties` übereinstimmen.

2. Backend bauen und starten:

```powershell
cd backend
mvn clean package
mvn spring-boot:run
# Oder nach dem Package:
java -jar target/demo-0.0.1-SNAPSHOT.jar
```

Hinweis: Das Backend verwendet Java 17 und erwartet eine MySQL-Verbindung auf `jdbc:mysql://localhost:3306/demodb` mit Benutzer `demo` / `password`.

Beim ersten Start wird die `DataLoader`-Komponente automatisch einige Testuser anlegen (Alice, Bob, Charlie, Admin).

---

## 6. Setup: Frontend starten

```powershell
cd frontend
npm install
npm run dev
```

Das Frontend (Dev-Server Vite) läuft danach unter `http://localhost:5173` und kommuniziert mit dem Backend unter `http://localhost:8080/api`.

---

## 7. Angriffsszenarien (Schritt-für-Schritt)

Hinweis: Führen Sie Angriffe nur in Ihrer Testumgebung durch. Die nachfolgenden Schritte zeigen manuelle und automatisierte Tests.

7.1 Externer Angriff: Login-Seite (Ziel: SQL-Injection)

- Ziel-URL: `POST http://localhost:8080/api/login` mit JSON-Body `{ "username": "...", "password": "..." }`.
- Manueller Test:
  1. Öffnen Sie die Login-Seite im Frontend und geben bekannte Daten ein (z. B. username `alice`, password `password123`) — prüfen, ob Login erfolgreich ist.
  2. Versuchen Sie typische SQLi-Payloads im Benutzernamen-Feld, z. B.:

```json
{ "username": "' OR '1'='1", "password": "irrelevant" }
```

  3. Beobachten Sie die Antwort des Backends. Falls die Anwendung auf rohe String-Konkatenation basiert, könnte die Authentifizierung umgehen werden.
  4. Screenshot-Platzhalter: An dieser Stelle bitte einen Screenshot einfügen, z. B. `screenshots/login_sqli_attempt.png`.

- Automatisierter Test mit `sqlmap` (Beispiel):

```powershell
sqlmap -u "http://localhost:8080/api/login" --data="username=alice&password=password123" -p username --json
```

  (Passen Sie `--data` entsprechend an, oder verwenden Sie Burp, um einen Request anzulegen und sqlmap mit `-r request.txt` zu füttern.)

7.2 Interner Angriff: User-Profile / SQLi im Profil-Kontext

- Ziel: Endpunkte, die Benutzereingaben in DB-Abfragen verwenden (z. B. Suche, Profil-Update, ID-Parameter). In diesem Projekt existiert ein allgemeiner `GET /api/users` und `POST /api/users`.
- Mögliche Szenarien:
  - Falls es einen Endpunkt gäbe, der SQL-Strings dynamisch zusammensetzt (z. B. `findByUsername` in einer unsicheren Implementierung), könnte ein Payload wie `admin' -- ` oder `admin' OR '1'='1` funktionieren.
  - Testen Sie `GET /api/users` und prüfen Sie, ob Sie über Query-Parameter einen Parameter injizieren können (falls das Frontend sowas anbietet).

- Beispiel: POST /api/users (Neuen Benutzer anlegen) — testen, ob nach dem Speichern gefährliche Eingaben zu späteren SQL-Fehlern führen:

```json
{ "name": "Evil", "email":"e@x.com", "username":"injection' OR '1'='1", "password":"pass" }
```

  - Screenshot-Platzhalter: `screenshots/profile_sqli_payload.png`

7.3 Beweismittel sammeln
- Logs: Backend-Logs (Konsole oder `log.txt`), MySQL-Logs.
- PCAPs: Starten Sie `Wireshark` oder `tcpdump` und nehmen Sie den Verkehr während eines Angriffs auf.
- Wenn Sie Burp verwenden, speichern Sie das Projekt und exportieren die relevanten Requests.

Hinweis: In dieser konkreten Anwendung werden standardmäßig Spring Data JPA und `findByUsername(...)` verwendet, die nicht per se SQLi-anfällig sind, da JPA Parameterbindung benutzt. Wenn ein test keine Schwachstelle findet, dokumentieren Sie den fehlgeschlagenen Exploitversuch (Beweis, dass die Anwendung sicher gegenüber diesem Vektor ist).

---

## 8. Gegenmaßnahmen (Implementation und Nachweis)

8.1 Sofortmaßnahmen
- Parameterisierte Abfragen / Prepared Statements.
- Benutzen Sie ORM (Spring Data JPA) korrekt — niemals Roh-SQL mit String-Konkatenation.
- Input-Validierung (Whitelist-Ansatz), Length-Limits.

8.2 Netzwerk / Infrastruktur
- Minimal-privilegierter DB-Benutzer (kein DBA-Recht), getrennte Konten für Applikation/DB-Admin.
- Härtung: WAF-Regeln, Web Application Firewall konfigurieren.

8.3 Erkennung
- IDS/IPS (z. B. Snort/Suricata) mit Regeln für typische SQLi-Payloads.
- Logging und Alerting (z. B. erfolgreiche Login-Versuche mit ungewöhnlichen Patterns).

8.4 Nachweis der Wirksamkeit
- Beispiel-Test: Deploy mit Prepared Statements + WAF, wiederhole SQLi-Payloads und dokumentiere, dass die Requests blockiert oder nicht erfolgreich sind.
- Screenshot-Platzhalter: `screenshots/mitigation_blocked.png`.

---

## 9. ROSI (Return on Security Investment) — Beispielrechnung

Formel: ROSI = (RiskReduction × AssetValue − Cost) / Cost

Beispielannahmen (vereinfachtes Beispiel):
- AssetValue (kritische Daten/Verlust pro Jahr ohne Schutz): 200.000 €
- Aktuelles erwartetes jährliches Verlust-Risiko (EAL) ohne Maßnahme: 10% → 20.000 €
- Nach Maßnahmen (WAF + Prepared Statements + Monitoring) geschätzte EAL: 1% → 2.000 €
- RiskReduction = (20.000 − 2.000) / 20.000 = 0,9 (90% Reduktion)
- Cost (One-time + jährliche Betriebskosten): WAF/Monitoring + Entwicklerzeit = 15.000 € (einmalig/erstes Jahr)

ROSI = ((0.9 × 200.000) − 15.000) / 15.000 = ((180.000) − 15.000) / 15.000 = 165.000 / 15.000 = 11 → 1100%

Interpretation: Ein ROSI > 1 bedeutet, dass sich die Investition rechnet. Dokumentieren Sie Annahmen sauber und variieren Sie Parameter (Sensitivitätsanalyse).

---


## 10. Dateien/Orte im Repo, die relevant sind
- Backend-Konfiguration: [backend/src/main/resources/application.properties](backend/src/main/resources/application.properties)
- Datenlade-Bean: [backend/src/main/java/com/example/demo/config/DataLoader.java](backend/src/main/java/com/example/demo/config/DataLoader.java)
- Auth-Endpoint: [backend/src/main/java/com/example/demo/controller/AuthController.java](backend/src/main/java/com/example/demo/controller/AuthController.java)
- User-API: [backend/src/main/java/com/example/demo/controller/UserController.java](backend/src/main/java/com/example/demo/controller/UserController.java)

---

