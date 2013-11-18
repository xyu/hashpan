Clover HashPAN Challenge
========================

Winner gets a Mac Pro, solid entries get an interview at Clover. Details here: https://www.clover.com/challenge

Say for some reason that a credit card processor was using SHA-1 hashes of credit card numbers ("PANs")—perhaps trying to "securely" identify cards without using card numbers—and accidentally spilled some out. Pretend you're a criminal and want to reverse those PAN hashes to raw PANs.

We provide a dataset of hashed PANs (completely fictionalized!) and your job is to produce the matching PANs. We also provide a set of PANs a criminal happened to have lying around—figure out if this is useful to you.

These hashes are base64(sha1(pan)) where the PAN is represented as ASCII bytes: https://github.com/clover/hashpan/blob/master/data/hashes.txt

This is the set of PANs you had lying around that might provide some clues: https://github.com/clover/hashpan/blob/master/data/pans.txt

Come up an algorithmically efficient implementation that minimizes work assuming a single core. Bonus points for efficiently using any/all cores you have at your disposal.

Join our HashPAN Challenge Google+ for discussion and questions: https://plus.google.com/communities/108841480692450403173

To play: Fork our repo, and issue a pull request when you're ready to submit. Include your description what the execution time is for your program and what kind of hardware you're running it on.
