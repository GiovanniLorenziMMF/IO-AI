# IO&AI – Come ti senti? Prima e dopo la visita

Pagina per tablet del gancio iniziale del percorso Target 3 (16–19 anni): la classe vota con tre faccine
prima e dopo la visita, e alla fine vede come è cambiato il sentiment.

## File

- `index.html` – l'applicazione, tutto in un file (logo incluso).
- `sw.js` – fa funzionare la pagina anche senza rete dopo la prima apertura online.

## Pubblicare su GitHub

1. Crea un repository e carica `index.html` e `sw.js` nella cartella principale.
2. Settings → Pages → Source: "Deploy from a branch", branch `main`, cartella `/root`.
3. Apri l'indirizzo `https://<utente>.github.io/<repository>/` sul tablet e aggiungilo alla schermata Home.

## Come si usa durante la visita

| Momento | Cosa fare |
| --- | --- |
| Inizio | "Nuova visita: apri Prima". Ogni studente tocca una faccina e passa il tablet. |
| Fine raccolta Prima | Tieni premuto "chiudi Prima". Il tablet mostra "Buona visita". |
| Fine visita | Tieni premuto "apri Dopo", si vota di nuovo, poi "chiudi Dopo e guarda il risultato". |
| Classe successiva | Tieni premuto "nuova visita". |

I comandi dell'educatrice/ore vanno tenuti premuti circa un secondo, così nessuno li attiva per sbaglio.
"Annulla l'ultima" toglie l'ultimo tocco se qualcuno ha sbagliato.
Se la pagina viene ricaricata o il tablet va in stand-by, la visita in corso non si perde.

Il punteggio è quello proposto: sorridente +1, neutra 0, triste −1. La posizione delle due facce nel
grafico è la media della classe (somma divisa per il numero di risposte), così Prima e Dopo restano
confrontabili anche se il numero di voti è diverso.

## Raccolta di tutte le risposte

Ogni tocco diventa una riga: `id; id_visita; data; ora; fase; risposta; punteggio; tablet`.

**Sempre, anche senza rete.** Le righe restano sul tablet. Dalla schermata dei risultati o dal menu
(tieni premuto ":: menu ::") si scaricano come CSV, già pronto per Excel in italiano. Se le risposte
non sono state inviate online, il CSV della visita si scarica da solo alla chiusura di Dopo.
Il menu permette anche di scaricare l'archivio di tutte le visite fatte su quel tablet.

**Online, in automatico.** Nel menu incolla un "indirizzo di raccolta": alla chiusura di Prima e di Dopo
il tablet spedisce le righe a quell'indirizzo. Se manca la rete le tiene in coda e riprova da solo.
L'indirizzo si imposta una volta per ogni tablet (oppure si scrive in `ENDPOINT_URL` dentro `index.html`,
ma in un repository pubblico chiunque potrebbe leggerlo e inviare righe false).

L'invio online non è stato provato su un vostro account: dopo la configurazione usate
"Invia una riga di prova" nel menu e controllate che la riga compaia nel foglio.

### Opzione A – Excel con Power Automate

Richiede una licenza Power Automate Premium (il trigger HTTP è un connettore premium).

1. Su OneDrive/SharePoint crea un file Excel con una tabella chiamata `Risposte` e le colonne
   `id`, `id_visita`, `data`, `ora`, `fase`, `risposta`, `punteggio`, `tablet`.
2. Power Automate → nuovo flusso cloud istantaneo → trigger "Quando viene ricevuta una richiesta HTTP".
   In "Chi può attivare il flusso" scegli "Chiunque".
3. Aggiungi "Analizza JSON" con contenuto l'espressione `json(triggerBody())` e questo schema:
   ```json
   {"type":"object","properties":{"rows":{"type":"array","items":{"type":"object","properties":{
   "id":{"type":"string"},"id_visita":{"type":"string"},"data":{"type":"string"},"ora":{"type":"string"},
   "fase":{"type":"string"},"risposta":{"type":"string"},"punteggio":{"type":"integer"},"tablet":{"type":"string"}}}}}}
   ```
4. Aggiungi "Applica a ogni" su `rows` e, dentro, "Excel Online (Business) – Aggiungi una riga a una tabella",
   collegando ogni colonna al campo con lo stesso nome.
5. Salva: il trigger mostra l'URL da incollare nel menu del tablet.

### Opzione B – Google Fogli (gratuita), leggibile anche da Excel

1. Crea un foglio Google → Estensioni → Apps Script, e incolla:
   ```js
   function doPost(e) {
     var lock = LockService.getScriptLock(); lock.waitLock(20000);
     try {
       var d = JSON.parse(e.postData.contents);
       var sh = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
       var cols = ['id','id_visita','data','ora','fase','risposta','punteggio','tablet'];
       if (sh.getLastRow() === 0) sh.appendRow(cols);
       d.rows.forEach(function (r) { sh.appendRow(cols.map(function (c) { return r[c]; })); });
     } finally { lock.releaseLock(); }
     return ContentService.createTextOutput('ok');
   }
   ```
2. Esegui il deployment → Nuovo deployment → Applicazione web → Esegui come: te → Accesso: Chiunque.
3. Incolla nel menu del tablet l'URL che termina con `/exec`.
4. Per averlo in Excel: nel foglio Google, File → Condividi → Pubblica sul Web → CSV; in Excel,
   Dati → Da Web con quel link. Excel si aggiorna con "Aggiorna tutto".

## Caratteri della mostra

I titoli usano un corsivo con grazie reso a pixel, il testo un sans tipo Helvetica, cioè i caratteri
più vicini a Untitled AI e Swis721 BT disponibili su ogni tablet. Se avete i file originali, metteteli in
`fonts/UntitledAI.woff2` e `fonts/Swis721BT.woff2`: la pagina li usa da sola.
