# Configurazione e regole Dev-Aware

> Disponibile (M3). `treex rules list` mostra tutte le regole integrate.

## Dove si trovano i file

| File | Ambito |
|---|---|
| preset integrati | Rust, JS/TS, Python, Apple, Java/Kotlin, C/C++, Docker |
| `~/.config/treex/rules.toml` (macOS e Linux), `%APPDATA%\TreeX\rules.toml` (Windows) | le tue regole personali |
| `.treex.toml` nella root di un progetto | regole del team, da versionare insieme al codice |

Le regole con lo stesso `id` sostituiscono quella del preset.

## Esempi

Disattivare un preset:

```toml
[[rule]]
id = "python.venv"
enabled = false
```

Aggiungere un artefatto del tuo stack (es. Bazel):

```toml
[[rule]]
id          = "bazel.out"
ecosystem   = "bazel"
label       = "Bazel output"
safety      = "regenerable"
regenerate  = "bazel build //..."
glob        = ["bazel-*"]
sibling_any = ["WORKSPACE", "MODULE.bazel"]
```

Proteggere una cartella:

```toml
[[protect]]
path = ["~/Work/clienti"]
```

## Software installato e cartelle nascoste

TreeX non cerca artefatti di *progetto* dentro il software installato: lì `node_modules`, `dist` e `build` fanno parte del prodotto.

- Per nome di cartella, ovunque: le app (`*.app`) e i framework (`*.framework`), i pacchetti scaricati (`node_modules`: un `dist/` lì dentro è il pacchetto stesso), le installazioni di Python (`python3.*`, `site-packages`), Homebrew (`Cellar`, `Caskroom`), conda, `AppData` (`inside_name` in `[[protect]]`, con i caratteri jolly).
- Su macOS il disco intero si può analizzare come `/System/Volumes/Data`: le protezioni valgono anche lì (`/System/Volumes/Data/Applications` è `/Applications`).
- Le cartelle di altri utenti non vengono proposte: solo il loro proprietario può eliminarle.

- Cartelle nascoste: niente artefatti di progetto sotto `~/.qualcosa/…` (estensioni dell'editor, tool installati). Si può disattivare con:

  ```toml
  [defaults]
  skip_hidden_parents = false
  ```
- Alberi gestiti: `inside` in `[[protect]]` (per default `~/Library`, `/Applications`, `/usr`, `/opt`…):

  ```toml
  [[protect]]
  inside = ["~/Tools"]
  ```

Le cache globali definite per percorso (DerivedData, Homebrew, pip, npm…) restano sempre riconosciute.

## Chiavi di firma e credenziali

Certificati, chiavi e credenziali non si toccano mai: TreeX non pulisce nulla **dentro, sopra o al posto** di queste cartelle, nemmeno con una regola per percorso o con una regola tua (`sealed` in `[[protect]]`, D48).

| Sistema | Sigillati per default |
|---|---|
| Tutti | `~/.ssh`, `~/.gnupg`, `~/.appstoreconnect`, `~/private_keys`, `~/.password-store`, `~/.aws`, `~/.azure`, `~/.kube`, `~/.config/gh`, `~/.config/gcloud`, le credenziali di Snapcraft, Cargo e Docker, `~/.android/debug.keystore`, `~/.gradle/gradle.properties` |
| macOS | portachiavi (`~/Library/Keychains`: Developer ID, profili di notarizzazione, account di Xcode), `~/Library/MobileDevice`, profili di provisioning di Xcode, `~/Library/Preferences` |
| Linux | portachiavi (`~/.local/share/keyrings`, KWallet), `~/.pki` |
| Windows | archivi dei certificati e delle chiavi (`AppData\Roaming\Microsoft\Crypto`, `SystemCertificates`, `Protect`), credenziali salvate |

Per nome, ovunque: `.ssh`, `.gnupg`, `private_keys`, `.appstoreconnect`, `Provisioning Profiles`, `Keychains`. Un file elencato (per esempio `debug.keystore`) tiene intera la sua cartella; le cache accanto (`~/.gradle/caches`) si puliscono come prima. Inoltre nessuna cartella che contiene direttamente un certificato o una chiave (`*.p12`, `*.p8`, `*.pfx`, `*.jks`, `*.keystore`, `key.properties`, `*.mobileprovision`, `*.provisionprofile`) viene mai proposta.

Per sigillare altre cartelle:

```toml
[[protect]]
sealed = ["~/Firma"]
sealed_name = ["certificati"]
```

## Pacchetti di release e report dei test

- **Pacchetti di release** (`release.packages`, *da verificare*, mai selezionati da soli): cartelle come `dist`, `dist-*`, `release`, `release-*`, `build`, `out` di un progetto (accanto a `.git`, `Cargo.toml`, `package.json`, `*.xcodeproj`…) che contengono, fino a 3 livelli sotto, installer con la versione nel nome (`App-1.4.2.dmg`, `.exe`, `.msi`, `.msix`, `.AppImage`, `.deb`, `.rpm`, `.apk`, `.ipa`, `.zip`…). Una cartella che tiene un certificato o una chiave di firma non viene mai proposta (le copie dentro le app compilate e nelle dipendenze scaricate non contano).
- **Report dei test e della copertura** (`test.coverage`, `test.reports`, *rigenerabili*): `coverage`, `htmlcov`, `.nyc_output`, `test-results`, `playwright-report`, `allure-results`, `TestResults`… accanto ai file di un progetto.

Le regole possono usare gli stessi controlli:

```toml
[[rule]]
id         = "my.installers"
dir        = ["pacchetti"]
holds_any  = ["*.dmg", "*.msi"]   # almeno uno, fino a 3 livelli sotto
versioned  = true                 # solo nomi con una versione (1.2, v0.3.0)
holds_none = ["*.p12", "*.jks"]   # nessuno, a qualsiasi profondità
```

## Simulatori di Xcode ed emulatori Android

- **Simulatori di Xcode** (`~/Library/Developer/CoreSimulator/Devices`, da verificare): si tolgono sempre con lo strumento di Xcode (`xcrun simctl … delete all`), mai spostati nel Cestino, così l'elenco dei dispositivi resta corretto. Si ricreano in Xcode › Devices and Simulators. Se un simulatore è acceso, TreeX non li tocca.
- **Emulatori Android** (`~/.android/avd/*.avd`, da verificare, su macOS, Windows e Linux): si tolgono solo i dati (dati utente, cache, snapshot), come *Wipe Data* di Device Manager; l'emulatore resta nell'elenco e riparte pulito. Un emulatore acceso viene saltato.
- **SDK Android**: immagini di sistema e NDK (da verificare, si riscaricano da SDK Manager o da Gradle), cache di `~/.android` e di Android Studio. TreeX guarda le cartelle abituali (`~/Library/Android/sdk`, `~/Android/Sdk`, `%LOCALAPPDATA%\Android\Sdk`) e quelle di `ANDROID_HOME`, `ANDROID_SDK_ROOT`, `ANDROID_AVD_HOME`.

Nelle regole, i `path` accettano caratteri jolly in ogni parte e una variabile d'ambiente all'inizio; `only` limita `delete-children` alle voci indicate e `lock` salta la cartella mentre il suo programma è in esecuzione:

```toml
[[rule]]
id     = "my.emulators"
path   = ["$MY_EMULATORS/*.vm"]
action = "delete-children"
only   = ["disk.img", "snapshots"]
lock   = "running.lock"
```

Le cartelle trovate che non contengono alcun file (solo cartelle vuote) non vengono proposte: non libererebbero nulla.

## Età minima

Per proporre solo i progetti fermi da un po':

```toml
[defaults]
min_age_days = 30
```

Oppure per singola regola, con `min_age_days` dentro la `[[rule]]`.

## Classi di sicurezza

- **cache**: si ricostruisce da sola. Inclusa in "Seleziona tutto".
- **regenerable**: si ricostruisce con un comando (mostrato in UI).
- **stateful**: potrebbe contenere modifiche manuali. Mai preselezionata.

## Cartelle fatte di cloni (macOS)

Con `clones = "likely"` una regola dice che le sue cartelle sono spesso fatte di **cloni APFS**: occupano molto più di quanto libera eliminarle. Finché TreeX non ha misurato lo spazio che si libera davvero (un attimo dopo l'analisi, e mai nella mappa), accanto alla dimensione compare l'avviso *Su APFS potrebbe liberare molto meno: file clonati*. Le regole integrate lo usano per i dispositivi dei simulatori di Xcode, DerivedData e lo store di pnpm; non per le cache di npm, Homebrew o pip, che di solito non contengono cloni.

```toml
[[rule]]
id = "mia.cache-clonata"
path = ["~/Library/Caches/MioStrumento"]
safety = "cache"
clones = "likely"
```

## Note e comandi

`note` è una spiegazione breve mostrata accanto alla cartella (nell'app, nella TUI e da `treex dev`/`treex clean`); `command` è un comando che pulisce la cartella nel modo dello strumento che l'ha creata, mostrato con un pulsante **Copia comando**. TreeX non esegue mai `command`.

```toml
[[rule]]
id = "mio.strumento"
path = ["~/Library/Caches/MioStrumento"]
safety = "cache"
note = "Cache del mio strumento, ricreata al prossimo avvio."
command = "mio-strumento cache clean"
```

## Simulatori di Xcode (macOS)

`action = "simctl-delete-all"` è per le cartelle che contengono un insieme di simulatori di Xcode, come `~/Library/Developer/XCTestDevices`: con l'eliminazione definitiva TreeX usa `xcrun simctl --set <cartella> delete all`, che aggiorna anche l'elenco dei dispositivi, poi toglie le cartelle rimaste di dispositivi non più elencati. Lo fa solo se `xcrun` trova `simctl` e Xcode non deve ancora installare i suoi componenti, e salta la cartella se uno dei simulatori è acceso (un test in corso); altrimenti, e con il Cestino, svuota la cartella come `delete-children`. Vale solo per percorsi (`path`) dentro `~/Library/Developer/`.

## Verificare le regole

```bash
treex rules check
```

```bash
treex rules test ~/Developer/app/node_modules
```
