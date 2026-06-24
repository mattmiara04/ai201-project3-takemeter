# ai201-project3-takemeter
# TakeMeter — Clash Royale Discourse Classifier

TakeMeter is a fine-tuned text classifier that labels Clash Royale community posts and comments based on the type and quality of the take. The goal is not just to decide whether a post is “good” or “bad,” but to classify what kind of discourse the post represents.

For this project, I focused on Clash Royale community discussion because the community has a wide range of post types: strategy advice, balance opinions, complaints, rage posts, and meme-style reactions.

---

## Community Choice

I chose the Clash Royale community because it is very opinion-heavy and has active discussion around cards, decks, balance changes, ladder experiences, and game updates.

This community works well for a discourse classifier because not every post has the same level of reasoning. Some posts explain card interactions, elixir trades, matchups, cycle management, and win conditions. Other posts are simple opinions, unsupported complaints, or emotional reactions. These differences are meaningful to people who actually participate in the Clash Royale community.

---

## Label Taxonomy

The classifier uses four labels:

1. `strategy_analysis`
2. `reasoned_opinion`
3. `unsupported_complaint`
4. `reaction_or_meme`

Each post belongs to exactly one label.

---

### `strategy_analysis`

A post belongs to `strategy_analysis` when it makes a clear claim and supports it with specific Clash Royale gameplay reasoning. This can include card interactions, elixir trades, counters, cycle management, matchups, win conditions, placements, or meta trends.

Examples:

* “Hog EQ is strong because Earthquake removes building value while Hog still gets chip damage, and the fast cycle lets you punish expensive decks before they reset.”
* “Against X-Bow, save a tank or high-health card for the lock and avoid overspending before they set up a defensive Tesla.”

---

### `reasoned_opinion`

A post belongs to `reasoned_opinion` when it gives an opinion about a card, deck, balance change, feature, or player experience and includes at least one understandable reason. The reasoning is usually general rather than deeply strategic.

Examples:

* “Firecracker needs a small nerf because she gives too much defensive value for only three elixir.”
* “Clan Wars needs better rewards because it takes time every week and does not feel worth the effort.”

---

### `unsupported_complaint`

A post belongs to `unsupported_complaint` when it complains, makes a strong claim, or calls something broken, unfair, trash, rigged, or overpowered without giving a clear reason or explanation.

Examples:

* “Mega Knight takes no skill and ruins every match.”
* “This game is rigged and matchmaking is fake.”

---

### `reaction_or_meme`

A post belongs to `reaction_or_meme` when it is mainly emotional, joking, sarcastic, celebratory, or meme-like rather than trying to make an argument.

Examples:

* “Me after losing to 2.6 Hog for the fifth time today.”
* “Average midladder match: Mega Knight at the bridge again.”

---

## Dataset

The dataset is stored in:

```text
labeled_dataset.csv
```

It contains 200 labeled examples with the columns:

```text
text,label,notes
```

The label distribution is balanced:

| Label                   | Count |
| ----------------------- | ----: |
| `strategy_analysis`     |    50 |
| `reasoned_opinion`      |    50 |
| `unsupported_complaint` |    50 |
| `reaction_or_meme`      |    50 |

No label accounts for more than 70% of the dataset, which helps prevent the model from simply predicting the majority class.

---

## Data Source and Labeling Process

The dataset is mixed-source and AI-assisted. I used public Clash Royale community discourse, including Reddit-style posts, public community examples, and common Clash Royale discussion patterns, as the basis for the examples. Some rows are based on public community text or screenshots I reviewed, while the remaining rows are AI-assisted, source-informed examples written to match the same Clash Royale discourse categories.

I manually reviewed the labels against my taxonomy before training. The important part of the labeling process was deciding what the post was doing:

* Is it explaining specific gameplay mechanics? → `strategy_analysis`
* Is it giving an opinion with at least one reason? → `reasoned_opinion`
* Is it complaining without support? → `unsupported_complaint`
* Is it joking, reacting, or meme-like? → `reaction_or_meme`

---

## Difficult Labeling Examples

### Example 1

Text:

```text
Unpopular opinion: She needs a nerf. This card is the textbook definition of a bail out. People love to talk about low-skill evos like witch and mega knight, but why is Valkyrie never included in this conversation? I'm just tired of seeing this card every meta.
```

Final label:

```text
unsupported_complaint
```

Reason:

This post complains that Valkyrie needs a nerf, but it does not explain specific mechanics, counters, elixir trades, or matchups. Because it is mostly a strong complaint without clear support, I labeled it as `unsupported_complaint`.

---

### Example 2

Text:

```text
Can someone explain how to beat Hog 2.6/3.0 and X-Bow 3.0? Against Hog: They cycle so fast that my main counter isn't back in hand in time, and they always get chip damage. Against X-Bow: Once they lock onto my tower and set up a defensive wall, it feels impossible to break through.
```

Final label:

```text
strategy_analysis
```

Reason:

This post discusses specific matchup problems, cycle speed, counters not being in hand, chip damage, X-Bow lock-on, and defensive setup. Even though it is asking for help, it includes enough gameplay reasoning to count as `strategy_analysis`.

---

### Example 3

Text:

```text
Firecracker needs a small nerf because she gives too much defensive value for only three elixir.
```

Final label:

```text
reasoned_opinion
```

Reason:

This gives an opinion and a reason, but the reasoning is general. It does not explain detailed interactions, matchups, placements, or counterplay. That makes it `reasoned_opinion`, not `strategy_analysis`.

---

## Fine-Tuning Approach

I fine-tuned:

```text
distilbert-base-uncased
```

The notebook used a 70% / 15% / 15% split:

| Split      | Examples |
| ---------- | -------: |
| Train      |      140 |
| Validation |       30 |
| Test       |       30 |

The training setup used the starter notebook defaults:

| Hyperparameter   |                     Value |
| ---------------- | ------------------------: |
| Epochs           |                         3 |
| Learning rate    |                      2e-5 |
| Train batch size |                        16 |
| Eval batch size  |                        32 |
| Weight decay     |                      0.01 |
| Warmup steps     |                        50 |
| Base model       | `distilbert-base-uncased` |

I kept these defaults because the dataset is small, and increasing epochs too much could overfit the model to the 140 training examples.

---

## Baseline Classifier

For the zero-shot baseline, I used Groq with:

```text
llama-3.3-70b-versatile
```

The baseline prompt defined all four labels, gave one example for each label, and instructed the model to output only one valid label name.

The baseline and fine-tuned model were evaluated on the same 30-example test set.

---

## Results Summary

| Model                   | Accuracy |
| ----------------------- | -------: |
| Zero-shot baseline Groq |    0.900 |
| Fine-tuned DistilBERT   |    0.633 |

Fine-tuning produced a regression of:

```text
0.267
```

This means the fine-tuned DistilBERT model performed worse than the zero-shot Groq baseline.

This result makes sense because the Groq model already has strong language understanding and can follow the label definitions directly. DistilBERT had only 140 training examples, which is a small dataset for learning subtle discourse boundaries.

---

## Fine-Tuned Model Metrics

| Label                   | Precision | Recall |        F1 | Support |
| ----------------------- | --------: | -----: | --------: | ------: |
| `strategy_analysis`     |      0.70 |   0.88 |      0.78 |       8 |
| `reasoned_opinion`      |      0.00 |   0.00 |      0.00 |       7 |
| `unsupported_complaint` |      0.53 |   1.00 |      0.70 |       8 |
| `reaction_or_meme`      |      0.80 |   0.57 |      0.67 |       7 |
| **Accuracy**            |           |        | **0.633** |      30 |

The fine-tuned model performed best on `strategy_analysis` and `unsupported_complaint`. It completely failed on `reasoned_opinion`, which became the most important error pattern.

---

## Baseline Groq Metrics

| Label                   | Precision | Recall |        F1 | Support |
| ----------------------- | --------: | -----: | --------: | ------: |
| `strategy_analysis`     |      1.00 |   1.00 |      1.00 |       8 |
| `reasoned_opinion`      |      1.00 |   0.71 |      0.83 |       7 |
| `unsupported_complaint` |      0.73 |   1.00 |      0.84 |       8 |
| `reaction_or_meme`      |      1.00 |   0.86 |      0.92 |       7 |
| **Accuracy**            |           |        | **0.900** |      30 |

The baseline did much better because it could directly use the written label definitions in the prompt.

---

## Confusion Matrix

Fine-tuned model confusion matrix:

| True Label ↓ / Predicted Label → | `strategy_analysis` | `reasoned_opinion` | `unsupported_complaint` | `reaction_or_meme` |
| -------------------------------- | ------------------: | -----------------: | ----------------------: | -----------------: |
| `strategy_analysis`              |                   7 |                  0 |                       0 |                  1 |
| `reasoned_opinion`               |                   3 |                  0 |                       4 |                  0 |
| `unsupported_complaint`          |                   0 |                  0 |                       8 |                  0 |
| `reaction_or_meme`               |                   0 |                  0 |                       3 |                  4 |

The confusion matrix shows that the fine-tuned model never predicted `reasoned_opinion` on the test set. Instead, it pushed those examples into either `strategy_analysis` or `unsupported_complaint`.

This means the model learned the stronger labels better than the middle category. It could recognize detailed strategy and obvious complaints, but struggled with general opinions that had some reasoning.

The generated image is saved as:

```text
confusion_matrix.png
```

---

## Error Analysis

The fine-tuned model made 11 wrong predictions out of 30 test examples.

### Wrong Prediction 1

Text:

```text
Mortar cycle needs constant pressure because defensive Mortars force awkward responses and give time to outcycle their main win condition.
```

True label:

```text
strategy_analysis
```

Predicted label:

```text
reaction_or_meme
```

Analysis:

This is clearly strategic because it talks about pressure, defensive Mortars, awkward responses, and outcycling a win condition. The model likely failed because the sentence is short and does not include obvious keywords like “elixir” or “counter.” This shows that the model did not fully learn strategic meaning; it may have relied too much on surface patterns.

---

### Wrong Prediction 2

Text:

```text
Evo Knight is still good because his damage reduction makes him reliable on defense.
```

True label:

```text
reasoned_opinion
```

Predicted label:

```text
unsupported_complaint
```

Analysis:

This is a reasoned opinion because it gives a claim and a simple reason. The model predicted `unsupported_complaint`, possibly because the sentence is short and contains a card judgment. This shows the model struggled to separate “short opinion with a reason” from unsupported statements.

---

### Wrong Prediction 3

Text:

```text
Me watching my tower disappear because I ignored one Balloon.
```

True label:

```text
reaction_or_meme
```

Predicted label:

```text
unsupported_complaint
```

Analysis:

This is a meme-style reaction because it is written as an emotional joke. The model predicted `unsupported_complaint`, likely because it saw negative game outcome language like “tower disappear” and treated it as a complaint. This shows the model struggled with humorous or sarcastic framing.

---

## Error Pattern Analysis

The biggest error pattern is that the fine-tuned model over-predicted `unsupported_complaint` and completely failed to predict `reasoned_opinion`.

This happened because `reasoned_opinion` sits between two stronger categories:

* It can look like `strategy_analysis` when it includes gameplay words.
* It can look like `unsupported_complaint` when it is short and negative.

The model learned the obvious ends of the taxonomy better than the subtle middle. In a future version, I would add more examples of `reasoned_opinion`, especially short posts that include one clear reason but are not detailed enough to be `strategy_analysis`.

---

## Reflection: What the Model Learned vs. What I Intended

I intended the model to learn a four-level distinction between detailed strategy, general reasoned opinion, unsupported complaint, and meme-style reaction.

The model partially learned this. It performed well on `strategy_analysis` and `unsupported_complaint`, which are more distinct. It also did moderately well on `reaction_or_meme`.

However, it did not learn `reasoned_opinion` well. The model treated reasoned opinions as either strategy posts or complaints. This suggests that my label boundary between `reasoned_opinion` and the other labels was still too subtle for a small fine-tuning dataset.

The model captured some discourse quality signals, but it did not fully capture the intended middle category.

---

## Spec Reflection

Writing the planning document before collecting the dataset helped because it forced me to define the labels clearly before training. The hard edge cases were especially useful because they made me think about what separates a general opinion from a strategic analysis.

One way the implementation diverged from my original plan is that the dataset became mixed-source and AI-assisted. I originally planned to collect more direct public examples manually, but finding enough clean short examples was more difficult than expected. I still manually reviewed the dataset against the label definitions to keep the labels consistent.

---

## AI Usage Transparency

I used AI tools in multiple parts of this project.

First, I used AI to help stress-test my label taxonomy. I checked whether the labels had clear boundaries and whether examples could reasonably be assigned to exactly one label.

Second, I used AI assistance to help expand the dataset into balanced Clash Royale-style examples. Some examples were based on public community posts or screenshots I reviewed, while others were AI-assisted and source-informed. I manually reviewed the labels before training.

Third, I used AI to help interpret the model results and identify error patterns. I verified the conclusions myself by reading the confusion matrix and wrong predictions.

I did not use AI results without checking them. I reviewed the labels, ran the notebook, compared the fine-tuned model against the baseline, and used the actual metrics in this report.

---

## Files Included

```text
planning.md
README.md
labeled_dataset.csv
evaluation_results.json
confusion_matrix.png
```

---

## How to Reproduce

1. Open the starter Colab notebook.
2. Set runtime to T4 GPU.
3. Upload `labeled_dataset.csv`.
4. Use this label map:

```python
LABEL_MAP = {
    "strategy_analysis": 0,
    "reasoned_opinion": 1,
    "unsupported_complaint": 2,
    "reaction_or_meme": 3,
}
```

5. Run Sections 1–6 of the notebook.
6. Download `evaluation_results.json` and `confusion_matrix.png`.
7. Commit both files to the GitHub repo.

---

## Demo Video Notes

In the demo video, I show:

* The label taxonomy and dataset.
* The fine-tuned model results.
* The baseline comparison.
* The confusion matrix.
* At least one correct prediction.
* At least one incorrect prediction and why it failed.
