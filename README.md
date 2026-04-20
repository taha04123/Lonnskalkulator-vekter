# Lønnskalkulator — Vekter

Ein personleg lønnskalkulator for vektarar tilsett under [Vekteroverenskomsten 2024–2026](https://arbeidsmandsforbundet.no/wp-content/uploads/2025/01/Vektere-overenskomsten-2024-2026.pdf). Kalkulatoren hentar vakter automatisk frå ein iCal-lenkje (t.d. Optima) og reknar ut brutto- og nettolønn basert på gjeldande tariffreglar.

**Live:** [taha04123.github.io/Lonnskalkulator-vekter](https://taha04123.github.io/Lonnskalkulator-vekter/)

---

## Funksjonar

- Automatisk henting av vakter via iCal-lenkje (Optima, Google Calendar o.l.)
- Alternativ: manuell opplasting av `.ics`-fil
- Reknar ut alle tillegg frå Vekteroverenskomsten §5.5:
  - Nattillegg (kl. 21–06): kr 28/t
  - Helgetillegg (lør kl. 18 – man kl. 06): kr 48/t
  - Heilagdagstillegg (100% av begynnerlønn): alle 7 tariffvindauge
  - Overtid over 41 t/veke: +50% kvardag, +100% helg/heilagdag
- Filtrerer ut ubetalte oppføringer (t.d. `UBETFERIE`)
- Estimert nettolønn basert på skatteprosent
- Månadsvis oversikt med detaljert vaktliste

---

## Tariffgrunnlag

Alle satsar er henta frå **Vekteroverenskomsten 2024–2026**, oppdatert per **1. april 2025** (mellomoppgjøret 2025, NHO/LO/YS).

### Grunnlønn (§5.1) — 35,5 t/veke, frå 1. april 2025

| Ansiennitet | Kr/time |
|---|---|
| 0–12 mnd | 248,99 |
| 1–3 år | 251,63 |
| 3–5 år | 253,96 |
| 5–7 år | 255,01 |
| 7–9 år | 256,91 |
| 9 år og over | 259,03 |

Vekterfagbrev gjev minst kr 9,50 ekstra per time.

### Tillegg for ubekvem arbeidstid (§5.5)

| Type | Tidsrom | Sats |
|---|---|---|
| Nattillegg | kl. 21–06 | +kr 28/t |
| Helgetillegg | lør kl. 18 – man kl. 06 | +kr 48/t |
| Heilagdagstillegg | sjå nedanfor | +100% av begynnerlønn |

### Heilagdagsvindauge (§5.5c)

| Heilagdag | Frå | Til |
|---|---|---|
| Påske | kl. 18 dagen før skjærtorsdag | kl. 06 tredje påskedag |
| 1. mai | kl. 21 dagen før | kl. 06 dagen etter |
| 17. mai | kl. 21 dagen før | kl. 06 dagen etter |
| Kristi himmelfartsdag | kl. 18 dagen før | kl. 06 dagen etter |
| Pinse | kl. 15 pinseaften | kl. 06 tredje pinsedag |
| Jul | kl. 15 julaften | kl. 06 tredje juledag |
| Nyttår | kl. 15 nyttårsaften | kl. 06 andre nyttårsdag |

### Overtid (§6)

Timar over **41 t/veke** (måndag–søndag) utløyser overtidsbetaling:

| Type | Tillegg |
|---|---|
| Kvardag | +50% av grunnlønn |
| Helg / heilagdag | +100% av grunnlønn |

---

## Teknisk oppsett

### Filer

```
Lonnskalkulator-vekter/
├── index.html       # Heile appen (éin fil, ingen dependencies)
└── README.md
```

### ICS-proxy (Cloudflare Worker)

Fordi nettlesaren blokkerer direkte henting av iCal-lenkjer frå andre domene (CORS), brukar appen ein eigen Cloudflare Worker som proxy:

**URL:** `https://ics-proxy.tahaahmed421.workers.dev/`

Worker-kode (`worker.js`):

```js
export default {
  async fetch(request) {
    const url = new URL(request.url);
    const target = url.searchParams.get('url');
    if (!target) return new Response('Missing url', { status: 400 });
    const res = await fetch(target, {
      headers: {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/120.0.0.0 Safari/537.36',
        'Accept': 'text/calendar, text/plain, */*',
      }
    });
    const text = await res.text();
    return new Response(text, {
      headers: {
        'Content-Type': 'text/calendar',
        'Access-Control-Allow-Origin': '*',
      },
    });
  },
};
```

Deploy gratis på [cloudflare.com/workers](https://workers.cloudflare.com/).

### GitHub Pages

Aktivert under **Settings → Pages → Branch: main / root**.

---

## Bruk

1. Opne kalkulatoren i nettlesaren
2. Lim inn iCal-lenkja frå Optima (eller last opp `.ics`-fil)
3. Juster satsar ved behov (grunnlønn, tillegg, skatteprosent)
4. Vel månad for å sjå vaktliste og lønnsoversikt

### Hente iCal-lenkje frå Optima

1. Logg inn på Optima-portalen
2. Finn kalenderintegrasjon / iCal-eksport
3. Kopier lenkja og lim inn i kalkulatoren

---

## Avgrensingar

- Skatteutrekning er ein **enkel estimat** basert på flat prosentsats — ikkje trekktabell
- Overtid blir rekna per veke (man–søn), ikkje per dag
- Heilagdagsutrekning er automatisk for kvart år (påskedato rekna ut algoritmisk)
- Kalkulatoren er laga for **35,5 t/veke turnus** som standard — juster grunnlønn for andre ordningar

---

## Kjelder

- [Vekteroverenskomsten 2024–2026 (NAF)](https://arbeidsmandsforbundet.no/wp-content/uploads/2025/01/Vektere-overenskomsten-2024-2026.pdf)
- [Tariffbrev 2025 — NHO Service og Handel](https://www.nhosh.no/contentassets/1b41a3810bb94fedafdc4f1bbd2251c9/tariffbrev-2025-vekter.pdf)
