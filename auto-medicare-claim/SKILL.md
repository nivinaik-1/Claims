---
name: auto-medicare-claim
description: Process eClinicalWorks (eCW) Medicare claims in Pending status - check the patient's last Annual Visit in Hub, and for qualifying Follow-Up visits add CPT 99214 + G2211, top up ICD codes to 4, and set status to Ready to Submit (Electronic). Logs every claim and produces an Excel report.
---

# Auto Medicare Claim (eCW Pending claims)

Works in the user's eCW web app through Claude in Chrome. The user signs in themselves; never type passwords.

## Scope

- Only claims with status **Pending** (Claim Lookup, filtered to Pending, Medicare payer).
- Work through every Pending claim until none remain unreviewed.

## Rules per claim

1. Open the claim and note the **service date** and **visit type**.
2. Open the patient's **Hub** and find the date of the **last Annual Visit (AV)**.
3. The claim **qualifies** only if both are true:
   - The last AV is within the **12 months before the service date** (not before today).
   - The visit type is **Follow-Up**.
4. If it does not qualify, **skip it** (leave it Pending) and log the reason:
   no AV on record / AV more than 12 months before service date / not a Follow-Up visit / patient deceased or inactive.
5. If it qualifies:
   - Add CPT **99214** and **G2211** (do not add a code that is already on the claim).
   - **ICD codes** - the claim must end with at least 4:
     - If the claim has **no ICD codes**, open the previous encounter's diagnoses (Prev Dx) and copy at least **4 chronic conditions**.
     - If it has **fewer than 4**, copy from the previous encounter only enough to make **4 in total**.
     - **Never add duplicate ICD codes.**
   - Make sure every CPT line has ICD pointers (fill blank pointers with 1-4). If eCW warns "You have not assigned the ICDs correctly", fix the pointers.
   - Set claim status to **Ready to Submit (Electronic)** and click **OK**.
   - If eCW shows any other **error/confirmation prompt, press Yes**.
   - Re-open or re-check the claim to confirm the status actually saved; if not, save again.

## Logging

Keep a CSV log with one row per claim:
`claim,patient,service_date,last_av,visit_type,result,notes`
where `result` is `UPDATED` or `SKIPPED` and `notes` gives the skip reason or anything unusual
(ICD codes added, only G2211 needed, clicked Yes on error, repeat 99214 for same patient, deceased/inactive, eligibility not verified, status had to be re-saved).

## Flag for the user's review

- Same patient getting 99214 on more than one claim.
- Deceased or inactive patients.
- Eligibility "Not Verified".
- Any claim where an error prompt was answered Yes or the status did not stick.

## Final report

When done, re-scan Pending to confirm only skipped claims remain, then build an Excel workbook
(Summary with formula counts, All Claims, Updated, Skipped, Needs Review tabs) from the log.

## Tips

- Long page operations can time out; run them in the background inside the page and poll.
- Some clicks (e.g. Prev Dx OK) don't always register - wait briefly before clicking and verify the ICD count afterwards.
- Never put patient data into URLs, commits, or anywhere outside the user's eCW and local files.
