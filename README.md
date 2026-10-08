# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Three or four sentences: what a user asks for, and what they get back. -->

FitFindr helps someone shopping second-hand. They type what they're looking for in plain language, like "vintage graphic tee under $30, size M", and it searches 40 thrift listings for the best match within their size and budget. It then suggests one or two outfits built around that item using clothes they already own, and writes a short social-media caption about the find. If nothing matches, it stops and tells them what to change (broader words, a different size, or a higher price) instead of inventing a result.


---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** Finds thrift listings whose text shares keywords with the user's description, optionally filtered by size and maximum price, and ranks them by how many keywords match.
- **Inputs:** `description` (str), e.g. `"vintage graphic tee"`; `size` (str or None), e.g. `"M"`, where None means any size; `max_price` (float or None), in dollars, inclusive, where None means any price. The listing's size is split at `/`, with anything in brackets ignored, and a size matches when it equals one of those parts exactly, ignoring case: `"M"` matches `"S/M"` but not `"XL"`, `"XL (oversized)"` counts as `"XL"`, shoe sizes need the full `"US 9"`, and `"One Size"` matches any size.
- **Returns:** A list of up to 10 listing dicts, best match first, each with `id`, `title`, `description`, `category`, `style_tags` (list), `size`, `condition`, `price` (float), `colors` (list), `brand` (str or None) and `platform`.
- **When it has nothing:** An empty list `[]`, never None and never an error.

### `suggest_outfit`

- **What it does:** Asks the AI model for one or two outfits built around the thrifted item, using pieces from the user's wardrobe.
- **Inputs:** `new_item` (dict), one listing dict from `search_listings`; `wardrobe` (dict) with an `"items"` key holding a list (possibly empty) of wardrobe dicts, each with `id`, `name`, `category`, `colors` (list), `style_tags` (list) and `notes`.
- **Returns:** A non-empty string (str) with one or two outfit suggestions, each naming the new item and the wardrobe pieces it's paired with by their `name`.
- **When it has nothing:** If `wardrobe["items"]` is empty, a non-empty string (str) of general styling advice for the item (what kinds of pieces and colors go with it), never `""` and never an error.

### `create_fit_card`

- **What it does:** Asks the AI model for a short caption about the find that reads like a real social-media post, not a product description.
- **Inputs:** `outfit` (str), the text from `suggest_outfit`; `new_item` (dict), the listing dict for the item.
- **Returns:** A string (str) of 2–4 sentences that mentions the item's `title`, `price` and `platform` once each and describes the vibe, with different wording on each run.
- **When it has nothing:** If `outfit` is empty or only spaces, the string `"Couldn't write a fit card: no outfit suggestion was provided."`, without calling the model and without raising an error.

---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:** If `search_listings` returns an empty list, put a message in `session["error"]` that says what the user could change (for example a higher price or a different size), and stop without calling `suggest_outfit`. Otherwise, take the first result as `session["selected_item"]` and go on to `suggest_outfit`, then `create_fit_card`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regex, in `agent.py::parse_query`. One pattern pulls out a price ceiling (`under $30`, `below $30`, `max $30`), another pulls out a size (`size M`, `size US 8`, `size W30`, or a bare size at the end like `, M`), and whatever is left becomes the description. I chose regex over asking the model because it costs no model call and gives the same answer every time, and when it's wrong the reason is visible in the pattern. The trade-off is that it misses wording it doesn't know: "nothing over thirty dollars" gives no price at all, so the price filter is silently skipped.

**What moves through the session:** In this order: `query` (what the user typed) → `parsed` (`description`, `size`, `max_price`) → `search_results` (the full list from `search_listings`) → `selected_item` (the first result, read back out of the session and passed to `suggest_outfit`) → `outfit_suggestion` (read back out and passed with `selected_item` to `create_fit_card`) → `fit_card`. `wardrobe` is set at the start and read by `suggest_outfit`. `error` is set only when the run stops early, and then the fields after it stay `None`.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30, size M'
[1] parse_query
      in:  vintage graphic tee under $30, size M
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Mesh Long-Sleeve Top — Black, 90s Silk Slip Dress — Floral, Midi Length … +7 more
      →    10 match(es)
[3] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[4] suggest_outfit
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: Hey friend! That butterfly baby tee is super cute and totally worth $18. Here are two easy ways to style it wi…
      →    10 wardrobe item(s)
[5] create_fit_card
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: Score! Just hunted down this dreamy Y2K baby tee on depop for only $18, and I am obsessed with the butterfly p…

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Hey friend! That butterfly baby tee is super cute and totally worth $18. Here are two easy ways to style it with what you already have:

**Outfit 1: Casual & Edgy**
Pair the Y2K baby tee with your baggy straight-leg jeans, dark wash. Toss the vintage black denim jacket on top and lace up your black combat boots. Throw on the black crossbody bag to finish the look. It's a great balance of tight top and loose denim!

**Outfit 2: Easy Everyday**
Keep it comfy by wearing the baby tee with your wide-leg khaki trousers. Slip on your chunky white sneakers, and if it gets chilly, just layer your black cropped zip hoodie right over it. Simple, cute, and ready to go!

  Fit card: Score! Just hunted down this dreamy Y2K baby tee on depop for only $18, and I am obsessed with the butterfly print. I styled it with baggy dark-wash jeans, a vintage black denim jacket, and combat boots for an edgy, balanced streetwear vibe.

0 model calls this session, 2 served from cache
```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"
[{'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_017', 'title': 'Mesh Long-Sleeve Top — Black', 'description': 'Sheer black mesh long-sleeve. Great for layering under a graphic tee or over a bralette. Stretchy material, fits true to size.', 'category': 'tops', 'style_tags': ['y2k', 'grunge', 'goth', 'layering'], 'size': 'S/M', 'condition': 'excellent', 'price': 15.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_033', 'title': 'Vintage Band Tee — Faded Grey', 'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 19.0, 'colors': ['grey', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_011', 'title': 'Low-Rise Cargo Pants — Khaki', 'description': 'Y2K era low-rise cargo pants. Lots of pockets. Khaki color, slightly distressed at the hems. Great for layering with a long tee.', 'category': 'bottoms', 'style_tags': ['y2k', 'cargo', '2000s', 'streetwear'], 'size': 'W29', 'condition': 'fair', 'price': 27.0, 'colors': ['khaki', 'tan'], 'brand': None, 'platform': 'poshmark'}, {'id': 'lst_015', 'title': 'Vintage Graphic Hoodie — Faded Black', 'description': 'Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'graphic', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 26.0, 'colors': ['black', 'charcoal'], 'brand': None, 'platform': 'depop'}]
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
Here are two easy ways to style your new Vintage Levi's 501 Jeans — Medium Wash:

**Casual Streetwear**
Pair the jeans with your white ribbed tank top. Throw on the oversized grey crewneck sweatshirt for a relaxed, comfortable layer. Finish it off with your chunky white sneakers and the black crossbody bag for an easy, everyday look.

**Edgy & Classic**
Tuck the white ribbed tank top into the jeans and add the brown leather belt. Layer the black cropped zip hoodie on top, and wear your black combat boots to give the classic denim a tougher edge. Complete the outfit with your black crossbody bag.
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Score! Snagged these vintage Levi's 501 jeans on depop for just $38, and they fit like an absolute dream. I kept the rest of the fit super effortless with crisp white sneakers for that classic, laid-back streetwear vibe.
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* Examples of questions to ask the model to test functions 
- *What came back:* A detailed set of questions/prompts that tested suituations where answers could be found, not found or tricky
- *What I changed:* Nothing

**Moment 2**

- *What I asked for:* I gave Claude my MCP tool description for `search_listings` in `mcp_server.py` and asked it to check that against the code.
- *What came back:* It found that my description said results were sorted "cheaper first among equal matches", but `tools.py::search_listings` only sorts by keyword score, so listings with the same score stay in catalog order. It also pointed out that I hadn't stated the 10-result limit.
- *What I changed:* I removed the false price-ordering claim, said that listings with the same score keep catalog order, added "at most 10", and fixed the typos.
<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. A matching query completes all three tools | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. An impossible query stops before the second tool | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. `selected_item` is the item `suggest_outfit` received | 5 of 5 | PASS | PASS | PASS | FAIL | PASS | MISSED (4/5) |
| 4. The fit card includes the exact price | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. A size "S" query returns no XL / US 9 / W30 sizes | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

Full output: `results/run_2026-10-07_2252_before.md`, written by `run_eval.py::main`, from `python run_eval.py --label before` (5 tries per scenario, caching off).

**Real output from one try**, pasted as text, naming the file and function
that produced it:

**Criterion 1**: "matching query completes", try 1. Trace from `agent.py::run_agent`, fit card from `tools.py::create_fit_card`:

```
[1] parse_query
      in:  vintage graphic tee under $30
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      →    10 match(es)
[3] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[4] suggest_outfit
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: Here are two cute, easy ways to style your new Y2K baby tee!  **Outfit 1: Casual & Classic** Pair the baby tee…
      →    10 wardrobe item(s)
[5] create_fit_card
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: I scored this adorable Y2K baby tee with a butterfly print on depop for just $18, and I am obsessed with how v…

Fit card:
I scored this adorable Y2K baby tee with a butterfly print on depop for just $18, and I am obsessed with how versatile it is! For a classic casual vibe, I paired it with baggy dark-wash jeans and a vintage black denim jacket, but it looks equally cute layered over khaki trousers with combat boots for a softer, edgy contrast.
```

**Criterion 2**: "impossible query stops early", try 1. Trace and `session["error"]` from `agent.py::run_agent`:

```
[1] parse_query
      in:  designer ballgown size XXS under $5
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
      →    0 match(es)
[3] branch
      →    search returned []: stopping before suggest_outfit

session["error"]:
Nothing in the listings matched description 'designer ballgown', size XXS, under $5.
Things to change: try broader words — 'jacket' finds more than 'cropped corduroy jacket'; drop the size, or try a neighbouring one; raise the price ceiling above $5.
```

**Criterion 3**: "selected item reaches suggest_outfit", try 4 (the FAIL). Trace and `session["error"]` from `agent.py::run_agent`:

```
[1] parse_query
      in:  90s track jacket
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: 90s Track Jacket — Navy/White Stripe, 90s Leather Bomber — Black, Denim Jacket — Light Wash, Cropped … +7 more
      →    10 match(es)
[3] select_item
      out: 90s Track Jacket — Navy/White Stripe ($45.0, poshmark)
[4] model unavailable
      →    stopping, search results kept

session["error"]:
The model couldn't be reached, so the outfit and caption steps didn't run. The search worked — 10 listing(s) were found. Check GEMINI_API_KEY in your .env, then run the same query again.
What the service said: Couldn't reach the model: 503 UNAVAILABLE. {'error': {'code': 503, 'message': 'This model is currently experiencing high demand. Spikes in demand are usually temporary. Please try again later.', 'status': 'UNAVAILABLE'}}
```

**Criterion 4**: "fit card includes exact price", try 1. `session["fit_card"]` from `tools.py::create_fit_card` (selected item: Denim Jacket — Light Wash, Cropped, $42.0, poshmark):

```
I just scored this vintage Wrangler denim jacket on Poshmark for only $42, and I am already obsessed with the light wash and cropped fit! I went for a casual, streetwear-inspired vibe by throwing it over a white ribbed tank with baggy dark-wash jeans and chunky sneakers for the ultimate double denim look.
```

**Criterion 5**: "size S filter excludes other sizes", try 1. Trace from `agent.py::run_agent`, with sizes of all 4 results from `tools.py::search_listings` (called via `mcp_client.call_tool`):

```
[1] parse_query
      in:  vintage graphic tee under $30, size S
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 4 items: Y2K Baby Tee — Butterfly Print, Mesh Long-Sleeve Top — Black, Leather Belt — Brown, Braided … +1 more
      →    4 match(es)

All 4 results with their sizes:
[('Y2K Baby Tee — Butterfly Print', 'S/M'), ('Mesh Long-Sleeve Top — Black', 'S/M'), ('Leather Belt — Brown, Braided', 'One Size (adjustable)'), ('Bucket Hat — Reversible, Brown Plaid', 'One Size')]
```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | A matching query completes all three tools | 4 of 5 | MET (5/5) | All 5 tries reached `[5] create_fit_card` in the trace, and `session["fit_card"]` was a non-empty string each time. |
| 2 | An impossible query stops before the second tool | 5 of 5 | MET (5/5) | All 5 traces end at `[3] branch` with no `suggest_outfit` step, and `session["error"]` says what to change (broader words, drop the size, raise the price). |
| 3 | `selected_item` is the item `suggest_outfit` received | 5 of 5 | MISSED (4/5) | In tries 1, 2, 3 and 5, the item in `[3] select_item` and the `in:` of `[4] suggest_outfit` were the same (90s Track Jacket, $45.0, poshmark). In try 4 there is no `suggest_outfit` step, so there's no evidence of what it received, and a pass can't be claimed. |
| 4 | The fit card includes the exact price | 4 of 5 | MET (5/5) | All 5 fit cards for the Denim Jacket ($42.0) contain "$42". |
| 5 | A size "S" query returns no XL / US 9 / W30 sizes | 5 of 5 | MET (5/5) | All 4 results each time were sized S/M, S/M, One Size (adjustable) and One Size, and none was XL, US 9 or W30. |

**Diagnoses**

**Criterion 3, missed on 1 of 5 (try 4). Place: the model, not the session.** In try 4, `suggest_outfit` called the Gemini model through `generate()`, and the service returned `503 UNAVAILABLE: This model is currently experiencing high demand`. `generate()` raised `ModelUnavailable`, and `agent.py::run_agent` caught it and stopped with the "model couldn't be reached" message. Step `[3] select_item` still shows the right item (90s Track Jacket, $45.0, poshmark), so the session held the correct value. But the `suggest_outfit` trace step is only written *after* the tool returns, so when the call raised, the step never appeared and its input was never recorded. The miss has two causes: the service was briefly unavailable, and my trace only logs a step's input after the step succeeds, so a failed call leaves no evidence of what it was given. The session worked; the evidence for it didn't survive the model error.

**Pattern.** The only failure across all 30 tries came from the model service being unavailable. It's the same "model unavailable" failure mode I triggered on purpose with a bad key, and the agent handled it the same way: no crash, a clear message, and the search results kept. Nothing failed in `search_listings`, the empty-search branch or the session. Those parts use no model, and they held 5 of 5 every time.

**Were my targets too low?** Criteria 1 and 4 targeted 4 of 5 and both got 5 of 5, so they had room to spare. Criterion 4 is the one I'd tighten. It only ever tested one item, the Denim Jacket at $42, so 5 of 5 shows the model copies that one price reliably, not that it does so for any item. A stricter version would run 5 *different* items, each with a different price, at 5 of 5.



---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```
$ python app.py ask 'vintage denim jacket under $50' --trace
[1] parse_query
      in:  vintage denim jacket under $50
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Denim Jacket — Light Wash, Cropped, Vintage Levi's 501 Jeans — Medium Wash, 90s Track Jacket — Navy/White Stripe … +7 more
      →    10 match(es)
[3] select_item
      out: Denim Jacket — Light Wash, Cropped ($42.0, poshmark)
[4] suggest_outfit
      in:  Denim Jacket — Light Wash, Cropped ($42.0, poshmark)
      out: Hey friend! That cropped Wrangler jacket is such a fun find. Here are two easy ways to style it using what's a…
      →    10 wardrobe item(s)
[5] create_fit_card
      in:  Denim Jacket — Light Wash, Cropped ($42.0, poshmark)
      out: Just scored this vintage cropped Wrangler denim jacket on Poshmark for only $42, and I am already obsessed wit…

```

**Empty search**

```
$ python app.py ask 'sequined astronaut ballgown under $5' --trace
[1] parse_query
      in:  sequined astronaut ballgown under $5
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
      →    0 match(es)
[3] branch
      →    search returned []: stopping before suggest_outfit

  Nothing in the listings matched description 'sequined astronaut ballgown', under $5.
Things to change: try broader words — 'jacket' finds more than 'cropped corduroy jacket'; raise the price ceiling above $5.

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->

I registered search_listings in mcp_server.py with a description and typed inputs, and the agent now calls it through call_tool instead of directly. Nothing changed. Over MCP, "graphic tee" under $30 gave the same 6 results as the direct call, and an empty search still returns [].

---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
