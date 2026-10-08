# Permessi su macOS (Full Disk Access)

macOS protegge con TCC (Transparency, Consent and Control) molte cartelle: `~/Library/Mail`, `~/Library/Safari`, `~/Library/Messages`, parti di `~/Library/Containers` e altre. Senza permesso, TreeX non può leggerle: compaiono come `errors` nelle statistiche e la loro dimensione **non** viene conteggiata.

Esempio misurato: una scansione di `~/Library` senza Full Disk Access ha prodotto 134 directory illeggibili.

## Concedere l'accesso completo al disco

1. Apri **Impostazioni di Sistema → Privacy e sicurezza → Accesso completo al disco**.
2. Premi **+** e aggiungi:
   - per la CLI: l'app del terminale che usi (Terminale, iTerm2, Ghostty…);
   - per la GUI (dalla 0.3.0): `TreeX.app`.
3. Riavvia il terminale o l'app.

L'analisi legge solo i **metadati** (nomi, dimensioni, date). Il contenuto dei file viene letto solo dalla ricerca dei duplicati, se la avvii, e mai inviato in rete.

## Controllo del Finder

**Ottieni informazioni** (menu contestuale) apre la finestra Info del Finder tramite AppleScript: la prima volta macOS chiede se TreeX può controllare il Finder. Se rifiuti, tutto il resto funziona; il permesso si cambia in **Impostazioni di Sistema → Privacy e sicurezza → Automazione**.

## Senza Full Disk Access
TreeX funziona lo stesso: le cartelle protette vengono saltate e segnalate. Dalla 0.2.0 la CLI elencherà le cartelle saltate; la GUI mostrerà un avviso con un pulsante che apre direttamente il pannello delle Impostazioni.

## Dopo aver concesso l'accesso

Riavvia TreeX: macOS applica il permesso solo all'avvio. Alla prossima analisi TreeX rilegge da solo le cartelle che prima erano bloccate e il conteggio si aggiorna. Se resta qualche cartella illeggibile, l'avviso lo dice senza proporre di nuovo il permesso: sono cartelle che nemmeno l'accesso completo apre (di sistema, di altri utenti o con permessi riservati all'amministratore).

