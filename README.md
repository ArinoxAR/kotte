# Bomberman World — [your game title]

**Student:** Md Anas Hossain Arif

> Replace the prompts below and remove these instructions. Keep answers brief. Your implementation explanations and reflection belong in the presentation; you do not need to write them out here. Complete the final section only if seeking Pass with Distinction (VG).

## How to play

- **Controls:** Move with **WASD** or the **Arrow keys**. Press **Space** to place a bomb. **R** to restart the game; **Escape** or **Q** to quit the game.
- **Objective and rules:** Destroy all enemies to win the game. The player loses when hit by an enemy or by a bomb blast. Bombs are solid, but their owner can move away from the bomb after placing it. The player can have up to three active bombs at once. Bomb blasts can trigger other bombs, creating chain reactions. Destroying crates and enemies gives score, with a higher score multiplier for multiple targets destroyed in the same explosion sequence. If the player is killed at the same time as the final enemy, the round is lost.
- **Configurations / demo setup:** [how to select gameplay/stress configurations and use any special setup or debug keys; write “default” where appropriate]

## Features and validation

(delete this pre-amble before submitting the assignment) Complete the five required features and three extensions of your choice. For each row, document **at least one useful check: what you tested, what you observed, and how the result compares with the expected behaviour**. One or two sentences per feature are enough. Choose a check that could reveal a mistake, such as a boundary case or interaction.

Use the practical checks in the assignment to validate the acceptance conditions during development; this table is a small sample of that work. Manual play in a small, repeatable setup is sufficient. You can also use a debugger, temporary logging, or small scenarios set up in code. Screenshots, recordings, automated tests etc are welcome, but not required. Be ready to explain the relevant rules in your code.

| Feature | Check, observed result, and expected behaviour |
| --- | --- |
| 1. Player damage and death | I tested both enemy contact and being caught by a bomb blast. In both cases the player entered the Lost state and could no longer continue normal gameplay, matching the expected behaviour. |
| 2. Round states and objective | I tested destroying the enemies until none remained. The round changed to Won, while losing to an enemy or blast changed it to Lost. The player could then restart the round. |
| 3. Safe restart | I played a round with active bombs, explosions, destroyed entities, and changed blast range, then restarted multiple times. The old round state and entities were cleared, the same level layout was recreated, and the player and gameplay state were reset correctly. |
| 4. Solid bombs with owner escape | I placed a bomb and tested movement around it. The bomb blocked movement like other solid entities, while the owner could first move away from the bomb and escape from its collision area. |
| 5. Blast-range pickup | I collected the blast-range pickup and verified that the displayed blast range increased. The range could increase up to the configured maximum and the pickup did not block movement or stop a blast. |
| 6. Multiple bombs per owner | I placed three bombs and verified that all three could remain active at the same time. A fourth bomb was rejected until one of the existing bombs detonated, and placing multiple bombs on the same tile was prevented. |
| 7. Chain reactions | I placed multiple bombs within blast range and allowed the first bomb to detonate. Its blast triggered the other bomb before its own fuse expired. I also tested a longer chain and verified that each bomb detonated only once without freezing the game or producing duplicate explosions. |
| 11. 	Scoring and combos | I destroyed crates and enemies with bomb explosions and checked that the score increased. I also tested a chain reaction destroying multiple targets and observed that the combo multiplier increased for the targets destroyed during the explosion sequence. |

Example: “I ended a round with live bombs and restarted three times to check that old-round state was cleared. No old explosions appeared, and bomb capacity reset each time.” Report your actual results, including any failures or remaining limitations; stating that something “works” does not describe a check.

**Known issues:** Two independent bombs that detonate during the same explosion-resolution frame can contribute to the same combo because the current scoring system groups targets resolved during that frame. This does not affect gameplay correctness, but means the combo system is a simplified scoring model rather than tracking separate chain-reaction groups. No other known issues were found after testing.

## Sources and assistance

This project builds on the shared Kotte foundations.

AI assistance was used during development for debugging code review, architectural discussion. I implemented the changes in my project, tested the behaviour myself in the game and fixed complier issues that occurred during development

No external code was copied directly into the project beyond the provided course foundations and material.

## Architectural investigation — VG only

Delete this section for a G submission. For VG, pose a focused engineering question about one decision in your implementation, investigate it, and draw a conclusion from evidence. Documenting the explanation already required for G is not enough. Aim for half a page to one page, excluding small tables, diagrams, or code excerpts. You can use the same decision in your presentation too.

### Decision

The current implementation separates gameplay consequences from structural changes. When a blast reaches an entity, the game adds a DestroyEntityRequest instead of immediately destroying the entity. Bombs are treated similarly but first create a BombDetonated fact so that their own explosion can be resolved. After gameplay facts have been resolved, apply_structural_mutations() performs the requested entity destruction.

A plausible alternative would be to destroy an entity immediately inside resolve_blast_tile() as soon as its tile is reached. This would reduce the amount of deferred state, but it could invalidate entity handles and spatial-grid references while the same explosion or chain reaction is still being processed.

The relevant implementation is in resolve_blast_tile(), resolve_gameplay_facts(), request_bomb_detonation(), and apply_entity_destructions().

[State the question you are investigating and the implementation decision it concerns. Describe your current approach and a plausible alternative. Identify the relevant code. You do not need to implement the alternative.]

### Method

I investigated the decision using the normal game configuration with multiple bombs placed so that one bomb's blast could trigger another bomb. I tested a single bomb, a two-bomb chain reaction, and a longer chain reaction with several bombs placed within each other's blast ranges.

I observed the state while the explosions were being processed and checked that bombs remained available long enough for their detonation facts to be resolved before the structural mutation phase removed them. I also tested restarting after explosions to check that destroyed entities did not remain in the spatial grid or active-handle lists.

[How did you investigate the decision? Give enough detail to reproduce the key observation: setup, input, configuration/seed, and the test, trace, debugger inspection, or other method used.]

### Evidence

With immediate bomb destruction, the bomb reached by the first blast would be removed before its own explosion could be processed. The chain reaction therefore requires the affected bomb to remain a valid entity while the detonation fact is generated and processed.

With the current deferred approach, the first bomb creates a BombDetonated fact. When its blast reaches another bomb, request_bomb_detonation() marks that bomb as requested and adds another detonation fact. resolve_gameplay_facts() continues processing newly added facts before the destruction phase runs.

A chain such as:
Bomb A → Bomb B → Bomb C

therefore produced three explosions, while each bomb was destroyed only once. After the chain reaction finished, the game continued normally and the destroyed bombs were removed from the relevant entity and spatial-grid structures.

[What did you actually observe in your game? Include the relevant result or a small excerpt, table, or link to supporting material.]

### Interpretation



[Explain why it happened using the relevant code or architecture. Analyse at least one edge case, failure, defect, limitation, or trade-off illustrated by the investigation. Discuss relevant consequences for correctness, maintainability, performance, and/or scalability; you do not need to cover all four.]

### Conclusion

[When is your approach appropriate, what limits it, and when might an alternative become preferable? Base the conclusion on your evidence. Confirming that the current approach is suitable is a valid result.]

<!-- Submission: commit and push this README with your source to GitHub or the internal Gitea server. Submit the final commit link and a runnable Release ZIP from that version in Studium. Ensure the teacher can access the repository throughout assessment. -->
