### Basic SELECT ###
1. List all regions

SELECT id, name, climate
FROM region
ORDER BY name;

2. List all settlements with their region name

SELECT s.id,
       s.name AS settlement_name,
       s.type,
       r.name AS region_name
FROM settlement AS s
LEFT JOIN region AS r ON s.region_id = r.id
ORDER BY r.name, s.name;

3. Find all players and their guilds

SELECT p.id,
       p.nickname,
       g.name AS guild_name
FROM player AS p
LEFT JOIN guild AS g ON p.guild_id = g.id
ORDER BY g.name, p.nickname;

4. All quests with their reward

SELECT q.id,
       q.name,
       q.difficulty,
       r.gold
FROM quest AS q
LEFT JOIN reward AS r ON q.reward_id = r.id
ORDER BY q.difficulty, q.name;

5. All weapons and their damage

SELECT w.id,
       w.name,
       ws.damage,
       ws.rarity
FROM weapon AS w
JOIN weapon_stats AS ws ON ws.weapon_id = w.id
ORDER BY ws.damage DESC;

### Aggregation - GROUP BY ###

1. Number of players per guild

SELECT g.id,
       g.name,
       COUNT(p.id) AS player_count
FROM guild AS g
LEFT JOIN player AS p ON p.guild_id = g.id
GROUP BY g.id, g.name
ORDER BY player_count DESC;

2. Average player level by guild

SELECT g.id,
       g.name,
       AVG(ps.level) AS avg_level
FROM guild AS g
JOIN player AS p ON p.guild_id = g.id
JOIN player_stats AS ps ON p.stats_id = ps.id
GROUP BY g.id, g.name
ORDER BY avg_level DESC;

3. Count of dungeons by region

SELECT r.id,
       r.name AS region_name,
       COUNT(d.id) AS dungeon_count
FROM region AS r
LEFT JOIN dungeon AS d ON d.region_id = r.id
GROUP BY r.id, r.name
ORDER BY dungeon_count DESC;

### Filtering - WHERE ###

1. Hard quests with deadline this month

SELECT id, name, deadline
FROM quest
WHERE difficulty = 'Hard'
  AND deadline >= DATE_TRUNC('month', CURRENT_DATE)
  AND deadline < (DATE_TRUNC('month', CURRENT_DATE) + INTERVAL '1 month')
ORDER BY deadline;


2. All legendary weapons

SELECT w.id, w.name, ws.damage, ws.critical_chance
FROM weapon AS w
JOIN weapon_stats AS ws ON ws.weapon_id = w.id
WHERE ws.rarity = 'Legendary'
ORDER BY ws.damage DESC;