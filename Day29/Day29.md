## Events and Logging in Solidity

**Events**

You've might have always wondered how chainlink or the graph or other off-chain protocols work under the hood.We're going to learn about logging and events in solidity, viewing those events on an ether scan, and working with them in brownie as well.

Now it's the Ethereum virtual machine, or EVM, that makes a lot of these blockchains tick, like Ethereum, and the EVM has this functionality called "logging functionality." When things happen on a blockchain, the EVM writes these things to a specific data structure called the "log." We can actually read these logs from our blockchain nodes that we run.In fact, if you run a node or you connect to a node, you can make an eth_getLogs call to get the logs.

Now inside these logs is an important piece of logging called events, and this is the main piece that we're going to be talking about. Events allow you to "print" information to the logging structure in a way that's more gas efficient than actually saving it to something like a storage variable.These events and logs live in a special data structure that isn't accessible to smart contracts. That's why it's cheaper because smart contracts can't access them. So that's the trade off here.We can still print some information that's important to us without having to save it in a storage variable, which is going to take up much more gas.

Each one of the events is tied to the smart contract or account address that emitted the event in the transactions.Listening to events is incredibly helpful. Let's say, for example, you want to do something every time somebody calls a transfer function. Instead of always reading all the variables and looking for some to flip the switch, all you've got to do is say "Listen for event to be emitted." Instead of writing some weird custom logic to see if the parameters changed at a certain time or some weird stuff like that, when a transaction happens, an event is emitted, and we can listen for these events. This is how a lot of off-chain infrastructure works.

When you're on a website and that website reloads when a transaction completes, it actually listens for that transaction to finish. It was listening for the event to be emitted so that it could reload or do something else. It's incredibly important for front ends.It's also incredibly important for things like chainlink and the graph.

In the chainlink network, a chainlink node is actually listening for request data events for it to get a random number, make an API call, etc. Sometimes there are way too many events and you need to index them in a way that makes sense. so that you can query all these events that happen at a later date. The graph listens for these events and stores them in the graph so that they're easy to query later on. So events are incredibly powerful and have a wide range of uses.They're also good for testing.

Now that we know what events are, let's take a look at what they look like, what we can use them for, and how we might use them in our smart contract development suite.

Here's what the event is going to look like:

![events](Images/m1.png)

We've got an event here called "storedAge." So we basically have a new type of event called "storedAge." We're saying, "Hey smart contract, we've got this new event thing.We're going to be emitting things of the type storedAge in the future." When we emit this event, it's going to have these four parameters.It's going to have a uint256 called "oldAge," a uint256 called "newAge," a uint256 called "addedAge," and an address called "sender." You might have noticed there's another keyword in here.The `indexed` keyword

When we emit one of these events, there are two kinds of parameters. There are two types of parameters: indexed parameters and non-indexed parameters.You can have up to three indexed parameters, and they're also known as `topics`.So if you see a topic, you know that's going to be an indexed parameter.Indexed parameters are parameters that are much easier to search for and much easier to query than the non-indexed parameters.In fact, way back in the `eth_getLogs` function, it even has a parameter allowing us to search for specific topics. So it's much more searchable than the non-indexed one.The non-indexed ones are harder to search because they are ABI encoded and you have to know ABI to decode them.

Now this just told our smart contract that there's a new type of storedAge, a new kind of event.We need to actually emit that event in order to store that data into the logging data structure of the EVM.To do that, we need to do something like:

![emit](Images/m2.png)

It looks very similar to calling a function.So you call it emit and then the name of the emit and then you add all the parameters in there that you like. Let's jump into Brownie to learn how to use them in our smart contracts and access them with brownie.

We're going to create a new directory called "events" and open it in VSCode. We're going to create a new brownie project:

`brownie init`

Let's grab the SimpleStorage.sol contract and put it in our contracts directory.

![SimpleStorage](Images/m3.png)

Let's add an event to our contract.

![event](Images/m4.png)

We want to emit an event to tell people, "Hey, something has happened." Something has changed. In our store function, we're going to emit an event.

![emit](Images/m5.png)

Normally, you'd want to change them before emitting an event, but whatever.

To check if everything is done correctly, hit the `brownie compile`

Let's go ahead now and we'll create a script.We're going to create our own deploy.py script. This is going to deploy the contract and then we're going to call that store function so we can see how this works.The first thing we're going to do is get an account.

So we're going to create our helpful_scripts.

![helpful_scripts](Images/m6.png)

as well as our brownie-config.yaml

![config](Images/m7.png)

and we can import the get_account function in our deploy scripts. We need to deploy our contract.

![deploy](Images/m8.png)

After we deploy our contract, we can do:

![callingStore](Images/m9.png)

So we've got the transaction. This transaction has a whole bunch of stuff.So let's print it and run the script on a development chain.

`brownie run scripts/deploy.py`

![output](Images/m10.png)

We get the transaction object.This transaction has all the events in it.So we could print the events.

![printingEvents](Images/m11.png) 

![output](Images/m12.png)

oldAge and newAge are the indexed parameters or the topics, and the totalAge and the sender are going to be non-indexed. We can read them here because we've got the ABI. We've got the contract code and we know what totalAge and sender are called. If we didn't have the contract code, non-indexed would show up as mumbled garbage. Since there's only one event, we could even do:

`print(txn.events[0]["newAge"]`

Let's deploy this to etherscan and see what it looks like on etherscan because working with the etherscan events and understanding how those events work and what they look like on etherscan is really important too.So in our config:

![config](Images/m13.png)

Let's verify the contract.

![verify](Images/m14.png)

Put the etherscan API key in your .env file.




`brownie run scripts/deploy.py --network kovan`

If you get an error like:

`"Non-hexadecimal digit found"`

Then, in your configuration, make the following change:

![config](Images/m15.png)

So we've deployed and verified this.Let's go take a look at this on-chain.In our Kovan etherscan of that contract that we just deployed and verified.If we go to the transaction and its log, we can see its event.

![logs](Images/m16.png)

The topic 0 is always going to be the whole signature of the event.




