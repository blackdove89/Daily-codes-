--Running find claim script to generate a claim to update
SELECT    
      DISTINCT d.CaseId, d.Claim, Status, k.Email Specialist, l.Email Reviewer
   FROM        
      dbo.vwCases d
         JOIN vwCaseServiceSummary e ON d.caseid = e.caseid
            JOIN vwCodeList m ON e.RetirementTypeId = m.CodeId
         JOIN dbo.tblResults A ON a.CaseId = d.CaseId
         JOIN dbo.tblAdjustments b ON d.CaseId = b.CaseId
            JOIN dbo.vwCodeList c ON b.AddDeductCodeId = c.CodeId      
         LEFT JOIN tblAnnuitySupplement h on d.Caseid = h.CaseId
         JOIN rvwUserList k on d.specialist = k.login
         JOIN rvwUserList l on d.reviewer = l.login
   WHERE    
    --d.Status = '300' AND
      c.CodeType = 'AddDeductCodes' AND
      c.CodeAbbrev = '67' AND
      b.RunType = (SELECT o.RunType FROM tblRunResults o WHERE b.CaseId = o.CaseId AND btriggered = 1) AND -- incluse AS fromn the trigger run
      e.CaseType IN(2, 3) AND
     RetirementTypeId <> (select [dbo].[fGetCodeId]('C','SepCodes')) AND -- exclude old 6C
      EXISTS(SELECT 1 FROM tblRunResults f where d.caseid = f.caseid and (CalcRetirementType <> 'C' OR CalcRetirementType IS NULL) AND bTriggered = 1) AND  -- exclude new 6C
      a.ASSystem_BeginDate > a.AnnuityStartDate
	--AND
     --ISNULL(ASUser_BeginDate, a.ASSystem_BeginDate) >= @CheckDate  

--Step 1..Set Status
USE [RETIRE]
GO

DECLARE	@return_value int,
		@Msg varchar(2000)

EXEC	@return_value = [dbo].[spSetStatus]
		@CaseId = 31240,
		@Status = 300,
		@Login = N'<system>',
		@Msg = @Msg OUTPUT

SELECT	@Msg as N'@Msg'

SELECT	'Return Value' = @return_value

GO

--Step 2 update date.

SELECT TOP (1000) [CaseId],
[ASSystem_BeginDate]
      
  FROM [RETIRE].[dbo].[tblResults]
  --WHERE ASSystem_BeginDate >='2025-07-01 00:00:00.000'
  --AND
    WHERE CaseId = 31240


	UPDATE tblResults
	SET ASSystem_BeginDate = '2025-09-12'
	WHERE CaseId = 31240

	--confirm claim is available in table by running the entire find claim script

--Step 3 running all to check claim
	DECLARE @CutOffDate DATETIME
DECLARE @CheckDate DATETIME


SELECT
   @CutOffDate = MIN(CutOffDate)
FROM
   rtblCutOff WHERE CutOffDate > GetDate() - 1  

SET @CutOffDate = DATEADD(m, 1, @CutOffDate)

SET @CheckDate = CAST(MONTH(@CutOffDate) AS VARCHAR(2)) + '/01/' + CAST(YEAR(@CutOffDate) AS VARCHAR(4))


SELECT    
      DISTINCT d.CaseId, d.Claim, Status, k.Email Specialist, l.Email Reviewer
   FROM        
      dbo.vwCases d
         JOIN vwCaseServiceSummary e ON d.caseid = e.caseid
            JOIN vwCodeList m ON e.RetirementTypeId = m.CodeId
         JOIN dbo.tblResults A ON a.CaseId = d.CaseId
         JOIN dbo.tblAdjustments b ON d.CaseId = b.CaseId
            JOIN dbo.vwCodeList c ON b.AddDeductCodeId = c.CodeId      
         LEFT JOIN tblAnnuitySupplement h on d.Caseid = h.CaseId
         JOIN rvwUserList k on d.specialist = k.login
         JOIN rvwUserList l on d.reviewer = l.login
   WHERE    
    d.Status = '300' AND
      c.CodeType = 'AddDeductCodes' AND
      c.CodeAbbrev = '67' AND
      b.RunType = (SELECT o.RunType FROM tblRunResults o WHERE b.CaseId = o.CaseId AND btriggered = 1) AND -- incluse AS fromn the trigger run
      e.CaseType IN(2, 3) AND
     RetirementTypeId <> (select [dbo].[fGetCodeId]('C','SepCodes')) AND -- exclude old 6C
      EXISTS(SELECT 1 FROM tblRunResults f where d.caseid = f.caseid and (CalcRetirementType <> 'C' OR CalcRetirementType IS NULL) AND bTriggered = 1) AND  -- exclude new 6C
      a.ASSystem_BeginDate > a.AnnuityStartDate
	AND
     ISNULL(ASUser_BeginDate, a.ASSystem_BeginDate) >= @CheckDate  

--Step 4 update recipients email
USE [RETIRE]
GO

UPDATE [dbo].[rtblReportEmailAddresses]
   SET 
      [Recipients] = 'Adekunle.keshiro@opm.gov; jenn-rong.chen@opm.gov;malathi.thadkamalla@opm.gov;'
      ,[Copy] = 'Adekunle.keshiro@opm.gov; jenn-rong.chen@opm.gov;malathi.thadkamalla@opm.gov;'
      ,[BlindCopy] = 'Adekunle.keshiro@opm.gov; jenn-rong.chen@opm.gov;malathi.thadkamalla@opm.gov;'
      ,[ErrorRecipients] = 'Adekunle.keshiro@opm.gov; jenn-rong.chen@opm.gov;malathi.thadkamalla@opm.gov;'
      ,[AdminRecipients] = 'Adekunle.keshiro@opm.gov; jenn-rong.chen@opm.gov;malathi.thadkamalla@opm.gov;'
 WHERE [ReportName] = 'Move Future AS cases back to EDIT' 
GO

--Step 5 Add email to white list
USE [RETIRE]
GO

INSERT INTO [dbo].[rtblReportWhiteListEmailAddresses]
           ([EmailAddress])
     VALUES
           ('Adekunle.keshiro@opm.gov')
GO

--Step 6 Run Move Future AS cases back to EDIT script to update job 
"C:\Users\DCTCRCTECAYK\Desktop\Hardcode Rewrite\Move Future AS cases back to EDIT.sql"

--Step 7 Run job at step 3 only
Change step 3 of job settings to complete job reporting success
Run job starting at step3


