# Hosting Assignment

I know I said there wouldn't be an assignment for the hosting module, but there is (sorry). And there's going to be a quiz afterwards as well on Wes' stuff. Don't worry, it should be easy. And it will be open book/internet/whatever.

### The Rules
1. Read the instructions VERY carefully! They might look identical, but they are not!
2. Teams are recommended (two/team)
3. Open book, internet, etc.
    * NOT open neighbor/other team.
4. There are some commands that will be run on the instances. You may optionally enter them as user data that will run automatically after the instance starts.
    * **Note:** If you chose to use user data, you **shouldn't** need your SSH key if done correctly. However, if something breaks, there will be no way to test it... I strongly suggest that you still create a key pair even when providing user data.
    * User data is configured on the Configure Instance page, under Advanced Details. Use the below script as a template.

```
#!/bin/bash
<docker command>
```

5. Ask questions.
6. Have fun.

**Note:** I've provided a custom image that already has Docker installed and configured for you.

### The Details

For this assignment, you will create two instances.
- A Gitea server (A lightweight Git server)
- A SQL server (PostgreSQL)

Once both servers are running:
- Connect Gitea to PostgreSQL.

Once you have Gitea connected to PostgreSQL:
- I'll add every Gitea server to a load balancer and you'll see how a load balancer works. (This will be the test)

---

## Create a new EC2 instance.
Log in to AWS. Make sure your region is set to Oregon, and proceed to EC2.

### SSH Keys
**Note:** All your keys have been deleted. You will need to create new keys!

Please please please! Ask me for help if you need it!

1. First, delete all of the pem files in `~/.ssh`
2. Next, click on "Key Pairs" under Network & Security.
3. Click Create Key Pair
    * Key pair name: [firstinitial_lastname] \(e.g. cpedersen\)
4. Save that key in `~/.ssh`
5. Run the following commands:
    * `chmod 600 ~/.ssh/[name_of_key.pem]` e.g. `ssh ~/.ssh/cpedersen.pem`
    * `ssh-add -k ~/.ssh/[name_of_key.pem]` e.g. `ssh-add -k ~/.ssh/cpedersen.pem`
6. Done.

---

## Instance #1 (Gitea)

### Choose AMI
- Return to the EC2 dashboard (EC2 Dashboard at the very top of the left menu)
- Click Launch Instance
- cohort6-base (Custom AMI/My AMI)

### Choose Instance Type
- Type: t2.micro

### Configure Instance
- Subnet: Select any PUBLIC subnet.
- IAM Role: Ec2FullAccess

### Add Storage
- Storage: 8 GB

### Add Tags
- Key = Name (this is case sensitive, remember)
- Value = mtech6-[your_initials]-gitea

### Configure Security Group:
Create a new security group with the following rules:
- Security group name: mtech6-[your_initials]-gitea-sg
- Description: Anything you'd like.
- Rule #1
    * Type: HTTP
    * Source: 205.118.217.10/32
    * Description: MTECH
- Rule #2
    * Type: SSH
    * Source: 205.118.217.10/32
    * Description: MTECH

### Review
- Launch
- Choose an existing key pair
    * Select the key you created in the first step.
- Launch Instances

### Log in
- `ssh -A ec2-user@<IPv4 Public IP>`
    * Accept the warning by typing 'yes'
    * **Note:** If you can't log in, you probably chose the wrong subnet...
- Run the following command on the instance:
    * `docker run -d --name gitea -p 80:3000 -p 2222:22 --restart always gitea/gitea:latest`
- STAY LOGGED IN!

---

## Instance #2 (PostgreSQL)

### Choose AMI
- Return to the EC2 dashboard (EC2 Dashboard at the very top of the left menu)
- Click Launch Instance
- cohort6-base (Custom AMI/My AMI)

### Choose Instance Type
- Type: t2.micro

### Configure Instance
- Subnet: Select any PRIVATE subnet.
- IAM Role: Ec2FullAccess

### Add Storage
- Storage: 8 GB

### Add Tags
- Key = Name (this is case sensitive, remember)
- Value = mtech6-[your_initials]-postgres

### Configure Security Group:
Create a new security group with the following rules:
- Security group name: mtech6-[your_initials]-postgres-sg
- Description: Anything you'd like.
- Rule #1
    * Type: PostgreSQL
    * Source: 172.31.0.0/16
    * Description: VPC
- Rule #2
    * Type: SSH
    * Source: 172.31.0.0/16
    * Description: VPC

### Review
- Launch
- Choose an existing key pair
    * Select the key you created in the first step.
- Launch Instances

### Log in
- **From the Gitea server:**
    * `ssh ec2-user@<Private IP>`
    * Accept the warning by typing 'yes'
- Run the following command on the instance:
    * `docker run -d --name postgres -p 5432:5432 -e POSTGRES_PASSWORD=mtech_cohort6 --restart always postgres:latest`

---

## Create the Connection

### Configure Gitea
1. Open a browser and browse to:
    * http://<gitea_public_ip>
2. Click Sign In
3. Configure Gitea using the information below. If a setting isn't listed, then don't change it!

```
Database Type: PostgreSQL
Host: `<postgres_private_ip>:5432`
Username: postgres
Password: mtech_cohort6
Database Name: postgres
Site Title: <your name(s)> (e.g. Clint)

Under Optional Settings:
* Fill out the Administrator Account Settings. 
  * You can use a fake email. It doesn't actually do anything except update some tables in the database.
```

<!-- `curl -s http://54.70.187.222/user/login | grep -E "Sign In \- .+"` -->
