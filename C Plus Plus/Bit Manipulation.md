
`std::bitset` is a useful tool defined in `<bitset>`.
- `std::bitset<#bits> var {init_val}`: create a `bitset`
- `test(pos)`: query if the position is set
- `set(pos)`: set a bit
- `reset(pos)`: reset a bit
- `flip(pos)`: flip a bit
- `size()`: returns number of bits
- `count()`: returns number of bits that are set
- `all()`: returns whether all are set
- `any()`: returns whether any is set
- `none()`: returns whether no bit is set

To test multiple bits:
- `flags & mask`

To set multiple bits:
- `flags |= mask`

To reset multiple bits:
- `flags &= ~mask`

To flip multiple bits:
- `flags ^= mask`

