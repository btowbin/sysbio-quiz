# Systems Biology Quiz

One self-contained `index.html`: 79 multiple-choice questions (Lectures 1–6 + "More exam topics"), two modes.

* **Practice** – students pick lectures, answer, get instant feedback + explanation. Progress is kept in their browser only.
* **Live** – you project the *presenter view*, students scan the QR code and answer on their phones, you reveal the solution and the class result (correct / wrong / per-option bars) is shown on the screen. Answers travel peer-to-peer (WebRTC, via the free public PeerJS signalling server) — no account, no backend of your own, no student data stored anywhere.

## Put it on GitHub Pages (5 minutes)

1. Create a GitHub repository (e.g. `sysbio-quiz`), upload `index.html` to the root.
2. *Settings → Pages → Build and deployment → Deploy from a branch → `main` / `(root)` → Save.*
3. After a minute the site is at `https://<your-user>.github.io/sysbio-quiz/`.

Student link: that address. Presenter view: that address + `#host`  (e.g. `https://<user>.github.io/sysbio-quiz/#host`).
The page loads KaTeX (math), PeerJS and a QR generator from the jsDelivr CDN, so students need internet (they have it on their phones anyway).

## Running a live session

1. Open the presenter view on the lecture-hall computer and put it on the projector. You see a QR code, the web address and a 5-letter room code.
2. Tick the lectures whose questions you want in this session → **Start quiz**.
3. Each question: students answer (the header counts *answered / online*) → press **Space** (or *Reveal answer*) to show the solution and the correct/wrong tally → **Space** again for the next question.
4. The room code is remembered by the browser, so the same QR code can stay on your slides for the whole semester. *New room code* creates a fresh one.

Keyboard shortcuts (presenter): `Space`/`→` next step, `←` back, `R` reveal, `E` show/hide explanation, `L` question list (jump to any question, download results as CSV), `Q` big QR code, `F` fullscreen.
If you reload the presenter page, the session (position and collected answers) is restored and students reconnect automatically. Keep the presenter tab open during the lecture.

**Test it once on the lecture-hall Wi-Fi before the first live lecture.** Peer-to-peer connections can be blocked by campus firewalls (typically when presenter and students are on different networks, e.g. eduroam vs. mobile data). The page already uses PeerJS' public relay as a fallback, but that is shared and best-effort (UDP port 3478 only). For reliable operation add **your own TURN relay** (see below).

### Adding a TURN relay
Near the top of the script (search for `TURN_SERVERS`) there are two settings, use one of them:

* **Fixed credentials** (own coturn server, or a provider that gives you a static user/password):
  `const TURN_SERVERS = [ {urls:'turn:turn.example.org:443?transport=tcp', username:'USER', credential:'PASSWORD'} ];`
* **Credentials URL** (e.g. Metered's free tier: create an app, copy the "credentials API" URL with your API key):
  `const TURN_CREDENTIALS_URL = 'https://YOURAPP.metered.live/api/v1/turn/credentials?apiKey=YOURKEY';`

The relay must be reachable from the students' phones and your laptop; a TURN entry on port 443 over TCP/TLS gets through most strict firewalls.

**Check that it works:** add `?relay` to the addresses to force relay-only connections. Presenter: `https://…/?relay#host`, student: `https://…/?join=CODE&relay`. If the quiz still connects, the relay works.

## Editing / adding questions

All questions are in one block at the top of `index.html` (search for `QUESTION BANK`). Example:

```js
{ id:'L2-15', lec:2, topic:'Hill function',
  q:r`What is $\rho$ for $x=K$?`,
  opts:[r`1/2`, r`0`, r`1`, r`depends on $n$`],      // FIRST option is the correct one – the page shuffles them
  exp:r`$\rho(K)=1/2$ for every $n$.`,               // explanation shown after the answer
  src:'Exam HS25 Q3a' }                              // optional
```

* The **first option is always the correct one**; options are shuffled automatically (same order for everyone in a live session). Use `fixed:true, ans:<index>` to keep your own order.
* Math: `$…$` inline, `$$…$$` display (KaTeX). Write `\lt` / `\gt` instead of `<` / `>` inside math. Always wrap texts in r`…` so backslashes work.
* `id` must be unique and should never change (progress and live sessions refer to it). `lec` is 1–6 (7 = "More exam topics"; names are in `LECTURES`).
* Figures: add `fig:{ x:[0,5], y:[0,3], curves:[{fn:'1-exp(-x)', c:'a'}], … }` — see the existing questions for examples (curves `fn` = y(x) or `fx` = x(y), points, dashed lines, labels).
* In the live session, students and presenter must run the same version of the page; if a student has an old cached copy they are told to reload.

## Notes on content

* Questions are modelled on exams HS21–HS24 and Assignments 1–4 and on the lecture handouts/slides; the source is shown for each one (`Modelled on: …`). Figures inside the exam PDFs were not used; the figures in the quiz are generated from formulas.
* Lecture 4 also holds the oscillator questions (shower analogy, relaxation oscillator, *C. elegans* clock) since that material sits in Lecture 4's handout and the start of Slides 5; the C. elegans size/proportion questions are in Lecture 6. Move them by changing `lec`.
* The Lecture 6 handout is short, so those questions rely mostly on the slides. Please review them especially.
* Please read through all solutions once — in particular `L5-04` (early vs. late reproduction, solved with the Euler–Lotka equation assuming full survival), `X-04` (MAPK feedback) and the wording of `L3-10`.
