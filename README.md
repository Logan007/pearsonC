### Pearson CRC32c Hashing

As [discussed](https://github.com/skeeto/hash-prospector/pull/13#issuecomment-977186194) over at Skeeto's [Hash Prospector](https://github.com/skeeto/hash-prospector), a CRC32c-MUL-CRC32c scheme seemingly forms a low-bias permutation on 32-bit values. Embedding it into a Pearson scheme as used with [Pearson B. Hashing](https://github.com/Logan007/pearsonB), we get a new hashing function. Following the naming series, let's call it **Pearson C. Hashing**.

Hope is that the CRC32c instruction as found at Intel comptaible CPU's starting with SSE 4.2 support will accelerate hashing a lot.

### Permutation

We use an all-same constant version as it has a comparable bias to an all-different constant version. This way, we might save some cycles loading constants from cahce or memory. 

```
uint32_t cmc (uint32_t x) {
    
    x = _mm_crc32_u32(x, 0x941325ab);
    x *= 0x941325ab;
    x = _mm_crc32_u32(x, 0x941325ab);
    
    return x;
}
```

This scheme passes SmallCrunch (15/15) and masters Crunch (140/144):

```
========= Summary results of SmallCrush =========

 Version:          TestU01 1.2.3
 Generator:        CRC32-MUL-CRC32
 Number of statistics:  15
 Total CPU time:   00:00:04.42

 All tests were passed
```

```
 ========= Summary results of Crush =========

 Version:          TestU01 1.2.3
 Generator:        CRC32-MUL-CRC32
 Number of statistics:  144
 Total CPU time:   00:19:25.30
 The following tests gave p-values outside [0.001, 0.9990]:
 (eps  means a value < 1.0e-300):
 (eps1 means a value < 1.0e-15):

       Test                          p-value
 ----------------------------------------------
 41  MaxOft, t = 5                  1 -  3.2e-7
 42  MaxOft, t = 10                 1 - 1.7e-13
 43  MaxOft, t = 20                 1 - 8.4e-13
 44  MaxOft, t = 30                 1 - 5.1e-13
 ----------------------------------------------
 All other tests were passed
```

### Further Steps

A few tests are still missing such as BigCrunch as well as the comprehensive evaluation of SMHasher – to follow soon. Same for the full C implementation.
