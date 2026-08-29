# Capstone Report

* **Author:** Haneen Elabd
* **Lane:** Refresh / Content Opportunity Scoring
* **Repo:** `flyrank-assiment`
* **Date:** August 2026

## 0. Abstract

This capstone asks whether observable content, search, engagement, and freshness signals can rank pages that are most likely to experience a decline in search visibility. The analysis uses an anonymized FlyRank dataset containing 30,000 content-level observations and 44 columns. A transparent rule-based baseline is compared with a Random Forest model using 21 observable features and Precision@20 and Precision@50 as the primary ranking metrics. On the current held-out split, the baseline achieved 0.85 Precision@20 and 0.76 Precision@50, while the Random Forest achieved 1.00 for both metrics. The resulting ranking is intended to help content teams prioritize pages for human review and possible refresh or improvement, rather than to establish causality or explain Google's ranking algorithm.

## 1. Problem framing

The decision supported is which content pages should receive attention first from a content team.

The unit of analysis is an individual content page. The output is a ranked score indicating the model's estimated likelihood that the page belongs to the declining proxy class.

The practical action is for an editor or content specialist to review high-ranked pages first and decide whether refresh, improvement, monitoring, or no action is appropriate.

A wrong positive call can waste editorial resources on a page that does not need intervention. A wrong negative call can cause a potentially declining page to be missed. Ranking is therefore useful because it concentrates limited review capacity on a small set of pages.

## 2. Data safety

The analysis uses an anonymized FlyRank content-level dataset with search, content, engagement, freshness, and performance signals.

The original dataset contains 30,000 rows and 44 columns. After filtering and preparing the modeling frame, 19,897 rows were used with 21 numerical features.

The following fields were deliberately excluded from the model because they are label-derived, categorical identifiers, or otherwise unsuitable for the modeling objective:

* `content_id` — identifier only.
* `client_id` — pseudonymous grouping identifier, never used as a predictive feature.
* `trend_direction` — derived from performance movement and therefore potentially label-related.
* `trend_pct` — derived from performance movement and therefore potentially label-related.
* Other categorical/tier fields were not included in the numerical modeling feature set.

The target, `declining_proxy`, represents whether impressions in the latest 30-day window were lower than impressions in the previous 30-day window.

The observed declining-proxy rate is approximately 54.2%, which is reported as the task base rate when interpreting Precision@K.

No client names, domains, private queries, URLs, credentials, or other identifying information are included in the public analysis.

## 3. Baseline

The baseline is a transparent rule-based score using two observable signals:

* average search position greater than 20
* scroll rate below 5

Each satisfied condition contributes to the baseline score, and pages are ranked by this score.

This is a fair comparison because it uses simple, interpretable signals that a content team could apply without machine learning. Both the baseline and the Random Forest are evaluated on the same held-out data using the same Precision@20 and Precision@50 metrics.

| Method        | Precision@20 | Precision@50 |
| ------------- | -----------: | -----------: |
| Baseline      |         0.85 |         0.76 |
| Random Forest |         1.00 |         1.00 |

The majority-class base rate for the declining proxy is approximately 54.2%.

## 4. Model / analysis

The problem is treated as a ranking task within the Refresh / Content Opportunity Scoring lane.

The target is `declining_proxy`, defined as whether a page's impressions in the latest 30-day window are lower than its impressions in the previous 30-day window.

A Random Forest classifier is trained and pages are ranked using the predicted probability of the declining class.

The final feature set contains 21 numerical features:

`search_volume`, `competition`, `cpc`, `word_count`, `char_count`, `impressions_90d`, `clicks_90d`, `pageviews_90d`, `sessions_90d`, `users_90d`, `engaged_sessions_90d`, `scroll_events_90d`, `days_with_impressions`, `days_with_sessions`, `content_age_days`, `days_since_last_update`, `ctr`, `avg_position`, `engagement_rate`, `scroll_rate`, `ai_traffic_pct`.

Label-derived fields such as `trend_direction` and `trend_pct` were excluded intentionally. Pseudonymous IDs were also excluded from the feature set.

The model is used for prioritization rather than causal inference.

## 5. Evaluation

The current experiment uses an 80/20 stratified train/test split with `random_state=42`.

The baseline and Random Forest are evaluated on the same held-out test set.

The primary metrics are Precision@20 and Precision@50 because the intended use case is prioritizing a small review queue.

| Method        | Precision@20 | Precision@50 |
| ------------- | -----------: | -----------: |
| Baseline      |         0.85 |         0.76 |
| Random Forest |         1.00 |         1.00 |
| Base rate     |        0.542 |        0.542 |

On the current split, the Random Forest places declining-proxy pages at the top of the ranking more effectively than the simple baseline.

The current results should be treated as preliminary evidence from this held-out experiment. They do not establish performance on unseen future time periods or prove a causal relationship between the observed signals and visibility decline.

## 6. Interpretation

The model combines multiple observable signals rather than relying on a single ranking rule. The strongest practical interpretation is that search position, engagement, traffic history, freshness, and content characteristics can be useful for prioritizing pages for review.

The analysis is directional: the model identifies pages associated with the declining proxy, but it does not explain why the decline occurred.

The 1.00 Precision@20 and 1.00 Precision@50 results on the current held-out split are substantially higher than the baseline. Because the declining-proxy base rate is approximately 54.2%, the result represents strong concentration within this particular evaluation split, but it should not be treated as proof that the same performance will generalize to future data.

No conclusion is made about Google's ranking algorithm, ranking penalties, or causal effects of content refreshes.

## 7. Recommendation

The model output can be used as a prioritized editorial review queue.

Recommended workflow:

1. Start with the highest-scoring pages.
2. Review their search position, CTR, engagement, traffic history, and freshness context.
3. Inspect the associated reason codes.
4. Decide whether the page should be refreshed, improved, monitored, or left unchanged.
5. Use human judgment before taking any content action.

A high model score means that the page is highly ranked by the model for association with the declining proxy. It does not mean that the page definitely needs a refresh.

Confidence is therefore highest in the ranking as a prioritization mechanism on the evaluated dataset, and lower when making claims about future performance or causality.

## 8. Reproducibility

The analysis is implemented in Python using pandas, NumPy, and scikit-learn.

The repository contains the weekly notebooks used to develop the capstone, including:

* research question
* ML task framing
* data contract
* feature leakage check
* baseline scoring
* signal audit
* model development
* validation audit
* action playbook
* capstone analysis

The Random Forest experiment uses:

* `n_estimators=200`
* `max_depth=8`
* `random_state=42`
* `class_weight="balanced"`
* `n_jobs=-1`

The train/test split uses:

* test size = 20%
* `random_state=42`
* stratification by the target

The capstone notebook contains the modeling and ranking workflow and exports the capstone results and ranked recommendation artifacts.

## 9. Acknowledgments & data credit

Built on the FlyRank ML Internship dataset.

Data and internship context are credited to FlyRank.

[FlyRank](https://flyrank.ai)
