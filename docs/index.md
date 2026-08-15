# Capstone Report — Content Refresh Prioritization

- **Author:** Piyush Singh
- **Lane:** Machine Learning — Content Refresh Prioritization
- **Repo:** https://github.com/PiyushSingh0615/flyrank-ml--Internship
- **Date:** August 2026

## 1. Problem framing

### Decision supported

This project supports the decision of **which content pages should receive
human review first for possible improvement or refresh**.

The intended output is a ranked action queue containing a score, reason code,
and suggested action.

The unit of analysis is approximately:

**one content page × one client × one report date**

The output is a ranked prioritization signal rather than a final content
decision.

A human reviewer can use the output to ask:

- Which pages should we inspect first?
- What measurable signal caused the page to be prioritized?
- What type of review should be considered?

The cost of a wrong call is that a team may spend time improving a page that
does not need intervention, overlook a page that deserves attention, or
interpret a weak signal as evidence that a particular content change will
improve future performance.

### Why data/ML helps

Content teams may have many pages to review. Search and engagement signals can
help organize this workload by identifying pages with measurable patterns
worth investigating.

The project therefore treats ML as **decision-support**, not as an automatic
content optimization system.

The final workflow is:

**observed signals → prioritization → human review → content decision**

rather than:

**observed signals → automatic content change**

---

## 2. Data safety

### Data used

The analysis uses the **FlyRank ML Internship dataset**, specifically the
warehouse sample used throughout the internship notebooks.

The working analysis uses June 2026 observations where Google Search Console
data is available.

The analytical grain is maintained at the content/client/date level, with
duplicate observations removed at:

`report_date × client_hash_id × content_hash_id`

The principal signals used by the model are:

- `gsc_impressions`
- `gsc_clicks`
- `gsc_avg_position`
- `ga4_sessions`
- `ga4_engaged_sessions`

### Deliberately excluded fields

`client_hash_id` and `content_hash_id` are identifiers only. They are retained
for grouping, traceability, and error review but are **not model features**.

`report_date` is used to construct the time-aware validation split but is not
used as a predictive feature.

Future-window or label-derived fields are not used as model features.

The leakage audit specifically considered fields such as:

- `trend_direction`
- `trend_pct`

because fields representing future change or derived outcomes could leak
information that would not be available at decision time.

### Leakage framing

The final feature set does not contain obvious future-window or
label-derived features.

However, the audit does not claim that every possible leakage mechanism has
been mathematically eliminated. It establishes that the selected feature set
avoids the obvious leakage risks identified during the internship.

### Public safety

The research artifact uses hashed identifiers and aggregate/model-level
findings.

No client names, private URLs, or private search queries are intentionally
included in the research report.

---

## 3. Baseline

The project began with a transparent rule-based baseline rather than starting
with machine learning.

The Week-4 baseline action score is:

- 40% from `gsc_impressions`
- 20% from `gsc_clicks`
- 20% from `ga4_sessions`
- 10% from `ga4_engaged_sessions`
- an additional position component:
  - position 1–10 → 10 points
  - position 11–20 → 5 points
  - otherwise → 0 points

Missing numeric signals are handled explicitly before scoring.

The baseline was chosen because it is simple, transparent, reproducible, and
easy for a human reviewer to understand.

It provides a fair reference for the model because the Random Forest uses the
same five observed search and engagement signals and is evaluated against the
same baseline score.

### Important limitation of the baseline comparison

The baseline score is a **ranking signal**, not a verified future business
outcome.

Therefore, comparing the Random Forest with this score measures how closely
the model reproduces the existing rule.

It does not establish that either method improves future content performance.

---

## 4. Model / analysis

### Method

The capstone model is a **Random Forest regression model**.

Random Forest was selected because the lane contains multiple tabular search
and engagement signals that may interact non-linearly.

The model configuration used in the Week-5 experiment was:

- `n_estimators = 100`
- `max_depth = 10`
- `random_state = 42`
- `n_jobs = -1`

### Feature list

The exact model feature list is:

1. `gsc_impressions`
2. `gsc_clicks`
3. `gsc_avg_position`
4. `ga4_sessions`
5. `ga4_engaged_sessions`

Missing average-position values are filled using the training-data median
before the remaining missing values are filled with zero.

### Target / proxy

The model predicts the **Week-4 baseline action score**.

This is explicitly a **proxy/ranking signal rather than a future outcome
label**.

The experiment therefore asks:

> Can a Random Forest reproduce the existing rule-based prioritization signal
> from the same observed search and engagement features?

It does not ask:

> Can the Random Forest predict whether a future content refresh will
> improve performance?

That second question remains unresolved by the current experiment.

---

## 5. Evaluation

### Split design

The model uses a **time-aware train/test split**.

Observations are ordered by `report_date`.

The earlier approximately 80% of observations are used for training and the
later observations are held out for testing.

This reflects the intended decision workflow more honestly than a random
split:

**past observations → train → later observations → evaluate**

The same held-out period is used when comparing the model with the baseline.

### Metric

The primary metric is **Mean Absolute Error (MAE)** between the Random Forest
prediction and the baseline score.

### Measured result

The Week-5 experiment measured:

| Metric | Result |
|---|---:|
| Model MAE | **0.34** |

The model's low MAE indicates that its predictions are generally close to the
baseline score.

However, this is a measure of **agreement with the baseline**, not proof that
the model improves future content-refresh decisions.

### Error analysis

The largest disagreements between model predictions and the baseline were
reviewed rather than relying only on the overall metric.

The errors were concentrated among observations with relatively large search
impression volumes and substantial baseline scores.

This is consistent with the model's feature-importance result.

The error analysis therefore reinforces the interpretation that the model is
largely learning the volume-driven structure of the baseline.

### Validation conclusion

The measured evidence supports the statement:

> The Random Forest closely reproduced the Week-4 rule-based baseline under a
> time-aware evaluation split.

It does not support the statement:

> The Random Forest improves future content-refresh decisions.

A legitimate future-window outcome would be required to test that stronger
claim.

---

## 6. Interpretation

### Feature importance

The most important finding from the Random Forest is that:

**`gsc_impressions` accounts for approximately 98.8% of model feature
importance.**

The remaining features contribute comparatively little to the model's
importance distribution.

This indicates that the model is overwhelmingly influenced by **search
impression volume**.

In practical terms, the Random Forest is largely reproducing the
volume-driven pattern already present in the rule-based baseline rather than
demonstrating strong independent predictive contribution from all five
signals.

### Negative result

This is an important negative result.

The purpose of introducing the Random Forest was to test whether a machine
learning model could provide useful additional signal beyond the transparent
baseline.

The current experiment does not demonstrate that additional value.

Instead, it shows that the model can reproduce the baseline closely while
depending heavily on one of the baseline's strongest signals.

### Research findings and methodology caution

The validation audit also reviewed two broader research findings used as
context for the content-refresh workflow.

#### Content age and performance

The research material reports an observed performance curve in which some
content-age groups perform differently, including stronger observed
performance around the 61–90 day range and weaker performance among much older
content.

The methodology question is whether differences between refreshed and
unrefreshed pages are sufficiently controlled before interpreting the
relationship.

The appropriate framing is therefore:

**observed association, not causal proof that refreshing content at a
particular age causes better performance.**

#### Engagement and visibility

The research material also reports that stronger engagement appears alongside
stronger visibility.

The methodology question is whether engagement is an explanatory signal or
a correlated outcome of visibility and other factors.

The safe interpretation is therefore:

**measured association, not evidence that increasing engagement will
necessarily cause better search visibility.**

These checks reinforce the same principle applied to the ML experiment:
correlation and model agreement should not be presented as causal evidence.

---

## 7. Recommendation

The final output is a ranked content-action playbook.

The queue uses the Week-4 scoring logic and assigns a human-readable reason
code.

### Ranked actions

| Reason code | Recommended action |
|---|---|
| `GOOD_POSITION` | **Optimize Existing Content** |
| `MID_POSITION` | **Review and Improve** |
| `LOW_VISIBILITY` | **Investigate Content** |
| `INSUFFICIENT_SIGNAL` | **Manual Review** |

### How an editor would use it

A FlyRank content/SEO reviewer can begin with the highest-ranked items and use
the reason code as the starting point for investigation.

Before acting, the reviewer should check:

1. Current page context
2. Search intent and content relevance
3. Recent page changes
4. Search-performance stability
5. Seasonality or temporary effects
6. Content quality and information gaps
7. Business importance and context

The reason code is therefore a **starting point for investigation**, not a
final diagnosis.

### Human-review requirement

Every recommendation should be reviewed by a person before action.

The playbook should not automatically:

- rewrite or publish content,
- delete or redirect pages,
- change SEO strategy,
- declare a page successful or unsuccessful,
- make high-impact client decisions,
- or claim that a high score guarantees a future performance improvement.

### Monitoring

The workflow should periodically monitor:

- `gsc_impressions`
- `gsc_clicks`
- `gsc_avg_position`
- `ga4_sessions`
- `ga4_engaged_sessions`
- reason-code distribution
- ranked-queue composition

Investigation should be triggered when feature distributions change
substantially, important signals become sparse, the queue becomes unusually
dominated by one reason code, the relationship between signals and observed
performance changes materially, or human reviewers repeatedly disagree with
the recommendations.

A rebuild or retraining exercise should remain human-controlled.

### Confidence and limits

The confidence level in the current recommendations is **directional**.

The queue is suitable for prioritizing human attention, but the evidence does
not establish that following the recommendations will improve future traffic,
rankings, engagement, or business outcomes.

A future verified outcome label would be needed to test that proposition.

---

## 8. Reproducibility

The project is organized as a sequence of notebooks under:

`work/notebooks/`

The main workflow is:

- `w01_research_question.ipynb`
- `w02_ml_task_framing.ipynb`
- `w03_data_contract.ipynb`
- `w03_feature_leakage_check.ipynb`
- `w04_baseline_score.ipynb`
- `w04_signal_audit.ipynb`
- `w05_model.ipynb`
- `w06_validation_audit.ipynb`
- `w07_action_playbook.ipynb`
- `capstone.ipynb`

### Key outputs

The baseline artifact is:

`work/outputs/baseline_action_score.csv`

The action-playbook artifact is:

`work/outputs/action_playbook_queue.csv`

The action queue is generated by the Week-7 notebook rather than manually
maintained.

### Model reproducibility

The Random Forest uses:

`random_state = 42`

with:

- 100 trees
- maximum depth 10
- parallel processing enabled

### Data access

The modeling notebooks retrieve the internship warehouse sample through the
FlyRank dataset repository and use the same June 2026 sample throughout the
baseline/model/action-playbook workflow.

### Fresh-clone workflow

From a fresh clone:

```bash
git clone https://github.com/PiyushSingh0615/flyrank-ml--Internship.git
cd flyrank-ml--Internship