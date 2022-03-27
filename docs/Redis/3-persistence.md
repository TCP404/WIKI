# Persistence principle

Redis runs on memory, so the data will increase 

## RDB

RDB is a **default memchanism** for Redis persistence. It is enable by default.

When it is enable, Redis will dump the data to the `.rdb` file regularly.

1. RDB is a file used to persist data. It is a binary file and is easy to transmit.
2. But RDB can not ensure the absolute security of data due to the time interval of writing. 

    If the time interval is too short, efficiency declines.

    If the time interval is too long, if there is an accident before the next interval, the data written from last write to the accident will be lost.



```sh title="redis.conf"
# Save the DB to disk.
# save <seconds> <changes>
save 900 1
save 300 10
save 60 10000

# Compress the DB file
rdbcompression yes

# Specify the DB filename
dbfilename dump.rdb
```

## AOF