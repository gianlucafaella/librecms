# Installazione — gianlucafaella CMS

Contenuto della consegna:

| File | A cosa serve |
|---|---|
| `gianlucafaella-cms.zip` | Il CMS completo (codice, asset, editor compilato). Nessuna dipendenza da installare. |
| `setup.php` | Estrazione guidata, solo se il pannello dell'hosting non sa estrarre gli ZIP. |
| `CREDENZIALE-INSTALLAZIONE.txt` | Token monouso per l'installer. **Non caricarlo sul server**, tienilo separato dallo ZIP. |
| `gianlucafaella-cms-update-X.Y.Z.zip` | Pacchetto di aggiornamento firmato (non serve per la prima installazione). |
| `SHA256SUMS.txt` | Impronte per verificare che i file non siano stati alterati. |

Non servono SSH, Composer, npm o comandi sul server.

## Requisiti dell'hosting

- PHP **8.4 o superiore** con estensioni: pdo_mysql, openssl, sodium, mbstring, fileinfo, json, session; consigliate gd (con WebP) e zip.
- MySQL 8+ o MariaDB 10.6+ con un database vuoto e un utente con permessi CREATE, ALTER, INDEX, INSERT, UPDATE, DELETE, SELECT, DROP, REFERENCES.
- Apache o LiteSpeed con `.htaccess` attivo (mod_rewrite). Con nginx la cartella `app/` deve stare fuori da `public_html` (vedi sotto).
- Certificato **HTTPS** attivo sul dominio prima di iniziare.
- Limite upload consigliato: `upload_max_filesize` ≥ 10M (immagini e pacchetti di aggiornamento).

L'installer verifica tutto con controlli reali (anche con richieste HTTP verso il sito stesso) e **si blocca** se i file riservati risultano scaricabili.

## Prima di iniziare: il vecchio sito WordPress

1. Dal WordPress attuale: Strumenti → Esporta (backup dei contenuti) e un backup completo da pannello hosting.
2. Svuota `public_html` (o rinominala) solo dopo aver salvato tutto. Gli URL del nuovo sito sono compatibili con quelli vecchi (stessi slug, redirect per categorie e pagine).

## Modalità A — Estrazione dal pannello hosting (consigliata)

1. Carica `gianlucafaella-cms.zip` in `public_html` dal File Manager.
2. Estrai l'archivio lì (devono comparire `index.php`, `.htaccess`, `assets/`, `media/`, `app/`).
3. **Elimina lo ZIP** dal server.
4. Apri `https://tuodominio/install/` e inserisci il token di `CREDENZIALE-INSTALLAZIONE.txt`.
5. Segui gli 8 passi: compatibilità → database → tabelle → dati del sito → amministratore con **2FA obbligatoria** (inquadra il QR con un'app Authenticator e salva i 10 codici di recupero) → verifica finale → completamento.

## Modalità B — `setup.php` (se il pannello non estrae gli ZIP)

1. Carica `gianlucafaella-cms.zip` **e** `setup.php` nella stessa cartella (`public_html`), in modalità binaria.
2. Apri `https://tuodominio/setup.php` e inserisci il token.
3. Lo script verifica che l'archivio sia esattamente quello della build (impronta SHA-256), controlla percorsi, dimensioni e spazio, estrae i file, **elimina lo ZIP e se stesso** e apre l'installer (stesso token).
4. Se compare l'avviso che `setup.php` non è stato eliminato, cancellalo via FTP (resta comunque disattivato).

`setup.php` funziona solo in HTTPS, accetta solo il suo ZIP, limita i tentativi del token e non fa nulla se il CMS è già presente.

## Configurazione più sicura (facoltativa): `app/` fuori da `public_html`

Se l'hosting permette cartelle sopra `public_html`:

1. Sposta la cartella `app/` un livello sopra (accanto a `public_html`). Il front controller la trova da solo in `../app`.
2. In alternativa crea `public_html/paths.php` con `<?php return ['app' => '/percorso/assoluto/app'];`.

Con nginx questa configurazione è obbligatoria.

## Trasferire i contenuti dal PC al sito online

I contenuti preparati in locale (articoli importati da WordPress, immagini, pagine) si trasferiscono con un backup:

1. In locale: `/admin` → **Sistema** → **Crea backup** (con media) → **Scarica**.
2. Online, dopo l'installazione: `/admin` → **Sistema** → **Ripristina da file**. Se il file supera il limite di upload, copialo via FTP in `app/storage/backups/` e ripristinalo dall'elenco.

Il ripristino sostituisce contenuti, media, impostazioni del sito e redirect; **non tocca utenti, password, 2FA e token**. Prima crea automaticamente un backup di sicurezza dello stato attuale.

## Dopo l'installazione

- Accedi a `https://tuodominio/admin/`.
- In **Sicurezza** crea il token API per Claude Code (vedi `docs/API.md` nel repository).
- In **Sistema** crea un primo backup e scaricalo.
- Le pubblicazioni programmate vengono rese visibili alla prima visita dopo l'orario previsto (nessun cron necessario): l'orario esatto al secondo non è garantito.

## Requisiti da verificare sul server reale

Non verificabili in sviluppo, controllati dall'installer al passo 2 e 7: rispetto del `.htaccess`, protezione di `app/` dentro `public_html`, nessuna esecuzione di PHP in `/media`, HTTPS, versione e privilegi del database, limiti di upload. Se un controllo fallisce l'installazione si ferma con l'indicazione del requisito mancante.
