# PoolTogether Workshop

In this workshop we're going to:

1. Deploy the entire PoolTogether system locally
2. Create a new Prize Pool and deposit and withdraw
3. Create a custom prize strategy and swap it out the old one for this one

# Setting up PoolTogether Contracts

Setup the [PoolTogether Contracts](https://github.com/pooltogether/pooltogether-pool-contracts).

```bash
$ git clone git@github.com:pooltogether/pooltogether-pool-contracts.git
$ cd pooltogether-pool-contracts/
$ git checkout version-3
$ nvm use
$ yarn
$ yarn start
```

In another terminal, load up a buidler console:

```bash
$ buidler console --network localhost
```

You can get all the deployed contracts like so:

```js
> deployed = await deployments.all()
```

Find the deployed addresses like so:

```js
> deployed.SingleRandomWinnerBuilder.address
> deployed.CompoundPrizePoolBuilder.address
> deployed.cDai.address 
> deployed.Dai.address
> deployed.RNGServiceMock.address
```

# Setup the Builder UI

Setup the [Builder UI](https://github.com/pooltogether/pooltogether-pool-builder-ui).

```bash
$ cd ..
$ git clone git@github.com:pooltogether/pooltogether-pool-builder-ui.git
$ cd pooltogether-pool-builder-ui
$ nvm use
$ yarn
```

Now copy over the .envrc.example

```
$ cp .envrc.example .envrc
```

Update the .envrc with your infura key and allow direnv:

```
$ direnv allow
```

Update the `lib/constants.js` file with the deployed address for network 31337.  

You can just use the cDai address for all Compound cTokens.

Now start the server:

```
$ yarn dev
```

Let's build a pool!

In the original PoolTogether Contract console let's give ourselves some ETH and Dai:

```js
> signers = await ethers.getSigners()
> signers[0].sendTransaction({ to: "your_address", value: ethers.utils.parseEther('10') })
> dai = await ethers.getContractAt('ERC20Mintable', deployed.Dai.address, signers[0])
> dai.mint("your_address", ethers.utils.parseEther('1000'))
```

Now we can go and create a new pool

# Setup the Reference Pool UI

Setup the [Reference Pool UI](https://github.com/pooltogether/pooltogether-reference-pool-ui)

```bash
$ cd ..
$ git clone git@github.com:pooltogether/pooltogether-reference-pool-ui.git
$ cd pooltogether-reference-pool-ui
$ nvm use
$ yarn
```

Now copy over the .envrc.example

```
$ cp .envrc.example .envrc
```

Update the .envrc with your infura key and allow direnv:

```bash
$ direnv allow
```

Start the reference UI server

```bash
$ yarn dev --port 4000
```

Navigate to `localhost:4000` and paste your newly created pool address


# Create a custom Prize Strategy

Use the [Custom Prize Strategy](https://github.com/pooltogether/custom-prize-strategy)

Setup the project:

```bash
$ cd ..
$ git clone git@github.com:pooltogether/custom-prize-strategy.git
$ cd custom-prize-strategy
$ nvm use
$ yarn
```

Deploy the strategy to the earlier node:

```bash
$ buidler --network pt deploy
```

Now spin up a console

```bash
$ buidler --network pt console
```

Get the contracts:

```js
> deployed = await deployments.all()
```

Get the builder contract:

```js
> signers = await ethers.getSigners()
> builder = await ethers.getContractAt('MultipleWinnersBuilder', deployed.MultipleWinnersBuilder.address, signers[0])
```

Now let's vampire the old strategy.

First we'll check to see what the address will be:

```js
> await builder.callStatic.createMultipleWinners('0x93498f3054Aca6dEe768DCA2762C83FC0F06c70d', 2)
```

Save that address!

Now let's create the strategy:

```js
> await builder.createMultipleWinners('0x93498f3054Aca6dEe768DCA2762C83FC0F06c70d', 2)
```

Now go back to the original Pool contracts console and get a reference to the pool:

```js
> pool = await ethers.getContractAt('PrizePool', '0x06134447473A1ad19dd05554780816C542486d15', signers[0])
```

Now let's swap out the strategy:

```js
> await pool.setPrizeStrategy('0xdbaA15927b1463EdD14Cf51D082BD7703Fd1C238')
```

