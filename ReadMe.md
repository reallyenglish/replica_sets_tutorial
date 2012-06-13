This small project allows you to try MongoDB's Replica Set on your local machine and aims to help you understand how it works.
Following the instruction below, you will have a replica set consisted by two mongodbs (and one arbiter).

## Prepare

This creates empty directories for mongodbs.

    % cd /path/to/replicaset (top of this repo)
    % ./prepare.sh

## Start db1 (master)

    % cd /path/to/replicaset
    % mongod -f db1.conf

## Check db1

    % mongo --port 27021 
    > rs.help()
    ...
    > rs.status()
    {
      "startupStatus" : 3,
      "info" : "run rs.initiate(...) if not yet done for the set",
      "errmsg" : "can't get local.system.replset config from self or any seed (EMPTYCONFIG)",
      "ok" : 0
    }

You can also check the status on browser with the following URL

    http://localhost:28021/_replSet

## Initiate the set

    % mongo --port 27021
    > rs.initiate()
    {
      "info2" : "no configuration explicitly specified -- making one",
      "me" : "ono-mba.local:27021",
      "info" : "Config now saved locally.  Should come online in about a minute.",
      "ok" : 1
    }

    > rs.status()
    > rs.isMaster()

You can also check the status change on the browser.

    http://localhost:28021/_replSet

## Insert some documents

    (% mongo --port 27021)
    > db
    test
    > db.foo.insert({x:1})
    > db.foo.insert({x:2})
    > db.foo.find()

And try to check oplog on the browser.

    http://localhost:28021/_replSetOplog?_id=0

## Add db2 to the replica set

**Start db2**

    % mongod -f db2.conf

**Add db2 to the set**

Instead of initiating, add db2 to the set created on db1. Thus you have to
connect to db1.

    % mongo --port 27021  # this is db1
    % rs.add("ono-mba.local:27022")

Replace 'ono-mba.local' to your PC's host name. Mongodb doesn't accept you to
set localhost here.

Then you can see db2 added to the set on 

    http://localhost:28021/_replSet

## Add db3 as arbiter

Imagin the situation db2 thinks db1 is down. Chance is that there can be a
network problem on db2. Mongodb solves this problem with a feature called
voting. Voting requires more than three nodes to define either one is right. So
we run a process only for voting which is called arbiter.

http://www.mongodb.org/display/DOCS/Replica+Sets+-+Voting

**Start db3**

Note small oplogsize is set on db3.conf since arbiter doesn't store data
actually.

    % mongod -f db3.conf

**Add db3 as arbiter to the set**

Likewise db2, you connect db1 to add db3.

    % mongo --port 27021  # this is db1
    % rs.addArb("ono-mba.local:27023")

Then you can see db3 added to the set as arbiter on 

    http://localhost:28021/_replSet


## Try failover

Stop db1 and see the URL below

    http://localhost:28022/_replSet

You will see db2 is now master. So try to connect db2 and execute some queries.

    % mongo --port 27022
    > db.foo.find()
      => should return records you added to db1
    > db.foo.insert({x:10})
      => You can write on db2 now.

Then restart db1.

    % mongod -f db1.conf

You will see db1 is restored as secondary by checking replication status on
browser.


## Try with your client

Try if the replica set is working with your app. For example, here is mongoid's
configuration.

http://mongoid.org/en/mongoid/docs/installation.html#replica


## See also

http://www.mongodb.org/display/DOCS/Replica+Sets
http://www.mongodb.org/download/attachments/9830402/mongodb+replica+sets+intro.pdf


