# 🚀 UGREEN NAS Docker Backup & Restore

[🇬🇧 English version](README.md) · [🇩🇪 Deutsche Version](README.DE.md)

![Docker Backup Pack](Screen/DockerBackupPackEN.png)

## 📦 Opis

Ten projekt udostępnia rozbudowany system tworzenia kopii zapasowych, odtwarzania i migracji projektów Docker działających na UGREEN NAS z systemem UGOS.

✔ Kopie zapasowe kontenerów i projektów Docker  
✔ Odtwarzanie z integracją z aplikacją Docker w UGOS  
✔ Migracja na inny UGREEN NAS  
✔ Obsługa samodzielnych kontenerów (standalone)  
✔ Opcjonalna zdalna kopia zapasowa przez SCP  
✔ Mapowanie ścieżek podczas migracji NAS  
✔ Powiadomienia e-mail  
✔ Obsługa zadań cron  
✔ Opcjonalne wyłączenie systemu po pomyślnym zakończeniu  
✔ Komunikaty uruchomieniowe w języku angielskim i niemieckim

---

## 📁 Struktura repozytorium

```text
UGREEN-NAS-Docker-Backup-Restore/
├── DockerBackup/
│   ├── backup-exclude-paths.txt
│   ├── dockersich.env.example
│   ├── path-remap.tsv
│   ├── ugreen-docker-backup.sh
│   └── ugreen-docker-restore.sh
│
├── Screen/
│   ├── DockerBackupPack.png
│   ├── DockerBackupPackEN.png
│   └── DockerBackupPack_1200.jpg
│
├── README.md
├── README.DE.md
├── README.pl.md
├── Changelog.txt
└── UGREEN_Docker_BR_DE_EN.pdf
```

---

## ⚙️ Instalacja

### 1. Utwórz folder współdzielony w UGOS

W aplikacji **Files** w UGOS utwórz folder współdzielony o nazwie **DockerBackup**.

Przykład:

```text
/volume2/DockerBackup
```

Uwagi:
- Pozostaw kosz wyłączony dla tego udziału.
- Upewnij się, że administratorzy mają prawa odczytu i zapisu.
- Udział może znajdować się również na innym woluminie, np. `/volume1` lub `/volume3`.

---

### 2. Skopiuj pliki

Skopiuj zawartość katalogu:

```text
DockerBackup/
```

do:

```text
/volume2/DockerBackup
```

---

### 3. Ustaw uprawnienia

```bash
cd /volume2/DockerBackup
cp dockersich.env.example dockersich.env
chmod +x ugreen-docker-backup.sh ugreen-docker-restore.sh
```

---

### 4. Dostosuj konfigurację

Plik konfiguracyjny:

```text
/volume2/DockerBackup/dockersich.env
```

Przy pierwszym teście warto sprawdzić co najmniej poniższe wartości:

```bash
LANGUAGE=en
HOST_LABEL="UGREEN NAS"
SOURCE_DIR=auto
BACKUP_DIR=/volume2/DockerBackup
SEND_MAIL=false
DRY_RUN=true
```

Ważne informacje:
- `SOURCE_DIR=auto` automatycznie wykrywa katalog projektów Docker.
- `DOCKER_ROOT_DIR=auto` automatycznie wykrywa katalog danych Dockera.
- `UGOS_DOCKER_DB=auto` automatycznie wykrywa bazę danych aplikacji Docker w UGOS.
- `DRY_RUN=true` jest zdecydowanie zalecane przy pierwszym teście odtwarzania.

---

## 🗂️ Najważniejsze opcje

### Ustawienia podstawowe i ścieżki

- `LANGUAGE` = język komunikatów, logów i wiadomości e-mail (`en` lub `de`)
- `HOST_LABEL` = nazwa NAS wyświetlana w logach i wiadomościach e-mail
- `SOURCE_DIR` = katalog projektów Docker, zwykle wykrywany automatycznie
- `BACKUP_DIR` = katalog docelowy dla archiwów, logów i plików tymczasowych
- `TEMP_DIR` = tymczasowy katalog roboczy
- `LOG_DIR` = katalog logów kopii zapasowej i odtwarzania

### Zachowanie kopii zapasowej

- `BACKUP_ALL_PROJECTS=true` = wykonaj kopię wszystkich wykrytych projektów Docker
- `INCLUDE_PROJECTS` = wykonaj kopię tylko wybranych projektów
- `EXCLUDE_PROJECTS` = wyklucz wskazane projekty z kopii zapasowej
- `BACKUP_STANDALONE_CONTAINERS=true` = uwzględnij również kontenery bez projektu Compose
- `STOP_CONTAINERS=true` = na krótko zatrzymaj działające kontenery przed wykonaniem kopii
- `BACKUP_EXCLUDE_PATHS_FILE=backup-exclude-paths.txt` = lista wykluczeń dla dużych katalogów cache

### Dodatkowa zawartość kopii zapasowej

- `BACKUP_IMAGES=false` = dodatkowo wykonaj kopię obrazów Docker
- `BACKUP_NAMED_VOLUMES=false` = dodatkowo wykonaj kopię named volumes
- `BACKUP_EXTERNAL_BINDS=false` = dodatkowo wykonaj kopię zewnętrznych bind mountów

### Zachowanie odtwarzania

- `DRY_RUN=true` = zasymuluj odtwarzanie bez wykonywania zmian
- `RESTORE_ALL_PROJECTS=false` = wykonaj selektywne odtwarzanie zamiast odtwarzać wszystko
- `RESTORE_PROJECTS` = odtwórz tylko wybrane projekty
- `RESTORE_OVERWRITE_EXISTING=false` = nie nadpisuj istniejących katalogów docelowych
- `RESTORE_STANDALONE_CONTAINERS=true` = odtwórz również samodzielne kontenery
- `ENABLE_PATH_REMAP=true` = mapuj ścieżki podczas migracji na inny NAS
- `PATH_REMAP_FILE=path-remap.tsv` = plik mapowania ścieżek źródłowych na docelowe
- `UPDATE_UGOS_DOCKER_DB=true` = zaktualizuj bazę danych aplikacji Docker w UGOS
- `REFRESH_UGOS_DOCKER_APP=true` = odśwież usługę aplikacji Docker po odtworzeniu

### Powiadomienia i zdalna kopia zapasowa

- `SEND_MAIL=true|false` = włącz lub wyłącz powiadomienia e-mail
- `MAIL_NOTIFY_ON=all|success|fail|none` = określ, kiedy mają być wysyłane powiadomienia
- `ENABLE_REMOTE_BACKUP=true|false` = dodatkowo skopiuj archiwum na inny system przez SCP
- `REMOTE_HOST`, `REMOTE_USER`, `REMOTE_PORT`, `REMOTE_PATH` = parametry docelowego systemu kopii zdalnej

---

## 🔄 Uruchomienie kopii zapasowej

```bash
cd /volume2/DockerBackup
./ugreen-docker-backup.sh
```

Skrypt:
- automatycznie wykrywa ścieżki Dockera,
- wybiera projekty zgodnie z konfiguracją,
- opcjonalnie na krótko zatrzymuje działające kontenery,
- tworzy skompresowane archiwum,
- ponownie uruchamia kontenery, które działały przed rozpoczęciem kopii,
- opcjonalnie wysyła wiadomości e-mail ze statusem.

---

## ♻️ Uruchomienie odtwarzania

Najpierw zawsze przetestuj odtwarzanie w trybie dry-run:

```bash
cd /volume2/DockerBackup
./ugreen-docker-restore.sh /volume2/DockerBackup/ugreen-docker-backup_YYYY-MM-DD_HH-MM-SS.tar.gz
```

Przy pierwszym teście ustaw w `dockersich.env`:

```bash
DRY_RUN=true
```

Dla właściwego odtwarzania ustaw później:

```bash
DRY_RUN=false
```

Ze względów bezpieczeństwa skrypt poprosi wtedy o wpisanie:

```text
RESTORE
```

---

## 🔁 Migracja kontenerów na inny UGREEN NAS

Pakiet może być również używany do migracji projektów Docker na inny UGREEN NAS.

Typowy przebieg:
1. Utwórz kopię zapasową na źródłowym NAS.
2. Skopiuj archiwum na docelowy NAS.
3. W razie potrzeby dostosuj `path-remap.tsv`.
4. Uruchom odtwarzanie na docelowym NAS.
5. Pozwól skryptowi automatycznie zsynchronizować odtworzone projekty z aplikacją Docker w UGOS.

Pozwala to w uporządkowany sposób przenieść katalogi projektów, projekty Compose oraz opcjonalnie dodatkowe dane Dockera na system docelowy.

---

## 🧩 Samodzielne kontenery

Kontenery bez etykiety projektu Docker Compose mogą być automatycznie zapisywane jako osobne projekty.

Przykład:

```text
ubuntu-1 -> standalone_ubuntu-1
```

Podczas odtwarzania dla takiego kontenera generowany jest projekt Compose, który następnie zostaje zintegrowany z aplikacją Docker w UGOS.

---

## ⏱️ Konfiguracja zadania cron

```bash
crontab -e
```

Przykład:

```bash
30 3 * * 0 cd /volume2/DockerBackup && /volume2/DockerBackup/ugreen-docker-backup.sh >> /volume2/DockerBackup/cron.log 2>&1
```

➡️ Uruchamianie w każdą niedzielę o 03:30.

---

## 📦 Funkcje

- Kopia wszystkich lub wybranych projektów Docker
- Odtwarzanie jednego lub wielu projektów
- Migracja na inny UGREEN NAS
- Automatyczne wykrywanie ścieżek Dockera
- Odtwarzanie z synchronizacją z aplikacją Docker w UGOS
- Obsługa samodzielnych kontenerów
- Wykluczanie dużych katalogów cache
- Opcjonalna kopia obrazów, named volumes i zewnętrznych bind mountów
- Zdalna kopia zapasowa przez SCP
- Mapowanie ścieżek dla różnych lokalizacji docelowych
- Powiadomienia e-mail przy rozpoczęciu, powodzeniu i błędzie
- Logowanie i praca z cron

---

## 📘 Instrukcja

W repozytorium znajduje się:

```text
UGREEN_Docker_BR_DE_EN.pdf
```

Instrukcja zawiera krok po kroku informacje dotyczące instalacji, konfiguracji, tworzenia kopii zapasowych, odtwarzania i migracji w języku niemieckim i angielskim.

---

## 🛠️ Rozwiązywanie problemów

| Problem | Rozwiązanie |
|---|---|
| Skrypt nie uruchamia się | Sprawdź uprawnienia `chmod +x` |
| Wiadomość e-mail nie jest wysyłana | Sprawdź ustawienia SMTP i `MAIL_NOTIFY_ON` |
| Nie wybrano żadnych projektów | Sprawdź `BACKUP_ALL_PROJECTS`, `INCLUDE_PROJECTS` i `EXCLUDE_PROJECTS` |
| Archiwum jest bardzo duże | Sprawdź `backup-exclude-paths.txt` i wyklucz katalogi cache |
| Odtworzony projekt nie pojawia się w UGOS | Użyj `REFRESH_UGOS_DOCKER_APP=true` lub ponownie wdróż projekt w UGOS |
| `scp` kończy się błędem | Przy testach ręcznych użyj `scp -O` |

---

## ⚠️ Zastrzeżenie

Projekt jest rozwiązaniem społecznościowym i nie jest oficjalnym produktem UGREEN.
Używasz go na własne ryzyko.

---

## 👨‍💻 Autor

Roman Glos  
UGREEN NAS Community

---

## ⭐ Wsparcie

Jeśli ten projekt jest dla Ciebie przydatny:

- Zostaw ⭐ na GitHubie.
- Opinie i uwagi są mile widziane.
- Sugestie ulepszeń i testy w rzeczywistych środowiskach pomagają rozwijać projekt.
