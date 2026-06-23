# TakeMeter — Planning Document

## Community Choice

For this project, I chose the Clash Royale community. I am focusing on posts and comments that discuss cards, decks, balance changes, matchups, ladder experiences, and general reactions to the game.

Clash Royale is a good community for this classification task because people constantly share different kinds of takes. Some posts are detailed strategic arguments about card interactions, elixir trades, matchups, or the current meta. Other posts are opinions with some reasoning, quick complaints, or emotional meme-style reactions after losing games. This makes the community varied enough for a discourse-quality classifier.

The goal of TakeMeter is not just to label a post as “good” or “bad.” Instead, the model should classify what type of take the post is making and how much reasoning it contains.

---

## Label Taxonomy

The classifier will use four labels:

1. `strategy_analysis`
2. `reasoned_opinion`
3. `unsupported_complaint`
4. `reaction_or_meme`

Each post or comment should belong to exactly one label.

---

### Label 1: `strategy_analysis`

**Definition:**
A post belongs to `strategy_analysis` when it makes a clear claim and supports it with specific Clash Royale reasoning. This reasoning may involve card interactions, elixir trades, deck archetypes, win conditions, counters, cycle management, matchups, balance effects, or meta trends.

**Clear examples:**

Example 1:
“Hog EQ is strong because it pressures buildings while still getting chip damage, and the fast cycle lets it punish expensive decks before they can reset.”

Example 2:
“Phoenix works well in beatdown decks because it forces the opponent to spend extra elixir on the egg, which gives your tank more time to cross the bridge.”

**Boundary rule:**
This label should only be used when the post includes specific game reasoning. A post saying “Hog EQ is good because it wins a lot” is too general and should usually be `reasoned_opinion`, not `strategy_analysis`.

---

### Label 2: `reasoned_opinion`

**Definition:**
A post belongs to `reasoned_opinion` when it gives an opinion about a card, deck, balance change, feature, or player experience and includes at least one understandable reason. The reasoning can be general, personal, or based on experience, but it is not detailed enough to count as strategy analysis.

**Clear examples:**

Example 1:
“I think Mega Knight needs a small nerf because lower ladder players struggle to defend him and he gets too much value on defense.”

Example 2:
“Clan Wars needs better rewards because right now it feels like a chore compared to ladder or challenges.”

**Boundary rule:**
If the post gives a reason but does not explain specific game mechanics, matchups, elixir trades, or deck interactions, it should usually be `reasoned_opinion`.

---

### Label 3: `unsupported_complaint`

**Definition:**
A post belongs to `unsupported_complaint` when it complains, makes a strong claim, or calls something broken, unfair, trash, or overpowered without giving a clear reason or explanation.

**Clear examples:**

Example 1:
“Firecracker is stupid and should be deleted from the game.”

Example 2:
“This game is completely rigged. Every match is unfair.”

**Boundary rule:**
This label is for complaints or claims with little to no explanation. If the post gives even one clear reason, it may belong in `reasoned_opinion` instead.

---

### Label 4: `reaction_or_meme`

**Definition:**
A post belongs to `reaction_or_meme` when it is mainly emotional, joking, sarcastic, celebratory, or meme-like. The post is not really trying to make an argument, even if it references a real game situation.

**Clear examples:**

Example 1:
“Me after losing to 2.6 hog for the fifth time today ”

Example 2:
“When your opponent drops Mega Knight at the bridge and you forgot your counter ”

**Boundary rule:**
If the main purpose is humor, reaction, sarcasm, or emotional expression, use `reaction_or_meme`. If the post is complaining seriously without humor or a meme format, use `unsupported_complaint`.

---

## Hard Edge Cases

Some posts will be difficult to label because they sit between two categories.

### Edge Case 1: Complaint with one weak reason

Example:
“Firecracker is broken because she always gets value.”

Possible labels:

* `unsupported_complaint`
* `reasoned_opinion`

Decision:
If the reason is extremely vague, like “always gets value,” I will label it as `unsupported_complaint`. If the post explains why she gets value, such as splash damage, long range, or forced spell usage, I will label it as `reasoned_opinion` or `strategy_analysis`.

---

### Edge Case 2: Opinion with gameplay terms but little explanation

Example:
“Hog EQ is too strong in this meta because buildings are bad right now.”

Possible labels:

* `reasoned_opinion`
* `strategy_analysis`

Decision:
This should be `reasoned_opinion` unless the post explains the interaction more clearly. To be `strategy_analysis`, it needs more specific reasoning about matchups, cycle, elixir trades, buildings, or win conditions.

---

### Edge Case 3: Meme that also contains a real complaint

Example:
“Me watching my opponent play E-Giant at the bridge and somehow win anyway ”

Possible labels:

* `reaction_or_meme`
* `unsupported_complaint`

Decision:
If the format is mainly a joke or reaction, I will label it as `reaction_or_meme`. If the post is written as a serious complaint without a joke structure, I will label it as `unsupported_complaint`.

---

### Edge Case 4: Short but valid strategic comment

Example:
“Save Log for Barrel or you lose the matchup.”

Possible labels:

* `strategy_analysis`
* `reasoned_opinion`

Decision:
Even though this is short, it gives specific matchup advice. I will label it as `strategy_analysis` because it identifies a clear tactical rule.

---

## Data Collection Plan

I will collect at least 200 Clash Royale posts or comments from public online sources. The main source will be public Clash Royale community discussion, especially Reddit-style posts or comments about cards, decks, balance, gameplay, and player reactions.

I will save the dataset as a CSV file named:

`labeled_dataset.csv`

The CSV will have three columns:

```csv
text,label,notes
```

* `text`: the post or comment text
* `label`: one of the four labels
* `notes`: a short explanation of why I chose that label, especially for difficult examples

I will aim for a balanced dataset so that no single label dominates the model. My target distribution is:

* `strategy_analysis`: about 50 examples
* `reasoned_opinion`: about 50 examples
* `unsupported_complaint`: about 50 examples
* `reaction_or_meme`: about 50 examples

This gives a total of at least 200 examples. If one label is underrepresented, I will collect more examples for that specific label before training. I will avoid having any label account for more than 70% of the dataset.

I will label each example myself using the definitions above. I will not skim quickly or label based only on keywords. For each example, I will ask what the post is actually doing: giving strategic reasoning, giving a general opinion, complaining without support, or reacting/joking.

---

## Evaluation Metrics

I will evaluate both the baseline model and the fine-tuned model on the same test set.

The main metrics I will use are:

1. Overall accuracy
2. Per-class precision
3. Per-class recall
4. Per-class F1 score
5. Confusion matrix

Accuracy alone is not enough because the dataset has multiple labels and the model could perform well on one label while failing on another. Per-class metrics matter because I need to know which labels the model understands and which labels it confuses.

The confusion matrix will be especially important because it will show which labels are mixed up. For example, if the model often predicts `reasoned_opinion` when the true label is `strategy_analysis`, that means the model is struggling to recognize the difference between general reasoning and specific gameplay analysis.

---

## Definition of Success

A successful model should do better than the zero-shot Groq baseline on the same test set. It should not just memorize the training examples; it should learn the difference between the four discourse types.

My target for “good enough” performance is:

* The fine-tuned model beats the baseline overall accuracy.
* No single label completely fails.
* At least most per-class F1 scores are reasonable, ideally around 0.65 or higher.
* The confusion matrix shows understandable mistakes instead of random guessing.
* The model can correctly classify several new Clash Royale posts with visible confidence scores.

I would consider the model useful if it can usually separate detailed strategy posts from unsupported complaints and meme-style reactions. I expect the hardest boundary to be between `strategy_analysis` and `reasoned_opinion`, because both can include reasoning.

---

## AI Tool Plan

I will use AI tools in three specific ways.

### 1. Label stress-testing

Before labeling all 200 examples, I will use ChatGPT to stress-test my label definitions. I will give it the four labels, definitions, and several sample Clash Royale posts. I will check whether the labels are clear enough and revise the definitions if examples are too ambiguous.

I will not let AI replace my own labeling decisions. The goal is to find weak boundaries before I spend time labeling the whole dataset.

---

### 2. Annotation assistance

I may use an AI tool to help pre-label some examples, but I will review every label myself. If I use AI pre-labeling, I will treat it as a first guess only. I will correct labels that do not match my definitions and document this in the README.

For difficult examples, I will write notes explaining why I chose the final label.

---

### 3. Failure analysis

After fine-tuning, I will use AI to help look for patterns in the model’s wrong predictions. I will give it the misclassified examples and ask it to identify possible error patterns, such as confusing `reasoned_opinion` with `strategy_analysis` or misreading sarcastic meme posts.

I will verify these patterns myself by re-reading the examples. I will only include patterns in the final report if they are actually supported by the data.

---

## Evaluation Report Plan

In the final README, I will include:

* Community choice and reasoning
* Label definitions with examples
* Data source and labeling process
* Label distribution
* Difficult examples and the final labeling decision
* Fine-tuning approach
* Baseline approach
* Metrics for both models
* Confusion matrix
* At least three wrong predictions with analysis
* Reflection on what the model learned versus what I intended it to learn
* Spec reflection
* AI usage transparency

---

## Expected Challenges

The biggest challenge will be keeping the labels consistent. Some Clash Royale posts are short, sarcastic, or written with slang, which can make them hard to classify.

The most difficult label boundary will likely be between `strategy_analysis` and `reasoned_opinion`. Both labels include reasoning, but `strategy_analysis` requires specific Clash Royale mechanics or matchup logic, while `reasoned_opinion` can be more general.

Another challenge will be distinguishing `unsupported_complaint` from `reaction_or_meme`. If the post is mainly joking or emotional, I will use `reaction_or_meme`. If it is mostly a serious complaint with no support, I will use `unsupported_complaint`.

---

## Stretch Features Considered

The most realistic stretch feature for this project would be error pattern analysis. After the model runs, I can look for a systematic pattern in the incorrect predictions and describe it in the README.

I am not planning to start with inter-annotator reliability or a deployed interface until the required project is complete.
