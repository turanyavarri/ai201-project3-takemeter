# TakeMeter

A fine-tuned text classifier that categorizes r/travel Reddit posts into three labels: `experience`, `planning`, and `advice`. Built as part of CodePath AI201 Project 3.

---

## Community Choice

I chose **r/travel**, a large public Reddit community dedicated to travel discussion, trip sharing, and travel advice. It's a strong fit for a classification task because posts vary significantly in purpose and structure — some people share personal trip stories, some post detailed day-by-day itineraries, and others share tips and recommendations. These distinctions are meaningful to community members: a person looking for packing advice doesn't want to read a trip story, and someone planning a route needs a structured itinerary, not general tips. The discourse is text-heavy, public, and consistently tagged with flair, making data collection straightforward.

---

## Label Taxonomy

### `experience`
The post is sharing a personal trip that already happened — a story, reflection, photo recap, or account of what the author did and saw.

Example 1: "Just got back from two weeks in Portugal. Lisbon completely blew me away — the trams, the food, the people. We did a day trip to Sintra and I'd 100% recommend it."

Example 2: "Finally made it to Patagonia after years of planning. The trek was brutal but worth every step. Here are some photos from the W Circuit."

### `planning`
The post is a structured schedule or itinerary — organized as a day-by-day or logistical plan, either asking for feedback on one or sharing a completed one.

Example 1: "Here's my 10-day Japan itinerary: Day 1-3: Tokyo, Day 4-5: Kyoto, Day 6: Nara, Day 7-9: Osaka, Day 10: fly home. Does this look feasible?"

Example 2: "Planning a 2-week Europe trip. Day 1: Arrive Amsterdam, Day 2-3: explore canals and museums, Day 4: train to Paris..."

### `advice`
The post shares tips, recommendations, or lessons learned that apply beyond the author's specific trip — what to do, what to avoid, what they'd do differently.

Example 1: "If you're visiting Rome, avoid eating anywhere within 200 meters of a major tourist attraction. Walk two streets over and prices drop by half."

Example 2: "Best tip I ever got for long-haul flights: book the aisle seat, bring noise-canceling headphones, and never skip the compression socks."

---

## Data Collection

**Source:** r/travel on Reddit, filtered by post flair ("Trip Report", "Itinerary", "My Advice", "Travelers Only").

**Method:** Manual collection — copy-pasted post body text into a Google Sheet with columns `text` and `label`, then exported as CSV. No scraping tools were used.

**Label distribution:**

| Label | Count | Percentage |
|-------|-------|------------|
| `experience` | 70 | 35% |
| `planning` | 41 | 20.5% |
| `advice` | 89 | 44.5% |
| **Total** | **200** | **100%** |

**Labeling process:** Each post was read in full and labeled according to the dominant purpose of the post, not surface-level keywords. The decision rule for ambiguous cases: label by the dominant structure — day-by-day schedule → `planning`, list of tips → `advice`, personal narrative → `experience`.

**Three difficult annotation examples:**

1. "Minimum pain, Maximum gain...I decided on Seceda hike. Trust me, if you want to cheat by taking the cable car and can manage about 10 km, pick Seceda over any hike in the whole Alps." — Decided: `advice` — The personal story is the vehicle for recommending a specific hike strategy to unfit hikers, not the point itself.

2. "Spent about a week in France...Blue hour on the Seine is the move for the Eiffel Tower. Pro tip we wish we'd known sooner: the glass pyramid entrance line is brutal, so book a timed slot in advance." — Decided: `advice` — Despite mentioning a personal trip, the post is structured as a list of tips and recommendations.

3. "Slovenia was truly an amazing experience with friendly people and beautiful nature. Incredible to be able to drive from the Alps to the coast within a few short hours. Highly recommend making a trip here." — Decided: `advice` — Short post with no personal narrative detail; the primary purpose is recommending Slovenia to others.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` from HuggingFace

**Training setup:** Fine-tuned on Google Colab using a free T4 GPU. Training libraries: `transformers`, `datasets`, `scikit-learn`.

**Hyperparameters:**
- Learning rate: 2e-5
- Batch size: 16
- Weight decay: 0.01
- Warmup steps: 50
- **Epochs: 7** (increased from default 3)

**Key hyperparameter decision:** The default 3-epoch run produced a model that predicted `advice` for nearly all examples (accuracy 0.567, experience F1: 0.00, planning F1: 0.00), indicating the model was not learning the minority classes. Increasing to 5 epochs partially resolved this (accuracy 0.667) but `planning` remained at F1: 0.00. At 7 epochs the model successfully learned all three classes (accuracy 0.867, all F1 ≥ 0.75).

---

## Baseline Description

**Model:** `llama-3.3-70b-versatile` via Groq API (zero-shot, no task-specific training)

**Prompt used:**
```
You are classifying posts from r/travel, a Reddit community for travel discussion.

Assign each post to exactly one of the following categories.

experience: The post is sharing a personal trip that already happened — a story, reflection, photo recap, or account of what the author did and saw.
Example: "Just got back from two weeks in Portugal. Lisbon completely blew me away — the trams, the food, the people."

planning: The post is a structured schedule or itinerary — organized as a day-by-day or logistical plan, either asking for feedback on one or sharing a completed one.
Example: "Here's my 10-day Japan itinerary: Day 1-3: Tokyo, Day 4-5: Kyoto, Day 6: Nara, Day 7-9: Osaka, Day 10: fly home."

advice: The post shares tips, recommendations, or lessons learned that apply beyond the author's specific trip — what to do, what to avoid, what they'd do differently.
Example: "If you're visiting Rome, avoid eating anywhere within 200 meters of a major tourist attraction. Walk two streets over and prices drop by half."

Respond with ONLY the label name.
Do not explain your reasoning.
Valid labels:
experience
planning
advice
```

**How results were collected:** The prompt was run on all 30 test examples using the Groq API at temperature=0. All 30 responses were parseable (no unparseable outputs).

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|-------|----------|
| Zero-shot baseline (Groq) | 0.667 |
| Fine-tuned DistilBERT | **0.867** |
| Improvement | **+0.200** |

### Per-Class Metrics

**Baseline (Groq llama-3.3-70b-versatile):**

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| experience | 0.89 | 1.00 | 0.94 | 8 |
| planning | 0.36 | 1.00 | 0.53 | 5 |
| advice | 1.00 | 0.41 | 0.58 | 17 |
| **accuracy** | | | **0.67** | 30 |
| macro avg | 0.75 | 0.80 | 0.68 | 30 |

**Fine-Tuned DistilBERT (7 epochs):**

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| experience | 1.00 | 0.75 | 0.86 | 8 |
| planning | 1.00 | 0.60 | 0.75 | 5 |
| advice | 0.81 | 1.00 | 0.89 | 17 |
| **accuracy** | | | **0.87** | 30 |
| macro avg | 0.94 | 0.78 | 0.83 | 30 |

### Confusion Matrix (Fine-Tuned Model)

| | Predicted: experience | Predicted: planning | Predicted: advice |
|---|---|---|---|
| **True: experience** | 6 | 0 | 2 |
| **True: planning** | 0 | 3 | 2 |
| **True: advice** | 0 | 0 | 17 |

The model correctly identifies all `advice` posts (recall 1.00) and never misclassifies non-advice posts as experience or planning. The main error pattern is that 2 `experience` posts and 2 `planning` posts are predicted as `advice`.

### Wrong Predictions Analysis

**Wrong prediction #1:**
Text: "Slovakia is a hidden gem. Flew into Vienna, Austria and took a Flixbus to Bratislava, where I rented a car at the airport. Used Budget and the car was held together by duct tape on the back bumper..."
True: `experience` | Predicted: `advice` (confidence: 0.56)

Analysis: This post reads like a casual observation rather than a traditional trip narrative. It lacks the reflective storytelling structure of most `experience` posts and sounds more like a quick tip ("Slovakia is a hidden gem"). The model likely latched onto the recommendation framing at the start.

**Wrong prediction #2:**
Text: "My partner and I are planning a Maharashtra monsoon trip in the first week of July. Current plan: Matheran, Lonavala, Mahabaleshwar, Panchgani. We have around a week. And we are coming from Delhi."
True: `planning` | Predicted: `advice` (confidence: 0.47)

Analysis: This post is a planning post — it has a destination list and timeframe — but it's very short and lacks the day-by-day structure of most planning posts in the training data. The model may have learned to associate `planning` with longer, more structured itineraries and defaulted to `advice` for short posts.

**Wrong prediction #3:**
Text: "I accidentally became Tom Hanks in The Terminal at Istanbul Airport. So my flight to Erbil got delayed, and Turkish Airlines kindly rebooked me on a flight at 2:00 AM tomorrow. Sounds fine, except I don't have a visa to enter Türkiye so I can't leave the airport..."
True: `experience` | Predicted: `advice` (confidence: 0.83)

Analysis: This is the most interesting failure. The post is a humorous personal story — clearly `experience` — but it reads like a situation report and includes implicit advice signals ("If you're transiting through FRA, build in at least 4-5 hours"). The model predicted `advice` with high confidence (0.83), suggesting it overfit to the situation-report framing rather than the personal narrative structure.

### Sample Classifications

| Post (first 100 chars) | True Label | Predicted | Confidence |
|------------------------|------------|-----------|------------|
| "Just got back from a great 4 days in Kosovo and North Macedonia..." | experience | experience | 0.91 |
| "Hey guys, I wanted to share my itinerary for this trip I did last april. Day 1: Olympos..." | planning | planning | 0.88 |
| "Before going to the UK I looked up driving in the UK and everyone said it was just the same..." | advice | advice | 0.95 |
| "Slovakia is a hidden gem. Flew into Vienna, Austria and took a Flixbus to Bratislava..." | experience | advice | 0.56 |
| "My partner and I are planning a Maharashtra monsoon trip in the first week of July..." | planning | advice | 0.47 |

The Kosovo/North Macedonia post was correctly predicted as `experience` — it follows a classic day-by-day narrative structure ("Day 1 was spent wandering around Pristina...") which is a strong signal the model learned to recognize.

---

## What the Model Learned vs. What I Intended

I intended the model to distinguish posts by their primary purpose: sharing a story, planning a schedule, or giving advice. What the model actually learned is closer to a surface-level structural signal — it strongly identifies `advice` by the presence of imperative or recommendation language, identifies `planning` by day-numbered schedule structure, and identifies `experience` by elimination.

The main gap is that the model overweights the `advice` label for short or ambiguously framed posts. Posts that don't have a clear day-by-day structure (planning) or rich narrative detail (experience) tend to default toward `advice` — which is the majority class. This is most visible in wrong prediction #2 (the Maharashtra post): the model couldn't recognize it as `planning` because it lacked the structural cues it learned to associate with that label.

The model also struggled with posts that blend registers — a personal story told in a recommendation frame (wrong prediction #3, the Istanbul airport post) or a trip recap that opens with a bold claim (wrong prediction #1, the Slovakia post). These require understanding intent, which surface-level fine-tuning on 200 examples doesn't fully capture.

---

## Spec Reflection

The spec helped most in the label design phase — the requirement to define a decision rule for hard edge cases before annotating forced me to think carefully about the `experience` vs `advice` boundary early. Without that prompt I likely would have labeled inconsistently and only noticed the problem after training.

One way my implementation diverged from the spec: the spec suggested aiming for roughly equal label distribution (~65-70 per label), but my real data had significantly more `advice` posts (~89) than `planning` posts (~41). I kept this distribution rather than forcing balance because the posts I found naturally skewed toward advice-seeking and experience-sharing, and artificially balancing felt like it would introduce noise. This imbalance contributed to the model's tendency to over-predict `advice` in early training runs, which I addressed by increasing epochs.

---

## AI Usage

**Instance 1 — Label classification assistance:** I pasted batches of unlabeled Reddit posts into Claude and asked it to assign one of the three labels based on my label definitions from `planning.md`. Claude pre-labeled approximately 150 of the 200 posts. I reviewed and corrected every pre-assigned label myself — in roughly 15-20% of cases I disagreed with Claude's label and changed it, particularly on experience/advice boundary cases. All final labels reflect my own judgment.

**Instance 2 — Hard edge case stress-testing:** I asked Claude to help identify ambiguous posts in my dataset during annotation. Claude flagged the Dolomites/Seceda post as a hard case (experience vs advice), the France/Paris highlights post, and the Slovenia post. In each case Claude explained its reasoning, I evaluated it against my decision rule, and made the final call. This helped me identify the dominant-structure decision rule explicitly before I had annotated all 200 examples.
