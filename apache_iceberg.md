## Issues with Hive Table Format

Changes made to data are inefficient
- No way to safely change the data in multiple partitions at a single time.
- In practice, multiple jobs modifying the same data isn’t a safe operation
- All of the directory listings needed for large tables take a long time. 
- Users have to know the physical layout of the table
- Hive table statistics are generally stale.
- The File system layout has poor performance on cloud object storage

Apache Iceberg is introduced or developed to overcome the issues with hive table format thereby, enabling faster query planning & execution, enable better and safer table evolution,etc.  Iceberg is not a storage or execution engine, it is a table format specification, and it is a set of APIs and libraries for engines to interact with the tables following the specifications. 

Architecture:

There are 3 layers in the architecure of an Iceberg table, with each higher layer tracks the information of the one below it
The Iceberg Catalog
The Metadata Layer, which contains the metadata files, manifest lists and manifest files
The Data Layer, where actual data of the files reside. 

Iceberg Catalog:
A store that houses the current metadata pointer for Iceberg Tables. 
Must support atomic operations for updating the current metadata pointer, this is what allows transactions on Iceberg Table to be atomic and provide correctness guarantees. 
So, when a select query is running on Iceberg Catalog, the first place query engines goes to is Iceberg Catalog, then retrieves the entry of the location of the current metadata file or location of the file that is required, and then opens to read it. 

Metadata file
As the name implies, it stores the metadata about the table. It includes the schema details, partition information and the snapshots, and also the current or latest snapshot, and is available in JSON format. 
When a SELECT query is running, it looks at the field “current-snapshot-id”, and it then uses this value to find the latest data in the “snapshots" array, and then retrieves the value of the snapshot’s "manifest-list” entry, and then opens the manifest list that location points to.

Manifest List
It is a list of manifest files, and it contains the information of each manifest file that makes up the snapshot, such as location of file, what snapshot it was added to, and info about the partitions it belong to and the lower and upper bounds for the partition columns for the data files.
When a SELECT query is running, and has manifest list open for the snapshot, the query engine reads the location of manifest files and the opens the file. It can do some optimizations at this stage like row counts or filtering the data based on partition info.

Manifest File
It tracks the data and  as well as additional details and stores the statistics of the file, which is the main difference between the open table format and Hive table format. 
Each manifest file keeps track of subset of data files for parallelism and reuse efficiency at scale. They contain lot of useful info to improve efficiency and performance like partition membership, lower and upper bounds of columns, and these stats are written for each manifest subset of data files, during the write operation, and hence they are accurate and up-to-date. 
