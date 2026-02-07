
# Quality Scenarios and Tactics Catalog

This document provides a comprehensive catalog of quality scenarios implemented in the retail application system. Each scenario follows the standard format: Source, Stimulus, Environment, Artifact, Response, and Response Measure, along with the specific architectural tactic employed.

The catalog covers seven quality attributes: **Integrability**, **Modifiability**, **Usability**, **Testability**, **Availability**, **Security**, and **Performance**. Each scenario includes specific, measurable response criteria to enable verification and validation of the system's quality attributes.

---

##  Integrability

### **Scenario 1: Partner Feed Ingestion**  
An external partner uploads a new product feed in CSV format while logged in to the partner dashboard. The system’s ingestion component (FeedIntermediary) automatically detects the file format, parses the CSV, normalizes product fields, and stores the records into the database instantly without requiring any changes to the ingestion logic.
 
- **Source:** External partner  
- **Stimulus:** Add new component (CSV feed)  
- **Environment:** Runtime (while logged into partner dashboard)  
- **Artifact:** Feed ingestion component (FeedIntermediary)  
- **Response:** The system detects the file format, parses the content, and saves normalized product records to the database.  
- **Response Measure:** Parsing and ingestion complete instantly with no code changes to existing ingestion logic or business logic.


**Tactic:** Use an Intermediary

---

### **Scenario 2: Adding New Payment Provider via Abstraction**  
A business stakeholder decides to add a new payment provider. The system integrates the new provider using a shared payment interface. The existing sale processing logic automatically incorporates the new payment type through this abstraction, requiring no changes to core business logic. The integration is completed with no changes to core logic and maximum of 1 developer-day effort.
 
- **Source:** Business stakeholder  
- **Stimulus:** Adds a new payment provider  
- **Environment:** Deployment  
- **Artifact:** Payment abstraction component  
- **Response:** System routes payments through the new provider using the shared interface  
- **Response Measure:** No changes required to core logic; maximum of 1 developer-day effort


**Tactic:** Abstract Common Services

---

##  Modifiability

### **Scenario 1: Modifying Database Logic**  
A developer needs to modify how product data is retrieved from the database. The system allows this change to be made during design time without affecting other modules. The modification is completed and tested in under one hour with no unintended side effects.

- **Source:** Developer  
- **Stimulus:** Modify how product data is retrieved from the database  
- **Environment:** Design time  
- **Artifact:** Database access logic  
- **Response:** Change is made without affecting unrelated parts of the system  
- **Response Measure:** Completed and tested in < 1 hour, no side effects  


**Tactic:** Encapsulation

---

### **Scenario 2: Adding Support for New Feed Format**  
A developer wants to add support for a new product feed format. The system is designed to support such changes by allowing runtime selection of the appropriate parser. The new format is integrated without changes to existing logic and completed in under 2 hours.
  
- **Source:** Developer  
- **Stimulus:** Add support for a new product feed format  
- **Environment:** Design time  
- **Artifact:** Feed ingestion component  
- **Response:** New format is integrated without altering existing logic  
- **Response Measure:** Integrated in < 2 hours, no impact on existing functionality 


**Tactic:** Defer Binding

---

## Usability

### **Scenario 1: Undo Upload After Completion

After uploading a CSV product feed through the partner dashboard, the partner realizes the file contained incorrect data and clicks the “Undo Upload” button. The system identifies all the records added during that upload session and removes them from the database ensuring that no unrelated data is affected and the amount of unintended data loss is zero. 

- **Source:** Partner
- **Stimulus:** Request to undo the effects of a completed upload
- **Environment:** Runtime (after upload completes)
- **Artifact:** Uploaded product records
- **Response:** System removes all records associated with the upload and updates related product statistics
- **Response Measure:** The amount of unintended data loss is zero. 


**Tactic:** Undo

---

### **Scenario 2: Provide Real-Time Progress Feedback During Uploads**
When a partner uploads a product feed file through the partner dashboard, the system displays a progress bar that updates in real time based on bytes uploaded. This provides clear and accurate feedback during the upload, enhancing the user’s satisfaction through visual feedback.
 
- **Source:** End user (partner)  
- **Stimulus:** Uploads a product feed and expects visual feedback  
- **Environment:** Runtime (during active upload in the partner dashboard)  
- **Artifact:** Progress bar  
- **Response:** System updates the progress bar in real time based on bytes uploaded  
- **Response Measure:** User satisfaction due to visual feedback  


**Tactic:** Maintain System Model
---

##  Testability

### **Scenario 1: Enforcing Business Rules**  
During automated testing, a unit tester triggers a business rule violation. The system raises a clear and descriptive error, enabling developers to identify the fault within 30 seconds of test execution.

- **Source:** Unit tester  
- **Stimulus:** Automated test cases  
- **Environment:** Testing environment (according to test schedule)  
- **Artifact:** Backend business logic  
- **Response:** System raises a clear and descriptive error  
- **Response Measure:** Faults are identified within 30 seconds of test execution  


**Tactic:** Executable Assertions
---

### **Scenario 2: Using Specialized Test Interfaces**   
During testing, a developer uses the specialized Test API to reset the database state. The system clears all previous sales data and restores product stock levels, enabling a consistent initial state for each test run within 10 seconds.

- **Source:** Developer  
- **Stimulus:** Uses the Test API  
- **Environment:** Development environment (non-production)  
- **Artifact:** Database state  
- **Response:** System clears all previous sales data and restores product stock levels  
- **Response Measure:** Reset completes within 10 seconds; test runs start with a consistent initial state


**Tactic:** Specialized Interfaces

---

## Availability

### **Scenario 1: Database Connection Failure Handling**
During peak shopping hours, the database connection times out due to high load. The system's circuit breaker detects 5 consecutive database failures and opens the circuit to prevent further attempts. After 60 seconds, the circuit enters half-open state and allows test requests to check if the database has recovered.

- **Source:** Database service  
- **Stimulus:** Connection timeout during peak traffic  
- **Environment:** Production environment under high load  
- **Artifact:** Order processing system with circuit breaker  
- **Response:** Circuit breaker opens after threshold failures, prevents cascade failures, automatically retries after timeout  
- **Response Measure:**99.9 % uptime during 7-day period; circuit opens within 5 failures; recovery check occurs after exactly 60 seconds  

**Tactic:** Circuit Breaker

---

### **Scenario 2: Transient Network Error Recovery**  
A customer attempts to place an order but encounters a transient network error during payment processing. The system automatically retries the operation with exponential backoff (1s, 2s, 4s delays).

- **Source:** Payment processing service  
- **Stimulus:** Transient network error during payment  
- **Environment:** Production environment during normal operation  
- **Artifact:** Payment processing system with retry logic  
- **Response:** System automatically retries failed operations with increasing delays  
- **Response Measure:** maximum retry delay does not exceed 30 seconds; ≥ 95 % of requests succeed within 3 retries.

**Tactic:** Retry

---

### **Scenario 3: Transaction Integrity During System Failure**  
While processing an order that involves creating an order record, updating inventory, and processing payment, the system crashes after creating the order but before completing payment. Upon restart, the system detects the incomplete transaction and automatically rolls back all changes, ensuring no partial data remains in the database.

- **Source:** System hardware/software failure  
- **Stimulus:** System crash during multi-step transaction  
- **Environment:** Production environment during order processing  
- **Artifact:** Order processing system with transaction rollback  
- **Response:** System detects incomplete transaction and rolls back all changes  
- **Response Measure:** 100% of incomplete transactions are rolled back within 30 seconds of system restart; zero data inconsistencies  

**Tactic:** Rollback

---

## Security

### **Scenario 1: Unauthorized Access Prevention via Role-Based Control**
A regular customer attempts to access the sales analytics dashboard intended only for partners. The system checks the user's role permissions and denies access, returning a 403 Forbidden response. The system logs the unauthorized access attempt for security monitoring.

- **Source:** Regular customer user  
- **Stimulus:** Attempts to access partner-only sales analytics  
- **Environment:** Production system during normal operation  
- **Artifact:** Sales analytics API with role-based access control  
- **Response:** System checks user permissions, denies access, logs security event  
- **Response Measure:** 100% of unauthorized access attempts blocked within 50ms; all violations logged with user ID and timestamp  

**Tactic:** Authorize Users

---

### **Scenario 2: SQL Injection Attack Prevention**  
A malicious actor attempts to inject SQL code through the product search field by entering `'; DROP TABLE users; --` as search criteria. The system's input validation detects the malicious pattern, sanitizes the input, and returns safe search results without executing any database commands.

- **Source:** Malicious external actor  
- **Stimulus:** SQL injection attempt via product search  
- **Environment:** Production system exposed to internet  
- **Artifact:** Product search API with input validation  
- **Response:** System validates and sanitizes input, prevents SQL execution, returns safe results  
- **Response Measure:** 100% of injection attempts blocked; malicious patterns detected within 10ms; legitimate searches complete normally  

**Tactic:** Validate Inputs


---

## Performance

### **Scenario 1: Rate Limiting Under High Load**  
During a flash sale event, rapid product search requests are sent that exceed the configured rate limit of 50 requests per minute per IP address. The system demonstrates rate limiting enforcement by blocking requests that exceed the threshold with 429 status codes and retry information.

- **Source:** Client application  
- **Stimulus:** Rapid product search requests exceeding rate limits during flash sale  
- **Environment:** Production system under peak load conditions  
- **Artifact:** Product search API with multi-tier rate limiting  
- **Response:** System enforces 50 requests/minute limit, blocks excess requests with 429 status, provides retry information  
- **Response Measure:** Rate limiting blocks excess requests with 429 status; 95% all requests completed within 500ms

**Tactic:** Increase Resource Efficiency

---

### **Scenario 2: Request Timeout Management**  
A customer initiates a complex order processing request that involves multiple database operations. If the request takes longer than 30 seconds to complete, the system automatically terminates the operation and returns a timeout error, freeing up server resources for other requests.

- **Source:** Customer user  
- **Stimulus:** Initiates complex order processing request  
- **Environment:** Production system during high database load  
- **Artifact:** Order processing API with execution time bounds  
- **Response:** System monitors execution time and terminates long-running requests  
- **Response Measure:** 100% of requests complete within 30 seconds or receive timeout error;

**Tactic:** Bound Execution Times