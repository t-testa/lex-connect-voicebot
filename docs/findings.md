# Findings log

Bot: BankingFrontDoor (Lex V2, en_US, ca-central-1)
Lex confidence threshold: 0.40 (flow threshold to be decided by eval)

## F1. Overlapping intents: "someone used my card without permission"
Date: 2026-10-15

| Intent | Score |
|---|---|
| ReportLostCard (chosen) | 0.93 |
| DisputeTransaction | 0.57 |
| CheckBalance | 0.26 |

- Top score is high, but the runner-up is above 0.5. A top-score threshold alone hides the ambiguity.
- Margin between top two (0.36) is a second signal. Eval will flag margins under 0.2.
- Scores are independent per intent, not probabilities. They do not sum to 1.
- Cost of confusion depends on routing. Both intents go to an agent here, so the cost is low. In production they would go to different queues.
- ReportLostCard is defensible: blocking the card is the safer first action.

## F2. Ambiguous access phrase: "I can't get into my account"
Date: 2026-10-15

| Intent | Score |
|---|---|
| ResetOnlineBanking (chosen) | 0.82 |
| DisputeTransaction | 0.66 |
| ReportLostCard | 0.63 |
| CheckBalance | 0.59 |

- Margin 0.16, flagged under the 0.2 rule.
- Phrase is genuinely ambiguous (app, card, or frozen account).
- Possible improvement: clarifying question via Lex intent disambiguation (uses Display name). Not built yet.

## F3. Fallback is not just "score below threshold": "help"
Date: 2026-10-15

| Intent | Score |
|---|---|
| FallbackIntent (chosen) | no score |
| SpeakToAgent | 0.73 |
| ReportLostCard | 0.65 |

- SpeakToAgent scored well above the 0.40 threshold, yet Lex still chose Fallback.
- Lex has its own out-of-scope logic. Fallback carries no score.
- Eval impact: record "Lex chose Fallback" and "low confidence on chosen intent" as two separate outcomes.
- Flow design: first Fallback reprompts with a short menu, second Fallback transfers to an agent.

## F4. Scores are compressed: "what's the weather like"
Date: 2026-10-15

| Intent | Score |
|---|---|
| FallbackIntent (chosen, correct) | no score |
| BranchHours | 0.76 |
| CheckBalance | 0.64 |

- Out-of-scope input still scores 0.5 to 0.76 on several intents.
- A score threshold alone cannot separate in-scope from out-of-scope. Lex's fallback logic does most of that work.
- Eval impact: thresholds 0.5, 0.6 and 0.7 mostly affect borderline in-scope phrases, so the test set needs ambiguous and out-of-scope phrasing.

## F5. Handoff verified in Contact search
Date: 2026-11-13

- Fallback contacts arrive with handoffreason=fallback, intent=FallbackIntent and an empty confidence, consistent with F3.

## F6. Exact training utterances score 1.0: "talk to a person"
Date: 2026-11-13

- "talk to a person" (an exact training utterance) scored 1.0 in production.
- Exact matches are uninformative for threshold tuning, so the eval test set must exclude training utterances.
