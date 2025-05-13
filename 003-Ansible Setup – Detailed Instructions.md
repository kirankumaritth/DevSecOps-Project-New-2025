# ✅ Step 2: Ansible Setup – Detailed Instructions

---

## 📁 Directory Structure Setup

First, create the following directory layout:

```bash
mkdir -p ansible/roles/{common,docker,jenkins,k3s,sonarqube,prometheus,grafana}/tasks
cd ansible
```

---

## 📄 Create Inventory File

Create the Ansible inventory file:

```bash
vi inventory
```

Paste the following content:

```ini
[devsecops]
<your-ec2-public-ip> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
```

Replace `<your-ec2-public-ip>` with the actual IP from Terraform output.

---

## 📄 Create Main Playbook

```bash
vi playbook.yml
```

Paste the following:

```yaml
- name: DevSecOps All-in-One Setup
  hosts: devsecops
  become: true
  roles:
    - common
    - docker
    - jenkins
    - k3s
    - sonarqube
    - prometheus
    - grafana
```

---

## 🔧 Create Each Role

---

### ✅ Role: `common`

```bash
vi roles/common/tasks/main.yml
```

Paste:

```yaml
- name: Update APT cache
  apt: update_cache=yes

- name: Install basic dependencies
  apt: name={{ item }} state=present
  loop:
    - curl
    - git
    - unzip
    - gnupg
    - ca-certificates
    - software-properties-common
```

---

### ✅ Role: `docker`

```bash
vi roles/docker/tasks/main.yml
```

Paste:

```yaml
- name: Install Docker
  apt:
    name: docker.io
    state: present
    update_cache: yes

- name: Enable and start Docker
  systemd:
    name: docker
    enabled: yes
    state: started

- name: Add user to docker group
  user:
    name: ubuntu
    groups: docker
    append: yes
```

---

### ✅ Role: `jenkins`

```bash
vi roles/jenkins/tasks/main.yml
```

Paste:

```yaml
- name: Add Jenkins GPG key and repo
  apt_key:
    url: https://pkg.jenkins.io/debian/jenkins.io.key
    state: present

- name: Add Jenkins repo
  apt_repository:
    repo: deb https://pkg.jenkins.io/debian binary/
    state: present

- name: Install Java and Jenkins
  apt:
    name:
      - openjdk-11-jdk
      - jenkins
    state: present
    update_cache: yes

- name: Enable and start Jenkins
  systemd:
    name: jenkins
    enabled: yes
    state: started
```

---

### ✅ Role: `k3s`

```bash
vi roles/k3s/tasks/main.yml
```

Paste:

```yaml
- name: Install K3s
  shell: curl -sfL https://get.k3s.io | sh -
  args:
    creates: /usr/local/bin/k3s

- name: Wait for K3s to start
  wait_for:
    port: 6443
    timeout: 120
```

---

### ✅ Role: `sonarqube`

```bash
vi roles/sonarqube/tasks/main.yml
```

Paste:

```yaml
- name: Pull SonarQube Docker image
  docker_image:
    name: sonarqube
    tag: latest
    source: pull

- name: Run SonarQube container
  docker_container:
    name: sonarqube
    image: sonarqube
    state: started
    restart_policy: always
    ports:
      - "9000:9000"
```

---

### ✅ Role: `prometheus`

```bash
vi roles/prometheus/tasks/main.yml
```

Paste:

```yaml
- name: Create Prometheus config directory
  file:
    path: /opt/prometheus
    state: directory

- name: Copy Prometheus config
  copy:
    content: |
      global:
        scrape_interval: 15s
      scrape_configs:
        - job_name: 'prometheus'
          static_configs:
            - targets: ['localhost:9090']
    dest: /opt/prometheus/prometheus.yml

- name: Run Prometheus container
  docker_container:
    name: prometheus
    image: prom/prometheus
    state: started
    restart_policy: always
    ports:
      - "9090:9090"
    volumes:
      - /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
```

---

### ✅ Role: `grafana`

```bash
vi roles/grafana/tasks/main.yml
```

Paste:

```yaml
- name: Run Grafana container
  docker_container:
    name: grafana
    image: grafana/grafana
    state: started
    restart_policy: always
    ports:
      - "3000:3000"
```

---

## 🚀 How to Run Ansible

Once all files are in place:

```bash
cd ansible/
ansible-playbook -i inventory playbook.yml
```

---

## ✅ Access the Tools

| Tool           | URL                    | Default Credentials                                          |
| -------------- | ---------------------- | ------------------------------------------------------------ |
| **Jenkins**    | `http://<ec2-ip>:8080` | Unlock using `/var/lib/jenkins/secrets/initialAdminPassword` |
| **SonarQube**  | `http://<ec2-ip>:9000` | `admin` / `admin`                                            |
| **Grafana**    | `http://<ec2-ip>:3000` | `admin` / `admin`                                            |
| **Prometheus** | `http://<ec2-ip>:9090` | No login required                                            |
| **K3s**        | Use `kubectl` on EC2   | Config at `/etc/rancher/k3s/k3s.yaml`                        |

---
