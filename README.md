# La Terrazza di Morro — Sito

Sito statico (una sola pagina) per la casa vacanze. Tutto è già pronto: HTML, foto, logo, favicon.

## Da personalizzare prima di pubblicare
Apri `index.html` e sostituisci:
1. **Numero WhatsApp/telefono** — cerca `393XXXXXXXXX` (formato senza + né spazi, es. `393331234567`) e `+393XXXXXXXXX`.
2. **Email** — cerca `info@casavacanzespoleto.it` e metti la tua (è quella del modulo FormSubmit; la prima richiesta inviata ti farà ricevere un'email di attivazione, da confermare una sola volta).
3. **Date occupate (opzionale)** — nella parte JavaScript trovi `const SHEET_URL = '';`. Per gestire la disponibilità da un foglio:
   crea un Google Sheet con 2 colonne `Check-in | Check-out` (gg/mm/aaaa), poi *File → Condividi → Pubblica sul web → CSV*, copia il link e incollalo tra gli apici. Vuoto = tutte le date disponibili.

## Pubblicare su GitHub + Cloudflare Pages
1. Crea un nuovo repository su GitHub e carica TUTTI i file di questa cartella (index.html + foto + logo + favicon + og-image).
2. Vai su Cloudflare → **Workers & Pages** → **Create** → **Pages** → **Connect to Git**.
3. Seleziona il repository.
4. Build settings: **Framework preset = None**, lascia vuoti "Build command" e "Build output directory" (è già statico).
5. Deploy. Il sito sarà online su un indirizzo `*.pages.dev`; poi puoi collegare un dominio tuo.

## File inclusi
- `index.html` — la pagina
- `foto-hero.jpg`, `foto1.jpg` … `foto6.jpg` — le foto
- `logo.png`, `logo-light.png` — logo (versione marrone e versione chiara)
- `favicon.svg` — icona del browser
- `og-image.jpg` — anteprima per condivisioni su WhatsApp/Facebook

## SEO — da fare dopo la pubblicazione
Quando avrai il dominio definitivo su Cloudflare, sostituisci `https://www.laterrazzadimorro.it` con il tuo dominio in 3 punti:
- `index.html` → riga `<link rel="canonical" ...>`
- `sitemap.xml` → tag `<loc>`
- `robots.txt` → riga `Sitemap:`

Poi vai su **Google Search Console**, aggiungi il sito e invia la `sitemap.xml`: serve a far indicizzare la pagina più in fretta.

Già incluso per la SEO: title e meta description, dati strutturati JSON-LD (VacationRental), Open Graph + Twitter card con immagine, meta geografici, robots.txt e sitemap.xml.
