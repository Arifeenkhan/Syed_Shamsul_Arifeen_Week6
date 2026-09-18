# Deployment decision memo

**To:** Content Moderation Product Team  
**From:** SYED SHAMSUL ARIFEEN  
**Subject:** Recommendation for toxicity-classifier deployment

## Decision

For the first production release, I recommend **TF-IDF + Logistic Regression** as the primary automatic toxicity classifier, with fine-tuned DistilBERT retained as a future upgrade candidate. This recommendation balances the measured results with operational constraints, not accuracy alone.

## Evidence from the held-out test set

| Measure | TF-IDF + Logistic Regression | Fine-tuned DistilBERT |
|---|---:|---:|
| Accuracy | 0.9572 | 0.9650 |
| Precision | 0.7721 | 0.8535 |
| Recall | 0.7850 | 0.7662 |
| F1 | 0.7785 | 0.8075 |
| ROC-AUC | 0.9747 | 0.9806 |
| Median per-comment latency (ms) | 2.65 | 16.40 |
| Model size (MB) | 8.73 | 256.11 |

For content moderation, F1 and recall deserve more attention than accuracy because toxic comments are less common than non-toxic comments. A model that labels everything non-toxic can appear accurate while missing harmful content. Precision also matters: excessive false positives can wrongly block legitimate users.

## Why this model should be deployed

DistilBERT achieved the strongest F1 (0.8075 versus 0.7785) and ROC-AUC (0.9806 versus 0.9747). Its precision was also much higher (0.8535 versus 0.7721), so it would generate fewer false positive toxicity flags. However, the baseline had better recall (0.7850 versus 0.7662), which matters because missed toxic comments can harm users.

The quality gain does not justify the operational cost for an initial general-purpose deployment. The TF-IDF model predicts a comment in a median 2.65 ms, compared with 16.40 ms for DistilBERT: roughly 6.2 times faster. Its stored model is 8.73 MB rather than 256.11 MB, roughly 29 times smaller, and it trained in 30.19 seconds rather than 236.67 seconds. It can run efficiently on ordinary CPU infrastructure, is inexpensive to retrain as moderation language changes, and its n-gram weights are easier to inspect. DistilBERT may need more memory, more expensive serving hardware, and careful cold-start planning. For a high-volume moderation API, these differences directly affect capacity and cost.

## Risk controls and next steps

I would not use either model as the only moderation mechanism. Route predictions near the operating threshold to human review, log confidence scores and appeals, and monitor false positives and false negatives by language variety and protected-group references. Choose the production threshold using a precision-recall trade-off that reflects moderation policy rather than blindly using 0.50. Re-evaluate after new labeled data arrives, because slang, abuse patterns, and class balance change over time.

## Conclusion

The deployment choice is **TF-IDF + Logistic Regression** for the first release. DistilBERT is the better quality model by F1 and ROC-AUC, but the baseline is faster, dramatically smaller, less costly to operate, easier to retrain, and has slightly higher recall. The numerical comparison is necessary evidence, but it is not by itself sufficient for a production decision.
