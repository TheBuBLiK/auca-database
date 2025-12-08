### Example ###
# when a player completes a quest: 

# Insert into player_quest_log (if not already there or mark as completed—in our simple schema it just means an entry exists).
# Increase the player's level.
# Optionally increase guild prestige.

BEGIN;

# Ensure the quest is in the player's log
INSERT INTO player_quest_log (player_id, quest_id)
VALUES (:player_id, :quest_id)
ON CONFLICT (player_id, quest_id) DO NOTHING;

# Increase player level
UPDATE player_stats AS ps
SET level = level + :level_gain
FROM player AS p
WHERE p.stats_id = ps.id
  AND p.id = :player_id;

# If player has a guild, increase its prestige
UPDATE guild AS g
SET prestige = COALESCE(prestige, 0) + :prestige_gain
FROM player AS p
WHERE p.guild_id = g.id
  AND p.id = :player_id;

COMMIT;

# If something goes wrong:
ROLLBACK;

### Another example ###
EXPLAIN ANALYZE
SELECT q.id,
       q.name,
       q.difficulty,
       q.deadline
FROM player_quest_log AS pql
JOIN quest AS q ON q.id = pql.quest_id
WHERE pql.player_id = 42;
CREATE INDEX idx_player_quest_player ON player_quest_log(player_id);
EXPLAIN ANALYZE
SELECT id, name, deadline
FROM quest
WHERE difficulty = 'Hard'
  AND deadline >= CURRENT_DATE
ORDER BY deadline;
CREATE INDEX idx_quest_difficulty_deadline
ON quest (difficulty, deadline);
CREATE INDEX idx_quest_deadline_not_null
ON quest (deadline)
WHERE deadline IS NOT NULL;

