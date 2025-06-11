Deployment Instruction

a) Please run the following steps in the RETIRE database to validate the "Move Future AS cases back to EDIT" job logic:

1) Run the SELECT script to find a valid claim:
   - Confirm the claim meets the filter conditions (e.g., AddDeductCode = '67', RetirementTypeId not '6C', etc.)

2) Set up the test case (e.g., CaseId = 31240):
   - Run spSetStatus to set status to 300
   - Update ASSystem_BeginDate in tblResults
   - Confirm claim appears in the main query

3) Update recipients for report email:
   - Update [rtblReportEmailAddresses]
   - Insert email into [rtblReportWhiteListEmailAddresses]

4) Execute the job update script:
   - Run: Move Future AS cases back to EDIT.sql

5) Run the job manually:
   - Start from Step 3
   - Ensure the job completes successfully

b) Save the following in `SystemDB_MSDB_Build102` folder:
   - Move Future AS cases back to EDIT.sql
   - ReadMe.txt (this file)

Note: Perform all steps in DEV before release.
