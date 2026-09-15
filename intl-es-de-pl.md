<!-- испанский (dev.to #spanish, r/programacion), немецкий (dev.to #german/#deutsch), польский (Wykop, 4programmers) · публикуются с аккаунта SMM как обзор -->

# Español · dev.to (#spanish) · ~1 300 caracteres

## Título
myc: memoria local para agentes de código (Claude Code, Codex) que sobrevive a la compactación del contexto, sin API key

## Texto

Quien usa Claude Code o Codex más de una hora conoce la escena: acuerdas con el agente «vamos con A, no con B, por C», una hora después el contexto se compacta y el agente propone B otra vez, con toda naturalidad. El motivo ya no está en su contexto.

Encontré en GitHub un proyecto open source hecho exactamente para esto: **myc** (de mycelium). No es otra API de memoria en la nube: es una capa local de tareas y memoria en un único archivo SQLite junto al proyecto.

Qué hace:

- cola de tareas con dependencias, bloqueos y claim atómico (dos agentes nunca toman la misma tarea);
- registro de decisiones con búsqueda híbrida (BM25 + vectores);
- hook de PreCompact: justo antes de la compactación guarda la sesión en disco (secretos enmascarados) y devuelve al contexto un «paquete de rescate» con lo esencial;
- las decisiones extraídas automáticamente de la conversación entran como candidatas y solo se vuelven hechos cuando un humano las confirma;
- memoria anclada a fragmentos de código: si el código se mueve, la memoria lo sigue; si desaparece, la entrada se degrada (`[code gone ×0.2]`) en vez de servirse como verdad;
- índice de código con tree-sitter (TypeScript/JavaScript/Python).

La velocidad es una restricción de diseño (100 000 nodos, p99): paquete de contexto 0,6 ms, búsqueda 8 ms, arranque en frío 21 ms, reproducible con `bun run scripts/bench-latency.ts`; la build del sitio falla si un número se desvía de la medición. Más de 3 700 tests.

Limitaciones dichas de frente: solo corre en Bun (bun:sqlite), macOS/Linux, uso individual, sin servidor ni modo equipo por ahora. MIT.

```bash
bun install -g @aistastudio/myc
myc init && myc wire   # conecta Claude Code / Codex / opencode / Kimi
```

Repo: https://github.com/aistastudio/myc · Sitio con mediciones reproducibles: https://aistastudio.github.io/myc/

# Deutsch · dev.to (#german, #deutsch) · ~1 300 Zeichen

## Titel
myc: lokales Gedächtnis für Coding‑Agenten (Claude Code, Codex), das die Kontext‑Kompaktierung überlebt, ohne API‑Key

## Text

Wer Claude Code oder Codex länger als eine Stunde benutzt, kennt die Szene: Man vereinbart mit dem Agenten „A, nicht B, wegen C“, eine Stunde später wird der Kontext kompaktiert, und der Agent schlägt B wieder vor, völlig arglos. Der Grund existiert in seinem Kontext einfach nicht mehr.

Auf GitHub habe ich ein Open‑Source‑Projekt gefunden, das genau dafür gebaut ist: **myc** (von Mycelium). Keine weitere Cloud‑Memory‑API, sondern eine lokale Schicht aus Aufgaben und Gedächtnis in einer einzigen SQLite‑Datei neben dem Projekt.

Was es macht:

- Aufgabenqueue mit Abhängigkeiten, Blockern und atomarem Claim (zwei Agenten nehmen nie dieselbe Aufgabe);
- Entscheidungslog mit hybrider Suche (BM25 + Vektoren);
- PreCompact‑Hook: unmittelbar vor der Kompaktierung wird die Session auf die Platte geschrieben (Secrets maskiert) und ein „Rettungspaket“ mit dem Wesentlichen zurück in den Kontext gedruckt;
- automatisch aus dem Gespräch extrahierte Entscheidungen sind Kandidaten, keine Fakten, bis ein Mensch sie bestätigt;
- Gedächtnis ist an Codestellen verankert: zieht der Code um, zieht das Gedächtnis mit; ist der Code weg, wird der Eintrag herabgestuft (`[code gone ×0.2]`), statt als Wahrheit serviert zu werden;
- Code‑Index mit tree-sitter (TypeScript/JavaScript/Python).

Geschwindigkeit ist eine Design‑Bedingung (100 000 Knoten, p99): Kontextpaket 0,6 ms, Suche 8 ms, Kaltstart 21 ms, reproduzierbar mit `bun run scripts/bench-latency.ts`; der Site‑Build schlägt fehl, wenn eine Zahl von der Messung abweicht. Über 3 700 Tests.

Grenzen, offen gesagt: läuft nur auf Bun (bun:sqlite), macOS/Linux, Einzelnutzer, bisher kein Server und kein Team‑Modus. MIT.

```bash
bun install -g @aistastudio/myc
myc init && myc wire   # bindet Claude Code / Codex / opencode / Kimi ein
```

Repo: https://github.com/aistastudio/myc · Site mit reproduzierbaren Messungen: https://aistastudio.github.io/myc/

# Polski · Wykop (#programowanie), 4programmers.net · krótki post, ~900 znaków

🧠 **Pamięć dla asystenta AI programisty**

myc nie pozwoli Claude Code zapomnieć, co razem ustaliliście.

Znajome? Mówisz agentowi: „kolejka, nie timer”. Godzinę później proponuje timer.

Nie ze złośliwości - jego pamięć jest ograniczona, a gdy kontekst się zapełnia, stare rzeczy znikają.

**Co robi myc:**

🔹 trzyma wszystkie decyzje i zadania w jednym pliku obok projektu
🔹 w chwili, gdy agent ma zapomnieć, zapisuje najważniejsze i oddaje mu z powrotem
🔹 to, co wyciągnięte z rozmowy automatycznie, traktuje jako szkic, dopóki nie potwierdzisz
🔹 wiąże pamięć z liniami kodu: kod się przenosi, pamięć za nim; kod usunięty, pamięć o tym wie

🔸 Nic nie wychodzi z komputera, bez kluczy API
🔸 Szukanie w 100 000 notatek w 8 ms, agent nie zauważa opóźnienia
🔸 Jedna komenda podłącza Claude Code, Codex, opencode i Kimi (wymaga Bun)

Open source, za darmo. Leży [tutaj](https://github.com/aistastudio/myc).
