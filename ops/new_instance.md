---
layout: page
title: Creating a new instance
nav_order: 20
parent: Ops (installation, running, ...)
---

(doc about adding a new domain to an existing instance is at the bottom of this page)

# Creating a new instance

This document is mainly a memo for not forgetting steps when trying to install a new instance (db + backend + frontend).
It barely explains the reasons behind these operations.
Reasons have to be found in the project repositories directly, in ops best-practices, or in my head🤷‍♂️.
Many of these steps may only apply if you do the same kind of setup as ours (very AWS specific).

## Required from others

- the domain name (then adding the domain name and TXT entry for the certificate)
- the supported languages and the default one
- the login-module client id and secret
- `itemPlatformId` to communicate with tasks
- the title on top left (inc. language-specific ones) and in browser tabs for the frontend
- a logo if any
- whether skills are enabled
- whether groups should be shown in the left menu

## SSL / DNS

- create a new certificate in the same region as the load balancer (+ TXT entry in the DNS to validate it)
- create a new DNS entry to the load balancer

## Database

Run the following action as a db admin. I suppose the new instance is called "myinstance" (short token to be chosen).

### Create the user/database
```
CREATE USER 'alg-myinstance-backend'@'%' IDENTIFIED BY '...'; # for the backend API
CREATE USER 'alg-myinstance-admin'@'%' IDENTIFIED BY '...'; # for migration and commands
CREATE DATABASE alg_myinstance_prod;
GRANT SELECT, INSERT, UPDATE, DELETE, DROP, CREATE TEMPORARY TABLES ON `alg_myinstance_prod`.* TO `alg-myinstance-backend`@`%`;
GRANT ALL PRIVILEGES ON alg_myinstance_prod.* TO 'alg-myinstance-admin'@'%';
```

### Create the db schema:

If you do not have a running DB, install the schema on the backend and run migration (see "Seeding the database" section of the backend README).

Otherwise, you can copy another instance schema:
```
mysqldump --no-data --triggers --routines --events --add-drop-trigger --no-create-db --skip-opt -h <HOSTNAME> -u <ADMIN_USER> --protocol=TCP <EXISTING_DB_NAME> -p > db-schema-dump.sql
```
Search and replace `<ADMIN_USER>` as trigger definer, to `alg-myinstance-admin`. Remove "@SESSION.SQL_LOG_BIN" and "GLOBAL.GTID_PURGED" lines. Then:
```
mysql -h <HOSTNAME> -u alg-myinstance-admin --protocol=TCP  alg_myinstance_prod -p < db-schema-dump.sql
```

### Creating initial data

#### Copy the migration table

Copy the  `gorp_migrations` from the model database to the new one:
```
mysqldump -h <hostname> --no-create-db --no-create-info -u <user> -p <sourcedatabase> gorp_migrations > db-migration-dump.sql
```
Remove the line not related to data insertion and:
```
mysql -h <HOSTNAME> -u alg-myinstance-admin --protocol=TCP  alg_myinstance_prod -p < db-migration-dump.sql
```

#### Create the default groups
```
INSERT INTO `groups` (id, name, type, text_id, description, is_open, is_public, frozen_membership) values (2, 'TempUsers', 'Base', 'TempUsers', 'temporary users', 0, 0, 1);
INSERT INTO `groups` (id, name, type, text_id, description, is_open, is_public, frozen_membership) values (3, 'AllUsers', 'Base', 'AllUsers', 'AllUsers', 0, 0, 1);
INSERT INTO `groups` (`id`, `name`, `type`, `text_id`, `description`, is_open, is_public, frozen_membership) VALUE (4, 'NonTempUsers', 'Base', 'NonTempUsers', 'non-temporary users',0, 0, 1);
INSERT INTO `groups_groups` (parent_group_id, child_group_id) values (3, 2), (3, 4)
```

#### Create supported languages for items
```
INSERT INTO languages VALUES ('en', 'English'), ('fr', 'French');
```

#### Create the initial item and set it as the root activity for the groups
```
SET FOREIGN_KEY_CHECKS=0;
INSERT INTO items (id, type, default_language_tag) VALUES (1, 'Chapter', 'en');
SET FOREIGN_KEY_CHECKS=1;
INSERT INTO items_strings (item_id, language_tag, title) VALUES (1, 'en', 'Root chapter (rename me)');
UPDATE `groups` SET root_activity_id = 1 WHERE id IN (2, 3);
INSERT INTO permissions_granted (group_id, item_id, source_group_id, origin, can_view) VALUES (2, 1, 2, 'group_membership', 'content') ;
INSERT INTO permissions_granted (group_id, item_id, source_group_id, origin, can_view) VALUES (3, 1, 3, 'group_membership', 'content') ;
```

#### Create the initial skill if enabled
```
SET FOREIGN_KEY_CHECKS=0;
INSERT INTO items (id, type, default_language_tag) VALUES (2, 'Skill', 'en');
SET FOREIGN_KEY_CHECKS=1;
INSERT INTO items_strings (item_id, language_tag, title) VALUES (2, 'en', 'Root skill (rename me)');
UPDATE `groups` SET root_skill_id = 2 WHERE id IN (2, 3);
INSERT INTO permissions_granted (group_id, item_id, source_group_id, origin, can_view) VALUES (3, 2, 3, 'group_membership', 'content') ;
```

#### Create an initial "admin" group
```
INSERT INTO `groups` (id, name, type, is_open, is_public, frozen_membership) VALUES (5, 'Platform admins', 'Other', 0, 0, 0);
INSERT INTO permissions_granted (group_id, item_id, source_group_id, origin, can_view,  can_grant_view, can_watch, can_edit, is_owner) VALUES (5, 1, 5, 'group_membership', 'solution', 'solution_with_grant', 'answer_with_grant', 'all_with_grant', 1) ;
INSERT INTO group_managers (group_id, manager_id , can_manage, can_grant_group_access , can_watch_members) VALUES (3, 5, 'memberships_and_group', 1, 1);
```

If skills are enabled:
```
INSERT INTO permissions_granted (group_id, item_id, source_group_id, origin, can_view,  can_grant_view, can_watch, can_edit, is_owner) VALUES (5, 2, 5, 'group_membership', 'solution', 'solution_with_grant', 'answer_with_grant', 'all_with_grant', 1) ;
```
### Create default "platform"
```
insert into platforms (id, name, `regexp`, priority) values (0,'default', '.*',0);
```

## Config

### Frontend

On AlgoreaConfig, fork an existing frontend branch and change:

- the build config
- in the main config:
  - `itemPlatformId` (for communication with tasks)
  - `oauthClientId` (for communication with the login-module)
  - `defaultActivityId`: `1` if you have followed the steps above
  - `defaultSkillId`: `2` if enabled and using our default
  - `allUsersGroupId`: `3` if using our default
  - `title` and `languageSpecificTitles`.
  - `searchApiUrl`: undefined for now
  - `forumServerUrl`: undefined for now

### Propagation end-point

In the ops repository, in `src/backend-propagation`, run `sls deploy --stage abc-prod --aws-profile algorea` to deploy the propagation end-point.

Update the "ALGOREA_SERVER__PROPAGATION_ENDPOINT" in the backend config with the end-point url.

### Backend

In the AWS parameter store, create a `/alg/myinstance-prod/Auth`, `/alg/myinstance-prod/Database`, and `/alg/myinstance-prod/Token` of type `SecureString` and copy-update value from other instances.

For the key pair, generate one this way:
```
openssl genrsa --out private_key.pem 2048
openssl rsa -in private_key.pem -pubout -out public_key.pem
```
And then replace the line breaks by `\n`.

Send the public key to the task manager if such tasks are used, and the token platform name if you chose it yourself.

On AlgoreaConfig, fork one backend branch and update the propagation end-point with the one given in the previous section.

### Ops

* In the ACM (certificate manager), create a new certificate for the new domain with DNS validation.

* On AlgoreaConfig, in the `opsbot_prod` branch, add the new deployment environment in the env file. Force redeployment of the main branch of "AlgoreaOps" via the CI  so that the bot is redeployed.

* Deploy the frontend and backend using the slack bot (`deploy frontend|backend <new_env_name> <version>`)

* Create manually the `released` tag on the "*-static-serve" and "server" lambda functions pointing to the deployed version

* re-compute the permissions via the slack bot: `command backend <env> db-recompute`

## ALB

* In EC2/LB, create a new target rule for the lambda function, pointing to the `released` tag of the lambdas, enable multi-header on the backend target

* In EC2/ALB:
  - add the new certificate
  - copy the other rules to match the hosts (frontend and backend)
  - depending on the default language you want to redirect the unknown path to, adapt the alias rule

## Allow access to assets

If a new domain is used, add it to allowed origin in the response header policy of the CloudFront distribution (in policies > response headers).

## Boostrap permissions on a first user

Once your first user has signed in, add him as a manager of group `5`:
```
INSERT INTO group_managers (group_id, manager_id, can_manage, can_grant_group_access, can_watch_members) VALUES (5,<new_user_group_id>,'memberships_and_group',1,1);
```
(find the the `group_id` of the user using `SELECT group_id, login FROM users WHERE temp_user = 0;` )

Then, the user can add himself (using the webapp) as the member of the group, which will give him access to manage content.


## Forum

To setup the forum, first, in the ops repository:
- create a directory in `envionments/forum` based on the existing ones
- refer it in the `environments/deployments.yaml` file

That should create the forum. Then copy the `wss` url to the frontend config.

## Search

TODO
