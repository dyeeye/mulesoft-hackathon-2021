# Needful Things a MuleSoft Hackathon 2021 project
Needful Things is a project for a MuleSoft Hackathon 2021

## Authors
- Patryk Bandurski
- Dominik Kruszewski

## Submission video



## Inspiration

When I buy some goods I just need to see them. On-site, online, that does not matter. Anytime I can see the bought product before I actually received it I am so excited. One of the online stores surprised me by sending me a photo of the products I ordered on the warehouseman's desk during collection. WOW. It built my trust in the company and I thought treated special. 

So I thought, why is this so rarely done? That is how I come with the idea of the **Needful Things** project.

## What it does

We expose the mobile web page for the warehouseman/woman to take a picture of the collected goods. Only the picture is needed for submission. The rest is done on MuleSoft using such third parties as **S3**, **Amazon Rekognition**, and **Salesforce**. We detect what goods are on the picture, then we look for orders that match products from the image and we send order numbers to the warehouseman/woman who can tell if the match is done or not. After simple confirmation, we send an email to the customer with the picture and information about the order being collected. Here you can see this simply depicted.

![What the project does](https://raw.githubusercontent.com/dyeeye/mulesoft-hackathon-2021/main/images/What%20it%20does.png)

## How we built it

![High Level diagram](https://raw.githubusercontent.com/dyeeye/mulesoft-hackathon-2021/main/images/High%20Level%20Architecture.png)
We adopted an API-led approach to building a scalable API network. On the high-level architecture diagram, you can see the main components of the Anypoint Platform we use. All the APIs are deployed within the private **VPC**, so they are not available from the internet. In front of the Experience API, we have a **Dedicated Load Balancer** that exposes the web socket and adds a security layer on top of it. In other words, we expose **secured web socket (WSS)** to the internet. The protocol was carefully picked to enable the mobile application to have a bidirectional communication channel to send and receive callback notifications.

As the AI image processing and matching orders logic may take a while (around 10 seconds), we added asynchronous processing enabled by **Solace Queues** between experience API and process APIs. Communication between process and system APIs is using standard REST APIs over HTTPS protocol. 

REST APIs are designed using RAML specification in Design Center and published to Anypoint Exchange. Process and System APIs for consistency reasons are using the Bounded Context Model. It is achieved by using API Fragments.

REST APIs are governed by using out-of-the-box policies available in **API Manager**. Services are secured with **Client ID Enforcement**. 

Let's take a look at three-layer architecture. You can see on the diagram that we expose functionality via a single Experience API. To do orchestration we have two process APIs. For each 3rd party we are connecting to we have a system API.

![API-led](https://raw.githubusercontent.com/dyeeye/mulesoft-hackathon-2021/main/images/API-led.png)

## Challenges we ran into

There were a couple of challenges. The first one was the consumption of the Amazon Rekognition service. This was a big unknown at the beginning of the project. Both authorization and API usage. At glance, it may seem to be trivial but the deeper we went the more interesting it became. First, we tried custom labels, to teach Amazon Rekognition to recognize our products, but it turned out to be too much for our free tier, so we adjusted it to work with the standard Amazon Rekognition tool. Although it will work the custom label if we will decide to use it in the future.

Another challenge was to understand how Order processes work in Salesforce. We needed to create products and associate them with the order. But this is again not a trivial thing if you are not a Salesforce Guru. Take a look at the diagram below. In other to connect your product with the order we need to go through a couple of objects. Taking into account the lack of experience in Salesforce, the challenge was to create an optimal algorithm that allows you to associate products with orders, having only a description of the items.
![Salesforce Order model](https://raw.githubusercontent.com/dyeeye/mulesoft-hackathon-2021/main/images/hackathon_sf.png)

## Accomplishments that we're proud of

I am proud of the algorithm used to authenticate to **Amazon Rekognition API**. Amazon secures services with a very complex algorithm - [AWS Signature Version 4](https://docs.aws.amazon.com/AmazonS3/latest/API/sig-v4-authenticating-requests.html). It contains many steps that we need to carefully compute in order to have an authentication token. I created reusable DataWeave module. [Here](https://raw.githubusercontent.com/dyeeye/mulesoft-hackathon-2021/main/rekognition-sapi/src/main/resources/dwl/com/ambassadorpatryk/aws/auth.dwl) you can see its implementation.  It was a challenge. 

## What we learned

We better understood now Salesforce in order context. During the project, we worked closely with Salesforce. It was something that we found very interesting as usually on work-related projects we cooperate with Salesforce Gurus who do the tricks with Salesforce. 

We played a little bit with Solace messaging system. It meets our needs and what is more, this is a cloud-based solution. Solace documentation is straightforward and has many tutorials include one about MuleSoft. 

## What's next for Needful Things

In case of the future I think of the following:
* teaching Amazon Rekognition custom labels to detect actual products
* make a mobile application for warehouse staff instead of the web page
* integrate with salesforce marketing cloud for customer notifications


## How to run
You need to replace secure properties in System APIs in dev-secure.yaml file. The project is designed to work well on CloudHub infrastructure.