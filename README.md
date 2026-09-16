# ONZE — Code Hunt

Aplicație web (browser, fără instalare) pentru lansarea magazinului ONZE din Promenada Mall: 5 modele expuse, fiecare cu un cod de 4 cifre. Vizitatorul introduce codul → primește 1 punct → la 5/5 primește un ecran de premiu → casierul îl validează la înmânarea premiului.

> Folderul și repo-ul se numesc încă `store-qr-hunt` din motive istorice. **Aplicația nu mai folosește QR-uri** — mecanica e cu coduri tastate.

## Fișiere

- `index.html` — aplicația completă (mobil, RO). Un singur fișier, fără dependențe de build.
- `code-sheet.html` — generator de foi A4 printabile, câte una per model, cu codul lui. Opțional, dacă nu faci designul foilor separat.
- `vercel.json` — config de hosting.

## Cum funcționează

1. Fiecare model are un cod de 4 cifre din lista `MODEL_CODES`. **Poziția din listă dă numărul modelului**: primul cod = Model 01, al doilea = Model 02 etc.
2. Vizitatorul apasă „Introdu codul", tastează cifrele. La a patra cifră aplicația verifică singură, fără încă un tap.
3. Codurile deja folosite sunt respinse („Ai introdus deja acest cod"), la fel cele inexistente.
4. La 5/5 → ecran premiu. **Dovada pentru casier e vizuală**: toate cele 5 pătrate bifate, 5/5, și un ceas care bate secundele — pe un screenshot ceasul e înghețat, se vede instant. Nu există cod de premiu.
5. Casierul ține apăsat „Validare casier" ~2 sec → ecranul devine „Premiu ridicat" definitiv. Din acel moment telefonul e blocat: rescanarea oricărui cod nu mai adaugă puncte.

Funcționează și un link direct de forma `https://SITE/#/c/4829`, dacă vrei să pui coduri scanabile pe viitor.

## Înainte de eveniment

1. În `index.html`, secțiunea `CONFIG` (aprox. linia 379):
   - `MODEL_CODES` — cele 5 coduri. Trebuie să fie identice cu cele de pe foile printate.
   - `TOTAL` — numărul de modele. Restul aplicației se adaptează singură.
   - `RESET_KEY` — **schimb-o înainte de eveniment** și nu o pune pe materiale printate.
2. Dacă folosești `code-sheet.html`, copiază acolo aceeași listă `MODEL_CODES`.
3. Publică folderul pe un hosting static (Vercel / Netlify / Cloudflare Pages). HTTPS obligatoriu.
4. Testează pe telefon cu un cod real înainte de deschidere.

Culorile se schimbă dintr-un singur loc: variabilele din `:root`. `--accent` e burgundy-ul închis pentru suprafețe pline, `--accent-hi` cel deschis pentru linii subțiri și text (pe fundal negru, burgundy-ul închis dispare la 3px).

## Progresul nu se pierde la refresh

Punctele se scriu **în trei locuri deodată**: `localStorage`, `sessionStorage` și un cookie. La încărcare aplicația le citește pe toate, reunește ce găsește și rescrie rezultatul în toate straturile — deci dacă browserul golește unul dintre ele, progresul se recuperează din celelalte și stratul lipsă se repară singur.

În plus, la citire codurile invalide sunt filtrate, iar „premiu ridicat" nu se poate anula golind un singur strat.

Dacă browserul blochează toate cele trei (foarte rar), aplicația afișează o atenționare și cere ca pagina să rămână deschisă până la premiu.

## Mai mulți vizitatori în același timp

Aplicația nu are server și nicio stare comună — sunt doar fișiere statice pe CDN. Fiecare telefon își ține punctele local, deci **fiecare vizitator are automat sesiunea lui**, complet independentă. Nu există limită de participanți simultani: 30, 100 sau 500 de oameni funcționează identic.

Limitarea e per telefon, nu per persoană: doi prieteni cu un singur telefon nu pot juca separat.

**Reset pentru personal:** `https://SITE/?reset=CHEIE`, unde cheia e `RESET_KEY` din `index.html`. Cheia dispare din bara de adrese imediat după folosire.

## Limite (varianta fără server)

- **Codurile sunt în JavaScript-ul paginii**, deci oricine se uită la sursă le poate citi pe toate fără să treacă prin mall. Asta e inevitabil la o aplicație fără backend. Pentru un eveniment de o zi protecția reală e fizică: premiul se validează la casă, de un om.
- Un cod trimis pe chat poate fi folosit fără a fi prezent în magazin.
- Blocarea „un premiu per telefon" ține cât timp vizitatorul nu șterge datele site-ului și nu trece în navigare privată. Browserele interzic intenționat identificarea unui dispozitiv, deci nicio soluție 100% client-side nu există.
- Fără cod de premiu, casierul se bazează pe ce vede: 5 pătrate bifate + ceasul care merge.

## Dacă vrei garanție reală (backend mic)

Se poate face fără costuri, pe planul gratuit de la Cloudflare Workers sau Netlify Functions:

- codurile stau pe server, nu în pagină — nu mai pot fi citite din sursă;
- fiecare telefon primește un token la prima deschidere, iar premiile ridicate se pot număra și verifica;
- pentru „un premiu per **persoană**", la 5/5 se cere numărul de telefon și serverul refuză un număr care a primit deja premiul — rezolvă problema, dar înseamnă date personale, deci are nevoie de o notă GDPR;
- bonus: statistici live în timpul evenimentului (câți participanți, câți au terminat, ce model se scanează cel mai des).
