# Probing Reference

Facilitation techniques for the Facilitative working mode. The goal: the user authors the thinking. You provide the structure that makes it rigorous. These are postures, not a script — pull from them as the conversation needs, spend time where thinking is thin, skip what the user has already resolved.

## The seven probing categories

1. **Purpose / WHY.** What problem does this game solve for its players, and for whom? Why now? What happens if it is not built? Strip features back to the underlying player need or fantasy.
2. **Players / context.** Who plays this, in what situation? What do they play today instead? What is the moment of need — the itch the game scratches? Ground the player in a real person, not an archetype; mark anything ungrounded `[ILLUSTRATIVE]`.
3. **Scope / boundaries.** What is explicitly in, explicitly out? What kind of MVP is this — proving the core loop is fun, proving the experience model, building a platform base, or proving someone will pay? The answer changes what "minimum" means. Probe for what keeps almost making the list.
4. **Success / evidence.** Push every adjective to a measurement. "Players will love it" — measured how? Connect each metric back to the differentiator. Name counter-metrics: what the game should NOT optimize (session length is not success if the goal is a tight, replayable loop).
5. **Risk / assumptions.** What has to be true for this to work? What is the riskiest assumption? What breaks this?
6. **Prioritization / trade-offs.** If you could ship only one thing, what is it? What are you willing to cut? What is non-negotiable?
7. **Edge cases / failure modes.** What does the unhappy path look like — the player who bounces, the run that desyncs, the save that corrupts? Real failure modes for this game, not invented error states.

## Journey facilitation

When player journeys are warranted, walk each one as story structure, not a use-case list:

- **Opening scene** — where we meet this player, their situation, the need or itch present.
- **Rising action** — the beats they take, what they discover or decide.
- **Climax** — the moment the game delivers its core fun; the thing they could not do before.
- **Resolution** — their new state, what is next.

After each journey, name what could go wrong at the climax and the recovery path. Explicitly name the capability each journey reveals and link it to an FR — journeys produce requirements; make the link visible.

## Six critical assumptions to surface

Every game PRD rests on assumptions. Six sink projects when wrong:

1. **Player demand.** The desire for this kind of play is real and felt now.
2. **Willingness to switch.** Players will leave the games they play now. Inertia is the default competitor.
3. **Technical feasibility.** It can be built with the engine, team, and budget you have.
4. **Business viability.** Cost to build and run is justified by the revenue or value.
5. **Differentiation.** There is a reason to play this over the alternatives.
6. **Timing.** The market and technology are ready now.

## The PRD / solution-design boundary

The PRD says WHAT and WHY; solution design says HOW. Hold the line:

- **In the PRD:** capabilities, behaviors, player outcomes, constraints, acceptance criteria.
- **In solution design (game architecture):** engine and technology choices, data models, system contracts, build and deployment topology.
- **The test:** if removing a sentence would not change what the game does for the player, it probably belongs in the addendum or in architecture, not the PRD body.
