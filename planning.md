# TakeMeter — Planning Document

## Community

I chose **r/travel**, a large public subreddit dedicated to travel discussion, trip sharing, and travel advice. The community is a strong fit for a classification task because posts vary significantly in purpose and structure: some people share personal stories, some post detailed day-by-day itineraries, and others share tips and recommendations. These distinctions are meaningful to community members — a person looking for packing advice doesn't want to read a trip story, and someone planning a route needs a structured itinerary, not general tips. The discourse is text-heavy, public, and consistently tagged with flair, making data collection straightforward.

---

## Label Taxonomy

### `experience`
**Definition:** The post is sharing a personal trip that already happened — a story, reflection, photo recap, or account of what the author did and saw.

**Example 1:**
> "Just got back from two weeks in Portugal. Lisbon completely blew me away — the trams, the food, the people. We did a day trip to Sintra and I'd 100% recommend it."

**Example 2:**
> "Finally made it to Patagonia after years of planning. The trek was brutal but worth every step. Here are some photos from the W Circuit."

---

### `planning`
**Definition:** The post is a structured schedule or itinerary — organized as a day-by-day or logistical plan, either asking for feedback on one or sharing a completed one.

**Example 1:**
> "Here's my 10-day Japan itinerary: Day 1-3: Tokyo, Day 4-5: Kyoto, Day 6: Nara, Day 7-9: Osaka, Day 10: fly home. Does this look feasible?"

**Example 2:**
> "Planning a 2-week Europe trip. Day 1: Arrive Amsterdam, Day 2-3: explore canals and museums, Day 4: train to Paris..."

---

### `advice`
**Definition:** The post shares tips, recommendations, or lessons learned that apply beyond the author's specific trip — what to do, what to avoid, what they'd do differently.

**Example 1:**
> "If you're visiting Rome, avoid eating anywhere within 200 meters of a major tourist attraction. Walk two streets over and prices drop by half."

**Example 2:**
> "Best tip I ever got for long-haul flights: book the aisle seat, bring noise-canceling headphones, and never skip the compression socks."

---

## Hard Edge Cases

**Anticipated ambiguous case:** A post structured as a day-by-day itinerary that *also* includes tips like "avoid this neighborhood" or "book this in advance."

**Decision rule:** Label by the **dominant structure** of the post.
- If the post is organized as a day-by-day or chronological schedule → `planning`
- If the post is organized as a list of tips/recommendations, even if it mentions a specific trip → `advice`
- If the post describes a completed trip narratively (not as a schedule) but includes some tips → `experience`

**Three difficult annotation examples** *(to be filled in after data collection)*:

Three difficult annotation examples:

Post: "Minimum pain, Maximum gain that is my motto nowadays...I decided on 
Seceda hike. Trust me, if you want to cheat by taking the cable car and can 
manage about 10 km, pick Seceda over any hike in the whole Alps." — 
Decided: advice — Why: Although written as a personal story, the primary 
purpose is recommending a specific hike and strategy to unfit hikers. The 
narrative is the vehicle for the advice, not the point itself.

Post: "Spent about a week in France this June...Blue hour on the Seine is the 
move for the Eiffel Tower. Pro tip we wish we'd known sooner: the glass pyramid 
entrance line is brutal, so book a timed slot in advance." — Decided: advice — 
Why: Despite mentioning a personal trip, the post is structured as a list of 
tips and recommendations ("is the move", "Pro tip"), not a personal narrative.

Post: "Slovenia was truly an amazing experience with friendly people and 
beautiful nature. Incredible to be able to drive from the Alps to the coast 
within a few short hours. Highly recommend making a trip here." — Decided: 
advice — Why: Short post with no personal narrative detail. The primary 
purpose is recommending Slovenia to others, not sharing a personal story.

---

## Data Collection Plan

**Source:** r/travel on Reddit, filtering by post flair.

**Method:** Use Reddit's flair filter to browse posts tagged "Trip Report", "Itinerary", and "My Advice" / "Travelers Only". Copy post title + body text into a CSV with columns: `text`, `label`. Collect manually — no scraping tools needed.

**Target distribution:**
| Label | Target Count |
|-------|-------------|
| `experience` | ~70 |
| `planning` | ~65 |
| `advice` | ~65 |
| **Total** | **~200** |

**If a label is underrepresented:** Search r/travel directly using flair filters to find more posts of that type. Do not pad with low-quality or borderline examples just to hit the number.

**CSV format:**
```
text,label
"Just got back from Japan...",experience
"Day 1: Arrive in Tokyo...",planning
"Always book airport transfers in advance...",advice
```

---

## Evaluation Metrics

**Primary metric: Per-class F1 score**
F1 is the harmonic mean of precision and recall. It's the right choice here because:
- The dataset may have slight class imbalance, which makes raw accuracy misleading
- All three labels matter equally — a model that ignores `planning` entirely is not useful even if overall accuracy looks okay
- F1 catches both over-prediction (low precision) and under-prediction (low recall) for each class

**Secondary metric: Overall accuracy**
Used for quick comparison between the fine-tuned model and the zero-shot baseline.

**Confusion matrix**
To identify which specific label pairs the model confuses most — especially the anticipated `planning` ↔ `advice` boundary.

---

## Definition of Success

The fine-tuned model will be considered successful if:
- Overall accuracy exceeds the zero-shot baseline by at least 10 percentage points
- Per-class F1 ≥ 0.65 for all three labels (no label is completely ignored)
- The confusion matrix shows no single off-diagonal cell accounting for more than 40% of one class's errors

A model meeting these thresholds would be genuinely useful as a post-categorization tool for a travel community — reliably distinguishing trip stories from itineraries from advice well enough to route users to the right content.

---

## AI Tool Plan

### Label stress-testing
I will provide my label definitions and edge case description to Claude and ask it to generate 5–10 posts that sit at the boundary between two labels. If those posts are hard to classify cleanly, I will tighten my definitions before annotating 200 examples.

### Annotation assistance
I may use an LLM to pre-label a batch of examples using my label definitions, then review and correct every prediction myself. If I do this, I will track which examples were pre-labeled and disclose it in the AI usage section of the README.

### Failure analysis
After fine-tuning, I will paste my misclassified examples into Claude and ask it to identify common patterns (e.g., short posts, sarcasm, topic-vs-structure mismatch). I will verify those patterns by re-reading the examples myself before including them in the evaluation report.

---

*Last updated: Milestone 2 — before data collection begins.*
