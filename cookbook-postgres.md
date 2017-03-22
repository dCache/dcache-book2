Chapter 24. PostgreSQL and dCache
=================================

Table of Contents

Installing a PostgreSQL Server
Configuring Access to PostgreSQL
Performance of the PostgreSQL Server



PSQL is used for various things in a DCACHE system: The SRM, the pin manager, the space manager, the replica manager, the CELL-BILLING, and the PNFS server might make use of one or more databases in a single or several separate PSQL servers.

The SRM, the pin manager, the space manager and the replica manager will use the PSQL database as configured at cell start-up in the corresponding batch files. The CELL-BILLING will only write the accounting information into a database if it is configured with the option `-useSQL`. The PNFS server will use a PSQL server if the `pnfs-posgresql` version is used. It will use several databases in the PSQL server.

Installing a PSQL Server
========================

The preferred way to set up a PSQL server should be the installation of the version provided by your OS distribution; however, version 8.3 or later is required.

Install the PSQL server, client and JDBC support with the tools of the operating system.

Initialize the database directory (for PSQL version 9.2 this is `/var/lib/pgsql/9.2/data/`) , start the database server, and make sure that it is started at system start-up.

    PROMPT-ROOT service postgresql-9.2 initdb
    Initializing database:                                     [  OK  ]
    PROMPT-ROOT service postgresql-9.2 start
    Starting postgresql-9.2 service:                           [  OK  ]
    PROMPT-ROOT chkconfig postgresql-9.2 on

Configuring Access to PSQL
==========================

In the installation guide instructions are given for configuring one PSQL server on the admin node for all the above described purposes with generous access rights. This is done to make the installation as easy as possible. The access rights are configured in the file `database_directory_name/data/pg_hba.conf`. According to the installation guide the end of the file should look like

    ...
    # TYPE  DATABASE    USER        IP-ADDRESS        IP-MASK           METHOD
    local   all         all                                             trust
    host    all         all         127.0.0.1/32                        trust
    host    all         all         ::1/128                             trust
    host    all         all         HostIP/32          trust

This gives access to all databases in the PSQL server to all users on the admin host.

The databases can be secured by restricting access with this file. E.g.

    ...
    # TYPE  DATABASE    USER        IP-ADDRESS        METHOD
    local   all         postgres                      ident sameuser
    local   all         pnfsserver                    password
    local   all         all                           md5
    host    all         all         127.0.0.1/32      md5
    host    all         all         ::1/128           md5
    host    all         all         HostIP/32          md5

To make the server aware of this you need to reload the configuration file as the user `postgres` by:

    PROMPT-ROOT su - postgres
    PROMPT-POSTGRES pg_ctl reload

And the password for e.g. the user `pnfsserver` can be set with

    PROMPT-POSTGRES psql template1 -c "ALTER USER pnfsserver WITH PASSWORD 'yourPassword'"

The PNFS server is made aware of this password by changing the variable `dbConnectString` in the file `/usr/etc/pnfsSetup`:

    ...
    export dbConnectString="user=pnfsserver password=yourPassword"

User access should be prohibited to this file with

    PROMPT-ROOT chmod go-rwx /usr/etc/pnfsSetup

Performance of the PSQL Server
==============================

On small systems it should never be a problem to use one single PSQL server for all the functions listed above. In the standard installation, the CELL-REPLICAMNGR is not activated by default. The CELL-BILLING will only write to a file by default.

Whenever the PSQL server is going to be used for another functionality, the impact on performance should be checked carefully. To improve the performance, the functionality should be installed on a separate host. Generally, a PSQL server for a specific funcionality should be on the same host as the DCACHE cell accessing that PSQL server, and the PSQL server containing the databases for CHIMERA should run on the same host as CHIMERA and the CELL-PNFSMNGR cell of the DCACHE system accessing it.

It is especially useful to use a separate PSQL server for the CELL-BILLING cell.

> **Note**
>
> The following is *work-in-progress*.

Create PSQL user with the name you will be using to run PNFS server. Make sure it has `CREATEDB` privilege.

    PROMPT-USER psql -U postgres template1 -c "CREATE USER johndoe with CREATEDB"
    PROMPT-USER dropuser pnfsserver
    PROMPT-USER createuser --no-adduser --createdb --pwprompt pnfsserver

| Component        | Database Host                                                                | Database Name           | Database User | Database Password |
|------------------|------------------------------------------------------------------------------|-------------------------|---------------|-------------------|
| SRM              | `srm.db.host`or if not set: `srmDbHost` or if not set: localhost             | srm                     | dcache        | `--free--`        |
| PinManager       | `pinManagerDatabaseHost` or if not set: `srmDbHost` or if not set: localhost | pinmanager              | pinmanager    | `--free--`        |
| CELL-REPLICAMNGR | `replica.db.host` or if not set: localhost                                   | replica                 | dcache        | `--free--`        |
| PNFS server      | localhost                                                                    | admin, data1, exp0, ... | pnfsserver    | --free--          |
| billing          | `billingDatabaseHost` or if not set: localhost                               | billing                 | dcache        | `--free--`        |


