# Reflection — Day 17 (≤ 200 words)

Answer briefly, in your own words. This is graded on reasoning, not length.

1. **The flywheel.** Day 13 emitted agent traces; today you turned them into an
   eval set and DPO pairs that Day 22 will train on. Which step in
   `traces → Bronze → datasets` would break most silently in production if you
   got it wrong — and how would you detect it?

2. **Decontamination.** Your run dropped 2 of 3 preference pairs because their
   prompts were in the eval set. What concretely goes wrong if you *skip* this
   step and train on those pairs? How would the lie show up in your metrics?

3. **Point-in-time.** The naive join leaked a future `lifetime_spend` into the
   training row. Describe one feature in a system you know that would be
   dangerous to join without an `ASOF`/point-in-time guard.

4. **Graph vs vector.** From `kg_demo.py`, name one question the knowledge graph
   answers well that flat chunk retrieval (`embed.py`) would struggle with, and
   one where the graph is overkill.

_Write your answers below._

1. The quietest failure would be incorrect trace-to-Bronze normalization: fields
   can still look valid while spans are assigned to the wrong trace or ordered
   incorrectly. I would detect it with schema checks, row-count reconciliation,
   unique trace/span keys, and sampled end-to-end assertions from raw traces to
   generated examples.

2. Skipping decontamination lets training memorize eval prompts and preferred
   answers. Evaluation would then measure recall of seen examples rather than
   generalization, producing suspiciously high win-rate or accuracy that drops
   on a fresh, held-out prompt set.

3. A dangerous example is a customer's chargeback status. Joining the latest
   status to an earlier fraud-decision event would reveal a future outcome and
   make the model appear able to predict chargebacks before they occurred.

4. The graph is useful for "Where does a widget ship from?" because it traverses
   `widget -> accessory -> Hanoi` across separate facts. It is overkill for a
   direct fact such as "How long is the gadget warranty?", where one retrieved
   chunk already contains the answer.
