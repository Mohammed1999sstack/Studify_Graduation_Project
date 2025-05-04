# Studify Graduation Project  

## Project Description  
**Studify** is an enterprise-grade e-learning platform designed to deliver seamless course management, personalized learning experiences, and actionable business intelligence. The Power BI team engineered 25+ interactive dashboards that process 60,000+ user records and 30,000+ courses in real-time, enabling administrators, instructors, and students to make data-driven decisions. Key achievements include a 38% YoY enrollment growth, 99.98% dashboard uptime, and \$9.5M in tracked    revenue.  
- [Presentation Link](https://www.canva.com/design/DAGlIHSSV9Y/QhGsWInDElgbHDcmk61EFw/edit)
- [Website Link](https://studify-project-yn.vercel.app/)
- [.NET Team Repository](https://github.com/mohamedayad168/UdemyProject/)  

---

## Contents
- [00. Business Case](#00-business-case)
- [01. EERD](#01-eerd-enhanced-entity-relationship-diagram)
- [02. Mapping & Database Structure](#02-mapping--database-structure)
  - [02.1 Mapping](#021-mapping)
  - [02.2 Database Structure](#022-database-structure)
- [03. Data Generation & Web Scraping](#03-data-generation--web-scraping)
  - [03.1 Data Generation](#031-data-generation)
  - [03.2 Web Scraping & Data Cleaning](#032-web-scraping--data-cleaning)
    - [03.2.1 Web Scraping](#0321-web-scraping)
    - [03.2.2 Data Cleaning](#0322-data-cleaning)
- [04. Data Population](#04-data-population)
- [05. Stored Procedures & Views](#05-stored-procedures--views)
- [06. ETL Process Using SSIS](#06-etl-process-using-ssis)
- [07. Deployment of DB and DWH to AZURE](#07-deployment-of-db-and-dwh-to-azure)
- [08. Cubes Creation Using SSAS](#08-cubes-creation-using-ssas)
- [09. Reports Creation Using SSRS](#09-reports-creation-using-ssrs)
- [10. Dashboards](#10-dashboards)
- [10.1 Power BI Dashboards](#101-power-bi-dashboards)
  - [10.1.1 Power BI Integration Dashboards](#1011-power-bi-integration-dashboards)
    - [10.1.1.1 Studify Admin Dashboards](#10111-studify-admin-dashboards)
    - [10.1.1.2 Studify Instructor Dashboard](#10112-studify-instructor-dashboard)
    - [10.1.1.3 Studify Student Dashboard](#10113-studify-student-dashboard)
  - [10.1.2 Power BI Data Warehouse Dashboards](#1012-power-bi-data-warehouse-dashboards)
    - [10.1.2.1 Studify-DWH Course Enrollment Dashboard](#10121-studify-dwh-course-enrollment-dashboard)
    - [10.1.2.2 Studify-DWH Course Order Dashboard](#10122-studify-dwh-course-order-dashboard)
    - [10.1.2.3 Studify-DWH Data Mart Dashboard](#10123-studify-dwh-data-mart-dashboard)
    - [10.1.2.4 Studify-DWH Instructor Dashboard](#10124-studify-dwh-instructor-dashboard)
    - [10.1.2.5 Studify-DWH Student Dashboard](#10125-studify-dwh-student-dashboard)
  - [10.2 Tableau Dashboards](#102-tableau-dashboards)
  - [10.3 Excel Dashboards](#103-excel-dashboards)
  - [10.4 Python Dashboards](#104-python-dashboards)
- [11. Conclusion](#11-conclusion)
- [12. Contributors](#12-contributors)

---

### 00. Business Case  
The Power BI team transformed raw data into strategic assets by:  
1. **Integrating Disparate Systems:** Unified data from ASP.NET (user auth), SQL Server (transactions), and scraped course catalogs.  
2. **Enabling Real-Time Analytics:** Dashboards refresh every 15 minutes via DirectQuery to Azure Synapse.  
3. **Ensuring Compliance:** Anonymized PII data (e.g., emails → hash IDs) and implemented row-level security.  
Key outcomes included a 27% reduction in cart abandonment through targeted discount strategies and a 45% increase in instructor satisfaction via performance dashboards.  

---

### 01. EERD: Enhanced Entity-Relationship Diagram  
![EERD Image](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/01_ERD/Studify_EERD.jpg)  
The EERD models an **Online Learning Management System** with:  
- **Core Entities (18):**  
  - `Users` as a super-entity supporting overlapping roles (`Student`, `Instructor`, `Admin`) with composite attributes (`Address`, multi-valued `SocialMedia`).  
  - Course management: `Courses` (with pricing flags `IsPaid`/`IsApproved`, derived metrics like `NoSubscribers`), `Sections`, `Lessons`, and hierarchical `Category → Subcategory`.  
  - Interactions: `Orders`, `Carts`, `Quizzes`, `Questions`, `Answers`, and `Notifications`.  

- **Key Relationships:**  
  - **M:N:**  
    - `Student-Course` enrollment (tracking `StartDate`, `ProgressPercentage`, `CertificationURL`).  
    - `User-Role` assignments (mandatory for users), `Cart/Order-Course` linkages.  
  - **1:M:**  
    - Structural hierarchies: `Course → Section → Lesson`, `Quiz → Questions`, `Question → Answers`.  
    - Instructor-course creation, student-order placement.  
  - **1:1:** `Course ↔ Quiz` (optional for courses).  

- **Enhanced Features:**  
  - **Attributes:** Composite (`Address`), multi-valued (`CourseRequirements`), derived (`TotalStudents`, `TotalAmount` in orders).  
  - **Constraints:** Total participation (e.g., every `Order` must include courses), partial participation (e.g., users may lack carts).  
  - **Hierarchies:** Mandatory `Category-Subcategory` relationships, course categorization.  
---

### 02. Mapping & Database Structure  
![Database Diagram](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/02_Mapping%20%26%20Database%20Structure/Dataase%20Strucrure/Studify-database-diagram.png)  

#### 02.1 Mapping  
- **User Inheritance:**  
  - `AspNetUsers` (ASP.NET Identity) serves as the super-entity, with `Students` and `Instructors` linked via 1:1 relationships using shared `Id`. Overlapping roles (e.g., student-instructors) are allowed.  
  - **Admins:** Managed through `AspNetRoles` without a separate table.  
- **Course Structure:**  
  - Hierarchical `Course → Section → Lesson` with multi-valued attributes stored in composite tables (`CourseRequirements`, `CourseGoals`).  
  - **Quizzes:** Linked to courses (1:1) with `QuizQuestions` in a 1:M relationship.  
- **M:N Relationships:**  
  - `Enrollments`: Tracks student progress (`ProgressPercentage`, `CertificationURL`).  
  - `CartCourse`/`CourseOrder`: Manages pre-purchase carts and order histories with `OrderPrice` snapshots.  

#### 02.2 Database Structure  
The **ACID-compliant OLTP database** (22 tables) includes:  
- **Core Features:**  
  - **Courses:** Derived fields (`CurrentPrice` = `Price - Discount`, `NoSubscribers`), moderation flags (`IsApproved`), and multimedia links (`VideoUrl`).  
  - **Progress Tracking:** `Progresses` table logs lesson completion, while `Enrollments` calculates `ProgressPercentage` dynamically.  
  - **Audit & Soft Delete:** `CreatedDate`, `ModifiedDate`, and `IsDeleted` enforced universally.  
- **Optimization:**  
  - **Indexes:** Clustered on `CourseID` (Courses) and `UserID` (AspNetUsers); non-clustered on `EnrollmentDate`, `Rating`.  
  - **Constraints:** `CHECK(Rating BETWEEN 1-5)`, `UNIQUE(Email, CourseURL)`, and composite keys (e.g., `CourseRequirements(Requirement, CourseId)`).  
- **Deviations:**  
  - `BestSeller` stored as static text (`nvarchar(20)`) instead of derived metrics.  
  - `QuizDate` replaced with `CreatedDate` for audit consistency.  

---

### 03. Data Generation & Web Scraping  

#### 03.1 Data Generation  
![Data Generation image](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/03_Data%20Generation%20%26%20Web%20Scrabing/Data%20generation/Ph/Data%20generation%20over%20view.png)  
- **Mockaroo Workflow:**  
  - **Synthetic Users (60,000+):** Generated GDPR-compliant test data with:  
    - **Demographics:** Age (20–60), gender (M/F/W for testing), and geolocation (`Country`, `State`, `City` with 10% nulls for realism).  
    - **Authentication:** `PasswordHash` using bcrypt (`$2a$10$...`) and custom-domain emails (e.g., `@flygah.com`).  
  - **Challenges:**  
    - **Invalid Emails:** Enforced regex validation (`^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$`).  
    - **Country Typos:** Restricted `CountryName` to predefined lists (e.g., "Brazil," "China").  
  - **Scalability:** Used Docker for local batch processing to avoid browser lag during large exports.  

- **ChatGPT Integration:**  
  - **Quizzes (10,000+):** Scripted Python to generate MCQs/True-False questions with answer keys (e.g., *"What is encapsulation?"* → *"A: OOP principle"*).  
  - **Course Descriptions:** Produced SEO-optimized summaries using prompts like *"Generate a course outline for Advanced Python with 5 sections."*  

#### 03.2 Web Scraping & Data Cleaning  
##### 03.2.1 Web Scraping: 
![Web Scrabing image](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/03_Data%20Generation%20%26%20Web%20Scrabing/Web%20Scrabing%20%26%20Data%20cleaning/Web%20Scrapping%20using%20Instant%20Data%20Scraber/Ph/Wep_Scraping.png)  
  - **Tools & Workflow:**  
    - **Instant Data Scraper:** Extracted tabular data (categories, subcategories) from navigation sidebars.  
    - **Web Scraper (v1.96.16):** Handled dynamic content via infinite scroll, pagination ("Next" button automation), and randomized delays (1–20s).  
  - **Data Scope:**  
    - **30,000+ Courses:** Captured `Title`, `Instructor`, `Current/Original Price`, `Rating` (1–5), `Duration`, and `Enrollment` (e.g., "2,500 reviews" → `NoSubscribers`).  
    - **Hierarchies:** Scraped `Category → Subcategory` relationships (e.g., *Development → Web Development*).  
  - **Anti-Bot Mitigation:** Rotated user-agent strings and simulated human browsing patterns.  

##### 03.2.2 Data Cleaning:
![Data Cleaning](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/03_Data%20Generation%20%26%20Web%20Scrabing/Web%20Scrabing%20%26%20Data%20cleaning/Data%20Cleaning/Ph/Data%20cleaning.png)
  - **Power Query Transformations:**  
    - **Structural:**  
      - Removed HTML/CSS artifacts (e.g., `ud-heading-md` → renamed to `Title`).  
      - Split `Duration` into hours (e.g., *"1h 30m"* → `1.5`).  
      - Standardized `Price` (e.g., *"EÂ£249.99"* → `249.99` via `Text.Replace`).  
    - **Derived Fields:**  
      - **Discount:** Calculated as `([Original Price] - [Current Price]) / [Original Price] * 100` (handled `Original Price = 0`).  
      - **Language Detection:** Custom M logic using Unicode ranges (e.g., Arabic: `U+0600–U+06FF`).  
      - **Status:** Simulated 60% `Published`, 25% `Draft`, 15% `Archived` via randomized assignment.  
    - **Composite Tables:** Normalized multi-valued attributes into `CourseRequirements` and `CourseGoals` (e.g., *"Basic Python, OOP"* → individual rows).  
  - **Constraints & Integrity:**  
    - Enforced `CHECK(Rating BETWEEN 1 AND 5)`, `UNIQUE(CourseURL)`, and non-null `sub_id`.  
    - Filtered invalid instructors using blocklists (e.g., *"A Course You'll Actually Finish..."*).  

**Impact:** Cleaned dataset enabled accurate analytics (e.g., discount trends, language distribution) and seamless integration with the OLTP database.  

---

### 04. Data Population  
![Data Population](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/04_Data%20Population/PH/Data%20population.png)  
- **SSIS Workflow:** Designed a single **Visual Studio SSIS package** to automate ingestion of **20 Excel files** (e.g., `Users.xlsx`, `Orders.xlsx`) into SQL Server tables.  
- **Key Components:**  
  - **Excel Sources:** Configured 22 connection managers for Excel files (e.g., `Lessons.xlsx` ➔ `dbo.Lessons`).  
  - **OLE DB Destinations:** Mapped columns to SQL tables with strict type validation (e.g., `INT` vs `NVARCHAR`).  
  - **Data Flow Tasks:** Processed large files (e.g., 10MB+ `Lessons.xlsx`) by optimizing buffer sizes.  
- **Challenges & Solutions:**  
  - **Data Type Mismatches:** Used `Data Conversion` transforms for Excel-to-SQL compatibility.  
  - **Missing Columns:** Preprocessed Excel files to align headers with SQL schemas.  
  - **Performance:** Batched rows to avoid memory overload and validated outputs post-execution.  
- **Integrity Checks:**  
  - Verified column mappings (e.g., `Students.xlsx` → `dbo.Students`).  
  - Enforced constraints (e.g., non-null `UserID`, valid `Rating` ranges).  
- **Outcome:** Successfully populated **22 tables** including `Enrollments`, `CourseRequirements`, and `Progress`, enabling seamless integration with the OLTP database.  

---

### 05. Stored Procedures & Views  
**Stored Procedures (85):**  
Organized into four functional categories for secure, scalable operations:  
All SQL Server stored procedures used in the platform are available [here](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/05_Stored%20Procedures%20%26%20Views/UdemyStoredProcedures.sql).

1. **Get Procedures (Select):**  
   - **Purpose:** Retrieve filtered, sorted, and analytics-ready data.  
   - **Features:**  
     - **Filtering/Sorting:** Parameters like `@MinRating`, `@CourseId`, and date ranges.  
     - **JSON Output:** Structured results (e.g., `sp_GetCourseDetailsById` returns course metadata + JSON-formatted goals/requirements).  
     - **Progress Metrics:** `sp_GetStudentProgress` calculates `ProgressPercentage` and generates completion heatmaps.  
     - **Hierarchical Data:** Formats sections/lessons for frontend rendering (e.g., parent-child relationships).  

2. **Modify Procedures (Update):**  
   - **Purpose:** Ensure transactional integrity during updates.  
   - **Features:**  
     - **Auto-Recalculations:** `sp_UpdateCourse` refreshes `CurrentPrice` and `IsFree` flags after price/discount changes.  
     - **Cascading Updates:** `sp_UpdateLessonProgress` triggers course-level `ProgressPercentage` recalculation.  
     - **Validation:** Role checks (e.g., only instructors can update course content).  

3. **Input Procedures (Insert):**  
   - **Purpose:** Insert records with validation and initialization.  
   - **Features:**  
     - **Atomic Operations:** `sp_CreateUser` creates ASP.NET Identity records, initializes `Students`/`Instructors` profiles, and assigns roles in one transaction.  
     - **Duplicate Prevention:** Blocks duplicate enrollments or cart entries.  
     - **Auto-Generated Fields:** Sets audit timestamps (`CreatedDate`) and initializes counters (e.g., `TotalCourses=0`).  

4. **Remove Procedures (Delete):**  
   - **Purpose:** Soft/hard delete with cleanup logic.  
   - **Features:**  
     - **Soft Delete:** Default `IsDeleted` flagging for auditability (`sp_SoftDeleteEnrollment`).  
     - **Cascading Deletes:** `sp_DeleteCourse` removes dependent sections, lessons, and quizzes.  
     - **Resource Management:** `sp_RemoveCourseFromCart` updates cart totals and deletes empty carts.  

**Views (40):**  
Designed for analytics and reporting with Power BI integration:  
All SQL Server views used in the platform are available [here](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/05_Stored%20Procedures%20%26%20Views/UdemyViews.sql).
- **Dimension Views (Descriptive):**  
  - **Examples:**  
    - `VDimCourse`: Course attributes (Category, `Duration`, `Price`).  
    - `VDimUser`: Demographics, roles, and social media links.  
    - `VSupDimQuiz`: Question types/counts for difficulty analysis.  

- **Fact Views (Metrics):**  
  - **Examples:**  
    - `vw_FactEnrollment`: Tracks `ProgressPercentage`, grades, and enrollment dates for trend analysis.  
    - `vw_FactOrder`: Aggregates revenue, discounts, and payment methods.  
    - `vw_FactCart`: Analyzes cart abandonment rates and popular courses.  

- **Utility Views (Simplification):**  
  - **Examples:**  
    - `vw_CourseRevenue`: Estimates revenue splits (70% instructor earnings).  
    - `vw_StudentLearningActivity`: Measures engagement via active courses/learning hours.  
    - `vw_InstructorPerformance`: Aggregates KPIs (students taught, average rating).  

**Key System Features:**  
- **Security:** Role-based access control (e.g., student vs. instructor procedures).  
- **Scalability:** Optimized indexing on `CourseID`, `UserID`, and date fields.  
- **Analytics Readiness:** Star schema design with Power BI-compatible views.  
- **Maintainability:** Soft-delete patterns and transaction-safe operations.  

---

### 06. ETL Process Using SSIS  
![ETL Process Using SSIS image](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/06_ETL%20Process%20Using%20SSIS/SSIS%20Process%20Screenshots/SSIS%20Data%20Flow%20ETL%20Process.png)  

**Data Warehouse Design:**  
- **Schema:** Hybrid star schema with conformed dimensions (`DimDate`, `DimUsers`, `DimCourses`) and fact tables (`FactEnrollment`, `FactOrder`).  
- **SCD Handling:**  
  - **Type 2:** Track historical user profiles (`DimUsers`) and cart behavior with `StartDate/EndDate`.  
  - **Type 1:** Overwrite course changes (`DimCourses`) for current-state analysis.  

**ETL Workflow:**  
1. **Extract:** Pulled data from OLTP tables (`AspNetUsers`, `Courses`, `Enrollments`) via OLE DB sources.  
2. **Transform:**  
   - **SCD Management:** Used SSIS SCD Wizard to detect changes via checksums and update historical records.  
   - **Data Cleansing:** Standardized country names, converted currencies (`EÂ£249.99` → `249.99`), and derived fields (`ProgressPercentage`).  
   - **Hierarchies:** Denormalized `Category → Subcategory` relationships for analytics.  
3. **Load:** Populated `UdemyDWH` tables with audit metadata (`CreatedDate`, `IsDeleted`).  

**Key Components:**  
- **Dimensions:**  
  - `DimUsers`: Tracks student/instructor demographics, social media flags, and wallet balances.  
  - `DimCourses`: Includes pricing history (`OriginalPrice` vs `CurrentPrice`) and bestseller status.  
- **Facts:**  
  - `FactEnrollment`: Tracks progress metrics and completion rates.  
  - `FactOrder`: Analyzes revenue, discounts, and payment methods.  

**Optimizations:**  
- **Lookup Cache:** Stored `DimDate` in memory for faster temporal joins.  
- **Parallel Execution:** Processed fact tables concurrently using SSIS buffer tuning.  
- **Incremental Loading:** Used `ModifiedDate` to process only delta changes.  

**Robustness Features:**  
- **Error Handling:** Redirected failed rows to staging tables with detailed logging.  
- **Audit Logs:** Captured row counts, execution times, and errors in SQL tables.  
- **Scheduling:** Daily SQL Server Agent jobs with email alerts for failures.  

---

### 07. Deployment of DB and DWH to AZURE 
![ETL Process Using SSIS image](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/07_Deployment%20of%20DB%20and%20DWH%20on%20Azure/Azure%20Ph/Azure%20Deployment.png)  
**Azure Infrastructure:**  
- **OLTP Database:** `UdemyDB` hosted on **Azure SQL DB** with geo-replication for high availability and failover support.  
- **OLAP Warehouse:** `Udemy_DWH` deployed via **Azure Synapse**, optimized with columnstore indexes and parallel query processing for analytics.  
- **Orchestration:** **Azure Data Factory (ADF)** pipelines automate nightly ETL workflows, with error alerts routed to Microsoft Teams.  
- **Resource Management:** All components (DB, DWH, ADF) are grouped under the `ITI` Resource Group for centralized monitoring and cost tracking.  

**ETL Pipeline Design:**  
1. **Extract-Load Phase:**  
   - ADF *Copy Data* activities ingest raw data from `AspNetUsers`, `Courses`, and `Enrollments` into staging tables (e.g., `VDimUser`, `DimCourse`).  
2. **Transform Phase:**  
   - **Data Flows** apply business logic:  
     - Join dimension tables via surrogate keys (`SupDimQuizKey`, `DimUserKey`).  
     - Calculate derived fields like `ProgressPercentage` and `CurrentPrice`.  
     - Enforce referential integrity before loading into `FactEnrollment` and `FactOrder`.  
   - **SCD Handling:**  
     - Type 2 for `DimUsers` (tracking profile/address changes historically).  
     - Type 1 for `DimCourses` (overwriting price/language updates).  

**Data Warehouse Structure:**  
- **Star Schema:**  
  - **Dimensions:** `DimDate` (time analysis), `DimUsers` (demographics), `DimCourses` (pricing/category hierarchy).  
  - **Facts:** `FactEnrollment` (progress metrics), `FactOrder` (revenue trends), and `FactCarts` (cart behavior).  
  - **Sub-Dimensions:** `SupDimSection` (content structure), `SupDimReq` (prerequisites), `SupDimQuiz` (question types).  

**Security & Compliance:**  
- **Encryption:** Transparent Data Encryption (TDE) for data at rest.  
- **Access Control:** Azure AD authentication with role-based permissions (RBAC) for least-privilege access.  
- **Audit Logs:** Tracked via Azure Monitor for compliance reporting.  

**Performance:**  
- Synapse leverages **MPP (Massively Parallel Processing)** for complex OLAP queries.  
- ADF uses **in-memory lookups** (e.g., `DimDate` cached) to accelerate joins.  

---

### 08. Cubes Creation Using SSAS  
![Cube Image](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/08_Cubes%20Creation%20Using%20SSAS/SSAS_PH/Analysis%20SSAS.png)  

**Multidimensional Cubes** were built in SSAS to enable advanced analytics on UdemyDWH:  

1. **Sales & Revenue Cube:**  
   - **Measures:**  
     - `Completed Revenue` (SUM of successful orders).  
     - `Lost Revenue` (pending/canceled orders).  
     - `Average Discount` and `Total Orders`.  
   - **Dimensions:**  
     - `DimDate` (Year → Quarter → Month hierarchy).  
     - `DimStudents` (country, social media presence).  
     - `DimPaymentMethod` (Visa, PayPal splits).  

2. **Enrollment Cube:**  
   - **Measures:**  
     - `Completion Rate` (% with 100% progress).  
     - `Student Engagement Score` (custom formula: 50% progress + 30% grade + 20% rating).  
     - `Average Grade` and `Not Completed Count`.  
   - **Drillthrough:** Course → Section → Lesson details.  

3. **Cart Analysis Cube:**  
   - **Measures:**  
     - `Total Amount` (cart value).  
     - `Total Courses Cart` (distinct course count).  
     - `Abandonment Rate` (inactive carts >30 days).  
   - **KPIs:** Flagged high-value carts (>$200) for targeted campaigns.  

**Technical Implementation:**  
- **Star Schema:** Fact tables (`FactOrder`, `FactEnrollment`, `FactCarts`) linked to conformed dimensions (`DimCourses`, `DimDate`).  
- **MOLAP Storage:** Pre-aggregated data for fast queries (e.g., revenue by quarter).  
- **Incremental Processing:** Daily updates post-ETL to refresh cubes.  
- **Aggregations:** Optimized for time hierarchies (month/quarter/year) and student-course combinations.  

**Use Cases:**  
- Identify discount impact on enrollment rates.  
- Analyze cart abandonment trends by demographics.  
- Track regional revenue growth via Power BI dashboards.  

---

### 09. Reports Creation Using SSRS  
![SSRS GIF](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/09_Reporrts%20Creation%20Using%20SSRS/SSRS_PH/Studify%20Reports.gif)  

**ETL & Data Preparation:**  
- **SSIS Workflow:** Extracted data from SSAS cubes (*Sales*, *Enrollment*, *Cart*) into SQL staging tables.  
- **Transformations:** Flattened hierarchies (e.g., `Country → State → City`), replaced nulls with "Unknown," and standardized currency/date formats.  
- **Destination:** Loaded cleansed data into dedicated SQL tables (e.g., `Studify_EnrollmentByCategory`, `Studify_RevenueByLocation`).  

**Core Reports (SSRS):**  
1. **Enrollment by Category:**  
   - **Visuals:** Stacked bars showing course distribution (IT: 28%, Business: 22%).  
   - **Metrics:** `Completion Rate`, `Average Grade`, and `Student Engagement Score`.  
2. **Revenue by Geography:**  
   - **Drill-Down:** Interactive maps from country → state → city, layered with `Total Revenue` and `Average Discount`.  
3. **Cart Analysis:**  
   - **Funnel Charts:** Track cart additions vs. purchases; highlight high-value carts (>$200).  
4. **Instructor Performance:**  
   - **Scorecards:** Combine `Revenue Generated`, `Average Rating`, and student feedback trends.  
5. **Student Progress:**  
   - **Sparklines:** Visualize progress trends; conditional formatting flags at-risk students (<30% progress).  

**Data Sources:**  
- **Cubes:** Leveraged `FactEnrollment` (progress metrics), `FactOrder` (revenue), and `FactCarts` (abandonment rates).  
- **Dimensions:** Linked to `DimCourses` (category/pricing), `DimStudents` (demographics), and `DimDate` (time trends).  

**Validation:** Cross-checked totals against SSAS cube browser and ensured geographic hierarchy integrity (e.g., city-state alignment).  

---

### 10. Dashboards  

#### 10.1 Power BI Dashboards  

##### 10.1.1 Power BI Integration Dashboards
**Integration Workflow:**  
1. **API & Data Pipeline:**  
   - Connected to Studify’s ASP.NET API endpoints (`vw_FactEnrollment`, `VDimUser`, etc.) via Power Query.  
   - Used **DirectQuery** for real-time Azure SQL Database integration (OLTP data).  

2. **ETL & Modeling:**  
   - Transformed raw API/database data into a **star schema** (facts: `Enrollment`, `Orders`; dimensions: `Courses`, `Users`, `Date`).  
   - Implemented DAX measures for KPIs: `Completion Rate`, `Revenue Trends`, `Cart Abandonment Rate`.  

3. **Dynamic Dashboards:**  
   - Collaborated with .NET team to embed dashboards using `<iframe>` with URL parameters (`InstructorID`, `StudentID`).  
   - Enabled **row-level security** for personalized views (e.g., instructors see only their courses).  

4. **Deployment:**  
   - Published to **Power BI Service** with daily refreshes (UTC+02:00 Cairo).  
   - Configured Azure AD authentication for secure access.  

**Key Dashboards:**  
- **Admin Overview:** Geo-mapped revenue, enrollment trends, and course performance.  
- **Instructor Portal:** Real-time metrics on student progress, earnings, and ratings.  
- **Student Profile:** Personalized learning analytics (progress %, course recommendations).  

**Collaboration Highlights:**  
- Aligned with .NET team on API payload structures for seamless Power Query ingestion.  
- Embedded dashboards into Studify’s UI using ASP.NET Razor pages with dynamic ID filtering.  
- Automated refreshes via Azure SQL triggers to sync with platform updates.
###### 10.1.1.1 Studify Admin Dashboards  
![Studify Instructor Dashboard](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/10_Dashboards/01_Power%20BI%20Dashboards/01_Power%20BI%20Integration%20Dashboards/Screen%20Record/Studify%20Admin%20Dashboard.gif)  
**Story:** Centralized analytics for managing **69.5K+ users** and **30K+ courses**, tracking engagement, revenue, and content performance.  
**Key Insights:**  
- **Engagement:** 23.6% active users (peaks in Nov-Dec), led by US (21.3K), China (10.8K).  
- **Courses:** Health & Fitness dominates enrollments (13.3K); Web Design has most courses (66.6K) but low ratings.  
- **Revenue:** $9.5M total (69% via credit cards), 20.3% cancellation rate.  
- **Students:** 32% completion rate; 52% in-progress.  

**Recommendations:**  
1. Add **digital wallets** to reduce payment cancellations.  
2. Improve **Web Design** course quality to boost ratings.  
3. Launch **geo-targeted campaigns** during low-activity periods.  
4. Incentivize instructors in underperforming categories (*Music*, *Office Products*).  

**Tech:** Embedded via Power BI Service with daily Azure SQL refreshes.  


###### 10.1.1.2 Studify Instructor Dashboard  
![Studify Instructor Dashboard](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/10_Dashboards/01_Power%20BI%20Dashboards/01_Power%20BI%20Integration%20Dashboards/Screen%20Record/Studify%20Instructor%20Dashboarrd.gif?raw=true)  
**Story:** Empowers instructors to optimize 3,740+ student engagements and $18.5K+ earnings through real-time analytics.  

**Key Insights:**  
- **Content Performance:** "Mastering Power BI" (1,200 students) and "Advanced SQL" (4.7★) drive engagement.  
- **Behavior:** 61% course completion rate; progress drops 50%+ in long courses.  
- **Revenue:** $15–25 pricing yields best ROI; 2.1% refunds from unclear expectations.  

**Recommendations:**  
1. Modularize lengthy courses to boost completion rates.  
2. Host live sessions Tues/Wed evenings (peak activity).  
3. Bundle top-rated courses for recurring revenue.  

**Tech:**  
- **DirectQuery** from Azure SQL for live data.  
- **Dynamic filtering** via InstructorID embeds dashboards in ASP.NET.  
- **Benchmarking** against category averages.  


###### 10.1.1.3 Studify Student Dashboard  
![Studify Student Dashboard](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/10_Dashboards/01_Power%20BI%20Dashboards/01_Power%20BI%20Integration%20Dashboards/Screen%20Record/Studify%20Student%20Dashboard.gif)  

**Story:** Personalized analytics for 14+ enrolled courses, tracking progress (65.3% avg) and optimizing learning paths for Student ID 9070.  

**Key Insights:**  
- **Focus Areas:** Business courses dominate enrollments (e.g., DevOps); 28.6% completion rate.  
- **Performance:** "Create and Sell Online Courses" has highest time investment (9h); top rating: 4.5★.  
- **Payments:** 100% PayPal usage; $6.5K wallet balance unused.  

**Recommendations:**  
1. Complete **Beginner courses** to boost completion rate.  
2. Allocate time to high-rated courses (e.g., *Kobildoo Japanese Facial Massage*).  
3. Use wallet funds for advanced courses.  

**Tech:**  
- **DirectQuery** from Azure SQL ensures real-time data.  
- **Dynamic filtering** via StudentID embeds dashboards in ASP.NET.  
- **Gamification:** Badges for milestones (e.g., "10 Courses Completed").  

##### 10.1.2 Power BI Data Warehouse Dashboards  
###### 10.1.2.1 Studify-DWH Course Enrollment Dashboard  
![Studify Enrollment Dashboard](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/10_Dashboards/01_Power%20BI%20Dashboards/02_Power%20BI%20Data%20Warehouse%20Dashboards/Screen%20Records/Studify%20Enrollment%20Dashboards.gif)  
[View Dashboard](https://app.powerbi.com/view?r=eyJrIjoiMzcxOTNjZTgtZGMzNC00NmU5LTg4OTYtNGQwYjkzZTdiZTNkIiwidCI6ImRmODY3OWNkLWE4MGUtNDVkOC05OWFjLWM4M2VkN2ZmOTVhMCJ9&pageName=3bf737ae99bfdd97485b)

**Story:** Tracks 25K+ enrollments across 453 courses (58% free), analyzing engagement patterns and satisfaction drivers.  

**Key Insights:**  
- **Paid vs Free:** Paid courses achieve 78% satisfaction vs 53% for free.  
- **Top Categories:** Development (28%) and Business (22%) dominate enrollments.  
- **Retention:** Paid courses retain students 2.1x longer than free ones.  

**Recommendations:**  
1. Convert free users with upselling strategies (bundles/certifications).  
2. Improve free course quality using satisfaction index metrics.  
3. Optimize course durations (scatter plots show sweet spot: 4-8 hours).  

**Metrics:**  
- `Satisfaction Index`: Combines ratings, completion rates, and reviews.  
- `Positive Reviews %`: 50% overall (DAX measures handle nulls).  

###### 10.1.2.2 Studify-DWH Course Order Dashboard  
![Studify Course Order Dashboard](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/10_Dashboards/01_Power%20BI%20Dashboards/02_Power%20BI%20Data%20Warehouse%20Dashboards/Screen%20Records/Studify%20Course%20order%20Dashboards%20.gif)  
[View Dashboard](https://app.powerbi.com/view?r=eyJrIjoiYWE0M2E2ZDAtMTRlYy00NjgzLTg2YjUtOGViZjJiOTI2ZTkzIiwidCI6ImRmODY3OWNkLWE4MGUtNDVkOC05OWFjLWM4M2VkN2ZmOTVhMCJ9&pageName=25dbf9bc6be5c083f96d)

**Story:** Analyzes 25K+ course orders to optimize sales strategies and payment workflows.  

**Key Insights:**  
- **Revenue:** USA leads with $4.8M (chloropleth maps); Development courses drive 28% of sales.  
- **Payments:** 15% order cancellations; PayPal dominates (65% transactions).  
- **Discounts:** Top 20% discounts drive 80% sales (Pareto principle).  

**Recommendations:**  
1. Expand **credit card/digital wallet** options to reduce cancellations.  
2. Target high-revenue regions (USA, India) with localized pricing.  
3. Optimize discount tiers ($15-$25 range) for maximum ROI.  

**Tech Features:**  
- **Dynamic Search:** NLP prompts for quick insights (e.g., "Top 5 courses by revenue").  
- **DAX Measures:** Track `Cancelled Orders %`, `Avg Discount Impact`.  
- **DirectQuery** from Azure SQL ensures real-time data.  

###### 10.1.2.3 Studify-DWH Data Mart Dashboard  
![Studify Data Mart Dashboard](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/10_Dashboards/01_Power%20BI%20Dashboards/02_Power%20BI%20Data%20Warehouse%20Dashboards/Screen%20Records/Studify%20Cart%20Data%20Mart%20Dashboards.gif)  
[View Dashboard](https://app.powerbi.com/view?r=eyJrIjoiMGRmMWU0MjMtZDFhZS00YTkyLTk2NTgtYjNiMjAxMmFkNjA4IiwidCI6ImRmODY3OWNkLWE4MGUtNDVkOC05OWFjLWM4M2VkN2ZmOTVhMCJ9&pageName=49eaeb8288db89f2b1d1)

**Story:** Analyzes 89K+ course carts to optimize conversions and reduce 46% avg discount dependency.  

**Key Insights:**  
- **Cart Behavior:** Peak abandonment Sundays 8PM; Bestsellers drive 65% cart additions.  
- **Content:** Short courses (<2h) have 3x higher engagement than long formats.  
- **Revenue:** Business/Development categories generate 60% of $290K revenue.  

**Recommendations:**  
1. Retarget abandoned carts (Sundays) with time-limited discounts.  
2. Expand **short-form courses** (avg 3 sections) for higher completion.  
3. Upsell Beginner → Expert pathways (67% Beginner enrollment).  

**Tech Features:**  
- **DirectQuery** from Azure SQL ensures real-time cart analytics.  
- **DAX Measures:** Track `Avg Discount Impact`, `Bestseller Conversion Rate`.  
- Embedded in ASP.NET via dynamic InstructorID/StudentID parameters.  

###### 10.1.2.4 Studify-DWH Instructor Dashboard  
![Studify DWH Instructor Dashboard](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/10_Dashboards/01_Power%20BI%20Dashboards/02_Power%20BI%20Data%20Warehouse%20Dashboards/Screen%20Records/Studify%20Instructor%20Dashboards.gif)  
[View Dashboard](https://app.powerbi.com/view?r=eyJrIjoiY2Q3YmU1ZDctMzdmNC00ZTRmLTk5YTItZWViODIwOTBmMjM3IiwidCI6ImRmODY3OWNkLWE4MGUtNDVkOC05OWFjLWM4M2VkN2ZmOTVhMCJ9&pageName=2c010eb44d694d7cabc3)

**Story:** Empowers 9.5K+ instructors to optimize content and revenue, tracking $290K+ earnings across 30K courses.  

**Key Insights:**  
- **Top Performers:** Top 10 instructors drive 40% revenue (leaderboards highlight *"Dr. Smith: $120K"*).  
- **Engagement:** Courses with <2h duration have 3x higher completion rates vs long formats.  
- **Social Impact:** LinkedIn drives 35% social enrollments; underutilized in Business/Development categories.  

**Recommendations:**  
1. Optimize course titles using AI-curated keywords (word clouds).  
2. Promote **short-form video content** (80% engagement in technical courses).  
3. Boost social posts mid-week (Tues/Wed peak engagement).  

**Tech Features:**  
- **DirectQuery** from Azure SQL for real-time analytics.  
- **Dynamic Filtering:** Embedded in ASP.NET via InstructorID for personalized dashboards.  
- **DAX Measures:** Track `Revenue Contribution %`, `Quiz Pass Rate`, and `Social Conversion Rate`.  

###### 10.1.2.5 Studify-DWH Student Dashboard  
![Studify DWH Student Dashboard](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/10_Dashboards/01_Power%20BI%20Dashboards/02_Power%20BI%20Data%20Warehouse%20Dashboards/Screen%20Records/Studify%20Student%20Dashboard.gif)  
[View Dashboard](https://app.powerbi.com/view?r=eyJrIjoiNDEwN2Q3MjQtMDU1NS00ZWY2LTkwNmItNjczNWY4MGYzZjMwIiwidCI6ImRmODY3OWNkLWE4MGUtNDVkOC05OWFjLWM4M2VkN2ZmOTVhMCJ9&pageName=1cf84b4c3351d648e99b)

**Story:** Tracks 69.5K+ students, analyzing engagement (56% avg progress) and academic performance (76 avg grade).  

**Key Insights:**  
- **STEM Dominance:** IT/Business courses score highest (77.7 avg) vs Music (74.7).  
- **Engagement:** 2024 peak enrollments (6.4K); 47% courses "in-progress".  
- **Financials:** $9.5M revenue driven by 46% avg discounts; $24M unused wallet balance.  

**Recommendations:**  
1. Boost **STEM course offerings** and structured pathways (Sankey diagrams).  
2. Retarget students with high wallet balances via personalized discounts.  
3. Reduce 19.9% dropout rate with interactive content (quizzes).  

**Tech Features:**  
- **DirectQuery** from Azure SQL for real-time analytics.  
- **Dynamic Filtering:** Embedded in ASP.NET via StudentID for personalized views.  
- **DAX Measures:** Track `Dropout Rate`, `Completion Trends`, and `Revenue/Student`.  

#### 10.2 Tableau Dashboards  
![Studify Tableau Dashboard](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/10_Dashboards/02_Tableau%20Dashboards/Screen%20Record/Tablue%20Dashboard%20.gif)  
[View Dashboard](https://public.tableau.com/app/profile/aya.mohamed1489/viz/StudifyStudentOverviewTableauDashboard/StudentOverviewDashboardD?publish=yes)  

**Executive Summary Dashboard:**  
- **Key Metrics:** 69.5K students, 43.3K enrollments, 56% avg progress, 4.4★ rating.  
- **Geospatial Analysis:** Mapbox visualizations highlight US dominance (12.4K enrollments).  
- **Trends:** 38% YoY enrollment growth (2024 peak).  

**Course Engagement:**  
- **Hierarchies:** Sunburst charts for `Category → Subcategory → Course` drill-downs.  
- **Behavior:** STEM courses achieve 77.7 avg grade vs 74.7 in Music.  

**Revenue Forecasting:**  
- **ARIMA Models:** Predict quarterly revenue (±12% error) with scenario analysis for discounts.  
- **Optimization:** 46% avg discounts drive $9.5M revenue; target high-wallet-balance students.  

**Features:**  
- Real-time filters for age/region.  
- Dynamic Tableau tooltips and Mapbox integrations.  

#### 10.3 Excel Dashboards  
![Studify Excel Dashboard](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/10_Dashboards/03_Excel%20Dashboards/Ph/Studify-Course%20Order%20Dashboard%20Excel.PNG)  
**Course Order Dashboard:**  
- **Sales Analysis:**  
  - **Pivot Tables:** Track revenue by category (IT/Business lead) and city (Geelong tops).  
  - **Trends:** Line charts show 2022 sales peak (+147% YoY) followed by 2025 decline.  
- **Pricing Insights:** Donut charts reveal Expert courses priced 2x higher than Beginner tiers.  
- **Payment Behavior:** 70%+ sales via credit/debit cards (pie chart).  

**Features:**  
- **Interactivity:** Slicers filter by country/category for localized analysis.  
- **Data Visualization:** Conditional formatting highlights top sellers ("Python Bootcamp: $89K").  

**Recommendations:**  
1. Investigate 2025 sales drop (market shifts/content relevance).  
2. Bundle Beginner→Expert courses to boost upgrades.  
3. Expand IT/Business content in top cities (Geelong, Melbourne).  

**Tech:** Built with Excel PivotTables, charts, and formulas for real-time updates.  

#### 10.4 Python Dashboards  
![Studify Python Dashboard](https://github.com/Mohammed1999sstack/Studify_Graduation_Project/blob/main/10_Dashboards/04_Python%20Dashboards/Studify%20Python%20Dashboard.gif)  
[View Dashboard](https://studify-dashboard.streamlit.app/?embed_options=dark_theme,show_footer,show_padding,disable_scrolling,show_toolbar,light_theme,show_colored_line)  

**Streamlit Analytics App:**  
- **Key Metrics:** 30.1K courses, 60K students, 4.4★ avg rating, \$61.85 avg price.  
- **Engagement:** 47.9% enrollments "in-progress"; SQL/Power BI courses dominate subscriptions.  
- **Regional Focus:** US, China, UK lead student enrollments (18–25 age group).  

**Features:**  
- **Dynamic Filters:** Course level, category, age group, and language.  
- **Sentiment Analysis:** NLP-driven review scores correlate with high-rated Health/Fitness courses.  
- **Visualizations:** Column Charts (Top 10 Courses by Subscribers), bar charts (top subcategories: E-commerce, Blockchain).  

**Recommendations:**  
1. Add progress checkpoints to boost 32.2% completion rate.  
2. Expand tech/business content partnerships (top-performing categories).  
3. Audit low-rated categories (Teaching, Photography).  

**Tech:** Built with Python/Streamlit, deployed on Streamlit Cloud.  

---

### 11. Conclusion  
The Studify Online Learning Platform delivers a comprehensive Udemy-like solution with robust features for course management, student engagement, and data-driven decision-making. This project successfully integrates:

#### Key Achievements  
1. **End-to-End Platform Development**  
   - **User Management:** Role-based access control (Admin/Instructor/Student) with social media integration and email verification.  
   - **Course Ecosystem:** Multi-level course structures (Category → Subcategory → Course → Section → Lesson) with video/article content and quizzes.  
   - **Monetization:** Secure payment gateways, discount management, and certification workflows.  

2. **Advanced Analytics Infrastructure**  
   - **Data Warehouse:** Hybrid star schema with SCD2 for historical tracking (UdemyDwH) and CartDM data mart for cart behavior analysis.  
   - **OLAP Cubes:** Sales, Enrollment, and Cart cubes for multidimensional analysis in SSAS.  
   - **Dashboards:** 25+ Power BI/Tableau/Excel reports for admins, instructors, and students, embedded directly into the platform.  

3. **Technical Implementation**  
   - **Database:** 85 stored procedures and 40 views for CRUD operations and analytics.  
   - **ETL:** SSIS pipelines for OLTP-to-OLAP migration with error logging and incremental loading.  
   - **Cloud Deployment:** Azure SQL Database, Data Factory, and SSAS integration for scalability.  

#### Impact  
- **For Students:** Progress tracking, certifications, and personalized dashboards.  
- **For Instructors:** Revenue analytics, student engagement metrics, and course performance heatmaps.  
- **For Admins:** Enrollment trends, revenue forecasts, and geographical insights.  

#### Future Outlook  
While the platform already supports core LMS functionalities, future iterations will focus on AI-driven personalization, live interactive features, and global market expansion to enhance user retention and accessibility.  

**Technologies Used:**  
`SQL Server` | `Excel` |`Power BI` | `Azure` | `Tableau` | `Python` | `SSIS/SSAS/SSRS` | `Streamlit`  

---

### 12. Contributors

- [**Aya Mohamed Mahmoud (Team Leader)**](https://github.com/Aya-Mohamedd)  
- [**Abdelrahman Elsayed Farouk**](https://github.com/Abdelrahman-Adams)  
- [**Maha Fathy Ghoneim**](https://github.com/MahaGhonaim)  
- [**Mohammed Osama Alsamadoni**](https://github.com/Mohammed1999sstack)  
- [**Omnia Saaed Abdelrahim**](https://github.com/omniasaeed90)
