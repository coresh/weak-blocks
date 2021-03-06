Weak Block Simulator for Bitcoin
================================

This is a tool which simulates nodes generating weak blocks.  It does this
using real bitcoin block data preprocessed by [bitcoin-iterate](https://github.com/rustyrussell/bitcoin-iterate), and the corresponding mempool data from [bitcoin-corpus](https://github.com/rustyrussell/bitcoin-corpus).

Interesting ranges are blocks 352720-352819 (which includes an 11-block run of mempool backlog), and the entire useful core of the corpus (352305-353009).

You can see some prepared results in the results directory:

* no-weak-full.csv.xz: blocks and mempool CSV without weak blocks (full corpus)
* no-weak-100-blocks.csv.xz: blocks and mempool CSV without weak blocks (blocks 352720-352819 only)
* 30-second-16-firstbonus-100-blocks.csv.xz: blocks and mempool CSV with 30 second weak blocks and 16x bonus for first weak block (full corpus)
* 30-second-16-firstbonus-full.csv.xz: blocks and mempool CSV with 30 second weak blocks and 16x bonus for first weak block (blocks 352720-352819 only)
* 30-second-16-firstbonus-with-weak-full.csv.xz: blocks and weak blocks and mempool CSV with 30 second weak blocks and 16x bonus for first weak block

What Are Weak Blocks
--------------------

When miners generate blocks, they inevitably generate blocks which
don't quite meet the difficulty threshold required.  We call these
*weak blocks*.

If the network were to propagate weak blocks, there are two main
advantages:

1. Everyone would have some insight into what is likely to be in the
   coming (strong) block.
2. Block transmission could be more efficient by referring to previous
   weak blocks.

The first property is interesting, but this simulator concentrates on
the second, producing an estimate for how large the (strong) blocks
would be in such a scheme.

For comparison, you can get non-weak-block output with --no-weak.

Generating Your Own Weak Simulation
-----------------------------------

```bash
# Generate transaction information for blocks 352720-352819
bitcoin-iterate -q --start 352720 --end 352820 --tx %bN,%tN,%th,%tl,%tF > txs-352720-to-352820.csv
# Simulate 30-second weak blocks between peers, with first weak block 16x easier
./weak-blocks --first-bonus=16 txs-352720-to-352820.csv  ../bitcoin-corpus/au ../bitcoin-corpus/sg ../bitcoin-corpus/sf ../bitcoin-corpus/sf-rn > 30-second-weak-blocks-16-bonus.csv
```

Output Format
-------------
The CSV output is designed for post-analysis, the format is as follows:

```
<FILE> := <BLOCKDESC>*
For each block, in incrementing order:
<BLOCKDESC> := <BLOCK-LINE><MEMPOOL-LINE>+
<BLOCK-LINE> := block,<BLOCKHEIGHT>,<OVERHEAD-BYTES>[,<TXID>]*
<BLOCKHEIGHT> := integer
<OVERHEAD-BYTES> := integer
<TXID> := hex // TXID
For each peer, after each <BLOCK-LINE>:
<MEMPOOL-LINE> := mempool,<PEERNAME>[,<TXID>]*
```

Transactions which were in previous weak blocks are eliminated from
the block TXIDs and the mempool TXIDs (though they add two bytes to
the overhead value).

Limitations of the Simulator
----------------------------

The simulator assumes each peer knows about the weak blocks instantly,
thus there is no problem with referring to a weak block they haven't
seen yet.

By default (without --include-weak) it only outputs the strong blocks,
not the weak ones.  This is because latency of the strong blocks is
what matters (unless you're worried about total bandwidth use).

It assumes a simple 16-bit encoding to calculate the overhead bytes.

Parameters to the Simulator
---------------------------

You can control how often weak blocks occur on average (default: 30
seconds), and also how much bonus the first weak block gets (default:
1, which means no bonus).  The bonus is useful when transactions are
backlogged, as a full block can be mined almost instantly (when there
won't be a weak block to help encode it).

You can also adjust the seed for the random number generator, which
can have dramatic effect: I recommend multiple runs with different
seeds for statistical analysis.
