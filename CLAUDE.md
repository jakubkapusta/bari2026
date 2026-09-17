# Plan wycieczki do Bari — instrukcja edycji

Cały plan jest w jednym pliku `index.html` — dane, styl i logika renderowania.
Treść planu to blok tekstowy `PLAN_DATA` parsowany przez JS na sekcje, dni i punkty harmonogramu.

Wycieczka: Apulia, 17–20.10.2026 (sob–wt), Kuba, Asia i Michał. Wylot z Krakowa sob 5:50 (tylko bagaż podręczny), powrót z Bari wt 23:00.
Logistyka: auto z wypożyczalni na lotnisku BRI na cały pobyt (odbiór sob rano, zwrot wt wieczorem), jedna baza — 3 noce w Monopoli. Plan jest celowo intensywny; restauracje są ważne.
Szkielet: D1 Polignano + Monopoli · D2 Dolina Itrii (Alberobello, Locorotondo, Ostuni, kolacja w Cisternino) · D3 Altamura + Matera (kolacja w Materze, powrót po ciemku) · D4 Grotte di Castellana rano → Bari → lot.
Strefa czasowa w kodzie: `TZ='Europe/Rome'` (ta sama co Polska — nie ma dni „na zegarze polskim").

## Struktura sekcji

Sekcje rozdzielone nagłówkami `###`: `META`, `JEDZENIE`, `PLAN`, `CIEKAWOSTKI`. W UI są tylko dwie zakładki: **Plan** i **Jedzenie**.

### META

```
title: Bari 2026
subtitle: Kuba, Asia i Michał · 17.10 → 20.10 · 4 dni w Apulii
updated: wrzesień 2026
start: 2026-10-17        ← data DNIA 1 (nie ma dnia 0)
```

`start` steruje: odliczaniem w nagłówku, datami pogody, wschodem/zachodem słońca i trybem TERAZ (`dayDate(n)` = start + n−1).

## Jedzenie (sekcja JEDZENIE, zakładka Jedzenie)

Checklista „spróbowane" (stan w `localStorage['bari26_food']`).

```
GRUPA :: NAZWA | OPIS | OPCJONALNY_MAP_QUERY | OPCJONALNE_LOKALE
```

- `GRUPA` — nagłówek grupy. Wygląd bierze się z mapy `FOOD_MON` w JS: `'Street food':['bread','p-1']` — pierwszy element to klucz ikony z `FOOD_ICON` (inline SVG, kreska w `currentColor`), drugi to klasa koloru (`p-1` kobalt, `p-2` terakota, `p-3` oliwka). Nowa grupa bez wpisu dostanie ikonę `plate` i kobalt.
- **Nagłówki grup nie używają emoji** — w kolorowym kwadracie wyglądały źle. Wzór to: monoliniowa ikona, tytuł serifem, kropkowana linia jak w karcie menu i licznik po prawej.
- `OPCJONALNE_LOKALE` — lista `Nazwa @ map query`, rozdzielana `;;` → chipy z linkiem do Google Maps.

```
Street food :: Focaccia barese | Gruba, oliwna focaccia... |  | Panificio Fiore @ Panificio Fiore Bari
```

## Części i dni (sekcja PLAN)

```
# Część 1: Bari i okolice                                   ← nagłówek części (part)
== Dzień 1 | sob 17.10 | Przylot + Bari Vecchia | Nocleg: do ustalenia   ← nagłówek dnia
Opis dnia — wolny tekst, renderowany jako intro.
- 10:30 | 🏰 Castello Normanno-Svevo | Opis. | Castello Normanno-Svevo Bari
```

- Numer dnia (`Dzień N`) = kotwica `#dN`, klucz w `DAY_IMG`, `DAY_GEO`, `DAY_CITY` i w ciekawostkach (`DN`).
- 4. pole nagłówka dnia to nocleg → link do mapy. Wyjątki (zwykły tekst): zaczyna się od `Lot` (ikona ✈️) albo zawiera `do ustalenia`.
- Części: kolor i cyfra rzymska z kolejności (`# ` pierwsza = kobalt, druga = terakota, trzecia = oliwka). Linia `# ` jest wymagana (dni bez części nie trafiają do planu), ale **nagłówek części renderuje się tylko przy ≥2 częściach**. Obecnie jedna część, więc nagłówka nie widać.

Format punktu harmonogramu: `- CZAS | EMOJI TYTUŁ | OPIS | OPCJONALNY_MAP_QUERY`
- Map query (po ostatnim `|`) — trafia do linku Google Maps (`maps/search/?api=1&query=...`)
- Emoji na początku tytułu to ikona punktu. Punkt jedzeniowy (emoji z `FOOD_EMO`) bez map query dostaje automatyczny link „restauracje <miasto>" z `DAY_CITY`.

⚠️ **Segment `🕐` musi być OSTATNIĄ rzeczą w opisie.** `fmtNote()` owija wszystko od pierwszego `🕐` do końca opisu w `<span class="hrs">` („twarde ograniczenie czasowe"). Zdanie dopisane po godzinach zostanie pomalowane jak godziny otwarcia.

```
- 11:45 | ⛪ Bazylika | ... Ramiona i kolana zakryte. 🕐 7:00–20:30. | Basilica di San Nicola Bari   ✅
- 11:45 | ⛪ Bazylika | ... 🕐 7:00–20:30. Ramiona i kolana zakryte. | Basilica di San Nicola Bari   ❌
```

Test (konsola przeglądarki) — powinien zwrócić pustą tablicę:

```js
[...document.querySelectorAll('.sr')].filter(e=>{const h=e.querySelector('.hrs');
return h&&/\.\s+[A-ZŻŹĆĄŚĘŁÓŃ]/.test(h.textContent.replace(/^🕐\s*/,''))})
.map(e=>e.dataset.tm+' '+e.querySelector('.hrs').textContent)
```

### Notki dnia (`>`)

Opis dnia to **wyłącznie narracja: max 3 zdania o tym, czym jest ten dzień**. Wszystko inne — pogoda, bilety, plan B, akcje do wykonania — to osobne notki pod opisem, jedna notka = jeden wątek.

```
> 🎫 Bilety regionalne | Bez rezerwacji miejsc. Papierowy bilet skasujcie przed wejściem.
>! 🧳 Bagaż po wymeldowaniu | Dogadajcie późny check-out albo przechowanie bagażu.
```

Format: `> EMOJI ETYKIETA | treść`
- `>!` zamiast `>` = **akcja do wykonania** (nie informacja) — kreska w kolorze Adriatyku + delikatne tło
- Etykieta opcjonalna: bez ` | ` cała reszta linii jest treścią
- Emoji na początku = ikona notki; **emoji nie rozsypujemy po zdaniach** wewnątrz treści
- Segment `🕐` działa jak w punktach — musi być ostatni

**Zasady redakcyjne:**
1. Etykieta 1–4 słowa i ma nieść treść („Ostatni pociąg na lotnisko", nie samo „Uwaga").
2. Wątki-kontynuacje scalamy w jedną notkę zamiast ciągu `⚠️ … ⚠️ …`.
3. CAPS-y tylko tam, gdzie naprawdę „nie przegap". Jeśli krzyczy każdy segment, nie krzyczy żaden.
4. Notka dłuższa niż 2 zdania = sygnał, że szczegóły należą do punktu harmonogramu. W notce zostaje krótka informacja, a rozwinięcie idzie do punktu dnia („Szczegóły przy punkcie 20:15").

### Warianty dnia A/B (rozwidlenia)

Dni, które rozgrywa się na dwa sposoby (pogoda, siły, wybór celu), mają przełącznik. Wybór zapisuje się w `localStorage['bari26_variant']` (klucz = numer dnia), `'ALL'` = pokaż oba warianty naraz z badge'ami A/B.

Deklaracja w bloku dnia, **zaraz pod linią opisu dnia**:

```
? ⚖️ Wariant dnia — kiedy i na jakiej podstawie zapada decyzja.     ← nagłówek (linia bez „ | ")
? A* | 🏠 Alberobello | Krótki opis wariantu | Kiedy go wybrać
? B | 🪨 Matera | Krótki opis wariantu | Kiedy go wybrać
```

- `*` przy kluczu = wariant domyślny (bez gwiazdki domyślny jest pierwszy)
- Klucz to jedna wielka litera (`A`, `B`, …)

Punkt należący do wariantu dostaje prefiks przed czasem:

```
- A> 10:45 | 🏠 Rione Monti | Opis... | Rione Monti Alberobello
- B> 10:15 | 🪨 Sasso Barisano | Opis... | Sasso Barisano Matera
- 20:30 | 🍽️ Kolacja w Bari | Punkt bez prefiksu = wspólny dla obu wariantów
```

**Zasady pisania rozwidleń:**
1. Punkty **wspólne** (bez prefiksu) muszą mieć czas identyczny w obu wariantach. Jeśli po rozwidleniu godzina się rozjeżdża — punkt trzeba zduplikować do obu gałęzi.
2. W źródle pisz **całą gałąź A, potem całą gałąź B**. W obrębie jednej gałęzi czasy muszą rosnąć — dzięki temu po wybraniu wariantu oś czasu jest chronologiczna.
3. Każdy wariant musi mieć wypełnione pole „kiedy go wybrać" — realne kryterium decyzji w terenie.

Dni z wariantami: obecnie **brak** (z autem Alberobello i Matera mieszczą się w osobnych dniach). Mechanizm zostaje na rozwidlenia typu pogoda / siły.

⚠️ `DAY_GEO` i `DAY_CITY` mają jeden wpis na dzień i nie znają wariantów — wartości odpowiadają wariantowi domyślnemu.

## Ciekawostki (sekcja CIEKAWOSTKI)

Rozwijane detale przyczepiające się do punktów harmonogramu po dopasowaniu dnia i godziny.

```
DNUM | CZAS | IKONA | TYTUŁ | TREŚĆ
```

```
D1 | 11:45 | 🎭 | Mikołaj „wykradziony" z Myry | W 1087 r. żeglarze z Bari...
D3 | A>10:45 | ⚙️ | Kopuła bez zaprawy | Trulli stawia się z sucho układanego wapienia...
D2 | A>16:00; B>6:00 | 🎭 | ... | Ta sama ciekawostka na punkcie w obu wariantach (średnik rozdziela klucze).
```

- `DNUM` — numer dnia z prefiksem `D`
- `CZAS` — musi dokładnie zgadzać się z czasem punktu (z prefiksem wariantu, jeśli punkt go ma)
- Ikony: `💡` = praktyczna porada, `🎭` = historia/kultura, `📖` = legenda/opowieść, `⚙️` = inżynieria/architektura (Kuba jest inżynierem, syn w technikum — te wpisy mają mieć konkretne liczby i zasadę działania, a nie ogólniki „imponująca budowla")
- Treść — jeden ciągły akapit, bez łamania linii

⚠️ **Przed dodaniem ciekawostki sprawdź, co już wisi na tym punkcie** (duplikaty). Test w konsoli:

```js
(d,tm)=>D.tips.filter(t=>t.day===d&&(t.keys||[]).some(k=>k.tm===tm)).map(t=>t.icon+' '+t.ti)
```

Kilka ciekawostek na jednym punkcie jest OK — ale każda musi brać inny kąt.

**Po każdej zmianie harmonogramu** sprawdź, czy czasy ciekawostek nadal pasują do punktów. Test „osieroconych" ciekawostek — powinien zwrócić pustą tablicę:

```js
(()=>{const days={};D.parts.flatMap(p=>p.days).forEach(d=>{const m=d.lb.match(/\d+/);if(m)days[m[0]]=d});
const orphan=[];D.tips.forEach(t=>(t.keys||[]).forEach(k=>{const d=days[t.day];
if(d&&!d.sc.some(s=>s.tm===k.tm&&(s.v||'')===(k.v||'')))orphan.push('D'+t.day+' '+(k.v?k.v+'>':'')+k.tm+' :: '+t.ti)}));return orphan})()
```

## Tablice per dzień w JS

Przy dodaniu/usunięciu dnia albo zmianie jego bazy zaktualizuj:

- `DAY_IMG` — zdjęcie w nagłówku karty dnia: `{f:'img/...', t:'Tytuł'}`, opcjonalnie `pos` (CSS `object-position`, np. `'center 22%'` — gdy w wąskim kadrze ucieka najważniejsza część zdjęcia) oraz `by`, `lic`, `u` (autor/licencja/link) — gdy są, pokazują się w lightboxie i w „Źródła zdjęć".
  - Zdjęcia pochodzą z **Wikimedia Commons** (wolne licencje, zawsze z autorem i licencją w `DAY_IMG`). Szukanie: API Commons (`generator=search`, `gsrnamespace=6`; `incategory:Quality_images` daje najlepsze ujęcia), poziome, ≥1600 px szerokości.
  - Pobieramy miniaturę 1600 px (`iiurlwidth=1600`) i przepakowujemy: `sips -s format jpeg -s formatOptions 78 --resampleWidth 1600 plik.jpg --out plik.jpg` (docelowo ~200–500 KB — plik trafia do cache offline).
  - Nazwy plików od miejsca (`img/matera.jpg`), nie od numeru dnia — dni się przesuwają.
- `DAY_GEO` — `[lat, lon]` bazy dnia → pogoda i wschód/zachód słońca.
- `DAY_CITY` — miasto do automatycznego linku „restauracje <miasto>" przy punktach jedzeniowych.

## Pogoda godzinowa (pasek w nagłówku dnia + parasolki na osi)

Jedno zapytanie do Open-Meteo (`fetchWx()`, bez klucza, `timezone=Europe/Rome`) pobiera dane dzienne **i** godzinowe (`hourly=precipitation_probability,precipitation`) dla dni 1…`TRIP_DAYS`. Do `localStorage['bari26_wx']` trafia tylko wycinek **6:00–23:00** dla daty danego dnia: `hp` (18× szansa %) i `hm` (18× mm/h). Cache 3 h, offline pokazuje ostatni pobrany stan.

⚠️ **Zmieniasz kształt zapisu w cache → podbij `WX_V`.** `fetchWx()` przerywa, gdy cache jest młodszy niż 3 h, więc bez podbicia wersji przeglądarki ze świeżym zapisem w starym formacie czekają z odświeżeniem do wygaśnięcia TTL — i wygląda to tak, jakby zmiana w ogóle nie weszła. Niezgodna wersja wymusza pobranie od razu.

- `dayWxStrip(dnum)` — pasek pod nagłówkiem dnia. Wysokość słupka = szansa opadów, ciemniejszy rdzeń = mm/h (skala do 4 mm/h). Gdy max <20% **i** max <0,2 mm → „☀️ Bez opadów w prognozie".
- Pod paskiem linijka `.wxs-out` z odczytem (szczyt dnia albo wybrana godzina — dotyk/przeciągnięcie, hover na desktopie). Obsługa delegowana na `document`, przeżywa przerysowanie przez `renderPlanTab()`. `.wxs-bars` ma `touch-action:pan-y`.
- `wxMark(dnum, tm, nextTm)` — parasolka w kolumnie godziny: maksimum z okna do następnego punktu, przycięte do 3 h (brak następnego / czas cofnięty przy wariancie ALL → okno 2 h).
- Progi (`wxLevel`): <40% nic · 40–69% niebieskie · ≥70% albo ≥2 mm/h czerwone. Druga linijka z mm tylko od 0,5 mm/h.
- Horyzont prognozy Open-Meteo to 16 dni — wcześniej pasków po prostu nie ma. Oba elementy mają klasę `noprint`.
- Gdy pasek się renderuje, `dayMetaLine()` pomija dzienne `☔ X%` (byłoby zdublowane).

## Wygląd — „Maiolica"

Kierunek: apulijskie kafle ceramiczne. Kobalt + cytryna na wapiennej bieli, serif Cormorant Garamond (nagłówki, numery) + Jost (tekst).

- Tło strony `--bg` = azzurro `#e7eef7` (blady błękit; białe karty mają się od niego wyraźnie odcinać). Na nim chipy dni w nawigacji mają białe tło — jasnoniebieskie by się zlewały. Boksy notek i hover wierszy liczą szary odcień od `--card`, nie od `--bg`.
- Tokeny w `:root` (tryb ciemny nadpisuje je w `html.dark`): `--indigo` kobalt (godziny, linki, akcent główny), `--lemon` cytryna (obwódki kafelków, ornament), `--gold` ochra (etykiety, ciekawostki), `--red` terakota (tryb TERAZ, ulewa), `--teal` oliwka (godziny otwarcia `🕐`, odhaczone), `--tint` jasny błękit (pogoda, warianty, akcje), `--r` promień kart.
- Kolory części: `--p1` kobalt, `--p2` terakota, `--p3` oliwka (klasa `.p-N` ustawia `--pc`).
- `.tile` — kafelek z cytrynową obwódką: numer dnia, cyfra rzymska części, ikona grupy w Jedzeniu. Kolor tła z `--pc`.
- `.tiles` — pas kafli (wzór SVG w `--tile-img`) na górze nagłówka i pod stopką.
- Oś dnia: romby zamiast kropek, kropkowana linia. Pulsowanie w trybie TERAZ musi zachować `rotate(45deg)` w keyframes.
- **Unikamy motywów japońskich**, które zostały po poprzednim planie: okrągłe „pieczątki"-monogramy, kanji, ostre 2px narożniki, arkusze z liniami „keisen", koncentryczne fale w tle, pionowy tekst.
- Ikona zakładki Jedzenie to lód w rożku (miski/talerze z makaronem wyglądały jak ramen). Ikony zakładek: inline SVG 24×24, obrys `currentColor`.
- Ikona aplikacji `icon.svg` = kafel maioliki; PNG (`icon-192`, `icon-512`, `apple-touch-icon` 180) renderowane z niej headless Chrome + `sips`.

## Pozostałe funkcje

- **Tryb TERAZ** (`updateNowMode`) — w trakcie wycieczki karta dzisiejszego dnia dostaje cytrynową obwódkę (`.dc-today`) i znaczek `DZIŚ`, bieżący punkt cytrynowe tło i pulsujący romb, a w rogu pigułka „▶ TERAZ" z następnym punktem. Przy starcie strona przewija do bieżącego punktu (przed pierwszym punktem — do początku karty). Styl w kolorach Maioliki (cytryna + kobalt); czerwień/terakota zostaje tylko dla ostrzeżeń pogodowych.
- **Podgląd dowolnego momentu:** `index.html?now=2026-10-19T10:50` w adresie (albo `window.__testNow='…';renderPlanTab()` w konsoli).
- **Odliczanie** w nagłówku (`cdHead`); **3× klik w odliczanie** → easter egg z psem (`img/dog.png`).
- **Motyw** jasny/ciemny (`bari26_theme`), **druk/PDF** (rozwija ciekawostki na czas wydruku).
- Wszystkie klucze `localStorage` mają prefiks `bari26_` — strona żyje na tym samym originie (`github.io`) co plan Japonii (`jp26_*`), więc prefiksów nie wolno mieszać.

## Git i deploy

Repo `jakubkapusta/bari2026`, gałąź `master`, remote po SSH. GitHub Pages deployuje się przez GitHub Actions (`.github/workflows/pages.yml`) przy każdym pushu.

**Nie pushuj po każdej zmianie.** Commituj lokalnie, a push rób dopiero, gdy użytkownik o to poprosi — seria pushów throttluje deploy Pages.

## Service worker (`sw.js`) — cache offline

`sw.js` cache'uje statyczne pliki do trybu offline. Tablica `CORE` musi zawierać dokładnie te pliki, które istnieją — `cache.addAll()` w evencie `install` failuje w całości, jeśli choćby jeden URL zwróci 404.

`activate` usuwa tylko cache z prefiksem `bari2026-` — cache innych stron z tego samego originu zostają nietknięte. Nie zmieniaj tego na „usuń wszystko poza bieżącym".

**Zawsze gdy dodajesz/usuwasz/zmieniasz nazwę pliku w `img/` (albo innego pliku z `CORE`):**
1. Zaktualizuj listę `CORE` w `sw.js` tak, żeby 1:1 zgadzała się z plikami w repo i referencjami w `DAY_IMG` w `index.html`.
2. Podbij wersję `CACHE` (`bari2026-v1` → `v2`) — inaczej przeglądarki z zainstalowanym service workerem nie zobaczą zmiany.

Szybka weryfikacja przed commitem:
```bash
grep -oE "img/[^']+" sw.js | sort -u
ls img/ | sed 's|^|img/|' | sort
# obie listy powinny być identyczne
```

## Checklist po zmianie planu dnia

1. Zaktualizuj harmonogram dnia (punkty `- CZAS | ...`)
2. Sprawdź i popraw czasy ciekawostek (`DNUM | CZAS`) — usuń nieaktualne, dodaj nowe
3. Jeśli zmieniła się baza dnia — `DAY_GEO`, `DAY_CITY`, ewentualnie `DAY_IMG` (+ `sw.js`)
