Ethash is intended to satisfy the following goals:

1. **IO saturation**: the algorithm should consume nearly the entire available memory access bandwidth (this is a strategy toward achieving ASIC resistance)
2. **Light client verifiability**: a light client should be able to verify a round of mining in under 0.01 seconds on a desktop in C, and under 1 second in Python or Javascript, with at most 32 MB of memory
3. **Light client slowdown**: the process of running the algorithm with a light client should be much slower than the process with a full client, to the point that the light client algorithm is not an economically viable route toward making an ASIC implementation.
4. **Light client fast startup**: a light client should be able to become fully operational and able to verify blocks within 40 seconds in Python or Javascript.

### BBS random number generation

Our modified version of Blum Blum Shub was chosen to be used as a random number generator because of several properties:

* It is simple to implement and simple to compute
* There is a shortcut method for computing any step of the BBS immediately (`quick_bbs(seed, i, P) = seed ** (3 ** i % (P-1)) % P`)
* It has a mathematically provable period length

When choosing a safe prime *p* for our random number generator, we wish to find ones where the [multiplicative order](http://en.wikipedia.org/wiki/Multiplicative_order) of 3 in ℤ/(*p* - 1) is *high*, since this determines the cycle length of the corresponding random number generator.

The multiplicative order of 3 in ℤ/(4294967086) is 1073741771.

The multiplicative order of 3 in ℤ/(4294963786) is 2147481893.

Note that cryptographic security is NOT required of the RNG here; we only need it to provide values which are roughly even across the entire output space `[0 ... 2**32 - 1]` and can be relied on to pass the [Diehard Tests](http://en.wikipedia.org/wiki/Diehard_tests).

This particular pseudo-random number generator may exhibit bias when taking its output modulo a value which is not a prime, which is why we have forced the size of the DAG and the cache to be products of a prime and sizes of memory chunks.

### FNV

FNV was used to provide a data aggregation function which is (i) non-associative, and (ii) easy to compute. A commutative and associative alternative to FNV would be XOR.

### Parameters

* A 32 MB cache was chosen because a smaller cache would allow for an ASIC to be produced far too easily using the light-evaluation method. The 32 MB cache still requires a very high bandwidth of cache reading, whereas a smaller cache could be much more easily optimized. A larger cache would lead to the algorithm being too hard to evaluate with a light client.
* 64 parents per DAG item was chosen in order to ensure that time-memory tradeoffs can only be made at a worse-than-1:1 ratio.
* The 1 GB DAG size was chosen in order to require a level of memory larger than the size at which most specialized memories and caches are built, but still small enough for ordinary computers to be able to mine with it.
* The growth parameters are required because fixed or bounded requirements make it easier to build an ASIC. The ~0.34x per year growth level was chosen to roughly be balanced with Moore's law increases at least initially (exponential growth has a risk of overshooting Moore's law, leading to a situation where mining requires very large amounts of memory and ordinary GPUs are no longer usable for mining).
* 32 accesses was chosen because a larger number of accesses would lead to light verification taking too long, and a smaller number would mean that the bulk of the time consumption is the SHA3 at the end, not the memory reads, making the algorithm not so strongly IO-bound.
* The epoch length cannot be infinite (ie. constant dataset) because then the algorithm could be optimized via ROM, and very long epoch lengths make it easier to create memory which is designed to be updated very infrequently and only read often. Excessively short epochs would increase barriers to entry as weak machines would need to spend much of their time on a fixed cost of updating the dataset. The epoch length can probably be reduced or increased substantially if design considerations require it.
* A 4 KB mix was chosen in order to ensure that a full page is fetched from memory, allowing the algorithm to access memory with maximum efficiency. Smaller reads will trigger transaction lookup buffer misses, a problem which specialized hardware will be able to get rid of.