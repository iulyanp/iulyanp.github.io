---
layout: post
title:  "How can you create a superuser in MySQL?"
description: 'How can you create a superuser in MySQL?'
slug: 'how-can-you-create-a-super-user-in-mysql'
date: 2020-07-11 08:02:00 +0300
category: database
tags: [mysql, create, user]
icon: table
---

This tutorial will show you how to set up a new MySQL account as a super user with privileged access to the databases.

### You must first log in as the root user, who has access to the CREATE USER capability.

To create a new user with a password, enter the following command:

```
CREATE USER 'johndoe'@'localhost' IDENTIFIED BY 'secret_pass';
```

The new user currently has no access rights to the databases. The new user's privileges must then be granted, which comes next. The following privileges are available to users:

- ALL PRIVILEGES - gives complete root access to the databases, if no database is specified, will have global access to it.
- CREATE – build new databases or tables
- DROP - remove databases or tables
- DELETE - remove rows from tables 
- INSERT - adding rows to tables
- SELECT - to browse databases, use the SELECT command.
- UPDATE - update table rows using UPDATE
- GRANT OPTION: Add or remove rights for other users.

### Create a superuser.

We must provide this new user complete root access to the whole database in order to transform it into a superuser, which entails GRANTING ALL PRIVILEGES:

```
GRANT ALL PRIVILEGES ON *.* TO 'johndoe'@'localhost' WITH GRANT OPTION;
```

The new user now has root-like permission, therefore the work is complete.


### Then make a second account with an identical username and grant it full root access:

```
CREATE USER 'johndoe'@'%' IDENTIFIED BY 'secret_pass';

GRANT ALL PRIVILEGES ON *.* TO 'johndoe'@'%' WITH GRANT OPTION;
```

Both superuser accounts "johndoe"@"localhost" and "johndoe"@"%" have full control over everything.

It is only possible to connect using the 'johndoe'@'localhost' account from a local host. The account "johndoe"@"%" employs the wildcard "%" in the host part, allowing connections from any host.

### Use the SHOW GRANTS command to confirm the permissions granted to the new user:

```
SHOW GRANTS FOR johndoe;
```

### Reload all the privileges once everything is in order and each adjustment to the users will become effective right away.

```
FLUSH PRIVILEGES;
```