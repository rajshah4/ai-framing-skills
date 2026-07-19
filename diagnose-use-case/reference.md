# ML and GenAI Diagnostic Reference

Load this file when the system under diagnosis is classical ML (prediction over structured data: churn, fraud, forecasting, ranking, regression) or GenAI/RAG (generation, retrieval, extraction, summarization) rather than agentic. It supplies the modality-specific tests behind the three questions, plus the classical-ML and GenAI forms of the five reframing moves. The method itself, the seven signals, and the persist/pivot/stop call live in SKILL.md; nothing here repeats them.

## ML-specific checks

### Is there signal?

**Distribution checks first.** Most "model problems" are input problems. Before blaming the model, check the shape of the target, the features, and the requests.

Target distribution:

| Pattern | Evidence | What it reveals | Fix |
|---|---|---|---|
| Zero-inflation | 90% of target values are zero | Standard regression (MSE) predicts tiny numbers for everyone | Two-stage model (classify then regress), Tweedie loss |
| Heavy tails | Top 1% of cases = most of the value | Optimizing the mean under-predicts the whales | Quantile regression (90th percentile), weighted loss |
| Class imbalance | 99%+ majority class | Model predicts the majority class for everyone | Class weights, change loss, undersample, or reframe as ranking |

Feature distribution:

| Pattern | Evidence | What it reveals | Fix |
|---|---|---|---|
| Sparsity | High-cardinality IDs, most appear once | Model memorizes single examples for rare entities | Dense features, behavior aggregates, embeddings with side info |
| Missing data | Missingness correlates with the outcome | The absence is signal, not noise | Add `field_missing` as a feature, segment-aware imputation |
| Multimodality | Works for segment X, fails for Y | Distinct subpopulations, one model cannot fit both | Segment models, decompose, or add segment as a feature |

Missing data is often the reframe itself: a missing lab test is a clinical judgment, missing activity in the last 30 days may BE the churn event, a missing loan-application field is often more predictive than any filled field.

Worked example, insurance claims (zero-inflation): most claim amounts are $0. An MSE model predicts $320 and $280 for zero-claim customers and $1,200 for an actual $8,750 claim. It captures the mean and fails the physics. Fix: classify "will an event happen," then regress the amount.

Worked example, customer spend (heavy tails): mean $2,340, median $130, top 1% of customers = 68% of revenue. The MSE model is accurate on the 95% of regular customers (32% of revenue) and terrible on whales. The real reframe: not "predict spend" but "identify and serve whales."

**Signal sanity checks.** Run these before error analysis; you cannot diagnose a broken experiment.

- Overfit test: train on 10 to 50 examples for many epochs. Loss should go near zero. If it doesn't, the pipeline is broken (labels disconnected from features), labels are noise, or the architecture can't represent the function. A model that can't memorize 10 examples can't learn from 10,000.
- Random labels test: shuffle the target column and train. If the model converges anyway, it is memorizing, not learning, and it is overfitting your real data too.
- Baseline sanity check: does predicting the mean/mode, one obvious feature, or a single rule beat the model? If yes, the complexity isn't earning its keep. (SimpleStream, 2025: streaming the last four frames to a VLM matched state of the art on video benchmarks, beating complex retrieval and memory mechanisms.)
- Random probe: add a column of pure noise, run feature importance. Any feature ranking below the noise column is garbage, and the hypothesis behind it is wrong.

**Learning curves** (performance vs training-set size):

| Pattern | Diagnosis | Fix |
|---|---|---|
| Large gap, train high, test low | Overfitting | More data, regularization, simpler model |
| Both curves low, converging | Underfitting | More features, more capacity, less regularization |
| Both curves high, converging | Good fit | Done, or add data for small gains |
| Test curve plateaus early | Data saturation | Better features or different approach; more rows won't help |

**Loss curves** (train and validation loss over epochs; for deep learning and LLM fine-tuning):

| Pattern | Diagnosis | Action |
|---|---|---|
| Val loss rises while train drops | Overfitting | Early stopping, regularization |
| Both losses plateau high | Underfitting | More capacity, less regularization, check data |
| Loss spikes or oscillates | Learning rate too high | Reduce LR, add warmup |
| Loss doesn't decrease | LR too low or broken pipeline | Raise LR, check gradients, verify data loading |
| Train loss hits 0 instantly | Memorization or data leak | Check duplicates, verify pipeline |

**Metric plateau** ("we added data and accuracy didn't move"): information ceiling. The signal isn't in your features. The fix is structural: new features, new data sources, or a different target. None of those are "tune harder."

### Is there cheating?

- Feature ablation test: remove the top-ranked feature, retrain, compare. If performance improves, you have target leakage, a spurious proxy, or overfitting to noise. A feature that seems too good usually is.
- Time split test: train on months 1-6, test on 7-8, compare to the random-split result. If the time split tanks performance: look-ahead bias, concept drift, or the random split was leaking across time.
- Too-good-to-be-true: 99% R-squared on a problem nobody has cracked at that level is almost always leakage. Check top features for time-horizon violations, drop or repair them, rerun. If the metric falls to a believable level, the leakage was real; if it stays sky-high, keep looking.
- Entity contamination: verify train/test splits don't share entities (same user, same session). Shared entities measure recall, not generalization.
- Dead Salmon test (before trusting SHAP, saliency, attention maps, or probes): run the same interpretability method on a null baseline: an untrained model with random weights, a model trained on shuffled labels, or a probe trained on random targets. If the explanations look similar to the real model's, the method is finding patterns in noise. If you can't state the null hypothesis or whether intervening on the explanation changes behavior, the explanation is storytelling.

### Is the metric real?

- Direct experience test: don't read the dashboard, experience what the user experiences. Amazon's metrics said support wait times were under 60 seconds; Bezos dialed the 1-800 number in a meeting and waited ten minutes. If lived experience disagrees with the metric, the metric definition is flawed, it measures the wrong point in the journey, or the average is hiding outliers. Applied: query your own RAG system with real questions, review flagged fraud cases manually, sample "approved" moderation content and ask if you would approve it.
- Subgroup decomposition: a single aggregate hides the failure. Worked example, churn model: 94% accuracy but 23% precision (predicting "no churn" for everyone scores about the same), Enterprise precision 20% because training data was 9,000 SMB vs 200 Enterprise rows, a random noise column ranked #10 in feature importance, and `last_login_days_ago` ranked #1 (it isn't predicting churn, it IS churn). The business fix was reframing classification to ranking: the CS team needed a prioritized list.
- Calibration: plot predicted confidence vs actual accuracy.

| Pattern | Interpretation | Fix |
|---|---|---|
| Points above diagonal | Underconfident, conservative | Often acceptable; the cost is throughput, not correctness |
| Points below diagonal | Overconfident, dangerous for routing and auto-approval | Temperature scaling, Platt scaling; if those fail, the training data or loss is mis-specified |
| S-curve | Extremes miscalibrated | Isotonic regression |

  Metrics: Expected Calibration Error (average confidence-accuracy gap per bin), Brier score. OOD confidence check: feed deliberately out-of-distribution inputs. If confidence stays high on inputs the model has never seen, the architecture has no "I don't know" mechanism; that is structural, not tunable.
- Comparison trap (Lord's paradox): delta comparison ("which improved more") and baseline-controlled comparison ("which ends higher from the same start") can give opposite answers on the same data. Model A went 70% to 85% (+15), Model B went 80% to 88% (+8): deploy B, invest in A's recipe. Finish "we will consider it better if..." before running the comparison. Segment version: before calling a 85% vs 70% segment gap bias, check each segment's baseline; 70% over a 60% baseline can be a bigger win than 85% over 80%.

## GenAI/RAG-specific checks

### Is there signal?

**RAG decomposition.** Sample 50 errors (not successes) and categorize each by where the pipeline failed. The fix depends entirely on the location.

| Category | Definition | Fix |
|---|---|---|
| Retrieval miss | Relevant document exists but wasn't retrieved | Hybrid search (add keywords), better chunking, query expansion |
| Retrieval hit + hallucination | Context correct, model added or changed information | Citation requirements, lower temperature, faithfulness checks |
| Retrieval hit + missed detail | Context correct, model missed key information | Better prompting, chunk highlighting |
| Confident without source | Answered from parametric memory, ignored retrieval | Force retrieval-only mode, train for abstention |

If 80% are retrieval misses, no amount of prompt engineering helps; fix retrieval. If 80% are hallucinations, retrieval tuning is wasted; add grounding. (HR policy RAG lab: of 20 user-flagged failures, about 33% were retrieval problems and 42% hallucination problems.)

Retrieval-quality anchors: Pinecone found accuracy dropped 20% as stuffed context grew, and targeted retrieval hit 95% accuracy at 25% of the tokens. Anthropic's contextual retrieval (prepend LLM-generated chunk context before embedding, so "revenue grew 3%" becomes "In ACME's Q2 2024 report, revenue grew 3%") cut retrieval failures 49%, 67% with reranking. The embedding model only sees the chunk text, never your metadata columns.

**GenAI error patterns.** Each points at a different layer, so each has a different fix.

| Pattern | What it reveals | Fix |
|---|---|---|
| Confident but factually wrong | Knowledge failure: weights hold outdated or noisy info | Retrieval (RAG), not fine-tuning; bigger models hallucinate more confidently |
| Logic or arithmetic failure | Reasoning failure, no scratchpad for multi-step work | Chain-of-thought, or tool use (calculator, interpreter) |
| Retrieval miss ("I don't know" when the doc exists) | Query embedding didn't match the document | Hybrid search, rechunk, query expansion |
| Retrieval hit, wrong answer | Model had what it needed and didn't use it | Grounding prompts: "answer only from provided context," citations, extractive mode |
| Prompt keeps growing (50+ "do not" rules) | Behavior problem patched with context; model fights its training | Fine-tune the behavior into the weights |

**"Why is this hard?" categorization.** For each sampled error: ambiguity (a human couldn't label it either: fix the task definition), missing info (answer isn't in the input: add retrieval or data, not prompts), logic error (info present, reasoning weak: bigger model, chain-of-thought, tools), edge case (training distribution doesn't match reality: expand data or accept the limit). Tally; the dominant category sets the fix. Teams waste months on prompts when the problem is retrieval, or fine-tune when the problem is ambiguous labels.

**Query distribution checks.**

- Zipfian traffic: if the top 100 queries are over 50% of traffic, tier the architecture and precompute the head. Instacart precomputed LLM-enriched answers for top queries, covered about 70% of traffic from cache, 10x cost reduction. Precomputation fails on highly personalized answers, fast-changing information, or a flat query distribution.
- Ambiguity: do users rephrase after the first response? Do short queries have higher "not helpful" rates? Then the system retrieves before it understands. Fix: ambiguity classifier, clarification flows, multi-intent retrieval.

### Is there cheating?

LLM benchmark and eval contamination checks:

| Check | What it catches |
|---|---|
| Exact match search | Test questions verbatim in the training corpus |
| N-gram overlap (8-grams or 13-grams) | Paraphrased versions of test data |
| Canary strings | Insert a unique synthetic string in the test set; if a model reproduces it unprompted, the set leaked |
| Temporal cutoff | Test items published after the model's training cutoff can't be contaminated; before it, discount the score |
| MCQ format shortcuts | Test the model on answer choices alone, no question; well above chance means it reads the answer key, not the question |

If MCQ shortcuts fire, switch to generative answer matching: free-form response scored against a reference.

### Is the metric real?

- Eval gap: playground or curated-test performance does not match production because the test set is not your users. Evaluate on real traffic. Meta Galactica passed curated evals and was pulled 72 hours after launch when adversarial users broke it immediately.
- LLM calibration is its own problem: token probabilities don't translate to answer correctness, and "I'm not sure" in text maps to no probability. Ask the model to self-assess and check whether that correlates with actual accuracy before trusting it for routing.

## Four special branches

### Coding and workflow agents

When the failing system writes code or executes multi-step workflows, the failures concentrate at predictable seams. Check these before blaming model capability.

- Implicit invariants: the agent violated a rule nobody wrote down (don't touch the schema, migrations run in order, this file is generated). The fix is a deterministic artifact the agent operates from (manifest, dependency graph, playbook, test oracle), not a longer prompt.
- Sequencing and shared state: steps correct in isolation, wrong in combination, or parallel agents clobbering each other. The fix is isolation (worktrees, disjoint files, a merge arbiter) and explicit ordering, not smarter agents.
- Speculation across missing context: the agent guessed instead of stopping. Check whether a stop-and-ask rule exists; if guessing was the only option, the framing under-specified when to halt.
- Prompt sediment: a static system prompt that grew one rule per incident until the model stopped following it. The fix moves controls to the right layer: a tool permission, a validator that rejects bad output, or a decision-time micro-instruction injected at the point of action (Replit's pattern: a lightweight classifier injects only the relevant guidance when it's needed, instead of bloating the standing prompt).
- Self-confirmation: the agent wrote the tests that pass its own code. Validation contracts set before generation, or an independent verifier, put the bar outside the system being tested.

### LLM-as-judge and evaluation systems

When the system under diagnosis IS an evaluator (a judge model scoring outputs, an eval pipeline gating releases), the question shifts: the judge can look right while being wrong, and everything downstream inherits the error.

- Expert agreement: what is the judge's agreement rate with human experts, measured on a held-out sample? No target agreed in advance means the judge's scores are unanchored numbers.
- Downstream correlation: does the judge's score correlate with the outcome that matters (user retention, task completion, business metric)? A judge that predicts nothing downstream is measuring style.
- Bias probes: test for position bias (swap candidate order, does the verdict flip?), verbosity bias (does longer reliably win?), self-preference (does it favor outputs from its own model family?), and style-over-substance.
- Disagreement review: sample the cases where judge and human disagree; the pattern in the disagreements is the diagnosis.
- Drift: judge calibration decays as the systems it scores improve or shift. Re-anchor against experts on a schedule, not once.
- The decision rule: a judge can rank and filter; it cannot be the final decision-maker where a wrong verdict is expensive. If the judge gates something irreversible, a human gate belongs behind it.

### Model training and infrastructure failures

When the failing system is a training run or serving stack rather than a product, the seven signals still apply but the fixes live lower. First separate the failure type: quality (converges to a bad model), stability (loss spikes, divergence, NaNs), throughput or latency, memory, or cost. Each is a different diagnosis.

- Stability failures are often one hyperparameter, not architecture: BLOOM's 176B training was stabilized by an initialization standard-deviation fix after repeated divergence. Check init, learning-rate schedule, and warmup before redesigning.
- Workarounds are valid pivots, not defeats: IDEFICS shipped around a slow CPU memory leak with watchdog, restart, and checkpointing rather than halting everything to find root cause. In infrastructure, "contain and continue" often beats "stop and debug" on value delivered; log the debt and move.
- The operational red flag maths still rule: if the fix is unaffordable at production scale, the diagnosis is a trade-off failure (Step 4), not an engineering to-do list.

### Fairness and high-stakes systems

When the system decides about people (credit, hiring, health, moderation, pricing), subgroup decomposition (above) is mandatory, and three checks go beyond it.

- Segment false positives and false negatives separately by group, context, language, geography, and device. The harm usually lives in one cell of that table, and the aggregate hides it.
- Proxy attributes discriminate without the protected column: pricing on engine size can encode gender, a zip code can encode race. Ablate suspected proxies and watch the subgroup gap, not the aggregate.
- Verify the operational safeguards exist: an appeal or override path a harmed person can actually use, a named list of prohibited uses the system could technically perform, red-team cases monitored in production, and a safety stop signal pre-committed alongside the quality one. A fairness problem with no appeal path is not a model bug, it is a framing failure at Step 1.

## The five moves in ML and GenAI form

**Retarget** (change what you predict).
ML: change the label (30-day churn to next-week churn, the window the save-offer team can act on; clicks to watch time when clicks reward bait), change the loss (class weights, focal loss, quantile regression), or change the granularity (user-level to session-level, document-level to chunk-level).
GenAI: change what the prompt asks for. "Summarize" is wrong if downstream needs structured fields; "answer" is wrong if the user needs sources. Generation to extraction, answering to citing.

**Decompose** (break the problem).
ML: per-segment models (enterprise vs SMB), hierarchical models, cascades (cheap model filters, expensive model handles the rest), multi-task with shared backbone. Canonical: recommenders split into retrieval (millions to ~1,000 candidates) and ranking, with different objectives and latency budgets.
GenAI: prompt chaining (one prompt extracts, the next reasons, the third generates), each step narrow and testable. RAG itself is a decomposition: retrieve first, generate second.

**Ensemble** (run it more than once).
ML: bagging (random forest, often the strongest tabular baseline), boosting (XGBoost, LightGBM as production default), stacking. Accuracy bought at inference cost.
GenAI: self-consistency (sample N reasoning paths, majority vote), judge models scoring N candidates, debate. The hybrid sandwich: rule layer (regex PII filter), LLM layer, rule layer (hallucination keyword check); rules guard the probabilistic middle. Ensembling only works when failure modes are uncorrelated; five copies of the same model voting buys nothing.

**Constrain** (reduce what the system can do).
ML: classification head instead of regression (output space R down to N classes), softmax to a probability distribution, monotonic constraints on tree models.
GenAI: structured output (JSON schema, function calling), constrained decoding (grammar or logit level, the model cannot emit violating tokens), tool whitelists, regex post-processing. Specific move: don't ask the LLM to "calculate the total," ask it to write a Python script that does; LLMs are bad calculators and good coders. Every constraint trades flexibility for reliability. Most production GenAI is heavily constrained; free-form chat is the demo, not the product. (Klarna: policies moved from the LLM's weights to a knowledge graph for deterministic lookup, LLM kept for language only.)

**Simplify** (replace the system with a smaller one).
ML: deep net to gradient boosted tree to logistic regression to a rule; each step down loses accuracy and gains debuggability, retraining speed, serving cost, and explainability. Feature version: 200 features to 20 when the extra 180 only created drift.
GenAI: fine-tuned model to prompting, multi-step chain to single prompt, LLM call to regex on structured inputs, open generation to template filling, generative chatbot to retrieval-only search. If simple captures 80% of the value at 5% of the cost, simple wins. Teams resist this move because it looks like backsliding; the simple version is what runs in production today, the ambitious one stays on the roadmap until it earns its place.
