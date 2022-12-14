---
layout: page
title: Creating a new instance
nav_order: 20
parent: Ops (installation, running, ...)
---

# Creating a new instance

This document is mainly a memo for not forgetting steps when trying to install a new instance (db + backend + frontend), it barely explains
the reasons behind these operations. Reasons have to be found in the project repositories directly, in ops best-practices, or in my head🤷‍♂️. Many of these steps may only apply if you do the same kind of setup as ours (very AWS specific).


## SSL / DNS

- create a new certificate in the same region as the load balancer (+ TXT entry in the DNS to validate it)
- create a new DNS entry to the load balancer


## Database

Run the following action as a db admin. I suppose the new instance is called "myinstance".

### Create the user/database
```
CREATE USER 'algorea-myinstance'@'%' IDENTIFIED BY '...';
CREATE DATABASE algorea_myinstance;
GRANT ALL PRIVILEGES ON algorea_myinstance.* TO 'algorea-myinstance'@'%';
```

### Create the db schema:

If you do not have a running DB, install the schema on the backend and run migration (see "Seeding the database" section of the backend README).

Otherwise, you can copy another instance schema:
```
mysqldump --no-data --triggers --routines --events --add-drop-trigger --no-create-info --no-create-db --skip-opt -h <HOSTNAME> -u <ADMIN_USER> --protocol=TCP <EXISTING_DB_NAME> -p > db-schema-dump.sql
```
Search and replace `<ADMIN_USER>` as trigger definer, to `algorea-myinstance`. Then:
```
mysql -h <HOSTNAME> -u algorea-myinstance --protocol=TCP  algorea_myinstance -p < db-schema-dump.sql
```

### Creating initial data

Note: If you have followed the "Seeding the database" section of the backend README, some of the following operations may have already
performed by the "install" command. This command does not do all the job currently, it probably needs to be improved a bit.

#### Create the default groups
```
INSERT INTO `groups` (id, name, type, description, is_open, is_public, frozen_membership) values (2, 'TempUsers', 'Base', 'temporary users', 0, 0, 1);
INSERT INTO `groups` (id, name, type, description, is_open, is_public, frozen_membership) values (3, 'AllUsers', 'Base', 'AllUsers', 0, 0, 1);
```

#### Create supported languages for items
```
INSERT INTO LANGUAGES VALUES ('en', 'English'), ('fr', 'French');
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

#### Create an initial "admin" group
```
INSERT INTO `groups` (id, name, type, is_open, is_public, frozen_membership) VALUES (5, 'Platform admins', 'Other', 0, 0, 0);
INSERT INTO permissions_granted (group_id, item_id, source_group_id, origin, can_view,  can_grant_view, can_watch, can_edit, is_owner) VALUES (5, 1, 5, 'group_membership', 'solution', 'solution_with_grant', 'answer_with_grant', 'all_with_grant', 1) ;
INSERT INTO group_managers (group_id, manager_id , can_manage, can_grant_group_access , can_watch_members) VALUES (3, 5, 'memberships_and_group', 1, 1);
```

### Recompute computed (cached) values for relations and permissions
Run on the backend: `make db-recompute`

### First login
Once your first user has signed in, add him as a manager of group `5`:
```
INSERT INTO group_managers (group_id, manager_id, can_manage, can_grant_group_access, can_watch_members) VALUES (5,<new_user_group_id>,'memberships_and_group',1,1);
```
(find the the `group_id` of the user using `SELECT group_id, login FROM users WHERE temp_user = 0;` )

Then, the user can add himself (using the app) in the managed group to get the other access.

## Frontend

### Config

In the config, change
- `apiUrl` (should be on the same domain to use cookies)
- `itemPlatformId` (for communication with tasks)
- `oauthClientId` (for communication with the login-module)
- `defaultActivityId`: `1` if you have followed the steps abobe)
- `title` and `languageSpecificTitles`.

### Compile for prod

Compilation has to be done one language at a time as the build override the previous build (language).

What follows make some assumption about the deployment (using S3) that should be described in another doc.

#### English
```
ng build --configuration production-en --base-href / --deploy-url //assets.algorea.org/deployments/myinstance/en/
aws s3 sync ./dist/algorea/ s3://algorea-static/deployments/myinstance --acl public-read --exclude "*/index.html" --cache-control 'max-age=86400' --profile ...
aws s3 cp ./dist/algorea/en/index.html s3://algorea-static/deployments/myinstance/en/index.html --acl public-read --cache-control 'max-age=300' --profile ...
```

#### French
```
ng build --configuration production-fr --base-href / --deploy-url //assets.algorea.org/deployments/myinstance/fr/
aws s3 sync ./dist/algorea/ s3://algorea-static/deployments/myinstance --acl public-read --exclude "*/index.html" --cache-control 'max-age=86400' --profile ...
aws s3 cp ./dist/algorea/fr/index.html s3://algorea-static/deployments/myinstance/fr/index.html --acl public-read --cache-control 'max-age=300' --profile ...
```

## AWS Lambda & ALB

* For static and backend: change the config, save to a new version, update/create the alias, restore the initial config.

* Config to change for the backend: ALGOREA_AUTH__CLIENTID, ALGOREA_AUTH__CLIENTSECRET, ALGOREA_DATABASE__DBNAME, ALGOREA_DATABASE__PASSWD, ALGOREA_DATABASE__USER, ALGOREA_SERVER__DOMAINOVERRIDE

* In EC2, create a new target rule for the lambda alias, enable multi-header on the backend target

* In EC2/ALB, copy the other rules to match the hosts.

