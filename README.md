# Bug-Tracking-Messaging-Server

## Table of Contents
- [Local Architecture](#Local-Architecture)
- [System Architecture](#System-Architecture)
- [References](#References)


### Local Architecture

This repository represents the Messaging server component of the distributed system.

In more detail we have:
![](https://github.com/alexandreLamarre/Bug-Tracking-Messaging-Server/blob/main/local_arch.png)

Technologies:
- **Backend: Golang/ Gorilla websockets**
- **Database: Redis**
- **Frontend: React.js/javascript websockets**

 ### Why Golang as the backend technology?

**Pros**: 
- First of all, Golang is **fast**. For example, comparing it to a Python backend framework like Flask, the difference in speed benchmarks is staggering. This allows us to **scale up our application more easily** and at a reduced cost. 

- Golang has excellent websocket support in the form go/gorilla.

- Golang's concurrency implementation, combined with its ease of use, allows us to scale up to a million concurrent bidirectional connections with very low resource cost.

- The language is simple and fast to develop in. Golang is also easy to test and benchmark.

- Golang is especially easy and fast to containerize.

**Cons**:

- Third party library support can be inconsistent, ill-maintained or non-existent for alot of nice backend features, which means you need to develop your own. Fortunately, we don't need to support any complex features in our messaging server so this isn't a huge concern.

- Golang has few built-in libraries and datatypes, which means you need to write your own. Again, we don't need to support any complex features in our message server so this isn't a huge concern.



### Why Redis as our database technology?

First, we need to explain why we need, or should have, a **database separate from our main shared persistent storage** for our messaging server.
Our messaging server needs to be **stateless** in order to be horizontally scalable. Part of what this means is that the websockets on one instance of the messaging server cannot directly communicate with the websockets of another instance. Thus, if two websockets instances need to communicate with each other, they have to reside on the same messagin server instance.

We can solve this by hashing the clients IP's of clients that need to communicate together to the same server. However, if we have on client that has to communicate with potentially every other client connected, then we would end up hashing most clients to the same server, probably crashing it (depending on the total current application traffic). 

To resolve the issue of requiring websockets to reside on the same instance we can introduce some **middleware** to control and manage hashing connections to different messaging server instances. But we eventually run into alot of the same issues; having to manage a large amount of hashing and potentially opening multiple, redudant websocket connections which will become resource expensive to maintain. Not to mention we may requirehave an engineer or a team of engineers to maintain thr middleware in case the messaging server or client server updates: it begins to **couple separate components**, which we absolutely don't want.

**The solution: Redis**. We can manage the inherit 'states' in open connections easily. If we use our main persistent storage we may slow down the other features that depend on it and needlessly increase traffic and costs.

- With redis, we need to **maintain only one open websocket connection per client**, regardless of the number of message namespaces they may be a part of.

-  We can **leverage the highly efficient caching of Redis** to hash the listener portions of the websocket to the connected namespaces inside one instance of the messaging server and hash back the response to redis. It keeps our messaging server lightweight as well, by not having to manage multiple messaging namespaces by itself.

-  We can easily **horizontally scale Redis**. 

-  **We don't need to maintain any persistent storage** related to Redis since Redis only needs to keep track of the open connections while running. Therefore there is no maintance cost, in terms of human labour, or persistent storage cost associated to Redis.

### Why Vanilla Javascript websockets?

- We don't need any complex behaviour (provided by third party libraries, such as socket.io) for our websockets.
- Redis already manages our namespace behaviour efficiently so we don't need to build complex behaviour into the websocket technology.


### System Architecture

The system is based on a cloud-hosted dsitributed systems architecture, deployed via Kubernetes & automated via Jenkins. 

![](https://github.com/alexandreLamarre/Bug-Tracking-FrontEnd/blob/main/bug_tracking_arch.png)





### References
