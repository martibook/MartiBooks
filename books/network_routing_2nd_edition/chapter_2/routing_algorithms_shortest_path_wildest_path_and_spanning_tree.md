

# Bellman–Ford Algorithm and the Distance Vector Approach

## Centralized View: Bellman–Ford Algorithm

![Alt text](image.png)


The following equations, known as Bellman's equations
![Alt text](image-1.png)

![Alt text](image-2.png)


###  Bellman–Ford centralized algorithm
![Alt text](image-3.png)


## Distributed View: A Distance Vector Approach

This view of the centralized Bellman–Ford algorithm is not directly suitable for a distributed environment
centralized 方式中 i 知道所有信息，以 i 为中心，慢慢发现
![Alt text](image-5.png)

 change the order of consideration for **distributed environment**
![Alt text](image-4.png)


### Distance vector algorithm (computed at node i)
![Alt text](image-6.png)