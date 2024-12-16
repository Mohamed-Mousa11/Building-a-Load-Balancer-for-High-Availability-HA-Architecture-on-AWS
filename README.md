# Building a Load Balancer for High Availability Architecture on AWS
![ec2_simple_high_availability_architecture_238f1c256f](https://github.com/user-attachments/assets/195c6e42-d4c9-4f96-b485-c46bff7b627d)

 
### **Overview**

A High Availability architecture ensures that your application is resilient to failures, by distributing traffic across multiple instances and availability zones. The key components include:
1. **VPC:** The private network for your architecture.
2. **Subnets:** Public and private subnets across multiple Availability Zones.
3. **Load Balancer:** Distributes traffic to EC2 instances.
4. **Auto Scaling Group (ASG):** Automatically scales EC2 instances to handle traffic spikes and redundancy.
5. **EC2 Instances:** Hosts the application.

---

### **Step 1: Create Your VPC**

1. **Open the AWS Console:**
   - Search for and select **VPC** in the AWS Management Console.

2. **Create a VPC:**
   - In the **VPC Dashboard**, select **Create VPC**.
   - Select **VPC and more** and configure:
     - **Name:** `HA-VPC`
     - **IPv4 CIDR Block:** `10.0.0.0/16`
     - **Number of Availability Zones:** `2`
     - **Number of public subnets:** `2`
     - **Number of private subnets:** `2`
     - **Public Subnet CIDR blocks:** 
       - `10.0.1.0/24` (AZ-1)
       - `10.0.2.0/24` (AZ-2)
     - **Private Subnet CIDR blocks:** 
       - `10.0.3.0/24` (AZ-1)
       - `10.0.4.0/24` (AZ-2)

3. **Enable NAT Gateway:**
   - Set NAT Gateway to `In 2 AZs` for internet access from private subnets.

4. **Create the VPC:**
   - Confirm settings and choose **Create VPC**.

---

### **Step 2: Configure Subnets and Routing**

1. **Verify Subnets:**
   - Navigate to **Subnets** in the VPC Dashboard and confirm the public and private subnets were created for both AZs.

2. **Route Table Configuration:**
   - Go to **Route Tables**.
   - Associate the **Public Route Table** with both public subnets and attach an **Internet Gateway** (IGW) to allow internet access.
   - Associate the **Private Route Table** with both private subnets and configure routes to the NAT Gateway for outbound internet traffic.

---

### **Step 3: Create a Security Group**

1. **Navigate to Security Groups:**
   - In the **VPC Dashboard**, select **Security Groups** and click **Create Security Group**.

2. **Configure Security Group Rules:**
   - **Security Group Name:** `LB-SG` (for Load Balancer)
     - **Inbound Rules:** Allow HTTP (80) and HTTPS (443) from anywhere (`0.0.0.0/0`).
   - **Security Group Name:** `EC2-SG` (for EC2 instances)
     - **Inbound Rules:** 
       - Allow HTTP (80) traffic from the Load Balancer's Security Group.
       - Allow SSH (22) access from your IP for management.

---

### **Step 4: Launch EC2 Instances**

1. **Navigate to EC2:**
   - Go to the **EC2 Dashboard** and select **Launch Instances**.

2. **Configure Instance Details:**
   - **Name:** `HA-Web-Instance`
   - **AMI:** Amazon Linux 2023
   - **Instance Type:** `t2.micro`
   - **Network Settings:**
     - **VPC:** Select `HA-VPC`.
     - **Subnet:** Choose one of the public subnets.
     - **Security Group:** Assign the `EC2-SG`.
   - **User Data Script:**
     ```bash
     #!/bin/bash
     yum update -y
     yum install -y httpd
     systemctl start httpd
     systemctl enable httpd
     echo "Welcome to High Availability Architecture" > /var/www/html/index.html
     ```

3. **Launch the Instance:**
   - Select a key pair (or create one) and launch the instance.

4. **Repeat for multiple instances:**
   - Launch at least 2 instances, one in each public subnet.

---

### **Step 5: Set Up an Elastic Load Balancer**

1. **Navigate to Load Balancers:**
   - In the EC2 Dashboard, choose **Load Balancers** and click **Create Load Balancer**.

2. **Configure Load Balancer:**
   - **Type:** Application Load Balancer (ALB)
   - **Name:** `HA-ALB`
   - **Scheme:** Internet-facing
   - **Network Mapping:**
     - **VPC:** Select `HA-VPC`.
     - Add both public subnets.

3. **Security Groups:**
   - Assign the `LB-SG` to the Load Balancer.

4. **Listeners:**
   - Add a listener for HTTP (port 80).
   - Configure the default action to forward traffic to a **Target Group**.

5. **Create a Target Group:**
   - **Name:** `HA-TG`
   - **Target Type:** Instances
   - **VPC:** Select `HA-VPC`.
   - Register your EC2 instances as targets.

6. **Review and Create:**
   - Finalize settings and create the Load Balancer.

---

### **Step 6: Set Up Auto Scaling Groups (ASG)**

1. **Navigate to Auto Scaling Groups:**
   - In the EC2 Dashboard, choose **Auto Scaling Groups** and click **Create Auto Scaling Group**.

2. **Configure ASG:**
   - **Name:** `HA-ASG`
   - **Launch Template:** Create or use an existing launch template with the EC2 instance details.
   - **VPC:** Select `HA-VPC`.
   - **Subnets:** Choose both private subnets.

3. **Scaling Policies:**
   - Configure scaling to maintain a minimum of 2 instances and scale up to 4 instances based on CPU utilization.

4. **Attach Load Balancer:**
   - Attach the `HA-ALB` created earlier to the ASG.

5. **Health Checks:**
   - Enable ELB-based health checks to ensure unhealthy instances are replaced automatically.

---

### **Step 7: Test the Architecture**

1. **Access the Load Balancer:**
   - Copy the DNS name of the ALB from the Load Balancer details and paste it into your browser.
   - You should see the default message: "Welcome to High Availability Architecture".

2. **Simulate Failover:**
   - Stop one EC2 instance and verify that traffic is routed to the other instance without downtime.

3. **Test Scaling:**
   - Use a load-testing tool (like Apache Benchmark or Locust) to simulate high traffic and verify that the ASG scales instances accordingly.

---

### **Conclusion**

You have successfully built a High Availability architecture with AWS services. This setup ensures fault tolerance, scalability, and resilience for your application. The Load Balancer distributes traffic, and the Auto Scaling Group maintains the desired number of instances based on demand, ensuring minimal downtime.

