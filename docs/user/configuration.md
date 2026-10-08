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

## Verificare le regole

```bash
treex rules check
```

```bash
treex rules test ~/Developer/app/node_modules
```
