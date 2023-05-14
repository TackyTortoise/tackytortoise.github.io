---
layout: post
title:  "Using bit flags in c++"
date:   2020-11-26 22:57:00 +0200
---
## An Introduction to Bit Flags
Bit flags are a programming technique to represent multiple boolean values or multiple states into a single integer. The amount of different states you can store inside of one integer depends on it’s memory size e.g. a 32 bit integer could hold 32 different states at once, one bit to represent each state.

If you are not familiar with binary notation or bitwise operators I recommend you <a href="https://tackytortoise.github.io/2020/11/26/bitwise-operators-in-cpp.html" target="_blank">read this article first</a>


## Setting Up Bit Flags Values/States

To set up different possible values for bit flags, you want to keep in mind that every state is represented by a single bit. For this to work each bit can only be used once for a flag value, otherwise 2 values could clash with each other. To comply with this you will only want to use powers of 2 to represent each state.
An easy way to do this is with the 1 << N operator, where N would be the current state count, this way you use the next bit every time, such that the state values would be 1, 2, 4, 8, 16… and so on. The most common way to represent the different possible states is in a simple enumeration.

Here’s an example of such an enum with 8 different possible states.

{% highlight cpp %}
enum class EFlagValue
{
    Flag1 = 1 << 0, // 1
    Flag2 = 1 << 1, // 2
    Flag3 = 1 << 2, // 4
    Flag4 = 1 << 3, // 8
    Flag5 = 1 << 4, // 16
    Flag6 = 1 << 5, // 32
    Flag7 = 1 << 6, // 64
    Flag8 = 1 << 7  // 128
};
{% endhighlight %}


## Manipulating bit flags

Now that we have a way of representing different states, bit by bit, it’s to put them to good use. For starters, you want to create a simple integer value to hold all the flags of course, the easiest way to do this is set up a struct that holds the value and has different functions for manipulating and checking the states of that value. For starters, let’s just make an empty struct with an 8 bit integer, so we could represent 8 different states with this singular number. Of course any size integer can be used to be able to represent more different states.

```cpp
struct BitFlag
{
    uint8_t m_FlagValue = 0;
};
```

### Setting a Flag

To set a flag to a “true” value we want to make the bit become 1, no matter what the previous value was. This is very easy to achieve with the \| operator, since you’re sure the flag is set on the input value (which is the actual flag) a bitwise or operation would always result in this bit set to 1, since it is 1 in the input.

A quick visualization of this would work on the bits of our current situation if we wanted to set Flag3:

```
  00000000 -> Our current flag value is 0
| 00000100 -> Do or operation with Flag3, which has a value of 4
= 00000100 -> The bit for Flag3 gets set to 1, flag value is now 4
```

To create a function that would do this for us inside our struct could look a bit like this:
```cpp
void SetFlag(EFlagValue flag)
{
    m_FlagValue |= (int)flag;
}
```


### Unsetting a Flag

This is obviously intended to do the opposite of our previous operation, we want to unset a state, so we always want to change the specific bit in our value to 0. Performing this is a bit more complex than setting the flag since the result has to be 0 no matter if the original bit in the value was 1 or 0. The way to do this is to first invert all the bits of the flag we want to unset, and then do an and operation with that result, so `value &= ~input`.

Here’s how that would work our bits if we would for example want to unset Flag3 from a value that has Flag2, Flag3 and Flag5 set:
```
   00000100 -> We get Flag3 as input to unset
~= 11111011 -> We flip all the bits of Flag3

   00010110 -> Our current value had Flag2, 3 and 5 set already
&  11111011 -> AND operation with the result of previous inversion
=  00010010 -> Our new value only has Flag2 and 5 set
```

We can expand our struct to include a method to unset flags as well:
```cpp
void UnsetFlag(EFlagValue flag)
{
    m_FlagValue &= ~(int)flag;
}
```


### Flipping a Flag

To flip a flag, we want our resulting bit to be 0 if it was previously one and vice versa. This result is quite easy to achieve by using a XOR operation with our input flag.

If we look at the bits, this is what it would look like to flip Flag3 on our value:

```
  00010010 -> Our current value has Flag2 and Flag5 set
^ 00000100 -> Perform XOR with Flag3
= 00010110 -> New value has Flag3 set
```

If you perform this operation again on the result, you will see that the bit for Flag3 goes back to 0, since it is 1 in both our current value and our input flag value:

```
  00010110 -> Our current value has Flag2, Flag3 and Flag5 set
^ 00000100 -> Perform XOR with Flag3
= 00010010 -> New value does not have Flag3 set
```

If we were to include this functionality in the struct that we have so far, we could write it in a function like this:

```cpp
void FlipFlag(EFlagValue flag)
{
    m_FlagValue ^= (int)flag;
}
```


### Checking if a Flag Is Set

All of this functionality is fine and dandy, but it is of no use if we cannot check if our value has a current state active or not. Luckily this is easily achieved with a AND operator again, if we do an & with our current value and the flag we want to check, the result will either be the value of the flag if it was set, or it will be 0 if the flag was not set in the original value.

Here’s how it would look like in the bits when checking for Flag3:

```
  00010110 -> Starting value has Flag2, Flag3 and Flag5 set
& 00000100 -> Perform & with Flag3
= 00000100 -> Result is equal to Flag3

-----------------------------------------------------------

  00010010 -> Starting value has Flag2 and Flag5 set
& 00000100 -> Perform & with Flag3
= 00000000 -> Result is equal to 0
```

Here’s how we could represent this function in our struct:

```cpp
bool HasFlag(EFlagValue flag)
{
   return (m_FlagValue & (int)flag) == (int)flag;
}
```


### Checking if Any of Multiple Flags Is Set

The above method will only work with checking one flag at a time, but it is also possible to check with multiple flags at the same time, these input flags can be combined with the \| operator to still just take one input in the function. It works very similar to checking if a specific flag is set, we also use the & operation, the only difference is that the result could be either of the input flags or all of them together. So instead of checking if the result equals the flag, we just check if the result is not equal to 0, meaning that any of the input flags was set on the value.

In code this could look like this:

```cpp
bool HasAnyFlag(EFlagValue multiFlag)
{
    return (m_FlagValue & (int)multiFlag) != 0;
}
```

## Full Example Code

If we combine all this together, we get a nice struct where we can set, unset, flip and check every individual flag. You can find full example code of this article <a href="https://github.com/TackyTortoise/Medium/blob/main/BitWise.cpp" target="_blank"> here on my github</a>.
