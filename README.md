# go-suffixarray-16

Extension of the standard [suffixarray](https://golang.org/pkg/index/suffixarray/)

The current real memory footprint of `suffixarray` is `5N` if `len(data) <= MaxInt32` and `9N` otherwise.
`32` and `64` bit ints are used for each byte in the data. And also the byte data is saved.

Since indicies are not packed, some ints use 32 or 64 bytes, but the index can be packed in 16 bytes, for example
(for indicies less than `1 << 15`)

This repo is a snapshot of `suffixarray` with `gen.go` modified to generate a `16 bit` implementation.

## How to use
If you have a `1GB` glob of strings that you want to convert into a suffixarray, you will normally need 
`1GB + sizeof(int) * 1GB = 5GB`. But if you split your blob into chunks of size `math.MaxInt16 = 32767`,
it will generate multiple suffix arrays, but each will hold indicies up to `MaxInt16`.

Thus, in the `1GB` example, we have `32769` suffix arrays, each containing `16 bit` index slices and also
the chunk itself. For a single chunk of size `N <= MaxInt16`, the memory footprint is `N + sizeof(int16) * N + sizeof(Index)`,
which translates roughly to `3N` (the size of the index is minuscule). We have reduced the footprint from `5N` to `3N`.

It's fair to say that the more chunks you have, the more the slices will add their size to the memory footprint. But
having `32769` chunks only adds `32769` slices and `32769` suffixarray.Index, which is a drop in the ocean, compared
to gigabyte sized memory