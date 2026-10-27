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
