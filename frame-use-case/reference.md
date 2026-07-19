# Modality reference: ML and GenAI archetypes

Load this file when the use case's primary modality is classical ML (prediction over structured data) or GenAI (generation, retrieval, or extraction over content), not agentic. SKILL.md's automation spectrum covers the agent case; this file supplies the equivalent menus for the Alternatives step and the atomic-unit check. Each archetype is a lens, a question asked of the data. The wrong lens, not the wrong model, is the usual failure. The baseline column doubles as the de-escalation candidate SKILL.md requires.

## Before ML: the de-escalation gate

Four signs the use case does not need ML at all:

1. Expert-can-explain test: a domain expert can write down the rules. If the logic can be enumerated, no model needed.
2. Edge-case limit: fewer than 20 meaningful branches. If/else trees stay maintainable below roughly 50 branches.
3. Data poverty: fewer than 1,000 examples. ML needs patterns; rules need logic.
4. Audit trail: regulators must understand every decision. Rules are transparent by default.

If accuracy plus auditability is the bind, interpretable models sit between rules and black boxes: scorecards, rule lists, RuleFit, GA2M. GA2M reaches near-black-box accuracy while staying fully auditable. A deployed simple model beats a complex model stuck in review. About 90% of problems tagged for ML are solved with standard regression approaches.

Graduate from rules to ML only when: the rule count is unmanageable (50+ and growing), edge cases keep appearing that rules cannot anticipate, patterns change faster than rules can be updated, or interpretable models verifiably cannot match the needed accuracy. If a threshold and a lookup table reach 75% and ML reaches 82%, ask whether the 7-point lift is worth the pipeline, training, monitoring, and maintenance.

## ML archetypes

| Archetype | Atomic unit: decides about | Fits when | Metric and its trap | Simplest baseline |
|---|---|---|---|---|
| Classification (binary or multi-class) | For each item, which known category | Clear labels exist for every class, 2 to 20 categories | Accuracy or AUC. Trap: extreme imbalance (1M normal, 3 fraud) gives 99.99% accuracy and zero value. 100+ categories: reframe as retrieval | Keyword rules, if/else tree, scorecard |
| Regression | For each independent row, what number | Continuous target, features known at prediction time, rows independent | Per-row error (MSE). Trap: random validation split when time matters leaks the future. Benchmark against the mean: linear baselines beat five foundation models on gene perturbation | Simple aggregation, linear model |
| Forecasting | For this series, what comes next | Planning ahead (inventory, staffing, budget); order matters; yesterday physically constrains today | Error on a temporal split, never a random split. Trap: treating time as just another regression feature; predicting Y from an X you will not know at prediction time | "Tomorrow = last week's Tuesday" seasonal naive |
| Anomaly detection | For each event, is this a deviation from normal | You can define normal but not enumerate bad; failure modes are novel and evolve; needle-in-haystack imbalance | Trap: classification metrics collapse at 0.01% positives, and a classifier trained on yesterday's fraud is blind to tomorrow's | Threshold: flag anything past 3 standard deviations |
| Clustering | Across all items, what groups exist (output is discovered structure, not a per-item call) | No labels, discovery goal, cold start, or you suspect the current categories are wrong | No ground truth to score against. Trap: decreeing segments by arbitrary rules first; your "VIPs" may be two groups needing different strategies | The rule-defined segments you already use |
| Ranking | For this candidate list, which order | User acts on the top of a list; position matters more than absolute score | Listwise quality, not pointwise score accuracy. Trap: Netflix predicted star ratings perfectly and engagement did not move; the list is the product | Sort by one obvious feature or popularity aggregate |
| Recommendation | For this user, which item next (personalized ranking) | Personalization required; implicit signals available | Trap: 99% of users never rate anything, so train on clicked vs ignored, not stars; users calibrate scales differently | Non-personalized ranking, same list for everyone |
| Optimization | Given predictions and constraints, which action or allocation | Finite resources, competing objectives, combinatorial choices (routes, schedules, budgets) | Objective value under constraints. Trap: shipping a demand forecast and wondering why ops cannot use it; prediction is information, optimization is the decision | Greedy heuristic, precomputed plan |

Two-sided assignment (matching A to B in a marketplace) routes to optimization and matching algorithms, not prediction.

Training-paradigm check, because it decides what data is needed: classification and regression are supervised (someone must produce labels; "we have no labeled data" rules out this family). Clustering and anomaly detection are unsupervised (raw data is enough). If the team asks "why did this happen" or "what is the impact of X", that is causal inference, not any prediction archetype.

### ML probes: questions that separate the lenses

Ask these when two ML archetypes both seem plausible. Each is a sign test from the field guide.

**Forecasting, not regression, when:**
- The next-week rule: stakeholders ask "what happens next", not "what drives this", and the prediction is worthless unless known 7+ days ahead.
- The snowball effect: yesterday's value physically constrains today's, like a water tank level.
- The missing-input problem: you want to predict Y from X but will not know X at prediction time. You are actually forecasting X.

**Anomaly detection, not classification, when:**
- Unknown unknowns: you can describe good but bad can look like anything; you cannot list every error, fraud, or defect type.
- The new-trick problem: adversaries change strategy weekly, so a classifier is stuck in the past.
- The open-world limit: data contains things never seen before. A classifier trained on cats and dogs calls a giraffe a weird dog; an anomaly detector says "I do not know what this is."

**Clustering, not classification, when:**
- No one has ever tagged the data and labeling is unaffordable.
- The discovery goal: you suspect the current categories are wrong or incomplete. Classification finds what you look for; clustering finds what you did not know to look for.
- Cold start: new product or market, no history to train on.

**Ranking, not pointwise prediction, when:**
- Choice paralysis: the model scores 50 items all at 5 stars; ranking forces a number one.
- Scale calibration: one user's 3 stars is another's 5, so order is learnable where absolute scores are noise.
- The context factor: the best item depends on what sits next to it (one milk, one bread, one egg, not five milks).

**Optimization, not prediction, when:**
- Conflicting goals: highest quality and lowest cost and fastest speed at once. Prediction gives values; optimization finds the trade-off point.
- Finite resources: a budget, time, weight, or space limit forces choosing a subset.
- The action gap: the dashboard is perfect and the team still argues about what to do. Information is not a plan.

Industry anchors, useful when the user needs a named precedent: Amazon forecasts per-fulfillment-center demand; Stripe flags novel fraud with anomaly detection because classifiers miss zero-day patterns; Spotify discovers micro-genres by clustering listening history instead of trusting genre tags; TikTok optimizes the order of the next five videos, not the rating of any one; UPS ORION routes trucks with optimization because a traffic prediction does not get a truck from A to B.

## Before GenAI: the non-LLM gate

Four signs the use case does not need an LLM:

1. Regex test: you can describe exactly what you are looking for. If it fits a pattern, an LLM is overkill.
2. Million-row problem: massive daily volume makes LLM costs prohibitive.
3. Millisecond constraint: real-time systems cannot wait for API calls.
4. Reproducibility: same input must give same output. LLMs are non-deterministic even at temperature 0.

Cheaper rungs before a full LLM: regex, spaCy or traditional NLP, BERT encoders, small LMs, templates. Production systems win by escalating from the cheapest adequate tool. Klarna's L1 support could have been automated without an LLM; it reversed to humans in 2025 over quality.

Graduate from traditional NLP to an LLM only when: text is unpredictable (slang, typos, novel queries), the task needs reasoning across multiple pieces of information or genuine synthesis, labeled data is too scarce to fine-tune a smaller model, or the output structure is genuinely open-ended. If an LLM looks like a 100% boost, traditional NLP plus templates likely gets 80% of the way there, faster, cheaper, and more reliably.

## GenAI archetypes

| Archetype | Atomic unit: decides about | Fits when | Metric and its trap | Simplest baseline |
|---|---|---|---|---|
| RAG / question answering | For each question, which passages ground the answer | Answers live in your documents; freshness matters; users ask "says who" and need citations | Grounded correctness, not fluency. Trap: fine-tuning the model to "know" your policies; knowledge freezes, policies change, hallucinated policy is a liability. Choosing the wrong search type is a top RAG failure mode | FAQ page plus keyword (BM25) search |
| Generation | For each prompt, what new text | Goal is novelty, not fidelity (50 ad variations); wording may differ per run | Subjective quality, hard to validate. Trap: using generation when downstream software needs structure; two runs give two different paragraphs | Templates, fill-in-the-blank |
| Extraction | For each document, which schema fields | Output feeds software (JSON, table rows); answer must appear verbatim in the source; you need 2 fields from a 50-page PDF | Field-level correctness, objectively checkable against the schema. Trap: asking the model to "write" or summarize when you need specific fields; extraction is more reliable and easier to validate | Regex, spaCy NER |
| Summarization | For each document or corpus, what condensed version | Reading burden is the cost; themes span the whole document; the answer connects page 2 to page 500 | Trap: RAG-chunking a transcript loses the arc, so global summaries want long context; if only two fields matter, reframe as extraction | Extractive summary: pull key sentences |
| LLM-as-classifier | For each text, which category | Prototyping, little labeled data, unpredictable text (slang, typos, novel queries) | Trap: prompting beats supervised models for prototyping but loses in production on cost, reliability, and often accuracy; one team cut monthly cost 60% moving to a small model with no quality drop | BERT or encoder fine-tune with labels; keyword rules |

### GenAI architecture choices (after picking the archetype)

| Choice | Pick this when | The trap |
|---|---|---|
| Semantic search | Conceptual queries, full sentences, user lacks the jargon ("charging thingy") | Fails on exact IDs: "Error 504" and "Error 505" sit close in vector space |
| Lexical search (BM25) | Part numbers, error codes, names, exact identifiers | Fails on paraphrase |
| Structured search (text-to-SQL) | Math, dates, hard filters ("under $50", totals, Q3) | Vector similarity cannot filter by dollar amount |
| Prompting / RAG | The model needs facts: policies, current data, your documents | "We must fine-tune so it knows our data" is the top GenAI misconception; it will hallucinate facts |
| Fine-tuning | The model needs behavior: brand voice, strict output format, a skill it has never seen; or the prompt repeats 2,000 words of examples every call | Slow to change, tied to a model version; wrong tool for facts |
| RAG (vs long context) | Precise lookup, one needle in a 10,000-page manual; cost scales with chunks, not corpus | Misses connections across documents the query does not resemble |
| Long context (vs RAG) | Synthesis across everything: themes over 50 contracts, messy data that chunks badly | Cost scales with tokens; lost in the middle |

### GenAI probes: questions that separate the lenses

**Retrieval (grounding), not pure generation, when:**
- The hallucination barrier: the cost of a wrong answer is high (medical dosages, legal precedents, refund policy).
- The news-cycle limit: the answer changed this morning; model knowledge is frozen at training.
- The intranet gap: the answer lives in private documents the model has never read.
- The receipts requirement: users ask "says who" and need a citation link, not a smoothie of blended sources.

**Extraction, not generation, when:**
- The schema requirement: output must land in a spreadsheet, SQL table, or JSON object.
- The highlight test: the answer must appear verbatim in the source; a single changed word breaks the downstream system.
- The trigger action: the output drives another piece of software, which needs machine-readable signals, not prose.

**Fine-tuning, not more prompting, when:**
- The token tax: the prompt is 2,000+ words because 50 examples get pasted on every call. Prompting is rent; fine-tuning is owning.
- Voice drift: answers are right but the tone is wrong; prompts are sticky notes that fall off.
- Instruction overload: rules too tangled for the model to hold in attention.
- The new-skill gap: the behavior was never in the base model (rare dialect, proprietary internal language). Prompting steers existing knowledge only.

**Long context, not RAG, when:**
- The global-summary problem: "recurring themes across these 50 contracts" needs the whole book, not the top 5 chunks.
- Connect the dots: the answer links page 2 to a contradiction on page 500 that the query does not resemble.
- Bad chunks: giant tables, codebases with dependencies, legal definitions 20 pages from their use; chunking cuts them apart.
- The vibe search: what you want is not explicitly written ("which email sounds most passive-aggressive").

Industry anchors: Intercom retrieves the refund policy verbatim because a made-up policy is a lawsuit; Jasper generates 50 ad variations because novelty is the goal; Brex extracts vendor, date, total from receipts because the database needs cells, not paragraphs; Bloomberg fine-tunes for JSON that never breaks schema (behavior) while Zendesk uses RAG to inject today's shipping policy (knowledge); AutoZone needs lexical search because "Part 99-B" and "99-C" are neighbors in vector space and completely different parts.

## Special archetypes

Archetypes the menus above don't fully cover. Each gets a few extra probes; weave them into the relevant GOATS steps.

**Coding and workflow agents.** Before build, name the deterministic artifact the agent operates from: a manifest, spec, dependency graph, migration plan, playbook, test oracle, or approval checklist. If none exists, the agent is doing design, not execution, and that's a different (harder) project. Force the invariants onto the assumptions list: which things must never be violated, which steps must happen in order, what rollback proves safe, and when the agent must stop instead of guessing. For parallel agents, isolation is a design control, not an optimization: worktrees, disjoint files, non-overlapping resources, or a merge arbiter. And ask where each piece of guidance belongs: the static prompt, a tool permission, a validator, or a decision-time micro-instruction injected at the point of action. A static prompt that grows a rule per incident is the smell that the controls are in the wrong layer.

**Vision and multimodal perception.** First classify the output, because the atomic unit changes with it: classify the whole image, locate objects, segment regions, track objects over time, read text, or describe the scene. Then name the operating constraint that rules the design: mAP, recall at a fixed false-positive budget, localization error, FPS or p99 latency, on-device compute, or downstream decision quality. Ask which error is worse, a false detection or a missed object; the answer sets the threshold and often the architecture (YOLO traded localization precision for real-time speed on purpose). For video, check whether temporal consistency matters: frame-level accuracy can still fail the workflow if detections flicker.

**Decision systems that commit capital.** When the prediction triggers purchases, pricing, trading, underwriting, or inventory commitments, add three probes: how much capital does one decision commit and is it reversible, what caps the aggregate exposure when the model is systematically wrong (not one bad call, a correlated thousand), and who can pull the kill switch, how fast. Zillow's offers collapse was not one bad prediction; it was uncapped aggregate exposure to a systematic one.

**Search and recommender extras.** Which bottleneck is the project actually about: recall, ranking quality, freshness, latency, or query understanding? Each is a different system. Two assumptions to force onto the list: every feature the model needs must exist at request time, not just in the warehouse after the fact, and offline lift must be assumed fragile until production traffic confirms it (training-production parity is the recurring silent killer). Name the cold-start path for new users and new items explicitly; "the model will figure it out" is not a path.

**LLM-as-judge and evaluation systems.** When the system's job is scoring other outputs, the Goal step changes: the metric is agreement, not quality. Require an expert-agreement target before build ("the judge matches senior reviewers X% on a held-out sample"), a named downstream behavior the score should predict, and a plan to test for position, verbosity, and self-preference bias. One boundary in the frame itself: the judge can rank and filter, but if its verdict gates something expensive or irreversible, a human sits behind it. A judge nobody calibrated is a random number generator with confidence.

**Data enrichment and synthetic labels.** First sharpen what's actually missing: labels, features, or user exposure. They imply different projects. If labels: can a proxy or synthetic label be validated against a small human-labeled sample before anyone trusts it at scale? If exposure: no data pipeline fixes a product nobody uses yet. Ask whether waiting for organic signal is genuinely the bottleneck or just the least glamorous option.

**Model training and infrastructure.** When the ask is about the training or serving stack itself, frame the failure type first: quality, stability, latency, cost, memory, throughput, or deployability. Each has a different owner and different signals. And put one alternative on the list that teams skip: the workaround that ships value while root cause waits (checkpoint-and-restart, caching, a smaller model). In infrastructure, contain-and-continue is a legitimate frame, not an admission of defeat.

## The fairness and high-stakes branch

Run this whenever the system decides about people: credit, hiring, health, housing, moderation, pricing, anything near a protected class. These join the assumptions and signals, not a compliance appendix.

- Who is harmed by a false positive, and who by a false negative? Name both groups; they're usually different people, and the trade-off between them is the real design decision.
- Are protected attributes involved, directly or by proxy? A model that never sees gender can price on engine size and discriminate anyway. List the suspected proxies at framing time so the diagnostic plan can ablate them.
- Commit to segmented reporting up front: performance by group, context, language, geography, device. An aggregate metric hides exactly the cell where the harm concentrates.
- Design the appeal path before launch: who can a wrongly-decided person talk to, and can the human actually override the system?
- Write the prohibited-use list: things the system could technically do that it must not.
- Add one safety stop signal to the dashboard alongside the quality one, pre-committed like every other stop criterion.

## The security and trust-control branch

Distinct from fairness: fairness protects people from the system, this protects the system from adversaries. Run it when the system enforces a guarantee (privacy indicators, permission gates, content authenticity, agent sandboxes, fraud controls).

- Write the threat model before choosing the architecture. Who is untrusted: the app, a plugin, the model itself, an admin, a vendor, an external service? What must remain impossible even if that actor is fully compromised?
- Prefer controls enforced by structure over controls enforced by instruction: isolation, permissions, hardware, secure enclaves, deterministic validators, separate services. A warning light, audit log, policy banner, or model refusal is not a control unless the attacker cannot bypass it.
- The AI-agent corollary: "the model was instructed not to do X" is not a security boundary. If X must never happen, X gets removed from the tool surface or blocked by a validator outside the model.
- Sometimes the reframe is architectural, not binary: Apple moved the camera indicator from a hardware LED to a software exclave, a third option invisible while the debate stayed "hardware vs software." Ask what isolated subsystem could enforce the guarantee before accepting either pole.

## Routing: four questions that pick the archetype

1. **What does the human do after they get the output?** Acts on one item: classification, regression, or extraction. Chooses between items: ranking, recommendation, or retrieval. Allocates limited resources: optimization. Investigates the unusual: anomaly detection. Plans ahead: forecasting. Reads and understands: generation or summarization. The output shape must match the human workflow.
2. **Is there a right answer you can learn from?** Clear labels and 2 to 20 categories: classification. Labels but 100+ categories: retrieval. A number: regression. No labels and free-form text needed: generation. No labels and structured output needed: extraction. No labels and the goal is discovering structure: clustering.
3. **Does order or time matter?** Shuffle test: if shuffling the rows destroys meaning, it is forecasting, not regression. If value is a property of the list, not the item, it is ranking, not prediction.
4. **Do latency and volume permit the choice?** Ask for p99 latency budget and peak QPS before finalizing. Tight budgets push toward precomputation, caching, smaller models, classical ML instead of LLMs, and rules instead of models. RAG at p99 under 100ms or 10K+ QPS means cache, precompute, or go classical.

## The five mismatches to watch for

1. **Classification when you need ranking.** "Predict which customers churn" but you can only call 100. Rank by P(churn) times customer value, not P(churn) > 0.5.
2. **Generation when you need retrieval.** "AI that answers policy questions" via fine-tuning. Policies change, weights freeze, hallucination is liability. Retrieve, then generate with citation.
3. **Prediction when you need optimization.** "Predict demand so we can staff." Knowing demand does not staff anyone; the forecast is an input to a constrained staffing decision.
4. **Forecasting when you need causal inference.** "Predict the impact of this promotion" from history where past promos were targeted, not random. Correlation is not causation; design an experiment or a causal model.
5. **ML when you need rules.** "AI to flag policy violations" when the policy is deterministic. If X > threshold is the whole logic, use a rule engine and save ML for judgment calls.

## Common combinations

Real systems stack archetypes. Retrieval then generation (RAG): large changing knowledge with citations. Classification then ranking: a two-stage funnel for efficiency. Forecasting then optimization: predict demand, then allocate under constraints. Clustering then classification: discover segments, then assign new items to them. Anomaly detection then generation: the flag plus the explanation. At retrieval scale (millions of items, sub-100ms), the industry pattern is two-tower retrieval to a top-1000 shortlist, then a richer reranker.
