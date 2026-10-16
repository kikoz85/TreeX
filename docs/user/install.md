# Installazione

> macOS 13 Ventura o successivo (Apple Silicon e Intel, binario universale) · Windows 10/11 x64 · Linux x86_64 (glibc). I file della release hanno la versione nel nome (`<ver>`).

- [macOS](#macos) · [Windows](#windows) · [Linux](#linux)

## macOS

### Dal DMG

1. Apri `TreeX-<ver>-macos-universal.dmg`.
2. Trascina **TreeX** sulla cartella **Applicazioni**.
3. Avvia TreeX da Applicazioni o da Spotlight.

### Primo avvio

Le build distribuite senza firma Developer ID (firma *ad-hoc*) non sono notarizzate da Apple. Al primo avvio Gatekeeper mostra un avviso:

- **clic destro su TreeX.app → Apri**, poi conferma **Apri**; oppure
- **Impostazioni di Sistema → Privacy e sicurezza → "Apri comunque"**.

Serve una volta sola.

Per analizzare l'intero disco serve l'**Accesso completo al disco**: TreeX lo segnala da solo quando trova cartelle che macOS non gli lascia leggere. Vedi [Permessi su macOS](macos-permissions.md).

### La CLI `treex`

La CLI è inclusa nell'app (`TreeX.app/Contents/Helpers/treex`). Per usarla da qualsiasi terminale, nell'app: **⌘K → Installa il comando treex nel terminale**. Collega `/usr/local/bin/treex` (e le pagine `man` e i completamenti) alla copia dentro l'app, che si aggiorna con l'app; macOS chiede la password se `/usr/local` non è scrivibile.

Solo la riga di comando, senza l'app: `treex-cli-<ver>-macos-universal.tar.gz`, poi `./install.sh` (in `/usr/local`; `--user` per `~/.local`). Dettagli in [Installare `treex` come comando](cli.md#installare-treex-come-comando).

### Homebrew

La release genera il cask `treex.rb` (app + link `treex` nel `PATH`):

```bash
brew install --cask ./treex.rb
```

### Verificare il download

```bash
shasum -a 256 -c SHA256SUMS
```

### Disinstallare

Trascina TreeX.app nel Cestino. Dati creati da TreeX (tutti facoltativi):

| Percorso | Contenuto |
|---|---|
| `~/Library/Caches/dev.treex.TreeX/` | snapshot delle analisi (si rigenerano) |
| `~/Library/Application Support/dev.treex.TreeX/` | storico delle pulizie |
| `~/Library/Application Support/TreeX/` | preferenze dell'app (lingua, vista) |
| `~/.config/treex/` | regole Dev-Aware personalizzate, se create |

## Windows

**Con l'installer** (consigliato): scarica `TreeX-<ver>-windows-x64-setup.exe` e avvialo. Non servono permessi di amministratore: TreeX si installa per il tuo utente in `%LOCALAPPDATA%\Programs\TreeX`, con la voce nel menu Start, l'icona sul desktop se la scegli e, se lasci spuntata l'opzione, il comando `treex` nel PATH (da usare in qualsiasi terminale: `treex tui`). Compare in **Impostazioni → App → App installate**, da dove si disinstalla. L'installer parla italiano o inglese secondo la lingua di Windows; avviare la versione nuova sopra quella installata la aggiorna, mantenendo le scelte fatte.

**Portatile, con lo zip**: scarica `TreeX-<ver>-windows-x64.zip`, estrailo dove preferisci (per esempio nella tua cartella utente o su una chiavetta) e avvia **TreeX.exe**. Per averlo nel menu Start: tasto destro su `TreeX.exe` → *Aggiungi a Start*.

La prima volta Windows SmartScreen può mostrare "Windows ha protetto il PC" (l'eseguibile non è firmato con un certificato di firma del codice): **Ulteriori informazioni → Esegui comunque**. Serve una volta sola.

**Amministratore (facoltativo)**: avviato come amministratore (tasto destro → *Esegui come amministratore*, oppure *Riavvia come amministratore* dal messaggio sulle cartelle protette) TreeX legge le cartelle di sistema protette, analizza un disco intero leggendo direttamente la Master File Table e dalla seconda volta lo aggiorna dal journal USN in una frazione di secondo.

**CLI**: `treex.exe` è nella cartella `bin` dell'archivio. Per usarla da qualsiasi terminale, nell'app: **Ctrl+K → Installa il comando treex nel terminale** (aggiunge quella cartella al PATH dell'utente; apri poi un nuovo terminale). Solo la riga di comando, senza l'app: `treex-cli-<ver>-windows-x64.zip`, poi doppio clic su `install.cmd` (nessun permesso di amministratore). Dettagli in [Installare `treex` come comando](cli.md#installare-treex-come-comando).

**Verificare il download** (PowerShell):

```powershell
Get-FileHash -Algorithm SHA256 TreeX-<ver>-windows-x64-setup.exe
```

e confronta con `SHA256SUMS-windows`.

**Disinstallare**: installato con l'installer, da **Impostazioni → App → App installate → TreeX → Disinstalla** (toglie anche la voce del PATH; alla fine chiede se eliminare anche analisi, storico e impostazioni). Con lo zip, elimina la cartella di TreeX. Dati creati da TreeX (tutti facoltativi):

| Percorso | Contenuto |
|---|---|
| `%LOCALAPPDATA%\TreeX\cache\` | snapshot delle analisi (si rigenerano) |
| `%LOCALAPPDATA%\TreeX\` | storico delle pulizie |
| `%APPDATA%\TreeX\data\` | preferenze dell'app (lingua, vista) |
| `%APPDATA%\TreeX\` | regole Dev-Aware personalizzate, se create |

## Linux

Tre formati, stessa app:

| Formato | Per chi | Note |
|---|---|---|
| **AppImage** (`TreeX-<ver>-x86_64.AppImage`) | chi vuole un file unico, senza installare nulla | gira sulla maggior parte delle distribuzioni (glibc 2.28 o successiva); vede tutto il sistema |
| **Flatpak** (`TreeX-<ver>-x86_64.flatpak`) | chi usa Flatpak (Fedora Silverblue, Linux Mint, elementary…) | aggiornamenti e runtime gestiti da Flatpak; vedi le note sulla sandbox qui sotto |
| **Archivio** (`treex-<ver>-linux-x86_64.tar.gz`) | chi vuole `treex` e l'app nella propria home | `install.sh` senza root |

Solo la **riga di comando** (server, macchine senza interfaccia grafica): il pacchetto `treex_<ver>_amd64.deb` per Debian e Ubuntu (`sudo apt install ./treex_<ver>_amd64.deb`), il pacchetto `treex-<ver>-1.x86_64.rpm` per Fedora, RHEL e openSUSE (`sudo dnf install ./treex-<ver>-1.x86_64.rpm`) oppure l'archivio `treex-cli-<ver>-linux-x86_64.tar.gz` con `./install.sh` per qualsiasi distribuzione; poi `treex tui` e `man treex`. Dettagli in [Installare `treex` come comando](cli.md#installare-treex-come-comando).

### AppImage

```bash
chmod +x TreeX-<ver>-x86_64.AppImage
```

```bash
./TreeX-<ver>-x86_64.AppImage
```

La CLI è nello stesso file: `./TreeX-<ver>-x86_64.AppImage --cli scan ~`, oppure crea un collegamento chiamato `treex` all'AppImage (`ln -s …/TreeX-<ver>-x86_64.AppImage ~/.local/bin/treex`). Se il sistema non ha FUSE 2 (`libfuse2`), aggiungi `--appimage-extract-and-run` o installa il pacchetto `libfuse2`/`fuse2`.

### Flatpak

```bash
flatpak install --user TreeX-<ver>-x86_64.flatpak
```

```bash
flatpak run dev.treex.TreeX
```

Il runtime (`org.freedesktop.Platform` 26.08) si scarica da Flathub se manca. La CLI: `flatpak run --command=treex dev.treex.TreeX scan ~`.

**Sandbox**: TreeX chiede l'accesso a tutti i file (`--filesystem=host`, serve a un analizzatore di spazio per leggere e, su richiesta, eliminare) e in sola lettura alle cartelle del sistema. Dentro la sandbox `/usr`, `/etc` e `/lib` sono quelli del runtime: le cartelle del sistema si analizzano in `/run/host/usr` (la tessera *Programmi* va già lì). Il Cestino è quello vero del desktop. Gli snapshot delle analisi stanno in `~/.var/app/dev.treex.TreeX/cache/treex/`. Per analizzare l'intero sistema dalla radice usa l'AppImage o l'archivio.

### Archivio

1. Scarica `treex-<ver>-linux-x86_64.tar.gz` ed estrailo.
2. Dalla cartella estratta esegui `./install.sh`: copia `treex` e `treex-gui` in `~/.local/bin`, aggiunge **TreeX** tra le applicazioni del desktop (icona compresa) e installa le pagine `man` (`man treex`) e il completamento con Tab per bash, zsh e fish in `~/.local/share`. Non servono i permessi di root.

```bash
tar xzf treex-<ver>-linux-x86_64.tar.gz
```

```bash
cd treex-<ver>-linux-x86_64 && ./install.sh
```

In alternativa avvia `./treex-gui` direttamente dalla cartella.

### Requisiti, verifica e disinstallazione

**Requisiti** (AppImage e archivio): un driver grafico Vulkan (Mesa su Intel e AMD, il driver proprietario NVIDIA) o OpenGL 3.3; X11 o Wayland; glibc 2.28 o successiva (Debian 10, Ubuntu 20.04, RHEL 8 o più recenti). L'archivio della 0.10.0 richiedeva ancora glibc 2.39. Per il Cestino e "Mostra nella cartella" serve un desktop freedesktop (GNOME, KDE, Xfce, Cinnamon…).

**Verificare il download** (vale per i tre formati):

```bash
sha256sum -c --ignore-missing SHA256SUMS-linux
```

**Disinstallare**: AppImage, cancella il file; Flatpak, `flatpak uninstall dev.treex.TreeX` (i suoi dati stanno in `~/.var/app/dev.treex.TreeX/`); archivio, `./install.sh --uninstall`. Dati creati da TreeX fuori dalla sandbox (tutti facoltativi):

| Percorso | Contenuto |
|---|---|
| `~/.cache/treex/` | snapshot delle analisi (si rigenerano) |
| `~/.local/state/treex/` | storico delle pulizie |
| `~/.local/share/treex/` | preferenze dell'app (lingua, vista) |
| `~/.config/treex/` | regole Dev-Aware personalizzate, se create |

## Aggiornare

Dall'app: **Aggiornamenti** in basso a destra nella home, oppure ⌘K → *Cerca aggiornamenti* ([dettagli](gui.md#aggiornare-treex)). TreeX scarica la nuova versione, ne verifica la firma digitale e il checksum, si sostituisce e si riavvia. In alternativa, o con Flatpak, scarica la nuova release e installala come la prima volta: le analisi salvate, lo storico delle pulizie e le preferenze restano.
