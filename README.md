# How to setup Near validator node on Google Cloud

Hello everyone, this article is about configuring your own chunk validator node on Near Protocol.
For validator node you need minimum 90% uptime, ideally 99%+, so it means that cloud solution would be best choise to run this node.
I recommend to use any of the famous cloud providers, such as: GCP, AWS, DigitalOcean etc.
I decided to stop on Google Cloud Provider.

Before we start to configure our node - lets do some calculations.
The requirements for machine, provided by Near team is:

<img width="318" alt="image" src="https://user-images.githubusercontent.com/15670713/179549321-94a30ca5-6baf-4fb6-b01b-602e823d2669.png">

So I decided to take 4-core CPU, 8GB RAM and 250GB of storage (I'll increase storage size on production mode). You can check my calculations by this <a href="https://cloud.google.com/products/calculator/#id=17004408-62d0-47d5-bb23-a7b5e857e940">link</a> or on image below.

<img width="378" alt="image" src="https://user-images.githubusercontent.com/15670713/179548363-9f729964-cf58-4135-afa8-88a8733754c3.png">

It will cost around 110-140$ per month.
If you pre-pay server for year or 3 years - it will be around 50-70$ per month.

Ok, after buying cloud instance - lets connect via SSH and configure NEAR validator step-by-step.
First of all we need to make sure all software is up to date:

```
sudo apt update && sudo apt upgrade -y
```
Next step - is installing NodeJS (Near-CLI works on NodeJS) and installing Near-CLI:

```
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash - 
sudo apt install -y build-essential nodejs
PATH="$PATH"
sudo npm install -g near-cli
```

Also, for the next commands we need to let Near-CLI know, that we want to connect to `shardnet`. We can do this by setting environment variable:

```
export NEAR_ENV=shardnet
```

To make sure that everything works correct you can run `near validators current` command. It should retrieve the list of current validators, like on screenshot.

<img width="840" alt="image" src="https://user-images.githubusercontent.com/15670713/180605253-81189c1d-3291-432a-a38e-2f67af64fccd.png">

Before we install validator - Near developers prepared script to make sure that your CPU support necessary instructions.

```
lscpu | grep -P '(?=.*avx )(?=.*sse4.2 )(?=.*cx16 )(?=.*popcnt )' > /dev/null \
  && echo "Supported" \
  || echo "Not supported"
```

If your console prints "Supported" - we can move forward. If it prints "Not supported" - your hardware require some update, before you can run validator node.
Installing necessary libraries and packages:

```
sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo python3-pip
```

Add some configurations and install Rust with package manager (Cargo):

```
USER_BASE_BIN=$(python3 -m site --user-base)/bin
export PATH="$USER_BASE_BIN:$PATH"
sudo apt install clang build-essential make
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

During the installation you need to answer 1 (proceed with installation) when asked.

One more config:

```
source $HOME/.cargo/env
```

Next steps - downloading and compiling Near core sources, to do this we need to clone repository:

```
git clone https://github.com/near/nearcore
```

!!Attention!! The validators on this moment (24.07.2022) is unstable feature, it's getting some hotfixes from developers, so last stable commit provided by developers by this <a href="https://github.com/near/stakewars-iii/blob/main/commit.md">link</a>. I suppose in the near future, when feature become stable - you can skip next step and compile last commit of default branch. The article will be updated when feature become stable.

```
git checkout 0f81dca95a55f975b6e54fe6f311a71792e21698
```

And finally, build it:

```
cargo build -p neard --release --features shardnet
```


Than we will run `init` command to let it create default configs:

```
./target/release/neard --home ~/.near init --chain-id shardnet --download-genesis
```

And we need to replace some default settings by our validator settings. This settings located in file `~/.near/config.json`. We will replace it by fresh list of boot nodes and shards:

```
rm ~/.near/config.json
wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
```

Before running the validator node we should do one more step - login into Near-CLI.

```
near login
```

This command will print the link to console (like on screenshot below). Copy the link and open it via browser.

<img width="398" alt="image" src="https://user-images.githubusercontent.com/15670713/180608282-d95b32cb-eb77-4a89-b28d-3a7384c3a855.png">

Make sure link contains `shardnet` word. It should start from `https://wallet.shardnet.near.org/`.
After login procedure you will see something like on the screen.

<img width="356" alt="image" src="https://user-images.githubusercontent.com/15670713/180608772-b7e98c99-bed5-4480-9662-95caba25fd82.png">

Don't be scared, it's ok. Redirect suppose that you have node on local machine. On our remote machine we need to return to our console and write wallet name on request:

<img width="345" alt="image" src="https://user-images.githubusercontent.com/15670713/180608924-b385e3fb-87c9-44ed-b3e3-f9e8ca95dda0.png">

If everything is ok - you will see "Logged as [...]" in console.

Next step - is configuring `validator_key.json`.
If you didn't use validators before on this machine - probably it shouldn't exist on your machine.
For the next steps we need to introduce new address: `xx.factory.shardnet.near`. This is pool address.

On my example main address `moshenskyi.shardnet.near`, so pool address will be `moshenskyi.factory.shardnet.near`.
Let's generate key pair for the pool (replace my pool address by yours) and copy it to `.near` folder:

```
near generate-key moshenskyi.factory.shardnet.near
cp ~/.near-credentials/shardnet/moshenskyi.factory.shardnet.near.json ~/.near/validator_key.json
```

We need to correct this file (change naming: private_key -> secret_key). Also make sure that account_id matches your pool address.

```
vim ~/.near/validator_key.json
```

Save the changes. That's the moment when we can start the node:

```
target/release/neard run
```

If everything is correct it should start to download headers, like on the screenshot below.

<img width="870" alt="image" src="https://user-images.githubusercontent.com/15670713/180609537-3e14c6b0-adb6-496d-9b7f-ab1ecd4c91a6.png">

All right. The node is running. This is real time process: when node is running - it validates the network and takes profit. So to prevent process from stopping when we close the console - we can configure it as a service (daemon).

```
sudo vim /etc/systemd/system/neard.service
```

Paste this content (replace with your path and user):

```
[Unit]
Description=NEARd Daemon Service

[Service]
Type=simple
User=root
#Group=near
WorkingDirectory=/root/.near
ExecStart=/root/nearcore/target/release/neard run
Restart=on-failure
RestartSec=30
KillSignal=SIGINT
TimeoutStopSec=45
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

Next command will initialize the service:

```
sudo systemctl enable neard
```

If we want to start or stop the service, we can do this any time with one of the commands:

```
sudo systemctl start neard
sudo systemctl stop neard
```

This command show the state of service, it should be green, when started:

```
sudo systemctl status neard
```

<img width="513" alt="image" src="https://user-images.githubusercontent.com/15670713/180610367-bdabab70-c65f-4c21-ba3a-8fd7239596e0.png">

For beautiful logs let's install `ccze`:

```
sudo apt install ccze
```

Next command checks the logs:

```
journalctl -n 100 -f -u neard | ccze -A
```

All right. Now we have an validator that is able to connect to other validators and support the network. Validator is attached to our account. Next step - is fulfilling validator by tokens. By design, validator should have at least "seat price" tokens staked into it. First staker will be our main account `moshenskyi.shardnet.near`. Let's stake some Near!
To have ability to stake near - we need to deploy a smart contract which will implement staking on account. Near-CLI has already template for it, so can be sure that smart contract is secure (change the parameters to your pool respectively). Public key can be found in `~/.near/validator_key.json`. Numeration parameter means how much percents of reward your pool will take from stakers as fee. For example my pool takes 3% fee.

```
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "moshenskyi", "owner_id": "moshenskyi.shardnet.near", "stake_public_key": "ed25519:HCWiykC8ceC7S2CjuvVZaL5eRzTJUiXsaWYtNWq5Vekc", "reward_fee_fraction": {"numerator": 3, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="moshenskyi.shardnet.near" --amount=30 --gas=300000000000000
```

Smart contract is configured. It's time to enrich it with some coins. Check the seat price on <a href="https://explorer.shardnet.near.org/nodes/validators">explorer</a> and send at least this amount of Near to validator:

```
near call moshenskyi.factory.shardnet.near deposit_and_stake --amount 300 --accountId moshenskyi.shardnet.near --gas=300000000000000
```

To make sure we staked some coins lets check it:

```
near view moshenskyi.factory.shardnet.near get_account_staked_balance '{"account_id": "moshenskyi.shardnet.near"}'
```

<img width="735" alt="image" src="https://user-images.githubusercontent.com/15670713/180616932-142467a7-645a-4bef-a00a-eaa5ac78fea9.png">

If you got a big number - it's ok. The measurement in YoctoNear. Check details here: <a href="https://nomicon.io/Economics/Economic">Near Economics</a>
The validator should appear in <a href="">Near Explorer</a>

<img width="839" alt="image" src="https://user-images.githubusercontent.com/15670713/180618962-b8df5bb9-da97-46ed-bbe9-be77d0d5324d.png">

We finished all preparations. Let's run the validator by `sudo systemctl start neard` and wait until it fully syncronized. It should download all headers first.
After that we can tell the network that we are ready to participate (and keep telling it every epoch, eg: every 4 hours). For this we need to ping the network:

```
near call moshenskyi.factory.shardnet.near ping '{}' --accountId moshenskyi.shardnet.near --gas=300000000000000
```

As far as I need to do it every 4 hours I decided to create a CRON task:

```
cd ~
vim ping.sh
------>Paste previous ping command.

crontab -e
------> paste the code below:
0 */2 * * * /ping.sh
```

If my server is working - every two hours it will send signal to network that my validator is OK and ready to work. It's importants, because if you didn't confirm your status - your validator will be kicked in next epoch.

You can use your instance as RPC node, so you can do requests to network. For example, if you want to check your node delegators and their stake - you can write (change pool id and accout id to yours):

```
near view moshenskyi.factory.shardnet.near get_accounts '{"from_index": 0, "limit": 10}' --accountId moshenskyi.shardnet.near
```

<img width="803" alt="image" src="https://user-images.githubusercontent.com/15670713/180619662-54c4181f-420c-48a0-8f53-fd1ad596332a.png">


That's it. Node is working and validating the network. If you have any questions or suggestions - please write in comments below!
Remember your validator should have uptime >90% to not be kicked from network, also it's much better to take it >99% for reward size.
