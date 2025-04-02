# IaaS Paskaitos Medžiaga

## 🧪 Papildoma praktinė užduotis: AWS + Linux (GUI + Ansible)

### 🔍 Užduoties tikslas
Sukurti pilnai veikiančią infrastruktūrą AWS debesijoje, naudojant tiek grafinę sąsają (AWS Console), tiek automatizuotai per **Ansible**:
- Virtualus tinklas (VPC) ir subnet
- Security Group su SSH prieiga
- EC2 instancija su Linux ir viešu IP
- Prisijungimas per SSH

---

## ✅ 1 būdas: Naudojant AWS Console (GUI)

1. **Sukurkite VPC**:
   - CIDR blokas: `10.0.0.0/16`
   - VPC pavadinimas: `demo-vpc`

2. **Sukurkite subnetą**:
   - CIDR blokas: `10.0.1.0/24`
   - AZ: `eu-central-1a`
   - Subnet pavadinimas: `demo-subnet`
   - Įjunkite „Auto-assign public IP“

3. **Sukurkite Security Group**:
   - Pavadinimas: `allow-ssh`
   - Pridėkite inbound taisyklę:
     - Type: SSH
     - Port: 22
     - Source: `0.0.0.0/0`

4. **Sukurkite Key Pair**:
   - Eikite į EC2 > Key Pairs
   - Sukurkite naują `demo-key`, pasirinkite `.pem` tipą
   - Atsisiųskite ir saugokite raktą `~/.ssh/demo-key.pem`

5. **Sukurkite EC2 instanciją**:
   - AMI: Amazon Linux 2023 arba Ubuntu 22.04
   - Type: `t2.micro`
   - Key pair: `demo-key`
   - Network: `demo-vpc`
   - Subnet: `demo-subnet`
   - Public IP: Enabled
   - Security Group: `allow-ssh`

6. **Prisijunkite per SSH**:
   ```bash
   chmod 400 ~/.ssh/demo-key.pem
   ssh -i ~/.ssh/demo-key.pem ec2-user@<EC2-PUBLIC-IP>
   ```

---

## 🤖 2 būdas: Automatizuotas diegimas su Ansible

### Failų struktūra:
```
aws_linux_instance/
├── create_ec2.yml
├── group_vars/
│   └── all.yml
```

### group_vars/all.yml
```yaml
region: eu-central-1
ami_id: ami-0c55b159cbfafe1f0  # Amazon Linux 2023 (Frankfurt regionas)
instance_type: t2.micro
key_name: demo-key
```

### create_ec2.yml
```yaml
- name: Create AWS Infrastructure (Linux EC2)
  hosts: localhost
  connection: local
  gather_facts: false
  vars_files:
    - group_vars/all.yml

  tasks:

    - name: Create VPC
      amazon.aws.ec2_vpc_net:
        name: my-vpc
        cidr_block: 10.0.0.0/16
        region: "{{ region }}"
        profile: ec2_micro_provisioner
      register: vpc

    - name: Create Internet Gateway
      amazon.aws.ec2_vpc_igw:
        state: present
        region: "{{ region }}"
        profile: ec2_micro_provisioner
        vpc_id: "{{ vpc.vpc.id }}"
        tags:
          Name: my-vpc-igw
      register: igw

    - name: Create Subnet
      amazon.aws.ec2_vpc_subnet:
        vpc_id: "{{ vpc.vpc.id }}"
        cidr: 10.0.1.0/24
        az: "{{ region }}a"
        map_public: yes
        region: "{{ region }}"
        profile: ec2_micro_provisioner
      register: subnet

    - name: Create Route Table with IGW route
      amazon.aws.ec2_vpc_route_table:
        vpc_id: "{{ vpc.vpc.id }}"
        region: "{{ region }}"
        profile: ec2_micro_provisioner
        tags:
          Name: my-vpc-rt
        routes:
          - destination_cidr_block: 0.0.0.0/0
            gateway_id: "{{ igw.gateway_id }}"
      register: route_table

    - name: Associate Route Table with Subnet (community.aws)
      community.aws.ec2_vpc_route_table_association:
        subnet_id: "{{ subnet.subnet.id }}"
        route_table_id: "{{ route_table.route_table.id }}"
        region: "{{ region }}"
        profile: ec2_micro_provisioner

    - name: Create Security Group
      amazon.aws.ec2_group:
        name: ssh-access
        description: Allow SSH
        vpc_id: "{{ vpc.vpc.id }}"
        region: "{{ region }}"
        profile: ec2_micro_provisioner
        rules:
          - proto: tcp
            from_port: 22
            to_port: 22
            cidr_ip: 0.0.0.0/0
      register: sg

    - name: Launch EC2 instance
      amazon.aws.ec2_instance:
        name: demo-linux
        key_name: "{{ key_name }}"
        region: "{{ region }}"
        profile: ec2_micro_provisioner
        instance_type: "{{ instance_type }}"
        image_id: "{{ ami_id }}"
        vpc_subnet_id: "{{ subnet.subnet.id }}"
        security_group: "{{ sg.group_id }}"
        wait: yes
        volumes:
          - device_name: /dev/xvda
            ebs:
              volume_size: 8
              delete_on_termination: true
      register: ec2

    - name: Output EC2 Public IP
      debug:
        msg: "Prisijunkite per SSH: ssh -i ~/.ssh/demo-key.pem ec2-user@{{ ec2.instances[0].public_ip_address }}"
```

### 🛠️ Vykdymas:
```bash
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
ansible-playbook create_ec2.yml
```

---

## 📝 Užduoties tikslas studentams:
- Supažindinti su AWS tinklų, instancijų, SSH raktų principais
- Praktiškai panaudoti GUI ir automatizavimo įrankį (Ansible)
- Suprasti infrastruktūros kaip kodo (IaC) reikšmę


