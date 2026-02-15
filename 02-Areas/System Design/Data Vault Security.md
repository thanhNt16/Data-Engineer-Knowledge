---
tags: [security, data-vault, secrets-management, encryption, authentication, iam]
date: 2026-02-15
status: learning
---

# Data Vault

## Overview

A Data Vault is a secure system for storing, managing, and controlling access to sensitive information such as passwords, API keys, certificates, and other secrets.

**Key Insight**: "Data Vault shifts trust from people to systems - no more 'what's the password for production?'"

---

## Core Concepts

### 1. Secrets vs. Credentials

| Type | Description | Example | Storage |
|-------|-------------|----------|----------|
| **Secret** | Passwords, API keys, certificates | Encrypted | Data Vault |
| **Credential** | Username/password pairs | Plain text | Application config |

**Python Example**:
```python
import os
import json
from typing import Dict, Any

class SecretManager:
    def __init__(self):
        # In-memory secrets (for development only!)
        self.secrets: Dict[str, Any] = {}

    def load_from_env(self):
        """Load secrets from environment variables"""
        self.secrets['db_password'] = os.getenv('DB_PASSWORD')
        self.secrets['api_key'] = os.getenv('API_KEY')
        return self.secrets

# DONT hardcode secrets!
# BAD:
config = {"db_password": "hardcoded_password", "api_key": "hardcoded_key"}

# GOOD:
manager = SecretManager()
secrets = manager.load_from_env()  # Loaded from .env file or environment
```

### 2. Static vs. Dynamic Secrets

| Aspect | Static Secrets | Dynamic Secrets |
|---------|------------------|------------------|
| **Storage** | Hardcoded in config files | Retrieved from vault at runtime |
| **Rotation** | Manual file update | Automated rotation (vault) |
| **Access Control** | File permissions | Vault policies |
| **Security** | Low (if file compromised) | High (encrypted, auditable) |
| **Scalability** | Difficult across envs | Easy (single source of truth) |

**SQL Example**: Static vs Dynamic
```sql
-- Static: Stored in database (bad practice)
CREATE TABLE app_config (
    app_id INT PRIMARY KEY,
    db_password TEXT,  -- Stored as plain text!
    api_key TEXT
);

INSERT INTO app_config VALUES (1, 'mysecretpassword', 'myapikey');

-- Dynamic: Retrieved from vault at runtime (best practice)
-- Application calls vault service API or uses secret injection
```

---

## Data Vault Architecture

### 1. HashiCorp Vault

**Components**:
- **Storage Backend**: Consistent, encrypted storage
- **Secrets Engine**: Key-value store with versioning
- **Authentication**: LDAP, OIDC, Token, GitHub
- **HTTP/S**: HTTPS with TLS
- **UI**: Web UI for managing secrets
- **CLI**: Command-line interface

**Features**:
- **Versioning**: Track all secret versions
- **Rollback**: Restore previous versions
- **Dynamic Secrets**: Generate secrets on demand
- **Audit Logging**: Who accessed what and when
- **Revocation**: Immediately revoke access
- **Lease Management**: Time-based access control
- **Key/Value Storage**: Nested secret storage

**Python Example: HashiCorp Vault Client**

```python
from hvac import Client
import json

class VaultSecrets:
    def __init__(self, vault_addr: str = 'http://localhost:8200'):
        # Create HashiCorp Vault client
        self.client = Client(
            url=vault_addr,
            token='root-token'  # Root token for admin access
        )

    def get_database_password(self, path: str) -> str:
        """Retrieve database password from vault"""
        secret_path = f"database/data/prod/{path}"
        try:
            # Read secret (version 2 API)
            read_response = self.client.secrets.kv.v2.read_secret_version(
                path=secret_path,
                version='2'  # Latest version
            )
            return read_response['data']['data']['value']
        except Exception as e:
            print(f"Vault error: {str(e)}")
            return None

    def create_api_key(self, app_name: str, ttl: int = 3600) -> str:
        """Generate and store API key with time-to-live"""
        secret_path = f"apps/{app_name}/api-key"
        api_key = ''.join([chr(random.randint(33, 126)) for _ in range(32)])

        write_response = self.client.secrets.kv.v2.create_secret_version(
            path=secret_path,
            secret={'api_key': api_key},
            options={'ttl': str(ttl)},  # 1 hour TTL
        )

        return write_response['data']['data']['value']

    def rotate_database_password(self, path: str, new_password: str) -> None:
        """Rotate database password (create new version)"""
        secret_path = f"database/data/prod/{path}"
        timestamp = datetime.datetime.now().isoformat()

        write_response = self.client.secrets.kv.v2.create_secret_version(
            path=secret_path,
            secret={'password': new_password, 'rotated_at': timestamp},
            custom_metadata={'rotation_reason': 'scheduled_rotation'}
        )

        print(f"Password rotated: {write_response['data']['data']['version']}")

# Usage
vault = VaultSecrets()

# Get current database password
db_password = vault.get_database_password('mysql')

print(f"Retrieved from vault: {db_password[:3]}***")

# Generate new API key (1 hour TTL)
api_key = vault.create_api_key('my-app', ttl=3600)
print(f"Generated API key: {api_key[:10]}***")

# Rotate password (new version)
vault.rotate_database_password('mysql', 'new_secure_password_123!')
```

### 2. AWS Secrets Manager

**Features**:
- **KMS (Key Management Service)**: Encryption as a service
- **Secrets Manager**: Secure secret storage
- **Parameter Store**: Secure config storage
- **Certificate Manager**: SSL/TLS certificate management
- **Secrets Hub**: Share secrets across AWS accounts
- **Rotation**: Automatic secret rotation
- **Audit Trails**: CloudTrail integration

**Python Example: AWS Secrets Manager**

```python
import boto3
import base64
import json

# Create AWS Secrets Manager client
client = boto3.client('secretsmanager',
    region_name='us-west-2')

def create_database_secret(self, secret_name: str, password: str) -> str:
    """Store database credential in Secrets Manager"""
    secret_value = json.dumps({
        'username': 'db_user',
        'password': password,
        'engine': 'mysql',
        'host': 'db.prod.example.com'
    })

    create_response = client.create_secret(
        Name=secret_name,
        SecretString=secret_value,
        Description='Production MySQL database password',
        Tags=[
            {'Key': 'Environment', 'Value': 'production'},
            {'Key': 'Application', 'Value': 'mysql'}
        ],
        KmsKeyId='alias/my-secrets-key',  # Encrypt with KMS key
        SecretBinary=base64.b64encode(password).decode('utf-8')  # Encrypt password
    )

    return create_response['ARN']

def retrieve_database_password(self, secret_name: str) -> dict:
    """Retrieve database password from Secrets Manager"""
    response = client.get_secret_value(
        SecretId=secret_name
    )

    secret_value = json.loads(response['SecretString'])

    return {
        'username': secret_value['username'],
        'password': secret_value['password'],
        'host': secret_value['host'],
        'engine': secret_value['engine']
    }

def rotate_database_password(self, secret_name: str, new_password: str) -> None:
    """Rotate database password (update version)"""
    new_secret_value = json.dumps({
        'username': 'db_user',
        'password': new_password,
        'engine': 'mysql',
        'host': 'db.prod.example.com',
        'rotated_at': datetime.datetime.now().isoformat()
    })

    # Update secret
    client.put_secret_value(
        SecretId=secret_name,
        SecretString=new_secret_value
    )

# Usage
arn = create_database_secret(
    secret_name='prod-mysql-primary',
    password='SecurePassword123!'
)
print(f"Secret stored in AWS: {arn}")

# Retrieve password
creds = retrieve_database_password('prod-mysql-primary')
print(f"Username: {creds['username']}, Host: {creds['host']}")

# Rotate password (new version)
rotate_database_password('prod-mysql-primary', 'NewSecurePassword456!')
```

---

## Security Best Practices

### 1. Secrets Management

| Best Practice | Description | Anti-Pattern |
|--------------|-------------|----------------|
| **Vault Storage** | Store secrets in vault (HashiCorp, AWS) | Hardcode in config |
| **Environment Variables** | Load from .env files | Hardcode in code |
| **Encrypted at Rest** | Secrets encrypted in storage | Plain text in DB |
| **Minimum Access** | App only gets secrets it needs | All secrets in one file |
| **Audit Logging** | Log all secret access | No logging |
| **Rotation** | Regular secret rotation | Never rotate | 
| **Secure Transmission** | TLS/HTTPS for vault API | Send in clear text |

### 2. Application Integration

| Pattern | Description | Example |
|---------|-------------|----------|
| **Vault Agent Pattern** | Sidecar proxy for secrets | Service mesh with sidecars |
| **Provider Pattern** | Application queries vault at runtime | Secrets as service |
| **Pull Secrets** | Vault client fetches on startup | Push secrets (rare) |
| **Lease Management** | Temporary access for operations | Long-lived credentials |

**Python Example: Vault Agent Pattern**

```python
from typing import Optional
import requests

class VaultClient:
    def __init__(self, vault_url: str):
        self.vault_url = vault_url

    def get_secret(self, path: str) -> Optional[str]:
        """Fetch secret from vault"""
        response = requests.get(
            f"{self.vault_url}/v1/{path}",
            headers={'X-Vault-Token': self.vault_token}
        )
        
        if response.status_code == 200:
            return response.json()['data']['value']
        else:
            return None

class DatabaseService:
    def __init__(self, vault_client: VaultClient):
        self.vault_client = vault_client

    def connect(self) -> bool:
        """Connect to database using credentials from vault"""
        # Fetch credentials from vault
        db_host = self.vault_client.get_secret('database/host')
        db_password = self.vault_client.get_secret('database/password')
        
        if db_host and db_password:
            # Connect to database
            print(f"Connecting to {db_host} with vault-retrieved password")
            # In real implementation: sqlalchemy.create_engine(...)
            return True
        else:
            print("Failed to retrieve credentials from vault")
            return False

# Usage
vault = VaultClient(vault_url='http://localhost:8200')
vault.vault_token = 's.XXXX.vault-token'  # From auth service

db_service = DatabaseService(vault)
db_service.connect()
```

---

## Encryption

### 1. Encryption at Rest

**Definition**: Secrets encrypted when stored (AES-256, ChaCha20-Poly1305).

**Python Example**: Encryption/Decryption

```python
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
import base64
import os

class SecretEncryption:
    def __init__(self, password: str = None):
        # Derive encryption key from password
        if password:
            kdf = PBKDF2HMAC(
                password.encode(),
                salt=b'salt_value',  # Should come from vault/env
                iterations=480000,
                dklen=32,
                algorithm='sha256'
            )
            self.key = kdf.derive(32)
        else:
            # Load key from vault
            self.key = os.getenv('ENCRYPTION_KEY')

    def encrypt_secret(self, plaintext: str) -> str:
        """Encrypt secret (AES-256)"""
        f = Fernet(self.key)
        encrypted = f.encrypt(plaintext.encode())
        return base64.b64encode(encrypted).decode('utf-8')

    def decrypt_secret(self, encrypted_text: str) -> str:
        """Decrypt secret"""
        f = Fernet(self.key)
        decrypted = f.decrypt(base64.b64decode(encrypted_text))
        return decrypted.decode('utf-8')

# Usage
vault = SecretEncryption(password='my-vault-password')

# Store encrypted password
encrypted_pw = vault.encrypt_secret('SecurePassword123!')
print(f"Encrypted password: {encrypted_pw}")

# Retrieve and decrypt
decrypted_pw = vault.decrypt_secret(encrypted_pw)
print(f"Decrypted password: {decrypted_pw}")
```

### 2. Hashing vs. Encryption

| Operation | Hashing | Encryption |
|----------|---------|-----------|
| **Purpose** | Verify data integrity | Protect data confidentiality |
| **Reversible** | ❌ No | ✅ Yes |
| **Performance** | Fast | Slower |
| **Storage** | Fixed size | Variable (based on data) |
| **Use Case** | Password verification, deduplication | Secret storage, data at rest |

**SQL Example**: Password Hashing

```sql
-- Hash passwords (bcrypt - one-way hash)
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(100) UNIQUE,
    password_hash VARCHAR(255),  -- One-way hash (bcrypt)
    salt VARCHAR(100)
);

-- Store password hash (never decrypt)
INSERT INTO users (username, password_hash, salt)
VALUES ('alice', '$2a$10$...', 'random_salt_value');

-- Verify password (hash and compare)
-- In Python: bcrypt.hashpw(plain_password.encode(), salt.encode())
```

---

## Access Control (IAM)

### 1. Role-Based Access Control (RBAC)

| Role | Permissions | Description |
|------|-------------|-------------|
| **Admin** | All permissions (read, write, delete, rotate) | Full system access |
| **Operator** | Read, write, execute jobs | Operational tasks only |
| **Analyst** | Read, query data | Business intelligence only |
| **Auditor** | Read, audit logs | Audit and compliance only |
| **Viewer** | Read (limited) | View-only access |

**Python Example: RBAC Check**

```python
from typing import List, Dict

class User:
    def __init__(self, username: str, roles: List[str]):
        self.username = username
        self.roles = roles

def check_permission(user: User, resource: str, action: str) -> bool:
    """Check if user has permission for action on resource"""
    # Role-based permissions mapping
    role_permissions = {
        'admin': ['read', 'write', 'delete', 'rotate', 'admin'],
        'operator': ['read', 'write', 'execute'],
        'analyst': ['read', 'query'],
        'auditor': ['read', 'audit_logs'],
        'viewer': ['read']
    }
    
    # Check all user's roles
    for role in user.roles:
        if action in role_permissions.get(role, []):
            return True
    
    return False

# Usage
admin_user = User('admin_user', ['admin'])
viewer_user = User('viewer_user', ['viewer'])

# Admin can delete
assert check_permission(admin_user, 'fact_sales', 'delete') == True

# Viewer cannot delete
assert check_permission(viewer_user, 'fact_sales', 'delete') == False
```

### 2. Attribute-Based Access Control (ABAC)

| Attribute | Description | Example |
|-----------|-------------|----------|
| **Department** | Sales, Marketing, Engineering | Department code |
| **Sensitivity** | Public, Internal, Confidential, Secret | Data classification |
| **Environment** | Production, Staging, Development | Deployment environment |
| **Project** | Project code | Business project |

**SQL Example**: ABAC Implementation

```sql
-- Create users with attributes
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(100) UNIQUE,
    department VARCHAR(50),
    role VARCHAR(50),
    sensitivity VARCHAR(20)
);

-- Create data resources
CREATE TABLE fact_sales (
    sales_id BIGINT PRIMARY KEY,
    user_id INT REFERENCES users(user_id),
    sales_amount DECIMAL(10,2),
    sales_date TIMESTAMP,
    sensitivity VARCHAR(20) DEFAULT 'internal'
);

-- Policy: Sales team can access all sensitivity levels
CREATE POLICY sales_team_policy ON fact_sales
USING (
    department = 'Sales'
)
FOR SELECT
TO ROLE sales_analyst;

-- Policy: Only users with appropriate clearance can access confidential data
CREATE POLICY confidential_data_policy ON fact_sales
FOR SELECT
TO ROLE sales_manager
WHERE sensitivity = 'confidential'
WITH CHECK (EXISTS (
    SELECT 1 FROM users
    WHERE user_id = current_user_id()
    AND (sensitivity = 'confidential' OR role = 'admin')
));
```

---

## Audit & Compliance

### 1. Audit Logging

**What to Log**:
- Secret access events (who, what, when, from where)
- Secret rotation events
- Failed access attempts
- Policy violations

**SQL Example**: Audit Log Table

```sql
CREATE TABLE audit_log (
    log_id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    user_id INT,
    action VARCHAR(50),           -- read_secret, rotate_secret
    resource_type VARCHAR(100), -- database, api_key, certificate
    resource_id VARCHAR(255),
    access_granted BOOLEAN,
    access_denied_reason VARCHAR(255),
    source_ip VARCHAR(45),
    user_agent VARCHAR(500)
);

-- Log secret access
INSERT INTO audit_log (user_id, action, resource_type, resource_id, access_granted)
VALUES (
    123,                              -- User who accessed
    'read_secret',                   -- Action performed
    'database',                       -- Resource type
    'prod-mysql-primary',             -- Resource identifier
    TRUE                              -- Access granted
);
```

### 2. Compliance Frameworks

| Framework | Description | Applicability |
|----------|-------------|--------------|
| **GDPR** | EU data protection | Personal data storage, right to deletion |
| **PCI DSS** | Payment card data | Credit card encryption, network security |
| **HIPAA** | Health data | Patient data access controls, audit trails |
| **SOC 2** | Security best practices | Incident response, access controls |
| **ISO 27001** | Information security | Risk management, compliance |

---

## Secret Rotation

### 1. Automatic Rotation

**Benefits**:
- Reduces risk from leaked secrets
- Compliance with security standards (rotate every 90 days)
- Automated, no manual intervention

**Rotation Strategy**:
```python
import secrets
from datetime import datetime, timedelta

class SecretRotator:
    def __init__(self):
        self.ttl_days = 90  # Rotate every 90 days

    def generate_new_password(self) -> str:
        """Generate new strong password"""
        import random
        import string
        chars = string.ascii_letters + string.digits + string.punctuation
        password = ''.join(random.choice(chars) for _ in range(32))
        return password

    def rotate_database_password(self, secret_name: str) -> dict:
        """Rotate database password in vault"""
        new_password = self.generate_new_password()

        # Update in vault
        # In real implementation: vault_client.secrets.kv.v2.update_secret_version(...)
        secret_data = {
            'password': new_password,
            'rotated_at': datetime.now().isoformat(),
            'rotated_by': 'automation_system'
        }

        print(f"Rotated {secret_name}: {new_password[:10]}***")
        return secret_data

# Usage
rotator = SecretRotator()

# Generate new password (simulating vault update)
new_creds = rotator.rotate_database_password('prod-mysql-primary')
```

### 2. Key Rotation

**When to Rotate**:
- After suspected compromise
- Regular schedule (90 days)
- Personnel changes (employee leaves)
- After security incident

**SQL Example**: Track Key Rotation

```sql
-- Track active and expired keys
CREATE TABLE api_keys (
    key_id INT PRIMARY KEY,
    key_name VARCHAR(100),
    encrypted_key VARCHAR(255),  -- Vault path to encrypted key
    status ENUM('active', 'expired', 'compromised'),
    created_at TIMESTAMP,
    expires_at TIMESTAMP,
    last_rotated_at TIMESTAMP
);

-- Mark key as expired
UPDATE api_keys
SET status = 'expired', expires_at = NOW()
WHERE key_id = 123;

-- Record rotation
UPDATE api_keys
SET status = 'active', last_rotated_at = NOW()
WHERE key_id = 123;
```

---

## Interview Questions

**Q1: What is a data vault and why use it?**

**A1**: A data vault is a secure system for storing and managing secrets (passwords, API keys, certificates). I would use it because:
- **Security**: Secrets are encrypted and centrally managed
- **Access Control**: Fine-grained permissions and audit logging
- **Rotation**: Automated secret rotation reduces risk
- **Compliance**: Meets security standards (SOC 2, PCI DSS)
- **Scalability**: Easier to manage secrets across applications
- **Avoids Hardcoding**: Removes secrets from code and configuration files

**Q2: How does secret rotation improve security?**

**A2**: Secret rotation limits the window of vulnerability. If a secret is leaked, rotating it limits the damage to the time between rotations. For example, if a database password is leaked but rotated every 90 days, an attacker can only use it for up to 90 days. Automatic rotation ensures regular updates without manual intervention.

**Q3: What's the difference between symmetric and asymmetric encryption in data vault context?**

**A3**:
- **Symmetric Encryption** (AES, ChaCha20): Same key for encryption and decryption. Faster, suitable for data at rest (secrets in vault). Key management is critical.
- **Asymmetric Encryption** (RSA, ECC): Public/private key pairs. Slower, used for key exchange, digital signatures. Used for encrypting the vault's master key or for transporting secrets.

**Q4: How would you implement role-based access control (RBAC) for a data vault?**

**A4**: I would implement RBAC with three components:
1. **Role Definitions**: Define roles (admin, operator, analyst, viewer) with associated permissions
2. **User-Role Assignment**: Store user roles in vault as part of user identity
3. **Permission Checking Middleware**: When a user requests access to a secret, check their role against the required permission. Use hierarchical roles (admin inherits all permissions).
4. **Audit Logging**: Log all secret access events with user, role, resource, timestamp
5. **Least Privilege**: Grant only the minimum permissions needed to perform a task

**Q5: What is the principle of least privilege and why is it important?**

**A5**: The principle of least privilege states that users should only have the minimum permissions necessary to perform their tasks, and only for the time needed. This limits the blast radius if a user's account is compromised. It's important because it reduces security risk by minimizing potential damage from unauthorized access.

**Q6: How would you handle audit logging in a distributed system?**

**A6**: I would implement centralized audit logging:
1. **Immutable Logs**: Write audit logs to immutable storage (append-only logs) to prevent tampering
2. **Structured Logs**: Use consistent format (JSON) with all relevant fields (user, action, resource, timestamp, source IP)
3. **Async Logging**: Write logs asynchronously to avoid performance impact
4. **Log Aggregation**: Centralize logs from all components (vault, applications) to a single SIEM (Security Information and Event Management) system
5. **Alerting**: Set up real-time alerts for suspicious patterns (unusual access attempts, privilege escalation)
6. **Retention**: Define log retention policies based on compliance requirements

---

## Related Notes

- [[Database Fundamentals]] - Transactions, isolation, consistency
- [[System Design]] - Scalability, CAP theorem, distributed systems
- [[Orchestration/Orchestration Workflows & Scheduling]] - Pipeline orchestration patterns
- [[Data Quality/Data Quality - Testing & Metrics]] - Data protection, integrity

---

## Resources

- [HashiCorp Vault Documentation](https://developer.hashicorp.com/vault)
- [AWS Secrets Manager Documentation](https://docs.aws.amazon.com/secretsmanager/)
- [OWASP Cheat Sheet Series: Secrets Management](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [CIS Controls](https://www.cisecurity.org/cis-controls)

---

**Progress**: 🟡 Learning (concepts understood, need implementation practice)

**Next Steps**:
- [ ] Set up HashiCorp Vault locally (for development)
- [ ] Implement AWS Secrets Manager integration
- [ ] Create secrets rotation automation
- [ ] Implement RBAC for application access
- [ ] Set up audit logging and SIEM integration
- [ ] Practice encryption/decryption operations
- [ ] Implement key rotation policies
