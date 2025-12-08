### Backup and recovery strategy ###
1. Logical backup
pg_dump -Fc -f mmorpg_db_full.backup mmorpg_db

2. Restore full backup
createdb mmorpg_db
pg_restore -d mmorpg_db mmorpg_db_full.backup

To include create/drop database:
pg_restore -C -d postgres mmorpg_db_full.backup

3. Schema-only backup
pg_dump -s -f mmorpg_schema.sql mmorpg_db

Then restore like:
psql -d mmorpg_db -f mmorpg_schema.sql

4. Single-table backup(player):
pg_dump -Fc -t player -f player_table.backup mmorpg_db

Restore only that table:
pg_restore -d mmorpg_db player_table.backup

5. Simple backup schedule:
Every night at 2am:
0 2 * * * pg_dump -Fc mmorpg_db > /backups/mmorpg_db_$(date +\%Y\%m\%d).backup