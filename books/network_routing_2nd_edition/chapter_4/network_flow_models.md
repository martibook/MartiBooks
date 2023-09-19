[Single-Commodity Network Flow](#single-commodity-network-flow)

[Minimum Cost Routing Objective](#minimum-cost-routing-objective)

[Load Balancing](#load-balancing)

[Average Delay](#average-delay)



Routing Efficiency -> Traffic Engineering

traffic volume - traffic network - IP/telephone network - traffic routing
demand volume - transport network - DS3-cross-connect, SONET, WDM network where circuits are deployed - transport routing/ circuit routing/ demand routing

In this chapter, we will uniformly use the term demand volume, 
without attaching a particular measurement unit or a network type
since our goal here is to present the basic concepts of network flow models.


 Any amount of demand volume that uses or is carried on a path is referred to as **flow**; 
 this is also referred to as **path flow**, 
 or **flowing demand volume** on a path, 
 or even **routing demand volume** on a path. 


## Single-Commodity Network Flow


three node network
![three node network](image.png)


actual routing decision should depend on the goal of routing, irrespective of the hop count. 
This means that we need a generic way to represent the problem so that various situations can be addressed in a capacitated network in order to find the best solution.

capacity: c
demand volume: h
demand volume on path 1-2: x12
demand volume on path 1-3-2: x132

![Alt text](image-1.png)

![Alt text](image-6.png)

![Alt text](image-5.png)

(4.2.1a), (4.2.1b), and (4.2.1c) is referred to as constraints of the problem

![Alt text](image-7.png)


### Minimum Cost Routing Objective


non-negative cost per unit of flow on each path

![Alt text](image-8.png)

The total cost is referred to as the objective function. In general, the objective function will be denoted by F. If the goal is to minimize the total cost of routing, we can write the complete problem as follows:

![description of the problem](image-9.png)


The problem presented in Eq. (4.2.3) is a **single-commodity network flow** problem; 
it is also referred to as a **linear programming problem** 
since the requirements  are all linear, 
and the goal is also linear.
formulation of an **optimization problem**


For clarity, the optimal solution to a problem such as Eq. (4.2.3) will be denoted with asterisks in the superscript, for example, 
![Alt text](image-10.png) and ![Alt text](image-11.png).



![Alt text](image-13.png)

if ![Alt text](image-14.png), then ![Alt text](image-15.png); similarly, 

if ![Alt text](image-16.png), then the minimum is observed when ![Alt text](image-17.png).


### Load Balancing


We now consider another goal — minimization of maximum link utilization, 
or the **minimax problem**. 
This goal is also referred to as **load balancing** flows in the network, 
or **congestion minimization** in the network. Perhaps, 
it is best to refer to this objective as the **Wozencraft objective** since he first suggested this objective [880]


The link utilization is defined as the amount of flow on a link divided by the capacity on that link. 

Then, the maximum utilization over all links means the maximum over these two expressions, i.e.

![Alt text](image-18.png)

![Alt text](image-19.png)

Thus, we see that when the load balancing of flows is the main goal, the optimal solution for Eq. (4.2.6) is to **split the flows equally on both paths**. Certainly, this result holds true as long as the demand volume h is up to and including 2c; the problem becomes infeasible when ![Alt text](image-21.png).

![Alt text](image-20.png)



Variation in Capacity

![Alt text](image-22.png)

![Alt text](image-23.png)



### Average Delay

Another goal commonly defined, especially in data networks, is the minimization of the **average packet delay**. 

For this illustration, we consider again the three-node network with demand volume h between node 1 and node 2; the capacity of all links is set to c. 

The average delay can be captured through the expression
(why?)

![Alt text](image-24.png)

![Alt text](image-25.png)

![Alt text](image-26.png)

This is a nonlinear function that we want to minimize. We can use calculus to solve this problem. 

![Alt text](image-27.png)