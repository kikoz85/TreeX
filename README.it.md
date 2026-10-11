<p align="center"><img src="assets/logo.svg" width="140" alt="TreeX logo"></p>

<h1 align="center">TreeX</h1>
<p align="center"><a href="README.md">English</a> · <b>Italiano</b></p>
<p align="center"><b>Il disk analyzer per sviluppatori.</b> Velocissimo, cross-platform, consapevole degli artefatti di build.</p>
<p align="center"><a href="https://github.com/kikoz85/TreeX/releases/latest"><img src="https://img.shields.io/github/v/release/kikoz85/TreeX?label=download&color=7BE486" alt="Download"></a> <a href="https://www.enricofanucchi.com/it/app/treex?utm_source=github&utm_campaign=readme"><img src="https://img.shields.io/badge/sito-enricofanucchi.com-F7941D" alt="Sito di TreeX"></a> <a href="https://github.com/sponsors/kikoz85"><img src="https://img.shields.io/badge/sponsor-%E2%9D%A4-ea4aaa?logo=githubsponsors&logoColor=white" alt="Sponsor TreeX"></a> <img src="https://img.shields.io/badge/macOS%20%C2%B7%20Windows%20%C2%B7%20Linux-1EBCC3" alt="macOS · Windows · Linux"></p>

<p align="center"><img src="docs/images/gui-iso.png" width="820" alt="TreeX: il disco come una città isometrica"></p>

---

TreeX scansiona milioni di file in pochi secondi usando le API native di ogni sistema operativo, riconosce lo spazio sprecato da `node_modules`, `target/`, `DerivedData`, `.venv`, cache Docker e Gradle, e lo mostra in una treemap 2D / 2.5D isometrica pensata per essere esplorata, non solo letta.

> **TreeX 1.0 funziona su macOS, Windows e Linux.** macOS 13 Ventura o successivo (Apple Silicon e Intel), Windows 10/11 (x64), Linux (x86_64). Stessa app, stesse viste, stessi comandi ovunque.

### Dev-Aware: trova lo spazio che puoi riprenderti

<p align="center"><img src="docs/images/gui-reclaim.png" width="820" alt="Modalità Reclaim: gli artefatti di build come torri corallo"></p>

`treex dev ~` oppure, nell'app, il tasto `4`: `node_modules`, `target`, build Flutter, `DerivedData`, `vendor`, virtualenv e cache di npm/pip/Homebrew emergono come torri corallo, con il comando per ricrearli e da quanto il progetto è fermo. 16 ecosistemi di sviluppo riconosciuti, più le cache di sistema, regole personalizzabili in TOML.

| Home | 2D con zoom sul dettaglio | Ricerca ⌘K |
|---|---|---|
| ![](docs/images/gui-home.png) | ![](docs/images/gui-zoom.png) | ![](docs/images/gui-palette.png) |

## Perché la 1.0 (dopo la 0.10.2)

TreeX è passato dalla 0.10.2 direttamente alla **1.0.0** perché lo riteniamo funzionante e stabile su tutti e tre i sistemi, non perché sia stata spuntata una lista di funzioni:

- **La stessa app ovunque**: macOS, Windows e Linux hanno le stesse viste, gli stessi comandi e le stesse regole di sicurezza, costruite e provate su macchine reali (un Mac, un PC Windows 11, una macchina Debian 13), non solo in CI.
- **Testato**: l'intera suite di test passa su tutti e tre i sistemi, compresi i test di rendering sulla GPU vera, il confronto degli aggiornamenti incrementali con le analisi complete e i test di eliminazione che provano a ingannare TreeX (cartelle scambiate, collegamenti simbolici, percorsi protetti).
- **Veloce, e misurato**: il più veloce di tutti gli strumenti confrontati su ogni sistema, con i numeri qui sotto.
- **Sicuro per costruzione**: tutto va nel Cestino per default, ogni elemento viene ricontrollato un istante prima di toccarlo, le cartelle di sistema e della home sono sempre protette, ogni pulizia è registrata.
- **Rodato dall'uso reale**: le versioni 0.10.x hanno corretto ciò che è emerso usando TreeX ogni giorno (layout sugli schermi piccoli, chiusura dei pannelli, accesso completo al disco, avanzamento durante l'aggiornamento di un'analisi).

1.0 **non** significa che tutto è finito. La firma del codice (così macOS e Windows smettono di chiedere conferma al primo avvio), i pacchetti per i negozi di app, un'interfaccia da terminale e alcuni miglioramenti del motore continuano nelle versioni 1.x. Dalla 1.0 in poi, all'interno della 1.x, la riga di comando e il formato dei file di analisi cambiano solo in modo compatibile.

## Prestazioni

TreeX usa la strada veloce di ogni sistema: `getattrlistbulk` su macOS, `getdents64` + `statx` su Linux, `GetFileInformationByHandleEx` su Windows e, per un intero disco NTFS da amministratore, la lettura diretta della Master File Table.

### macOS

Apple M4, macOS, stessi alberi per tutti i tool (hyperfine, 5 run).

**Dalla seconda scansione** TreeX non riscansiona: carica lo snapshot e chiede a macOS (FSEvents) cosa è cambiato, poi rilegge solo quello.

| | **treex (seconda scansione)** | treex (prima scansione) | dua | du |
|---|---|---|---|---|
| fixture 324k entry | **0,05 s** | 0,35 s | 0,48 s | 0,80 s |
| fixture deep 1M entry | **0,06 s** | 1,99 s | 1,97 s | 12,14 s |
| fixture wide 1M entry | **0,05 s** | 1,48 s | 1,55 s | 4,60 s |
| `/Applications` (808k entry) | **0,06 s** | 1,70 s | 1,56 s | 8,60 s |

Anche alla prima scansione TreeX è alla pari o davanti al più veloce dei concorrenti (`dua`) e 1,7–2,5× più veloce di pdu, ncdu, dust e gdu.

### Linux

Intel N95, ext4, Debian 13, cache calda (hyperfine, 5 run). TreeX è il più veloce su tutti gli alberi:

| | **treex** | pdu | gdu | dust | dua | du |
|---|---|---|---|---|---|---|
| fixture, 324k voci | **0,21 s** | 0,28 s | 0,47 s | 0,44 s | 0,59 s | 0,69 s |
| fixture profonda, 1M voci | **0,66 s** | 0,88 s | 1,35 s | 1,51 s | 1,69 s | 2,28 s |
| `/usr` | **0,29 s** | 0,33 s | 0,51 s | 0,49 s | 0,57 s | 0,84 s |
| home, 3,8M voci | **2,82 s** | 3,64 s | 4,73 s | 6,14 s | 7,07 s | 9,33 s |

### Windows

Windows 11, 8 core, SSD SATA, NTFS, da amministratore, cache calda (hyperfine). TreeX è il più veloce su tutti gli alberi:

| | **treex** | gdu | dua | pdu | dust |
|---|---|---|---|---|---|
| fixture, 324k voci | **0,56 s** | 0,76 s | 0,87 s | 6,20 s | 6,74 s |
| `C:\Program Files` | **0,50 s** | 0,76 s | 0,90 s | 3,76 s | 4,02 s |
| `C:\Users`, 703k voci | **2,68 s** | 2,86 s | 3,31 s | 15,03 s | 16,73 s |
| intero disco `C:`, 1,56M voci | **5,82 s** | 11,29 s | 12,53 s | 27,96 s | 32,19 s |

Dalla seconda analisi il journal USN aggiorna l'intero `C:` in **0,2 s**.

Mentre l'app è aperta la mappa **si aggiorna dal vivo** su tutti e tre i sistemi: TreeX rilegge solo le cartelle cambiate (FSEvents su macOS, il journal USN e `ReadDirectoryChangesW` su Windows, inotify su Linux).

### Fluida su ogni GPU

La città 2.5D gira sulla GPU. TreeX misura ogni frame: sulle GPU potenti tiene l'antialiasing 4× al massimo, e solo quando una mappa enorme è troppo pesante per una piccola grafica integrata lo riduce per restare fluida. Su un portatile con Intel UHD 620 una mappa di 143.000 blocchi gira a 60 fps in 2D e in 2.5D.

### Pulizia sicura

Selezioni nel pannello Dev-Junk (o `treex clean --apply`), confermi, e le torri **affondano**. Ogni cartella viene ricontrollata un istante prima di toccarla (stessa cartella, non un symlink, marker ancora presenti, nessun repository git dentro); per default finisce nel **Cestino**. Ogni operazione resta nello storico (`treex history`).

| Scegli | Conferma | Fatto |
|---|---|---|
| ![](docs/images/gui-clean-select.png) | ![](docs/images/gui-clean-confirm.png) | ![](docs/images/gui-clean-done.png) |

### Tipi di file e duplicati

Premi `T` per vedere dove va lo spazio **per estensione**: clicca un tipo e nella mappa, in 2D e in 2.5D, restano accesi solo i suoi file. Premi `U` per **trovare i file duplicati**: TreeX confronta prima le dimensioni, poi l'inizio e la fine di ogni file, e legge il contenuto completo solo dei file che coincidono ancora. Gli artefatti di build sono esclusi per default, perché pieni di file di pacchetti identici. Scegli quale copia tenere (per default la più vecchia) ed elimina le altre: ognuna viene confrontata di nuovo con quella tenuta un istante prima. Da terminale: `treex types` e `treex dupes --delete`.

| Tipi di file (`T`) | Duplicati (`U`) |
|---|---|
| ![](docs/images/gui-types.png) | ![](docs/images/gui-dupes.png) |

### Ogni file, ogni volume, qualsiasi eliminazione

- **Lista dei file** (`L`): tutti i file o le cartelle dell'analisi, ricerca con wildcard (`*.mov`, `Downloads/*.dmg`), filtri per dimensione ed età, colonne ordinabili ed **export CSV / JSON** (`treex export`).
- **Elimina qualsiasi cosa**: ⌘-clic per selezionare, ⌘⌫ per il Cestino, ⌥⌘⌫ per eliminare definitivamente. TreeX ricontrolla ogni elemento un istante prima di toccarlo, non segue mai i collegamenti e rifiuta sempre le cartelle di sistema e quelle principali della Home (`treex delete`).
- **Tasto destro** per Quick Look, Apri con, Ottieni informazioni, Mostra nel Finder e altro.
- **Questo Mac / Questo PC / Questo computer**: dischi, dischi esterni, immagini disco e condivisioni di rete, con lo spazio libero (`treex volumes`).
- **Salva un'analisi** (⌘S) e riaprila dopo, anche su un altro Mac, senza riscansionare (⇧⌘O, `--save` / `--from`).

### Più di una mappa del disco

- **Cosa è cambiato?** (`C`, `treex diff`): TreeX tiene da solo l'analisi precedente di ogni cartella, così ti dice cosa è nuovo, cosa è cresciuto e cosa è sparito da ieri, senza salvare niente.
- **Cloni APFS**: le copie fatte sullo stesso volume spesso condividono i blocchi. TreeX mostra lo spazio che un'eliminazione libera **davvero**, segnala i cloni tra i duplicati e non promette spazio che non torna.
- **Spazio eliminabile**: ogni volume mostra quanto macOS libererà da solo (cache, istantanee locali di Time Machine, file di iCloud scaricati).

<p align="center"><img src="docs/images/gui-changes.png" width="820" alt="Cosa è cambiato dall'analisi precedente"></p>

| Lista dei file (`L`) | Eliminazione con conferma |
|---|---|
| ![](docs/images/gui-files.png) | ![](docs/images/gui-delete-confirm.png) |

## Installazione

Dalle [release](https://github.com/kikoz85/TreeX/releases):

- **macOS**: apri `TreeX-<versione>-macos-universal.dmg` e trascina **TreeX** in Applicazioni. La CLI `treex` è inclusa nell'app.
- **Windows**: estrai `TreeX-<versione>-windows-x64.zip` e avvia **TreeX.exe** (la CLI è `bin\treex.exe`). Da amministratore l'analisi di un disco intero legge la Master File Table ed è istantanea dalla seconda volta (journal USN).
- **Linux**: `TreeX-<versione>-x86_64.AppImage` (un file solo, gira sulla maggior parte delle distribuzioni), `TreeX-<versione>-x86_64.flatpak` (`flatpak install --user …`), oppure `treex-<versione>-linux-x86_64.tar.gz` con `./install.sh` (senza root).

Primo avvio, Homebrew e disinstallazione: [docs/user/install.md](docs/user/install.md).

**Aggiornare**: clicca **Aggiornamenti** nell'angolo in basso a destra della home (oppure ⌘K → *Cerca aggiornamenti*). TreeX chiede a GitHub qual è l'ultima release e, se scegli di aggiornare, la scarica, ne verifica la firma digitale, si sostituisce e si riavvia. Flatpak non permette alle app di aggiornarsi da sole: installa il nuovo file `.flatpak`.

## Uso rapido

```bash
treex scan ~/Developer -d 2
treex dev ~
treex types ~                    # spazio per estensione
treex dupes ~/Pictures           # file duplicati (≥ 1 MiB per default)
treex export ~ --name '*.mov' -o film.csv   # tutti i .mov, in CSV
treex delete ~/Downloads/vecchio.iso       # nel Cestino, con i controlli
treex diff ~/Developer                     # cosa è cambiato dall'analisi precedente
```

Nell'app: `Tab` alterna 2D / 2.5D, in 2.5D ↺ ↻ (o `Q` / `E`) ruotano la città, come due dita sul trackpad del Mac, rotella per zoomare sul dettaglio, `⌘K` cerca ovunque, `T` mostra i tipi di file, `U` cerca i duplicati, `L` elenca tutti i file, `C` mostra cosa è cambiato; ⌘-clic seleziona, ⌘⌫ sposta nel Cestino, tasto destro per il menu (su Windows e Linux: `Ctrl`-clic, `Canc`). Guida completa in [docs/user/gui.md](docs/user/gui.md).

## Documentazione

Le guide per l'utente sono in [`docs/`](docs/README.md): installazione, l'app, la riga di comando, la configurazione e i permessi di macOS. Le novità di ogni versione sono nelle [note di rilascio](https://github.com/kikoz85/TreeX/releases).

TreeX usa la rete solo quando clicchi **Aggiornamenti**: una richiesta a GitHub per sapere qual è l'ultima release, più il download se scegli di aggiornare. Nulla di te o del tuo disco viene inviato. Le scansioni leggono solo i metadati (nomi, dimensioni, date); il contenuto dei file viene letto solo quando avvii una ricerca di duplicati, e solo per i file che hanno la stessa dimensione di un altro.

## Sostieni TreeX

TreeX è gratuito. Se ti ha fatto recuperare qualche gigabyte (o un po' di tempo), puoi sostenerne lo sviluppo su **[GitHub Sponsors](https://github.com/sponsors/kikoz85)**. Le sponsorizzazioni pagano ciò che serve a un'app gratuita per crescere:

- i certificati di firma del codice, così macOS e Windows smettono di chiedere conferma al primo avvio;
- i pacchetti per i negozi di app (Homebrew, winget, Flathub);
- le macchine di prova per i tre sistemi, e il tempo per mantenere TreeX il più veloce della sua categoria.

<p align="center"><a href="https://github.com/sponsors/kikoz85"><img src="https://img.shields.io/badge/Sostieni%20TreeX-%E2%9D%A4-ea4aaa?style=for-the-badge&logo=githubsponsors&logoColor=white" alt="Sostieni TreeX su GitHub"></a></p>

Trovi anche il pulsante **♥ Sostieni TreeX** nell'angolo in basso a destra della home dell'app. Aiuta anche mettere una stella al repository e parlare di TreeX ad altri sviluppatori.

## Licenza

A scelta, [MIT](LICENSE-MIT) o [Apache-2.0](LICENSE-APACHE). Le licenze delle dipendenze sono in [THIRD_PARTY_LICENSES.md](THIRD_PARTY_LICENSES.md).
