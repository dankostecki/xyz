# Połączenie `xyz` (MARKET) i `feed` (NEWS) w jednej aplikacji na Vercelu

Cel: jedno repo (`feed`), jeden deploy na Vercelu, zero iframe'a między domenami.
Nic z obecnego działania nie jest zmieniane: terminal rynkowy to ten sam plik co `index.html`
(różni się **jedną linią**: adres ramki NEWS), a aplikacja feed nie jest ruszana.

## Co dokładnie się zmienia w repo `feed`

| Plik | Zmiana |
|---|---|
| `public/market.html` | **nowy** — kopia `index.html` z `xyz`; ramka NEWS ma `src="/"` zamiast `https://feed-olive-pi.vercel.app/` |
| `next.config.js` | dodany `rewrites()`: `/market` → `/market.html` |

Efekt po deployu:

- `https://<twój-projekt>.vercel.app/` — **bez zmian**, dokładnie ta sama aplikacja feed co teraz (stary adres i zakładki nadal działają).
- `https://<twój-projekt>.vercel.app/market` — pełny terminal: zakładki **MARKET** (Hyperliquid) i **NEWS** (feed w ramce, ten sam origin).
- `/api/rss` — bez zmian (`force-dynamic`, `no-store`), wszystkie 20 feedów, przycisk REFRESH i AUTO 60 s działają jak dotąd.

Feedy w `feed/src/app/api/rss/route.ts` (stan na dziś, nie ruszane): FED 5, ECB 4, NBP 1, REUTERS 3, BLOOMBERG 3, STOOQ 3, AXIOS 1 = **20**.

## Kroki na komputerze

```bash
# 1. kod terminala (ta gałąź, albo main po merge)
git clone https://github.com/dankostecki/xyz
git -C xyz checkout claude/tradfi-terminal-hyperliquid-NLwcQ

# 2. aplikacja feed
git clone https://github.com/dankostecki/feed
cd feed
git checkout -b add-market-tab

# 3. nałóż zmiany (2 pliki)
cp -r ../xyz/migration/feed-overlay/. .
git status          # oczekiwane: M next.config.js, ?? public/market.html
git diff            # next.config.js: tylko dodany blok rewrites()

# 4. sprawdź lokalnie
npm ci
npm run dev         # http://localhost:3000/market  oraz  http://localhost:3000/
```

Jeśli `next.config.js` w `feed` ma już inną zawartość niż `images: { unoptimized: true }`, nie kopiuj go —
dopisz tylko funkcję `rewrites()` z przykładu powyżej.

## Vercel

```bash
npm i -g vercel
vercel login
vercel link          # wskaż projekt feed (albo utwórz nowy z własnego konta)
git push -u origin add-market-tab
```

Jeśli projekt jest podpięty do GitHuba, push gałęzi utworzy **Preview deployment** — przetestuj na nim,
zanim zrobisz merge do `main`. Jeśli projekt jest na cudzym koncie i `vercel link` go nie widzi:
utwórz nowy projekt z własnego konta, wskaż repo `feed`, framework Next.js, bez żadnych zmiennych środowiskowych
(aplikacja ich nie używa). Nowy adres zastąpi `feed-olive-pi.vercel.app`.

## Lista kontrolna na Preview / produkcji (`/market`)

- [ ] Tabela MARKET: wszystkie 12 instrumentów ma cenę (Hyperliquid: NDX100, S&P 500, WTI, BRENT, GOLD, SILVER, NAT GAS, COPPER, EUR/USD, USD/JPY, BTC, ETH), CHG % się uzupełnia.
- [ ] Kliknięcie wiersza otwiera wykres, przełączanie interwałów 1m…1d działa, formatowanie miejsc po przecinku (EUR/USD 4).
- [ ] Przycisk SCREENSHOT: udostępnianie / pobieranie, a anulowanie okna nie pobiera pliku.
- [ ] Zakładka NEWS: ramka ładuje feed, wszystkie źródła mają wpisy (szczególnie **BLOOMBERG**, **REUTERS**, **STOOQ**, **AXIOS**).
- [ ] REFRESH i AUTO (60 s) dociągają nowe wpisy; kliknięcie w wpis z Google News otwiera wyszukiwanie po tytule.
- [ ] `/` (sam feed) działa jak wcześniej.

## Czego nie dało się sprawdzić w tej sesji

Środowisko sesji blokuje ruch do `api.hyperliquid.xyz` i do serwerów RSS (odpowiedzi 403), więc na żywo
sprawdzone zostały tylko: build (`next build` przechodzi), trasy `/`, `/market`, `/market.html` (200),
oraz w przeglądarce (desktop 1440 px i mobile 390 px) z podstawionymi odpowiedziami API: tabela MARKET,
przełączenie na NEWS, ramka pod tym samym originem, REFRESH ściągający nowy wpis, stary adres `/`.
Dwie rzeczy do potwierdzenia u siebie na Preview:

1. **Hyperliquid z domeny Vercela.** Dziś zapytania idą z `github.io`, więc to też ruch cross-origin
   i powinno działać identycznie, ale nie było jak tego zmierzyć.
2. **RSS z Vercela** — bez zmian względem obecnego stanu, ale po zmianie projektu/konta warto zerknąć,
   czy Bloomberg i Reuters nadal odpowiadają.

## Po migracji

- Kod terminala żyje wtedy w `feed/public/market.html`; **nie edytuj już `xyz/index.html`** (rozjadą się kopie).
- Stronę GitHub Pages z `xyz` można wyłączyć (Settings → Pages) albo zostawić jako zapasową — dalej działa,
  bo jej ramka NEWS nadal wskazuje na `feed-olive-pi.vercel.app`. Jeśli ten adres przestanie istnieć
  (nowy projekt Vercela), zmień `src` ramki w `xyz/index.html` na nowy adres.
- Opcjonalnie: terminal jako strona główna (`/`) zamiast `/market` — w `rewrites()` dodać `/` → `/market.html`
  i przenieść aplikację feed pod `/news` (wtedy zmienić `src` ramki na `/news`). Nie robiłem tego, żeby stary adres
  feedu działał bez zmian.
- Otwarte, poza zakresem tej migracji: zgłaszany wcześniej brak reakcji strzałek zmiany kolejności źródeł na mobile
  (w `feed`, `Terminal.tsx`) — nie sprawdzałem, czy nadal występuje.

## Gotowy prompt dla Claude Code na komputerze

> W repo `feed` wykonaj migrację z `../xyz/migration/README.md`: skopiuj `../xyz/migration/feed-overlay/`
> do katalogu głównego `feed` (nowy `public/market.html`, zmiana `next.config.js` tylko o `rewrites()`),
> uruchom `npm run build` i `npm run dev`, sprawdź `/` oraz `/market`, potem `vercel link` i deploy preview.
> Niczego innego nie zmieniaj: `src/`, `api/rss` i lista feedów mają zostać bez zmian.
