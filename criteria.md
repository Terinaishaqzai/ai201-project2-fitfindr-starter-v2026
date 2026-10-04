# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"The agent handles errors"* is an opinion.
*"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing, the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:**
Even when search finds a listing, the next two tools depend on model calls that may fail because of a temporary API error or rate limit. A target of 4 of 5 allows one failed run while still requiring the full process to work consistently.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
No model call is needed on the empty-search path. The loop checks an empty list, so it should stop every time and tell the user to change their description, size, or price limit.

---

## 3. Something about state
Using the first five queries displayed by python app.py examples that match at least one listing, the item ID in session["selected_item"] matches the new_item["id"] logged inside suggest_outfit — in 5 of 5 runs, without asking the user to enter the item again.





**Why this target:**
The loop passes existing data between tools, so the item ID should match every time. A mismatch could produce outfit suggestions for the wrong item.


---

## 4. Something about the fit card

Generate one uncached fit card for each of the first five listings in data/listings.json, using the outfit returned by suggest_outfit for that listing and the provided wardrobe. At least 4 of 5 captions must include the listing’s brand when it is nonempty; otherwise, they must include a title word with at least four letters, excluding “vintage.” Comparisons ignore capitalization and punctuation. Each passing caption must also contain the correct price with a dollar sign, either without decimal places for a whole-dollar price or with exactly two decimal places, and contain between 1 and 60 whitespace-separated words. For example, a price of 24.0 accepts $24 or $24.00, but not “24 bucks.”


**Why this target:**
Identifying details and an explicit price make the caption useful, while the word limit keeps it short enough to post. Because model responses vary and may occasionally miss an instruction, the target allows one caption to fail rather than requiring all five to pass.


---

## 5. Your choice

For five calls to search_listings using the description "tee", no size restriction, and max_price values of $10, $20, $30, $40, and $50, every returned listing costs less than or equal to its requested limit, and at least three calls return a nonempty list.


**Why this target:**
Price filtering is a numeric comparison on existing data, so it should be exact in all five calls. Requiring at least three nonempty results prevents a search that always returns an empty list from passing.


---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 4 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 4. Something about the fit card

         The fit card is different every time.

         **Why this target:** ...

         > **Revised in unit 4:** For 5 different items, the 5 fit cards share
         > no opening sentence.
         >
         > **Why revised:** "different" wasn't checkable — two cards that
         > differed by one word still counted. The new version is something I
         > can actually score.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said the empty search stops it 5 of 5 times, but I got 3 of 5,
            so 3 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
