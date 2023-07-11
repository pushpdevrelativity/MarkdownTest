Playbook for .NET Core Migration
-------------------------------------------
The objective of the migration of the review service is to upgrade dotnet framework to dotnet core 6.0.

The primary objectives of the migration evaluation service are:
* Multitenancy-supporting
* Get rid of the Kepler service.
* Review service must be compatible with open-source operating systems for deployment.

Using the strategy listed below Review Service migration.

### Step-by-step guide ###
Step-by-step guide

| **STAGE**  | **STAGE ACTIVITIES** | **WinWire** | **Relativity** | **Outcome** |
|--------------|-----------|------------|------------|--------------|
| Initial assessment | 1. High level overview of the Kepler Service which needs to be Migrated <br> 2. Understand the source code <br> 3. Understand the ADS Components needed by the Service <br> 4. Kepler specific questions <br> 5. Dependencies <br>6. Configuration <br>7. Data Model <br>8. Understand multi-tenancy requirements <br>9. Test Projects considered for migration. <br>10. Information needed for [Backstage](https://relativity.roadie.so/create?filters%5Bkind%5D=template&filters%5Buser%5D=all) to create a new service <br>11. Conduct grooming session with WinWire Team |C|R|Documented as Service Team Discovery Interview|
|User Story creation in Jira|1. Project Manager along with Architects will define the Description and Acceptance Criteria for each User Story<br>2. Criteria for an User Stories is that the deliverable should be testable.<br>3. Logically one Endpoint should be translated to one User Story<br>4. If the endpoint has many files, multiple PR's can be raised to avoid bulk PR<br>5. All Unit Test and Integration test cases need be raised as part of PR|R|C||
|Set up dev environment for service|1. Create a new Micro-service using [Backstage](https://relativity.roadie.so/create?filters%5Bkind%5D=template&filters%5Buser%5D=all) and the details gathered from the repo owner.<br>2. The legacy source code will be copied to the New Service along with the Git History for each and every folder. follow the Step instructions for copying Git History.<br>3. The entire source code of the legacy will be commented, compiled and PR raised for merging to main branch. The commented code forms the baseline for Migration.<br>4. The new project structure to follow the .NET Core Web API.<br>5. Clone the new repo created in GitHub for migration|R|C|New repository in target environment. Cloned files carry version history.|
|Set up dev environment for SDK|Clone the existing repo from Bitbucket for migration|R|C||
|Estimation and Sizing|1. The Endpoints to be migrated will be determined by Relativity based on the relevance to business and containing the least dependencies.<br>2. The Estimation is done as per the Estimation template sheet.<br>3. The complexity of the migration depends upon the dependencies.|R|C||
|Branching and Cloning|1. Development Branch needs to follow the naming Convention:REL_JiraTicketNo_Short_Description_Dev<br>2. The Branch name needs to be linked to Jira Ticket<br>3. The PR Commits to contain short description of the steps done for Migration<br>4. The Dev branch can have any number of Commits<br>5. At the end of development, review and local testing. When the Pull Request needs to be raised to Relativity, create a new branch with the same naming convention but the Dev suffix to be names as Rel(Release)|R|C||
|Source code migration|1. Creating API Controller based on .NET 6 and migrate the source code of the Manager classes.<br>2. Changing project configuration in AppSettings.json<br>3. Create environment configurations in the User Secret file<br>4. Follow the Step Instructions for the Dependency Resolutions<br>5. Change code for new .NET 6 API<br>6. Routing pattern as in Legacy application needs to be followed.<br>7. Follow the checklist provide to maintain the Code Quality and to avoid repeated reviews as mentioned in the Code review checklist<br>8. Compile and Build the solution<br>9. Use Swagger and open API for Local Testing and Development<br>10. Create and Update the Postman collection<br>11. Include Postman collection in the Solution before every PR.|R|C|Migrated code in target environment|
|Test code migration|1. Changing test project configuration<br>2. Updating dependencies<br>3. Update code changes to test in harness |R|C|Migrated test suite in target environment|
|Multi-tenancy|1. Evaluate code for multi-tenancy support.<br>2. Modify to enable multi-tenancy<br>3. Validate that the API is capable of handling Requests from multiple clients|R|C|Support stateless APIs|
|Review and Build|1. The Developer at the end of Development and running the Local Unit Testing, should send out an email to the Team Lead for Code Review.<br>2. The Team Lead needs to complete the Code Review Checklist and save the checklist in the Teams folder for the Architects approval.<br>3. Developer needs to address the internal comments given by the Reviewer<br>4. After the iteration of developer completing the review comments will send an email to Architect for final Review<br>5. Architect will verify the code review checklist and code fixes are appropriate will do a final code review before sending confirmation email for QA for testing locally.<br>6. The QA Team to verify the testing and upon successfull testing will send the confirmation mail for raising PR.<br>7. The internal process to be followed with email communication marked to Architects and the Project Manager.<br>8. After the PR is raised with the correct Commit message, a seperate slack channel will be created for discussion with Relativity for further communication related to the PR.<br>9. The Reviews given by Relativity Team will be addressed and responded in the GitHub as well as in the slack message for further action.<br>10. After the PR is approved by Relativity, an Architect from WinWire will merge the code to main branch.|R|C|1. Code reviewed<br>2. Build pipeline validation and build complete. |
|Deploy|1. After the code merge to main branch, PowerShell scripts written by Cloud engineering Team will use Github Action for deployment to Development and Regression environment.<br>2. The Development team will confirm to Relativity on successful completion of Deployment|C|R||
|Build UT|Ensure test suit completes successfully during build process|R|C|Unit tests complete on migrated code.|
|Sanity Test|1. WinWire Development team ensures that the Unit Tests/ Integration Tests Project builds and compiles and all tests will pass.<br>2. WinWire QA conducts basic tests to ensure application operational in Regression environment|R|C|Basic validation of code. Ready for functional testing (may have to be done after step below)| 
|Functional Regression Setup|1. Make changes to regression environment to use migrated components.<br>2. A recursive exercise|C|R|Environment ready for functional regression testing|
|Functional Regression|1. Conduct regression tests on impacted functionality<br>2. A recursive exercise|R|C|Complete validation of migrated code|
|Fix Identified defects|1. Fix defects identified during functional regression tests.<br>2. A recursive exercise|R|C|Close identified and agreed defects.|
|Sign off|Getting Signoff from identified stakeholders.|R|C|Migration complete|



|R = Responsible|
|---------------|
|C = Consult|
