### Tupple VS List

| tupple | list |
|--------|------|
| immutable | mutable |
| fixed size | dynamic |
| similar to `struct` in C | similar to `array`  in C|
| can't add elements to a tuple. Tuples have no append or extend method| |
| can find elements in a tuple, since this doesn’t change the tuple | |
| can find elements in a tuple, since this doesn’t change the tuple | |
| You can also use the in operator to check if an element exists in the tuple | |
| faster | slower |
| makes your code safer | |

### Return VS Yeild

| Yeild | Return |
| ------|------- |
| Suspends function’s execution and sends a value back to caller, retains enough state to enable function to resume where it is left off |  sends a specified value back to its caller whereas Yield can produce a sequence of values|
| allows its code to produce a series of values over time, rather them computing them at once and sending them back like a list | |
|  when we want to iterate over a sequence, but don’t want to store the entire sequence in memory | |
| used in Python generators | |

### Genrators

Generator function is defined like a normal function, but whenever it needs
to generate a value, it does so with the yield keyword rather than return.
