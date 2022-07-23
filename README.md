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

Next steps - downloading and compiling Near core sources, to do this we need to clone repository and build it with cargo:

```
git clone https://github.com/near/nearcore

```
