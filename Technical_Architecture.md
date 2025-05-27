# Capita Financial Platform - Technical Architecture

## Architecture Overview

Capita is built using a modern, cloud-native microservices architecture designed for scalability, security, and maintainability. The platform consists of three main client applications (Web, iOS, Android) communicating with a robust backend API ecosystem.

**Primary Brand Color:** #0C0950

---

## System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        Client Applications                      │
├─────────────────┬─────────────────┬─────────────────────────────┤
│   Web App       │   iOS App       │   Android App               │
│   (React.js)    │   (Swift/       │   (Kotlin/                  │
│                 │   SwiftUI)      │   Jetpack Compose)          │
└─────────────────┴─────────────────┴─────────────────────────────┘
                            │
                    ┌───────────────┐
                    │  Load Balancer │
                    │  (AWS ALB)     │
                    └───────────────┘
                            │
┌─────────────────────────────────────────────────────────────────┐
│                     API Gateway                                 │
│                   (AWS API Gateway)                            │
└─────────────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────────┐
│                    Microservices Layer                         │
├─────────────┬─────────────┬─────────────┬─────────────────────┤
│ Auth        │ User        │ Financial   │ AI/Claude           │
│ Service     │ Service     │ Service     │ Service             │
├─────────────┼─────────────┼─────────────┼─────────────────────┤
│ Banking     │ Analytics   │ Notification│ Subscription        │
│ Service     │ Service     │ Service     │ Service             │
└─────────────┴─────────────┴─────────────┴─────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────────┐
│                      Data Layer                                │
├─────────────┬─────────────┬─────────────┬─────────────────────┤
│ PostgreSQL  │ Redis       │ S3 Storage  │ ElasticSearch       │
│ (Primary)   │ (Cache)     │ (Files)     │ (Search/Analytics)  │
└─────────────┴─────────────┴─────────────┴─────────────────────┘
```

---

## Backend Architecture

### Technology Stack

#### Core Framework
- **Language:** Node.js with TypeScript
- **Framework:** Express.js with Helmet for security
- **API Style:** RESTful APIs with GraphQL for complex queries
- **Documentation:** OpenAPI 3.0 (Swagger)

#### Database Layer
- **Primary Database:** PostgreSQL 14+
  - ACID compliance for financial data
  - Advanced indexing for performance
  - Row-level security for data isolation
  - Automated backups and point-in-time recovery

- **Caching Layer:** Redis 6+
  - Session storage
  - API response caching
  - Real-time data caching
  - Rate limiting storage

- **Search Engine:** ElasticSearch 8+
  - Full-text search capabilities
  - Analytics and reporting
  - Log aggregation and monitoring

#### File Storage
- **Primary Storage:** AWS S3
  - Document storage (PDFs, images)
  - Backup and archival
  - CDN integration via CloudFront

### Microservices Architecture

#### 1. Authentication Service
**Responsibilities:**
- User registration and login
- JWT token management
- Multi-factor authentication
- Password reset and recovery
- OAuth integration (Google, Apple)

**Technology:**
- Express.js with Passport.js
- bcrypt for password hashing
- speakeasy for TOTP
- AWS Cognito integration

**Endpoints:**
```
POST /auth/register
POST /auth/login
POST /auth/logout
POST /auth/refresh
POST /auth/forgot-password
POST /auth/reset-password
POST /auth/verify-email
POST /auth/enable-2fa
POST /auth/verify-2fa
```

#### 2. User Service
**Responsibilities:**
- User profile management
- Preferences and settings
- Subscription management
- Account verification

**Technology:**
- Express.js with TypeORM
- Joi for validation
- Multer for file uploads

**Endpoints:**
```
GET /users/profile
PUT /users/profile
GET /users/preferences
PUT /users/preferences
POST /users/upload-document
DELETE /users/account
GET /users/subscription
PUT /users/subscription
```

#### 3. Financial Service
**Responsibilities:**
- Financial goal management
- Income and expense tracking
- Investment portfolio management
- FIRE calculations
- Tax planning

**Technology:**
- Express.js with Prisma ORM
- Decimal.js for precise calculations
- node-cron for scheduled tasks

**Endpoints:**
```
GET /financial/goals
POST /financial/goals
PUT /financial/goals/:id
DELETE /financial/goals/:id
GET /financial/transactions
POST /financial/transactions
GET /financial/portfolio
POST /financial/portfolio/sync
GET /financial/fire-calculation
GET /financial/tax-optimization
```

#### 4. AI/Claude Service
**Responsibilities:**
- Claude 4 API integration
- Financial advice generation
- Knowledge base management
- Response validation and safety

**Technology:**
- Express.js with Anthropic SDK
- Custom prompt engineering
- Response caching and optimization

**Endpoints:**
```
POST /ai/chat
GET /ai/advice/:category
POST /ai/analyze-portfolio
POST /ai/tax-advice
GET /ai/fire-strategies
```

#### 5. Banking Service
**Responsibilities:**
- Open Banking API integration
- Bank account management
- Transaction synchronization
- Account balance updates

**Technology:**
- Express.js with Axios
- TrueLayer/Plaid integration
- Encryption for sensitive data

**Endpoints:**
```
GET /banking/providers
POST /banking/connect
GET /banking/accounts
GET /banking/transactions
POST /banking/sync
DELETE /banking/disconnect
```

#### 6. Analytics Service
**Responsibilities:**
- User behaviour tracking
- Financial analytics
- Reporting and insights
- Performance metrics

**Technology:**
- Express.js with ClickHouse
- Apache Kafka for event streaming
- Chart.js data preparation

**Endpoints:**
```
GET /analytics/dashboard
GET /analytics/spending-trends
GET /analytics/goal-progress
GET /analytics/fire-projection
POST /analytics/custom-report
```

#### 7. Notification Service
**Responsibilities:**
- Email notifications
- Push notifications
- SMS alerts
- In-app notifications

**Technology:**
- Express.js with Bull Queue
- SendGrid for email
- Firebase for push notifications
- Twilio for SMS

**Endpoints:**
```
POST /notifications/send
GET /notifications/preferences
PUT /notifications/preferences
GET /notifications/history
```

#### 8. Subscription Service
**Responsibilities:**
- Payment processing
- Subscription management
- Billing and invoicing
- Usage tracking

**Technology:**
- Express.js with Stripe SDK
- Webhook handling
- Subscription lifecycle management

**Endpoints:**
```
POST /subscriptions/create
GET /subscriptions/current
PUT /subscriptions/upgrade
PUT /subscriptions/downgrade
POST /subscriptions/cancel
GET /subscriptions/invoices
```

### Database Schema Design

#### Core Tables

```sql
-- Users table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    date_of_birth DATE,
    postcode VARCHAR(10),
    phone VARCHAR(20),
    email_verified BOOLEAN DEFAULT FALSE,
    two_fa_enabled BOOLEAN DEFAULT FALSE,
    two_fa_secret VARCHAR(255),
    subscription_tier VARCHAR(20) DEFAULT 'free',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Financial goals table
CREATE TABLE financial_goals (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    target_amount DECIMAL(15,2) NOT NULL,
    current_amount DECIMAL(15,2) DEFAULT 0,
    target_date DATE,
    goal_type VARCHAR(50) NOT NULL,
    priority INTEGER DEFAULT 1,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Transactions table
CREATE TABLE transactions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    account_id UUID,
    amount DECIMAL(15,2) NOT NULL,
    description TEXT,
    category VARCHAR(100),
    transaction_date DATE NOT NULL,
    transaction_type VARCHAR(20) NOT NULL, -- 'income', 'expense', 'transfer'
    is_recurring BOOLEAN DEFAULT FALSE,
    external_id VARCHAR(255), -- For bank integration
    created_at TIMESTAMP DEFAULT NOW()
);

-- Investment portfolios table
CREATE TABLE investment_portfolios (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    platform_name VARCHAR(100),
    account_type VARCHAR(50), -- 'isa', 'pension', 'general'
    total_value DECIMAL(15,2) DEFAULT 0,
    last_synced TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- AI conversations table
CREATE TABLE ai_conversations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    session_id UUID NOT NULL,
    user_message TEXT NOT NULL,
    ai_response TEXT NOT NULL,
    conversation_type VARCHAR(50), -- 'general', 'tax', 'investment', 'fire'
    tokens_used INTEGER,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Security Implementation

#### Authentication & Authorization
- **JWT Tokens:** Short-lived access tokens (15 minutes)
- **Refresh Tokens:** Long-lived tokens (30 days) stored securely
- **Role-Based Access Control (RBAC):** User, admin, support roles
- **API Rate Limiting:** Per-user and per-endpoint limits

#### Data Protection
- **Encryption at Rest:** AES-256 encryption for sensitive data
- **Encryption in Transit:** TLS 1.3 for all communications
- **Field-Level Encryption:** For PII and financial data
- **Key Management:** AWS KMS for encryption key management

#### Compliance
- **GDPR Compliance:** Data portability, right to deletion
- **PCI DSS:** For payment processing
- **SOC 2 Type II:** Security and availability controls
- **ISO 27001:** Information security management

---

## Web Application Architecture

### Technology Stack

#### Frontend Framework
- **Framework:** React 18+ with TypeScript
- **State Management:** Redux Toolkit with RTK Query
- **Routing:** React Router v6
- **UI Library:** Custom components with Tailwind CSS
- **Form Handling:** React Hook Form with Yup validation

#### Build Tools
- **Bundler:** Vite for fast development and builds
- **Package Manager:** npm with package-lock.json
- **Code Quality:** ESLint, Prettier, Husky pre-commit hooks
- **Testing:** Jest, React Testing Library, Cypress

#### Performance Optimization
- **Code Splitting:** Route-based and component-based splitting
- **Lazy Loading:** Dynamic imports for heavy components
- **Caching:** Service Worker for offline functionality
- **CDN:** CloudFront for static asset delivery

### Application Structure

```
src/
├── components/           # Reusable UI components
│   ├── common/          # Generic components (Button, Input, etc.)
│   ├── forms/           # Form-specific components
│   ├── charts/          # Data visualization components
│   └── layout/          # Layout components (Header, Sidebar, etc.)
├── pages/               # Route components
│   ├── auth/           # Authentication pages
│   ├── dashboard/      # Dashboard and main app pages
│   ├── goals/          # Financial goals pages
│   ├── portfolio/      # Investment portfolio pages
│   └── settings/       # User settings pages
├── hooks/               # Custom React hooks
├── services/            # API service functions
├── store/               # Redux store configuration
│   ├── slices/         # Redux Toolkit slices
│   └── api/            # RTK Query API definitions
├── utils/               # Utility functions
├── types/               # TypeScript type definitions
├── constants/           # Application constants
└── assets/              # Static assets (images, icons, etc.)
```

### Key Features Implementation

#### Dashboard
- Real-time financial data display
- Interactive charts and graphs
- Goal progress visualisation
- Quick action buttons
- Responsive grid layout

#### FIRE Calculator
- Dynamic calculation engine
- Multiple scenario comparison
- Interactive sliders and inputs
- Real-time result updates
- Export functionality

#### AI Chat Interface
- Real-time messaging
- Typing indicators
- Message history
- Context-aware responses
- File attachment support

---

## Mobile Application Architecture

### iOS Application

#### Technology Stack
- **Language:** Swift 5.7+
- **UI Framework:** SwiftUI with UIKit integration
- **Architecture:** MVVM with Combine
- **Networking:** URLSession with async/await
- **Local Storage:** Core Data with CloudKit sync
- **Security:** Keychain Services, biometric authentication

#### Project Structure
```
Capita-iOS/
├── App/                 # App configuration and main files
├── Models/              # Data models and Core Data entities
├── Views/               # SwiftUI views and screens
├── ViewModels/          # MVVM view models
├── Services/            # API and business logic services
├── Utilities/           # Helper functions and extensions
├── Resources/           # Assets, localizations, etc.
└── Tests/               # Unit and UI tests
```

#### Key Features
- **Biometric Authentication:** Face ID/Touch ID integration
- **Offline Support:** Core Data for local storage
- **Push Notifications:** Firebase Cloud Messaging
- **Background Sync:** Background app refresh for data updates
- **Accessibility:** VoiceOver and Dynamic Type support

### Android Application

#### Technology Stack
- **Language:** Kotlin
- **UI Framework:** Jetpack Compose
- **Architecture:** MVVM with Android Architecture Components
- **Networking:** Retrofit with OkHttp
- **Local Storage:** Room database
- **Security:** Android Keystore, biometric authentication

#### Project Structure
```
app/src/main/java/com/capita/
├── ui/                  # Compose UI screens and components
│   ├── theme/          # Material Design theme
│   ├── screens/        # Screen composables
│   └── components/     # Reusable UI components
├── data/                # Data layer (repositories, data sources)
│   ├── local/          # Room database and DAOs
│   ├── remote/         # API services and DTOs
│   └── repository/     # Repository implementations
├── domain/              # Business logic and use cases
│   ├── model/          # Domain models
│   ├── repository/     # Repository interfaces
│   └── usecase/        # Use case implementations
├── di/                  # Dependency injection (Hilt)
└── util/                # Utility classes and extensions
```

#### Key Features
- **Material Design 3:** Modern Android design system
- **Biometric Authentication:** Fingerprint and face unlock
- **Offline Support:** Room database with sync capabilities
- **Push Notifications:** Firebase Cloud Messaging
- **Background Sync:** WorkManager for scheduled tasks
- **Accessibility:** TalkBack and accessibility services support

### Cross-Platform Considerations

#### Shared Features
- Consistent user experience across platforms
- Synchronized data and state
- Platform-specific optimizations
- Native performance and feel

#### Platform-Specific Adaptations
- **iOS:** Human Interface Guidelines compliance
- **Android:** Material Design Guidelines compliance
- **Navigation:** Platform-appropriate navigation patterns
- **Gestures:** Platform-specific gesture handling

---

## Infrastructure & DevOps

### Cloud Infrastructure (AWS)

#### Compute Services
- **ECS Fargate:** Containerized microservices
- **Lambda:** Serverless functions for specific tasks
- **Auto Scaling:** Automatic scaling based on demand

#### Database Services
- **RDS PostgreSQL:** Primary database with Multi-AZ deployment
- **ElastiCache Redis:** Caching and session storage
- **OpenSearch:** Search and analytics

#### Storage Services
- **S3:** Object storage for files and backups
- **EFS:** Shared file system for containers
- **CloudFront:** CDN for global content delivery

#### Networking
- **VPC:** Isolated network environment
- **Application Load Balancer:** Traffic distribution
- **Route 53:** DNS management
- **CloudFlare:** Additional security and performance

### CI/CD Pipeline

#### Source Control
- **Git:** Version control with GitFlow branching strategy
- **GitHub:** Repository hosting with branch protection
- **Code Review:** Pull request workflow with required reviews

#### Build Pipeline
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Commit    │───▶│   Build     │───▶│    Test     │───▶│   Deploy    │
│   to Git    │    │   & Lint    │    │   & Scan    │    │   to Env    │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

#### Deployment Strategy
- **Blue-Green Deployment:** Zero-downtime deployments
- **Feature Flags:** Gradual feature rollouts
- **Rollback Capability:** Quick rollback on issues
- **Environment Promotion:** Dev → Staging → Production

### Monitoring & Observability

#### Application Monitoring
- **DataDog:** Application performance monitoring
- **Sentry:** Error tracking and alerting
- **CloudWatch:** AWS service monitoring
- **Custom Metrics:** Business-specific KPIs

#### Logging
- **Structured Logging:** JSON format with correlation IDs
- **Log Aggregation:** Centralized logging with ElasticSearch
- **Log Retention:** Configurable retention policies
- **Security Logging:** Audit trails for compliance

#### Alerting
- **PagerDuty:** Incident management and escalation
- **Slack Integration:** Real-time notifications
- **Custom Dashboards:** Business and technical metrics
- **SLA Monitoring:** Uptime and performance tracking

---

## Security Architecture

### Defense in Depth

#### Network Security
- **WAF:** Web Application Firewall (CloudFlare)
- **DDoS Protection:** CloudFlare and AWS Shield
- **VPC Security:** Private subnets and security groups
- **Network ACLs:** Additional network-level filtering

#### Application Security
- **OWASP Top 10:** Protection against common vulnerabilities
- **Input Validation:** Comprehensive input sanitization
- **SQL Injection Prevention:** Parameterized queries
- **XSS Protection:** Content Security Policy and sanitization

#### Data Security
- **Encryption:** AES-256 at rest, TLS 1.3 in transit
- **Key Management:** AWS KMS with key rotation
- **Data Classification:** Sensitive data identification
- **Access Controls:** Principle of least privilege

#### Identity & Access Management
- **Multi-Factor Authentication:** TOTP and SMS
- **Session Management:** Secure session handling
- **Password Policy:** Strong password requirements
- **Account Lockout:** Brute force protection

### Compliance Framework

#### GDPR Compliance
- **Data Mapping:** Complete data inventory
- **Consent Management:** Granular consent controls
- **Data Portability:** Export functionality
- **Right to Deletion:** Secure data removal

#### Financial Regulations
- **FCA Compliance:** Financial advice regulations
- **PCI DSS:** Payment card security
- **Open Banking:** Secure API integration
- **Audit Trails:** Comprehensive logging

---

## Performance & Scalability

### Performance Targets
- **API Response Time:** <200ms for 95% of requests
- **Web App Load Time:** <3 seconds initial load
- **Mobile App Launch:** <2 seconds cold start
- **Database Queries:** <100ms for 99% of queries

### Scalability Strategy
- **Horizontal Scaling:** Auto-scaling groups
- **Database Scaling:** Read replicas and sharding
- **Caching Strategy:** Multi-layer caching
- **CDN Usage:** Global content distribution

### Optimisation Techniques
- **Database Indexing:** Optimised query performance
- **Connection Pooling:** Efficient database connections
- **Compression:** Gzip compression for responses
- **Image Optimisation:** WebP format and lazy loading

---

## Disaster Recovery & Business Continuity

### Backup Strategy
- **Database Backups:** Daily automated backups with 30-day retention
- **File Backups:** S3 cross-region replication
- **Configuration Backups:** Infrastructure as code
- **Application Backups:** Container image versioning

### Recovery Procedures
- **RTO (Recovery Time Objective):** 4 hours
- **RPO (Recovery Point Objective):** 1 hour
- **Failover Process:** Automated failover to secondary region
- **Data Recovery:** Point-in-time recovery capabilities

### Testing
- **Disaster Recovery Drills:** Quarterly testing
- **Backup Verification:** Monthly restore testing
- **Failover Testing:** Semi-annual failover tests
- **Documentation:** Detailed recovery procedures

---

## Development Standards

### Code Quality
- **Linting:** ESLint for JavaScript/TypeScript, SwiftLint for iOS
- **Formatting:** Prettier for consistent code formatting
- **Testing:** Minimum 80% code coverage
- **Documentation:** Comprehensive API and code documentation

### Git Workflow
- **Branching Strategy:** GitFlow with feature branches
- **Commit Messages:** Conventional commit format
- **Code Review:** Required reviews before merging
- **Branch Protection:** Protected main and develop branches

### API Standards
- **RESTful Design:** Consistent REST API patterns
- **Versioning:** Semantic versioning for APIs
- **Documentation:** OpenAPI 3.0 specifications
- **Error Handling:** Standardized error responses

---

## Future Considerations

### Scalability Enhancements
- **Microservices Decomposition:** Further service separation
- **Event-Driven Architecture:** Kafka for event streaming
- **CQRS Implementation:** Command Query Responsibility Segregation
- **GraphQL Federation:** Distributed GraphQL schema

### Technology Evolution
- **AI/ML Integration:** Enhanced machine learning capabilities
- **Blockchain Integration:** Cryptocurrency and DeFi support
- **Edge Computing:** Edge deployment for global performance
- **Serverless Migration:** Increased serverless adoption

### International Expansion
- **Multi-Region Deployment:** Global infrastructure
- **Localization Support:** Multi-language and currency
- **Regulatory Compliance:** International financial regulations
- **Data Residency:** Regional data storage requirements