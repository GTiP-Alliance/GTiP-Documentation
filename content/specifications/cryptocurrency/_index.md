+++
title = "GTiP Cryptocurrency ELI5"
description = "How is a Transaction Message Created? Explain it Like I'm 5"
# Type of content, set "slide" to display it fullscreen with reveal.js
type="default"
# Creator's Display name
creatordisplayname = "GTiP Community"
# Creator's Email
creatoremail = "community@gtip.org"
# LastModifier's Display name
lastmodifierdisplayname = "GTiP Community"
# LastModifier's Email
lastmodifieremail = "community@gtip.org"
alwaysopen = true
weight = 1
+++
GTiP uses a process similar to Cryptonote or Monero for creating transactions. In these systems the sender, recipient and the amount are all anonymous, this article explains in plain english how this is possible.

Let's suppose that Alice wants to send some GTiPs to Bob for his birthday, it works a bit like this...



# The Keys

Every participant in the GTiP network has two pairs of keys (so four keys in total) and we have two participants so there are 8 keys required to make a transaction:



|                    | **Alice** | **Bob** |                               |
| :----------------- | :-------: | ------- | ----------------------------- |
| Secret Key         |    aSk    | bSk     | Used to verify transaction Id |
| Public Key         |    aPk    | bPk     |                               |
| Secret Viewing Key |    aSkV   | bSkV    | Used to verify transaction Id |
| Public Viewing Key |    aPkV   | bPkV    | Used to sign transaction Id   |

The secret keys are generated when Alice and Bob first install the wallet app and must kept super secure.



# UTXO

First Alice has to establish that she has some money to send. Alice's available balance of GTiPs is calculated by looking at all the outputs from previous transactions where she has received money and that haven't been used as inputs for her other transactions yet. These are called **UTXOs** (Unspent Transaction Outputs). Alice's wallet app has a list of all her previous transaction outputs including:

-   The amount received in each transaction
-   Secret keys for each transaction that she generated as the recipient (see below)
-   The public key or **Stealth Address** for each transaction

She can see in her wallet application what all these past transaction outputs add up to - yep currently she has 100GTiP, she only wants to send 10GTiP to Bob so all is good.

Alice's wallet application has to decide which UTXO's she is going to use as the source of funds or **inputs** for the transaction. Let's assume that Alice receives lots of small transactions but is paying Bob a relatively large amount. The wallet app decides to use 3 of Alice's UTXO's as the funds for this transaction, but these three UTXOs add up to 12 GTiP so Alice will need some change. GTiP deals with this by creating an additional output in each transaction that effectively sends money back to the sender. To do this Alice will create a set of transaction keys using the process below and include the resulting spending key as an additional transaction output - so there are usually more than one outputs in each transaction message.

# Hiding the Recipient with a Stealth Address

The transaction does not exist until it is part of the public GTiP record and has been verified by QTiP minter servers. In normal cryptocurrencies the record of each transaction contains public addresses of the sender and receiver and the amount and a signature that represents the coins that are being transmitted. In Cryptonote style transactions each record contains only a transaction Id and the details of the UTXOs being used in the transactions, there are no addresses or keys (or amounts) for either party in the transaction.

How does Bob know which transactions are for him? At the beginning of the transaction process Alice (the sender) and Bob (the recipient) set up a secure communication channel using the [SIKE](http://sike.org/) protocol.

1.  Alice generates a random number _**r**_ - this is the secure one-time-key 
2.  Alice creates a public key that corresponds to _r_ - let's call it _**R**_ this is the shared secret upon which the output spending key is based
3.  Bob send Alice his public key and his public viewing key bPk & bPkv
4.  Alice uses bPkv to encrypt R
5.  She sends the encrypted R to Bob
6.  Bob decrypts R using his private viewing key bSkv

They have now securely shared a secret number that Bob can use to identify this transaction later.



Now Alice will calculate the output spending key by hashing **R** (Monero uses keccak-256) with Bob's viewing key to create _**stealthKeyBob**_. We can now create the first part of the transaction message - the outputs. We put this address as the first output key. Alice also needs some change (as mentioned above) so she uses the same stealth address generation process to create a second output address.

    {output:[stealthKeyBob, stealthKeyForChange]}



Bob's wallet will listen to every transaction that is broadcast across the GTiP network and try to decrypt every output using his secret viewing key bSkv. If he sees a stealthKey that decrypts back to an R in his list he can confirm that he is the recipient of the transaction. Alice will do the same, listening out for the stealthKeyForChange.



The combination of the Bobs public and viewing keys is called a **tracking key** and any user that has this can match the txId back to the shared secret and establish that the transaction was destined for Bob - _**this is how tellers could work.**_

#### Hiding the Sender with Ring Signatures

Earlier on Alice's wallet decided which UTXO's will be used to fund this transaction (3 of them in fact). But if she just sent this txId to Bob then to would be easy to track the progress of a GTiPs through the network and, with a bit of detective work, figure out who everyone is. So what we do is hide the UTXO's txId inside a Ring Signature. This is complicated, wrap a cold towel around your head...

Every node on the GTiP network (including Alice and Bob) has a complete history of all the transactions that have ever taken place. The ring signature hides our real UTXOs inside some dummy ones that are selected at random from some past transactions taken from the history (4 dummies for each UTXO in this case). Note these dummies are not Alice's own transactions, but just random others from any point in the blockchain. The number of fake transactions is called the **mixin level**.

Remember that we're using 3 inputs to fund 2 outputs (spend and change) now we're going to hide these in 4 fakes per input to create a  matrix of stealth addresses for each input ready to pass into our Linkable Ring Signature signing algorithm. Alice uses the stealth key that was generated for this transaction and combines it with each input matrix then passes it through the ring signature generation algorithm to create linkability tags (Monero calls this a key image). We can add this to our message:

    GTiP:{

    txV: 1.0,

    output:[stealthKey(Bob), stealthKey(Change)],

    ringSignatures:{

          input1:{linkTag:

                  UTXO:{   mixin1:[spendKey1, changeKey1],

                           mixin2:[spendKey2, changeKey2],

                           mixin3:[spendKey3, changeKey3],

                           mixin4:[spendKey4, changeKey4], //real

                           mixin5:[spendKey5, changeKey5]

                   }},

          input2:{linkTag:

                  UTXO:{   mixin1:[spendKey1, changeKey1],

                           mixin2:[spendKey2, changeKey2],

                           mixin3:[spendKey3, changeKey3],

                           mixin4:[spendKey4, changeKey4], //real

                           mixin5:[spendKey5, changeKey5]

                   }},

          input3:{linkTag:

                  UTXO:{   mixin1:[spendKey1, changeKey1],

                           mixin2:[spendKey2, changeKey2],

                           mixin3:[spendKey3, changeKey3],

                           mixin4:[spendKey4, changeKey4], //real

                           mixin5:[spendKey5, changeKey5]

                   }}

    }



#### How Does the Linkability Tag Prevent Double Spending?

You'll recall that Alice cfeated a private key for each of the out puts in the transaction 9in this case Bab and Change). The Linkable Ring Signature uses this private key to uniquely generate a Linkability Tag in such a way that a Minter can check a purported key image really is correct. When spending an output, its Linkability Tag is calculated by the sender, and included in the transaction. Minters then check these tags are real, and check whether they were already seen in the blockchain. If they were, it means the output being spent was already spent in a previous transaction, which is then discarded.&lt;/p>

**## How does he know Alice didn't spend this GTiP twice?**

&lt;p>Bob's wallet app now goes to the first UTXO, he uses the public key for this transaction to see if the link tag was created using the secret key held by Alice for this transaction. If Bob know Alice has the secret key for this transaction the secret key cobbined with the transaction key will always produce the same linkability tag. Bob can check if this linkability Tag has every apeared before, if so he can tell that Alice has spent this UTXO previously.




