# 🛒 ShopiTry E-Commerce Application (AWS Microservices)

A **production-ready, scalable e-commerce platform** built with the MERN stack and deployed on AWS using a Microservices architecture. This repository demonstrates a complete separation of concerns, with backend services running on dedicated EC2 instances and serverless functions handling specific tasks.


**Repository:** [https://github.com/Vickybarai/shopitry-e-commerce](https://github.com/Vickybarai/shopitry-e-commerce)

---

## 📚 Table of Contents

-   [🏗️ Architecture Overview](#-architecture-overview)
-   [🛠 Tech Stack](#-tech-stack)
-   [📂 Project File Structure](#-project-file-structure)
-   [🚀 Deployment Guide (Step-by-Step)](#-deployment-guide-step-by-step)
    -   [Phase 1: MongoDB Atlas Configuration](#phase-1-mongodb-atlas-configuration)
    -   [Phase 2: AWS S3 Setup (Frontend Hosting)](#phase-2-aws-s3-setup-frontend-hosting)
    -   [Phase 3: AWS Lambda & API Gateway Setup](#phase-3-aws-lambda--api-gateway-setup)
    -   [Phase 4: EC2 Command Center (Orchestration)](#phase-4-ec2-command-center-orchestration)
    -   [Phase 5: Deploying Backend Microservices (EC2)](#phase-5-deploying-backend-microservices-ec2)
    -   [Phase 6: Deploying Frontend (S3)](#phase-6-deploying-frontend-s3)
    -   [Phase 7: CloudFront & Route 53 (Custom Domain)](#phase-7-cloudfront--route-53-custom-domain)
-   [🔧 Environment Variables](#-environment-variables)
-   [🧪 Verification & Testing](#-verification--testing)
-   [🐛 Troubleshooting](#-troubleshooting)

---

## 🏗️ Architecture Overview

This application uses a **Gateway Pattern** where all frontend traffic hits the Gateway Service, which then routes requests to specific backend services.

```mermaid
flowchart LR
    A[Users] --> B[CloudFront CDN]
    B --> C[S3 Storefront<br/>store.shopitry.com]
    B --> D[S3 Admin Dashboard<br/>admin.shopitry.com]
    B --> E[API Gateway<br/>api.shopitry.com]
    
    E --> F[Gateway Service<br/>EC2 Port 5000]
    
    F --> G[Catalog Service<br/>EC2 Port 5001]
    F --> H[Cart Service<br/>EC2 Port 5002]
    F --> I[Order Service<br/>EC2 Port 5003]
    
    I --> J[Payment Service<br/>Lambda]
    I --> K[Notification Service<br/>Lambda]
    
    G --> L[(MongoDB Atlas<br/>catalog_db)]
    H --> M[(MongoDB Atlas<br/>cart_db)]
    I --> N[(MongoDB Atlas<br/>orders_db)]
```

---

## 🛠 Tech Stack

-   **Frontend:** React.js, Redux, Tailwind CSS.
-   **Backend:** Node.js, Express.js.
-   **Database:** MongoDB Atlas (Cluster: `edublitz`).
-   **Infrastructure (AWS):**
    -   **EC2:** Virtual Machines for microservices.
    -   **Lambda:** Serverless functions (Payment, Notification).
    -   **S3:** Static web hosting.
    -   **API Gateway:** HTTP endpoint for Lambda.
    -   **CloudFront:** Content Delivery Network.
    -   **Route 53:** DNS Management.

---

## 📂 Project File Structure

```text
shopitry-e-commerce/
├── backend/
│   ├── gateway-service/      # Entry point (Port 5000)
│   ├── catalog-service/      # Products (Port 5001)
│   ├── cart-service/         # Cart Logic (Port 5002)
│   ├── order-service/        # Order Processing (Port 5003)
│   ├── payment-service/      # Lambda Code
│   └── notification-service/ # Lambda Code
├── frontend/
│   ├── store-front/          # Customer App
│   └── admin-dashboard/      # Admin App
└── README.md
```

---

## 🚀 Deployment Guide (Step-by-Step)

### 🎯 Quick Reference Table

| Phase | What to Change | Example Value | Why |
| :--- | :--- | :--- | :--- |
| **MongoDB** | IP Access List | `54.123.45.67` | Allows EC2 to connect |
| **EC2 Security** | Inbound Ports | `5000-5003` | Allows traffic between services |
| **Lambda** | IAM Role ARN | `arn:aws:iam::...` | Grants execution permissions |
| **API Gateway** | API ID & Stage | `abc123` & `prod` | Creates HTTP endpoint |
| **Backend .env** | Service URLs | `http://10.0.1.2:5001` | Internal routing config |
| **CloudFront** | ACM Cert ARN | `arn:aws:acm:us-east-1:...` | Enables HTTPS |

---

### 🔄 Phase 1: MongoDB Atlas Configuration

1.  **Create Cluster:**
    *   Go to [MongoDB Atlas](https://www.mongodb.com/atlas).
    *   Create a Free Cluster named `edublitz` (Region: Mumbai `ap-south-1`).
2.  **Create Database User:**
    *   Go to **Database Access** → **Add New Database User**.
    *   Username: `linux`
    *   Password: `redhat@123` (Note this down).
    *   Privileges: Read and write to any database.
3.  **Whitelist IP Addresses:**
    *   Go to **Network Access** → **Add IP Address**.
    *   Add your **EC2 Public IP** (for security) or `0.0.0.0/0` (for development ease).
    *   *Note:* If you don't whitelist the IP, you will get `MongoNetworkError`.

---

### 🌐 Phase 2: AWS S3 Setup (Frontend Hosting)

We need two buckets to host our React applications.

1.  **Create Buckets:**
    *   Go to AWS S3 → **Create Bucket**.
    *   **Bucket 1:** `store.shopitry.com` (or your desired name).
    *   **Bucket 2:** `admin.shopitry.com` (or your desired name).
    *   **Region:** `ap-south-1`.
2.  **Enable Static Hosting:**
    *   For each bucket, go to **Properties** → **Static website hosting** → **Enable**.
    *   Set "Index document" to `index.html`.
3.  **Permissions:**
    *   Go to **Permissions** → **Block public access (bucket settings)** → **Edit** → **Uncheck** all blocks.
    *   Add a **Bucket Policy** to allow public read access (S3 will provide a template).

---

### ⚙️ Phase 3: AWS Lambda & API Gateway Setup

We deploy Payment and Notification services as serverless functions.

#### A. IAM Role Creation
1.  Go to **IAM Console** → **Roles** → **Create Role**.
2.  Select **AWS Service** → **Lambda**.
3.  Attach Policy: `AWSLambdaBasicExecutionRole`.
4.  Name: `ShopiTryLambdaRole` → Create.
5.  Copy the **Role ARN** (e.g., `arn:aws:iam::123456789012:role/ShopiTryLambdaRole`).

#### B. Lambda Deployment
1.  **Prepare Code:** Zip the contents of `backend/payment-service` and `backend/notification-service`.
2.  **Create Function:**
    *   Go to AWS Lambda → **Create Function**.
    *   Runtime: **Node.js 18.x**.
    *   Upload `.zip` file.
    *   **Configuration:** Update the **Role** with the ARN created in Step A.
    *   **Timeout:** Set to **5 minutes** (Default 3s is too low).
    *   **Environment Variables:** Add `MONGODB_URI` if required.

#### C. API Gateway (The "Glue")
1.  **Create API:**
    *   Go to **API Gateway** → **Create API** → **HTTP API**.
    *   Name: `ShopiTry-Payment-API`.
2.  **Create Routes:**
    *   Create a route `/payments/process` -> Integration type: **Lambda function** -> Select your Payment function.
    *   Repeat for Notification service route.
3.  **Deploy API:**
    *   Go to **Stages** → **Create Stage** → Name: `prod`.
    *   **Copy the Invoke URL** (e.g., `https://abc123.execute-api.ap-south-1.on.aws/prod`).
    *   *Note:* You will need these URLs for the Order Service `.env` file.

---

### 🖥️ Phase 4: EC2 Command Center (Orchestration)

We create a "Command Center" VM to orchestrate deployments.

1.  **Launch Instance:**
    *   Name: `Command Center`.
    *   OS: Ubuntu Server.
    *   Instance Type: `t2.medium`.
2.  **IAM Role:** Attach `AdministratorAccess` to this instance.
3.  **Install Tools (SSH into instance):**
    ```bash
    # Update OS
    sudo apt update && sudo apt upgrade -y

    # Install Node.js (NVM)
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
    source ~/.bashrc
    nvm install --lts

    # Install PM2 (Process Manager)
    sudo npm install -g pm2

    # Install AWS CLI
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    sudo apt install unzip
    unzip awscliv2.zip
    sudo ./aws/install
    ```

---

### 🧩 Phase 5: Deploying Backend Microservices (EC2)

We will launch 4 separate instances for the backend services.

#### 1. Launch 4 EC2 Instances
*   Name them: `gateway-service`, `catalog-service`, `cart-service`, `order-service`.
*   **Security Group:** Ensure **Inbound Rules** allow **TCP ports 5000, 5001, 5002, 5003** from `0.0.0.0/0`.

#### 2. Deploy Cart Service (Port 5002)
SSH into `cart-service`:
```bash
git clone https://github.com/Vickybarai/shopitry-e-commerce.git
cd shopitry-e-commerce/backend/cart-service

# Install Dependencies
npm install

# Configure .env
nano .env
```
*   **Paste this into `.env`:**
    ```env
    PORT=5002
    MONGODB_URI=mongodb+srv://linux:redhat@123@edublitz.laegfsa.mongodb.net/cart_db?retryWrites=true&w=majority
    ```
*   **Start Service:**
    ```bash
    pm2 start src/index.js --name cart-service
    ```

#### 3. Deploy Order Service (Port 5003)
SSH into `order-service`:
```bash
git clone https://github.com/Vickybarai/shopitry-e-commerce.git
cd shopitry-e-commerce/backend/order-service
npm install
nano .env
```
*   **Paste this into `.env`:**
    ```env
    PORT=5003
    MONGODB_URI=mongodb+srv://linux:redhat@123@edublitz.laegfsa.mongodb.net/orders_db?retryWrites=true&w=majority
    PAYMENT_SERVICE_URL=https://<YOUR_API_GATEWAY_ID>.execute-api.ap-south-1.on.aws/prod/payments/process
    NOTIFICATION_SERVICE_URL=https://<YOUR_API_GATEWAY_ID>.execute-api.ap-south-1.on.aws/prod/notifications/send
    CART_SERVICE_URL=http://<CART_VM_PRIVATE_IP>:5002
    ```
*   **Start Service:**
    ```bash
    pm2 start src/index.js --name order-service
    ```

#### 4. Deploy Catalog Service (Port 5001)
SSH into `catalog-service`:
```bash
git clone https://github.com/Vickybarai/shopitry-e-commerce.git
cd shopitry-e-commerce/backend/catalog-service
npm install
nano .env
```
*   **Paste this into `.env`:**
    ```env
    PORT=5001
    MONGODB_URI=mongodb+srv://linux:redhat@123@edublitz.laegfsa.mongodb.net/catalog_db?retryWrites=true&w=majority
    ```
*   **Start Service:**
    ```bash
    pm2 start src/index.js --name catalog-service
    ```

#### 5. Deploy Gateway Service (Port 5000)
SSH into `gateway-service`:
```bash
git clone https://github.com/Vickybarai/shopitry-e-commerce.git
cd shopitry-e-commerce/backend/gateway-service
npm install
nano .env
```
*   **Paste this into `.env`:**
    ```env
    PORT=5000
    JWT_SECRET=a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6

    # Use Private IPs if services are on different VMs, else localhost
    CATALOG_SERVICE_URL=http://<CATALOG_PRIVATE_IP>:5001
    CART_SERVICE_URL=http://<CART_PRIVATE_IP>:5002
    ORDER_SERVICE_URL=http://<ORDER_PRIVATE_IP>:5003

    PAYMENT_SERVICE_URL=https://<YOUR_API_GATEWAY_ID>.execute-api.ap-south-1.on.aws/prod/payments/process
    NOTIFICATION_SERVICE_URL=https://<YOUR_API_GATEWAY_ID>.execute-api.ap-south-1.on.aws/prod/notifications/send
    ```
*   **Start Service:**
    ```bash
    pm2 start src/index.js --name gateway-service
    ```

---

### 📦 Phase 6: Deploying Frontend (S3)

We will build the React apps and upload them to the S3 buckets created in Phase 2.

#### 1. Store Front Deployment
*   **Update API Config:** Open `frontend/store-front/src/constants.js` (or similar). Change the `API_URL` to your **Gateway Public IP:5000**.
*   **Build:**
    ```bash
    cd frontend/store-front
    npm install
    npm run build
    ```
*   **Upload:**
    *   Go to S3 Console → Select `store.shopitry.com` bucket.
    *   Upload contents of the `build/` folder.
    *   Open the **S3 Website Endpoint** URL to test.

#### 2. Admin Dashboard Deployment
*   **Update API Config:** Change `API_URL` to Gateway IP.
*   **Build:**
    ```bash
    cd frontend/admin-dashboard
    npm install
    npm run build
    ```
*   **Upload:**
    *   Upload `build/` contents to `admin.shopitry.com` bucket.

---

### 🚀 Phase 7: CloudFront & Route 53 (Custom Domain)

This phase connects your domains to the S3 buckets.

#### 1. SSL/HTTPS (ACM Certificate)
1.  Go to **AWS Certificate Manager (ACM)**.
2.  **Crucial:** Switch region to **US East (N. Virginia)** `us-east-1`.
3.  Request a public certificate for `*.shopitry.com` (or your specific subdomains).
4.  Validate via DNS (Route 53).

#### 2. CloudFront Distribution
1.  Create **Distribution**.
2.  **Origin Settings:**
    *   Origin Domain: Select your S3 bucket (e.g., `store.shopitry.com`).
3.  **Settings:**
    *   Viewer Protocol Policy: **Redirect HTTP to HTTPS**.
4.  **Alternate Domain Names (CNAMEs):** Add `store.shopitry.com`.
5.  **Custom SSL Certificate:** Select the cert created in Step 1 (from `us-east-1`).
6.  Create Distribution.

#### 3. Route 53 DNS
1.  Go to **Route 53** → **Hosted Zones** → `shopitry.com`.
2.  **Create Record**:
    *   Record Name: `store`
    *   Type: `A`
    *   Alias: Yes.
    *   Route traffic to: **Alias to CloudFront distribution**.
    *   Select the distribution created in Step 2.

---

## 🔧 Environment Variables

**Database Credentials (Used in Microservices):**
*   **User:** `linux`
*   **Password:** `redhat@123`
*   **Cluster:** `edublitz`

**Ports:**
*   Gateway: `5000`
*   Catalog: `5001`
*   Cart: `5002`
*   Order: `5003`

---

## 🧪 Verification & Testing

1.  **Frontend Access:** Open `http://store.shopitry.com`. You should see products (Catalog Service working).
2.  **Cart Test:** Add a product to cart. Check `cart_db` in MongoDB.
3.  **Order Test:** Proceed to checkout.
    *   Verify `orders_db` has an entry.
    *   Check CloudWatch Logs for Lambda execution.
4.  **Health Check:**
    ```bash
    curl http://<GATEWAY_IP>:5000/health
    ```

---

## 🐛 Troubleshooting

| Error | Cause | Solution |
| :--- | :--- | :--- |
| **MongoNetworkError** | IP not whitelisted. | Add EC2 Public IP to MongoDB Network Access. |
| **Connection Refused** | EC2 Security Group. | Open Ports 5000-5003 in Security Group. |
| **Lambda Timeout** | Default is 3 seconds. | Increase timeout to 5 min in Configuration. |
| **CloudFront 502** | S3 Bucket not public. | Check S3 Bucket Policy (Block Public Access must be OFF). |
| **Frontend Blank Screen** | Wrong API URL. | Update `API_URL` in frontend code to Gateway IP. |

---

## 💡 Interview Notes

**Key Talking Points:**
1.  **Microservices:** "I decoupled the application into distinct services (Catalog, Cart, Order) for scalability and independent deployment."
2.  **Serverless:** "I used AWS Lambda for Payment and Notifications to optimize costs for sporadic event-driven tasks."
3.  **Gateway Pattern:** "I implemented an API Gateway to abstract backend complexity and manage routing logic centrally."
4.  **MongoDB:** "I chose MongoDB for its flexible document schema, ideal for e-commerce products with varying attributes."

---

## 👤 Author

**Vickybarai**
- GitHub: [Vickybarai](https://github.com/Vickybarai)