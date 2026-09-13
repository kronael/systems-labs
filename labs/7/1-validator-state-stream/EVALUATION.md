> Spoilers. Open only when stuck.

# Validator state stream — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

Faults fire at named barriers, never on a timer. When the store has
acknowledged the write for account 4711's update at slot S, the store stops
accepting writes for thirty seconds while the generator keeps driving
transfers. When the update for account 4711 at slot S arrives, the fault hook
delays every subsequent notification by a fixed interval, so the plugin
itself is the slow party. When the update for account 4711 at slot S has been
delivered, the controller freezes the validator long enough that slots in the
frozen window are skipped when it resumes; verification reads the
slot-status stream to learn which slots were skipped or marked dead and
asserts that no confirmed answer ever reflected them. The controller also
kills the validator after a named slot is rooted and restarts it against the
preserved ledger, kills the learner's store-side path during the write for account 4711's
update at slot S, and starts
the whole system with an empty store against a validator holding prior
state.

Verification queries at moments named relative to the notification stream:
after an account update arrives but before any status for its slot, between
a slot's processed and confirmed statuses, and after the root. It does not
inspect private functions or require a named design. It compares every
answer against the expected per-account history derived from the generator
seed and the observed slot stream, at exact account and slot identities,
never counts alone.
