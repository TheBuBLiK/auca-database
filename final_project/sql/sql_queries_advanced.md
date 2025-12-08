### Advanced SELECT ###

1. Total gold reward per player

WITH player_rewards AS (
    SELECT pql.player_id,
           SUM(r.gold) AS total_potential_gold
    FROM player_quest_log AS pql
    JOIN quest AS q ON pql.quest_id = q.id
    JOIN reward AS r ON q.reward_id = r.id
    GROUP BY pql.player_id
)
SELECT p.id,
       p.nickname,
       COALESCE(pr.total_potential_gold, 0) AS total_potential_gold
FROM player AS p
LEFT JOIN player_rewards AS pr ON pr.player_id = p.id
ORDER BY total_potential_gold DESC, p.nickname;

2. Ranking players by level inside each guild

SELECT g.name AS guild_name,
       p.nickname,
       ps.level,
       RANK() OVER (PARTITION BY g.id ORDER BY ps.level DESC) AS level_rank_in_guild
FROM player AS p
JOIN player_stats AS ps ON p.stats_id = ps.id
LEFT JOIN guild AS g ON p.guild_id = g.id
ORDER BY guild_name, level_rank_in_guild, p.nickname;

3. Damage percentile for weapons

SELECT w.name,
       ws.damage,
       PERCENT_RANK() OVER (ORDER BY ws.damage) AS damage_percent_rank
FROM weapon AS w
JOIN weapon_stats AS ws ON ws.weapon_id = w.id
ORDER BY ws.damage DESC;

4. Players with at least one "Hard" quest

SELECT p.id,
       p.nickname
FROM player AS p
WHERE EXISTS (
    SELECT 1
    FROM player_quest_log AS pql
    JOIN quest AS q ON pql.quest_id = q.id
    WHERE pql.player_id = p.id
      AND q.difficulty = 'Hard'
);

5. Dungeons with average enemy level

WITH dungeon_enemies AS (
    SELECT d.id AS dungeon_id,
           d.name AS dungeon_name,
           AVG(es.level) AS avg_enemy_level,
           COUNT(e.id) AS enemy_count
    FROM dungeon AS d
    LEFT JOIN enemy AS e ON e.dungeon_id = d.id
    LEFT JOIN enemy_stats AS es ON e.enemy_stats_id = es.id
    GROUP BY d.id, d.name
)
SELECT dungeon_id,
       dungeon_name,
       enemy_count,
       avg_enemy_level
FROM dungeon_enemies
ORDER BY avg_enemy_level DESC NULLS LAST;