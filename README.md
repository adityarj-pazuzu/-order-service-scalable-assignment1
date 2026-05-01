# Order Service

The Order Service manages customer orders. It calls the Catalog Service over HTTP to check product information and reserve stock before saving an order.

## Responsibilities

- Create orders.
- List orders.
- Read order details.
- Collaborate with Catalog Service.

## Run Locally

To test the Order Service with the Catalog Service, Clone the https://github.com/adityarj-pazuzu/catalog-service-scalable-assignment1

Without the Catalog Service, you can still test the Order Service's endpoints, but order creation will fail due to the inability to check product information and reserve stock.
What can work without the Catalog Service:
- Health check endpoint will work.
- List orders endpoint will work (will return empty list if no orders created).
- Get order details endpoint will work for existing orders, but creating new orders will fail.

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
$env:CATALOG_SERVICE_URL="http://localhost:8001" # Required if testing with Catalog Service running on localhost
uvicorn app.main:app --host 0.0.0.0 --port 8002
```

## Run Tests

```powershell
pytest
```

## Pre-commit

This repository has its own pre-commit configuration in `.pre-commit-config.yaml`.

```powershell
pip install pre-commit
pre-commit install
pre-commit run --all-files
```

## API Endpoints

| Method | Endpoint | Description |
| --- | --- | --- |
| GET | `/health` | Health check |
| POST | `/orders` | Create order |
| GET | `/orders` | List orders |
| GET | `/orders/{order_id}` | Get order |

## Testing the API

### Prerequisites

Ensure the Catalog Service is running on `http://localhost:8001` before creating orders.

### Health Check

**PowerShell:**

```powershell
Invoke-RestMethod http://localhost:8002/health
```

**Bash/Git Bash:**

```bash
curl http://localhost:8002/health
```

### Create an Order

Before creating an order, ensure a product exists in the Catalog Service (see the Catalog Service README for how to create products).

**PowerShell:**

```powershell
Invoke-RestMethod -Method Post -Uri http://localhost:8002/orders -ContentType "application/json" -Body '{"customer_name":"Aditi","items":[{"product_id":1,"quantity":2}]}'
```

**Bash/Git Bash:**

```bash
curl -X POST http://localhost:8002/orders \
  -H "Content-Type: application/json" \
  -d '{"customer_name":"Aditi","items":[{"product_id":1,"quantity":2}]}'
```

### List Orders

**PowerShell:**

```powershell
Invoke-RestMethod http://localhost:8002/orders
```

**Bash/Git Bash:**

```bash
curl http://localhost:8002/orders
```

### Get Order Details

**PowerShell:**

```powershell
Invoke-RestMethod http://localhost:8002/orders/1
```

**Bash/Git Bash:**

```bash
curl http://localhost:8002/orders/1
```

## Docker

Build the image:

```powershell
docker build -t order-service:1.0 .
```

Run the container. The Catalog Service must be reachable through the `CATALOG_SERVICE_URL` environment variable:

```powershell
docker network create store-network
docker run -d --name order-service --network store-network -p 8002:8000 -e CATALOG_SERVICE_URL=http://catalog-service:8000 order-service:1.0
```

## Kubernetes

This repository owns its own Kubernetes manifests in the `k8s` folder.

The Catalog Service should be deployed first because this service calls it using the Kubernetes service name `catalog-service`.

For Minikube:

```powershell
minikube start
minikube docker-env | Invoke-Expression
docker build -t order-service:1.0 .
kubectl apply -f .\k8s\namespace.yaml
kubectl apply -f .\k8s\deployment.yaml
kubectl get all -n store-app
```

Access the service:

```powershell
minikube service order-service -n store-app
```

### Kubernetes Dashboard

Enable and open the dashboard:

```powershell
minikube dashboard
```

In the dashboard:

- Select the namespace `store-app` then check the resources:
  - Deployments: `order-service`
  - Pods: running status and restart count
  - Services: NodePort service exposure
  - Logs: request handling and any service communication errors

## GitHub Actions

This repository has its own workflow:

```text
.github/workflows/ci.yml
```

The workflow runs pre-commit checks, runs tests, and builds the Docker image.

The current workflow does not need credentials because it does not push images or deploy to a cloud cluster. If Docker image push is added later, add these secrets in GitHub repository `Settings` -> `Secrets and variables` -> `Actions`:

- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`

Never hardcode passwords, tokens, or kubeconfig values in the workflow file.
