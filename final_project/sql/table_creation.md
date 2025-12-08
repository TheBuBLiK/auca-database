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

CREATE TABLE action (
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

CREATE TABLE weapon_deco (
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

