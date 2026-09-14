# SkoleOS-Anti-log
SkoleOS Anti-log sikker elev data

Udvidelse til Chromium-baserede browsere (Chrome, SkoleOS, Chromebooks), der blokerer baggrundstelemetri, brugeradfærdsmåling og unødvendig dataindsamling fra Google-tjenester i undervisningsmiljøer.

Udvidelsen er udformet efter princippet om mindst mulige rettigheder (Manifest V3) og er tilpasset, så undervisningsværktøjer og danske skoleportaler fungerer uden afbrydelser.

---

## Formål

Når elever og lærere anvender Google Workspace (Docs, Classroom, YouTube), opsamler Google løbende hændelsesdata i baggrunden – herunder tastaturforsinkelser (CSI), brugerflade-klik (`logImpressions`), streamingstatistik og telemetri (`play.google.com/log`).

SkoleOS Anti-log afskærer denne dataindsamling lokalt i browseren, uden at det påvirker selve dokumentredigeringen, lektieafleveringer eller videoafspilning.

---

## Hvad der blokeres

* **Google Workspace-telemetri:** Afskærer baggrundsrapportering via `play.google.com/log`, `docs.google.com/*/logImpressions` og `classroom.google.com/*/log`.
* **Måling af tastatur og latenstid:** Blokerer Client Side Instrumentation (`csi.gstatic.com`).
* **YouTube-adfærd og interaktionsdata:** Stopper `youtube.com/youtubei/v1/log_event`, reklamesporing (`api/stats/ads`) og `ptracking`.
* **Google Analytics og sporing:** Blokerer forespørgsler til `google-analytics.com`, `googletagmanager.com`, `doubleclick.net` og tilhørende sporingsnetværk på alle websider.
* **Integrerede AI-prompter:** Skjuler Gemini-prompter i Workspace-dokumenter, så elevtekster ikke sendes til ekstern modeltræning.

---

## Skolekompatibilitet og undtagelser

For at sikre stabil drift på skoler og institutioner indeholder udvidelsen faste undtagelser:

* **Aula og Unilogin:** Autentificering og Single Sign-On (`accounts.google.com`) er fuldt tilladt.
* **Google Analytics-attrap (Surrogate):** På eksterne skolesider (f.eks. forlag og skoleportaler) erstattes blokeret Analytics-kode med en lokal stub, der afvikler eventuelle ventende formular-callbacks med det samme. Derved undgås det, at afleveringsknapper eller logins hænger.
* **Google Classroom:** Aflevering af lektier (`/turnin`, `/submit`) og synkronisering med Google Drev er fredet.
* **Google Meet:** WebRTC-videokanaler og lydstrømme berøres ikke.
* **YouTube:** Selve videostrømmen (`googlevideo.com/videoplayback`) og afspillerens integritetstokens (`/api/jnn/`) er fredet, så videoer kan afspilles normalt.
* **Skrifttyper:** Google Fonts tillades, så typografi og ikoner vises korrekt.

---

## Teknisk arkitektur

Udvidelsen anvender tre koordinerede mekanismer:

1. **Declarative Net Request (DNR):** 53 præcise regler i `rules.json`, der blokerer kendte sporings-endpoints på browserens netværkslag før afsendelse.
2. **In-Page Interceptor (`scripts/page-interceptor.js`):** Overtager `fetch`, `XMLHttpRequest` og `sendBeacon` i dokumentets eksekveringskontekst (MAIN world). Når et telemetrikald afskæres, returneres et simuleret succes-svar (`204 No Content` eller tomt JSON-objekt), så applikationen lokalt tømmer sin kø uden fejltilstande.
3. **Surrogate Scriptlet (`surrogates/google-analytics.js`):** Definerer neutrale `ga`, `gtag` og `dataLayer`-objekter for at opretholde websiders interne scripts.

---
