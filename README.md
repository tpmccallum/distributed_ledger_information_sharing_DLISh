# Distributed Ledger Information Sharing (DLISh)
A system which stores and shares all reuired information on, and between, blockchains.

## Background
Decentralized systems are inherently bad at storing and sharing data. This is due, in part, to the fact that all parties store a redundant copy of all data. DLISh seeks to solve this problem. DLISh is a storage and communication system for decentralized ledger applications i.e. blockchains.

## Storage format
All data is exclusively UTF-8. The system comprises of a master list of key value pairs. The values are groups of letters (words) only. The keys are numbers (sequentially allocated as values are added) only. 

## Premis
Some say that the blockchain can be considered a world computer (which has promised to decentralize everything). A world computer needs to store the world's data. In the case of decentralization there is no single "server" to store the data for reference. In a decentralized architecture all of the data must be stored only on the peers (which make up the peer to peer network). Simply mirroring the data is not sustainable. For example, the bitcoin blockchain is over 150GB < https://blockchain.info/charts/blocks-size >. As time goes on, blockchains will continue to grow. It is important to note that less users are using desktop PCs and servers and more users are using hand-held mobile devices. Some of the latest mbile devices have only between 32 and 128 GB of total storage.

Imagine a completely decentralized e-commerce ecosystem where all buyers, sellers, merchants, delivery services and so forth are registered and interacting in real time using handheld mobile devices. The sorts of information changing hands would include product Id's, product names, product prices, first names, last names, addresses, phone numbers, post codes, transactions and more.

Fortunately languages are all very repetative and traditional storage of data holds an incredible amount of redundancy (per item). For example, 9, 600 roads in North America alone are named "park" i.e. Park Rd. Many of us share first names or last names with others globally and so on. With this in mind, it should never be the case that a single word is stored in more than one place on the blockchain. To achieve this we create and store a master list of words for reference within each peer node of a blockchain. You might be surprized to learn that a text file containing over 1/2 million words only consumes around 5MB. That's 0.005 of a GB. Put simply a master file containing over 1/2 million words takes up 30, 000 times less space than the bitcoin blockchain.

## How DLISh works
Imagine now, that each of the words stored in the master file can be compressed to between 1 and 3 UTF-8 characters. DLISh works by looking up a word, finding the corresponding numerical key, then performing a mathematical operation on that key in order to reduce it to the size of 1 UTF-8 character (or at worst 2 to 3 UTF-8 characters). Once encoded, DLISh allows for information to be encrypted and transmitted inside blockchain transactions. The encryption and decryption can be performed between two peers on the blockchain using proxy re-encryption. The proxy re-encryption component (provided by NuCypher [2]) has been successfully tested.

Now, let's consider the following hypothetical master list.

![master list](https://github.com/tpmccallum/distributed_ledger_information_sharing_DLISh/blob/master/master_list.png)

If we were to store and/or transmit a users full name and address our application would do so by performing the following steps:
- allow the user to enter data
- validate and cleanse the data
- obtain the relavent sequential numerical key or append any new words to the end of the masterlist and obtain the newly created key
- select the appropriate mathematical function to perform on the key (based on optimal compression)
- reveal the encoded string for storage and/or transmission

Here is a hypothetical example
Essentially we are storing the following data
john smith 
sixty one park road 
lane cove 
sydney 
australia

If we break this data up into individual keys we will have the following
2
1
65536
65538
1024
1025
1026
1027
3
5

Already we can simply create the first and last name (as the keys are single characters) i.e. john (2) smith (1) will start off the encoding process where each field is separated by the exclaimation point.

2!1!

Next we have 65536 which needs to be compressed. DLISh has 52 different mathematical equations which are capable of compressing large numbers. Each mathematical equation is signalled through the use of a letter. The letters range from the UTF-8 representation of (lower case) a through z as well as from (capital) A through Z (26 letters in the alphabet and therefore a total of 52 equation possibilities).

The algorithm return two numbers the first number is the encoded value which will eventually be decoded and the second number is the argument which will be passed to the decoding algorithm.

Each letter requires both an encoding algorithm and a decoding algorithm.

Here is the psuedo code for a hypothetical example to encode 65536

```
#encode
function a_encode()
    import math
    numberToEncode = 65536
    tempNumber = 0
    counter = 0
    while numberToEncode.length() > 1:
        counter = counter + 1
        tempNumber = math.findSquareRoot(numberToEncode)
        numberToEncode = tempNumber
    return [tempNumber, counter]
    
```
The above code will return 24 (the square root function was performed four times and resulted in the single digit 2)

```
#decode
function a_decode(number, multiplier)
    for i in multiplier:
        number = number * number
    return number
```
The above code will return 65536 (the original number 2 was transformed each time it was multiplied by itself and this operation was performed 4 times) i.e. 2x2=4, 4x4=16, 16x16=256, 256x256=65536.

### Metadata storage
The metadata storage container is used to define the type of data being stored. DLISh uses convention instead of configuration and metadata which is not lower camel case will not be accepted. Below is only a small example. You can add as many metadata values as you need via the API.

```

--------------------------
|    k    |      v       |
--------------------------
|    1    |  firstName   |
|    2    |  lastName    |
|    3    |  unitNumber  |
|    4    |  streetNumber|
|    5    |  streetName  |
|    6    |  streetType  |
|    7    |  suburb      |
|    8    |  city        |
|    9    |  state       |
|    10   |  country     |
|    11   |  salutation  |

--------------------------

```

### Message format
Message formats are completely customizable. Message formats are ephemeral. They are generated from the values in the metadata storage each time they are used. For example the requesting application knows what values it wants i.e. a person's phone number and so the requesting application is pre-programmed to query the data and return the key for the phone number value. This ephemeral operation is to allow for performance tuning of the bandwidth of the system. Performance tuning will be covered later. The aim of the message format is to allow specific parts of information to be requested and sent. For example, a third-party delivery driver only needs the address of a buyer. The delivery driver does not need to know how much the item cost. The message format is mandatory and is passed as the first part of the message. The separate components of a message are separated by an exclamation mark (!). Spaces and other non text characters are not stored anywhere in the system. Instead (if required) spaces, hyphens, full stops and commas are implemented via message formatting. Numbers are native. Numbers are not stored as values anywhere in the system. For example street numbers, dollars and cents are simply added (as per the message format) the only difference being that they are prefixed with the letter n and terminated with the letter u. Unlike words, with numbers, there are just too many permutations to store. In addition values can be capitalized if prefixed with the letter c. Here is an example of a request.

```

11!2!4!5!6!7!8!9!10

```

Using the tables above, the request signals that we are constructing a delivery address as follows

```

salutation, lastName, streetNumber, streetName, streetType, suburb, city, state, country

```

Using the tables above as a reference, the response would be as follows

```

c7!c2!n14u!c1!c4!c3!c5!c6

```

When decoded would look like this 

```

Mr Smith 14 Park Road Newtown Sydney Australia

```

It is more likely that a value like streetName or suburb will be a single word as apposed to a double or triple word etc. For example singular value Sydney suburbs like "newtown" are more common than others like "lane cove west". Nevertheless, these multi word values (such as hyphenated last names) must be catered for in the encoding and decoding. Values are joined by a space if separated by the letter j and values are joined using a hyphen if separated by the letter h. Let's look at another example of message formatting.

Using a new set of data (different to what we used above) the following message format requests a suburb, city, state and country.

```

7!8!9!10

```

```

--------------------------
|    k    |      v       |
--------------------------
|    1    |  firstName   |
|    2    |  lastName    |
|    3    |  unitNumber  |
|    4    |  streetNumber|
|    5    |  streetName  |
|    6    |  streetType  |
|    7    |  suburb      |
|    8    |  city        |
|    9    |  state       |
|    10   |  country     |
|    11   |  salutation  |

--------------------------

```

The response returns the following results

```

c5jc3jc2!c1!c7jc8jc9!c6

```

which when decoded translates to 

```

Lane Cove West, Sydney, New South Wales Australia

```

```

-------------------------
|    k    |      v      |
-------------------------
|    1    |  sydney     |
|    2    |  west       |
|    3    |  cove       |
|    4    |  road       |
|    5    |  lane       |
|    6    |  australia  |
|    7    |  new        |
|    8    |  south      |
|    9    |  wales      |

-------------------------

```

### Atomic transactions
Firstly, information passing is request driven. A good analogy for this would be how we use the Internet. A user requests content and the content is delivered. To take this a step further, in this system it is mandatory for the receiver/requester to firstly decode the information and then immediately provide acknowledgement that the information is sound. If this does not occur no part of the transaction is recorded permanently. Put simply, the transaction is either completely successful in real-time or it is completely discarded on both ends. Hence the terminology "atomic". The reason for this, is again to do with performance tuning.

### A brief example - a helicopter view
A delivery driver's mobile application requests the delivery address for an item (the prior steps in the supply chain which led to this point are also all request driven and more examples of this can be provided. However for simplicity sake, let's stick to just the delivery of an item for now). The delivery address is encoded and encrypted by the senders mobile application. The encrypted data is sent to the delivery drivers mobile application where it is unencrypted and decoded for immediate use. This successful transaction is recorded in the blockchain. 

### Encryption
Whilst the encoding and decoding is trivial (at the base layer of the application) the encryption is a bit more complicated. This is due to the fact that a user should never be asked to share their secret keys. The information being passed through the public blockchain must be encrypted after encoding (on the senders end) and decrypted before decoding can take place on the receiving end. 

### Implementation

Blockchain technologies (Cosmos Zones) [1] now offer consistently fast finality and as such proxy re-encryption could be performed in a peer to peer environment by sending the necessary (lightweight) data in blockchain transaction (Tx) wrapped messages (Msg). NuCyphers pyUmbral [2] facilitates proxy re-encryption whereby a user can share their encrypted data with genuine recipient user without revealing their private key. As part of the decentralized encryption/decryption process, the pyUmbral system creates a capsule which can be modified by a third untrusted party (who never sees the data) in readiness for the genuine recipient to decrypt the data as intended. The pyUmbral component of this implementation has been tested in a closed environment and was successful. The execution of a transaction between 2 blockchains (using the Cosmos network) has not been tested as yet. The encoding/decoding as described above will need to be coded. The key value containers could be constructed using either Google's LevelDB [3] or Facebook's RocksDB [4].

### Performance tuning
As mentioned above, the performance tuning of DLISh relates to the bandwidth of the system. For example, the communication of information between parties will always reveal patterns. Imagine that our blockchain application is storing our entire supermarket product line. On or around christmas, the word "christmas" is going to be used a lot more, than during other periods. We may find that the item descriptions like "christmas ham", "christmas cracker", "christmas wrapping paper" and so on are far more frequent around christmas time. If our blockchain application is transacting and frequently using the word christmas, we do not want christmas to have an index of 500046 in our word storage container. We would prefer it to have an index in the single digits (if winning the word frequency contest). DLISh only performs atomic transactions. Again, this means that information is requested, encoded, encrypted, sent, received, decrypted, decoded for display and acknowledged as final, or nothing at all. To this end, the key value containers are able to be reorganized based on the frequency of word use (or metadata use). As a further example, say there were 50000 metadata values defined and yet the most recently added metadata value (say, itemId) was now being used in every transaction from browsing to purchasing, to delivery and so forth. The system would become aware of this and move this metadata value of itemId from position 50000 to position 1. This would make the payload of a transaction carry the number 1 as apposed to the number 50000. It is obvious how the concept of performance tuning would benefit the bandwidth of DLISh. Of course the performance tuning would have a negligible impact on the storage and key value querying time of the system.

```

[1] https://cosmos.network/
[2] https://github.com/nucypher/pyUmbral
[3] https://github.com/google/leveldb
[4] https://github.com/facebook/rocksdb

```


