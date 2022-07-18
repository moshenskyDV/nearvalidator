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
