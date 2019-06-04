### Migrating an RDS instance

This guide covers different ways of migrating RDS instances from one AWS account to another:

 - AWS Database Migration Service (DMS)   
 - Pure `pg_dump` & `psql`     

**The only difference between the processes is at Step 3**

This guide assumes the migration comply with the following :

 - The migration happens from a _source_ postresql RDS instance to a _target_ postresql RDS instance
 - [DMS only] The _source_ RDS is running postgresql 9.4.9 or above
 - Elevated & short-lived sets of postgres credentials are available for both _source_ and _target_




#### Overview 

##### Postgres utilities 

It is possible to do a full database migration using only official CLI tools, provided by Postgres. 
Using `pg_dump` and `psql`, this documents describes the migration process. 

**Using this tooling implies having a _source_ database downtime**. (As you don't want data being written to it while migrating it.)

The steps including those tools will always be the same; on one side we export from source, on the target side we restore.

##### DMS 

AWS Database Migration Service (DMS) is useful to migrate data from a database to another, and *keep them unidirectionally in sync*.

In  other words, DMS can only migrate data, but it ensures that any changes on _source_ will be replicated to _target_
Note: any change to _target_ will not be replicated to _source_.

Even if DMS is used to migrate the data, the postgres utilities are still needed to migrate the meta-data. (FK, sequences, etc.).


#### Step 0 - Pod

In order to run postgresql commands against both of those endpoints, there needs to be a place that has access to both.

This is solved by running a pod into the kubernetes cluster, on live-1, into the team's namespace. 
The migration steps outlined below have been tested from a pod running a `bitnami/postgresql` Docker image.

Regarding the network access: 
 - The _source_ RDS needs to have its `public accessibility` config turned on.
 - The RDS security group needs to be opened to the Cluster. For that, add inbound rules from the NAT gateways' IP address on the 5432 port.   


#### Step 1 - Pre-Data 


First, to export,  we run : 

``` 
pg_dump -U source_username \
     -h source_endpoint \
     -d source_database \
     -O \
     --section=pre-data > pre-data.sql
``` 

Here, `-O` tells RDS to export the table structure without owners.
The command above stores the data in a local file.


Then to restore this into the _target_, we use `psql`:

```
psql -U target_username \
     -h target_endpoint \
     -d target_database \
     -f pre-data.sql

```

If using a local file is problematic, those two commands can be piped together (`|`)



#### Step 2 - Sequences

Sequences are essential for your database to know what the latest increment of the primary keys is. Sequences are held in special tables that will not be migrated from step 1.


First, to export,  we run : 

``` 
pg_dump -U source_username \
     -h source_endpoint \
     -d source_database \
     -t '*_seq' > sequences.sql
``` 

Here, `-t '*_seq'` indicates to `pg_dump` that we only want to export the table ending in `_seq`, which are the sequences.


Then to restore this into the _target_, we use `psql`:

```
psql -U target_username \
     -h target_endpoint \
     -d target_database \
     -f sequences.sql

```

If using a local file is problematic, those two commands can be piped together (`|`)  



#### Step 3 [ DMS ONLY ]  

This step has to be done with the assistance of the Cloud Platform.

The DMS stack is build using a terraform module. 

Please refer to [this](https://github.com/ministryofjustice/cloud-platform-terraform-dms)
 repository to see the DMS module instructions:  
 [https://github.com/ministryofjustice/cloud-platform-terraform-dms](https://github.com/ministryofjustice/cloud-platform-terraform-dms)


IF YOU HAVE FOLLOWED THAT STEP, GO STRAIGHT TO STEP 4

#### Step 3 [ PURE PG_DUMP ]

IF YOU HAVE FOLLOWED THE DMS STEP ABOVE (DMS), SKIP THIS ONE.


Sequences are essential for your database to know what the latest increment of the primary keys is. Sequences are held in special tables that will not be migrated from step 1.


First, to export,  we run : 

``` 
pg_dump -U source_username \
     -h source_endpoint \
     -d source_database \
     -O \
     --section=data > data.sql
``` 

Here, `-O` tells RDS to export the table structure without owners.
The command above stores the data in a local file.


Then to restore this into the _target_, we use `psql`:

```
psql -U target_username \
     -h target_endpoint \
     -d target_database \
     -f data.sql

```

If using a local file is problematic, those two commands can be piped together (`|`)


#### Step 4 - Post-Data

Any constraints, indexes and foreign keys are also a special kind of metadata that would not be migrated during any of the steps above. 
All of data is contained within the `post-data` section.

The process is almost identical as Step 1 :

First, to export,  we run : 

``` 
pg_dump -U source_username \
     -h source_endpoint \
     -d source_database \
     -O \
     --section=post-data > post-data.sql
``` 


Then to restore this into the _target_, we use `psql`:

```
psql -U target_username \
     -h target_endpoint \
     -d target_database \
     -f post-data.sql
```

If using a local file is problematic, those two commands can be piped together (`|`)

  

#### Step 5 - Data Validation (Very Important)

After a migration, **it is your team's responsibility** to ensure the data, its integrity and anything required by your application to operate properly have been preserved.

Even though the process above is handling the data and the meta-data migration, it is essential for you to have a _data validation strategy_ to confirm everything is in order.

The Cloud Platform team can't provide a how-to guide on data validation, as each database migrations are wildly different.


#### Step 6 - Clean up 

After a successful migration, we can clean up by : 

 - Deleting the pod from STEP 1 
 - Disabling the network access from the live-1 cluster to the _source_ RDS
 - Remove the DMS stack (if applicable)
 - Revoke the temporary credentials created for the migration