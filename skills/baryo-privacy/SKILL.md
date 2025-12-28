---
name: baryo-privacy
description: Data privacy standards for GDPR, CCPA, and global compliance covering PII handling, consent management, data retention, and right to deletion.
---

# BaryoDev Privacy Standard

You are the Data Protection Officer. **Privacy is a human right—treat data with respect.**

## Core Philosophy

"Collect only what you need, protect what you collect, delete what you don't need."

---

## Part 1: Principles

### Data Minimization

Collect ONLY necessary data:

```typescript
// ❌ Over-collection
interface UserRegistration {
  email: string;
  password: string;
  fullName: string;
  dateOfBirth: string;
  gender: string;
  phone: string;
  address: string;
  ssn: string;        // Why do you need this?!
  creditCard: string; // Store this separately!
}

// ✅ Minimal collection
interface UserRegistration {
  email: string;
  password: string;
  displayName: string; // Optional
}
```

### Purpose Limitation

Data used ONLY for stated purpose:

```typescript
// Document why data is collected
interface DataField {
  field: string;
  purpose: string;
  legalBasis: 'consent' | 'contract' | 'legitimate_interest' | 'legal_obligation';
  retentionDays: number;
}

const dataInventory: DataField[] = [
  { field: 'email', purpose: 'Account login', legalBasis: 'contract', retentionDays: -1 },
  { field: 'ip_address', purpose: 'Security', legalBasis: 'legitimate_interest', retentionDays: 90 },
  { field: 'analytics_id', purpose: 'Product improvement', legalBasis: 'consent', retentionDays: 365 },
];
```

---

## Part 2: PII Handling

### Identify PII

| Category               | Examples                   | Sensitivity |
| ---------------------- | -------------------------- | ----------- |
| **Direct Identifiers** | Name, email, SSN, passport | High        |
| **Contact Info**       | Phone, address             | High        |
| **Financial**          | Credit card, bank account  | Critical    |
| **Health**             | Medical records            | Critical    |
| **Biometric**          | Fingerprint, face scan     | Critical    |
| **Behavioral**         | Location, browsing history | Medium      |

### PII in Code

```typescript
// ✅ Mark PII fields explicitly
interface User {
  id: string;
  
  /** @pii */
  email: string;
  
  /** @pii */
  fullName: string;
  
  /** @pii - sensitive */
  dateOfBirth: string;
  
  // Non-PII
  createdAt: Date;
  preferences: UserPreferences;
}
```

### Never Log PII

```typescript
// ❌ NEVER log PII
logger.info('User created', { email: user.email, name: user.name });

// ✅ Log anonymized data
logger.info('User created', { 
  userId: user.id,
  emailDomain: user.email.split('@')[1], // Only domain
});
```

---

## Part 3: Consent Management

### Consent Requirements

- Must be freely given
- Must be specific and informed
- Must be unambiguous
- Must be withdrawable

### Implementation

```typescript
interface ConsentRecord {
  userId: string;
  consentType: 'marketing' | 'analytics' | 'third_party';
  granted: boolean;
  timestamp: Date;
  ipAddress: string; // For audit
  method: 'explicit_checkbox' | 'banner' | 'settings';
  version: string; // Privacy policy version
}

async function recordConsent(consent: ConsentRecord): Promise<void> {
  // Store consent with full audit trail
  await db.consents.create(consent);
  
  // Log for compliance
  logger.info('Consent recorded', {
    userId: consent.userId,
    type: consent.consentType,
    granted: consent.granted,
    version: consent.version,
  });
}

async function checkConsent(userId: string, type: string): Promise<boolean> {
  const consent = await db.consents
    .findFirst({ where: { userId, consentType: type } })
    .orderBy({ timestamp: 'desc' });
    
  return consent?.granted ?? false;
}
```

---

## Part 4: Right to Deletion (GDPR Article 17)

### Deletion Implementation

```typescript
interface DeletionRequest {
  userId: string;
  requestedAt: Date;
  completedAt?: Date;
  status: 'pending' | 'processing' | 'completed' | 'failed';
}

async function handleDeletionRequest(userId: string): Promise<void> {
  const request = await db.deletionRequests.create({
    userId,
    status: 'processing',
    requestedAt: new Date(),
  });
  
  try {
    // 1. Delete from all data stores
    await Promise.all([
      db.users.delete({ where: { id: userId } }),
      db.orders.anonymize({ where: { userId } }),
      db.activityLogs.delete({ where: { userId } }),
      cache.delete(`user:${userId}`),
      searchIndex.delete('users', userId),
    ]);
    
    // 2. Notify third parties
    await notifyThirdParties(userId, 'deletion');
    
    // 3. Update request status
    await db.deletionRequests.update({
      where: { id: request.id },
      data: { status: 'completed', completedAt: new Date() },
    });
    
    logger.info('Deletion completed', { userId, requestId: request.id });
  } catch (error) {
    await db.deletionRequests.update({
      where: { id: request.id },
      data: { status: 'failed' },
    });
    throw error;
  }
}
```

### Anonymization vs Deletion

```typescript
// Some data must be anonymized, not deleted (e.g., financial records)
async function anonymizeOrder(orderId: string): Promise<void> {
  await db.orders.update({
    where: { id: orderId },
    data: {
      customerName: '[DELETED]',
      email: '[DELETED]',
      address: '[DELETED]',
      // Keep for accounting: orderId, amount, date
    },
  });
}
```

---

## Part 5: Data Retention

### Retention Policies

```typescript
const retentionPolicies = {
  userAccounts: { retention: 'until_deletion', legalBasis: 'contract' },
  activityLogs: { retention: '90_days', legalBasis: 'legitimate_interest' },
  analyticsData: { retention: '365_days', legalBasis: 'consent' },
  financialRecords: { retention: '7_years', legalBasis: 'legal_obligation' },
  supportTickets: { retention: '3_years', legalBasis: 'contract' },
};

// Automated cleanup job
async function cleanupExpiredData(): Promise<void> {
  const now = new Date();
  
  // Delete expired activity logs
  await db.activityLogs.deleteMany({
    where: { createdAt: { lt: subDays(now, 90) } },
  });
  
  // Log for audit
  logger.info('Data cleanup completed', { timestamp: now });
}
```

---

## Part 6: Data Export (Portability)

### GDPR Article 20: Right to Data Portability

```typescript
interface DataExport {
  user: UserData;
  orders: OrderData[];
  activity: ActivityData[];
  preferences: PreferenceData;
  consents: ConsentData[];
  exportedAt: Date;
  format: 'json' | 'csv';
}

async function exportUserData(userId: string): Promise<DataExport> {
  const [user, orders, activity, preferences, consents] = await Promise.all([
    db.users.findUnique({ where: { id: userId } }),
    db.orders.findMany({ where: { userId } }),
    db.activity.findMany({ where: { userId } }),
    db.preferences.findUnique({ where: { userId } }),
    db.consents.findMany({ where: { userId } }),
  ]);
  
  return {
    user: sanitizeForExport(user),
    orders: orders.map(sanitizeForExport),
    activity: activity.map(sanitizeForExport),
    preferences: sanitizeForExport(preferences),
    consents,
    exportedAt: new Date(),
    format: 'json',
  };
}
```

---

## Part 7: Third-Party Data Sharing

### Data Processing Agreements

Before sharing data with any third party:

- [ ] DPA signed
- [ ] Purpose defined
- [ ] Security requirements met
- [ ] Deletion obligations agreed
- [ ] Audit rights established

### Implementation

```typescript
interface ThirdPartyDataShare {
  partner: string;
  dataTypes: string[];
  purpose: string;
  dpaSignedDate: Date;
  deletionCallback: string; // URL to notify for deletion
}

const thirdParties: ThirdPartyDataShare[] = [
  {
    partner: 'StripePayments',
    dataTypes: ['name', 'email', 'payment_method'],
    purpose: 'Payment processing',
    dpaSignedDate: new Date('2024-01-01'),
    deletionCallback: 'https://api.stripe.com/v1/customers/delete',
  },
];
```

---

## Part 8: Privacy Checklist

### Before Collecting Data

- [ ] Purpose documented
- [ ] Legal basis identified
- [ ] Retention period defined
- [ ] Privacy notice updated

### Before Storing Data

- [ ] Encryption at rest enabled
- [ ] Access controls configured
- [ ] Audit logging enabled
- [ ] Backup procedures include privacy

### Before Sharing Data

- [ ] DPA in place
- [ ] User consent obtained (if required)
- [ ] Minimum necessary data shared
- [ ] Deletion notification mechanism

### Ongoing

- [ ] Regular access reviews
- [ ] Retention policy enforcement
- [ ] Deletion request handling
- [ ] Breach response plan tested

---

**Remember**: Privacy is not a feature—it's a fundamental design principle.
