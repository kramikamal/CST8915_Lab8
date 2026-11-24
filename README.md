# CST8915 Lab 8: Deploying and Managing the Algonquin Pet Store

- **Student Name**: Kamal Krami
- **Student ID**: 041273436
- **Course**: CST8915 Full-stack Cloud-native Development
- **Semester**: Fall 2026

## Demo Video

🎥 [Watch Demo Video](https://youtu.be/bCPC8JSwNDk)

## What I Did in This Lab

In this lab, I deployed the Algonquin Pet Store to an AKS cluster with two worker nodes.

- I applied the ConfigMap and the Secrets for the OpenAI keys.
- I deployed the application with `aps-all-in-one.yaml`, and the virtual customer and worker with `admin-tasks.yaml`.
- I tested the Store Front and the Store Admin.
- I scaled the `order-service` to 3 replicas and used `kubectl top` to see the CPU and memory.
- I added a new product with the AI Assistant, which created a description and an image.
- I checked RabbitMQ with `kubectl port-forward` and MongoDB with `mongosh`.

## Task 1: My Own Docker Images

I imported the eight service repositories into my GitHub account, built the Docker images, and pushed them to my Docker Hub account.

| Service | My GitHub Repository | My Docker Image |
| --------- | --------------------- | ----------------- |
| store-front | [store-front-L8](https://github.com/kramikamal/store-front-L8) | [kamalkrami/store-front:lab8](https://hub.docker.com/r/kamalkrami/store-front) |
| store-admin | [store-admin-L8](https://github.com/kramikamal/store-admin-L8) | [kamalkrami/store-admin:lab8](https://hub.docker.com/r/kamalkrami/store-admin) |
| order-service | [order-service-L8](https://github.com/kramikamal/order-service-L8) | [kamalkrami/order-service:lab8](https://hub.docker.com/r/kamalkrami/order-service) |
| product-service | [product-service-L8](https://github.com/kramikamal/product-service-L8) | [kamalkrami/product-service:lab8](https://hub.docker.com/r/kamalkrami/product-service) |
| makeline-service | [makeline-service-L8](https://github.com/kramikamal/makeline-service-L8) | [kamalkrami/makeline-service:lab8](https://hub.docker.com/r/kamalkrami/makeline-service) |
| ai-service | [ai-service-L8](https://github.com/kramikamal/ai-service-L8) | [kamalkrami/ai-service:lab8](https://hub.docker.com/r/kamalkrami/ai-service) |
| virtual-customer | [virtual-customer-L8](https://github.com/kramikamal/virtual-customer-L8) | [kamalkrami/virtual-customer:lab8](https://hub.docker.com/r/kamalkrami/virtual-customer) |
| virtual-worker | [virtual-worker-L8](https://github.com/kramikamal/virtual-worker-L8) | [kamalkrami/virtual-worker:lab8](https://hub.docker.com/r/kamalkrami/virtual-worker) |

My updated files are in the `Deployment Files` folder:

- [`aps-all-in-one-Task1.yaml`](./Deployment%20Files/aps-all-in-one-Task1.yaml): the six application services use my images.
- [`admin-tasks.yaml`](./Deployment%20Files/admin-tasks.yaml): the virtual customer and the virtual worker use my images.

MongoDB and RabbitMQ still use the official images.

> **Note:** The `secrets.yaml` file is not in this repository, because it has the OpenAI keys.

## Task 2: Azure Managed Services

In this lab, MongoDB and RabbitMQ run inside my cluster, so I need to take care of them myself. Azure has managed services that can replace them.

### MongoDB → Azure Cosmos DB for MongoDB

**What it is:** [Azure Cosmos DB for MongoDB](https://learn.microsoft.com/azure/cosmos-db/mongodb) is a managed database in Azure that works like MongoDB, so the application can use the same MongoDB drivers and tools.

**Why it is a good fit:** There is no doubt that it makes the work easier. First, Azure takes care of the servers, the updates and the backups. Second, the data is saved outside the cluster, so it is safe even if a Pod or the cluster has a problem. Third, the application does not need big changes, because it works like MongoDB.

### RabbitMQ → Azure Service Bus

**What it is:** [Azure Service Bus](https://learn.microsoft.com/azure/service-bus-messaging/service-bus-messaging-overview) is a managed message broker in Azure. It has queues, like RabbitMQ, so the order-service can send orders and the makeline-service can read them later.

**Why it is a good fit:** I strongly believe that it solves the problem I saw in Lab 7, when my orders were lost after I deleted the RabbitMQ Pod. With Service Bus, the messages are saved safely by Azure, and we do not need to manage the broker ourselves.

Overall, given all these points, it is clear that these two managed services would make the Pet Store more reliable and easier to manage.

## What I Noticed

- The MongoDB and RabbitMQ Pods had fixed names (`mongodb-0` and `rabbitmq-0`), because they are StatefulSets.
- In the Store Admin, the order list kept changing. The virtual customer adds new orders, and the virtual worker completes them, so they leave the list.
- When I refreshed the Store Admin on the `/orders` page, I got a `not found` error. Using the menu instead of refreshing worked fine.

## Challenges and Learnings

My first challenge was the VM size. The size in the instructions was not available for my Azure for Students subscription, so I used `Standard_B2s_v2` in Sweden Central.

In this lab, I learned how ConfigMaps and Secrets keep the settings out of the code, why databases and queues use StatefulSets, and how to use my own Docker images in Kubernetes.

## Sources

- [Microsoft Learn: What is Azure Cosmos DB for MongoDB?](https://learn.microsoft.com/azure/cosmos-db/mongodb)
- [Microsoft Learn: What is Azure Service Bus?](https://learn.microsoft.com/azure/service-bus-messaging/service-bus-messaging-overview)