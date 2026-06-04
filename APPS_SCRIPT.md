# Prenotazione live — Google Apps Script (passo per passo)

Questo abilita il comportamento "il cliente prenota e le date si bloccano subito per tutti".
È gratis e non serve un server: usa un Foglio Google + un piccolo script.

## 1) Crea il Foglio Google
Nuovo foglio su Google Drive. Nella prima riga metti queste intestazioni (colonne A→J):

`Timestamp | Nome | Email | Telefono | Ospiti | Check-in | Check-out | Notti | Prezzo | Stato | Messaggio`

## 2) Aggiungi lo script
Nel foglio: menu **Estensioni → Apps Script**. Cancella tutto e incolla questo codice,
poi cambia l'email del proprietario in alto. Salva.

```javascript
const OWNER_EMAIL = 'tua-email@esempio.it'; // <- dove ricevere le prenotazioni

function doGet(e){ return json({ bookings: getBookings() }); }

function doPost(e){
  const p = e.parameter;
  if (p.action === 'book') {
    const ci = p.checkin, co = p.checkout;
    if (!ci || !co) return json({ ok:false, error:'missing' });
    if (overlaps(ci, co)) return json({ ok:false, error:'overlap' });
    const sh = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
    sh.appendRow([ new Date(), p.nome, p.email, p.telefono, p.ospiti, ci, co, p.notti, p.prezzo, 'In attesa', p.messaggio ]);
    try {
      MailApp.sendEmail(OWNER_EMAIL, 'Nuova pre-prenotazione — La Terrazza di Morro',
        'Nome: '+p.nome+'\nEmail: '+p.email+'\nTelefono: '+p.telefono+'\nOspiti: '+p.ospiti+
        '\nDal '+ci+' al '+co+' ('+p.notti+' notti)\nPrezzo: '+p.prezzo+' EUR all inclusive\n\nMessaggio:\n'+(p.messaggio||'—'));
    } catch(err){}
    return json({ ok:true });
  }
  return json({ ok:false, error:'unknown' });
}

function getBookings(){
  const sh = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
  const data = sh.getDataRange().getValues(); const out = [];
  for (let i=1; i<data.length; i++){
    const stato = (''+(data[i][9]||'')).toLowerCase();
    if (stato === 'annullata') continue;            // le righe "Annullata" liberano le date
    const ci = fmt(data[i][5]), co = fmt(data[i][6]);
    if (ci && co) out.push({ checkin: ci, checkout: co });
  }
  return out;
}
function overlaps(ci, co){
  const s = new Date(ci+'T00:00:00'), e = new Date(co+'T00:00:00');
  return getBookings().some(b => {
    const bs = new Date(b.checkin+'T00:00:00'), be = new Date(b.checkout+'T00:00:00');
    return s < be && bs < e;                         // si sovrappongono?
  });
}
function fmt(v){
  if (v instanceof Date) return Utilities.formatDate(v, Session.getScriptTimeZone(), 'yyyy-MM-dd');
  const s = (''+v).trim();
  if (s.includes('/')){ const p = s.split('/'); if (p.length===3) return p[2]+'-'+('0'+p[1]).slice(-2)+'-'+('0'+p[0]).slice(-2); }
  return s;
}
function json(o){ return ContentService.createTextOutput(JSON.stringify(o)).setMimeType(ContentService.MimeType.JSON); }
```

## 3) Pubblica come Web App
In Apps Script: **Esegui il deployment → Nuovo deployment → Tipo: App web**.
- Esegui come: **Me**
- Chi ha accesso: **Chiunque**
- Fai **Deploy**, autorizza, e **copia l'URL** che finisce con `/exec`.

## 4) Collega al sito
Apri `index.html`, cerca `const APPS_SCRIPT_URL = '';` e incolla l'URL tra gli apici.
Fatto: ora quando un cliente invia il form, la prenotazione viene scritta nel foglio,
ricevi l'email, e le date diventano subito occupate sul calendario (per tutti).

## Gestione prenotazioni
- Le righe nel foglio sono le prenotazioni. Per **liberare** delle date, scrivi `Annullata`
  nella colonna **Stato** (o cancella la riga).
- Quando confermi, puoi cambiare lo Stato in `Confermata` (resta comunque occupata).
- Se due persone provano lo stesso periodo, la seconda riceve "date appena prenotate" e deve sceglierne altre.

> Nota: senza `APPS_SCRIPT_URL` il modulo invia una semplice email via FormSubmit (nessun blocco automatico delle date).
