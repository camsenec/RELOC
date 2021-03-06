# Data-analysis-enabled Notification Generation and Delivery System for Edge Computing

## Overview
This system allows real-time and interactive data-analysis-enabled notification delivery among clients such as self-driving car and IoT devices. The importance of real-time notification has been growing for social services and Intelligent Transporting System (ITS). As an advanced version of topic-based Pub/Sub systems, topic-based publish-process-subscribe systems, where published messages are spooled and processed on edge servers, have been proposed to achieve data-driven intelligent notifications. This system allows a topic to be managed on multiple edge servers so that messages are processed near publishers and subscribers, even when publishers are spread over a wide area. Moreover, this system allocate topics adequately on edge servers with limited edge server resources to achieve real-time notifications.

## Infrastracture
This system is implemented with a web framework Django, which is one of the Python web frameworks and offers high security and scalability. AWS EC2 instances are used as server side which can dynamically adjust resource usage according to user demand. Database is constructed with SQLite. Clients send request to Edge Manager to get server to be connected at regular intervals considering client mobility. 

![infra](https://user-images.githubusercontent.com/27656483/77508784-de3b1400-6eae-11ea-8de7-a501ec7935ec.png)


## API

### Base URI

http://<host_name>/api/v1/manager



## User [/user]

### (POST) Client Registration [/post]

Register a client to the Edge Manager service. Client ID is given to the registered client. 

Besides, Two-way connection is built between client and allocated home server represented as `home`.



+ Attributes (multipart/formdata)

  + `x` :  x coordinate of client position (latitude)
  + `y` :  y coordinate of client position (longitude)



- Parameters (URL parameter)
  - `application_id` : id of application the client who send request is using.



- Request Example

  ```bash
  $curl -XPOST -F "x=30.0" -F "y=30.0" http://localhost:8000/api/v1/manager/user/post?application_id=1
  ```

  

+ Response 200 (application/json)

  + Body

    ```json
    {
     "application_id" : 1,
     "client_id" : 1149989,
     "x" : 30,
     "y" : 30,
     "home" : 89
    }
    ```



### (PUT) Update Client Location and Home Server [/update_location]

Position data of the client is updated and new home server is allocated according to the mobility of a client. 

+ Attributes (multipart/formdata)

  + `x` :  x coordinate of current client position (latitude)
  + `y` :  y coordinate of current client position (longitude)

  

- Parameters (URL parameter)
  - `application_id` : id of application the client who send request is using.
  - `client_id` : id of the client who send request



- Request Example

  ```bash
  $curl -XPUT -F "x=70.0" -F "y=70.0" http://localhost:8000/api/v1/manager/user/update_location?application_id=1&client_id=1149989
  ```

  

+ Response 200 (application/json)

  + Body

    ```json
    {
     "application_id" : 1,
     "client_id" : 1149989,
     "x" : 70,
     "y" : 70,
     "home" : 45
    }
    ```



## Edge Server [/server]

### (POST) Server Registration [/post]

Register an edge server to the Edge Manager service. Server ID is given to the registered edge server.



+ Attributes (multipart/formdata)

  + `x` :  x coordinate of server position (latitude)
  + `y` :  y coordinate of server position (longitude)
  + `capacity`: The cache capacity given to the application identified by the `applicaiton_id` field



- Parameters (URL parameter)
  - `application_id` : id of application the client who send request is using.



- Request Example

  ```bash
  $curl -XPOST -F "x=30.0" -F "y=30.0" -F "capacity=10000" http://localhost:8000/api/v1/manager/user/post?application_id=1
  ```

  

+ Response 200 (application/json)

  + Body

    ```json
    {
     "application_id" : 1,
     "client_id" : 1149989,
     "x" : 30.0,
     "y" : 30.0,
     "capacity" : 10000,
     "used" : 0,
     "cluster_id" : 0
    }
    ```
    
## Theory

1. Our is to realize immediate and flexible content sharing levaraging edge servers.

![fig1](https://user-images.githubusercontent.com/27656483/77507360-a8486080-6eab-11ea-9bfe-281d6e26337d.png)

2. There are many existing research on content caching utilizing edge servers. In common, what content should be placed on which edge server is being considered considering the finiteness of edge server resources \[1\], \[2\], \[3\]. The studies shown on the left propose a method for selecting content to be cached on edge servers based on access frequency, etc. This allows users to get the desired content instantly.On the other hand, we are interested in applications that are highly interactive. However, Previous study as shown in the upper right only realize sharing public content among clients\ [4\]. Our aim is to enable real-time content sharing among specific users, as shown at the bottom right. Here, in the lower right figure, the content is shared with the vehicles colored by blue, but not with the vehicles colored by green.

![fig2](https://user-images.githubusercontent.com/27656483/77507364-aaaaba80-6eab-11ea-89f3-42b005cdeadd.png)


3. As a related work on content sharing among specific users, a system based on the Publisher/Subscriber model that can transmit content only to a specific destination has been proposed [5]. In this system, each user hold a cache of shared content on a fixed edge server. The fixed edge server is called the `home server`.  Suppose that the publisher transmit content updates to its subscriber. First, Publisher updates content on her/his home server. The updates are immediately synchronized to the subscriber's home server. This gives the consistency of the content. Finally, the updated content is propagated by push notification to the destination user. At this time, the content is not shared to User P who is not set as the destination and User Q who is not sharing the same content. We realize resource sharing and real-time content sharing based on the model.

![fig3](https://user-images.githubusercontent.com/27656483/77507365-ab435100-6eab-11ea-8b9f-a2e7e322731e.png)

4. The problem is that cache capacity and bandwidth can be over-consumed as the number of users increases.

![fig4](https://user-images.githubusercontent.com/27656483/77507372-aed6d800-6eab-11ea-8e20-bfd9f305f995.png)


5. Our policy is to realize efficient use of edge server resources and to reduce content delivery delays.
   First, we first formulated the content transmission delay. The formulated cost model is suitable for interactive content sharing among specific clients. Second, we developed algorithm to reduce the content delivery delay D which is defined in the first step.
   One idea is to use the strong connectivity between publishers and subscribers.
   If users who have strong connectivity utilize the same content cache, the consumed cache capacity can be reduced.

![fig5](https://user-images.githubusercontent.com/27656483/77507378-b26a5f00-6eab-11ea-9806-464a86ae18ad.png)


6. Based on these ideas We propose RLCCA Algorithm which effectively reduce resource usage on edge computing node and reduce content delivery delay remarkably. We implement RLCCA Algorithm on this Edge Manager service and realize novel client and content cache allocation on edge servers. Numerical experiments which uses this Edge Manager show that the proposed method can reduce the content transfer delay by up to 55% compared to existing methods.

![fig6](https://user-images.githubusercontent.com/27656483/77507382-b4342280-6eab-11ea-93f8-44ecc4868db5.png)

### References

[1]  Q. Li, W. Shi, Y. Xiao, X. Ge, A. Pandharipande, “Content size-aware edge caching: a size-weighted popularity-based approach,” in IEEE Global Communications Conference (GLOBECOM), Dec. 2018. 

[2]  J. Ahn, S. H. Jeon, H. Park, “A novel proactive caching strategy with community- aware learning in CoMP-enabled small-cell networks,” IEEE Communication Let- ters, Vol. 22, No. 9, pp. 1918–1921, Sept. 2018. 

[3]  L. Hou, L. Lei, K. Zheng, X. Wang, “A Q-learning-based proactive caching strategy for non-safety related services in vehicular networks,” IEEE Internet of Things Journal, Vol. 6, No.3, pp. 4512–4520, June 2019. 

[4]  N. Itoh, H. Kaneko, A. Kohiga, T. Iwai, H. Shimonishi, “Novel packet scheduling for supporting various real-time IoT applications in LTE networks,” in IEEE International Workshop Technical Commitee on Communications Quality and Reliability (CQR), May 2017. 

[5]  T. Nagato, T. Tsutano, T. Kamada, Y. Takaki, C. Ohta, “Distributed key-value storage for edge computing and its explicit data distribution method,” IEICE Transactions on Communications, Vol. E103.B, No.1, pp. 20–31, Jan. 2020. 




## Programing language

- [Base System] Go

- [Edge Manager] Python (Django)

- [Simulator] Java

  

## Other git repository

- [Base System] Private
- [Simulator] https://github.com/thanatoth/edge-simulator



## Author

Tomoya Tanaka
