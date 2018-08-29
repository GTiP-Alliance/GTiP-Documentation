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
<p>GTiP uses a process similar to Cryptonote or Monero for creating and validating transactions. Let's suppose that Alice wants to send some GTiPs to Bob for his birthday, it works a bit like this...</p>

### UTXO

<p>First Alice has to establish that she has some money to send. Alice's available balance of GTiPs is calculated by looking at all the outputs from previous transactions where she has received money and that haven't been used as inputs for her other transactions yet. These are called UTXOs (Unspent Transaction Outputs). Alice's wallet app has a list of all her own previous transacton outputs including:</p> 

  * The amount received in each transaction
  * A _secret_ key that she generated as the reciepient (see below)
  * A public _spending_ key that is important for the process below
  * A public _viewing_ key that is important for Minters (see next article)

 <p>She can see in her wallet application what all these past transaction outputs add up to - yep currently she has 100GTiP, she only wants to send 10GTiP to Bob so all is good.</p>

 <p> Alice's wallet application has to decide which UTXO's she is going to use as the source of funds for the transaction. Let's assume that Alice receives lots of small transactions but is paying Bob a relatively large amount. It might take 3 UTXO's to "pay" for the funds.</p>

<p>It is totally possible that no combination of Alice's UTXOs will exactly match the amount that she wants to send to Bob. If this is the case he will have to return some change to her. The process for doing this is described below.</p>

## Set Up a Transaction - Share the Sealth Address

<p> To ceate the new transaction Bob (the recipient) will create the secret key and the two public keys (spending and viewing) that are unique for this transaction. Generating these one-time transaction keys relies on Bob's own personal key pair which is held securely in his wallet and is never shared. Anyone who knows Bob's personal secret key can create intercept and spend UTXOs that are indtended for him! GTiP uses a cutting edge secure communication scheme called SIKE to create a channel between Alice and Bob and to pass the spending key to Alice's wallet in a totally secure way.</p>

<p>Very importantly Alice also sends the amount that she will be sending to Bob via the secure SIKE channel. Once the public key and amount are shared the secure chanel between Alice and Bob is ended and they need never hear from each other again!</p>

<p>The transaction does not exist until it is part of the public GTiP record and has been verified by QTiP minter servers. In normal cryptocurrencies the record of each transaction contains public addresses of the sender and receiver and the amount and a signature that represents the coins that are being transmitted. In Cryptonote style transactions each record contains only the details of the UTXOs being used in the transactions, there are no addresses or keys for either party in the transaction.</p>

### What About Change?
<p>Usually the amount in the UTXO's will be greater than the amount required for the transaction (it can never be smaller). GTiP deals with this be creating an additional out put in each transaction that effectively sends money back to the sender. To do this Alice will create a set of transaction keys and includes the spending key in the tramsaction output - so there are usually more than one outputs in each transaction message</p>

<p>We can now create the first part of the transaction message - the output</p>

```
{output:[spendingKey1, spendingKey2]}
```
_Where spendingKey1 is the one-time address that Bob created and spendingKey2 was created by Alice to receive change_

### Create the Ring Singatures

<p>Alice's wallet will now hide each of these UTXO's inside ring singatures.</p>
<p>Everynode on the GTiP network actually maintains a records of every transaction that is created. And the first stage of creating the ring signature is to select at random some past transactions (4 for each UTXO that she is using in this case) from the public UTXO's, note these are not Alice's own transactions, but just random others used to hide the real one's that are going to be used to fund this transaction. The number of fake transactions is called the mixin level.</p>
<p>We will include the spending keys of all the transactions - real and mixins in our transaction message</p>

## Ring Signature

<p>We now have our matrices (sometimes called vectors) ready to pass into our Linkable Ring Signature signing algorithm. (Which our crypto guys call &Sigma;). Alice uses the secret one-time key that was generated for this transaction and combines it with each input matrix to create linkability tags (Monero calls this a key image). We can add this to our message:</p>

```
GTiP:{
txV: 1.0,
output:[spendingKey1, spendingKey2],
ringSignatures:{
       input1:{linkTag:tagString,
               UTXO:{   mixin1:[spendKey1, changeKey1],
                        mixin2:[spendKey2, changeKey2],
                        mixin3:[spendKey3, changeKey3],
                        mixin4:[spendKey4, changeKey4], //the real ones
                        mixin5:[spendKey5, changeKey5]
                }},
       input2:{linkTag:tagString,
               UTXO:{   mixin1:[spendKey1, changeKey1],
                        mixin2:[spendKey2, changeKey2],
                        mixin3:[spendKey3, changeKey3],
                        mixin4:[spendKey4, changeKey4], //the real ones
                        mixin5:[spendKey5, changeKey5]
                }},
       input3:{linkTag:tagString,
               UTXO:{   mixin1:[spendKey1, changeKey1],
                        mixin2:[spendKey2, changeKey2],
                        mixin3:[spendKey3, changeKey3],
                        mixin4:[spendKey4, changeKey4], //the real ones
                        mixin5:[spendKey5, changeKey5]
                }}
}
```
### How Does the Linkability Tag Prevent Double Spending?


### How Does Bob get his GTiP?

<p>Bob's wallet is constantly looking for his transactions - remember that when Alice agreed to send him money she send him a public key that he keeps in his store of incompleted transactions</p>

<p>Bob's waalet is busily taking a copy of every single message that comes through the network. For every single one it tries to veryify the tx key using the list of incomplete public transcatios keys that it has. Suddenly he finds a match! He knows that this transcation's Tx key was created with the sectet key that Alice created for this transaction, using te ring signature algorithm! </p>

## How does he know Alice didn't spend this GTiP twice?

<p>Bob's wallet app now goes to the first UTXO, he uses the public key for this transaction to see if the link tag was created using the secret key held by Alice for this transaction. If Bob know Alice has the secret key for this transaction the secret key cobbined with the transaction key will always produce the same linkability tag. Bob can check if this linkability Tag has every apeared before, if so he can tell that Alice has spent this UTXO previously.




