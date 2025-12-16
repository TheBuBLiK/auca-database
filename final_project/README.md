# MMO RPG Database

A PostgreSQL database schema for an MMO RPG world. The project models the core gameplay loop and world-building entities: regions and settlements, guilds and players, inventories and loot, quests, dungeons, enemies/bosses, spells/abilities, mounts, and mythical creatures.

---

## Tech Stack

- **DBMS:** PostgreSQL
- **Key features used:** foreign keys, `ON DELETE` actions, unique constraints, indexes, CTEs (`WITH`), window functions (`RANK`, `PERCENT_RANK`), transactions, `EXPLAIN ANALYZE`, backup/restore tools (`pg_dump`, `pg_restore`)

---

## Schema Overview

### World / Locations
- **`region`** — top-level world areas (name, climate)
- **`settlement`** — towns/cities/villages; belongs to a region (`region_id`)
- **`dungeon`** — located in a region; can have a reward (`reward_id`)

### Social / NPC
- **`guild`** — guilds based in a settlement (`town_id`)
- **`citizen`** — NPC citizens linked to their home town (`home_town_id`)
- **`mythical_creature`** — creatures associated with a region (`region_id`)

### Player & Progression
- **`player`** — main entity: nickname, occupation, home town, guild, mount, and links to build/loadout
- **`player_stats`** — RPG stats (HP/MP, class/race, level, etc.). One-to-one with `player` via `stats_id` (UNIQUE)
- **`player_quest_log`** — many-to-many between players and quests (UNIQUE `(player_id, quest_id)`)

### Quests & Rewards
- **`quest`** — quests with difficulty, deadline, and optional reward
- **`guild_quest`** — many-to-many between guilds and quests (UNIQUE `(guild_id, quest_id)`)
- **`reward`** — gold and/or an item reward (weapon)

### Combat / Items
- **`inventory`** — container for items
- **`potion`** — belongs to inventory (`inventory_id`, cascades on delete)
- **`weapon`** — may belong to inventory (`inventory_id`)
- **`weapon_stats`** — one-to-one weapon stats (`weapon_id` UNIQUE, cascades on weapon delete)
- **`weapon_desc`** — one-to-one lore/creator (`weapon_id` UNIQUE, cascades on weapon delete)

### Abilities / Magic
- **`ability_menu`** — a “loadout menu” container
- **`ability`** — linked to a menu (`ability_menu_id`)
- **`spellbook`** — container for spells
- **`magic_spell`** — spells linked to a spellbook (`spellbook_id`, cascades on delete)

### Enemies
- **`enemy_stats`** — reusable stat template (hp/dmg/level/exp drop)
- **`enemy`** — dungeon enemies referencing `enemy_stats` and `dungeon`
- **`boss`** — bosses referencing `enemy_stats` and `dungeon`

### Mounts
- **`mount`** — mounts (speed, special ability) linked from `player`

---

## DDL (Tables)

> Full schema used in the project:

```sql
CREATE TABLE region (
    id SERIAL PRIMARY KEY,
    name VARCHAR(80) NOT NULL,
    climate VARCHAR(30)
);

CREATE TABLE settlement (
    id SERIAL PRIMARY KEY,
    name VARCHAR(80) NOT NULL,
    type VARCHAR(30),
    ruler VARCHAR(80),
    currency VARCHAR(30),
    primary_language VARCHAR(30),
    region_id INTEGER REFERENCES region(id) ON DELETE SET NULL
);

CREATE TABLE guild (
    id SERIAL PRIMARY KEY,
    name VARCHAR(80) NOT NULL,
    prestige INTEGER,
    master VARCHAR(80),
    size INTEGER,
    town_id INTEGER REFERENCES settlement(id) ON DELETE SET NULL
);

CREATE TABLE citizen (
    id SERIAL PRIMARY KEY,
    home_town_id INTEGER REFERENCES settlement(id) ON DELETE SET NULL,
    name VARCHAR(80) NOT NULL,
    occupation VARCHAR(30)
);

CREATE TABLE inventory (
    id SERIAL PRIMARY KEY
);

CREATE TABLE potion (
    id SERIAL PRIMARY KEY,
    inventory_id INTEGER REFERENCES inventory(id) ON DELETE CASCADE,
    name VARCHAR(80) NOT NULL,
    type VARCHAR(30),
    duration INTEGER,
    buff VARCHAR(80)
);

CREATE TABLE weapon (
    id SERIAL PRIMARY KEY,
    name VARCHAR(80) NOT NULL,
    inventory_id INTEGER REFERENCES inventory(id) ON DELETE SET NULL
);

CREATE TABLE weapon_stats (
    id SERIAL PRIMARY KEY,
    type VARCHAR(30),
    material VARCHAR(30),
    damage INTEGER,
    durability INTEGER,
    rarity VARCHAR(30),
    critical_chance NUMERIC(5,2),
    elemental_type VARCHAR(30),
    special_ability VARCHAR(80),
    weapon_id INTEGER UNIQUE REFERENCES weapon(id) ON DELETE CASCADE
);

CREATE TABLE weapon_desc (
    id SERIAL PRIMARY KEY,
    weapon_id INTEGER UNIQUE REFERENCES weapon(id) ON DELETE CASCADE,
    lore VARCHAR(80),
    creator VARCHAR(80)
);

CREATE TABLE reward (
    id SERIAL PRIMARY KEY,
    weapon_id INTEGER REFERENCES weapon(id) ON DELETE SET NULL,
    gold INTEGER
);

CREATE TABLE dungeon (
    id SERIAL PRIMARY KEY,
    reward_id INTEGER REFERENCES reward(id) ON DELETE SET NULL,
    name VARCHAR(80) NOT NULL,
    difficulty VARCHAR(30),
    region_id INTEGER REFERENCES region(id) ON DELETE SET NULL
);

CREATE TABLE enemy_stats (
    id SERIAL PRIMARY KEY,
    type VARCHAR(30),
    hp INTEGER,
    damage INTEGER,
    speed INTEGER,
    level INTEGER,
    exp_drop INTEGER
);

CREATE TABLE enemy (
    id SERIAL PRIMARY KEY,
    name VARCHAR(80),
    enemy_stats_id INTEGER REFERENCES enemy_stats(id) ON DELETE CASCADE,
    dungeon_id INTEGER REFERENCES dungeon(id) ON DELETE SET NULL
);

CREATE TABLE boss (
    id SERIAL PRIMARY KEY,
    enemy_stats_id INTEGER REFERENCES enemy_stats(id) ON DELETE CASCADE,
    name VARCHAR(80),
    dungeon_id INTEGER REFERENCES dungeon(id) ON DELETE SET NULL
);

CREATE TABLE ability_menu (
    id SERIAL PRIMARY KEY,
    name VARCHAR(80)
);

CREATE TABLE ability (
    id SERIAL PRIMARY KEY,
    name VARCHAR(80) NOT NULL,
    type VARCHAR(30),
    damage INTEGER,
    cooldown INTEGER,
    ability_menu_id INTEGER REFERENCES ability_menu(id) ON DELETE SET NULL
);

CREATE TABLE spellbook (
    id SERIAL PRIMARY KEY
);

CREATE TABLE magic_spell (
    id SERIAL PRIMARY KEY,
    name VARCHAR(80) NOT NULL,
    type VARCHAR(30),
    elemental_type VARCHAR(30),
    spellbook_id INTEGER REFERENCES spellbook(id) ON DELETE CASCADE,
    damage INTEGER,
    damage_type VARCHAR(30),
    mana_cost INTEGER,
    special_ability VARCHAR(80)
);

CREATE TABLE mount (
    id SERIAL PRIMARY KEY,
    name VARCHAR(80) NOT NULL,
    type VARCHAR(30),
    speed INTEGER,
    special_ability VARCHAR(80)
);

CREATE TABLE player_stats (
    id SERIAL PRIMARY KEY,
    hp INTEGER,
    mp INTEGER,
    race VARCHAR(30),
    class VARCHAR(30),
    damage INTEGER,
    speed INTEGER,
    level INTEGER
);

CREATE TABLE player (
    id SERIAL PRIMARY KEY,
    ability_menu_id INTEGER REFERENCES ability_menu(id) ON DELETE SET NULL,
    stats_id INTEGER UNIQUE REFERENCES player_stats(id) ON DELETE CASCADE,
    nickname VARCHAR(80) NOT NULL,
    occupation VARCHAR(30),
    inventory_id INTEGER UNIQUE REFERENCES inventory(id) ON DELETE SET NULL,
    spellbook_id INTEGER UNIQUE REFERENCES spellbook(id) ON DELETE SET NULL,
    guild_id INTEGER REFERENCES guild(id) ON DELETE SET NULL,
    mount_id INTEGER REFERENCES mount(id) ON DELETE SET NULL,
    home_town_id INTEGER REFERENCES settlement(id) ON DELETE SET NULL
);

CREATE TABLE quest (
    id SERIAL PRIMARY KEY,
    name VARCHAR(80) NOT NULL,
    difficulty VARCHAR(30),
    location VARCHAR(80),
    outcome VARCHAR(80),
    description VARCHAR(80),
    deadline DATE,
    reward_id INTEGER REFERENCES reward(id) ON DELETE SET NULL
);

CREATE TABLE guild_quest (
    id SERIAL PRIMARY KEY,
    guild_id INTEGER REFERENCES guild(id) ON DELETE CASCADE,
    quest_id INTEGER REFERENCES quest(id) ON DELETE CASCADE,
    UNIQUE (guild_id, quest_id)
);

CREATE TABLE player_quest_log (
    id SERIAL PRIMARY KEY,
    player_id INTEGER REFERENCES player(id) ON DELETE CASCADE,
    quest_id INTEGER REFERENCES quest(id) ON DELETE CASCADE,
    UNIQUE (player_id, quest_id)
);

CREATE TABLE mythical_creature (
    id SERIAL PRIMARY KEY,
    region_id INTEGER REFERENCES region(id) ON DELETE SET NULL,
    name VARCHAR(80) NOT NULL,
    type VARCHAR(30),
    alignment VARCHAR(30),
    size VARCHAR(30),
    special_ability VARCHAR(80)
);
```

## Indexes

```sql
CREATE INDEX idx_settlement_region ON settlement(region_id);
CREATE INDEX idx_guild_town ON guild(town_id);
CREATE INDEX idx_weapon_inventory ON weapon(inventory_id);
CREATE INDEX idx_weapon_stats_weapon ON weapon_stats(weapon_id);
CREATE INDEX idx_weapon_desc_weapon ON weapon_desc(weapon_id);
CREATE INDEX idx_potion_inventory ON potion(inventory_id);
CREATE INDEX idx_dungeon_region ON dungeon(region_id);
CREATE INDEX idx_enemy_dungeon ON enemy(dungeon_id);
CREATE INDEX idx_boss_dungeon ON boss(dungeon_id);
CREATE INDEX idx_magic_spell_book ON magic_spell(spellbook_id);
CREATE INDEX idx_player_guild ON player(guild_id);
CREATE INDEX idx_player_home_town ON player(home_town_id);
CREATE INDEX idx_player_quest_player ON player_quest_log(player_id);
CREATE INDEX idx_player_quest_quest ON player_quest_log(quest_id);
CREATE INDEX idx_guild_quest_guild ON guild_quest(guild_id);
CREATE INDEX idx_guild_quest_quest ON guild_quest(quest_id);
CREATE INDEX idx_mythical_creature_region ON mythical_creature(region_id);
```

---

### Notes / Assumptions

Many relationships are optional and use ON DELETE SET NULL to keep historical entities (e.g., a player can remain even if their guild or town is removed).
Some entities are one-to-one by design:
player ↔ player_stats (stats_id UNIQUE)
weapon ↔ weapon_stats and weapon_desc (weapon_id UNIQUE)
Quest completion is represented simply by presence in player_quest_log.
