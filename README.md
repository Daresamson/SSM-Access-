
# Step-by-Step Documentation: Enabling and Using SSM Access on EC2

---

## Part 1: Setting up SSM Access

1. Launch an **EC2 instance** (Ubuntu or Amazon Linux 2).
2. Attach an **IAM Role** with the policy **AmazonSSMManagedInstanceCore** to the instance.
3. Ensure the **SSM Agent** is installed and running on the instance.

---

## Part 2: Using EC2 User Data to Install the SSM Agent

If SSH is not available, you can use the **EC2 User Data** script to automatically install the SSM Agent on boot.

**Steps:**

* Go to **EC2 Console → Instances → Select your DB Instance**.
* **Actions → Instance Settings → Edit User Data**.
* Paste one of the following scripts depending on your OS.

### For Amazon Linux 2:

```bash
#!/bin/bash
yum install -y amazon-ssm-agent
systemctl enable amazon-ssm-agent
systemctl start amazon-ssm-agent
```

### For Ubuntu:

```bash
#!/bin/bash
apt-get update -y
snap install amazon-ssm-agent --classic
systemctl enable snap.amazon-ssm-agent.amazon-ssm-agent.service
systemctl start snap.amazon-ssm-agent.amazon-ssm-agent.service
```

**After saving the User Data update:**

1. Stop the instance.
2. Start it again (**not just reboot — a full stop/start is required**).
3. Wait **2–3 minutes** for the User Data script to run.

---

## Part 3: Verifying SSM Connection

To confirm the instance is registered with SSM, run:

```bash
aws ssm describe-instance-information \
  --region us-east-1 \
  --query "InstanceInformationList[?InstanceId=='<instance-id>'].[InstanceId,PingStatus,PlatformName,PlatformVersion]" \
  --output table
```

---

## Part 4: IAM Permissions Required

You must configure IAM permissions as follows:

* **EC2 Role (Instance Profile)** → Attach to your EC2 instance

  * Policy: `AmazonSSMManagedInstanceCore`

* **IAM User (e.g., Damilare2)** → To run SSM commands from AWS CLI

  * Policy: `AmazonSSMFullAccess` or an equivalent custom policy that allows SSM actions

---

## Part 5: How an External User Can Access via SSM

1. The external user must have an **IAM user** with SSM permissions (`AmazonSSMFullAccess` or equivalent).

2. Install and configure AWS CLI on their local machine:

   ```bash
   aws configure
   ```

   (provide Access Key, Secret Key, and region)

3. Verify connectivity by running:

   ```bash
   aws ssm describe-instance-information --region us-east-1
   ```

4. Start a session into the EC2 instance:

   ```bash
   aws ssm start-session --target <instance-id> --region us-east-1
   ```

At this point, the user can securely connect to the EC2 instance **without requiring SSH or a public IP address**.

---

✅ Done!

Do you want me to also save this as a `.md` file so you can open it directly in **VS Code or GitHub**?
