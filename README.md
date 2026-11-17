# CST8915 Lab 6 - Deploying the Algonquin Pet Store to AKS

**Student:** David Yin

## Demo Video

[Watch the Lab 6 AKS demo on YouTube](https://youtu.be/PZ5BEmxLMFI)

## Deployment

I deployed the Algonquin Pet Store to Azure Kubernetes Service (AKS) in **South Central US**. Both the cluster and its resource group were named `AlgonquinPetStoreRG`. The cluster used the AKS Free tier and Kubernetes `1.36.5`.

| Node pool | Mode | Nodes | VM size |
| --- | --- | ---: | --- |
| `agentpool` | System | 1 | `Standard_D2as_v6` |
| `workerspool` | User | 1 | `Standard_D2as_v6` |

I used the [lab's Kubernetes manifest](https://github.com/ramymohamed10/26F_CST8915_Lab6/blob/main/algonquin-pet-store-all-in-one.yaml) and its prebuilt images. The manifest defines a Deployment and a Service for each of Store Front, Order Service, Product Service, and RabbitMQ.

## How the Services Share One Public IP

The `store-front` Service uses `LoadBalancer` to expose HTTP port 80. Nginx in the frontend container serves the Vue app and forwards requests to the backend `ClusterIP` Services by path. Kubernetes DNS resolves the internal Service names, and the Vue app calls `/products` and `/orders` on the same origin as the storefront.

| Public path | Destination |
| --- | --- |
| `/` | Vue storefront served by Nginx |
| `/products` | `http://product-service:3030/products` |
| `/orders` | `http://order-service:3000/orders` |
| `/rabbitmq/` | RabbitMQ management UI at `rabbitmq:15672`, with the `/rabbitmq` prefix removed |

See the [Lab 6 frontend Nginx configuration](https://github.com/ramymohamed10/store-front-L6/blob/main/nginx.conf).

## Verify the Deployment

- Use `kubectl get nodes`, `kubectl get pods -o wide`, and `kubectl get services` to check the nodes, Pod placement, and Store Front external IP.
- Open `/products` to check the catalog. Place an order from the storefront and check `order_queue` in RabbitMQ to verify that the order reached the broker.
- Scale `deployment/store-front` from one replica to three. Check the three Pods and reload the storefront using the same external IP.
- Watch the frontend Pods while deleting one. The ReplicaSet creates a replacement to restore the desired count. Check the frontend logs with `kubectl logs deployment/store-front --tail=20` and scale back to one replica.

[Lab 6 instructions and submission requirements](https://github.com/ramymohamed10/26F_CST8915_Lab6#submission) · [kubectl cheat sheet](https://github.com/ramymohamed10/26F_CST8915_Lab6/blob/main/Kubectl-Cheat-Sheet.md)
