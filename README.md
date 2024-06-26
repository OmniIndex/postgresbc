# PostgresBC

![alt PostgresBC](readmeimages/postgresbc_small.png)


+ Getting Started
+ Users
+ Drop and Delete Objects
+ Chains and Schems
+ Blocks or Tables
+ Smart Contracts
+ Peer-to-Peer Replication
+ More Information

## Getting Started  

Once installed either via the deb package or within a Docker container, it is understandable that you will want to get playing  
as soon as possible. This is where this section of the README is really useful. We will start with an empty PostgresBC instance  
and then do the following using psql.

+ Login to psql with the postgres user
+ Change the postgres users password
+ Initialize postgresbc without passing a key
+ Create a new user with login privileges
+ Alter the user to add a password
+ Logout with the postrgres user and log in with our new user
+ Create a new Chain to hold our log blocks
+ Create a new log Block within that Chain
+ Add data to a Block
+ Searching in an encrypted object
+ Conclusion

### Login to psql with the postgres user

In a command window on a system that has postgres-client installed type

```text
sudo -upostgres psql<enter>
```

This will connect you to the PostgresBC instance and may ask for your systrem password (NOT PostgresBC)  


![alt psql logged in as postgres user](readmeimages/psqli_loggedin.png)


### Change the postgres users password

You do this by typing

```SQL
ALTER USER postgres WITH PASSWORD 'new password';<enter>
```

Ok, so now we are ready to start using PostgresBC fully. lets start by initializing the system. We will let PostgresBC create a key for us, using OpenSSL.

### initialize postgresbc without passing a key

```OIDXQL
initialize '';
```

The return will be a boolean true or false. If false check the logs in
/var/logs/postgresql/postgresql{version}.log  
If you have setup a peer network within your /etc/postgresbc.conf file, then all of the attached nodes  
will also be initialized.

### Create a new user with login privileges

Now we will move on and create an initial user. We **should not**  to use the postgres super user for all of our tasks.

```SQL
CREATE ROLE my_awsome_user WITH LOGIN;<enter>
```

Once entered you will see the message

```TEXT
CREATE ROLE
```

### Alter the user to add a password

This has created a role that has login capabilities, but as yet does not have a password.  

```SQL
ALTER USER my_awsome_user WITH PASSWORD 'my_awsome_password';<enter>
```

Once entered PostgresBC will return

```TEXT
ALTER ROLE
```

Now we will logout of psql and log back in as our new user.

### Logout with the postrgres user and log in with our new user

To do this at the prompt type

```TEXT
\q<enter>
```

You should now be back at the command prompt where you need to type

```TEXT
psql -Umy_awsome_user -dpostgres<enter>
```

You will be prompted for the password, so type in the one you gave the user

### Create a new Chain to hold our log blocks

A chain in PostgresBC is an enhanced postgres Schema. If you are unsure about the differences between a  
Postgresql and other DB platforms schemas plesea see the Chain section below.

Here we will be creating a PostgresBC Chain, to do this in your psql command prompt, logged in as your new user type.

```SQL
CREATE CHAIN IF NOT EXISTS my_aswome_chain<enter>
```

After a few seconds you should recieve the message

```TEXT
NOTICE:  Your chain is ready for use.
```

### Create a new log Block within that Chain

As with Chains being special Schemas. Blocks are enahnced Tables. To create a Block within the psql window type

```SQL
CREATE BLOCK IF NOT EXISTS my_awsome_chain.my_awsome_logs (logged_date TEXT, called_by TEXT, messageencrypt TEXT, log_file TEXT);<enter>
```

All being well you will have the message below returned to you.

```TEXT 
CREATE TABLE
```

This tells you that all is good and that your new Block has been created. So, what have we just done?  
When we tell PostgresBC to create a Block it adds some special fields in to it. These are ones that you have not defined, but are there to enable a Web 3.0 use and also to enable AI Analytics to take place. If you wish to see theses you could try typing the psql command \dn into the command window. However; as yopu will quickly see PostgresBC is very locked down. The best way to examine a Blocks structure is to use a tool such as PGAdmin and log in as the postgres user. With that you will be able to see a Block and tables structure.

![alt PGAdmin Block Structure](readmeimages/pgadmin_block.png)

You will instantly see that there are far more objects (fields) in place than the 4 that we created. If we break these down:  

+ messagesearchable
+ message_pgbc

These are the messageencrypt objects. When you append the word encrypt to an object name you are telling PostgresBC to encrypt all data that is sent to the object. These two enable you to view (Only if you have INSERT, UPDATE & SELECT priviliges) and Searc (Only if you have SELECT priviliges).  

+ hash
+ priorhash

These hold a unique hash for this object and also the hash of the object inserted directly before this one.

+ oidxid

This is a unique serial identifier for the Block

+ ai_*

These hold the AI descriptors for this Block based on the Boudica thesauruses that have been placed in the Chain.

### Add data to a Block

As you saw in the creation of a block, we added an object that holds encrypted data. In this section we will show you how to add data to the encrypted object and other objects within a block.

In the psql command window type 

```SQL
INSERT INTO my_awsome_chain.my_awsome_logs (logged_date, called_by, message_encrypt, log_file) VALUES
('06-26-2024 08-38', 'Tutorial', 'This is a small block of text that simon has added for the tutorial', 'README');<enter>
```

This will probably not give you the response that you were expecting! As stated before PostgresBC is very locked down. While you have CREATE priviliges on your Chain, you do not yet have any access priviliges. To do this you will need to log out of psql

```TEXT
\q<enter>
```

And log back in as the postgres user

```TEXT
su -u postgres psql<enter>
```

Now you are able to set priviliges on the data structure. For this tutorial  we will give my_awsome_user all priviliges on the Block.

```SQL
GRANT SELECT, INSERT, UPDATE ON my_awsome_chain.my_awsome_logs TO my_awsome_user;<enter>
```

You will then be returned with

```TEXT
GRANT
```

Now log back in as my_awsome_user, when you are at the logged in command prompt, up arrow until you see the insert command and then press the enter key. This shoudl then return 

```SQL
INSERT 0 1
```

If it does not you will see an error stsing where there has been an issue with the insert. Follow this and then re-insert the data.

### Searching in an encrypted object

If you now go back to PGAdmin and open up a query tool (Right hand mouse click on the Block). In the query tool type

```SQL
SELECT hash, message_pgbc, messagesearchable FROM my_awsome_chain.my_awsome_logs;
```

Press the Run button and you will be presented with the results. As you will see we have a Genesis hash and both the message fields have hex data in them. The postgres user is unable to view this data as by default they do not have full access rights.  

Back in your psql command window type

```SQL
SELECT hash FROM my_awsome_chain.my_awsome_logs;<return>
```

You will see that you can return this object. Now let us return an encrypted object, to do this type.

```SQL
SELECT * FROM my_awsome_chain.my_awsome_logs;<enter>
```

You will be returned with a CSV output from your search with the encrypted data showin in plain text. Next we will do two more searches. The first brining back the hash using a search within the encryprted object, and the second the same search but this time searching on text that is not in the encrypted object.  
For the first search type

```SQL
SELECT hash FROM my_awsome_chain.my_awsome_logs WHERE { my_awsome_chain.my_awsome_logs.messageencrypt LIKE '%of text%' };<enter>
```

and for the second type

```SQL
SELECT hash FROM my_awsome_chain.my_awsome_logs WHERE { my_awsome_chain.my_awsome_logs.messageencrypt LIKE '%not in here%' };
```

Before we look at the return, let me explain the query.  

+ Dot notation for blocks and tables

With PostgresBC whenever we are dealing with tables, or block s we must allways provide the full path. So here we have the Chain 'my_awsome_chain' then a . followed by the block name 'my_awsome_logs

+ Dot notation on encrypted objects

If we want to place a where clause on an encrypted object then we need to add the OIDXQL language additions. These are 

+ Surrounding the clause with curly braces {}
+ Full . notation on the object names Chain/Schema.Block/Table.Object

In the folloowing image you will see my returned output. The first query giving me a result, and the second is empty. This is because the fiirst query found the text 'of text' within the messageencrypt object while the second searched for 'not in here' which is not in the object.  

![alt Searching encrypted objects in psql](readmeimages/searchableencrypted_psql.png)

### Conclusion

As you have seen working with PostgresBC is almost identicle to working with PostgreSQL with teh exception of Chains, Block, Encrypted Objects and user access. Because PostgresBC is a fork of PostgreSQL you are able to use the same tools and drivers that you currently use to access postgreSQL with no changes. For more indepth tutorials on

+ The Boudica AI engines SLM
+ The Boudica AI autonomous data management
+ Peer-to-Peer Replication
+ Smart Contracts

Please see our other specific video walk throughs.

## Users

Users are controlled mainly by the standard postgresql user management. So you can create, alter and manage users, with the postgres user commands. CREATE ROLE, ALTER ROLE etc.  
It is important to remebere that PostgresBC also has its own rules when it comes to user access of the data.  
To be able to search within encrypted data a user needsthe SELECT privilege on the block/table being searched.  
To see encrypted data dein its' original format a user needs the SELECT, INSERT & UPDATE privileges on the block being accessed.  
To add block/data to a chain/table a user must have INSERT, UPDATE privileges for that chain/table  
The commands DROP, DELETE, UPDATE are not accessible to any user.  

## Drop and Delete Objects

By default a Web 3 data store cannot have objects deleted, updated or dropped. This can run foul of some local data protection laws. To overcome this PostgresBC comes with the ability to override the standard Web 3 immutable data rule. When the system is initialized an overide file is crearted within teh /etc folder on teh system. If you wish to DELETE , DROP or Update data then you have to be logged in as teh postgres super user, and at teh end of your command you need to append the keys password.

```SQL
DROP BLOCK myBlock:override password;
```

This override will not enable an unauthorized person to search data. Only perform drop, delete and update commands.  

## Chains and Schemas

in PostgreSQL a schema is simmilar to a databaser in other RDMS such as Oracle, MySQL and MSSQL. It is a place where a range of objects are stored and accessed. The same is true of PostgresBC with the addition  that when you create a CHAIN it will be pre-poulated with teh Boudica AI objects, allowing other objects within the Chain to also have access to the PostgresBC AI engine.


## Blocks or Tables

PostgresBC can utilize bith a Block and a Table. However to use teh Boudica AI engine you must create teh object as a Block.  

Both Blocks and Tables can store and have seartchablke encrypted data,

## Smart Contracts 

These are a special type of block that when created will also create a trigger function. The trigger function  
shel script is placed in the
/var/lib/postgresql/pgbc/sql  
folder. To calll it create a block like so  
CREATE BLOCK IF NOT EXISTS {schema}.{USER}_contract (contractCount SERIAL, your other objects);  
This will automatically trigger the smart contract script

## Peer-to-Peer Replication

This happens automatically if the node details are placed in to the postgresbc.conf file, which can be found  
in the /etc folder. The make up of peer nodes is  
node0 <address>  
node0port <port>  
node0user <username>  
node0password <password>  
node0database <database>   
node1 <address>  
node1port <port>  
node1user <username>  
node1password <password>  
node1database <database>  

## Further Information

For further information please view our short tutorial videos and vistit our wenb site <www.omniindex.io>
