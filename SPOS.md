 

 

SAFE
====

SPOS Consensus Algorithm(First draft)
========================

Block production process
------------------------

1)  When the program starts, a timer is started, and the timer interval
    is set to 50 milliseconds;

2)  At the time of startup, select 9 bookkeepers from the list of
    masternodes;

3)  In the timer callback, 9 bookkeepers generate blocks in turn, 1, 2,
    3, 4, 5, \....., 9;

4)  After the round of block generation, re-select 9 bookkeepers;

Initialization
--------------

1)  At the start of the program, and the master node data has been
    synchronized, do the following steps;

2)  According to the current block height, the search is opened backward
    until the block height is 9 and the result is 0;

3)  According to Rule 1.2, select 9 bookkeepers from the second step;

Choose of 9 bookkeepers
-----------------------

1)  After receiving a new block and passing the test, use the latest
    height of the blockchain to find the remainder of 9 (the
    bookkeeper), and obtain the remaining result. If it is not equal to
    0, return directly, otherwise continue the following steps;

2)  Determine whether the current network has switched on forcibly
    selecting 9 bookkeepers from the official masternode list, and if
    so, select from the official masternode list; otherwise, selecting
    from the user masternode list;

3)  From the masternode list (may be: the official masternode list, the
    user masternode list), select the masternode whose status
    is Enabled and online duration greater than 3 days, and sort the
    selected masternodes according to the score from large to small.
    Rule of calculating the score: generate a HASH value from the
    masternode collateral address and the current block\'s latest block
    time, calculate an integer for HASH, the algorithm is as follows:

> uint32\_t static inline ReadLE32(const unsigned char\* ptr)
>
> {
>
> return le32toh(\*((uint32\_t\*)ptr));
>
> }
>
> arith\_uint256 UintToArith256(const uint256 &a)
>
> {
>
> arith\_uint256 b;
>
> for(int x=0; x\<b.WIDTH; ++x)
>
> b.pn\[x\] = ReadLE32(a.begin() + x\*4);
>
> return b;
>
> }

4)  Randomly select 9 nodes from the list of pre-selected masternodes as
    the bookkeeper. The random algorithm is as follows:

> auto now\_hi = uint64\_t(head\_block\_time().sec\_since\_epoch()) \<\<
> 32;

for( uint32\_t i = 0; i \< \_wso.current\_shuffled\_witnesses.size();
++i )

{

/// High performance random generator

/// http://xorshift.di.unimi.it/

uint64\_t k = now\_hi + uint64\_t(i)\*268589657736338717ULL;

k \^= (k \>\> 12);

k \^= (k \<\< 25);

k \^= (k \>\> 27);

k \*= 268589657736338717ULL;

uint32\_t jmax = \_wso.current\_shuffled\_witnesses.size() - i;

uint32\_t j = i + k%jmax;

std::swap( \_wso.current\_shuffled\_witnesses\[i\],

\_wso.current\_shuffled\_witnesses\[j\] );

}

now\_hi is calculated from the value (current block chain latest
time) \<\< 32

Consensus algorithm switch
--------------------------

Add a control selects nine bookkeeper switches to prevent the occurrence
of evil behaviors by selecting nine bookkeepers from the user\'s master
list, leading to the inability to generate blocks. The rules are as
follows:

1)  Add Spork to send a notification, open to force the selection of 9
    bookkeepers from the list of official nodes; the rules are as
    follows:

2)  Built in some official masternode list in the code;

3)  Select 9 bookkeepers from the list of official masternodes;

Generating block
----------------

1)  Check if the masternode is valid, and if the block data has been
    synchronized, if not, return directly, otherwise continue with the
    following steps;

2)  Obtain the collateral address of the masternode, determine whether
    it is in the 9 bookkeeper list, if it does not exist, return
    directly, otherwise continue the following steps;

3)  Get the local time, determine whether the acquisition time is less
    than the time of the latest block. If it is less, return directly,
    otherwise continue the following steps;

4)  Use the local time plus the interval of the production block, use it
    to subtract the time recorded by the 9 bookkeepers, and divide the
    result by the time interval of the block, and then get the remainder
    of the result to 9. This gives an index;

5)  Using the index calculated in step 4, find the planned block time
    and the masternode collateral address in the 9 bookkeeper list;

6)  Determine whether the collateral address of the masternode is the
    same as the collateral address of this particular masternode. If it
    is not the same, return directly, otherwise continue with the
    following steps;

7)  Use the next block time minus the time obtained in step 3, and take
    the absolute value of the result to determine whether it is greater
    than 500 milliseconds. If it is greater, return directly, otherwise
    the block is packaged and broadcast;

coinbase adds extra parameters
------------------------------

In the coinbase transaction output field, add the following two fields:

1)  the collateral address of the masternode;

2)  using the masternode private key to generate a signature on the
    masternode\'s collateral address

Modify block parameters
-----------------------

nBits and nNonce in the block are modified to 0 ;

Enable SPOS
-----------

After reaching a certain block height (tentative: 1200000),
the SPOS consensus algorithm is officially launched ;

Verification block
------------------

1)  While receiving a block, determine whether the block height is
    greater than or equal to 1200000. If it is greater than, add the
    following rule of determination; otherwise, the following rules are
    not included;

2)  Determine block nBits, nNonce is 0; if not, reject this block;
    otherwise, continue with the following steps;

3)  Get the local time, use it to subtract the time in the block to
    determine whether it is greater than negative 10 seconds. If it is
    greater, reject the block; otherwise continue the following steps;

4)  Verify the signature value in the block with the mortgage address of
    the master node in the block. If the verification does not pass,
    reject this block, otherwise continue with the following steps;

5)  If the node is currently in the sync block phase, the verification
    is completed directly, otherwise the following steps are continued;

6)  Find the collateral address of the masternode in the
    local 9 bookkeeper list by index, and determine whether it is equal
    to the collateral address of the masternode in the block. If not,
    reject the block; otherwise continue the following steps;

7)  Verify the signature value in the block by using the collateral
    address of the masternode in the block. If it does not pass the
    verification, the block is rejected, otherwise accept the block;

How to deal with the block production timeout
---------------------------------------------

There is no special processing logic for the block production timeout,
because the block timer is always running, and the block is generated
when the block time is reached. If the abnormal block producing node
does not produce the block within the specified time, the next block
producer will automatically produce the next block.

If the abnormal block producing node can be later used normally, and
enter the block production process, we know the block producer is the
node behind it according to block time, and it is only the abnormal
block producing node's turn in the next round, so the abnormal node is
skipped.

How to handle multiple blocks at the same time
----------------------------------------------

1)  While receiving a new block, determine whether the previous HASH in
    the current block is the latest local block HASH ;

2)  If yes, join the blockchain directly in the region; otherwise,
    directly reject it;

    If multiple blocks are generated at the same time, the following
    occurs:

<!-- -->

1)  If at some point two blocks are generated, one block is recognized
    and accepted by other nodes, other nodes
    being two-thirds of 9 nodes;

2)  Another block is recognized and accepted by one third of the other
    nodes, then at the time two networks have formed;

3)  The two networks, with increasing blocks produced over time,
    eventually connect to a junction and see which chain is the longest,
    it will automatically switch to the longest chain;

4)  Here is only an example of a situation. For more complicated
    situations, please see the \"Problem of the Competitive Chain\"
    document;

block time
----------

1 ) The block time is changed from the original 2.5 minutes
to 10 seconds;

2 ) Block reward, according to the block time block reward is adjusted,
the new consensus algorithm reward each block less than the
existing POW consensus algorithm, but the number of blocks produced per
day is increased, making total reward per day not reduced;

What should I do if the bookkeeper drops offline?
-------------------------------------------------

1 ) If some of the 9 bookkeepers drops offline, start the next round of
re-selection of 9 bookkeepers, then start again from the beginning to
start block production. This way, if all nodes are offline, there is no
way to produce a block, but this extreme case occurs very little in
probability;

2 ) If all 9 bookkeepers are offline, the project official can establish
more masternodes to keep the network running normally;
