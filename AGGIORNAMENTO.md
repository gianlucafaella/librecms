# Aggiornamento — gianlucafaella CMS

Gli aggiornamenti del **codice** (design, nuove funzioni, correzioni) si preparano sul PC con Claude Code e si installano dal browser con un pacchetto **firmato digitalmente**. I **contenuti** non vengono toccati.

## Una tantum sul PC: chiave di firma

```powershell
tools\php\php.exe bin\keygen.php
```

Crea la chiave privata Ed25519 in `%USERPROFILE%\.gianlucafaella-cms\release-signing.key` (fuori dal progetto, mai nel repository né sul server). La chiave pubblica viene inclusa automaticamente in ogni pacchetto e il sito accetta solo aggiornamenti firmati con la tua chiave.

**Fanne una copia offline** (es. password manager). Se la perdi: crea una nuova chiave, fai una nuova build e sostituisci via FTP `app/config/release.pub` sul server con la nuova chiave pubblica (stampata da `keygen.php`).

## 1. Preparare la nuova versione (sul PC)

1. Fai le modifiche con Claude Code e verifica in locale (`bash bin/dev-server.sh`, poi i test).
2. Aumenta la versione in `app/bootstrap.php` (`const GF_VERSION = '0.2.0';`).
3. Se servono modifiche al database, aggiungi una migrazione numerata in `app/migrations/` (es. `004_nuova_colonna.php`), scritta in modo che sia ripetibile senza errori.
4. Esegui la build:

```powershell
tools\php\php.exe bin\build.php
```

La build compila l'editor, esegue i test (si ferma se falliscono) e produce `dist/release-0.2.0/gianlucafaella-cms-update-0.2.0.zip`.

## 2. Installare l'aggiornamento (dal browser)

1. `/admin` → **Sistema** → carica il pacchetto → **Verifica e aggiorna**. Viene chiesta una nuova conferma con password e codice 2FA.
2. Il CMS, prima di modificare qualsiasi file:
   - verifica la **firma** e l'impronta SHA-256 di ogni file;
   - controlla che la versione sia più recente, la compatibilità (`min_from`), PHP ed estensioni richieste e lo spazio su disco;
   - accetta solo file del codice (`app/src`, `app/views`, `app/migrations`, `app/bootstrap.php`, `index.php`, `.htaccess`, `assets/`): configurazione, `app/storage`, media caricati e `paths.php` non possono essere toccati;
   - crea un **backup del database** (`pre-aggiornamento-<versione>`) e una **copia dei file attuali**.
3. Il sito pubblico va in **manutenzione** (HTTP 503), i file vengono sostituiti uno a uno in modo atomico, poi una nuova richiesta con il nuovo codice esegue le **migrazioni**, verifica versione e integrità dei file, svuota la cache e chiude la manutenzione.
4. Esito e dettagli compaiono in **Sistema** e nel **Registro attività**.

## Se qualcosa va storto

| Situazione | Cosa fare |
|---|---|
| Errore durante la scrittura dei file | Il CMS ripristina automaticamente i file precedenti e chiude la manutenzione. |
| Errore nelle migrazioni o nella verifica finale | Il sito resta in manutenzione. In **Sistema** premi **Ripristina versione precedente**; se il database era già stato modificato, ripristina anche il backup `pre-aggiornamento-…` dall'elenco backup. |
| Pagina bianca / admin non raggiungibile | Via FTP: ricopia i file dalla cartella `app/storage/backups/rollback-<versione>-<data>/` nelle posizioni originali (`public/…` → `public_html/…`, `app/…` → `app/…`) ed elimina `app/storage/maintenance.flag` e `app/storage/update-state.json`. |
| Recupero completo | Reinstalla lo ZIP della versione precedente su un database vuoto e ripristina l'ultimo backup dei contenuti. |

## Backup

- `/admin` → **Sistema** → **Crea backup**: database dei contenuti + media in uno ZIP salvato in `app/storage/backups/` (cartella privata, non scaricabile dal web). Scaricalo da lì e conservane una copia fuori dal server.
- **Ripristina** sostituisce i contenuti con quelli del backup, dopo averne creato uno di sicurezza. Utenti, password, 2FA e token non sono inclusi né modificati.
- I file di backup possono essere grandi: eliminali periodicamente dall'elenco dopo averli scaricati.

## Integrità

**Sistema** → **Integrità dei file** confronta il codice installato con il manifest della versione distribuita: segnala file modificati a mano o mancanti (utile anche per accorgersi di manomissioni).
