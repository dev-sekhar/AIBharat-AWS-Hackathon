# Design Document: EduChamps

## Overview

EduChamps is an AI-powered educational progress tracking platform that automates feedback generation, monitors student performance in real-time, and provides personalized insights for students, educators, and parents. The system uses machine learning to analyze student work, identify learning gaps, generate adaptive study plans, and provide early intervention for at-risk students.

The platform serves three primary user roles:
- **Students**: Submit assignments, receive AI-generated feedback, track progress, and follow personalized study plans
- **Educators**: Monitor class performance, customize AI feedback, identify at-risk students, and generate progress reports
- **Parents**: View child's progress, receive notifications, and access support recommendations

The design emphasizes scalability, security, and real-time responsiveness while maintaining educational effectiveness through AI-powered personalization.

## Architecture

### System Architecture

EduChamps follows a microservices architecture with three distinct layers: Frontend, Backend, and Persistence.

```
┌──────────────────────────────────────────────────────────────────┐
│                          FRONTEND                                 │
├──────────────┬──────────────┬──────────────┬──────────────────────┤
│   Student    │   Educator   │    Parent    │  Platform Admin      │
└──────────────┴──────────────┴──────────────┴──────────────────────┘
                              │
                              │ HTTPS/REST
                              ▼
                    ┌──────────────────┐
                    │   API Gateway    │
                    │    (NestJS)      │
                    └──────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌────────────────────────────────────────────────────────────────────┐
│                          BACKEND                                    │
│                                                                     │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐        │
│  │ OCR Service  │    │ Auth Service │    │  Reporting   │        │
│  │   (Python)   │    │   (NestJS)   │    │   Engine     │        │
│  └──────────────┘    └──────────────┘    │   (NestJS)   │        │
│                                           └──────────────┘        │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐        │
│  │ Notification │    │ External Data│    │   Analysis   │        │
│  │   Service    │    │   Service    │    │   Engine     │        │
│  │   (NestJS)   │    │   (NestJS)   │    │   (NestJS)   │        │
│  └──────────────┘    └──────────────┘    └──────────────┘        │
│                                                                     │
│                      ┌──────────────┐                              │
│                      │  AI Service  │                              │
│                      │   (NestJS)   │                              │
│                      └──────┬───────┘                              │
│                             │                                      │
└─────────────────────────────┼──────────────────────────────────────┘
                              │
                              ▼
                    ┌──────────────────┐
                    │    AI Models     │
                    │  (External APIs) │
                    └──────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Gemini 1.5   │    │   Qwen-VL    │    │ Cloud Vision │
│  Pro/Flash   │    │ (Hugging Face│    │   (Google)   │
└──────────────┘    └──────────────┘    └──────────────┘
                              │
                              ▼
                    ┌──────────────────┐
                    │    Syllabus      │
                    │   Repository     │
                    │  (File Storage)  │
                    └──────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────────┐
│                        PERSISTENCE                                  │
│                                                                     │
│                    ┌──────────────────┐                            │
│                    │     Database     │                            │
│                    │   (MySQL 8.0)    │                            │
│                    │                  │                            │
│                    │  - Users         │                            │
│                    │  - Assignments   │                            │
│                    │  - Feedback      │                            │
│                    │  - Analytics     │                            │
│                    └──────────────────┘                            │
│                                                                     │
│                    ┌──────────────────┐                            │
│                    │      Redis       │                            │
│                    │   (7-alpine)     │                            │
│                    │                  │                            │
│                    │  - Cache         │                            │
│                    │  - BullMQ Jobs   │                            │
│                    │  - Sessions      │                            │
│                    └──────────────────┘                            │
└────────────────────────────────────────────────────────────────────┘
```

### Architecture Overview

**Frontend Layer (Next.js 14 + React 18)**
- **Student Portal**: Submit assignments, view feedback, track progress, access study plans
- **Educator Portal**: Review class analytics, customize AI feedback, generate reports, manage students
- **Parent Portal**: Monitor child's progress, receive notifications, access support recommendations
- **Platform Admin**: System configuration, user management, platform monitoring

**API Gateway (NestJS)**
- Single entry point for all client requests
- Authentication and authorization enforcement
- Request routing to appropriate backend services
- Rate limiting and request throttling
- API versioning and documentation

**Backend Services (NestJS + Python)**

1. **OCR Service (Python)**
   - Extracts text from uploaded documents (PDF, DOCX, images)
   - Hybrid multi-engine approach for optimal accuracy
   - Integrates with Gemini, Qwen-VL, and Cloud Vision APIs

2. **Auth Service (NestJS)**
   - User authentication (JWT, OAuth)
   - Role-based access control (RBAC)
   - Session management
   - Parent-student account linking

3. **Reporting Engine (NestJS)**
   - Generates student and class progress reports
   - Aggregates performance data
   - Exports reports in multiple formats (PDF, CSV)

4. **Analysis Engine (NestJS)**
   - Analyzes student performance patterns
   - Identifies learning gaps
   - Detects at-risk students
   - Calculates performance metrics

5. **Notification Service (NestJS)**
   - Multi-channel notifications (email, SMS, in-app)
   - Notification scheduling and batching
   - User preference management

6. **External Data Service (NestJS)**
   - Integrates with external educational systems
   - Data import/export functionality
   - LMS synchronization

7. **AI Service (NestJS)**
   - Orchestrates AI model interactions
   - Generates personalized feedback
   - Manages AI API calls and fallbacks
   - Accesses syllabus repository for context

**AI Models Layer**
- **Google Gemini 1.5 Pro/Flash**: Primary OCR and analysis engine for complex documents
- **Qwen-VL-7B (Hugging Face)**: Open-source alternative for vision-language tasks
- **Google Cloud Vision**: Fast baseline text extraction
- **Syllabus Repository**: File-based storage containing curriculum, rubrics, and learning objectives

**Persistence Layer**
- **MySQL 8.0 Database**: Stores all application data (users, assignments, feedback, analytics)
- **Redis 7-alpine**: Caching layer, BullMQ job queue, session storage

**Data Flow Example: Assignment Submission**
1. Student uploads assignment via Student Portal
2. API Gateway authenticates request and routes to OCR Service
3. OCR Service extracts text using AI Models (Gemini/Qwen-VL/Cloud Vision)
4. AI Service generates feedback using extracted text + syllabus context
5. Analysis Engine evaluates performance and identifies learning gaps
6. Feedback stored in MySQL Database
7. Notification Service alerts student and educator
8. Results cached in Redis for fast dashboard access

### Technology Stack

#### 1.1 Frontend (Student, Teacher/Admin, Parent Portals)
- **Framework**: [Next.js 14](https://nextjs.org/) (React 18)
- **Language**: TypeScript
- **Styling**: 
  - [Tailwind CSS](https://tailwindcss.com/) (Utility-first CSS)
  - `clsx` & `tailwind-merge` (Class conditional logic)
  - `lucide-react` (Icons)
  - `framer-motion` (Animations)
- **State Management & Data Fetching**: 
  - [TanStack Query (React Query) v5](https://tanstack.com/query/latest)
- **Forms**: `react-hook-form`
- **HTTP Client**: `axios`
- **Markdown Rendering**: `react-markdown`, `react-syntax-highlighter`
- **Utilities**: `js-cookie`, `date-fns`
- **Notifications**: `react-hot-toast`

#### 1.2 Backend (API & Orchestration)
- **Framework**: [NestJS v10](https://nestjs.com/)
- **Language**: TypeScript
- **Runtime**: Node.js
- **Database ORM**: [Prisma v5](https://www.prisma.io/)
- **Authentication**: 
  - `passport` (Strategies: JWT, Google OAuth, Microsoft)
  - `bcrypt` (Password hashing)
  - JSON Web Tokens (JWT) for stateless auth
- **Queue & Background Jobs**: 
  - [BullMQ](https://docs.bullmq.io/)
  - Redis (as the message broker)
- **File Processing**: 
  - `multer` (Uploads)
  - `pdf-lib`, `pdf-parse`, `pdf2pic` (PDF manipulation)
  - `sharp`, `graphicsmagick` (Image processing)
- **Logging**: `winston`, `nest-winston`

#### 1.3 AI & Machine Learning (OCR & Analysis Engine)
- **Core Strategy**: Hybrid Multi-Engine Approach
- **Languages**: Python (invoked via `child_process` from NestJS)
- **Primary OCR Engine**: 
  - **Google Gemini 1.5 Pro / Flash** (via `universal_ocr.py`) - Best for complex layouts and tables
  - **Hugging Face Inference** (Qwen-VL-7B-Instruct) - Open-source alternative
  - **Google Cloud Vision** (via Node.js SDK) - Fast, baseline text extraction
- **Analysis Capabilities**:
  - `universal_analysis.py`: dedicated script to compare OCR output against Rubrics
- **Key Libraries**: `tesseract.js` (local fallback)

#### 1.4 Database & Storage
- **Relational Database**: **MySQL 8.0**
  - Host: Docker container (`studaicoach-mysql`)
  - Management: Prisma Migrations
- **Caching & Message Broker**: **Redis 7-alpine**
  - Host: Docker container (`studaicoach-redis`)
  - Usage: Caching API responses, BullMQ job persistence, valid tokens

#### 1.5 DevOps & Infrastructure
- **Containerization**: Docker & Docker Compose
- **Package Manager**: Yarn (Enforced)
- **Environment Management**: `dotenv` (.env files)


## Components and Interfaces

### 1. API Gateway

**Responsibilities:**
- Route requests to appropriate backend microservices
- Handle authentication and authorization
- Rate limiting and request throttling
- Request/response logging
- Load balancing across service instances

**Key Interfaces:**
```typescript
interface APIGatewayConfig {
  routes: RouteConfig[];
  rateLimits: RateLimitConfig;
  authProvider: AuthenticationProvider;
}

interface RouteConfig {
  path: string;
  method: HTTPMethod;
  service: string;
  requiresAuth: boolean;
  allowedRoles: UserRole[];
}
```

### 2. Auth Service

**Responsibilities:**
- User authentication and session management
- User profile management
- Role-based access control
- Parent-student account linking
- Invitation system for parent registration
- JWT token generation and validation

**Key Interfaces:**
```typescript
interface AuthService {
  authenticate(credentials: Credentials): Promise<AuthToken>;
  validateToken(token: string): Promise<TokenValidation>;
  createUser(userData: UserData, role: UserRole): Promise<User>;
  linkParentToStudent(parentId: string, studentId: string): Promise<void>;
  generateInvitation(educatorId: string, studentId: string): Promise<Invitation>;
  validateInvitation(inviteCode: string): Promise<boolean>;
  updateNotificationPreferences(userId: string, prefs: NotificationPreferences): Promise<void>;
}

interface User {
  id: string;
  email: string;
  passwordHash: string;
  role: UserRole;
  profile: UserProfile;
  createdAt: Date;
  lastLogin: Date;
}

enum UserRole {
  STUDENT = 'student',
  EDUCATOR = 'educator',
  PARENT = 'parent',
  ADMIN = 'admin'
}

interface Invitation {
  code: string;
  educatorId: string;
  studentId: string;
  expiresAt: Date;
  used: boolean;
}
```

### 3. OCR Service

**Responsibilities:**
- Extract text from uploaded assignment files (PDF, DOCX, images)
- Use hybrid multi-engine approach for optimal accuracy
- Handle different document layouts and formats
- Normalize and clean extracted text
- Return structured text data for analysis

**Key Interfaces:**
```typescript
interface OCRService {
  extractText(file: File, options: OCROptions): Promise<OCRResult>;
  validateFile(file: File): ValidationResult;
  selectOptimalEngine(fileMetadata: FileMetadata): OCREngine;
}

interface OCRResult {
  text: string;
  confidence: number;
  engine: OCREngine;
  metadata: {
    pageCount: number;
    language: string;
    processingTime: number;
  };
}

enum OCREngine {
  GEMINI_PRO = 'gemini_1.5_pro',
  GEMINI_FLASH = 'gemini_1.5_flash',
  QWEN_VL = 'qwen_vl_7b',
  CLOUD_VISION = 'google_cloud_vision',
  TESSERACT = 'tesseract_fallback'
}

interface OCROptions {
  preferredEngine?: OCREngine;
  language?: string;
  enhanceQuality?: boolean;
}

const ALLOWED_FILE_TYPES = ['application/pdf', 'application/vnd.openxmlformats-officedocument.wordprocessingml.document', 'text/plain', 'image/jpeg', 'image/png'];
const MAX_FILE_SIZE = 50 * 1024 * 1024; // 50MB
```

### 4. AI Service

**Responsibilities:**
- Orchestrate AI models for various tasks
- Generate personalized feedback using LLMs
- Coordinate with OCR Service for text extraction
- Access syllabus repository for curriculum context
- Manage AI model selection and fallback strategies
- Handle AI API rate limiting and error recovery

**Key Interfaces:**
```typescript
interface AIService {
  generateFeedback(assignmentText: string, context: AssignmentContext): Promise<Feedback>;
  generateBulkFeedback(assignments: AssignmentData[]): Promise<Feedback[]>;
  analyzeLearningGaps(studentId: string, performanceHistory: Performance[]): Promise<LearningGap[]>;
  generateStudyPlan(studentId: string, learningGaps: LearningGap[]): Promise<StudyPlan>;
  assessRiskLevel(studentId: string, performanceData: PerformanceData): Promise<RiskAssessment>;
  analyzeSelfAssessment(responses: AssessmentResponse[]): Promise<SelfAssessmentResult>;
  getSyllabusContext(subjectId: string, topic: string): Promise<SyllabusContext>;
}

interface AssignmentContext {
  studentId: string;
  subjectId: string;
  gradeLevel: number;
  syllabusContext: SyllabusContext;
  studentHistory: PerformanceHistory;
}

interface Feedback {
  id: string;
  assignmentId: string;
  studentId: string;
  content: string;
  strengths: FeedbackPoint[];
  improvements: FeedbackPoint[];
  examples: string[];
  isAIGenerated: boolean;
  isModified: boolean;
  generatedAt: Date;
  approvedBy?: string;
  approvedAt?: Date;
}

interface FeedbackPoint {
  category: string;
  description: string;
  specificExample: string;
  actionableAdvice: string;
}

interface SyllabusContext {
  subjectId: string;
  topic: string;
  learningObjectives: string[];
  rubric: RubricCriteria[];
  expectedConcepts: string[];
}
```

### 5. Analysis Engine

**Responsibilities:**
- Analyze student performance data
- Identify learning gaps and patterns
- Detect at-risk students
- Calculate performance metrics and trends
- Generate insights for educators
- Track progress over time

**Key Interfaces:**
```typescript
interface AnalysisEngine {
  analyzeAssignment(assignmentText: string, rubric: RubricCriteria[]): Promise<AnalysisResult>;
  identifyLearningGaps(studentId: string, performanceHistory: Performance[]): Promise<LearningGap[]>;
  detectAtRiskStudents(classId: string): Promise<RiskAssessment[]>;
  calculatePerformanceTrend(studentId: string, subjectId: string): Promise<PerformanceTrend>;
  identifyCommonGaps(classId: string): Promise<CommonGap[]>;
  generateInsights(classId: string): Promise<ClassInsights>;
}

interface AnalysisResult {
  assignmentId: string;
  performanceScore: number;
  strengths: string[];
  weaknesses: string[];
  conceptsMastered: string[];
  conceptsNeedingWork: string[];
  confidenceScore: number;
}

interface LearningGap {
  id: string;
  studentId: string;
  subjectArea: string;
  concept: string;
  severity: 'low' | 'medium' | 'high';
  confidenceScore: number;
  identifiedAt: Date;
  resolved: boolean;
}

interface RiskAssessment {
  studentId: string;
  isAtRisk: boolean;
  riskLevel: 'low' | 'medium' | 'high';
  factors: RiskFactor[];
  recommendations: string[];
  assessedAt: Date;
}

interface RiskFactor {
  factor: string;
  weight: number;
  description: string;
}
```

### 6. Reporting Engine

**Responsibilities:**
- Generate progress reports for students
- Create class-wide analytics reports
- Aggregate performance data across time periods
- Export reports in multiple formats (PDF, CSV)
- Schedule automated report generation
- Provide customizable report templates

**Key Interfaces:**
```typescript
interface ReportingEngine {
  generateStudentReport(studentId: string, timeRange: TimeRange): Promise<StudentReport>;
  generateClassReport(classId: string, timeRange: TimeRange): Promise<ClassReport>;
  generateParentReport(parentId: string): Promise<ParentReport>;
  scheduleReport(config: ReportScheduleConfig): Promise<ScheduledReport>;
  exportReport(reportId: string, format: ExportFormat): Promise<ExportedReport>;
}

interface StudentReport {
  studentId: string;
  timeRange: TimeRange;
  overallPerformance: PerformanceSummary;
  subjectBreakdown: SubjectPerformance[];
  learningGaps: LearningGap[];
  improvements: string[];
  recommendations: string[];
  generatedAt: Date;
}

interface ClassReport {
  classId: string;
  timeRange: TimeRange;
  classPerformance: ClassPerformance;
  studentSummaries: StudentSummary[];
  commonGaps: CommonGap[];
  performanceDistribution: DistributionData;
  generatedAt: Date;
}

enum ExportFormat {
  PDF = 'pdf',
  CSV = 'csv',
  JSON = 'json',
  EXCEL = 'xlsx'
}
```

### 7. Notification Service

**Responsibilities:**
- Send notifications via multiple channels (email, SMS, in-app)
- Respect user notification preferences
- Queue and batch notifications
- Track notification delivery status
- Handle notification templates
- Manage notification scheduling

**Key Interfaces:**
```typescript
interface NotificationService {
  sendNotification(notification: Notification): Promise<void>;
  sendBulkNotifications(notifications: Notification[]): Promise<void>;
  updatePreferences(userId: string, preferences: NotificationPreferences): Promise<void>;
  getNotificationHistory(userId: string): Promise<Notification[]>;
  scheduleNotification(notification: Notification, scheduledTime: Date): Promise<void>;
}

interface Notification {
  id: string;
  userId: string;
  type: NotificationType;
  channel: NotificationChannel[];
  subject: string;
  content: string;
  priority: 'low' | 'medium' | 'high';
  scheduledAt: Date;
  sentAt?: Date;
  readAt?: Date;
}

enum NotificationType {
  FEEDBACK_READY = 'feedback_ready',
  LEARNING_GAP = 'learning_gap',
  MILESTONE_ACHIEVED = 'milestone_achieved',
  AT_RISK_ALERT = 'at_risk_alert',
  PROGRESS_UPDATE = 'progress_update',
  EDUCATOR_MESSAGE = 'educator_message'
}

enum NotificationChannel {
  EMAIL = 'email',
  SMS = 'sms',
  IN_APP = 'in_app'
}

interface NotificationPreferences {
  userId: string;
  channels: NotificationChannel[];
  frequency: 'immediate' | 'daily' | 'weekly';
  enabledTypes: NotificationType[];
}
```

### 8. External Data Service

**Responsibilities:**
- Integrate with external educational systems
- Import student data from SIS (Student Information Systems)
- Sync with learning management systems (LMS)
- Export data to external reporting tools
- Handle data transformation and mapping
- Manage API credentials for external services

**Key Interfaces:**
```typescript
interface ExternalDataService {
  importStudentData(source: DataSource, config: ImportConfig): Promise<ImportResult>;
  exportData(destination: DataDestination, data: ExportData): Promise<ExportResult>;
  syncWithLMS(lmsConfig: LMSConfig): Promise<SyncResult>;
  validateExternalData(data: any, schema: DataSchema): ValidationResult;
}

interface DataSource {
  type: 'SIS' | 'LMS' | 'CSV' | 'API';
  endpoint?: string;
  credentials?: Credentials;
}

interface ImportResult {
  success: boolean;
  recordsImported: number;
  errors: ImportError[];
  timestamp: Date;
}
```

## Data Models

### Database Schema

**Users Table:**
```sql
CREATE TABLE users (
  id VARCHAR(36) PRIMARY KEY DEFAULT (UUID()),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  role VARCHAR(50) NOT NULL,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  last_login TIMESTAMP NULL,
  account_locked BOOLEAN DEFAULT FALSE,
  failed_login_attempts INT DEFAULT 0,
  locked_until TIMESTAMP NULL
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
```


**Students Table:**
```sql
CREATE TABLE students (
  id VARCHAR(36) PRIMARY KEY,
  grade_level INT,
  enrollment_date DATE,
  active BOOLEAN DEFAULT TRUE,
  FOREIGN KEY (id) REFERENCES users(id) ON DELETE CASCADE
);
```

**Educators Table:**
```sql
CREATE TABLE educators (
  id VARCHAR(36) PRIMARY KEY,
  department VARCHAR(100),
  specialization VARCHAR(100),
  FOREIGN KEY (id) REFERENCES users(id) ON DELETE CASCADE
);
```

**Parents Table:**
```sql
CREATE TABLE parents (
  id VARCHAR(36) PRIMARY KEY,
  FOREIGN KEY (id) REFERENCES users(id) ON DELETE CASCADE
);
```

**Parent-Student Links:**
```sql
CREATE TABLE parent_student_links (
  parent_id VARCHAR(36),
  student_id VARCHAR(36),
  linked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (parent_id, student_id),
  FOREIGN KEY (parent_id) REFERENCES parents(id) ON DELETE CASCADE,
  FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE
);
```

**Invitations Table:**
```sql
CREATE TABLE invitations (
  id VARCHAR(36) PRIMARY KEY DEFAULT (UUID()),
  code VARCHAR(100) UNIQUE NOT NULL,
  educator_id VARCHAR(36),
  student_id VARCHAR(36),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  expires_at TIMESTAMP NOT NULL,
  used BOOLEAN DEFAULT FALSE,
  used_at TIMESTAMP NULL,
  FOREIGN KEY (educator_id) REFERENCES educators(id) ON DELETE CASCADE,
  FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE
);

CREATE INDEX idx_invitations_code ON invitations(code);
```

**Subjects Table:**
```sql
CREATE TABLE subjects (
  id VARCHAR(36) PRIMARY KEY DEFAULT (UUID()),
  name VARCHAR(100) NOT NULL,
  description TEXT,
  grade_level INT
);
```

**Classes Table:**
```sql
CREATE TABLE classes (
  id VARCHAR(36) PRIMARY KEY DEFAULT (UUID()),
  educator_id VARCHAR(36),
  subject_id VARCHAR(36),
  name VARCHAR(100) NOT NULL,
  academic_year VARCHAR(20),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (educator_id) REFERENCES educators(id) ON DELETE CASCADE,
  FOREIGN KEY (subject_id) REFERENCES subjects(id) ON DELETE SET NULL
);
```

**Class Enrollments:**
```sql
CREATE TABLE class_enrollments (
  class_id VARCHAR(36),
  student_id VARCHAR(36),
  enrolled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (class_id, student_id),
  FOREIGN KEY (class_id) REFERENCES classes(id) ON DELETE CASCADE,
  FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE
);
```


**Assignments Table:**
```sql
CREATE TABLE assignments (
  id VARCHAR(36) PRIMARY KEY DEFAULT (UUID()),
  student_id VARCHAR(36),
  class_id VARCHAR(36),
  subject_id VARCHAR(36),
  file_url VARCHAR(500) NOT NULL,
  file_name VARCHAR(255) NOT NULL,
  file_size BIGINT NOT NULL,
  mime_type VARCHAR(100) NOT NULL,
  status VARCHAR(50) NOT NULL,
  submitted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  processed_at TIMESTAMP NULL,
  FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE,
  FOREIGN KEY (class_id) REFERENCES classes(id) ON DELETE CASCADE,
  FOREIGN KEY (subject_id) REFERENCES subjects(id) ON DELETE SET NULL
);

CREATE INDEX idx_assignments_student ON assignments(student_id);
CREATE INDEX idx_assignments_class ON assignments(class_id);
CREATE INDEX idx_assignments_status ON assignments(status);
```

**Feedback Table:**
```sql
CREATE TABLE feedback (
  id VARCHAR(36) PRIMARY KEY DEFAULT (UUID()),
  assignment_id VARCHAR(36),
  student_id VARCHAR(36),
  content TEXT NOT NULL,
  is_ai_generated BOOLEAN DEFAULT TRUE,
  is_modified BOOLEAN DEFAULT FALSE,
  generated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  approved_by VARCHAR(36) NULL,
  approved_at TIMESTAMP NULL,
  viewed_by_student BOOLEAN DEFAULT FALSE,
  viewed_at TIMESTAMP NULL,
  FOREIGN KEY (assignment_id) REFERENCES assignments(id) ON DELETE CASCADE,
  FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE,
  FOREIGN KEY (approved_by) REFERENCES educators(id) ON DELETE SET NULL
);

CREATE INDEX idx_feedback_assignment ON feedback(assignment_id);
CREATE INDEX idx_feedback_student ON feedback(student_id);
```

**Feedback Points Table:**
```sql
CREATE TABLE feedback_points (
  id VARCHAR(36) PRIMARY KEY DEFAULT (UUID()),
  feedback_id VARCHAR(36),
  type VARCHAR(50) NOT NULL, -- 'strength' or 'improvement'
  category VARCHAR(100),
  description TEXT NOT NULL,
  specific_example TEXT,
  actionable_advice TEXT,
  FOREIGN KEY (feedback_id) REFERENCES feedback(id) ON DELETE CASCADE
);
```

**Learning Gaps Table:**
```sql
CREATE TABLE learning_gaps (
  id VARCHAR(36) PRIMARY KEY DEFAULT (UUID()),
  student_id VARCHAR(36),
  subject_id VARCHAR(36),
  concept VARCHAR(200) NOT NULL,
  severity VARCHAR(20) NOT NULL,
  confidence_score DECIMAL(3,2),
  identified_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  resolved BOOLEAN DEFAULT FALSE,
  resolved_at TIMESTAMP NULL,
  FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE,
  FOREIGN KEY (subject_id) REFERENCES subjects(id) ON DELETE SET NULL
);

CREATE INDEX idx_learning_gaps_student ON learning_gaps(student_id);
CREATE INDEX idx_learning_gaps_resolved ON learning_gaps(resolved);
```


**Study Plans Table:**
```sql
CREATE TABLE study_plans (
  id VARCHAR(36) PRIMARY KEY DEFAULT (UUID()),
  student_id VARCHAR(36),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE
);
```

**Study Tasks Table:**
```sql
CREATE TABLE study_tasks (
  id VARCHAR(36) PRIMARY KEY DEFAULT (UUID()),
  study_plan_id VARCHAR(36),
  description TEXT NOT NULL,
  target_concept VARCHAR(200),
  estimated_minutes INT,
  priority INT,
  completed BOOLEAN DEFAULT FALSE,
  completed_at TIMESTAMP NULL,
  FOREIGN KEY (study_plan_id) REFERENCES study_plans(id) ON DELETE CASCADE
);
```

**Self Assessments Table:**
```sql
CREATE TABLE self_assessments (
  id VARCHAR(36) PRIMARY KEY DEFAULT (UUID()),
  student_id VARCHAR(36),
  subject_id VARCHAR(36),
  started_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  completed_at TIMESTAMP NULL,
  score DECIMAL(5,2),
  FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE,
  FOREIGN KEY (subject_id) REFERENCES subjects(id) ON DELETE SET NULL
);
```

**Assessment Responses Table:**
```sql
CREATE TABLE assessment_responses (
  id VARCHAR(36) PRIMARY KEY DEFAULT (UUID()),
  assessment_id VARCHAR(36),
  question_text TEXT NOT NULL,
  student_response TEXT,
  is_correct BOOLEAN,
  concept_tested VARCHAR(200),
  FOREIGN KEY (assessment_id) REFERENCES self_assessments(id) ON DELETE CASCADE
);
```

**Performance Metrics Table:**
```sql
CREATE TABLE performance_metrics (
  id VARCHAR(36) PRIMARY KEY DEFAULT (UUID()),
  student_id VARCHAR(36),
  subject_id VARCHAR(36),
  metric_date DATE NOT NULL,
  average_score DECIMAL(5,2),
  assignments_completed INT,
  learning_gaps_count INT,
  at_risk_flag BOOLEAN DEFAULT FALSE,
  FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE,
  FOREIGN KEY (subject_id) REFERENCES subjects(id) ON DELETE SET NULL
);

CREATE INDEX idx_performance_student_date ON performance_metrics(student_id, metric_date);
```

**Notifications Table:**
```sql
CREATE TABLE notifications (
  id VARCHAR(36) PRIMARY KEY DEFAULT (UUID()),
  user_id VARCHAR(36),
  type VARCHAR(50) NOT NULL,
  subject VARCHAR(255),
  content TEXT NOT NULL,
  priority VARCHAR(20),
  scheduled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  sent_at TIMESTAMP NULL,
  read_at TIMESTAMP NULL,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE INDEX idx_notifications_user ON notifications(user_id);
CREATE INDEX idx_notifications_sent ON notifications(sent_at);
```


**Notification Preferences Table:**
```sql
CREATE TABLE notification_preferences (
  user_id VARCHAR(36) PRIMARY KEY,
  email_enabled BOOLEAN DEFAULT TRUE,
  sms_enabled BOOLEAN DEFAULT FALSE,
  in_app_enabled BOOLEAN DEFAULT TRUE,
  frequency VARCHAR(20) DEFAULT 'immediate',
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

**Audit Logs Table:**
```sql
CREATE TABLE audit_logs (
  id VARCHAR(36) PRIMARY KEY DEFAULT (UUID()),
  user_id VARCHAR(36),
  action VARCHAR(100) NOT NULL,
  resource_type VARCHAR(50),
  resource_id VARCHAR(36),
  ip_address VARCHAR(45),
  user_agent TEXT,
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL
);

CREATE INDEX idx_audit_logs_user ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_timestamp ON audit_logs(timestamp);
```

## API Design

### Authentication Endpoints

**POST /api/auth/login**
```typescript
Request: {
  email: string;
  password: string;
}

Response: {
  token: string;
  user: {
    id: string;
    email: string;
    role: UserRole;
    profile: UserProfile;
  };
  expiresAt: string;
}
```

**POST /api/auth/logout**
```typescript
Request: {
  token: string;
}

Response: {
  success: boolean;
}
```

**POST /api/auth/reset-password**
```typescript
Request: {
  email: string;
}

Response: {
  success: boolean;
  message: string;
}
```

### User Management Endpoints

**POST /api/users/register**
```typescript
Request: {
  email: string;
  password: string;
  role: UserRole;
  profile: UserProfile;
  inviteCode?: string; // Required for parent registration
}

Response: {
  user: User;
  token: string;
}
```

**GET /api/users/me**
```typescript
Response: {
  user: User;
  linkedAccounts?: LinkedAccount[]; // For parents
}
```

**PUT /api/users/me/preferences**
```typescript
Request: {
  notificationPreferences: NotificationPreferences;
}

Response: {
  success: boolean;
}
```


### Assignment Endpoints

**POST /api/assignments**
```typescript
Request: FormData {
  file: File;
  subjectId: string;
  classId: string;
}

Response: {
  assignment: Assignment;
  message: string;
}
```

**GET /api/assignments/:id**
```typescript
Response: {
  assignment: Assignment;
  feedback?: Feedback;
}
```

**GET /api/assignments**
```typescript
Query Parameters: {
  studentId?: string;
  classId?: string;
  status?: AssignmentStatus;
  startDate?: string;
  endDate?: string;
}

Response: {
  assignments: Assignment[];
  total: number;
}
```

### Feedback Endpoints

**GET /api/feedback/:assignmentId**
```typescript
Response: {
  feedback: Feedback;
}
```

**PUT /api/feedback/:id**
```typescript
Request: {
  content: string;
  feedbackPoints: FeedbackPoint[];
}

Response: {
  feedback: Feedback;
}
```

**POST /api/feedback/:id/approve**
```typescript
Response: {
  feedback: Feedback;
  notificationSent: boolean;
}
```

**POST /api/feedback/bulk-generate**
```typescript
Request: {
  classId: string;
  assignmentIds: string[];
}

Response: {
  jobId: string;
  estimatedCompletionTime: string;
}
```

**GET /api/feedback/bulk-generate/:jobId**
```typescript
Response: {
  status: 'processing' | 'completed' | 'failed';
  progress: number;
  feedbackItems: Feedback[];
}
```


### Dashboard Endpoints

**GET /api/dashboard/student/:studentId**
```typescript
Query Parameters: {
  timeRange?: 'week' | 'month' | 'semester' | 'year';
}

Response: {
  dashboard: StudentDashboard;
}
```

**GET /api/dashboard/educator/:educatorId/class/:classId**
```typescript
Response: {
  dashboard: EducatorDashboard;
}
```

**GET /api/dashboard/parent/:parentId**
```typescript
Response: {
  dashboard: ParentDashboard;
}
```

### Analytics Endpoints

**GET /api/analytics/student/:studentId/progress**
```typescript
Query Parameters: {
  subjectId?: string;
  startDate?: string;
  endDate?: string;
}

Response: {
  performanceTrend: PerformanceTrend;
  learningGaps: LearningGap[];
  improvements: string[];
}
```

**GET /api/analytics/class/:classId/common-gaps**
```typescript
Response: {
  commonGaps: CommonGap[];
}
```

**GET /api/analytics/assignment/:assignmentId**
```typescript
Response: {
  analytics: AssignmentAnalytics;
}
```

**POST /api/analytics/reports/generate**
```typescript
Request: {
  studentIds: string[];
  timeRange: TimeRange;
  includeRecommendations: boolean;
}

Response: {
  reportId: string;
  downloadUrl: string;
}
```

### Study Plan Endpoints

**GET /api/study-plans/student/:studentId**
```typescript
Response: {
  studyPlan: StudyPlan;
}
```

**PUT /api/study-plans/tasks/:taskId/complete**
```typescript
Response: {
  task: StudyTask;
  updatedPlan: StudyPlan;
}
```


### Self-Assessment Endpoints

**POST /api/self-assessments**
```typescript
Request: {
  studentId: string;
  subjectId: string;
}

Response: {
  assessment: SelfAssessment;
  questions: AssessmentQuestion[];
}
```

**POST /api/self-assessments/:id/submit**
```typescript
Request: {
  responses: AssessmentResponse[];
}

Response: {
  result: SelfAssessmentResult;
  identifiedGaps: LearningGap[];
  updatedStudyPlan: StudyPlan;
}
```

### Invitation Endpoints

**POST /api/invitations**
```typescript
Request: {
  educatorId: string;
  studentId: string;
}

Response: {
  invitation: Invitation;
  inviteUrl: string;
}
```

**GET /api/invitations/:code/validate**
```typescript
Response: {
  valid: boolean;
  studentName?: string;
  educatorName?: string;
}
```

### Notification Endpoints

**GET /api/notifications**
```typescript
Query Parameters: {
  unreadOnly?: boolean;
  limit?: number;
}

Response: {
  notifications: Notification[];
  unreadCount: number;
}
```

**PUT /api/notifications/:id/read**
```typescript
Response: {
  success: boolean;
}
```

## AI/ML Architecture

### Feedback Generation Pipeline

The AI Engine uses a multi-stage pipeline for generating personalized feedback:

**Stage 1: Content Extraction**
- Extract text from uploaded files (PDF, DOCX, images via OCR)
- Normalize and clean text data
- Identify document structure (sections, paragraphs, answers)

**Stage 2: Content Analysis**
- Use NLP models to analyze writing quality, grammar, coherence
- Identify key concepts and topics covered
- Compare against expected learning objectives
- Calculate performance metrics

**Stage 3: Context Enrichment**
- Retrieve student's historical performance data
- Identify learning patterns and preferences
- Consider current learning gaps and study plan

**Stage 4: Feedback Generation**
- Use LLM (GPT-4, Llama 2, or Mistral) to generate personalized feedback
- Structure feedback with strengths, improvements, and examples
- Ensure feedback is constructive, specific, and actionable
- Generate 3-5 specific feedback points per assignment


**Stage 5: Quality Assurance**
- Validate feedback meets quality standards
- Check for appropriate tone and language
- Ensure actionable advice is included
- Calculate confidence score

**Prompt Template for Feedback Generation:**
```
You are an educational AI assistant helping students improve their work.

Student Context:
- Grade Level: {grade_level}
- Subject: {subject}
- Historical Performance: {performance_summary}
- Current Learning Gaps: {learning_gaps}

Assignment Analysis:
- Content: {assignment_content}
- Performance Score: {score}
- Concepts Identified: {concepts}

Generate personalized feedback that:
1. Identifies 2-3 specific strengths with examples from the work
2. Identifies 2-3 areas for improvement with specific examples
3. Provides actionable advice for each improvement area
4. Uses encouraging and constructive tone
5. Is appropriate for the student's grade level

Format the response as JSON with strengths and improvements arrays.
```

### Learning Gap Detection

**Algorithm:**
1. Collect performance data across multiple assignments
2. Identify concepts where performance is below threshold (< 70%)
3. Calculate severity based on:
   - Performance gap magnitude
   - Consistency of poor performance
   - Foundational importance of concept
4. Assign confidence score based on data volume
5. Categorize by subject area and difficulty level

**Machine Learning Model:**
- Use supervised learning (Random Forest or Gradient Boosting)
- Features: assignment scores, concept coverage, time trends, engagement metrics
- Labels: learning gap severity (low/medium/high)
- Train on historical data with educator-validated gaps

### Adaptive Study Plan Generation

**Algorithm:**
1. Prioritize learning gaps by severity and foundational importance
2. Generate tasks targeting each gap:
   - Practice exercises
   - Review materials
   - Concept explanations
3. Estimate time based on gap severity and student pace
4. Order tasks by dependency and priority
5. Adjust plan as tasks are completed and performance improves

**Personalization Factors:**
- Student's learning pace (fast/medium/slow learner)
- Preferred learning modalities (visual, reading, practice)
- Available study time
- Historical task completion rates


### At-Risk Student Detection

**Risk Factors:**
1. Declining performance trend (3+ consecutive assignments with decreasing scores)
2. Multiple high-severity learning gaps (3+ gaps)
3. Low assignment completion rate (< 70%)
4. Poor self-assessment performance
5. Lack of engagement with study plans

**Risk Scoring:**
```python
risk_score = (
  0.3 * performance_decline_factor +
  0.25 * learning_gap_severity +
  0.2 * completion_rate_factor +
  0.15 * self_assessment_factor +
  0.1 * engagement_factor
)

if risk_score > 0.7: risk_level = 'high'
elif risk_score > 0.4: risk_level = 'medium'
else: risk_level = 'low'
```

**Intervention Recommendations:**
- High risk: Immediate educator intervention, parent notification, intensive support plan
- Medium risk: Enhanced study plan, weekly check-ins, targeted resources
- Low risk: Monitor progress, maintain current support level

### Bulk Feedback Processing

**Architecture:**
- Use message queue (RabbitMQ/Kafka) for async processing
- Worker pool processes assignments in parallel
- Maximum 5 concurrent AI requests to manage API costs
- Progress tracking via Redis cache
- Estimated time: 30 seconds per assignment (class of 30 = ~15 minutes)

**Workflow:**
1. Educator submits bulk feedback request
2. API creates job and queues assignments
3. Workers process assignments in parallel
4. Results stored in database as they complete
5. Educator notified when all feedback ready
6. Educator reviews and approves/edits feedback
7. Approved feedback distributed to students

## Security and Authentication

### Authentication Strategy

**JWT-Based Authentication:**
- Access tokens valid for 1 hour
- Refresh tokens valid for 7 days
- Tokens include user ID, role, and permissions
- Tokens signed with RS256 (asymmetric encryption)

**Password Security:**
- Passwords hashed with bcrypt (cost factor 12)
- Minimum password requirements: 8 characters, 1 uppercase, 1 lowercase, 1 number
- Password reset via email with time-limited tokens (1 hour expiry)

**Account Lockout:**
- Lock account after 3 failed login attempts
- Lockout duration: 15 minutes
- Reset counter after successful login


### Authorization and Access Control

**Role-Based Access Control (RBAC):**

**Student Permissions:**
- View own assignments and feedback
- Submit assignments
- Access own dashboard and progress data
- Complete self-assessments
- View and update own study plan

**Educator Permissions:**
- View all students in their classes
- Access class analytics and dashboards
- Review and customize AI-generated feedback
- Generate progress reports
- Create parent invitations
- View assignment analytics for their classes

**Parent Permissions:**
- View linked child's progress and dashboard
- Access child's feedback (read-only)
- View progress reports
- Configure own notification preferences
- Cannot modify any student data

**Admin Permissions:**
- Full system access
- User management
- System configuration
- Audit log access

**Authorization Middleware:**
```typescript
interface AuthorizationRule {
  resource: string;
  action: string;
  allowedRoles: UserRole[];
  ownershipCheck?: (userId: string, resourceId: string) => Promise<boolean>;
}

// Example: Student can only view their own assignments
{
  resource: 'assignment',
  action: 'read',
  allowedRoles: [UserRole.STUDENT, UserRole.EDUCATOR, UserRole.PARENT],
  ownershipCheck: async (userId, assignmentId) => {
    const assignment = await getAssignment(assignmentId);
    const user = await getUser(userId);
    
    if (user.role === UserRole.STUDENT) {
      return assignment.studentId === userId;
    }
    if (user.role === UserRole.PARENT) {
      const links = await getParentStudentLinks(userId);
      return links.some(link => link.studentId === assignment.studentId);
    }
    if (user.role === UserRole.EDUCATOR) {
      const classes = await getEducatorClasses(userId);
      return classes.some(c => c.id === assignment.classId);
    }
    return false;
  }
}
```


### Data Encryption

**Encryption at Rest:**
- Database encryption using PostgreSQL's Transparent Data Encryption (TDE)
- File storage encryption using AES-256
- Encryption keys managed via AWS KMS or similar key management service
- Separate encryption keys for different data types

**Encryption in Transit:**
- TLS 1.3 for all API communications
- Certificate pinning for mobile apps
- HTTPS enforced for all web traffic
- Secure WebSocket connections for real-time features

**Sensitive Data Handling:**
- PII (names, emails) encrypted in database
- Assignment files encrypted in object storage
- Passwords never stored in plain text
- API keys and secrets stored in environment variables or secret management service

### Audit Logging

**Logged Events:**
- User authentication (login, logout, failed attempts)
- Data access (viewing student data, assignments, feedback)
- Data modifications (creating, updating, deleting records)
- Administrative actions (user management, system configuration)
- File uploads and downloads

**Log Format:**
```typescript
interface AuditLog {
  id: string;
  timestamp: Date;
  userId: string;
  userRole: UserRole;
  action: string;
  resourceType: string;
  resourceId: string;
  ipAddress: string;
  userAgent: string;
  success: boolean;
  errorMessage?: string;
}
```

**Log Retention:**
- Audit logs retained for 2 years
- Logs stored in separate database for security
- Regular log analysis for security monitoring
- Automated alerts for suspicious activity

### Compliance Considerations

**FERPA Compliance (Family Educational Rights and Privacy Act):**
- Student education records protected
- Parent access limited to their own child's data
- Audit logs for all data access
- Data retention and deletion policies

**COPPA Compliance (Children's Online Privacy Protection Act):**
- Parental consent for students under 13
- Limited data collection from minors
- Clear privacy policy
- Secure data handling practices


## User Interface Design

### Design Principles

1. **Role-Specific Interfaces**: Tailored experiences for students, educators, and parents
2. **Clarity Over Complexity**: Simple, intuitive navigation with minimal cognitive load
3. **Data Visualization**: Charts and graphs for progress tracking and analytics
4. **Responsive Design**: Mobile-first approach for accessibility on all devices
5. **Accessibility**: WCAG 2.1 AA compliance for inclusive design

### Student Interface

**Dashboard Layout:**
- Hero section: Current progress summary with motivational messaging
- Recent feedback cards with quick access to details
- Progress charts showing performance trends
- Active study plan with task checklist
- Quick actions: Submit assignment, start self-assessment

**Assignment Submission Flow:**
1. Select subject and class
2. Drag-and-drop file upload with format validation
3. Confirmation screen with submission details
4. Redirect to dashboard with success message

**Feedback View:**
- Clean, readable layout with sections for strengths and improvements
- Highlighted examples from student's work
- Actionable advice presented as clear steps
- Option to ask questions or request clarification

### Educator Interface

**Analytics Dashboard:**
- Class performance overview with key metrics
- At-risk student alerts prominently displayed
- Common learning gaps visualization
- Quick filters by subject, time period, performance level
- Drill-down capability to individual student details

**Feedback Review Interface:**
- Side-by-side view: assignment content and AI-generated feedback
- Inline editing with rich text editor
- Approve/reject/regenerate actions
- Bulk approval for multiple students
- Preview before sending to students

**Progress Report Generator:**
- Student/class selection interface
- Time range picker
- Report customization options
- Preview before generation
- Export options (PDF, email)


### Parent Interface

**Simplified Dashboard:**
- Child's current performance in plain language
- Visual indicators (green/yellow/red) for performance levels
- Recent achievements and milestones highlighted
- Areas needing support with home activity suggestions
- Simple progress charts with explanatory text

**Notification Center:**
- Chronological list of updates
- Clear categorization (achievements, concerns, feedback)
- One-click access to related details
- Notification preferences easily accessible

### Common UI Components

**Navigation:**
- Top navigation bar with role-specific menu items
- User profile dropdown with settings and logout
- Breadcrumb navigation for deep pages
- Mobile: Hamburger menu with slide-out drawer

**Data Visualization:**
- Line charts for performance trends over time
- Bar charts for subject comparison
- Pie charts for time allocation in study plans
- Heat maps for class-wide performance patterns
- Color-coded indicators for at-risk status

**Forms:**
- Clear labels and helper text
- Real-time validation with helpful error messages
- Progress indicators for multi-step forms
- Auto-save for long forms

## Integration Points

### Student-Educator Flow

1. **Assignment Submission → Feedback Generation:**
   - Student submits assignment via Assignment Service
   - Assignment Service triggers AI Engine analysis
   - AI Engine generates feedback and stores in database
   - Notification Service alerts educator of pending review
   - Educator reviews/approves via Educator Dashboard
   - Notification Service alerts student of available feedback

2. **Learning Gap Detection → Intervention:**
   - AI Engine identifies learning gaps from performance data
   - Analytics Service flags at-risk students
   - Notification Service alerts educator
   - Educator views details in Analytics Dashboard
   - Educator can generate targeted progress report
   - System updates student's study plan automatically


### Student-Parent Flow

1. **Parent Registration:**
   - Educator creates invitation via User Service
   - Invitation email sent to parent
   - Parent clicks link and completes registration
   - User Service links parent account to student account
   - Parent gains access to child's dashboard

2. **Progress Updates:**
   - Student completes assignment and receives feedback
   - Analytics Service updates performance metrics
   - If significant change detected, Notification Service alerts parent
   - Parent views update in Parent Dashboard
   - Parent can view detailed feedback and progress charts

### Educator-Parent Flow

1. **Progress Report Sharing:**
   - Educator generates progress report via Analytics Service
   - Report stored and made available to parent
   - Notification Service alerts parent
   - Parent accesses report from dashboard
   - Parent can download or view online

### Cross-Role Data Synchronization

**Real-Time Updates:**
- WebSocket connections for live dashboard updates
- Redis pub/sub for cross-service event broadcasting
- Optimistic UI updates with eventual consistency

**Data Consistency:**
- Database transactions for multi-table operations
- Event sourcing for audit trail
- Idempotent API operations to handle retries

## Error Handling

### Client-Side Error Handling

**Network Errors:**
- Retry logic with exponential backoff
- Offline mode with local caching
- Clear error messages with suggested actions
- Automatic reconnection for WebSocket

**Validation Errors:**
- Real-time field validation
- Clear, specific error messages
- Highlight problematic fields
- Prevent form submission until resolved

**User-Friendly Messages:**
- Avoid technical jargon
- Provide actionable guidance
- Include support contact information
- Log errors for debugging


### Server-Side Error Handling

**API Error Responses:**
```typescript
interface ErrorResponse {
  error: {
    code: string;
    message: string;
    details?: any;
    timestamp: string;
    requestId: string;
  };
}

// Example error codes:
// AUTH_001: Invalid credentials
// AUTH_002: Account locked
// FILE_001: Invalid file format
// FILE_002: File size exceeds limit
// PERM_001: Insufficient permissions
// DATA_001: Resource not found
// SYS_001: Internal server error
```

**Error Recovery Strategies:**

1. **Transient Errors (Network, Timeout):**
   - Automatic retry with exponential backoff
   - Circuit breaker pattern to prevent cascade failures
   - Fallback to cached data when available

2. **Validation Errors:**
   - Return detailed validation errors to client
   - Log for pattern analysis
   - No retry (client must fix input)

3. **Authorization Errors:**
   - Return 403 Forbidden with clear message
   - Log for security monitoring
   - No retry (requires permission change)

4. **System Errors:**
   - Return 500 Internal Server Error
   - Log full stack trace
   - Alert on-call engineer
   - Graceful degradation where possible

**AI Engine Error Handling:**

1. **Analysis Failures:**
   - Retry with different model parameters
   - Fall back to simpler analysis method
   - Flag for manual educator review
   - Notify educator of processing issue

2. **LLM API Failures:**
   - Retry with exponential backoff
   - Switch to backup LLM provider
   - Queue for later processing
   - Notify educator of delay

3. **Quality Issues:**
   - Validate feedback against quality criteria
   - Regenerate if below threshold
   - Flag for educator review
   - Track quality metrics for model improvement

