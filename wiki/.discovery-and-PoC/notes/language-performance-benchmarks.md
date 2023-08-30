# Performance Benchmarks

Having concerns about the performance of csharp in a microservice environment, I ran benchmarks against
csharp and go for one of the most frequent algos we'll be using in search. Namely updating a single element
in an array of thousands based on a primary key.

These algos are all unavoidably O(n) (unless we do some hashing or sorting, but that in itself would be adding O(n) 
again, without the added memory usage for merge sort). A memory-intense alternative might be to use mergesort and 
bin-search (O(logn) + O(logn)) and I'm not sure it's worth the memory usage.

| Lang   | Method                                                        |           Mean | Allocated |
|--------|---------------------------------------------------------------|---------------:|----------:|
| csharp | List Upsert [*](#list-upsert)                                 | 645.927 millis |  32.84 MB |
| csharp | Bare Array Upsert [*](#bare-array-upsert)                     | 619.761 millis |  32.83 MB |
| csharp | List with known index [*](#list-with-known-index)             |   6.674 millis |  32.84 MB |
| csharp | Bare Array with Known Index [*](#bare-array-with-known-index) |   6.529 millis |  32.82 MB |
| go     | Slice Upsert                                                  |    2.49 millis |     458KB |
| go     | Bare Array Upsert                                             |    2.49 millis |     160KB |
| go     | Slice with Known Index                                        |     8.34 nanos |     458KB |
| go     | Bare Array with Known Index                                   |    8.006 nanos |     160KB |

## Alternatives

Also tried firstOrDefault and dictionary conversion in csharp. Ridiculous results:

| Method                        | Mean          | Allocated   |
|-------------------------------|---------------|-------------|
| First or Default              | 8,984.306 ms  | 137.82 MB   |
| BenchmarkInventoryDict_Update | 19,143.978 ms | 53651.31 MB |

Foreach compiles down to basically the same thing as a for loop, so not much point benching that.

## Algos
### List Upsert

(`data = new List<Inventory>()`)

```
for (int j = 0; j < data.Count; j++)
{
    if (data[j].LibraryKey == inv.LibraryKey)
    {
        data[j] = inv;
        return;
    }
}
data.Add(inv);
```

### List with known index

(`data = new List<Inventory>()`)

```
if (data.Count < i+1)
{
    set.data.Add(inv);
}
else
{
    set.data[i] = inv;
}
```

### Bare Array Upsert

(`data = new Inventory[2000]`)

```
inv = new Inventory(key, availability);

for (int j = 0; j < data.Length; j++)
{
    if (data[j] == null)
    {
        data[j] = inv;
        return;
    }
    
    if (data[j].LibraryKey == inv.LibraryKey)
    {
        data[j] = inv;
        return;
    }
}
```

### Bare Array With Known Index:

(`data = new Inventory[2000]`)

Basically, just reference the index directly - `a[i] = b`