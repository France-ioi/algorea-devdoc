---
layout: page
title: Database Migrations
nav_order: 200
parent: Backend
---

# Database migration

We use the package [sql-migrate](https://github.com/rubenv/sql-migrate) to handle database migrations.

Each migration is a *.sql file named `$DATETIME$_migration_name.sql` located in `db/migrations/`.

`$DATETIME$` format is `YYMMDDHHII` -> 2301011000 for the 1st of January 2023 at 10:00.

The migrations are run in ascending order of filename, thus in ascending order of `$DATETIME$`.

The table `gorp_migration` in the database keeps track of which migrations have been run, *by filename*, and when.


## Create a migration

In order to create a new migration, create the migration file `$DATETIME$_migration_name.sql`.

The file is split between two blocks: `-- +migrate Up` to make the migrations, and `-- +migrate Down` to undo it.

```
-- +migrate Up
UPDATE `items_strings` SET `image_url` = SUBSTRING(`image_url`, 1, 2048) WHERE LENGTH(`image_url`) > 2048;
ALTER TABLE `items_strings` MODIFY COLUMN `image_url` varchar(2048) DEFAULT NULL COMMENT 'Url of a small image associated with this item.';

-- +migrate Down
ALTER TABLE `items_strings` MODIFY COLUMN `image_url` TEXT DEFAULT NULL COMMENT 'Url of a small image associated with this item.';
```

In case you need to write complex SQL statement like a trigger, **the DELIMITER keyword will NOT work**. Instead, you'll have to use the `-- +migrate StatementBegin`/`-- +migrate StatementEnd` block to be able to use `;`:

```

-- +migrate StatementBegin
CREATE TRIGGER itemTextIdUniqueUpdate BEFORE UPDATE ON `items` FOR EACH ROW BEGIN
  SET @counter = 1;
  SET NEW.`text_id_unique` = OLD.`text_id`;
  WHILE exists (SELECT 1 FROM `items` WHERE `items`.`text_id_unique` = NEW.`text_id_unique`) DO
    SET NEW.`text_id_unique` = CONCAT(OLD.`text_id`, @counter);
    SET @counter = @counter + 1;
  END WHILE;
END
-- +migrate StatementEnd
```

Also, note that **each migration is run in a transaction**, so you don't need to create one yourself.


## Run the migrations

Run the command `AlgoreaBackend db-migrate` to run the migrations that weren't previously run.


## Undo the last migration

Run the command `AlgoreaBackend db-migrate-undo`.


## Resources

* [sql-migrate Documentation](https://github.com/rubenv/sql-migrate)
