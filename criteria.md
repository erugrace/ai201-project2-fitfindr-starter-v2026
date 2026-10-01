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
Search is a plain keyword mat so a phrasing like "tee" won't find a listing under "Tshirt". One miss in 5 allows for that

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
It is important for the agent to not try returning any fit card for a query that is not related to any fit all the time so it doesn't give the wrong answers to the user.

## 3. Something about state

Given a query that matches at least one listing, the `id` of
`session["selected_item"]` equals the `id` of the item `suggest_outfit`
actually received — in 5 of 5 tries.

**Why this target:**
Passing the chosen item from the session into the next tool is plain Python
with no model involved, so nothing random can change it. A single miss would
mean my loop is reading the item from the wrong place, so anything less than
5 of 5 would hide a real bug.

---

## 4. Something about the fit card


Given a query that matches at least one listing, the fit card includes the
selected item's exact price (e.g. "$24") — in at least 4 of 5 tries.

**Why this target:**
The price is in the prompt I send to the model, but the model writes the
words, so it can still leave the price out or round it. One miss in five
allows for that; more than one would mean my prompt isn't asking for the
price clearly enough.

---

## 5. Your choice

Given a query asking for size "S", no item in `session["search_results"]`
has a size like "XL", "US 9" or "W30" — in 5 of 5 tries.

**Why this target:**
My size rule matches whole words only, so "S" should match "S" and "S/M" but
never "XL" or "US 9", which a plain substring check would let through. The
filter is plain code with no model involved, so it should be right every
time.

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
