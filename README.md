# Systems Biology Quiz

One self-contained `index.html`: 79 multiple-choice questions (Lectures 1–6 + "More exam topics"), two modes.

* **Practice** – students pick lectures, answer, get instant feedback + explanation. Progress is kept in their browser only.
* **Live** – you project the *presenter view*, students scan the QR code and answer on their phones, you reveal the solution and the class result (correct / wrong / per-option bars) is shown on the screen. Messages travel through free public MQTT message brokers (HiveMQ and EMQX) — no account, no backend of your own, nothing to install. Students are anonymous (random browser id); answers are stored only in the presenter's browser.

## Put it on GitHub Pages (5 minutes)

1. Create a GitHub repository (e.g. `sysbio-quiz`), upload `index.html` to the root.
2. *Settings → Pages → Build and deployment → Deploy from a branch → `main` / `(root)` → Save.*
3. After a minute the site is at `https://<your-user>.github.io/sysbio-quiz/`.

Student link: that address. Presenter view: that address + `#host`  (e.g. `https://<user>.github.io/sysbio-quiz/#host`).
The page loads KaTeX (math), the MQTT client and a QR generator from the jsDelivr CDN, so students need internet (they have it on their phones anyway).

## Running a live session

1. Open the presenter view on the lecture-hall computer and put it on the projector. You see a QR code, the web address and a 5-letter room code.
2. Click **Choose questions…** to set up the session:
   * **Lectures** – which lectures the questions come from.
   * **Random selection** – pick how many (3, 5, 8, … or all); *Draw again* gives a new random set; *Prefer questions not asked yet* skips questions that already have answers from earlier sessions. *Edit by hand* turns the current draw into a manual selection you can adjust.
   * **Pick questions myself** – tick individual questions (each shows its topic and question text; "all"/"none" per lecture).
   * **Order** – lecture order or shuffled.

   The lobby shows a one-line summary (e.g. "8 questions · random draw from L2, L3"). The setup is remembered by the browser. → **Start quiz**. You can also change the selection mid-session from the question list (`L` → *Change questions…*); answers already collected are kept.
3. Each question: students answer (the header counts *answered / online*) → press **Space** (or *Reveal answer*) to show the solution and the correct/wrong tally → **Space** again for the next question.
4. The room code is remembered by the browser, so the same QR code can stay on your slides for the whole semester. *New room code* creates a fresh one.

Keyboard shortcuts (presenter): `Space`/`→` next step, `←` back, `R` reveal, `E` show/hide explanation, `L` question list (jump to any question, download results as CSV), `Q` big QR code, `F` fullscreen.
If you reload the presenter page, the session (position and collected answers) is restored and students reconnect automatically. Keep the presenter tab open during the lecture.

**Test it once on the lecture-hall Wi-Fi before the first live lecture** (scan the QR code with your phone on eduroam and on mobile data).

### How the live connection works
The presenter page connects to **all** brokers listed in `BROKERS` (near the top of the script) and mirrors every message to each of them; each student phone uses the first broker it can reach. So a session keeps working as long as each phone reaches at least one of them. The presenter status line says *Room online*, or *Room online (1 of 2 message services reachable)* if one broker is down.

Student page messages: *Step 1 of 2* = reaching a message service (fails only if the network blocks ports 8884/8084 of both brokers – try mobile data); *Step 2 of 2* = waiting for the presenter page (check the room code / that the presenter page is open and says *Room online*).

Notes: the brokers are public, so anyone who knows a room code could in principle read or send messages in that room (no personal data is involved; answers are anonymous). To use your own broker (e.g. Mosquitto on a university VM with WebSockets over TLS), add its address to `BROKERS`, e.g. `'wss://mqtt.example.org:443/mqtt'`.

## Practice statistics

Open `https://<your-site>/#stats` (also linked from the presenter lobby). It shows anonymous totals from the self-study mode: page visits, practice sessions started, number of answered questions and % correct — overall, per lecture and per question (sortable, e.g. lowest % correct first). Live sessions are not included (use the CSV export there), and the repeat round of wrongly answered questions is not counted.

* No login, no cookies, no personal data: each answer just increments two counters (`answered`, `correct`) at the free counting service Abacus (abacus.jasoncameron.dev), namespace `STATS_NS` near the top of the script.
* Loading the page takes about a minute (the service allows ~30 requests per 10 s; one request per counter). The last result is cached in your browser.
* Limits: counters unused for 6 months expire; anyone who knows the namespace could tamper with the numbers; if many students practise at the same time behind one IP address (e.g. a computer lab), some counts may be dropped by the rate limit. Treat the numbers as indicative.
* Switch counting off with `const STATS_NS = '';` — or change the name to start counting from zero (e.g. per semester).

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
