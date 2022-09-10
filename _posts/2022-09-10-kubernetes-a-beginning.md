---
layout: post
title: *Kubernetes:* A Beginning
subtitle: Day 2
tags: [100DaysOfCloud, DevSecOps]
cover-img: [REPLACE_ME]
thumbnail-img: [REPLACE ME]
---

# 🤨 Let's define Kubernetes

Kubernetes is an **orchestration** tool that manages **containerized applications**. What does that mean....?

(I broke down the definition of Kubernetes so that I - a noob - can understand it. It's very high level. I'll link blog posts down the road getting more in depth about things like *Monolith vs Microservice* and *Containers*)

## Application
*Applications are split up into smaller components.*

In the past, applications used to be monolithic architecture. *Huge* pieces of code that contained all the logic to run your application. Take Amazon online shopping for example: Imagine one repository of code managing the following:

1. User Account (account creation, login, forgot password, etc)
2. Search engine (search algorithm, sorting algorithm)
3. Sessions (shopping cart, recommendations)
4. Payment (custom payment or integration with Stripe)

However, successful applications grow over time. More users -> means more money -> Means more developers. This is where a shift from a Monolithic architecture to microservices makes sense. Microservices let you easily **Scale Up**. Microserices will break the 4 previously mentioned functions into separate components. We can now call each function a micro-application. 

## Containerized
*Each application component is served on its own container.*

Back to the Amazon online shopping example. If the shopping experience is a whole application, then managing user accounts is the micro-application. As a developer, you need to write individual functions for when a user creates an account, log in, uploads a profile pic, and so muh more. All those functions will interact with a database to store, reference, modify, and delete entries. Boom. you've got an application.

In order to run the application, you need to host it on a server. But User Accounts is only one application. There's so many more. That's costs a lot of money to buy a brand new server for all 6+ applications you're going to write (shopping-cart, payment, checkout, comments etc). That's why you could use containers. A container is like a server, but gutted of everything that you don't need to run the application. No useless libraries, no GUIs, no operating systems (kind of).

Now I've convinced myself containers is the way to go, time to set it up. Let's say 10 containers for your microapplications. Simple right? Not really...

## Orchestration
*Managing all the containers by hand is hard, that's why we use orchestration*

Sure, setting up containers is easier than setting up servers. Servers means physical installation, getting an operatin system, creating user accounts, downloading apps. It's a lot of work. Containers are easier.

But containers still require some elbow grease. Especially for 10 containers. Using Docker to create containers, I still have to type in a couple lines of code
```
docker pull <image name> # Download the application
docker run <image_name>  # Start the application in a container
docker ps -a             # Check that your container is up and running
```
Do that 10 times and you'll wish "Gee, I wish there was a way to automate that work." Well there is. It's using an orchestration like Kubernetes. *Role Credits.*

# 💤 TL;DR
Kubernetes **orchestrates containerized applications**.

`Lots of pieces of code + Placed across differnt machines + Conductor managing the machines.`

Think literal conductor, like an orchestra.

`Lots of notes on sheet of music + Different musicians "executing" the music + Conductor managing all the musicians.`

In the end, Kubernetes helps your scale your product. If you need to add/stop/change ~~musicians~~ I mean containers, Kubernetes lets you do that easily and all at once.

# 📚 Resources
- [What is Kubernetes | Kubernetes explained in 15 mins ](https://youtu.be/VnvRFRk_51k?t=87) <- I didn't watch the full video. Literally stopped at this one frame.
- [Moving from Monoliths to Microservices 🎂 → 🍰🍰🍰 ](https://www.youtube.com/watch?v=rckfN7xFig0)
