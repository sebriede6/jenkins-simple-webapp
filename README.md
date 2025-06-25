
---

```markdown
# Jenkins Simple WebApp CI

Dieses Repository dient als Basis für die Aufgabe "Erste Jenkins Pipeline". Es enthält eine einfache Vite/React-Anwendung und eine `Jenkinsfile`, um eine grundlegende CI-Pipeline in Jenkins aufzusetzen und auszuführen.

## Aufgabe

Ziel der Aufgabe ist es, eine lokale Jenkins-Instanz mittels Docker aufzusetzen, den Jenkins-Workflow zu verstehen und eine einfache Pipeline zu erstellen, die den Code dieser Webanwendung aus einem Git-Repository klont und einen Build-Prozess startet.

## Webanwendung

Die Anwendung ist eine Standard-Vite-Anwendung mit React.

-   **Installation der Abhängigkeiten:** `npm ci`
-   **Build für die Produktion:** `npm run build`

## Jenkins-Pipeline Konfiguration und Ausführung

Der folgende Prozess beschreibt die finalen, funktionierenden Schritte, um eine robuste Jenkins-Umgebung für Docker-basierte Pipelines aufzusetzen.

### Schritt 1: Jenkins Docker-Image vorbereiten

Das offizielle Jenkins-LTS-Image (`jenkins/jenkins:lts-jdk17`) enthält standardmäßig nicht den Docker-Client. Um Pipeline-Schritte innerhalb von Docker-Containern auszuführen (`agent { docker ... }`), muss der Docker-Client im Jenkins-Container verfügbar sein.

Dazu wird ein benutzerdefiniertes Image mit einer `Dockerfile` erstellt.

**`Dockerfile`**

```dockerfile
FROM jenkins/jenkins:lts-jdk17

USER root
RUN apt-get update && apt-get install -y docker.io
USER jenkins
```

### Schritt 2: Benutzerdefiniertes Jenkins-Image bauen

Dieser Befehl liest die `Dockerfile` im aktuellen Verzeichnis und erstellt ein neues lokales Image mit dem Namen `my-jenkins-with-docker`.

```bash
docker build -t my-jenkins-with-docker .
```

### Schritt 3: Jenkins-Container starten

Der folgende Befehl startet den Jenkins-Container mit dem neu erstellten Image. Er bindet den Docker-Socket des Host-Systems ein, um die Kommunikation mit dem Docker-Daemon zu ermöglichen, und verwendet ein benanntes Volume für die Persistenz der Daten.

```bash
docker run \
  -d \
  --name my-simple-jenkins \
  -p 8080:8080 \
  -u root \
  -v jenkins_home:/var/jenkins_home \
  -v //var/run/docker.sock:/var/run/docker.sock \
  my-jenkins-with-docker
```

### Schritt 4: Jenkins-Setup abschließen

Nach dem Start wird Jenkins unter `http://localhost:8080` initial konfiguriert (Unlock mit Passwort aus `docker logs my-simple-jenkins`, Installation der empfohlenen Plugins, Erstellung eines Admin-Users).



### Schritt 5: Pipeline-Job Konfiguration

Ein neuer "Pipeline"-Job wird erstellt. Die entscheidende Konfiguration ist "Pipeline script from SCM", die auf dieses Repository verweist.



### Schritt 6: Jenkinsfile

Die `Jenkinsfile` im Root dieses Repositories definiert die Pipeline-Struktur. Sie verwendet einen Docker-Agenten, um den Build in einer sauberen Node.js-Umgebung auszuführen.



### Schritt 7: Erfolgreicher Pipeline-Lauf

Nach dem Klick auf "Build Now" führt Jenkins die in der `Jenkinsfile` definierten Stages aus. Der Build ist erfolgreich und alle Stufen sind grün.



---

## Reflexion

### Welche Schritte waren notwendig, um Jenkins lokal mit Docker zum Laufen zu bringen?
Um Jenkins lokal mit Docker für diese Aufgabe zum Laufen zu bringen, waren folgende Schritte notwendig:
1.  **Erstellung einer `Dockerfile`**: Da das Standard-Jenkins-Image den Docker-Client nicht enthält, wurde eine `Dockerfile` erstellt, die auf `jenkins/jenkins:lts-jdk17` basiert und den Docker-Client (`docker.io`) nachinstalliert.
2.  **Bau eines benutzerdefinierten Docker-Images**: Mit dem Befehl `docker build -t my-jenkins-with-docker .` wurde aus der `Dockerfile` ein neues, lokales Image gebaut.
3.  **Start des Jenkins-Containers**: Mit `docker run` wurde ein Container aus diesem neuen Image gestartet. Dabei wurde der Port `8080` gemappt, ein benanntes Volume (`jenkins_home`) für die Persistenz der Daten eingebunden und der Docker-Socket des Host-Systems (`//var/run/docker.sock`) für die Kommunikation mit dem Docker-Daemon durchgereicht.
4.  **Abschluss der Installation**: Über die Web-Oberfläche wurde Jenkins mit dem initialen Passwort entsperrt, die empfohlenen Plugins wurden installiert und ein Admin-Benutzer wurde angelegt.

### Was ist der Zweck der Datei Jenkinsfile, und wo muss sie im Verhältnis zu deinem Anwendungscode liegen?
Die `Jenkinsfile` dient dazu, den gesamten CI/CD-Prozess als Code zu definieren. Dieses Konzept nennt sich "Pipeline as Code". Dadurch wird die Pipeline versionierbar, reproduzierbar und kann gemeinsam mit der Anwendung weiterentwickelt werden. Die `Jenkinsfile` muss im Wurzelverzeichnis (Root) des Projektrepositories liegen, damit Jenkins sie bei der Konfiguration "Pipeline script from SCM" standardmäßig finden und ausführen kann.

### Beschreibe die zwei Hauptstages, die du in deiner Pipeline definiert hast, und was der Hauptzweck der Steps in jeder Stage ist.
Die Pipeline wurde in einem `agent { docker { image 'node:20-alpine' } }`-Block ausgeführt, was bedeutet, dass alle Stages in einem sauberen Docker-Container liefen. Die zwei Hauptstufen waren:

1.  **Checkout**: Der Hauptzweck dieser Stage ist das Herunterladen des Quellcodes aus dem konfigurierten Git-Repository in den Build-Workspace des Containers. Der Step `checkout scm` ist ein Jenkins-eigener Schritt, der die in der Job-Konfiguration hinterlegten SCM-Informationen (Repository-URL, Branch) nutzt.

2.  **Build Application**: Der Hauptzweck dieser Stage ist die Kompilierung und das Bauen der Anwendung. Die `sh`-Steps führen die dazu notwendigen Shell-Befehle aus: `npm ci` installiert die exakten Projekt-Abhängigkeiten auf Basis der `package-lock.json`, und `npm run build` startet das Build-Skript, welches die React-Anwendung in statische Assets für die Produktion umwandelt.

### Wie hast du in Jenkins konfiguriert, von welchem Git-Repository und welchem Branch der Code für die Pipeline geholt werden soll?
In der Konfiguration des Jenkins-Pipeline-Jobs wurde im Abschnitt "Pipeline" die Option **"Pipeline script from SCM"** ausgewählt. Darunter wurden folgende Einstellungen vorgenommen:
-   **SCM**: "Git" wurde als System ausgewählt.
-   **Repository URL**: Hier wurde die HTTPS-URL des GitHub-Repositories eingetragen.
-   **Branches to build**: Hier wurde der Branch von `*/master` auf `*/main` geändert, um den korrekten Hauptbranch zu verwenden.

### Was ist der Unterschied zwischen dem checkout scm Step in deiner Pipeline und dem git clone Befehl, den du manuell im Terminal ausführen würdest?
Der `checkout scm` Step ist eine Abstraktion von Jenkins, während `git clone` ein reiner Git-Befehl ist. Die wesentlichen Unterschiede sind:

-   **Integration**: `checkout scm` ist vollständig in Jenkins integriert. Er greift automatisch auf die SCM-Konfiguration des Jobs zu, einschließlich Repository-URL, Branch und gespeicherter Credentials. Bei `git clone` müssten diese Informationen manuell und unsicher im Skript hinterlegt werden.
-   **Workspace Management**: `checkout scm` kümmert sich um den Jenkins-Workspace. Es stellt sicher, dass der Workspace vor dem Checkout sauber ist und holt bei nachfolgenden Builds oft nur die neuesten Änderungen, was effizienter ist.
-   **Funktionalität**: Der Jenkins-Step bietet erweiterte Funktionen (z.B. für Submodule, Timeouts, Shallow Clones), die über einfache Parameter gesteuert werden können, ohne komplexe Shell-Skripte schreiben zu müssen. `checkout scm` ist somit der deklarative, sichere und wartbare Weg, Code in einer Jenkins-Pipeline zu beziehen.

```