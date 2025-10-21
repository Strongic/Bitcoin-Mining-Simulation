# Overview

Cryptocurrency is a form of digital currency that relies on cryptography to control its creation and ensure secure fund transfers. Bitcoin, introduced in 2009, was the first widely adopted cryptocurrency. Its purpose is to enable direct, peer-to-peer money transfers. Every bitcoin transaction is recorded and verified on a decentralized, distributed, and publicly accessible digital ledger called the blockchain. 

The process of authenticating a bitcoin transaction works in the following way:  

* **Initiation**: One party agrees to send a digital asset to another party.
* **Broadcast**: The digital transaction is broadcast to the Bitcoin peer-to-peer network for verification.
* **Block Formation**: Recently verified transactions are grouped into a structure called a block.
* **Hash Creation**: A hash of the block is created using the data inside it, the hash of the previous block, and a random number called a *nonce*.
**Mining**: Special computer nodes called *miners* generate the *nonce*. They compete to solve a complex mathematical problem, and the first to solve it gets the right to add the new block to the blockchain and earn a reward.

The *nonce* in a block is a 32-bit number. Its value is adjusted so that the hash of the block contains a specific number of leading zeros. These *leading zeros* are used to control the difficulty of the problem set for miners: more *leading zeros* mean fewer possible solutions, making the problem harder to solve.

The security of the blockchain comes from the fact that each block verifies all previous blocks. Therefore, to successfully change a single block, one would also need to change all subsequent blocks. As a result, the security of all previously mined blocks increases with each new block added to the blockchain.

*Bitcoin mining pools* allow miners to combine their resources and share their hashing power. The rewards are split equally based on the amount of work each miner contributes to solving a block. *Solo miners* work independently and typically use a process called *bitcoind*, a daemon that helps obtain information about new transactions from the Bitcoin network.


# Bitcoin Minging Simulation 

In this project I simulate the process of Bitcoin mining using Python and **stomp.py**. The suggested architecture for the simulation uses two topic queues named: **/topic/bitcoin/tasks** and **/topic/bitcoin/solutions**. 

## The Main Process

A *bitcoin task* is represented using a Python dictionary and it has two parts: 

* data: a sequence of 32 bytes;
* zeros: an integer number that specifies the number of leading zeros.

For example: 

```
 {
    'data': [119, 99, 109, 110, 104, 102, 108, 113, 102, 118, 122, 116, 121, 111, 119, 108, 105, 99, 109, 102, 100, 112, 102, 102, 108, 110, 98, 101, 117, 101, 97, 106], 
    'zeros': 8
 }
```

The main process will publish tasks to the miner processes using a specific topic queue named **/topic/bitcoin/tasks**. To simplify implementation, assume that the number of leading zeros is always a multiple of eight. A task is solved if a *nonce* is found such that the digest of the data portion of the task, combined with the computed nonce, has the required number of leading zeros. 

The main process subscribes to a topic queue named **/topic/bitcoin/solutions**, which is used by the miner processes to broadcast when a solution for a task is found. A *bitcoin solution* is represented using a Python dictionary and it has two parts: 

* task: the task dictionary;
* nonce: the nonce that solved the task.

When a solution is announced, the main process should extract the task reported by the miner process and save the found *nonce* into an output file. Then, if there are more tasks in the backlog, the main process should generate and publish another task.

## The Miner Processes

A miner process will receive a notification whenever a new task is published in the /**topic/bitcoin/tasks** topic queue. When a miner process finds a solution, it immediately reports back to all processes using the **/topic/bitcoin/solutions** topic queue. If a miner process receives a notification that a task it is currently working on has been solved, it should stop working on that task immediately.

## File Formats 

The input file format consists of an arbitrary number of tasks, with one task per line. Each task is defined by two parts, separated by a comma: a sequence of bytes (the data portion of the task) and the number of leading zeros. Use the provided [data/input.txt](data/input.txt) file to test your program.

The output file, named [data/output.txt](data/output.txt), contains the same information as the input file, plus the *nonce* value found, represented as a sequence of four bytes. All fields are also separated by commas.
