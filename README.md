# 🛍️ ShopiTry E-Commerce Application

A complete, production-grade MERN stack E-commerce platform designed to teach modern cloud architecture and microservices deployment. This project demonstrates how to scale an application using AWS services, MongoDB Atlas, and a microservices architecture.

## 📂 Repository
[https://github.com/Vickybarai/shopitry-e-commerce.git](https://github.com/Vickybarai/shopitry-e-commerce.git)

---

## 🏗️ Ecosystem Architecture

This application is built using a **Microservices Architecture**. Instead of one giant backend, we separate logic into different services. This makes the application scalable and easier to manage.

```text
                               ┌─────────────────────────────────────────┐
                               │       AWS CloudFront / AWS S3           │
                               │  ┌──────────────────┐ ┌───────────────┐ │
                               │  │ShopiTry Storefront│ │ ShopiTry Admin│ │
                               │  └────────┬─────────┘ └───────┬───────┘ │
                               └───────────┼───────────────────┼─────────┘
                                           │                   │
                                           ▼                   ▼
                               ┌─────────────────────────────────────────┐
                               │     VM Server 1: Gateway Service        │
                               │           (Port 5000 / JWT Auth)        │
                               └───────────────────┬─────────────────────┘
                                                   │
         ┌───────────────────┬─────────────────────┼─────────────────────┬───────────────────┐
         │                   │                     │                     │                   │
         ▼                   ▼                     ▼                     ▼                   ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌───────────────────┐ ┌───────────────────┐
│  VM Server 2    │ │   VM Server 3   │ │   VM Server 4   │ │   AWS Lambda 1    │ │   AWS Lambda 2    │
│ Catalog Service │ │  Cart Service   │ │  Order Service  │ │  Payment Service  │ │Notification Service│
│   (Port 5001)   │ │   (Port 5002)   │ │   (Port 5003)   │ │    (Port 5004)    │ │    (Port 5005)    │
└────────┬────────┘ └────────┬────────┘ └────────┬────────┘ └───────────────────┘ └───────────────────┘
         │                   │                   │
         ▼                   ▼                   ▼
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│                             MongoDB Atlas Cluster: edublitz                                      │
│        (catalog_db)               (cart_db)                    (orders_db)                       │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 🚀 Quick Start Guide (Beginner's Navigation)

We will break down the deployment into **6 Phases**. Follow them in order.

### 📋 Prerequisites
1.  **AWS Account** (Free Tier).
2.  **GitHub Account**.
3.  **MongoDB Atlas Account** (Free Tier).
4.  **VS Code** (or any code editor).
5.  **Git** installed on your local machine.

---

## 🌐 Phase 1: Database Setup (MongoDB Atlas)

We need a cloud database to store our product, cart, and order data.

1.  **Create Cluster:**
    *   Log in to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas).
    *   Click **Build a Database**.
    *   Select **M0 Sandbox (Free)**.
    *   **Cluster Name:** `edublitz`.
    *   Create it.

2.  **Setup Network Access:**
    *   Go to **Network Access** > **Add IP Address**.
    *   For practice, select `Allow Access from Anywhere` (`0.0.0.0/0`).

3.  **Create Database User:**
    *   Go to **Database Access** > **Add New Database User**.
    *   **Username:** `linux`
    *   **Password:** `redhat123` (Choose a secure password and remember it!)
    *   **Database User Privileges:** Read and write to any database.

4.  **Get Connection String:**
    *   Go to **Database** > **Connect** > **Connect your application**.
    *   Select Node.js version.
    *   Copy the connection string. It looks like:
        `mongodb+srv://linux:<password>@edublitz.laegfsa.mongodb.net/?appName=edublitz`
    *   *Note:* You will need to replace `<password>` with your actual password later.

---

## 💻 Phase 2: Command Center Setup (AWS EC2)

We will create a central server to handle our Git operations and frontend builds.

1.  **Launch EC2:**
    *   Go to AWS Console > **EC2** > **Launch Instance**.
    *   **Name:** `Command-Center`.
    *   **OS:** Ubuntu Server.
    *   **Instance Type:** `t2.medium` (Recommended for compiling React apps).
    *   **Key Pair:** Create a new Key Pair (e.g., `my-key`) and download the `.pem` file.

2.  **Configure Security Group:**
    *   Allow **SSH (Port 22)** (My IP).
    *   Allow **All Traffic (0.0.0.0/0)** for learning purposes (so you can access all services easily).

3.  **Connect & Install Tools:**
    *   Connect to the instance via SSH (using MobaXterm or Terminal).
    *   Run these commands one by one:

    ```bash
    # Update system
    sudo apt update

    # Install Node.js (Version 20)
    curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
    sudo apt-get install -y nodejs

    # Install Git
    sudo apt install git

    # Install AWS CLI
    curl -fsSL https://amazonaws.com | bash -s -- --system

    # Install Zip/Unzip
    sudo apt install zip unzip

    # Install PM2 (Process Manager)
    sudo npm install -g pm2
    ```

4.  **Clone Repository:**
    ```bash
    git clone https://github.com/Vickybarai/shopitry-e-commerce.git
    cd shopitry-e-commerce
    ```

---

## ⚡ Phase 3: Serverless Services (AWS Lambda)

We will deploy **Payment** and **Notification** services as AWS Lambda functions. These are serverless, meaning AWS manages the servers for you.

### 1. Payment Service Setup
1.  **Prepare Code:**
    *   Inside your `Command-Center` EC2, navigate to:
        `cd shopitry-e-commerce/backend/payment-service`
    *   **Important:** Check code for hardcoded regions. Open `src/local-server.ts` (or similar). If it says `us-east-1` and you are in Mumbai (`ap-south-1`), change it to `ap-south-1`.
    *   Install dependencies: `npm install`
    *   Zip the contents:
        ```bash
        zip -r payment.zip . -x "*.git*" "node_modules/*"
        ```

2.  **Deploy to Lambda:**
    *   Go to AWS Console > **Lambda** > **Create Function**.
    *   **Name:** `shopitry-payment-service`
    *   **Runtime:** Node.js 20.x.
    *   **Architecture:** x86_64.
    *   Click **Create**.

3.  **Upload Code:**
    *   Go to the **Code** tab > **Upload from** > **.zip file**.
    *   Upload the `payment.zip` file.

4.  **Configuration:**
    *   **Timeout:** Go to **Configuration** > **General configuration** > **Edit**. Set Timeout to **5 min**.
    *   **Environment Variables:** Add any required DB keys if the code uses them.
    *   **Function URL:** Go to **Configuration** > **Function URL** > **Create Function URL**.
        *   Auth type: `NONE`.
        *   Copy the generated URL (e.g., `https://xyz...lambda-url.us-east-1.on.aws/`).

### 2. Notification Service Setup
Repeat the exact steps above for the `backend/notification-service` folder.
*   **Function Name:** `shopitry-notification-service`
*   **SNS Integration:** If you want email notifications, create an SNS Topic in AWS, add your email as a subscriber, and put that Topic ARN in the Lambda code.

---

## 🖥️ Phase 4: Backend Microservices (EC2)

We need 4 separate Virtual Machines (VMs) to run our core backend logic.

### Step A: Launch 4 EC2 Instances
*   Launch **4 Ubuntu** instances (use `t2.micro` or `t2.small`).
*   **Names:** `Gateway-Service`, `Catalog-Service`, `Cart-Service`, `Order-Service`.
*   **Security Group:** Allow **All Traffic** (Ports 5000, 5001, 5002, 5003).
*   **Key Pair:** Use the same key as the Command Center.

### Step B: Deploy Catalog Service (Port 5001)
1.  SSH into the `Catalog-Service` instance.
2.  Clone repo: `git clone https://github.com/Vickybarai/shopitry-e-commerce.git`
3.  Navigate: `cd shopitry-e-commerce/backend/catalog-service`
4.  **Update `.env` file:**
    ```env
    PORT=5001
    MONGO_URI=mongodb+srv://linux:YOUR_PASSWORD@edublitz.laegfsa.mongodb.net/catalog_db?appName=edublitz
    ```
5.  Install & Run:
    ```bash
    npm install
    pm2 start src/index.js --name "catalog-service"
    ```

### Step C: Deploy Cart Service (Port 5002)
1.  SSH into `Cart-Service` instance.
2.  Clone repo and navigate to `backend/cart-service`.
3.  **Update `.env` file:**
    ```env
    PORT=5002
    MONGO_URI=mongodb+srv://linux:YOUR_PASSWORD@edublitz.laegfsa.mongodb.net/cart_db?appName=edublitz
    ```
4.  Install & Run:
    ```bash
    npm install
    pm2 start src/index.js --name "cart-service"
    ```

### Step D: Deploy Order Service (Port 5003)
1.  SSH into `Order-Service` instance.
2.  Clone repo and navigate to `backend/order-service`.
3.  **Update `.env` file:**
    ```env
    PORT=5003
    MONGO_URI=mongodb+srv://linux:YOUR_PASSWORD@edublitz.laegfsa.mongodb.net/orders_db?appName=edublitz
    # Add the Lambda URLs you created in Phase 3
    PAYMENT_SERVICE_URL=https://YOUR_PAYMENT_LAMBDA_URL
    NOTIFICATION_SERVICE_URL=https://YOUR_NOTIFICATION_LAMBDA_URL
    # You also need the Cart Service Private IP here
    CART_SERVICE_URL=http://CART_SERVICE_PRIVATE_IP:5002
    ```
4.  Install & Run:
    ```bash
    npm install
    pm2 start src/index.js --name "order-service"
    ```

### Step E: Deploy Gateway Service (Port 5000)
The Gateway is the "traffic police". It receives all requests and routes them to the correct service.

1.  SSH into `Gateway-Service` instance.
2.  Clone repo and navigate to `backend/gateway-service`.
3.  **Update `.env` file:**
    ```env
    PORT=5000
    # You need the Private IPs of the other EC2 instances here
    CATALOG_SERVICE_URL=http://CATALOG_PRIVATE_IP:5001
    CART_SERVICE_URL=http://CART_PRIVATE_IP:5002
    ORDER_SERVICE_URL=http://ORDER_PRIVATE_IP:5003
    PAYMENT_SERVICE_URL=https://YOUR_PAYMENT_LAMBDA_URL
    NOTIFICATION_SERVICE_URL=https://YOUR_NOTIFICATION_LAMBDA_URL
    ```
4.  Install & Run:
    ```bash
    npm install
    pm2 start src/index.js --name "gateway-service"
    ```

---

## 🎨 Phase 5: Frontend Deployment (S3)

We will deploy the React frontend to AWS S3 (Static Hosting).

### 1. Create S3 Buckets
*   Go to AWS S3 > **Create Bucket**.
*   Create two buckets:
    1.  `shopitry-storefront` (For Customers)
    2.  `shopitry-admin` (For Admins)
*   **ACL Settings:** Enable "Block all public access" -> **OFF** (Click Acknowledge).
*   **Properties:** Scroll down to "Static website hosting", click **Edit**, and **Enable**.

### 2. Build & Deploy Storefront
1.  Go back to your **Command Center** EC2 (or your local laptop).
2.  Navigate to: `shopitry-e-commerce/frontend/storefront`
3.  **Connect API:** Open the code. Look for where the API base URL is defined. Change it to your **Gateway Service Public IP**.
    *   *Note:* In the audio, this was found in a file or configured during the build. Ensure the frontend points to `http://GATEWAY_PUBLIC_IP:5000`.
4.  Build:
    ```bash
    npm install
    npm run build
    ```
5.  Upload to S3:
    ```bash
    aws s3 cp ./dist s3://shopitry-storefront --recursive --acl public-read
    ```

### 3. Build & Deploy Admin Dashboard
Repeat the steps above for the `frontend/admin-dashboard` folder, uploading to the `shopitry-admin` bucket.

---

## 🌍 Phase 6: Accessing Your Application

### Method 1: Direct Practice Mode (No Domain)
This is the easiest way to test immediately after setup.

1.  **Storefront (User):**
    *   Go to your S3 Bucket > **Properties** > **Static website hosting**.
    *   Use the provided **Endpoint** (e.g., `http://bucket-name.s3-website...amazonaws.com`).
2.  **Admin Dashboard:**
    *   Use the Endpoint for the Admin bucket.
3.  **API:**
    *   The Frontend will talk to `http://YOUR_GATEWAY_EC2_PUBLIC_IP:5000`.

### Method 2: Professional Domain Mode (Non-Direct)
To make it look like a real website (e.g., `shopitry.com`), follow these advanced steps:

<details>
<summary><strong>Click to Expand Domain Setup Guide</strong></summary>

1.  **Buy a Domain:** Purchase a domain (e.g., `shopitry.com`) from Route 53 or GoDaddy.
2.  **SSL Certificate (ACM):**
    *   Go to AWS Certificate Manager (ACM).
    *   Request a certificate for `*.shopitry.com`.
    *   **Crucial:** You must request this in the **US East (N. Virginia)** region, even if your servers are in Mumbai.
    *   Validate via DNS.
3.  **CloudFront (CDN):**
    *   Create a CloudFront Distribution.
    *   **Origin Domain:** Enter your S3 bucket endpoint.
    *   **Viewer Protocol Policy:** Redirect HTTP to HTTPS.
    *   **CNAMEs:** Add `store.shopitry.com`.
    *   **Custom SSL Certificate:** Select the ACM certificate you created.
4.  **Route 53 (DNS):**
    *   Go to Route 53 Hosted Zones.
    *   Create an **A Record**:
        *   Name: `store`
        *   Type: `A`
        *   Alias: Yes
        *   Route Traffic to: `CloudFront distribution` (Select your distribution).
5.  **Repeat for Admin:** Do the same for `admin.shopitry.com` pointing to the Admin S3 bucket (or a different CloudFront distro).
6.  **API Domain (Load Balancer):**
    *   In a professional setup, you wouldn't use a direct IP for the Gateway. You would create an **AWS Application Load Balancer (ALB)** in front of your Gateway EC2.
    *   Create an A Record `api.shopitry.com` pointing to the ALB.

</details>

---

## 🧪 Testing the Application

1.  **Open Storefront URL.**
2.  **Admin:** Open Admin URL.
3.  **Create Product:**
    *   Login to Admin.
    *   Add a product (e.g., "Laptop", Price: 50000).
4.  **Buy Product:**
    *   Go to Storefront.
    *   Search for the product.
    *   Add to Cart.
    *   Proceed to Checkout.
5.  **Verify Flow:**
    *   Check MongoDB Atlas -> Collections -> You should see data in `catalog_db`, `cart_db`, and `orders_db`.
    *   Check your email -> You should receive an order confirmation (if Lambda is configured correctly).

---

## ❓ Troubleshooting (Common Errors)

1.  **MongoNetworkError:**
    *   *Cause:* Your EC2 IP is not whitelisted in MongoDB Atlas.
    *   *Fix:* Go to MongoDB Atlas -> Network Access -> Add IP Address -> Paste EC2 Public IP.
2.  **Connection Refused:**
    *   *Cause:* PM2 is not running or Port is blocked.
    *   *Fix:* Run `pm2 logs` inside the EC2 instance. Ensure Security Group allows ports 5000-5005.
3.  **Payment Fails:**
    *   *Cause:* Lambda Timeout is too short or URL is wrong in `.env`.
    *   *Fix:* Set Lambda Timeout to 5 mins. Check Function URL in Order Service `.env`.
4.  **Images Not Loading:**
    *   *Cause:* S3 Bucket is private.
    *   *Fix:* Enable "Static website hosting" and make objects public-read.

---

## 📝 Project Summary for Interviews

*   **Architecture:** Microservices based.
*   **Backend:** Node.js, Express.
*   **Database:** MongoDB Atlas (NoSQL).
*   **Infrastructure:** AWS (EC2 for heavy compute, Lambda for event-driven tasks, S3 for static hosting).
*   **Process Management:** PM2.
*   **Key Achievement:** Successfully deployed a scalable MERN stack application with separation of concerns (Catalog, Cart, Order, Payment).

---
---

<details>
<summary><strong>end</strong></summary>
# 📝 Part 1: LinkedIn Profile Description

**Option A: The "Project Featured" Section (Under your Experience or Projects)**

> **ShopiTry E-Commerce Platform** | *Full Stack Developer & Cloud Architect*
> Developed a scalable, production-grade Microservices-based E-commerce application from scratch.
> *   **Architecture:** Designed a decoupled microservices architecture separating Catalog, Cart, Order, and Payment logic.
> *   **Tech Stack:** MERN (MongoDB, Express, React, Node.js), AWS (EC2, Lambda, S3, API Gateway).
> *   **Infrastructure:** Deployed backend microservices on AWS EC2 instances using PM2 for process management.
> *   **Serverless:** Implemented event-driven Payment and Notification services using AWS Lambda to optimize cost and scalability.
> *   **Database:** Configured MongoDB Atlas with sharded databases (`catalog_db`, `cart_db`, `orders_db`) for high availability.
> *   **CI/CD:** Automated frontend build pipelines and deployed static assets to AWS S3 with CloudFront integration.
> *   **Result:** Achieved low latency inter-service communication and implemented a secure gateway service for centralized authentication and routing.

**Option B: The "About Me" / Summary Section (Short Pitch)**

> Passionate Full Stack Developer with experience in building scalable cloud architectures. Recently built **ShopiTry**, a MERN-stack e-commerce platform utilizing **AWS Microservices (EC2, Lambda)** and **MongoDB Atlas**. Demonstrated expertise in system design, REST API development, and DevOps by deploying a modular system capable of handling independent service scaling.

---

# 🎤 Part 2: Interview Explanation (The STAR Method)

In an interview, don't just list technologies. Tell a story using the **S.T.A.R. method** (Situation, Task, Action, Result).

### 1. The Overview (The Elevator Pitch)

*"The project I'm most proud of is **ShopiTry**, a fully functional e-commerce platform. Unlike typical monolithic applications, I built this using a **Microservices Architecture**. This means instead of one big backend code, I separated the logic into distinct services like Catalog, Cart, Orders, and Payments. I deployed these using **AWS EC2** and **Lambda**, and used **MongoDB Atlas** for the database."*

### 2. Deep Dive Questions & Answers (Technical Drill-down)

#### Q: Why did you choose Microservices over a Monolith?
**Answer:**
*"I chose Microservices to demonstrate scalability and separation of concerns. In an e-commerce app, the 'Catalog' service has different load patterns than the 'Cart' or 'Payment' service. By decoupling them, if I need to scale up the 'Cart' service during a flash sale, I can just add more resources to that specific service without touching the others. It also makes the codebase easier to maintain."*

#### Q: How does your architecture handle communication between services?
**Answer:**
*"I implemented a central **Gateway Service (Port 5000)**. All frontend requests first hit the Gateway. The Gateway acts as a traffic police—it authenticates the user via JWT, validates the request, and then routes the traffic to the specific service needed (e.g., routing to Port 5001 for products, or 5002 for cart items). This abstracts the backend complexity from the frontend."*

#### Q: How did you handle the Payment and Notification logic?
**Answer:**
*"For **Payment and Notifications**, I used **AWS Lambda (Serverless)**. I reasoned that these are event-driven tasks. Payment processing doesn't need to run 24/7; it only needs to run when a user clicks 'Checkout'. Using Lambda is cost-effective because I only pay for the compute time used during the transaction, not for idle server time. It also automatically scales if thousands of people try to pay at once."*

#### Q: Tell me about your Database choice.
**Answer:**
*"I used **MongoDB Atlas**. E-commerce data is unstructured; a 'Laptop' product has different attributes (specs, RAM, CPU) compared to a 'T-shirt' (size, color). MongoDB's flexible document structure handles this perfectly. I also implemented separate databases (`catalog_db`, `cart_db`, `orders_db`) to ensure that a heavy load on orders doesn't slow down the product browsing experience."*

#### Q: How did you deploy the Frontend?
**Answer:**
*"The Frontend is a React application. I built the production bundle and hosted it on **AWS S3** because S3 is highly reliable and cheap for static assets. For the real-world professional setup, I also integrated **CloudFront (CDN)** and **Route 53** for custom domain management and SSL termination to ensure HTTPS security."*

### 3. Challenges Faced & Solutions (Show Problem Solving)

**Challenge:**
*"Initially, I faced CORS issues and connection timeouts because the microservices were running on different EC2 instances with different IPs."*

**Solution:**
*"I solved this by strictly configuring the **Security Groups** in AWS to allow traffic only on specific ports (5000-5005). I also centralized the configuration in the Gateway Service's `.env` file to manage the private IPs of the other services securely. Additionally, I used **PM2** on the servers to ensure that if a node process crashed, it would restart automatically."*

---

# 🚀 Key Buzzwords to Drop (Use these naturally)

*   **Microservices Architecture** (System Design)
*   **Horizontal Scaling** (Growth)
*   **Event-Driven Architecture** (Lambda)
*   **Decoupling** (Clean Code)
*   **Load Balancing / API Gateway** (Networking)
*   **JWT Authentication** (Security)
*   **Process Management (PM2)** (DevOps)
*   **Serverless Computing** (Cost Optimization)
*   **NoSQL Sharding** (Database Management)

---

# 📌 Example LinkedIn "Featured Skills" tags to add

#JavaScript #React #NodeJS #MongoDB #AWS #Microservices #SystemDesign #RESTAPI #DevOps #CloudComputing #Serverless #FullStackDevelopment

---

### Summary Cheat Sheet for the Interviewer:

*   **What is it?** E-commerce App.
*   **Architecture:** Microservices (Gateway, Catalog, Cart, Order).
*   **Cloud:** AWS (EC2 for backend, Lambda for Payment/Notification, S3 for Frontend).
*   **Database:** MongoDB Atlas.
*   **Why?** To learn scalability and modern cloud deployment patterns.
*   **Role:** Full Stack Developer (Built Frontend, Backends, Database Schema, and Cloud Infra).


</details>
