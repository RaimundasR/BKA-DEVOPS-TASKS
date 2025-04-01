## Praktinės užduotys – IAM ir prieigos valdymas naudojant AWS ir Ansible

### 1. IAM naudotojo kūrimas AWS sistemoje
- Prisijunkite prie [AWS Console](https://console.aws.amazon.com/).
- Eikite į **IAM** > **Users** > **Add users**.
- Sukurkite naudotoją „dev-jonas“, priskirkite programinę prieigą (programmatic access).
- Priskirkite jam rolę „AmazonEC2ReadOnlyAccess“.
- Tai RBAC pavyzdys: naudotojas turi teisę skaityti EC2 instancų informaciją, bet negali jų kurti ar ištrinti.

### 2. IAM grupės ir politikos taikymas
- Sukurkite grupę „Developers“.
- Priskirkite politiką `AmazonS3ReadOnlyAccess`.
- Pridėkite kelis naudotojus prie šios grupės (pvz., jonas, egle, ruta).
- Visi grupės nariai turės S3 peržiūros prieigą, bet negalės redaguoti failų.

### 3. ABAC AWS pavyzdys su žymomis (tags)
- Sukurkite politiką, kuri leidžia naudotojui pasiekti tik EC2 instancus, pažymėtus žyma `"Department": "DevOps"`.
- Politikos pavyzdys:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Department": "DevOps"
        }
      }
    }
  ]
}
```
- Priskirkite šią politiką naudotojui arba grupei – tai ABAC naudojimas realiame scenarijuje.

### 4. Federacija tarp AWS ir Azure (ar SAML tiekėjo)
- Naudokite **AWS IAM Identity Provider**.
- Sukurkite **SAML provider**, importuokite Azure AD metaduomenų XML.
- Sukurkite IAM rolę su `sts:AssumeRoleWithSAML` politika.
- Azure AD naudotojai galės prisijungti prie AWS naudodami SSO.

### 5. Programinis vartotojas (Service Account) su Access Key
- Sukurkite IAM naudotoją „terraform-bot“.
- Suteikite jam minimalias teises (pvz., tik EC2 ir S3 valdymui).
- Sugeneruokite **access key** ir **secret key**, naudokite `.aws/credentials` faile.
- Šis naudotojas gali būti naudojamas CI/CD pipeline arba Terraform skambučiams.

### 6. IAM naudotojo kūrimas naudojant Ansible modulį `iam`

Naudojant Ansible `community.aws` kolekciją, galima automatizuoti IAM naudotojų kūrimą ir politikų priskyrimą.

**Pavyzdinis playbook**:
```yaml
- name: Sukurti IAM naudotoją per Ansible
  hosts: localhost
  connection: local
  gather_facts: no
  vars:
    aws_region: eu-central-1
    iam_user_name: dev-egle
  tasks:
    - name: Sukurti IAM naudotoją
      community.aws.iam_user:
        name: "{{ iam_user_name }}"
        state: present
        region: "{{ aws_region }}"

    - name: Priskirti politiką naudotojui (EC2 read only)
      community.aws.iam_policy_attachment:
        policy_arn: arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess
        users:
          - "{{ iam_user_name }}"
        state: present
        region: "{{ aws_region }}"
```

**Reikalavimai:**
- Įdiegta `community.aws` kolekcija (`ansible-galaxy collection install community.aws`)
- AWS kredencialai prieinami aplinkoje (`~/.aws/credentials` arba aplinkos kintamieji)

Šis pavyzdys parodo, kaip naudoti Ansible norint kurti naudotojus pagal RBAC modelį.

### 7. Saugesnis IAM naudotojų kūrimas su minimaliomis teisėmis

**Principas:** visada naudoti „least privilege“ politiką. IAM naudotojas turėtų gauti tik tas teises, kurios būtinos jo užduočiai atlikti.

**Praktinis pavyzdys su Ansible (sukurti naudotoją be jokių prieigos raktų):**
```yaml
- name: Sukurti saugų IAM naudotoją be programinės prieigos
  hosts: localhost
  connection: local
  gather_facts: no
  vars:
    iam_user_name: secure-user
  tasks:
    - name: Sukurti IAM naudotoją be programmatic access
      community.aws.iam_user:
        name: "{{ iam_user_name }}"
        state: present
        password_reset_required: true
        permissions_boundary: arn:aws:iam::aws:policy/PowerUserAccess  # pasirenkama pagal poreikį
        manage_password: true
        password: "Laikinas123!"
        update_password: on_create
```

**Papildomai:** galima priskirti MFA reikalavimą naudotojui – tai sustiprina saugumą.

**IAM politika reikalaujanti MFA:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```

### 8. Draudimas kurti „Access Keys“ naudotojams
- Sukurkite politiką, kuri neleidžia IAM naudotojui susigeneruoti programinės prieigos raktų:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "iam:CreateAccessKey",
        "iam:UpdateAccessKey",
        "iam:DeleteAccessKey"
      ],
      "Resource": "*"
    }
  ]
}
```
- Priskirkite ją visiems naudotojams, kuriems leidžiama tik interaktyvi prieiga per konsolę (Web login).

### 9. Techninis vartotojas su teisėmis kurti EC2 instancus

Šis pavyzdys parodo, kaip sukurti techninį naudotoją, kuris gali kurti EC2 instancus, bet neturi kitų teisių (tik launch instance).

**IAM politika, leidžianti kurti EC2 instancus:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:RunInstances",
        "ec2:DescribeInstances",
        "ec2:DescribeImages",
        "ec2:CreateTags"
      ],
      "Resource": "*"
    }
  ]
}
```

**Ansible playbook:**
```yaml
- name: Sukurti techninį IAM naudotoją EC2 konfigūravimui
  hosts: localhost
  connection: local
  gather_facts: no
  vars:
    iam_user_name: ec2-launcher
    policy_name: ec2-launch-policy
  tasks:
    - name: Sukurti IAM naudotoją
      community.aws.iam_user:
        name: "{{ iam_user_name }}"
        state: present

    - name: Sukurti politiką leidžiančią kurti EC2
      community.aws.iam_policy:
        name: "{{ policy_name }}"
        state: present
        policy_document: |
          {
            "Version": "2012-10-17",
            "Statement": [
              {
                "Effect": "Allow",
                "Action": [
                  "ec2:RunInstances",
                  "ec2:DescribeInstances",
                  "ec2:DescribeImages",
                  "ec2:CreateTags"
                ],
                "Resource": "*"
              }
            ]
          }

    - name: Priskirti politiką naudotojui
      community.aws.iam_policy_attachment:
        policy_name: "{{ policy_name }}"
        users:
          - "{{ iam_user_name }}"
        state: present
```

**Pastabos:**
- Politiką galima papildyti sąlygomis (pvz., tik tam tikrame regione, tik su tam tikrais žymėjimais).
- Galima naudoti ir kaip Service Account CI/CD įrankiui, kuris deployina EC2 instancus.

---

Jei pageidauji – galiu pridėti papildomą YAML ar Terraform pavyzdį, kad parodytum studentams realią konfigūraciją. 🙂

