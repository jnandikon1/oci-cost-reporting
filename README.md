# OCI Infrastructure Reporting - Automated Inventory & Reporting

Automated **Oracle Cloud Infrastructure (OCI)** inventory collection, data persistence, and reporting using **Ansible playbooks** and **Autonomous Data Warehouse (ADW)**.

## 🎯 Overview

This project provides an enterprise-grade pipeline for discovering, documenting, and reporting on OCI infrastructure resources (ADB and non-ADB databases, VM clusters) with **daily scheduled execution** via Ansible Tower/AWX.

**Key Features:**
- ✅ **Automated OCI Inventory** - Collect ADB and VM cluster resources from all compartments
- ✅ **Data Persistence** - Load inventory to Oracle Autonomous Data Warehouse
- ✅ **Excel Reporting** - Generate detailed cost/capacity reports with calculations
- ✅ **Email Delivery** - Auto-send reports to stakeholders
- ✅ **Enterprise Scheduling** - Ansible Tower/AWX with REST API integration
- ✅ **Production Ready** - Error handling, validation, logging, and role-based design

## 🚀 Quick Start

### 1. Prerequisites

```bash
# Install Ansible (control node)
pip3 install ansible

# Install Python packages (managed node)
pip3 install oci pandas cx_Oracle openpyxl xlsxwriter
```

### 2. Configure Inventory

```bash
cd ansible
cp inventory.yml.example inventory.yml
nano inventory.yml  # Edit with your OCI/ADW settings
```

### 3. Run Playbook

```bash
# Dry run first
ansible-playbook -i inventory.yml playbooks/oci-reporting-daily.yml --check

# Run full pipeline
ansible-playbook -i inventory.yml playbooks/oci-reporting-daily.yml
```

## 📋 What It Does

### Three-Stage Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│ 1. INVENTORY COLLECTION                                     │
│    • Query OCI APIs (adb_inventory.py)                     │
│    • Discover ADB resources                                │
│    • Discover Non-ADB VM clusters                          │
│    • Output: CSV files                                      │
└────────────────┬────────────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────────────┐
│ 2. DATA PERSISTENCE (ADW)                                   │
│    • Load CSVs to Autonomous Data Warehouse                │
│    • Create/update OCI_ADB_INVENTORY table                 │
│    • Create/update OCI_NON_ADB_INVENTORY table             │
│    • Validate data integrity                                │
└────────────────┬────────────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────────────┐
│ 3. REPORTING & DISTRIBUTION                                 │
│    • Generate Excel reports (pandas + xlsxwriter)          │
│    • Calculate costs (OCPU, storage, CPU cores)            │
│    • Send email with attachments                            │
│    • Output: Excel reports, email notifications            │
└─────────────────────────────────────────────────────────────┘
```

## 📂 Project Structure

```
oci-infra-reporting/
├── ansible/                           # Ansible orchestration
│   ├── playbooks/
│   │   └── oci-reporting-daily.yml   # Main entry point
│   ├── roles/
│   │   ├── oci-inventory-collection/ # Collect ADB/VM cluster inventory
│   │   ├── adw-data-loader/          # Load to ADW
│   │   ├── excel-report-generator/   # Generate Excel reports
│   │   └── email-notification/       # Send email notifications
│   ├── inventory.yml                 # Hosts & variables
│   ├── inventory.yml.example         # Configuration template
│   ├── ansible.cfg                   # Ansible settings
│   └── README.md                     # Ansible-specific docs
│
├── scripts/                           # Python inventory scripts
│   ├── adb_inventory.py              # Collect ADB resources
│   ├── non_adb_infra_inventory.py    # Collect Non-ADB resources
│   ├── adb_inventory_to_excel.py     # Generate ADB Excel reports
│   └── non_adb_inventory_to_excel.py # Generate Non-ADB Excel reports
│
├── data/                              # CSV data directory (auto-created)
├── reports/                           # Excel reports (auto-created)
├── logs/                              # Execution logs (auto-created)
├── docs/                              # Documentation
│   ├── ANSIBLE_PLAYBOOK_GUIDE.md     # Complete playbook guide
│   └── ANSIBLE_TOWER_JOB_TEMPLATE.md # Tower/AWX setup
│
├── requirements.txt                   # Python dependencies
├── README.md                          # This file
└── .github/copilot-instructions.md   # Architecture & patterns
```

## 🔧 Usage

### Local Execution

```bash
cd ansible

# Full pipeline
ansible-playbook -i inventory.yml playbooks/oci-reporting-daily.yml

# Specific stages (using tags)
ansible-playbook -i inventory.yml playbooks/oci-reporting-daily.yml --tags collect-inventory
ansible-playbook -i inventory.yml playbooks/oci-reporting-daily.yml --tags load-adw
ansible-playbook -i inventory.yml playbooks/oci-reporting-daily.yml --tags generate-reports
ansible-playbook -i inventory.yml playbooks/oci-reporting-daily.yml --tags send-email

# Verbose output
ansible-playbook -i inventory.yml playbooks/oci-reporting-daily.yml -v

# Dry run
ansible-playbook -i inventory.yml playbooks/oci-reporting-daily.yml --check
```

### Ansible Tower/AWX (Recommended for Production)

See [docs/ANSIBLE_TOWER_JOB_TEMPLATE.md](docs/ANSIBLE_TOWER_JOB_TEMPLATE.md) for complete setup.

**Quick Steps:**
1. Create Project in Tower (import from Git)
2. Create Job Template pointing to `playbooks/oci-reporting-daily.yml`
3. Create Schedule for daily execution
4. Configure credentials (OCI profile, ADW password)
5. Set up notifications (email, Slack)
6. Job runs automatically on schedule

## 🛠️ Configuration

### Environment Variables

Required:
```bash
export ADW_PASSWORD=your_adw_password
```

Optional (have defaults):
```bash
export OCI_PROFILE=DEFAULT
export ADW_WALLET_PATH=$HOME/ADWCUSG
export MAIL_FROM_EMAIL=oci-reports@company.com
export MAIL_TO=team@company.com
```

### Inventory File (ansible/inventory.yml)

```yaml
all:
  vars:
    project_base_dir: /home/opc/oci-infra-reporting
    oci_profile: DEFAULT
    adw_wallet_path: ~/ADWCUSG
    adw_user: USAGE
    # ... more variables

all:
  children:
    oci_reporting_hosts:
      hosts:
        localhost:
          ansible_connection: local
```

Full template: [ansible/inventory.yml.example](ansible/inventory.yml.example)

## 📦 Roles

### 1. oci-inventory-collection
Collects ADB and Non-ADB infrastructure inventory from OCI APIs.

**Output:** CSV files in `data/` directory
**Tags:** `collect-inventory`, `adb-collection`, `non-adb-collection`

### 2. adw-data-loader
Loads CSV inventory data to Autonomous Data Warehouse.

**Creates Tables:**
- `OCI_ADB_INVENTORY` - ADB resource details
- `OCI_NON_ADB_INVENTORY` - Non-ADB VM cluster details

**Tags:** `load-adw`, `validate`

### 3. excel-report-generator
Generates Excel reports from latest CSV inventory files.

**Output:** XLSX files in `reports/` directory
**Tags:** `generate-reports`
**Calculates:** CPU costs, storage costs, resource summaries

### 4. email-notification
Sends generated reports via email to stakeholders.

**Tags:** `send-email`
**Requires:** Postfix/mail configured on execution node

## 📚 Documentation

- [ansible/README.md](ansible/README.md) - Ansible-specific documentation
- [docs/ANSIBLE_PLAYBOOK_GUIDE.md](docs/ANSIBLE_PLAYBOOK_GUIDE.md) - Complete playbook usage guide
- [docs/ANSIBLE_TOWER_JOB_TEMPLATE.md](docs/ANSIBLE_TOWER_JOB_TEMPLATE.md) - Tower/AWX setup & configuration
- [.github/copilot-instructions.md](.github/copilot-instructions.md) - Project architecture & patterns

## 🔐 Security

### OCI Credentials
- Uses standard OCI SDK profile authentication (`~/.oci/config`)
- Specify profile via `OCI_PROFILE` environment variable

### ADW Password
**Option 1: Environment Variable (Development)**
```bash
export ADW_PASSWORD=password
```

**Option 2: OCI Vault Secret (Production - Recommended)**
```yaml
adw_secret_id: ocid1.vaultsecret.oc1.iad.xxx
adw_vault_compartment_id: ocid1.compartment.oc1.xxx
```

**Option 3: Ansible Vault (Local Execution)**
```bash
ansible-vault create group_vars/oci_reporting_hosts/vault.yml
ansible-playbook -i inventory.yml playbooks/oci-reporting-daily.yml --ask-vault-pass
```

**Option 4: Tower Vault (Ansible Tower)**
- Store password in Tower Web UI
- Automatic credential substitution at runtime

## ✅ Testing

```bash
# Syntax check
ansible-playbook -i inventory.yml playbooks/oci-reporting-daily.yml --syntax-check

# Inventory verification
ansible-inventory -i inventory.yml --list

# Connectivity test
ansible all -i inventory.yml -m ping

# Dry run (no changes)
ansible-playbook -i inventory.yml playbooks/oci-reporting-daily.yml --check

# Run specific role only
ansible-playbook -i inventory.yml playbooks/oci-reporting-daily.yml --tags load-adw
```

## 🐛 Troubleshooting

### OCI Connectivity Issues
```bash
# Verify OCI config
echo $OCI_PROFILE
ls -la ~/.oci/config

# Test OCI CLI
oci iam compartment list
```

### ADW Connection Issues
```bash
# Verify wallet
ls -la ~/ADWCUSG

# Test SQLPlus
sqlplus -version
```

### Email Delivery Issues
```bash
# Test mail command
echo "test" | mail -s "test" recipient@domain.com

# Check postfix
systemctl status postfix
```

### Ansible Debugging
```bash
# Maximum verbosity
ansible-playbook -i inventory.yml playbooks/oci-reporting-daily.yml -vvv

# Check variables
ansible localhost -i inventory.yml -m debug -a "var=adw_wallet_path"
```

## 📊 Output

### Inventory Files (CSV)
```
data/
├── adb_inventory_20240115_073000.csv      # ADB resources
└── non_adb_infra_inventory_20240115_073000.csv  # VM clusters
```

### Reports (Excel)
```
reports/
├── OCI_ADB_Inventory_20240115_073000.xlsx      # ADB report
└── OCI_NonADB_Infrastructure_20240115_073000.xlsx  # Non-ADB report
```

### Database Tables (ADW)
```
OCI_ADB_INVENTORY           -- ADB resource inventory
OCI_NON_ADB_INVENTORY       -- Non-ADB VM cluster inventory
```

## 🔄 Scheduling

### Option 1: Cron Job (if not using Tower)
```bash
crontab -e

# Daily at 7 AM
0 7 * * * cd /home/opc/oci-infra-reporting && \
  ansible-playbook -i ansible/inventory.yml \
  ansible/playbooks/oci-reporting-daily.yml >> logs/ansible-cron.log 2>&1
```

### Option 2: Ansible Tower/AWX (Recommended)
See [docs/ANSIBLE_TOWER_JOB_TEMPLATE.md](docs/ANSIBLE_TOWER_JOB_TEMPLATE.md)

**Benefits:**
- ✅ Web UI for easy management
- ✅ Credential management
- ✅ Scheduling with cron expressions
- ✅ Notifications (email, Slack, webhooks)
- ✅ Execution history & audit trail
- ✅ REST API for CI/CD integration
- ✅ Role-based access control

## 📝 Requirements

**System:**
- Python 3.6+
- Ansible 2.9+
- Linux/macOS (Windows with WSL)

**Python Packages:**
```bash
pip install -r requirements.txt
```

**OCI & ADW:**
- OCI account with API credentials
- Autonomous Data Warehouse instance
- ADW wallet downloaded locally
- USAGE user unlocked

**Email (optional):**
- Postfix or mail command configured
- OCI Email Delivery (for SMTP relay)

## 🤝 Contributing

1. Clone repository
2. Create feature branch: `git checkout -b feature/your-feature`
3. Commit changes: `git commit -am 'Add feature'`
4. Push to branch: `git push origin feature/your-feature`
5. Submit pull request

## 📄 License

[Specify License - e.g., MIT, Apache 2.0, Proprietary]

## 📧 Support

For questions or issues:
1. Check [ansible/README.md](ansible/README.md)
2. Review [docs/ANSIBLE_PLAYBOOK_GUIDE.md](docs/ANSIBLE_PLAYBOOK_GUIDE.md)
3. See [docs/ANSIBLE_TOWER_JOB_TEMPLATE.md](docs/ANSIBLE_TOWER_JOB_TEMPLATE.md) for Tower setup
4. Open an issue in the repository

## 🎯 Next Steps

1. ✅ Review project documentation
2. ✅ Configure `ansible/inventory.yml` with your environment
3. ✅ Set required environment variables
4. ✅ Run playbook locally: `ansible-playbook -i inventory.yml playbooks/oci-reporting-daily.yml --check`
5. ✅ Set up in Ansible Tower/AWX for production scheduling
6. ✅ Monitor logs and reports

---

**Version:** 2.0 - Ansible Implementation  
**Last Updated:** 2024  
**Status:** Production Ready

## Suggestions for a good README

Every project is different, so consider which of these sections apply to yours. The sections used in the template are suggestions for most open source projects. Also keep in mind that while a README can be too long and detailed, too long is better than too short. If you think your README is too long, consider utilizing another form of documentation rather than cutting out information.

## Name
Choose a self-explaining name for your project.

## Description
Let people know what your project can do specifically. Provide context and add a link to any reference visitors might be unfamiliar with. A list of Features or a Background subsection can also be added here. If there are alternatives to your project, this is a good place to list differentiating factors.

## Badges
On some READMEs, you may see small images that convey metadata, such as whether or not all the tests are passing for the project. You can use Shields to add some to your README. Many services also have instructions for adding a badge.

## Visuals
Depending on what you are making, it can be a good idea to include screenshots or even a video (you'll frequently see GIFs rather than actual videos). Tools like ttygif can help, but check out Asciinema for a more sophisticated method.

## Installation
Within a particular ecosystem, there may be a common way of installing things, such as using Yarn, NuGet, or Homebrew. However, consider the possibility that whoever is reading your README is a novice and would like more guidance. Listing specific steps helps remove ambiguity and gets people to using your project as quickly as possible. If it only runs in a specific context like a particular programming language version or operating system or has dependencies that have to be installed manually, also add a Requirements subsection.

## Usage
Use examples liberally, and show the expected output if you can. It's helpful to have inline the smallest example of usage that you can demonstrate, while providing links to more sophisticated examples if they are too long to reasonably include in the README.

## Support
Tell people where they can go to for help. It can be any combination of an issue tracker, a chat room, an email address, etc.

## Roadmap
If you have ideas for releases in the future, it is a good idea to list them in the README.

## Contributing
State if you are open to contributions and what your requirements are for accepting them.

For people who want to make changes to your project, it's helpful to have some documentation on how to get started. Perhaps there is a script that they should run or some environment variables that they need to set. Make these steps explicit. These instructions could also be useful to your future self.

You can also document commands to lint the code or run tests. These steps help to ensure high code quality and reduce the likelihood that the changes inadvertently break something. Having instructions for running tests is especially helpful if it requires external setup, such as starting a Selenium server for testing in a browser.

## Authors and acknowledgment
Show your appreciation to those who have contributed to the project.

## License
For open source projects, say how it is licensed.

## Project status
If you have run out of energy or time for your project, put a note at the top of the README saying that development has slowed down or stopped completely. Someone may choose to fork your project or volunteer to step in as a maintainer or owner, allowing your project to keep going. You can also make an explicit request for maintainers.
