# PVA4 — PHP 08: Formuláře

Obsahem repozitáře je cvičení k přednášce *Formuláře*. Navazuje na
`PHP_07_Funkce` — pokračujete ve fiktivní aplikaci **Maturita**,
portálu pro maturitní témata a konzultace.

Aplikace zatím jen ukazuje data. Dnes začne data od uživatele **přijímat**:
vyhledávání v historických tématech, kontaktní formulář pro vedoucího práce
a nahrání fotky do profilu. Každý vstup od uživatele je nedůvěryhodný, proto
ho načtete, ověříte a vypíšete přes funkce z knihovny `includes/formular.php`,
kterou si postupně napíšete podle přednášky.

## Struktura projektu

```
/includes
  config.php        ← Než začnete
  functions.php     ← funkce aplikace (z PHP_07), ÚKOL 3 a 8
  formular.php      ← knihovna pro formuláře (ÚKOL 1–7)
/templates
  header.php, footer.php, menu.php, metric-card.php   ← beze změny
/uploads
  .gitkeep          ← sem se ukládají fotky, ÚKOL 7
index.php           ← Přehled: ÚKOL 1–3, 8
aboutme.php         ← Profil: ÚKOL 1, 2, 7
contact.php         ← Kontakt: ÚKOL 1, 4–6
odpovedi.md         ← odpovědi na otázky z úkolů
```

## Jak postupovat

Zadání úkolů je níže v tomto README. V kódu najdete **značky**
`// --- ÚKOL N: … ---` (v PHP) a `<!-- ÚKOL N: … -->` (v HTML), které
ukazují, kam řešení patří. Když úkol hotový, značku smažte.

| Úkol | Soubor | Co děláte |
| --- | --- | --- |
| 1 | `formular.php`, všechny stránky | Knihovna: `nactiText()` a `escapuj()` |
| 2 | `formular.php`, `index.php`, `aboutme.php` | Bezpečné hodnoty z adresy: `nactiVolbu()` a `nactiCislo()` |
| 3 | `functions.php`, `index.php` | Vyhledávání v historických tématech (GET) |
| 4 | `formular.php`, `contact.php` | Kontaktní formulář: validace a chyby u polí (POST) |
| 5 | `formular.php`, `contact.php` | Po odeslání přesměrovat (Post/Redirect/Get) **[+ odpověď]** |
| 6 | `formular.php`, `contact.php` | Ochrana proti CSRF |
| 7 | `formular.php`, `aboutme.php`, `uploads/` | Fotka do profilu: bezpečné nahrání souboru **[+ odpověď]** |
| 8 | `formular.php`, `functions.php`, `index.php` | **Bonus:** filtr podle známek (checkboxy) |
| 9 | `odpovedi.md` | Pokus s podvrženou hodnotou **[odpověď]** |

Úkoly označené **[+ odpověď]** chtějí kromě kódu i krátkou odpověď do
souboru `odpovedi.md` pod nadpis daného úkolu.

**Názvy, parametry, typy funkcí i texty chybových hlášek dodržte přesně
podle zadání** — automatická kontrola funkce volá, posílá formuláře
a porovnává, co se vrátí a co se zobrazí.

## Pravidla pro knihovnu `formular.php`

Platí všechna pravidla pro funkce z PHP_07, navíc:

- Funkce dostanou data jako **parametr** (`$_GET`, `$_POST`, `$_FILES['foto']`),
  takže fungují pro GET i POST. Uvnitř funkce na `$_GET` ani `$_POST`
  nesahají (výjimkou je `$_SESSION` u CSRF tokenu).
- Funkce nic nevypisují, jen vrací hodnotu (výjimkou je `presmeruj()`,
  která pošle hlavičku a skončí).
- Vše, co přišlo od uživatele, se do HTML vypisuje **jen přes `escapuj()`**.
- Zpracování formuláře patří na **začátek** stránky, před
  `include … header.php`. Jinak nebude fungovat `session_start()`
  ani přesměrování.

## Než začnete

V `includes/config.php` přepište `yourName` na své jméno a verzi
aplikace zvyšte na `1.5`.

## Zadání

### ÚKOL 1 · Knihovna `formular.php` (`formular.php`, všechny stránky)

Soubor `includes/formular.php` je připravený, zatím prázdný. Napište do něj
první dvě funkce:

| Funkce | Vrací |
| --- | --- |
| `nactiText(array $data, string $klic): string` | hodnotu z pole `$data` pod klíčem `$klic` bez mezer na začátku a na konci (`trim`); chybí-li klíč, nebo místo textu přišlo pole, vrátí `''` |
| `escapuj(string $text): string` | text převedený přes `htmlspecialchars()` |

Na stránkách `index.php`, `aboutme.php` a `contact.php` knihovnu načtěte
pod načtením konfigurace:

```php
require_once __DIR__ . '/includes/formular.php';
```

**Očekávaný výsledek:** `nactiText(['jmeno' => '  Jan '], 'jmeno')` vrátí
`'Jan'`, `nactiText(['jmeno' => ['x']], 'jmeno')` vrátí `''` (útočník poslal
pole) a `nactiText([], 'jmeno')` také `''`.

### ÚKOL 2 · Bezpečné hodnoty z adresy (`formular.php`, `index.php`, `aboutme.php`)

Stránky už z minula čtou parametry z adresy (`?hour=`, `?days=`, `?status=`),
ale věří jim. Zkuste `index.php?days=abc` — aplikace tvrdí, že termín je
dnes, protože `(int) 'abc'` je `0`.

Napište dvě funkce:

| Funkce | Vrací |
| --- | --- |
| `nactiVolbu(array $data, string $klic, array $povolene): ?string` | hodnotu, je-li v seznamu `$povolene` (porovnání `in_array(…, true)`), jinak `null` |
| `nactiCislo(array $data, string $klic, int $min, int $max): ?int` | celé číslo v rozsahu `$min`–`$max` (přes `filter_var` a `FILTER_VALIDATE_INT`), jinak `null` |

Na stránkách nahraďte čtení z `$_GET`:

```php
// index.php
$hour = nactiCislo($_GET, 'hour', 0, 23) ?? (int) date('G');
$daysLeft = nactiCislo($_GET, 'days', -365, 365) ?? 12;

// aboutme.php
$status = nactiVolbu($_GET, 'status', ['approved', 'pending', 'rejected']) ?? 'pending';
```

**Očekávaný výstup:** `index.php?days=abc` → `Do odevzdání zbývá 12 dní.`,
`index.php?days=3` → `Do odevzdání zbývá 3 dny.`,
`aboutme.php?status=hacked` → stav `čeká na schválení`.

### ÚKOL 3 · Vyhledávání v historických tématech (`functions.php`, `index.php`)

Nad tabulkou *Historická témata* je připravený formulář s metodou GET:
textové pole `hledat` a výběr `rok` (Všechny roky, 2023, 2024, 2025).
Zatím nic nedělá.

1. Do `includes/functions.php` napište funkci
   `filterTopics(array $topics, string $search, int $year = 0): array`.
   Vrátí jen témata, jejichž název obsahuje `$search` bez ohledu na velikost
   písmen (`mb_stripos($topic['title'], $search) !== false`), a je-li
   `$year` různý od `0`, jen témata z daného roku. Prázdné hledání vrátí
   všechna témata. Témata projděte `foreach` a vyhovující přidávejte do
   nového pole: `$found[] = $topic;`.
2. V `index.php` načtěte hodnoty z formuláře a tabulku vypisujte
   z výsledku funkce:

   ```php
   $search = nactiText($_GET, 'hledat');
   $year = (int) (nactiVolbu($_GET, 'rok', ['2023', '2024', '2025']) ?? 0);
   $foundTopics = filterTopics($topics, $search, $year);
   ```

   Karty se statistikami počítejte dál ze všech témat (`$topics`).
3. Nad tabulku vypište `Nalezeno: 5` (počet nalezených témat) a je-li
   hledání vyplněné, i `Výsledky pro „…“` s hledaným textem.
4. Formulář si pamatuje zadání: do `value` pole `hledat` vypište hledaný
   text a vybranému roku přidejte atribut `selected`.

Hledaný text píše uživatel, do stránky ho proto vypisujte **jen přes
`escapuj()`** — v nadpisu výsledků i v atributu `value`.

**Očekávaný výstup:** `index.php?hledat=škol` → `Nalezeno: 5` (i „auto**škol**u“),
`index.php?rok=2024` → `Nalezeno: 3`, `index.php?hledat=ŠKOL&rok=2024` →
`Nalezeno: 2`. `index.php?hledat=<b>ahoj</b>` → na stránce je vidět text
`<b>ahoj</b>`, ne tučné `ahoj`.

### ÚKOL 4 · Kontaktní formulář (`formular.php`, `contact.php`)

Stránka *Kontakt* má připravený formulář s metodou POST: jméno (`jmeno`),
e-mail (`email`), typ dotazu (`typ`: `tema`, `konzultace`, `odevzdani`)
a zprávu (`zprava`). Pod každým polem je místo pro chybovou hlášku.
Formulář schválně nemá atribut `required` — prohlížeč ho tak pustí
i prázdný a uvidíte, jak data kontroluje server.

1. Do knihovny napište dvě kontrolní funkce, které vrací **text chyby**,
   nebo `null`, když je hodnota v pořádku:

   | Funkce | Chyby (přesně tento text) |
   | --- | --- |
   | `overText(string $text, string $nazev, int $maxDelka): ?string` | `Vyplňte pole jméno.` / `Pole jméno může mít nejvýš 50 znaků.` (za `jméno` se dosadí `$nazev`, délku počítejte `mb_strlen()`) |
   | `overEmail(string $email): ?string` | `Vyplňte e-mail.` / `E-mail nemá správný formát.` (`filter_var` s `FILTER_VALIDATE_EMAIL`) |

2. Na začátku `contact.php` formulář po odeslání zpracujte (jen když
   `$_SERVER['REQUEST_METHOD'] === 'POST'`). Hodnoty načtěte přes
   `nactiText()` a `nactiVolbu()`, chyby posbírejte pod klíči polí:

   ```php
   $chyby = array_filter([
       'jmeno' => overText($jmeno, 'jméno', 50),
       'email' => overEmail($email),
       'typ' => $typ === null ? 'Vyberte typ dotazu.' : null,
       'zprava' => overText($zprava, 'zpráva', 1000),
   ]);
   ```

3. Chybu vypište pod příslušné pole (`escapuj($chyby['jmeno'] ?? '')`)
   a vyplněné hodnoty vraťte do formuláře, ať je uživatel nemusí psát
   znovu. Vybraný typ dotazu dostane `selected`.

**Očekávaný výstup:** prázdný formulář → čtyři chyby
(`Vyplňte pole jméno.`, `Vyplňte e-mail.`, `Vyberte typ dotazu.`,
`Vyplňte pole zpráva.`). Jméno `<script>alert(1)</script>` se vrátí do pole
jako text a nic se nespustí.

### ÚKOL 5 · Po odeslání přesměrovat (`formular.php`, `contact.php`) [+ odpověď]

Když formulář nemá žádnou chybu, stránka nesmí odpověď vykreslit rovnou —
po F5 by prohlížeč formulář odeslal znovu.

1. Do knihovny napište `presmeruj(string $adresa): never`, která pošle
   hlavičku `Location` s kódem `303` a ukončí skript (`exit`).
2. Bez chyb zavolejte `presmeruj('contact.php?odeslano=1');`.
3. Je-li v adrese `odeslano`, ukažte nad formulářem zelenou zprávu
   `Děkujeme, dotaz jsme přijali.` (Skutečné odeslání e-mailem doplníme
   v cvičení k e-mailu.)

**Očekávaný výstup:** platný formulář → prohlížeč skončí na
`contact.php?odeslano=1` se zprávou `Děkujeme, dotaz jsme přijali.`

**Do `odpovedi.md`:** Dočasně zakomentujte `presmeruj(…)`, odešlete
platný formulář a stiskněte F5. Co prohlížeč udělá? Proč tomu přesměrování
zabrání? (Pak `presmeruj(…)` vraťte.)

### ÚKOL 6 · Ochrana proti CSRF (`formular.php`, `contact.php`)

1. Do knihovny napište dvě funkce:

   | Funkce | Vrací |
   | --- | --- |
   | `csrfToken(): string` | token uložený v `$_SESSION['csrfToken']`; když tam není, vytvoří ho přes `bin2hex(random_bytes(32))` |
   | `overCsrfToken(array $data): bool` | `true`, je-li v `$data['csrfToken']` řetězec shodný s tokenem (porovnání `hash_equals()`) |

2. Na úplný začátek zpracování v `contact.php` dejte `session_start();`.
3. Do formuláře přidejte skryté pole
   `<input type="hidden" name="csrfToken" value="…">` s hodnotou `csrfToken()`.
4. Při odeslání s neplatným tokenem odpovězte kódem `403` a skončete:

   ```php
   http_response_code(403);
   exit('Neplatný token. Načtěte stránku a odešlete formulář znovu.');
   ```

**Očekávaný výstup:** formulář odeslaný ze stránky funguje jako dřív.
Požadavek bez tokenu (např. `curl -d "jmeno=Jan" http://localhost:8000/contact.php`)
skončí textem `Neplatný token.`

### ÚKOL 7 · Fotka do profilu (`formular.php`, `aboutme.php`, `uploads/`) [+ odpověď]

V kartě profilu je připravený formulář pro nahrání fotky
(`enctype="multipart/form-data"`, pole `foto`).

1. Do knihovny napište tři funkce podle přednášky:

   | Funkce | Vrací |
   | --- | --- |
   | `priponaObrazku(string $cesta): ?string` | `jpg`, `png` nebo `webp` podle skutečného obsahu souboru (`mime_content_type()`), jinak `null` |
   | `overObrazek(?array $soubor, int $maxVelikost): ?string` | text chyby, nebo `null`: `Vyberte jeden soubor.` (chybí, nebo přišlo pole souborů), `Vyberte soubor.` (`UPLOAD_ERR_NO_FILE`), `Soubor se nepodařilo nahrát.` (jiná chyba), `Soubor je příliš velký.`, `Povolené jsou jen obrázky JPG, PNG a WebP.` |
   | `ulozObrazek(array $soubor, string $slozka): ?string` | soubor přesune (`move_uploaded_file()`) pod náhodným názvem `bin2hex(random_bytes(16))` + přípona z `priponaObrazku()` a vrátí nový název; když se přesun nepovede, `null` |

2. V `aboutme.php` (s `session_start()` a CSRF tokenem jako v úkolu 6)
   po odeslání soubor ověřte s limitem **2 MB** (`2 * 1024 * 1024`),
   uložte do složky `uploads/`, název si zapamatujte
   v `$_SESSION['foto']` a přesměrujte zpět na `aboutme.php`.
   Chybu vypište pod formulář.
3. Je-li v `$_SESSION['foto']` název, ukažte fotku:
   `<img src="uploads/…" alt="Fotka">` (název přes `escapuj()`).
4. Vytvořte soubor `uploads/.htaccess`, který ve složce zakáže spouštění
   PHP — druhá pojistka, kdyby kontrola selhala:

   ```apache
   <FilesMatch "\.(php|phtml|phar)$">
       Require all denied
   </FilesMatch>
   ```

**Očekávaný výstup:** obrázek PNG se nahraje a zobrazí v profilu, ve
složce `uploads/` je pod náhodným názvem s příponou `.png`. Soubor
`virus.php` přejmenovaný na `foto.jpg` skončí hláškou
`Povolené jsou jen obrázky JPG, PNG a WebP.`

**Do `odpovedi.md`:** Proč fotku neukládáte pod jménem
`$_FILES['foto']['name']`? Uveďte dva důvody.

### ÚKOL 8 · Bonus: filtr podle známek (`formular.php`, `functions.php`, `index.php`)

Do vyhledávacího formuláře přidejte pět checkboxů `znamka[]` s hodnotami
`1`–`5` (Výborně … Nedostatečně).

1. Do knihovny napište
   `nactiVolby(array $data, string $klic, array $povolene): array` — vrátí
   jen povolené hodnoty z pole; přijde-li místo pole text, vrátí `[]`.
2. Funkci `filterTopics()` rozšiřte o čtvrtý parametr
   `array $grades = []` (seznam známek jako řetězce `'1'`–`'5'`).
   Prázdný seznam znamená všechny známky, jinak projde jen téma, jehož
   známka v seznamu je: `in_array((string) $topic['grade'], $grades, true)`.
3. Zaškrtnuté checkboxy zůstanou po odeslání zaškrtnuté (`checked`).

**Očekávaný výstup:** `index.php?znamka[]=1&znamka[]=2` → `Nalezeno: 5`,
`index.php?znamka[]=1&rok=2024` → `Nalezeno: 1`.

### ÚKOL 9 · Podvržená hodnota [odpověď]

Otevřete stránku *Kontakt*, v nástrojích vývojáře (F12 → Elements) změňte
u výběru typu dotazu hodnotu `value` jedné možnosti na `hack`, vyberte ji
a formulář odešlete. Co formulář ukáže a která funkce z knihovny to
zařídila? Odpověď napište do `odpovedi.md`.

## Jak spustit stránku

**Přes FlyEnv:** nastavte tuto složku jako document root webu a otevřete
`http://localhost/index.php` (port si ověřte v nastavení FlyEnv).

**Bez nastavování** — vestavěný server PHP, ve složce s cvičením spusťte:

```sh
php -S localhost:8000
```

…a otevřete `http://localhost:8000/index.php`. Vestavěný server soubor
`.htaccess` nečte — pojistka z úkolu 7 se projeví až na Apache (FlyEnv).

## Konvence, které se od vás čekají

Stejné jako v předchozích cvičeních, navíc z přednášky o formulářích:

- **Funkce knihovny** česky podle přednášky (`nactiText`, `escapuj`),
  funkce aplikace v `functions.php` dál anglicky (`filterTopics`).
- **Klíče formulářů** česky malými písmeny: `jmeno`, `email`, `zprava`.
- **Každý výpis vstupu přes `escapuj()`**, hodnoty atributů v uvozovkách.
- **Data měnit jen metodou POST**, s CSRF tokenem a přesměrováním po úspěchu.

## Jak poznáte, že to máte správně

- Vyhledávání, kontaktní formulář i fotka fungují a formuláře si po chybě
  pamatují, co jste vyplnili.
- Ani podvržená hodnota (pole místo textu, hodnota mimo nabídku, soubor
  `.php`) nevyvolá `Warning`, `Fatal error` ani `TypeError`.
- Text `<b>ahoj</b>` zadaný kamkoli do formuláře se na stránce ukáže jako
  text, ne jako tučné písmo.

## Odevzdání

Práci odevzdáváte do svého repozitáře přes **Classroom 50**:

```sh
git add .
git commit -m "Cviceni 08 - vypracovano"
git push
```

Po každém pushi dostanete v pull requestu *Feedback* automatickou
zpětnou vazbu: které požadavky jsou splněné a nápovědu k těm, které ne.
