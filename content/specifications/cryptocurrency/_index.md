+++
title = "GTiP Cryptocurrency"
description = ""
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

# ELI5

<p>GTiP uses a version of the Monero system for creating and validating transactions. Let's suppose that Alice wants to send some GTiPs to Bob for his birthday, it works a bit like this...</p>

### UTXO

<p>First Alice has to establish that she has some money to send. Alice's available balance of GTiPs is calculated by looking at all the outputs from previous transactions where she has recived money that haven't been used as inputs for her other transactions. These are called UTXOs (Unsepnt Transaction Outputs). Alice's wallet app has a list of all her own previous transacton outputs including: 

* The amounts involved
* The public key of the transaction (which was shared with the other party)
* Where she was the sender she will also have a secret key

 Note that the wallet application will also contain a slist of all the GTiP transactions in the world including everyone's UTXOs - they are completely anonymous and no amounts are shared in this list.

 <p>She can see in her wallet application what all her past transactions add up to - yep currently she has 100GTiP, she only wants to send 10GTiP to Bob so all is good.</p>


## Set Up a Transaction

<p> To ceate the new transaction Alice will create a secret and public pair of keys that are unique for this transaction. GTiP uses a cutting edge ecryption scheme called SIKE to create these keys and to share the public key with Bob's wallet in a totally secure way.</p>

<p>Very importantly Alice also sends the amount that she will be sending to Bob via this secure channel. One the public key and amount are shared the secure chanel between Alice and Bob is ended and they need never hear from each other again!</p>

### Create the Ring Singature

<p>The transaction does not exist until it is part of the public record and has been verified by QTiP minter servers. In normal cryptocurrencies the record of each transaction contains public addresses of the sender and receiver and the amount and a signature that represents the coins that are being transmitted. In monero style transactions each record contains only the details of the UTXOs being used in the transactions. If these were just simple keys it would be possible to trace the entire history of a coin and deduce the identities of the wallets through which it passed. To keep everything anonymous the UTXO records are anynoymised using ring sugnatures</p>

<p> First Alice's wallet application has to decide which UTXO's she is gong to use as the source of funds for the transaction. Let's assume that Alice receives lots of small transactions but is paying bob a relatively large amount. It might take 3 UTXO's to "pay" for the funds.</p>

<p>Alice's wallet now wants to hide each of these real UTXO's inside a ring singature. To do this for each UTXO (3 in this case) it randomly selects a set of previous transaction records from the public UTXO's, note these are not Alice's own transactions (but they could be by chance), but just random others used to hide the real one's that ae going to be used to fund this transaction. The number of fake transactions is called the mixin level.</p>

## Ring Signature

<p>For UTXO 1 Alice takes the Tx Keys of the 5 transactions (the four random ones plus the real one) and puts them in an array, which is refered to by the maths guys as a vector. She passes this vector plus the secret transacton key through the Linkable ring signature signing alogrithm. This returns a new UTXO ring key that will used for this transaction in the public record. The ring signature algorithm also returns a special tag or key image which is the unique combination the real UTXO and the transaction's secrect key. This "tag" is eferd to in a monero transaction as "extra" and will be used later used later on by Bob to verify that this UTXO has only been spent once.</p>

<p>Alice's wallet runs the ring signature 3 times - one for each UTXO.</p>

<p>We've now got a message that looks like this</p>

```
{   txId: currently blank,
    utxo1:{ mixins:[txIds that could be the source of funds],
            link: the linkability tag for this UTXO    
    },
    utxo2:{ mixins:[txIds that could be the source of funds],
            link: the linkability tag for this UTXO    
    },
    utxo3:{ mixins:[txIds that could be the source of funds],
            link: the linkability tag for this UTXO    
    }
}
```
The next step is to create the txId for this transaction. Alice's wallet does this by taking the message so far and signing it with the one-off sectret key for this transaction which results in a hash that can be verified by anyone with the matching public key... like Bob...

### What about the amount?


### Broadcast

Now the transactio message is complete it can be broadcast to every wallet and minter on the network


### How Does Bob get his GTiP?

<p>Bob's wallet is constantly looking for his transactions - remember that when Alice agreed to send him money she send him a public key that he keeps in his store of incompleted transactions</p>

<p>Bob's waalet is busily taking a copy of every single message that comes through the network. For every single one it tries to veryify the tx key using the list of incomplete public transcatios keys that it has. Suddenly he finds a match! He knows that this transcation's Tx key was created with the sectet key that Alice created for this transaction, using te ring signature algorithm! </p>

## How does he know Alice didn't spend this GTiP twice?

<p>Bob's wallet app now goes to the first UTXO, he uses the public key for this transaction to see if the link tag was created using the secret key held by Alice for this transaction. If Bob know Alice has the secret key for this transaction the secret key cobbined with the transaction key will always produce the same linkability tag. Bob can check if this linkability Tag has every apeared before, if so he can tell that Alice has spent this UTXO previously.




