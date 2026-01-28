# 🏗️ **AWS VPC Architecture – Production‑Ready Networking Design**

**Author:** Deepak Pilli

**Project:** Scalable Multi‑Tier VPC with Subnet Sizing & Routing Architecture

------

# 📘 **Project Overview**

This project delivers a **scalable, future‑proof AWS VPC architecture** built using enterprise CIDR planning, multi‑tier subnetting, controlled internet access, and proper routing separation.

The design supports:

- Admin access
- Public‑facing edge services
- Web and application backend tiers
- Shared internal services
- High‑scale container platforms

The network is engineered to scale for years without redesign.

------

# 🎯 **Objectives**

- Build a VPC large enough for long‑term organizational growth
- Create **6 subnets of unequal sizes** based on capacity needs
- Follow **strict CIDR alignment rules**
- Implement **public & private routing architecture**
- Enforce **security‑driven traffic behavior**
- Validate traffic flow & isolation
- Provide full documentation

------

# 🧠 **Business Scenario**

The organization is building a shared cloud platform that will host a wide mix of workloads such as:

- Admin & bastion services
- Internet-facing edge layers
- Web tiers
- Application backends
- Internal shared services
- High-density container platforms

The network must be:

- Highly scalable
- Predictable
- Secure
- Easy to audit
- Future‑extensible

------

# 📝 **Task 1: VPC CIDR & Capacity Planning**

Selected VPC CIDR:

```
VPC CIDR: 10.0.0.0/16
```

Reasons:

- Provides 65,536 total IPs

- Allows unequal subnet sizes

- Ensures long‑term scalability

- Prevents future rearchitecture

  ![](Screenshot%202026-01-28%20112650.png)

------

# 🧮 **Task 2: Subnet Design**

Six subnets created with **proper CIDR boundaries** and **no overlaps**.

With the help of this table now we are going to create subnets

| Subnet Name  | Purpose                 | Required IPs | Final CIDR Assigned |
| ------------ | ----------------------- | ------------ | ------------------- |
| **Shared**   | Internal large services | ~8,192       | **10.0.32.0/19**    |
| **Platform** | Containers / tooling    | ~4,096       | **10.0.16.0/20**    |
| **App**      | Application tier        | ~2,048       | **10.0.8.0/21**     |
| **Web**      | Web tier                | ~1,024       | **10.0.4.0/22**     |
| **Edge**     | LB / ingress            | ~512         | **10.0.2.0/23**     |
| **Admin**    | Bastion / ops           | ~256         | **10.0.0.0/24**     |

### Subnet Rules Followed

✔ Largest subnets placed first
 ✔ No overlapping CIDRs
 ✔ All boundaries aligned correctly
 ✔ All subnets fit within 10.0.0.0/16

Images are attached below for reference

Using CIDR math create 5 subnets

![](Screenshot5202026-01-28%20113104.png)

----

![](Screenshot%202026-01-28%20113226.png)

----

5 Subnets are created using CIDR math

![](Screenshot%202026-01-28%20114457.png)

----



------

# 🌍 **Task 3: Internet Gateway**

- Created IGW

- Attached to VPC

- Used **only** by public route table

- Ensures controlled external access

  ![](Screenshot%202026-01-28%20112746.png)

------

# 🧭 **Task 4: Route Table Architecture**

Two route tables created:

### **Public Route Table**

Used by:

- **Admin**
- **Edge**

Routes:

```
Local VPC Route
0.0.0.0/0 → IGW
```

![](Screenshot%202026-01-28%20114605.png)

-----

Using subnet associations attach admin and edge subnets which are to be public 

![](Screenshot%202026-01-28%20114630.png)

----------------

In routes attach IGW so that they gain access to internet

![](Screenshot%202026-01-28%20114802.png)



### **Private Route Table**

Used by:

- Web
- App
- Platform
- Shared

Routes:

```
Local VPC Route Only
(No default internet route)
```

------

------------

Create a Private Route Table

![](Screenshot%202026-01-28%20114954.png)

--------------------

In subnet Associations attach platform,shared,web,app which are to be private and remember donot assign IGW since it makes routes to expose to the internet just associate subnets 

![](Screenshot%202026-01-28%20115028.png)

#  Route Table Associations

​    Now our subnets are associated

| Subnet   | Route Table |
| -------- | ----------- |
| Admin    | Public‑RT   |
| Edge     | Public‑RT   |
| Web      | Private‑RT  |
| App      | Private‑RT  |
| Platform | Private‑RT  |
| Shared   | Private‑RT  |

✔ None rely on “Main Route Table”
 ✔ All are explicitly associated

------

# 🔐 **Task 6: Security‑Driven Network Behavior**

- Only **Admin** & **Edge** subnets have internet access

  I am now creating a server for public subnets  

  ![](Screenshot%202026-01-28%20115856.png)

  -----------

  Use the VPC and Public subnet in Network settings before creating EC2

  ![](Screenshot%202026-01-28%20115932.png)

  ---------------

  We Created a public server now lets test that

  ![](Screenshot%202026-01-28%20120550.png)

  ----------------

  It has internet Access

  ![](Screenshot%202026-01-28%20120832.png)

- Private subnets **cannot reach the internet**

  Lets create an EC2 using our private subnets

  ![](Screenshot%202026-01-28%20120956.png)

  ---

  Server is created

  ![](Screenshot%202026-01-28%20121135.png)

  -----------------

  The private network Donot have access to internet

  ![](Screenshot%202026-01-28%20121207.png)

------

# 🧪 **Task 7: Validation & Testing**

### **Public Subnets (Admin & Edge):**

✔ Can reach internet
 → Because Public‑RT contains `0.0.0.0/0 → IGW`

### **Private Subnets (Web, App, Platform, Shared):**

❌ Cannot reach internet
 → No IGW route in Private‑RT

### **Internal Communication:**

✔ All subnets reach each other internally
 → AWS automatically provides local VPC routing

------

# 📊 **Task 8: Failure & Audit Scenarios**

### ❌ If IGW is detached:

- Public subnets instantly lose internet
- Private subnets remain unchanged
- Ingress stops working
- Systems requiring outbound updates fail

### ❌ If a private subnet is mistakenly mapped to Public‑RT:

- It would gain unintended internet access
- Violates security & compliance
- Auditors will mark it as critical misconfiguration

### ❌ Misaligned CIDR (/19 at wrong start):

- Overlaps would occur
- AWS rejects subnet creation
- Future subnets get blocked
- Network becomes fragmented

### ✔ How the design supports future growth:

- Plenty of unused IP space in the /16
- Easy to add new /24 /23 /22 /21 blocks
- Subnets grow without renumbering
- Future services can be added safely

------

# 🗺️ **Architecture Diagram (Text Version)**

```
                          +-------------------------------+
                          |            VPC                |
                          |         10.0.0.0/16           |
                          +-------------------------------+

 Public Route Table (Internet)
 ├── Admin (10.0.62.0/24)
 └── Edge  (10.0.60.0/23)

 Private Route Table (No Internet)
 ├── Web      (10.0.56.0/22)
 ├── App      (10.0.48.0/21)
 ├── Platform (10.0.32.0/20)
 └── Shared   (10.0.0.0/19)
```

------

# 📄 **Task 9: Deliverables Summary**

### ✔ VPC & Subnet CIDR Table

Included above

### ✔ Route Table Mapping

Included above

### ✔ Traffic Flow Explanation

Completed in validation section

### ✔ Risk & Failure Analysis

Documented above
