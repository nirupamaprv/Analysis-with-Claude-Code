# Zomato Dataset Assessment — Session Summary

- **Date:** 25 Sep 2026, 20:45–21:51 UTC (about 66 minutes)
- **Tool:** Claude Code, model Claude Opus 5.5 (`claude-opus-5-5`)
- **Files:** `Zomato Restaurant Dataset.csv` (input), `zomato_assessment.ipynb` (output notebook; re-runs start to finish with no errors)

---

## 1. How the session ran

| Phase | Permission mode | What happened |
|---|---|---|
| Start | Auto | Asked Claude to list the permission modes, then asked for plan mode. |
| Planning | Plan (read-only) | Claude read the CSV and asked clarifying questions. The first plan was rejected with a question and the second was approved. |
| Build and all follow-ups | Default (manual approval of each action) | Python environment setup, notebook build, Parts 4 and 5. |

**Installs:** after the plan was approved, Claude created a project virtual environment (`python3 -m venv .venv`) and installed `pandas numpy matplotlib seaborn scipy statsmodels jupyter nbformat`. The session log records the mode only on user messages, and shows no switch back to auto for this step.

## 2. Token usage (from the session log)

| Type | Tokens |
|---|---|
| Output | 186,049 |
| Input (not cached) | 214 |
| Cache writes | 229,196 |
| Cache reads | 9,183,156 |
| **Total** | **≈ 9.6 million** (mostly cache reads, which are billed at a lower rate) |

These figures don't include the later session that wrote this summary.

## 3. Clarifying questions Claude asked, and the answers

1. **What should be planned?** → "assess Zomato data set"
2. **What should the assessment produce?** → A Jupyter notebook. Claude had recommended an HTML report.
3. **What is the main goal?** → Both, in order: data quality and EDA first, then what drives ratings.
4. **First plan rejected with a question:** *"what is broken currency strings"*. Claude showed the raw bytes. The UK "Pounds(£)" label is mojibake: the £ sign was saved with the wrong encoding and came out garbled. The plan was then approved.
5. **Which questions should the final section cover?** The request to add a Q&A section arrived without the questions. The reply was "I'll paste my list", and the list followed.

## 4. Notebook contents

### Part 1: Data quality
- 9,551 rows, of which 7,403 are rated. A rating of 0 means "not rated" (22% of rows) and is left out of the rating analysis.
- Costs are in 12 currencies, so an approximate USD column was added using fixed exchange rates.
- The Philippines rows are labelled "Botswana Pula"; this was corrected. The broken £ sign was fixed.
- Accented letters were lost when the file was saved and can't be recovered. Brasília, São Paulo and İstanbul were fixed by hand.
- Smaller issues: 499 restaurants have coordinates of 0, 18 have a cost of 0, and 9 have no cuisine. One column has the same value in every row and was dropped.
- The data is heavily skewed: India is 91% of rows.

### Part 2: EDA
- Charts of where restaurants are, and distributions of rating, votes, cost and price range.
- Top cuisines, and how common online delivery and table booking are.
- A check that the rating labels (Poor … Excellent) match the numeric ratings.
- A map of restaurant locations.

### Part 3: What goes with higher ratings
These are associations, not proof of cause.

| Factor | Finding |
|---|---|
| Votes | Strongest link (correlation 0.68). It likely runs both ways: good places attract more reviews. |
| Price range | The most expensive tier rates +0.35 stars above the cheapest, or +0.12 once votes are accounted for. |
| Location | Delhi-area cities rate 0.5–0.7 stars lower than other cities. |
| Main cuisine | North Indian, Chinese and pizza places rate about 0.2 stars lower. |
| Table booking | +0.17 stars on its own, but none once votes are accounted for. |
| Online delivery | −0.09 stars on the raw numbers, but +0.10 once price, location and cuisine are accounted for. |

The model explains 43% of the variation in rating (R² = 0.43), or 58% when votes are included.

**Correction during the session:** Claude first wrote the delivery finding by hand and got it wrong. The conclusions cell now takes its numbers straight from the model.

## 5. Part 4: Questions from the dataset

### The questions, as asked
1. Which city has the most highly rated restaurants and cafés?
2. The top 10 restaurants by rating, with chains merged so they don't repeat, and the same list limited to India.
3. For India, the most highly rated cuisine, based on the best-restaurant analysis.
4. Is India's most preferred cuisine (top 3) different from the cuisines in the top 10? The reasoning given: restaurant ratings depend on price, locality and consistency, while cuisine depends on preference.
5. Any other observations?

### Nuances requested, and how they were handled
- **Merge chains.** Each chain is rated as a vote-weighted average across its branches. The ranking gives little weight to ratings with only a few votes, so a 4.9 from 8 votes can't outrank a 4.8 from 5,000.
- **Separate India list.** Every ranking was produced worldwide and again for India only.
- **Cuisine judged by the best restaurants.** Q3 compares how often each cuisine appears among India's top 50 with how often it appears overall.
- **The hypothesis in Q4** was tested directly: cost, number of branches and cuisine popularity for the top 10.
- **Preference measured two ways:** by how many restaurants offer a cuisine and by how many votes they get.

### Answers
- **Q1:**
  - New Delhi has the most restaurants rated 4.5 or higher (28), but that's only 0.7% of its 4,048 rated restaurants.
  - London has the highest share (15 of 20).
  - New Delhi has the most highly rated cafés (9), then Bangalore (4).
- **Q2:**
  - Worldwide, Talaga Sampireun (Jakarta) is first, and six of the top ten are in the US.
  - India's top 10 is Mirchi And Mime, Indian Accent, Grandson of Tunday Kababi, Naturals Ice Cream, Toit, Sagar Gaire Fast Food, Masala Library, Le Plaisir, Prankster and Spice Kraft.
  - The best chains with 3 or more branches are AB's Absolute Barbecues, Chili's and Barbeque Nation.
- **Q3:**
  - Mediterranean is top on paper, but 8 of its 12 excellent restaurants are branches of AB's and Barbeque Nation, so it isn't reliable.
  - The more reliable answer is Mexican, Asian, American and European, which each appear 4–5 times more often among India's top 50 than among Indian restaurants overall.
- **Q4: yes, they differ.**
  - Most offered: North Indian, Chinese, Fast Food. Most voted: North Indian, Chinese, Italian.
  - Chinese, the #2 preference, isn't in India's top 10.
  - The hypothesis mostly holds:
    - The top 10 cost about 2.5 times the Indian average.
    - Nine of the ten are single-site restaurants.
    - Popular cuisines rate below average.
  - Price isn't everything: three of the ten are cheap specialists.
- **Q5, other observations:**
  1. Only the Delhi area is fully listed. Other cities have about 20 listings each, which look hand-picked. That makes comparisons between cities unreliable.
  2. A rating appears only after 4 votes.
  3. Chain restaurants rate slightly lower than independents (3.35 vs 3.48).
  4. Table booking is almost only offered by expensive restaurants.
  5. The "delivering now" column only reflects the moment the data was collected, so it tells you nothing about the restaurant.

## 6. Part 5: NCR (Delhi area) influence

**Question raised:** How big is NCR's share of the data? North Indian (Delhi/Punjabi) and Chinese (momos) are favourites in this metro, so is that influencing the answers to all five questions?

**Findings**
- **Size:** NCR is 83% of the dataset, 92% of Indian restaurants and 71% of Indian votes, but only 37% of India's excellent-rated restaurants.
- **Momos:** Zomato has no momo tag. Momo shops are filed under Chinese or Tibetan, so they can't be separated from Chinese.
- **Favourite cuisines:**
  - North Indian and Chinese lead both inside and outside NCR.
  - NCR adds Fast Food and Mughlai at #3 and #4.
  - Outside NCR, Continental, Italian and cafés take those places.
- **The same cuisine rates 0.4–0.8 stars higher outside NCR.** That comes from how the data was collected (every NCR restaurant is listed, but only about 20 hand-picked ones per city elsewhere), not from differences in taste or food.
- **Q1–Q4 re-run inside NCR only.** This is the fairest test, because NCR is the only fully listed region.
  - **Q1:** New Delhi has the most excellent restaurants (28). Gurgaon has the highest share rated 4.0 or higher (10.7%).
  - **Q2:** Indian Accent, Naturals Ice Cream, Masala Library, Prankster and Caterspoint lead.
  - **Q3:** European (40%), Asian (39%) and Mexican (32%) restaurants rate 4.0 or higher far more often than North Indian (6%), Chinese (5%) and Fast Food (4%).
  - **Q4:** Chinese appears once in NCR's top 10, as part of Pa Pa Ya's pan-Asian menu. This corrects the earlier statement that Chinese was absent.

**How NCR affects each original answer**

| Q | Effect |
|---|---|
| Q1 | Not a fair comparison across cities. |
| Q2 | Under-represents NCR. |
| Q3 | The direction holds, but the size was overstated. |
| Q4 | Holds, with nuance. |

**Conclusion:** the hypothesis holds inside NCR, where the sampling problem doesn't apply. Ratings reflect the restaurant (price, locality and consistency), while cuisine choice reflects preference.

## 7. How to reproduce

```bash
python3 -m venv .venv
.venv/bin/pip install pandas numpy matplotlib seaborn scipy statsmodels jupyter nbformat
.venv/bin/jupyter nbconvert --to notebook --execute --inplace zomato_assessment.ipynb
```
