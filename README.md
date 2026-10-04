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


FitFindr is a thrifting agent. The user types what they want, like "vintage graphic tee under $30", and the agent searches a listings file, picks the best match, suggests outfits that pair it with the user's wardrobe, and writes a short caption they could post. If nothing matches, it stops and tells the user what to change instead of calling the later tools.


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


- **What it does:** Searches data/listings.json for listings matching a description, an optional size, and an optional price ceiling.
- **Inputs:** `description` (str), `size` (str or None), `max_price` (float or None, inclusive)
- **Returns:** A list of listing dicts (id, title, description, category, style_tags, size, condition, price, colors, brand, platform), best match first, at most config.SEARCH_RESULT_LIMIT items. Ranking is keyword overlap with the title, description, category, brand, style tags and colors, with title matches counted twice. Ties go to the cheaper listing. Plurals are stripped ("tees" matches "tee"). A size matches when every token of the requested size appears as a whole token in the listing's size, case-insensitive, so "M" matches "S/M" and "M/L" but not "XL".
- **When it has nothing:** Returns an empty list `[]`, never None.

### `suggest_outfit`

- **What it does:** Takes one found listing and the user's wardrobe and suggests outfits pairing the new item with items they already own.
- **Inputs:** `new_item` (dict, one listing), `wardrobe` (dict with an "items" list of wardrobe item dicts)
- **Returns:** A string of outfit ideas that names specific wardrobe items.
- **When it has nothing:** If the wardrobe is empty, returns a string of general styling advice for the item, not an error.

### `create_fit_card`

- **What it does:** Writes a short caption someone would post about the new item and its outfit.
- **Inputs:** `outfit` (str, the output of suggest_outfit), `new_item` (dict, one listing)
- **Returns:** A string caption, 2 to 4 sentences, that mentions the item, its price and its platform.
- **When it has nothing:** If `outfit` is empty or whitespace, returns a descriptive message string instead of raising.

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

**Branch rule:** If search_listings returns an empty list, set session["error"] to a message naming what to change (raise the price, drop the size, or use a broader description) and stop without calling suggest_outfit or create_fit_card. Otherwise, store the first result in session["selected_item"] and continue to suggest_outfit.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regex, in `_parse_query` in agent.py. A price phrase ("under $30") becomes `max_price`, "size M" becomes `size`, and the remaining words become the description.

**What moves through the session:** `parsed`, then `search_results`, then `selected_item` (the first result), then `outfit_suggestion`, then `fit_card`. Each tool reads its input back out of the session. If the search is empty, `error` is set and the later fields stay None.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30'
[suggest_outfit] new_item id = lst_033

  Found:    Vintage Band Tee — Faded Grey — $19.0 on depop

  Outfit:   **Outfit 1: Effortless Grunge**
Pair the vintage band tee with the baggy straight-leg jeans (dark wash) for a classic streetwear silhouette. Add the black combat boots to lean into the grunge aesthetic, and sling the black crossbody bag over your shoulder for easy everyday wear.

**Outfit 2: Casual Contrast**
Tuck the vintage band tee into the wide-leg khaki trousers for a mix of earth tones and edgy graphics. Cinch the waist with the brown leather belt, and finish the look with the chunky white sneakers for a relaxed, modern streetwear vibe.

  Fit card: Scored this perfectly faded grey vintage band tee on depop for just $19. Paired it with dark baggy jeans and combat boots for an effortless grunge look today. Obsessed with how worn-in it feels.

$ python app.py ask 'designer ballgown size XXS under $5'

  No listings matched "designer ballgown". Try to raise the price limit ($5), or drop or change the size (XXS), or use a broader description.
```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print([(l['title'], l['price']) for l in search_listings('graphic tee', max_price=30)])"
[('Graphic Tee — 2003 Tour Bootleg Style', 24.0), ('Y2K Baby Tee — Butterfly Print', 18.0), ('Vintage Band Tee — Faded Grey', 19.0), ('Mesh Long-Sleeve Top — Black', 15.0), ('Vintage Graphic Hoodie — Faded Black', 26.0), ('Oversized Crewneck Sweatshirt — Vintage Navy', 20.0), ('Low-Rise Cargo Pants — Khaki', 27.0)]

```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
[suggest_outfit] new_item id = lst_001
**Outfit 1: Casual Streetwear**
Pair the vintage Levi's 501 Jeans with the white ribbed tank top, black cropped zip hoodie, chunky white sneakers, and black crossbody bag. Cinch the waist with the brown leather belt for a polished contrast.

**Outfit 2: Cozy Retro**
Style the vintage Levi's 501 Jeans with the oversized grey crewneck sweatshirt and chunky white sneakers. Add the brown leather belt to define the waist, and layer the vintage black denim jacket on top for a classic, textured finish.

```
**Empty wardrobe test**

```text
$ python -c "from tools import suggest_outfit; from utils.data_loader import load_listings; print(suggest_outfit(load_listings()[0], {'items': []}))"
[suggest_outfit] new_item id = lst_001
These vintage Levi's 501s are a versatile closet staple. Here are two easy ways to style them:

**1. The Casual Classic:** Pair the jeans with a tucked-in plain white t-shirt and white canvas sneakers. Add a simple leather belt and a tote bag for an effortless, everyday look that never goes out of style.

**2. Elevated Streetwear:** Layer an oversized black hoodie or a cropped cardigan over the tee, and swap the sneakers for chunky loafers or retro runners. Accessorize with silver jewelry or a baseball cap to lean into the vintage streetwear vibe.
```

```
$ AI201_CACHE=0 python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Pulled these Levi's 501 jeans off depop for $38 and honestly, the fit is everything. Threw them on with my beat-up white sneakers for that effortlessly lazy Sunday coffee run vibe. Absolute gold mine find.

$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('', load_listings()[0]))"
No outfit suggestion was provided, so there is nothing to write a caption about yet.

```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* Help writing `search_listings`, including how to match sizes without a plain substring test.
- *What came back:* Whole-token size matching and keyword scoring. When I tested "graphic tee", the Mesh Long-Sleeve Top ranked first, ahead of the real graphic tees.
- *What I changed:* I added a bonus for words that match the title, re-ran the test, and the Graphic Tee moved to first.

**Moment 2**

- *What I asked for:* A `create_fit_card` that writes a caption mentioning the item, price and platform.
- *What came back:* It worked, but running it twice gave word-for-word identical captions.
- *What I changed:* I checked `config.py` and found `TEMPERATURE` was already 0.9, so the cause was the cache replaying identical prompts. I ran with `AI201_CACHE=0` and got three different captions.

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
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

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
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**



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

```

**Empty search**

```

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



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
