# Underlying 

## Data Structure

```mermaid
graph TD
    STR[String]
    L[List]
    S[Set]
    Z[Zset]
    H[Hash]

    SDS[Simple Dynamic String]
    DLL[Double Linked List]
    ZL[Ziplist]
    HT[Hash Table]
    SL[Skip List]
    IA[Integet Array]

    STR --> SDS
    L --> DLL
    L --> ZL
    S --> HT
    S --> IA
    H --> ZL
    H --> HT
    Z --> ZL
    Z --> SL
```

ZSet is implemented by Ziplist and Skip List.

If the numbers of ZSet element is less than 128, OR the size of all ZSet elements is less than 64 bytes, the Ziplist is used, otherwise the Skip List is used.
- Use Skip List:
    
    When `ZCARD key` < 128 || all ZSet elements < 64Byte
    