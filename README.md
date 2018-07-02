# p5-mysql-upgrade-checker

## Description

- This is derived work of "MySQL Shell".
- Original work is licensed under GPLv2, so this script is licensed under GPLv2 too.

## Options

- `-h`, `--host`     Specify host for connection.
- `-P`, `--port`     Specify port for connection.
- `-u`, `--user`     Specify user for connection(recommend is root)
- `-p`, `--password` Specify password for connection.
- `-S`, `--socket`   Specify socket for connection(effective only when host = localhost)
- `-d`, `--database` Specify database for connection(This parameter doesn't affect script's behavior. Please leave it blank)
- `--execute`        Add this option for executing query(default: only print SQL)

## Example

```
$ perl p5-mysql-upgrade-checker.pl -uroot -S/usr/mysql/5.7.22/data/mysql.sock
get_removed_functions_check
================
SELECT routine_schema, routine_name, '', routine_type, UPPER(routine_definition) FROM information_schema.routines ;

get_partitioned_tables_in_shared_tablespaces_check
================
SELECT table_schema, table_name, CONCAT('Partition ', partition_name, ' is in shared tablespace ', tablespace_name) AS description FROM information_schema.partitions WHERE partition_name IS NOT NULL AND (tablespace_name IS NOT NULL AND tablespace_name != 'innodb_file_per_table') ;

obsolete_sql_mode_flags_check
================
SELECT routine_schema, routine_name, CONCAT(routine_type, ' uses obsolete DB2 sql_mode') FROM information_schema.routines WHERE FIND_IN_SET('DB2', sql_mode) ;

get_maxdb_sql_mode_flags_check
================
SELECT routine_schema, routine_name, CONCAT(routine_type, ' uses obsolete MAXDB sql_mode') FROM information_schema.routines WHERE FIND_IN_SET('MAXDB', sql_mode) ;

get_maxdb_sql_mode_flags_check
================
SELECT routine_schema, routine_name, CONCAT(routine_type, ' uses obsolete MAXDB sql_mode') FROM information_schema.routines WHERE FIND_IN_SET('MAXDB', sql_mode) ;

get_foreign_key_length_check
================
SELECT table_schema, table_name, 'Foreign key longer than 64 characters' AS description FROM information_schema.tables WHERE table_name IN (SELECT LEFT(SUBSTR(id, INSTR(id, '/') + 1), INSTR(SUBSTR(id, INSTR(id, '/') + 1), '_ibfk_') -1 ) FROM information_schema.innodb_sys_foreign WHERE LENGTH(SUBSTR(id, INSTR(id, '/') + 1)) > 64) ;

get_old_temporal_check
================
SET show_old_temporals = ON;
SELECT table_schema, table_name, column_name, column_type FROM information_schema.columns WHERE column_type LIKE 'timestamp /* 5.5 binary format */' ;
SET show_old_temporals = OFF;

get_mysql_schema_check
================
SELECT TABLE_SCHEMA, TABLE_NAME, 'Table name used in mysql schema in 8.0' AS WARNING FROM information_schema.tables WHERE LOWER(table_schema) = 'mysql' AND LOWER(table_name) IN ('catalogs', 'character_sets', 'collations', 'column_type_elements', 'columns', 'dd_properties', 'events', 'foreign_key_column_usage', 'foreign_keys', 'index_column_usage', 'index_partitions', 'index_stats', 'indexes', 'parameter_type_elements', 'parameters', 'routines', 'schemata', 'st_spatial_reference_systems', 'table_partition_values', 'table_partitions', 'table_stats', 'tables', 'tablespace_files', 'tablespaces', 'triggers', 'view_routine_usage', 'view_table_usage', 'component', 'default_roles', 'global_grants', 'innodb_ddl_log', 'innodb_dynamic_metadata', 'password_history', 'role_edges') ;

get_reserved_keywords_check
================
SELECT schema_name, 'Schema name' AS WARNING FROM information_schema.schemata WHERE schema_name IN ('ADMIN', 'CUBE', 'CUME_DIST', 'DENSE_RANK', 'EMPTY', 'EXCEPT', 'FIRST_VALUE', 'FUNCTION', 'GROUPING', 'GROUPS', 'JSON_TABLE', 'LAG', 'LAST_VALUE', 'LEAD', 'NTH_VALUE', 'NTILE', 'OF', 'OVER', 'PERCENT_RANK', 'PERSIST', 'PERSIST_ONLY', 'RANK', 'RECURSIVE', 'ROW', 'ROWS', 'ROW_NUMBER', 'SYSTEM', 'WINDOW') ;

get_utf8mb3_check
================
SELECT schema_name, CONCAT("schema's default character set: ", default_character_set_name) FROM information_schema.schemata WHERE schema_name NOT IN ('information_schema', 'performance_schema', 'sys') AND default_character_set_name IN ('utf8', 'utf8mb3') ;

get_zerofill_check
================
SELECT table_schema, table_name, column_name, column_type FROM information_schema.columns WHERE table_schema NOT IN ('sys', 'performance_schema', 'information_schema', 'mysql') AND ( column_type REGEXP 'zerofill' OR (column_type LIKE 'tinyint%' AND column_type NOT LIKE 'tinyint(4)%' AND column_type NOT LIKE 'tinyint(3) unsigned%') OR (column_type LIKE 'smallint%' AND column_type NOT LIKE 'smallint(6)%' AND column_type NOT LIKE 'smallint(5) unsigned%') OR (column_type LIKE 'mediumint%' AND column_type NOT LIKE 'mediumint(9)%' AND column_type NOT LIKE 'mediumint(8) unsigned%') OR (column_type LIKE 'int%' AND column_type NOT LIKE 'int(11)%' AND column_type NOT LIKE 'int(10) unsigned%') OR (column_type LIKE 'bigint%' AND column_type NOT LIKE 'bigint(20)%') ) ;

Check_table_command
================
CHECK TABLE `d1`.`t1`FOR UPGRADE;

Please add --execute argument if you want to execute statements.
```

```
$ perl p5-mysql-upgrade-checker.pl -uroot -S/usr/mysql/5.7.22/data/mysql.sock --execute
ok 1 - Usage of db objects with names conflicting with reserved keywords in 8.0
ok 2 - Usage of utf8mb3 charset
ok 3 - Usage of use ZEROFILL/display length type attributes
ok 4 - Issues reported by 'check table x for upgrade' command
ok 5 - Table names in the mysql schema conflicting with new tables in 8.0
ok 6 - Usage of old temporal type
ok 7 - Foreign key constraint names longer than 64 characters
ok 8 - usage of obsolete MAXDB sql_mode flag
ok 9 - get_obsolete_sql_mode_flags_check
ok 10 - Usage of partitioned tables in shared tablespaces
d1.test1() uses removed function ENCRYPT (consider using SHA2 instead)
not ok 11 - get_removed_functions_check
#   Failed test 'get_removed_functions_check'
#   at p5-mysql-upgrade-checker.pl line 121.
1..11
# Looks like you failed 1 tests of 11.
```
