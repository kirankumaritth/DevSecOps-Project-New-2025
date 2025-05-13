# ✅ Terraform Script – AWS EC2 + EBS (Optional Volume)

Here's a **complete, ready-to-use Terraform configuration**:

---

### 📁 Directory Structure:

```
terraform/
├── main.tf
├── variables.tf
├── outputs.tf
```

---

### 📄 `main.tf`

```hcl
provider "aws" {
  region = var.aws_region
}

resource "aws_key_pair" "deployer" {
  key_name   = var.key_name
  public_key = file(var.public_key_path)
}

resource "aws_instance" "devsecops_ec2" {
  ami           = var.ami_id
  instance_type = var.instance_type
  key_name      = aws_key_pair.deployer.key_name

  root_block_device {
    volume_size = 50
    volume_type = "gp2"
  }

  tags = {
    Name = "DevSecOps-Instance"
  }

  vpc_security_group_ids = [aws_security_group.devsecops_sg.id]
}

resource "aws_security_group" "devsecops_sg" {
  name        = "devsecops_sg"
  description = "Allow DevSecOps tool ports"
  
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Jenkins
  }

  ingress {
    from_port   = 9000
    to_port     = 9000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # SonarQube
  }

  ingress {
    from_port   = 3000
    to_port     = 3000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Grafana
  }

  ingress {
    from_port   = 9090
    to_port     = 9090
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Prometheus
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

---

### 📄 `variables.tf`

```hcl
variable "aws_region" {
  default = "us-east-1"
}

variable "instance_type" {
  default = "t2.medium"
}

variable "ami_id" {
  description = "Ubuntu 22.04 AMI ID (update for your region)"
  default     = "ami-0c7217cdde317cfec"  # Ubuntu 22.04 LTS (us-east-1)
}

variable "key_name" {
  default = "devsecops-key"
}

variable "public_key_path" {
  default = "~/.ssh/id_rsa.pub"
}
```

---

### 📄 `outputs.tf`

```hcl
output "instance_public_ip" {
  value = aws_instance.devsecops_ec2.public_ip
}

output "instance_id" {
  value = aws_instance.devsecops_ec2.id
}
```

---

## 🧪 How to Run This

1. ✅ Save the files in a `terraform/` directory.
2. ✅ Run the following in that directory:

```bash
terraform init
terraform plan
terraform apply
```

You’ll be prompted to approve. Once applied, it will:

* Spin up a t2.medium EC2 with 50GB storage.
* Open ports for Jenkins, SonarQube, Prometheus, Grafana.
* Assign a key for SSH access.

---

## ⚠️ Notes

* ✅ Update the `ami_id` to match your region from [Ubuntu AMIs](https://cloud-images.ubuntu.com/locator/).
* ✅ Ensure your key at `~/.ssh/id_rsa.pub` exists, or point to another public key.
* ❗ This EC2 **must have sufficient IAM permissions**, e.g., access via an IAM role or CLI credentials.
* 🔒 You may want to restrict `cidr_blocks` to your IP for better security.

---

## ✅ Next Step?

Once your EC2 is up, we'll proceed to:

### ➡️ Step 2: **Ansible Playbook** to:

* Install Docker, Jenkins, K8s (Minikube or K3s), SonarQube, Prometheus, Grafana, etc.
