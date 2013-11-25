# Proposed Process

For the following solution I'm assuming that I have access to an appropriate hash cracking machine. For the purposes of time calculation I'm assuming that I have a Linux box with a single AMD Radeon HD 6990 graphics card and oclHashcat-plus installed. According to http://hashcat.net SHA1 cracking times for an "Ubuntu 12.04.1, 64 bit Catalyst 13.8beta1 1x AMD hd6990 stock core clock" machine is **3139M c/s**.

## Scope out the Issue

To reduce cracking time we want to know what we are going to be dealing with, types of cards, issuing banks, etc. Some of this information could have been gleamed from where we got the hash. (Was it a ecommerce shop that only took a couple particular brands of cards? Where is the main customer base located? Etc.) A second method is to take a look at a random sample of representative cards which is what I'm assuming the unhashed card file is.

How well the provided sample represents the hashed list is currently unknown, as the source of both files are unknown. However it's the only thing we have so it's better then doing an exhaustive key search on all card numbers so let's analyze it a bit.

Putting the numbers into R and running a `hist()` shows the following distribution:

![](R-PAN.png?raw=true)

So the numbers are not randomly distributed which means we can use various methods to narrow down the scope of our attack.

To better find patterns for steps below we will sort our sample PAN file.

```
$ sort pans.txt > pans-sorted.txt
```

## Quick Hit

If we want a quick hit we can look at most productive issuer codes first so let's look at the best 10 issuer codes.

```
$ cut -c 1-3 pans-sorted.txt | uniq -c | sort | tail -n 10
  27 502
  27 552
  27 573
  30 443
  32 536
  43 437
  43 469
  65 403
  84 503
  96 473
```

We already knew from above that the distribution is not random and this shows that there is quite a bit of clumping when looking at 3 digit prefixes. We can now lengthen the issuer code and see if anything changes.

```
$ cut -c 1-4 pans-sorted.txt | uniq -c | sort | tail -n 10
...
$ cut -c 1-5 pans-sorted.txt | uniq -c | sort | tail -n 10
...
$ cut -c 1-6 pans-sorted.txt | uniq -c | sort | tail -n 10
...
$ cut -c 1-7 pans-sorted.txt | uniq -c | sort | tail -n 10
...
```

Running the above we can see that up to 6 digits the top 10 counts remain relatively steady. Once we get to 7 digits the counts drop dramatically so it must mean our most productive banks have issuers codes 6 digits long. Totaling the counts of our top 10 list of 6 digit prefixes we see that each prefix accounts for at-least ~2.5% of the sample.

```
$ cut -c 1-6 pans-sorted.txt | uniq -c | sort | tail -n 10
  24 573811
  25 424467
  27 552698
  30 443010
  32 536232
  34 437437
  39 469011
  65 403646
  74 473055
  84 503957
```

Building a dict list with this sample means we only have to crack `10 * 10^10` hashes which gives us a cracking time of **~19** hours and should provide us with ~35.6% of cards.

```
$ cut -c 1-6 pans-sorted.txt | uniq -c | sort | tail -n 10 | sort -r | cut -b 6- > pre-6.dict
$ oclHashcat-plus ... pre-6.dict ?d?d?d?d?d?d?d?d?d?d
```

## More Extensive Method

If we wanted to instead do a more extensive search we should look at all the issuer numbers present in the sample file. To do that we will first do a count of all card prefixes to see what issuers we are dealing with. (These fake numbers means we have fake banks/issuers but we can still use statistical analysis to find common fake banks.)

```
$ cut -c 1-2 pans-sorted.txt | uniq -c
  88 40
  34 41
  49 42
  46 43
  35 44
  42 45
  81 46
 106 47
   8 49
 151 50
  10 51
  13 52
  41 53
  30 54
  35 55
  25 56
  66 57
  60 58
  63 59
```

If our PAN sample is representative of hashed cards then we are only dealing with Visa and MasterCards, perhaps this site does not accept Amex/Discover cards which means we need to only crack `'/[45][0-9]{15}/'` in the worst case which narrows our list down a bit.

But to speed up cracking we can do something even smarter and use what hashcat calls a "hybrid_attack" and sort our 'dict' list by the most common first 2 digits so that they are brute forced first.

```
$ cut -c 1-2 pans-sorted.txt | uniq -c | sort -r | cut -b 6- > pre-2.dict
```

Not only does this attempt to crack prefixes '50' & '47' first which seem the most productive ranges it also allows us to skip prefixes '48' which our simple analysis shows that there are no matching card numbers. This means we only have 19 prefixes to crack, this gives us `19 * 10^14` hashes which gives us a cracking time of **~168** hours.

```
$ oclHashcat-plus ... pre-2.dict ?d?d?d?d?d?d?d?d?d?d?d?d?d?d
```

## Exhaustive Search

Finally, if we both of the above were tried and we still see a significant number of uncracked cards then we want to run a full on exhaustive search of card numbers. To do so we will need additional time as follows:

| Prefix                              | Permutations  |
| ----------------------------------- | ------------: |
| Prefix 48                           |       `10^14` |
| Amex cards prefix 34                |       `10^13` |
| Amex cards prefix 37                |       `10^13` |
| Discover cards prefix 6011          |       `10^12` |
| Discover cards prefix 622126-622925 | `800 * 10^10` |
| Discover cards prefix 644-649       |   `6 * 10^13` |
| Discover cards prefix 65            |       `10^14` |

In total this comes out to **~25.6** additional hours.

## More Optimizations

We can cut these times down by an order of magnitude if we extend hashcat to do mod10 checking on iterated card numbers before hashing it however given that hashcat is not open source this is difficult to accomplish at best. In addition as shown above we can crack all Visa and MasterCard numbers in about a week on one PC with a single video card so optimization may not be worth it.