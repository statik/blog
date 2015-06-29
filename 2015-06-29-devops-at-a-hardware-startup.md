# DevOps at a Hardware Startup
#### Jordan Stone - Chief Software Architect at Notion <getnotion.com>

## What is Notion?
The Internet of Things (IoT) is all the rage these days. Every major tech blog is writing about it, and maybe even your own mother has asked "Hello sonny. What is a Connected Home and why do I want one?"

As IoT moves to the mainstream, it's increasingly important for us to build products that make adopting this new technology dead simple for the average consumer. And that's what we are doing at Notion.

Notion is an oreo-sized multi-function sensor capable of all kinds of things. Place a single Notion Sensor on your front door, and we can tell you if that door opens or closes, the temperature in the room, and even if a nearby smoke alarm goes off. But that's not all. Notion can tell you if a water leak springs up from that bathroom sink or water heater, if a liquor cabinet or gun safe is accessed, and even the level of propane left in the tank on your grill.

We do this through multiple sensors all packed together and working in concert. I won't go into the details of all of the sensors we use, but feel free to check out the [How it Works](http://getnotion.com/how-it-works) and [Technology](http://getnotion.com/technology) pages on our website.

## DevOps and Hardware?
So how does DevOps work differently at a hardware company than at a software company? Well, it doesn't really. While we have a physical product that we build alongside all of our software, we view those physical products as clients, just as we view our mobile apps and our website as clients. Sure, the hardware clients consume a different set of resources, but they're clients all the same.

However, the fact that we sell a "thing", and building "things" to sell is *very expensive*, our DevOps from a software perspective is actually impacted. Our sensors have a radio inside that speaks a custom protocol that we built. Your phone doesn't have the radio to speak this protocol. Neither does your WiFi router. And so, we also make a Bridge that is responsible for coordinating messages from sensors and turning those into messages that can be consumed by another service for processing, alerting, storage, etc. Building a bridge that can do all of the message translation, as well as the data processing, storage, alerting, and any other feature we may add in the future becomes very expensive very quick. Since one of the barriers to entry for IoT is the cost (how do you convince someone to replace their $30 thermostat with a $250 one?), we were very sensitive to how much cost we were adding to our product. Building in the cloud is cheap, by comparison, and so that's where most of the "brains" of Notion lives.

## Our DevOps Journey
Like most other companies, we started with a monolithic application that was easier to build and test. As the functionality of our system grew in complexity, we slowly started breaking our monolith into smaller pieces. Some people like to call these microservices. You may also know it as SOA. Either way, I firmly believe that having built the monolith first allowed us to better understand the service definitions and boundaries.

Additionally, automation and monitoring have been two of our biggest assets. As a very small team (our entire company is just 11 people), we obviously have a ways to go, but one of the biggest benefits of our DevOps process is the automation we've put into place.

We use [Codeship](https://codeship.com) for Continuous Integration and Continuous Delivery. We basically maintain two main git branches (`develop` and `master`), along with any feature branches. Pushing to `develop` triggers a Codeship build, which runs all of the tests for that microservice. If the tests pass, we build a new Docker image from that code, and notify all of the services running that container in our development environment to pull the latest image and restart themselves. It's not technically a zero-downtime deploy, but containers spin up so quickly as compared to VM's that we haven't seen any impact on our users thus far. Once that new deployment goes through some QA, we can merge and push to `master`, which follows a similar deployment process for Production.

## Containers, Microservices, and DevOps
This is maybe an over-used title at this point, as it seems like everyone is trying to figure out how to rewrite their entire system to use containers and microservices. For us, the concept of immutable infrastructure is what is most appealing about container services like Docker. Previously, we had built an [Autoscaler for DigitalOcean](https://github.com/looplabsinc/autoscalr) that, while functional, required a ton of maintenance. Any time we updated a package for a service, we needed to bring down that VM to take a snapshot, use the DigitalOcean API to find that snapshot's ID, update the `autoscalr.yml` file, redeploy the file, and restart Autoscalr to pick up the changes. While we weren't making package changes all that often, it became extremely cumbersome once we started distributing our architecture across hosts. And bringing down a host to take a snapshop meant every service needed a load balancer with multiple instances running to make sure we didn't interupt service.

Immutable infrastructure with containers allows us to do a few things: First, since all of the packages and dependencies are contained in the image, we can update those images locally, test those changes, and deploy to Production without worrying (as much) about "will this work in my Production environment?". I say "as much" because, at the end of the day, pushing to Production always comes with a little bit of uncertainty, no matter how similar to Production you make your lower environments. Your Staging environment could be the exact same as Production, but if a Production host goes down, you're S.O.L. no matter what. Secondly, an immutable infrastructure sets you up for a self-healing system without much additional effort. If you know that every deploy of a service is going to result in the same artifact, you can kill misbehaving or unhealthy servers without a second thought. It's the whole "Pets vs Cattle" thing. Pets are servers you care too much about. You gave them specific names. They've been configured just so. And, you probably did that configuration so long ago that it has either drifted from the configuration you need today, or you forgot what steps you took to configure it in the first place. Cattle, on the other hand, are a dime a dozen. You can slaughter them and raise new ones on a whim. Maybe it's not the most PETA-friendly analogy, but I think you get the point. Immutable infrastructure also shields you from that configuration drift.

Now, containers, microservices, and immutable infrastructure come with their own set of complexities. As does building a cloud solution instead of a local one. And we had some of our early customers express interest in a local version of Notion that didn't depend on the cloud. But, at the end of the day, we're building Notion for everyone, not any one group of people. And building a product that is accessible like that comes with tradeoffs. Time will tell if we made the right ones, but, for now, we're incredibly excited about Notion and all of the intelligence it can bring to you and to your home.

## Want to know more?
Check out our website at [http://getnotion.com](http://getnotion.com), or shoot me an email at [jordan@getnotion.com](jordan@getnotion.com). We'd love to hear your thoughts, or answer any questions you may have.