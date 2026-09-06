# Store Launch — QR Hunt

Aplicație web (browser, fără instalare) pentru lansarea magazinului: 10 modele în mall, fiecare cu QR-ul lui. Vizitatorul scanează → primește 1 punct live → la 10/10 primește un voucher pe ecran → casierul îl validează la înmânarea premiului.

Design: limbaj vizual Apple (tipografie SF, materiale de sistem, inel de progres tip Activity, animații spring, light + dark automat după setarea telefonului).

## Fișiere
- `index.html` — aplicația (mobil, RO). Scanner QR în pagină + suport pentru scanare cu camera nativă a telefonului.
- `qr-sheet.html` — generator + printare: cele 10 carduri cu QR, fiecare pe pagina lui A4.

## Logica
1. Fiecare model are un cod secret (lista `MODEL_CODES`, identică în ambele fișiere).
2. QR-ul modelului N conține `https://SITE/#/c/COD-N`. Merge și scanat din aplicație, și cu camera telefonului.
3. Punctele se țin în `localStorage` pe telefonul vizitatorului. Dublurile sunt respinse („Ai scanat deja acest model").
4. La 10/10 → ecran premiu. **Dovada pentru casier e vizuală: toate cele 10 pătrate bifate**, plus 10/10 și un ceas care bate secundele (pe un screenshot ceasul e înghețat — se vede instant). Nu există cod de premiu.
5. Casierul ține apăsat „Validare casier" ~2 sec → ecranul devine „Premiu ridicat" definitiv. Din acel moment telefonul e blocat: rescanarea oricărui QR nu mai adaugă puncte și nu există niciun buton care să repornească jocul.

## Un singur premiu per telefon
După ridicarea premiului, telefonul rămâne blocat pe „Premiu ridicat" — verificat: rescanarea codurilor nu face nimic, iar starea supraviețuiește refresh-ului și închiderii browserului.

Ce **nu** poate face o aplicație web: să recunoască telefonul dacă vizitatorul șterge datele site-ului, deschide navigare privată sau folosește alt browser. Browserele interzic intenționat identificarea unui dispozitiv (motive de confidențialitate), deci nicio soluție 100% client-side nu există — asta e valabil pentru orice aplicație web, nu doar asta.

În practică bariera nu e tehnică, ci fizică: ca să ajungă din nou la 10/10, omul trebuie să refacă tot traseul și să scaneze din nou toate cele 10 modele, prin mall, în fața personalului. Ștergerea datelor durează 15 secunde, traseul durează 10+ minute și se vede.

**Reset pentru personal:** dacă chiar e nevoie să se reseteze un telefon (ex. doi prieteni cu un singur telefon), se deschide `.../?reset=CHEIE` — cheia e `RESET_KEY` din `index.html`. Schimb-o înainte de eveniment și nu o pune pe materiale printate. Cheia dispare din bara de adrese imediat după folosire.

Dacă vrei garanție reală (un singur premiu per persoană, nu per telefon), singura variantă care chiar funcționează e un backend mic care ține evidența — se poate face gratuit; detalii la finalul fișierului.

## 30 de oameni în același timp
Aplicația nu are server și nicio stare comună — sunt doar fișiere statice. Fiecare telefon își ține punctele local, în browserul lui, deci **fiecare vizitator are automat sesiunea lui**, complet independentă. Nu există limită de participanți simultani: 30, 100 sau 500 de oameni funcționează la fel (testat: 30 de cereri simultane, toate sub 70 ms pe serverul local de test; pe un hosting real e CDN, deci și mai simplu).

Singurul caz în care progresul nu se salvează e **navigarea privată** (Safari Private / Chrome Incognito), unde browserul blochează stocarea locală. Aplicația detectează asta și afișează o atenționare: în acel caz vizitatorul trebuie să scaneze din butonul aplicației (pagina rămâne deschisă și punctele se țin în memorie), nu cu camera telefonului.

---

## Test de pe telefon, prin WiFi-ul de acasă

Ambele servere rulează deja pe Mac. Telefonul trebuie să fie pe **aceeași rețea WiFi**.

**Pentru test complet, cu camera din aplicație — folosește HTTPS:**

```
https://192.168.1.129:8443
```

Safari va spune că certificatul nu e de încredere (e normal, e un certificat făcut local): apasă **Show Details → visit this website → Visit Website**. Camera funcționează doar pe HTTPS — pe `http://` browserele o blochează, indiferent de aplicație.

**Varianta HTTP** (merge tot, mai puțin butonul de scanare din aplicație):

```
http://192.168.1.129:8123
```

Simulare scan fără cameră (pui linkul direct în browser-ul telefonului):
`https://192.168.1.129:8443/#/c/LX1-K7QP` … până la `LX10-N0RB`.
Reset progres: adaugă `?reset=1` → `https://192.168.1.129:8443/?reset=1`.

### Dacă serverele nu mai rulează (după restart Mac / închiderea sesiunii)

```bash
cd "/Users/danilgherbali/Documents/local animations" && npx --yes http-server store-qr-hunt -p 8123 -c-1
```

Pentru HTTPS trebuie și certificatul (se regenerează în 2 secunde dacă a fost șters):

```bash
mkdir -p ~/.qr-hunt-certs && openssl req -x509 -nodes -newkey rsa:2048 -days 825 -keyout ~/.qr-hunt-certs/key.pem -out ~/.qr-hunt-certs/cert.pem -subj "/CN=192.168.1.129" -addext "subjectAltName=IP:192.168.1.129,IP:127.0.0.1,DNS:localhost" && cd "/Users/danilgherbali/Documents/local animations" && npx --yes http-server store-qr-hunt -p 8443 -c-1 -S -C ~/.qr-hunt-certs/cert.pem -K ~/.qr-hunt-certs/key.pem
```

IP-ul Mac-ului se poate schimba la reconectare. Îl afli cu:

```bash
ipconfig getifaddr en0
```

---

## Înainte de eveniment
1. În `index.html` și `qr-sheet.html`: schimbă `BRAND_NAME` și, dacă vrei, regenerează `MODEL_CODES`.
2. Publică folderul pe un hosting static gratuit — **HTTPS obligatoriu**:
   - Netlify Drop (drag & drop folderul pe https://app.netlify.com/drop), sau
   - GitHub Pages / Cloudflare Pages / Vercel.
3. Deschide `https://SITE/qr-sheet.html`, pune adresa finală, Generează → Printează (hârtie mată).
4. Testează pe telefon cu un QR printat.

Culoarea de accent se schimbă dintr-un singur loc: variabilele `--accent` / `--accent-2` din `:root`.

## Limite (varianta fără server)
- Punctele sunt pe telefon (localStorage): dacă vizitatorul șterge datele browserului sau schimbă telefonul, progresul se pierde.
- Un link de QR trimis pe chat poate fi deschis fără a fi fizic acolo — pentru un eveniment de o zi e un risc acceptat; protecțiile reale sunt codurile secrete (nu pot fi ghicite) + validarea fizică la casă.
- Fără cod de premiu, casierul se bazează pe ce vede: 10 pătrate bifate + ceasul care merge.
- Blocarea „un premiu per telefon" ține cât timp vizitatorul nu șterge datele site-ului / nu trece în navigare privată — vezi secțiunea de mai sus.

## Dacă vrei garanție reală (backend mic)
Se poate face fără costuri, pe planul gratuit de la Cloudflare Workers sau Netlify Functions:
- fiecare telefon primește la prima deschidere un token aleator, iar completarea jocului se înregistrează pe server → premiile ridicate se pot număra și verifica;
- pentru „un premiu per **persoană**" (mai puternic decât per telefon), la 10/10 se cere numărul de telefon și serverul refuză un număr care a primit deja premiul — asta chiar rezolvă problema, dar înseamnă date personale, deci are nevoie de o notă GDPR;
- bonus: statistici live în timpul evenimentului (câți participanți, câți au terminat, ce model se scanează cel mai des).

Spune-mi dacă vrei să-l construiesc.
