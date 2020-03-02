# 21e8 Work Puzzle

**Don't assume this is safe. A miner can steal your outputs if they wanted to**


**EDIT**: *So definitely don't use this for anything that you care about. It's not safe*

---


A work puzzle for solving 21e8 hashes with a given 21e8 parent. This puzzle format allows for users to ask for a child
21e8 hash with a given amount of difficulty, that was derived from a parent hash. 

If you think you can break this, please do. 

`SHA256(<PARENT HASH> + OFFSET) = OUTPUT HASH`

This puzzle maintains that the output hash is 
- derived from the parent hash
- has a valid number of trailing zeroes.


We have to convert parent hashes into hex format, so a hash such as `21e800000a0782feaf894b082904c2b3151510b030b7dbddcba0a0c717d1d594` would be converted into 
`32316538303030303061303738326665616638393462303832393034633262333135313531306230333062376462646463626130613063373137643164353934`.

There are two different formats. One for an even number of trailing zeroes, one for an odd number of trailing zeroes.

All of these examples can be tried out in either [Scrypt Debugger](http://scrypt.studio/) or [Learn on Chain](http://learnonchain.com/script).

## Even Difficulty

**Locking**
`<HEX OF PARENT> OP_SWAP OP_CAT OP_SHA256 OP_3 OP_SPLIT OP_DROP 21e800 OP_EQUALVERIFY`
`<HEX OF PARENT> OP_SWAP OP_CAT OP_SHA256 OP_4 OP_SPLIT OP_DROP 21e80000 OP_EQUALVERIFY`

**Unlocking**
`<HEX OF OFFSET>`

This script isn't doing anything that complicated. First, the hexadecimal representations of the `offset` and the `parent` are loaded onto the stack. The `offset` and the `parent` are then swapped and concatenated (to make sure it's `parent` + `offset` instead of `offset` + `parent`).

The resulting string is then hashed with SHA256. The first `N` (difficulty / 2) bytes of the hash are then stripped and the 
backside of the hash is dropped. The two strings are then compared and verified.


#### Examples

*Valid locking script* ([heres](https://search.matterpool.io/tx/94fd8c388f3584dd460e8f7014aa4c44dc06a9b21e9ab3e7b4325d9af2197f61) a live example waiting to be redeemed)

`32316538303030303061303738326665616638393462303832393034633262333135313531306230333062376462646463626130613063373137643164353934 OP_SWAP OP_CAT OP_SHA256 OP_3 OP_SPLIT OP_DROP 21e800 OP_EQUALVERIFY`

*Valid unlocking script*

`313935373537363034`
`3533363236343239`

*Invalid locking script*

`383533333439`


## Odd Difficulty

**Locking**
`
<HEX OF PARENT> OP_SWAP OP_CAT OP_SHA256 OP_3 OP_SPLIT OP_SWAP 21e800 OP_EQUAL OP_SWAP OP_1 OP_SPLIT OP_DROP OP_DUP OP_15 OP_LESSTHANOREQUAL OP_SWAP 00 OP_GREATERTHANOREQUAL OP_VERIFY
`

**Unlocking**
`<HEX OF OFFSET>`


The odd difficulty (i.e. `21e8000`) is more difficult to maintain because the outputted SHA256 hash is in byte representation, meaning you can't look at only 4 bits of that byte. The first portion is virtually the same as the even difficulty, except
the backside of the hash isn't dropped, it's swapped instead. The major addition to the script for odd difficulty is:

`OP_SWAP OP_1 OP_SPLIT OP_DROP OP_DUP OP_15 OP_LESSTHANOREQUAL OP_SWAP 00 OP_GREATERTHANOREQUAL OP_VERIFY`

First, we split the first byte off of the backside of the hash and drop the backside of that split. We now have one byte that
we expect to look like `0X`, where `X` can be anything. This makes a range of numbers from `00` (0) to `0F` (15). The difficulty here comes from the fact that these bytes are 
treated as signed, so we have to look at them with the 2's compliment representation. That means a byte like `A5` will appear
as `-91`, so we also have to verify that the number is greater than or equal to 0. 

We do an `OP_DUP`, verify the number is less than or equal to 15 but greater than or equal to 16. This verifies that the number
is within the range of `00` to `0F`. 

#### Examples

*Valid locking script* ([heres](https://search.matterpool.io/tx/0763c906dffb65adfb754036a8c61b3cb311d9bab21667683ceefb1661460802) a live example waiting to be redeemed)

`
32316538303030303061303738326665616638393462303832393034633262333135313531306230333062376462646463626130613063373137643164353934 OP_SWAP OP_CAT OP_SHA256 OP_3 OP_SPLIT OP_SWAP 21e800 OP_EQUAL OP_SWAP OP_1 OP_SPLIT OP_DROP OP_DUP OP_15 OP_LESSTHANOREQUAL OP_SWAP 00 OP_GREATERTHANOREQUAL OP_VERIFY
`

*Valid unlocking script*

`313935373537363034`


*Invalid unlocking script*

`3533363236343239`

