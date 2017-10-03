# Microsoft Azure
### Microsoft Azure Storage
- Blob Storage
- Table Storage
- Queue Storage
- File Storage

1. **Blob Storage**: Binary Large Object Storage for storing unstructured data.
2. **Table Storage**: Tables to store data, table have no schema, can be used table storage to develop based on fast changing requirement.
3. **Queue Storage**: Distributed app need to communicate async. Queueing is one mechanism for such communication.
4. **File Storage**: It is like a file share. Can be mapped using a network drive.
 
### Account Types 
1. **Locally Redundant (Standard - LRS)**: 3 data copies in single facility in single location.
2. **Zone Redundant (Standard - ZRS)**: 3 data copies in two or three facilities in a single location (BLOB only).
3. **Geo Redundant (Standard - GRS)**: LRS + 3 data copies stored in secondary location.
4. **Read - Access Geo Redundant (Standard - RAGRS)**: Same as GRS but at a failover state the data in secondary location is read only.
 
 