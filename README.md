# CloudLaunch AWS Deployment

**AltSchool of Cloud Engineering – Tinyuka 2024 Third Semester**  
**Month 1 Assessment - David Yaya**

## Project Overview

CloudLaunch is a lightweight platform that showcases a basic company website and stores internal private documents. This project demonstrates the deployment of AWS core services including S3, IAM, and VPCs with strict access controls.

---

## Task 1: Static Website Hosting with S3 and IAM

### S3 Buckets Created

#### 1. cloudlaunch-site-bucket-david-yaya
- **Purpose**: Hosts the public-facing static website
- **Configuration**: Static website hosting enabled with public read access
- **Website URL**: http://cloudlaunch-site-bucket-david-yaya.s3-website-us-east-1.amazonaws.com/

#### 2. cloudlaunch-private-bucket-david-yaya
- **Purpose**: Secure storage for internal documents
- **Access**: IAM user can GetObject and PutObject only (no delete permissions)

#### 3. cloudlaunch-visible-only-bucket-david-yaya
- **Purpose**: Bucket listing demonstration
- **Access**: IAM user can list but cannot access contents

### Implementation Summary

- Created three S3 buckets with appropriate access controls
- Enabled static website hosting on the site bucket
- Configured bucket policy for public read access
- Uploaded responsive HTML website
- Implemented IAM policies for granular access control

---

## Task 2: VPC Network Design

### VPC Configuration

**VPC Name**: cloudlaunch-vpc  
**CIDR Block**: 10.0.0.0/16  
**Region**: us-east-1

### Subnets

| Subnet Name | CIDR Block | Type | Purpose |
|-------------|------------|------|---------|
| cloudlaunch-public-subnet | 10.0.1.0/24 | Public | Load balancers, bastion hosts |
| cloudlaunch-app-subnet | 10.0.2.0/24 | Private | Application servers |
| cloudlaunch-db-subnet | 10.0.3.0/28 | Private | Database instances |

### Network Components

#### Internet Gateway
- **Name**: cloudlaunch-igw
- **Status**: Attached to cloudlaunch-vpc
- **Purpose**: Enables internet connectivity for public subnet

#### Route Tables

**cloudlaunch-public-rt** (Public Route Table)
- Associated with: cloudlaunch-public-subnet
- Routes: `10.0.0.0/16` → local, `0.0.0.0/0` → cloudlaunch-igw

**cloudlaunch-app-rt** (Application Route Table)
- Associated with: cloudlaunch-app-subnet
- Routes: `10.0.0.0/16` → local (No internet access - fully private)

**cloudlaunch-db-rt** (Database Route Table)
- Associated with: cloudlaunch-db-subnet
- Routes: `10.0.0.0/16` → local (No internet access - fully private)

#### Security Groups

**cloudlaunch-app-sg**
- Inbound: HTTP (TCP 80) from 10.0.0.0/16 (VPC only)
- Purpose: Control traffic to application layer

**cloudlaunch-db-sg**
- Inbound: MySQL/Aurora (TCP 3306) from 10.0.2.0/24 (App subnet only)
- Purpose: Secure database access

---

## AWS Account Access Information

### Console Access
- **Account ID**: `870999104851`
- **Console Sign-In URL**: https://870999104851.signin.aws.amazon.com/console
- **IAM Username**: `cloudlaunch-user`
- **Initial Password**: `CloudLaunch2024!`
- **Password Policy**: ✅ User must change password on first login

### First Login Instructions
1. Navigate to: https://870999104851.signin.aws.amazon.com/console
2. Enter username: `cloudlaunch-user`
3. Enter password: `CloudLaunch2024!`
4. You will be prompted to create a new password
5. Set a strong password meeting AWS requirements

---

## IAM Policy Documentation

### CloudLaunchUserPolicy (S3 Access)
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ListAllBuckets",
            "Effect": "Allow",
            "Action": ["s3:ListBucket"],
            "Resource": [
                "arn:aws:s3:::cloudlaunch-site-bucket-david-yaya",
                "arn:aws:s3:::cloudlaunch-private-bucket-david-yaya",
                "arn:aws:s3:::cloudlaunch-visible-only-bucket-david-yaya"
            ]
        },
        {
            "Sid": "ReadSiteBucket",
            "Effect": "Allow",
            "Action": ["s3:GetObject"],
            "Resource": "arn:aws:s3:::cloudlaunch-site-bucket-david-yaya/*"
        },
        {
            "Sid": "ReadWritePrivateBucket",
            "Effect": "Allow",
            "Action": ["s3:GetObject", "s3:PutObject"],
            "Resource": "arn:aws:s3:::cloudlaunch-private-bucket-david-yaya/*"
        },
        {
            "Sid": "NoAccessToVisibleOnlyContent",
            "Effect": "Deny",
            "Action": ["s3:GetObject", "s3:PutObject"],
            "Resource": "arn:aws:s3:::cloudlaunch-visible-only-bucket-david-yaya/*"
        }
    ]
}
```

### CloudLaunchVPCReadOnly (VPC Read Access)
```json
{
    "Version": "2012-10-17",
    "Statement": [{
        "Sid": "VPCReadOnly",
        "Effect": "Allow",
        "Action": [
            "ec2:DescribeVpcs",
            "ec2:DescribeSubnets",
            "ec2:DescribeRouteTables",
            "ec2:DescribeSecurityGroups",
            "ec2:DescribeInternetGateways"
        ],
        "Resource": "*"
    }]
}
```

### Policy Explanation

**S3 Permissions:**
- **ListBucket**: User can view all three buckets in S3 console
- **GetObject (site-bucket)**: Read-only access to website files
- **GetObject + PutObject (private-bucket)**: Read and write access (no delete)
- **Explicit Deny (visible-only-bucket)**: User can see bucket exists but cannot access contents

**VPC Permissions:**
- Read-only access to view VPC configuration
- Cannot create, update, or delete any VPC resources

---

## Network Architecture
```
Internet
    |
    | (IGW: cloudlaunch-igw)
    |
[VPC: 10.0.0.0/16]
    |
    ├── Public Subnet (10.0.1.0/24)
    |   └── Route to 0.0.0.0/0 via IGW
    |
    ├── App Subnet (10.0.2.0/24) [Private]
    |   └── No internet route
    |   └── Security Group: HTTP from VPC
    |
    └── DB Subnet (10.0.3.0/28) [Private]
        └── No internet route
        └── Security Group: MySQL from App Subnet
```

---

## Testing Verification

### S3 Access Tests
✅ Static website loads publicly at the provided URL  
✅ IAM user can list all three buckets  
✅ IAM user can upload/download from private bucket  
✅ IAM user can read from site bucket  
✅ IAM user cannot delete from any bucket  
✅ IAM user can see but not access visible-only bucket  

### VPC Access Tests
✅ IAM user can view VPC details  
✅ IAM user can view subnets, route tables, and security groups  
✅ IAM user cannot modify or delete VPC resources  

---

## Technologies Used

- **AWS S3**: Static website hosting and object storage
- **AWS IAM**: User and permission management with custom policies
- **AWS VPC**: Network isolation and security
- **HTML/CSS**: Responsive static website

---

## Security Best Practices Implemented

- ✅ Principle of least privilege for IAM permissions
- ✅ Explicit deny for sensitive bucket access
- ✅ Private subnets isolated from internet
- ✅ Security groups with minimal required access
- ✅ Password change enforced on first login
- ✅ No unnecessary public access

---

## Cost Optimization

- All resources within AWS Free Tier
- No paid services deployed (EC2, NAT Gateway, RDS)
- Static website hosting (cost-effective)
- Estimated monthly cost: $0.00

---

## Key Features

### Task 1 - S3 & IAM
- Three S3 buckets with different access levels
- Public static website with responsive design
- Custom IAM policies with granular permissions
- Secure credential management

### Task 2 - VPC Design
- Logically separated network tiers
- Proper subnet CIDR planning
- Internet Gateway for public subnet only
- Private subnets without internet access
- Security groups with restricted access

---

## Lessons Learned

1. **IAM Policies**: Explicit deny takes precedence over allow statements
2. **VPC Design**: Proper CIDR planning crucial for scalability
3. **Route Tables**: Each subnet needs explicit route table associations
4. **Security Groups**: Stateful firewalls automatically allow return traffic
5. **S3 Static Hosting**: Simple, cost-effective solution for static content

---

## Future Enhancements

- Deploy EC2 instances in App Subnet
- Set up RDS database in DB Subnet
- Implement NAT Gateway for private subnet updates
- Add Application Load Balancer
- Implement CloudWatch monitoring
- Add CloudFront CDN for HTTPS and caching

---

## Author

**David Yaya**  
AltSchool of Cloud Engineering – Tinyuka 2024  
Semester 3 – Month 1 Assessment

---

## Acknowledgments

Special thanks to AltSchool Africa for this comprehensive hands-on AWS assessment.
