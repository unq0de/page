# jbre.dev — Style Guide

Referenzdokument für die Weiterentwicklung von jbre.dev. Ziel: Jede neue Section,
jedes neue Projekt oder jede Anpassung soll sich anfühlen, als wäre sie von Anfang
an Teil der Seite gewesen — nicht wie ein nachträglich draufgeklebtes Feature.

Diese Datei ist bewusst technisch/konkret gehalten, damit sie als Kontext für
zukünftige Claude-Chats funktioniert: Design-Tokens, Komponenten-Patterns und
Content-Regeln, keine Marketing-Prosa.

---

## 1. Konzept

**"Developer Terminal" — eine Portfolio-Seite, die aussieht, als würde sie in
einem Code-Editor / einer Shell laufen.**

Das zieht sich durch die ganze Seite:
- Section-Labels sehen aus wie Code-Kommentare: `// stack`, `// projects`, `// about`
- Die Hero-Section ist ein simuliertes Terminal-Fenster (`me.sh`) mit Tippanimation
- Projekte werden wie Git-Commits dargestellt: Hash + Jahr (`#a3f9c1d 2026`)
- Der About-Abschnitt ist als README-Datei im Code-Editor-Stil aufgebaut (`about.md`)
- Der Kontaktbereich sieht aus wie ein Terminal-Befehl (`$ mailto contact@jbre.dev`)
- Monospace-Schrift durchgängig, auch im Fließtext — keine Antiqua/Grotesk-Mischung

**Faustregel für neue Inhalte:** Wenn ein Feature nicht plausibel als Datei,
Kommandozeile, Commit oder Code-Kommentar dargestellt werden könnte, passt es
wahrscheinlich nicht zum Konzept.

---

## 2. Design-Tokens

Alle Werte als CSS Custom Properties auf `:root`, mit Light-Theme-Override via
`html[data-theme="light"]`.

### Farben — Dark (Standard)
```css
--bg:            #0D0F12   /* Seitenhintergrund */
--bg-alt:        #15181D   /* Cards, Fenster, Sections */
--bg-raised:     #1B1F26   /* Erhöhte Elemente: Nav-Buttons, Tags, Tabs */
--border:        #262B33   /* Standard-Rahmen */
--border-soft:   #1E222A   /* Dezente Trenner (Section-Divider, Footer) */
--text:          #E7E9EC   /* Haupttext */
--text-muted:    #99A1AE   /* Sekundärtext, Beschreibungen */
--accent:        #E8A33D   /* Orange/Amber — primäre Akzentfarbe */
--accent-contrast: #0D0F12 /* Text auf Accent-Flächen */
--accent-blue:   #5FA8D3   /* Sekundärakzent — Links, Keys, aktive Nav */
--accent-rose:   #D97AA0   /* Tertiärakzent — Commit-Hashes, Arrays */
```

### Farben — Light
```css
--bg:            #F1F3F1
--bg-alt:        #FFFFFF
--bg-raised:     #E7EAE7
--border:        #D7DBD7
--border-soft:   #E2E5E2
--text:          #14171A
--text-muted:    #545E5A
--accent:        #9A5F12
--accent-contrast: #FFFFFF
--accent-blue:   #2A5F91
--accent-rose:   #9C3D63
```

**Regel:** Nie neue Rohfarben einführen. Jede neue Komponente nutzt ausschließlich
diese Tokens. Wenn eine vierte Akzentfarbe nötig scheint — lieber eine der drei
bestehenden in anderer Opazität/Kontext verwenden, als eine neue zu erfinden.

### Typografie
```css
--font-head: 'JetBrains Mono', ui-monospace, monospace;   /* Headings, Labels, UI-Chrome */
--font-body: 'IBM Plex Mono', ui-monospace, monospace;    /* Fließtext */
```
- Google Fonts, per `<link>` eingebunden, Weights 400–800 (Head) bzw. 400–600 (Body)
- Basis-Schriftgröße Body: `15.5px`, Zeilenhöhe `1.65`
- Überschriften: `font-weight: 700`, `letter-spacing: -0.01em`
- Responsive Heading-Größen via `clamp()`, z. B. Hero-H1: `clamp(28px, 4.6vw, 44px)`

### Sonstige Tokens
```css
--radius: 6px;      /* einheitlicher Border-Radius, überall */
--maxw:   1080px;   /* max-width für .wrap-Container */
```

### Hintergrund-Textur
Ein fixes, sehr dezentes Dot-Grid über der gesamten Seite:
```css
body::before {
  background-image: radial-gradient(circle, rgba(127,133,143,0.10) 1px, transparent 1px);
  background-size: 26px 26px;
}
```
Das bleibt in beiden Themes gleich (nur Rendering über dem jeweiligen `--bg`).

---

## 3. Layout-Grundgerüst

- Alles läuft über einen `.wrap`-Container: `max-width: var(--maxw)`, `padding: 0 24px`, zentriert
- Sections: `.section` mit `padding: 84px 0` und `border-top: 1px solid var(--border-soft)` als Trenner zur vorherigen Section
- Jede Section beginnt mit `<p class="eyebrow">// section-name</p>` gefolgt von `<h2>`
- Mobile Breakpoint bei `680px` (Nav → Hamburger) und `640px` (generelles Layout, reduziertes Padding)
- Ein Extra-Breakpoint bei `380px` für sehr kleine Screens (kleineres Terminal-Fenster etc.)

**Section-Reihenfolge (fix):** Hero → Stack → Projects → About → Contact → Footer

---

## 4. Komponenten-Patterns

### 4.1 Nav
- Sticky, `backdrop-filter: blur(10px)`, halbtransparenter Hintergrund passend zum Theme
- Logo: `~/Jbre`, wobei `Jbre` in `--accent` eingefärbt ist
- Aktiver Nav-Link wird per IntersectionObserver auf die sichtbare Section gesetzt,
  darunter läuft ein animierter `.nav-underline`-Strich mit
- Unter der Nav: ein 2px `.progress-track` mit `.progress-fill`, der den Scroll-Fortschritt der Seite anzeigt
- Mobile: Hamburger-Button, Vollbild-Dropdown-Panel, Body-Scroll wird bei offenem Menü gesperrt

### 4.2 Hero — Terminal-Fenster
- macOS-artiges Fenster: `.window` mit `.window-bar` (drei Dots: rot/gelb/grün) + Titel `me.sh`
- Beim Laden: Tippanimation eines Befehls (`whoami --as-json`), dann zeilenweises
  "Ausgeben" einer JSON-Antwort mit farbcodierten Keys/Strings/Arrays
  (`--accent-blue` für Keys, `--accent` für Strings, `--accent-rose` für Array-Werte)
- Cursor blinkt (`▌`), wird bei `prefers-reduced-motion: reduce` ausgeblendet
- Radialer "Spotlight"-Hover-Effekt im Fenster folgt der Maus (nur bei `pointer: fine`)
- Darunter: `.hero-thesis` mit H1 + Sub-Satz, faded erst nach Abschluss der Typing-Animation ein
- Status-Pill mit pulsierendem grünen Dot ("Open to new projects")
- Zwei CTAs: `.btn-primary` (Accent-gefüllt) + `.btn-ghost` (Outline)

### 4.3 Tags
- `.tag`: kleine Pills, `--bg-raised`-Hintergrund, `--border`-Rahmen, Monospace
- Interaktive Tags (im Stack-Bereich) sind `<button>`-Elemente mit `data-tech="..."`
- Statische Tags (in Projekt-Cards) sind `<span>`-Elemente mit demselben `data-tech`-Wert
- Spezialvariante `.tag-learning`: gestrichelter Rahmen, `--accent-blue`-Text — für "aktuell am Lernen"

### 4.4 Stack-Section ("Tree")
- Grid aus `.tree-group`-Karten, 2 Spalten Desktop / 1 Spalte Mobile
- Jede Gruppe: Ordner-Titel im Stil `mobile/`, `frontend/` etc. (Slash separat in `--text-muted`)
- Darunter Tags, darunter ein `.tree-note` als Code-Kommentar (`// native feel, cross-platform speed`)
- Letzte Gruppe ("currently exploring") spannt beide Spalten (`.span-2`)
- Klick auf einen Tag scrollt zu passenden Projekten und hebt sie kurz hervor (siehe 5.3)

### 4.5 Projects ("Commits")
- Jedes Projekt ist eine `.commit-card`:
  - `.commit-meta`: Hash (`#a3f9c1d`, `--accent-rose`) + Jahr
  - `<h3>`: `Projektname — kurze Kategorie` (z. B. "— cross-platform app")
  - Kurzbeschreibung (2–3 Sätze max.)
  - Ausklappbare `.commit-more` mit mehr Detail (Rolle, technische Herausforderung),
    über `+ details` / `– hide`-Button, `max-height`-Transition
  - Tags (gleiches `.tag`-System wie Stack)
  - Links: `Code →`, `Live →` bzw. `Docs →`
- Hover: Karte hebt sich leicht (`translateY(-2px)`), Rahmen wird `--accent-blue`
- `.highlight`-Klasse für den kurzen Pulse-Effekt beim Tag-Klick aus der Stack-Section

**Hash-Format:** 7-stelliger Hex-String, wirkt wie ein echter Git-Short-Hash.
Für neue Projekte einfach einen neuen zufälligen 7-Zeichen-Hex-Wert nehmen — muss
nicht real sein, aber im Format bleiben (`[a-f0-9]{7}`).

### 4.6 About ("README.md")
- `.readme`-Block sieht aus wie eine geöffnete Datei im Editor:
  - `.readme-tab`: Dateiname-Reiter (`about.md`)
  - `.readme-body`: Markdown-artig gerendert — `## Überschrift`-Zeilen als `.md-heading`
    (in `--accent-blue`), Absätze in `--text-muted`, `<strong>` in `--text`
  - Listen (`.md-list`) mit Gedankenstrich (`–`) statt Standard-Bullets, in `--accent` eingefärbt
  - `.file-meta`-Footer: "Last edited [Monat Jahr] · ~1 min read" (wird per JS live gesetzt)
- Feste Struktur der Inhalte: `## who I am` → `## how I work` → `## currently` → `## off the clock`
  (letzterer Abschnitt: ein einzelner, leicht selbstironischer Satz)

### 4.7 Contact
- `.contact-code`: Terminal-Zeile im Stil `$ mailto contact@jbre.dev` mit Copy-Button
- Copy-Button hat drei Zustände: default / hover / `.copied` (kurzzeitig grün-ish via `--accent`)
- Zwei CTAs analog zum Hero: `Send an email` (primary) + `GitHub →` (ghost)

### 4.8 Footer
- Minimal: Copyright links, rotierender Status-Text rechts
  (`Open to new projects`, `Currently shipping a Flutter app`, `Runs on coffee and curiosity`, `Always up for talking APIs`)
- Rotation alle 4,2s mit Fade, deaktiviert bei `prefers-reduced-motion`

---

## 5. Interaktions- & Motion-Regeln

1. **Scroll-Reveal:** Fast alle Sections/Elemente tragen `.reveal`, werden per
   `IntersectionObserver` beim Eintritt in den Viewport auf `.in-view` gesetzt
   (Fade + `translateY(18px) → 0`, `0.55s ease`). Threshold: `0.15`.
2. **Reduced Motion:** Jede Animation hat einen `prefers-reduced-motion: reduce`-Fallback.
   Globale Regel setzt Animationsdauern auf ~0. Zusätzlich werden Typing-Effekt,
   Footer-Rotation und Spotlight-Hover bei reduzierter Bewegung komplett deaktiviert
   bzw. sofort im Endzustand gerendert. **Das ist Pflicht bei jeder neuen Animation.**
3. **Theme-Toggle:** Speichert Wahl in `localStorage` (`jbre-theme`), respektiert sonst
   `prefers-color-scheme`. Icon-Wechsel Mond/Sonne.
4. **Stack ↔ Projects Verlinkung:** Klick auf einen Stack-Tag sucht Projekt-Cards mit
   gleichem `data-tech`, hebt sie kurz hervor und scrollt hin. Gibt es keinen Treffer,
   scrollt es einfach zur Projects-Section. Dieses Pattern sollte bei neuen Tags/Projekten
   automatisch weiterfunktionieren, solange `data-tech`-Werte exakt übereinstimmen
   (lowercase, wie in bestehenden Beispielen: `"react native"`, `"fastapi"` etc.)
5. **Aktive Nav-Section:** IntersectionObserver mit `rootMargin: '-45% 0px -50% 0px'`
   bestimmt, welche Section "aktiv" ist — nicht Scroll-Position der Seite.

---

## 6. Content- & Ton-Regeln

- **Sprache der Seite: Englisch.** (Unabhängig davon, dass wir hier Deutsch sprechen.)
- Kurze, direkte Sätze. Keine Buzzword-Kaskaden ("synergistic", "cutting-edge", "passionate about").
- Selbstironie in Maßen ist ok (`off the clock`-Abschnitt), aber sparsam — die Seite
  soll kompetent wirken, nicht wie ein Comedy-Bit.
- Projektbeschreibungen: **2–3 Sätze Kurzversion + optionale ausklappbare Detail-Version.**
  Kurzversion beantwortet: Was macht es, für wen, welche Rolle hattest du.
  Detailversion: ein konkretes technisches Detail oder eine Herausforderung — kein Recap.
- Stack-Kommentare (`.tree-note`) sind kurze, leicht pointierte Ein-Zeiler
  ("APIs that scale without drama"), kein vollständiger Satz mit Punkt.
- Keine Emojis im Seiteninhalt (im Gegensatz zu GitHub-Bio/README, wo das ok ist).

---

## 7. Technischer Aufbau

- **Eine einzige statische HTML-Datei** — Inline `<style>` im `<head>`, Inline `<script>`
  am Seitenende. Kein Build-Step, kein Framework, kein Bundler.
- Vanilla JS, keine externen JS-Libraries.
- Fonts über Google Fonts CDN (`JetBrains Mono`, `IBM Plex Mono`).
- Keine externen CSS-Frameworks (kein Tailwind trotz Erwähnung im Stack-Bereich —
  das bezieht sich auf Skills/Projekte, nicht auf jbre.dev selbst).
- Barrierefreiheit: `:focus-visible`-Outlines, `aria-expanded`/`aria-controls` an
  interaktiven Elementen, `prefers-reduced-motion` überall berücksichtigt.

**Wichtig für zukünftige Änderungen:** Neue Features sollten sich in dieses
Single-File-Setup einfügen — kein Wechsel zu einem Framework oder Split in
mehrere Dateien, außer der Nutzer sagt das explizit.

---

## 8. Checkliste für neue Inhalte

**Neues Projekt hinzufügen:**
- [ ] Neuer 7-stelliger Hex-Hash + Jahr
- [ ] Titel im Format "Name — Kurzkategorie"
- [ ] 2–3 Sätze Kurzbeschreibung
- [ ] Optionale Detail-Zeile für `.commit-more`
- [ ] Tags mit `data-tech` in lowercase, passend zu bestehenden Stack-Tags wo möglich
- [ ] Passende Links (`Code →`, `Live →` oder `Docs →`)

**Stack-Eintrag ändern/hinzufügen:**
- [ ] Passt in eine der bestehenden Gruppen (mobile / frontend / backend+apis / tooling)
      oder rechtfertigt eine neue `.tree-group`
- [ ] `data-tech`-Wert konsistent mit evtl. verwandten Projekt-Tags
- [ ] Ein-Zeiler-Kommentar im `// ...`-Stil ergänzt

**Neue Section:**
- [ ] Eyebrow im `// name`-Format + `<h2>`
- [ ] Passt visuell/konzeptionell zur Terminal/Code-Metapher
- [ ] Trägt `.reveal` für Scroll-Animation
- [ ] Nutzt ausschließlich bestehende Design-Tokens
