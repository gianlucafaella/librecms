# Aggiornamento — LibreCMS

Ci sono due situazioni:

- **A.** Sul sito c'è già LibreCMS e vuoi installare una versione più recente: aggiornamento firmato, dal browser.
- **B.** Il sito usa la versione da cui LibreCMS deriva (profili «gianlucafaella» o «culturasommersa», versioni 0.1.x) e vuoi passare a LibreCMS: aggiornamento manuale via FTP, una sola volta.

In entrambi i casi **contenuti, account, 2FA, file caricati, redirect e URL restano come sono**.

## A. Da LibreCMS a una versione più recente di LibreCMS

1. Scarica il pacchetto `librecms-update-X.Y.Z.zip` dalla pubblicazione ufficiale e confronta l'impronta con `SHA256SUMS.txt`.
2. Vai in `/admin` → **Sistema** → **Aggiornamento del CMS**, scegli lo ZIP dal tuo computer e premi **Verifica e aggiorna**. Non servono FTP né SSH. Ti verranno chiesti di nuovo password e codice 2FA.
3. Prima di modificare qualsiasi file, il CMS:
   - verifica la **firma digitale**, che deve corrispondere a `app/config/release.pub`;
   - verifica l'impronta di ogni file, la versione, PHP, le estensioni e lo spazio su disco;
   - salva un backup del database e una copia dei file attuali.
4. Il sito va in manutenzione per qualche istante. I file vengono sostituiti, poi vengono applicate le migrazioni del database, controllata l'integrità e chiusa la manutenzione.

Configurazione (`app/config`), `app/storage`, media caricati e `paths.php` non vengono mai toccati.

**Se qualcosa va storto:**
- Se fallisce la scrittura dei file, il CMS ripristina da solo i file precedenti.
- Se falliscono le migrazioni o la verifica finale, il sito resta in manutenzione. In **Sistema** premi **Ripristina versione precedente** e, se serve, ripristina il backup `pre-aggiornamento-…`.
- Se il pannello non risponde, ricopia via FTP i file da `app/storage/backups/rollback-<versione>-<data>/`, poi elimina `app/storage/maintenance.flag` e `app/storage/update-state.json`.

### Per chi pubblica LibreCMS: firma dei pacchetti

```powershell
tools\php\php.exe bin\keygen.php --site=eucms
```

```powershell
tools\php\php.exe bin\build.php --site=eucms
```

`keygen.php` crea la chiave privata in `%USERPROFILE%\.librecms\release-signing.key`, fuori dal progetto. La chiave pubblica finisce in ogni pacchetto, e i siti installati accettano solo aggiornamenti firmati con quella chiave. **Conserva una copia offline della chiave privata.** Se la perdi, i siti già installati dovranno sostituire `app/config/release.pub` via FTP.

Ogni modifica al database va scritta come nuova migrazione numerata in `app/migrations/`, ripetibile e non distruttiva: aggiunge colonne o tabelle, non cancella dati.

## B. Dalla versione precedente a LibreCMS (via FTP, senza SSH)

Il pacchetto firmato del punto A non vale qui: la versione precedente accetta solo aggiornamenti firmati con la chiave del proprio sito. Il passaggio si fa quindi una sola volta, copiando i file.

### Prima di iniziare

1. In `/admin` → **Sistema**, premi **Crea backup** con i media e scaricalo.
2. Fai anche un backup del database dal pannello dell'hosting, ad esempio con phpMyAdmin.

### Copia dei file

3. Estrai `librecms.zip` sul PC e carica il suo contenuto via FTP in `public_html`, **sovrascrivendo i file esistenti**.
   - Il pacchetto non contiene `app/config/config.php`, `app/config/secret.key`, `app/config/installed.lock`, `app/storage/` né i media caricati: quei file restano come sono.
   - Il pacchetto sostituisce `app/config/release.pub` con la chiave pubblica di LibreCMS. Da quel momento il sito accetta gli aggiornamenti firmati di LibreCMS.
   - I file della vecchia versione che LibreCMS non usa possono restare: sono ignorati. Lascia in particolare `app/vendor/` se c'è, perché contiene mPDF e serve al modulo PDF.
4. L'installer resta bloccato, perché `installed.lock` è al suo posto.

### Migrazioni e controlli

5. Accedi a `/admin` con il tuo account abituale (password e 2FA restano quelli di prima).
6. **Sistema** mostra «migrazioni del database in sospeso». Premi **Applica migrazioni**: prima viene salvato un backup del database (`pre-migrazioni-…`). Le migrazioni:
   - riconoscono la **configurazione** dai contenuti: lavori del portfolio → Portfolio; articoli → Blog; solo episodi o brani → Podcast;
   - accendono i **moduli** che hanno già dati: tabelle dei moduli, episodi, embed, note, menu.
7. Apri **Moduli** e controlla l'elenco. **PDF** e **Protezione dalla copia** non lasciano tracce nei dati, quindi vanno riaccesi a mano se li usavi.
8. In **Aspetto** imposta logo, favicon e colori. Il nome del sito resta quello di prima.

### Cosa cambia e cosa no

- **Cambia l'aspetto pubblico.** Passa al tema di LibreCMS, che puoi personalizzare in **Aspetto**. I modelli grafici fatti su misura per il vecchio sito non fanno parte di LibreCMS.
- **Non cambiano:** indirizzi delle pagine e dei contenuti, redirect, file caricati, utenti, ruoli, password, 2FA, codici di recupero, impostazioni e tabelle dei moduli. Le migrazioni aggiungono solo colonne e impostazioni.

### Se qualcosa va storto

- Ricarica via FTP i file della versione precedente.
- In **Sistema → Ripristina**, scegli il backup `pre-migrazioni-…`, oppure importa il backup fatto dal pannello dell'hosting.

### Prova eseguita

Il passaggio è stato provato su **copie** dei dati di entrambe le versioni di partenza, con `tests/upgrade-eucms.php`. Le copie comprendevano database, configurazione, chiave di cifratura e media.

| Sito di partenza | Configurazione riconosciuta | Esito |
|---|---|---|
| gianlucafaella (Portfolio) | Portfolio | righe di ogni tabella, account, contenuti e pagine invariati; 29 URL della sitemap e 1 redirect con lo stesso esito; media raggiungibili |
| Cultura Sommersa (Blog con podcast) | Blog, con 9 moduli riaccesi | come sopra, con 14 URL e 30 redirect; migrazioni applicate dal pannello con backup; accesso con l'account esistente (password e 2FA) |

## Backup e integrità

- **Sistema → Crea backup** salva database e media in uno ZIP privato in `app/storage/backups/`. Scaricalo e conservane una copia fuori dal server.
- **Ripristina** sostituisce i contenuti dopo aver creato un backup di sicurezza. Utenti, password, 2FA e token non vengono toccati.
- **Integrità dei file** confronta il codice installato con il manifest della versione e segnala i file modificati o mancanti.
