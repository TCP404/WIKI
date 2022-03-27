# Persistence principle

Redis runs on memory, but the memory capcity is not unlimited. So there will be insufficient memory.

To resolve this problem, Redis has 3 ==Data Elimination Strategy==.

1. **RDB**: Redis Database File (Default).
2. **AOF**: Append Only File.
3. **Algorithm**: 
    - *volatile-lru*: Delete a ==L==east ==R==ecently ==U==sed key from the keys that **sets expire** when memory is deficient.
    - *volatile-lfu*: Delete a ==L==east ==F==requently ==U==sed key from the keys that **sets expire** when memory is deficient.
    - *volatile-random*: Delete a key ==Randomly== from the keys that **sets expire** when memory is deficient. 
    - *volatile-ttl*: Delete the key with the **nearest expire time** (Minor TTL).
    - *allkeys-lru*: Delete a ==L==east ==R==ecently ==U==sed key from all keys.
    - *allkeys-lfu*: Delete a ==L==east ==F==requently ==U==sed key from all keys.
    - *allkeys-random*: Delete a key ==Randomly== from all keys.
    - *noeviction*: Just returns an error on write operations when memory is deficient.
## RDB

RDB is a **default memchanism** for Redis persistence. It is enable by default.

This is a snapshot method. What the `.rdb` file saves is the latest data.

``` mermaid
graph LR
    A[Memory];
    B[Disk];
    A --> |rdbSave| B;
    B --> |rdbLoad| A;
```


When it is enable, Redis will dump the data to the `.rdb` file regularly.

1. RDB is a file used to persist data. It is a binary file and is easy to transmit.
2. But RDB can not ensure the absolute security of data due to the time interval of writing. 

    If the time interval is too short, efficiency declines.

    If the time interval is too long, if there is an accident before the next interval, the data written from last write to the accident will be lost.



```sh title="redis.conf"
# Specify save opportunity.
# When `redis-cli SHUTDOWN` command is executed, the following setting also save the DB to disk.
# Configure format: save <seconds> <changes>
save 900 1      # Save if 1 data be written in 15 minutes.
save 300 10     # Save if 10 data be written in 5 minutes.
save 60 10000   # Save if 10000 data be written in 1 minutes.
# As long as one item is satisfied, Redis will save the DB to disk.


# Compress the DB file
rdbcompression yes

# Specify the DB filename
dbfilename redis.rdb
```

## AOF
AOF means Append Only File. What the `.aof` file saves is all the manipulation.

For data security, AOF mode is safer than RDB mode. 

BTW, Redis official recommends to enable both AOF mode and RDB mode.

AOF mode is a higher priority than RDB mode. After Redis goes down and restart, AOF file will be perferred to recover data.

```sh title="redis.conf"
# Enable the Append Only Mode
appendonly yes

# Specify the Append Only filename.
appendfilename redis.aof

# Specify synchronization opportunity (choose one)
#  - everysec: Each second records the operation to the AOF file. This is DEFAULT configure.
#  - always:   Each write operation records the operation to the AOF file, but efficiency is low.
#  - no:       Set different synchronization time intervals according to the runtime environment.
appendfsync everysec 
# appendfsync no
# appendfsync always 
```