# Installazione — LibreCMS

Contenuto del pacchetto:

| File | A cosa serve |
|---|---|
| `librecms.zip` | Il CMS completo: codice, asset, editor compilato. Non ci sono dipendenze da installare. |
| `setup.php` | Estrazione guidata, da usare solo se il pannello dell'hosting non sa estrarre gli ZIP. |
| `librecms-update-X.Y.Z.zip` | Pacchetto di aggiornamento firmato. Non serve per la prima installazione. |
| `SHA256SUMS.txt` | Impronte per verificare che i file non siano stati alterati. |
| `docs/` | README (moduli, requisiti, versioni provate) e API per Claude Code. |
| `LICENSE.txt` | Licenza MIT: puoi usare LibreCMS liberamente, anche per siti commerciali. |

Non servono SSH, Composer, npm o comandi sul server. Prima di iniziare controlla i requisiti nel README: PHP 8.2+ (consigliato 8.4), MySQL o MariaDB, HTTPS.

## Il codice di installazione

Il pacchetto è uguale per tutti, quindi non contiene un codice. Lo crea il tuo sito la prima volta che apri l'installer e lo scrive in un file che solo chi gestisce l'hosting può aprire. In questo modo nessun altro può installare il CMS sul tuo dominio nel tempo che passa tra il caricamento e l'installazione.

Per trovarlo:

1. Apri il pannello del tuo hosting e poi il **File Manager** (oppure un programma FTP come FileZilla).
2. Entra nella cartella del sito (di solito `public_html`), poi in `app` e poi in `config`.
3. Apri il file `install-token.php` (di solito: clic destro → *Modifica* o *Visualizza*).
4. Copia il codice di 16 caratteri, simile a `K7QM-2XPA-9HTD-R4WN`, e incollalo nell'installer. Maiuscole, spazi e trattini non contano.

L'installer mostra le stesse istruzioni, con il percorso esatto del file. Se l'hai cancellato per errore, ricarica la pagina: viene creato un nuovo file con un nuovo codice. Il codice vale una sola volta e il file si cancella da solo a fine installazione.

## Modalità A — Estrazione dal pannello hosting (consigliata)

1. Crea dal pannello dell'hosting un database MySQL o MariaDB vuoto e un utente con tutti i permessi su quel database. Annota host, nome, utente e password.
2. Carica `librecms.zip` in `public_html` dal File Manager ed estrailo lì. Devono comparire `index.php`, `.htaccess`, `assets/`, `media/` e `app/`.
3. **Elimina lo ZIP** dal server.
4. Apri `https://tuodominio/install/` e inserisci il **codice di installazione** (vedi sopra: è nel file `app/config/install-token.php`).
5. Segui i passi successivi:
   1. compatibilità;
   2. database;
   3. tabelle;
   4. dati del sito e **scelta della configurazione** (Portfolio, Blog o Podcast);
   5. amministratore con **2FA obbligatoria**: inquadra il QR con un'app Authenticator e salva i 10 codici di recupero;
   6. verifica finale;
   7. completamento.

Alla fine l'installer cancella il file del codice e si blocca: `/install/` risponde 404.

## Modalità B — `setup.php` (se il pannello non estrae gli ZIP)

1. Carica `librecms.zip` **e** `setup.php` nella stessa cartella (`public_html`), in modalità binaria.
2. Apri `https://tuodominio/setup.php`. Lo script crea accanto a sé il file `setup-token.php`: aprilo con il File Manager e copia il codice di 16 caratteri, come sopra.
3. Lo script controlla che l'archivio sia esattamente quello del pacchetto (impronta SHA-256), poi verifica percorsi, dimensioni e spazio. Quindi estrae i file, **elimina lo ZIP, il file del codice e se stesso** e apre l'installer. Da lì prosegui dal punto 4 della Modalità A.
4. Se compare l'avviso che `setup.php` non è stato eliminato, cancellalo via FTP. Resta comunque disattivato.

`setup.php` funziona solo in HTTPS, accetta solo il suo ZIP, limita i tentativi e non fa nulla se il CMS è già presente.

## Configurazione più sicura (facoltativa): `app/` fuori da `public_html`

Se l'hosting permette cartelle sopra `public_html`, sposta la cartella `app/` un livello sopra, accanto a `public_html`: il front controller la trova da solo.

In alternativa crea `public_html/paths.php` con `<?php return ['app' => '/percorso/assoluto/app'];`.

Con nginx questa configurazione è obbligatoria.

## Dopo l'installazione

1. Accedi a `https://tuodominio/admin/`. Al primo accesso si apre la **Guida**: cinque schermate su come è fatto il pannello della configurazione scelta, i primi passi, sicurezza e aggiornamenti, e come collegare **Claude Code** con un token API. La riapri quando vuoi dalla voce **Guida** del menu.
2. In **Aspetto** imposta nome, logo, favicon e colori.
3. In **Moduli** accendi le funzioni che ti servono.
4. Le pagine di esempio sono già create e vanno completate. La **Privacy policy** resta in bozza finché non la scrivi tu. La **Cookie policy** descrive i soli cookie tecnici del CMS.
5. In **Sistema** crea un primo backup e scaricalo.
6. Le pubblicazioni programmate diventano visibili alla prima visita dopo l'orario previsto, senza bisogno di un cron. Per questo l'orario non è preciso al secondo.

## Controlli che si possono fare solo sul server reale

L'installer, ai passi «Compatibilità» e «Verifica», controlla:
- che il `.htaccess` venga rispettato;
- che `app/` sia protetta;
- che in `/media` non si possa eseguire PHP;
- HTTPS;
- versione e privilegi del database;
- i limiti di upload.

Se un controllo fallisce, l'installazione si ferma e indica il requisito mancante.
