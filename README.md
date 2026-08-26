Active Directory Change Audit
Interactive PowerShell reporting for Active Directory account, group membership, Tier-0, and directory-service changes
Overview
This PowerShell script queries Security event logs on reachable domain controllers, normalizes relevant Active Directory change events, highlights privileged-group activity, and exports a consolidated report.
It supports interactive use and parameter-driven execution. When the ImportExcel module is available, results are written to a multi-worksheet Excel workbook; otherwise, the script creates one consolidated CSV file.
Key Features
•	Discovers domain controllers or accepts an explicit target list.
•	Tests DNS resolution and TCP 135 reachability before collection.
•	Caches reachable domain controllers for the current session.
•	Validates required advanced audit policy subcategories.
•	Collects user, group, group membership, and directory object change events.
•	Identifies changes involving high-privilege built-in groups.
•	Enumerates direct and nested Tier-0 group membership.
•	Applies per-domain-controller, overall runtime, and consecutive-failure safeguards.
•	Exports a formatted Excel workbook when ImportExcel is installed.
•	Falls back to a consolidated CSV without requiring ImportExcel.
Prerequisites
•	Windows PowerShell 5.1 or a compatible PowerShell environment.
•	An elevated PowerShell session.
•	ActiveDirectory PowerShell module through RSAT or the applicable Windows Server feature.
•	Network connectivity to each target domain controller.
•	Permission to read the Security event log on each target domain controller.
•	PowerShell remoting configured when checking or changing audit policy remotely.
•	Optional: ImportExcel module for multi-worksheet Excel output.
Recommended Audit Configuration
Configure these advanced audit policy subcategories for Success auditing on domain controllers:
•	Audit Security Group Management
•	Audit User Account Management
•	Audit Directory Service Changes
Use domain Group Policy as the authoritative configuration method. The script can attempt a local audit policy change, but Group Policy may overwrite it during the next refresh.
Directory Service Changes events are generated only on domain controllers and depend on applicable system access control lists (SACLs). A report with no matching events does not prove that no change occurred.
Parameters
Parameter	Description	Default
OutputPath	Folder for the workbook or consolidated CSV.	Interactive prompt
DaysBack	Number of days of Security log history to query.	30
TopCount	Maximum number of objects shown in the combined overview.	25
DomainControllers	Optional array of domain controller names. When omitted, targets are discovered.	All discovered DCs
CheckAuditPolicy	Runs the full audit with audit policy validation and bypasses the menu.	False
OfferEnableAuditPolicy	Attempts to enable missing subcategories locally and bypasses the menu.	False
ReachabilityTimeoutSeconds	Timeout for each domain controller connectivity precheck.	5
DcQueryTimeoutSeconds	Maximum time for each domain controller query.	120
OverallRuntimeMinutes	Maximum event collection runtime before partial results are returned.	30
MaximumConsecutiveTargetFailures	Stops collection after this many consecutive target failures.	3

Usage
Interactive Mode
Save the script as AD-Change-Audit.ps1, open an elevated PowerShell session, and run:
.\AD-Change-Audit.ps1
The menu provides options to run the full audit, validate audit policy, offer a local policy enable, scan domain controllers, or exit.
Full Audit with Explicit Output Folder
.\AD-Change-Audit.ps1 -OutputPath C:\Reports -DaysBack 30
Target Specific Domain Controllers
.\AD-Change-Audit.ps1 -OutputPath C:\Reports -DomainControllers DC01.contoso.com,DC02.contoso.com
Unattended Audit with Policy Validation
.\AD-Change-Audit.ps1 -OutputPath C:\Reports -CheckAuditPolicy
Unattended Audit with Local Policy Enable Attempt
.\AD-Change-Audit.ps1 -OutputPath C:\Reports -OfferEnableAuditPolicy
Output
When ImportExcel is installed, the script produces AD-Change-Audit-Report.xlsx with these worksheets:
•	Membership Changes
•	Directory Changes
•	Combined Overview
•	Tier 0 Report
•	AD Groups Change Inventory
•	Tier 0 Members Detail
•	Audit Policy Status
•	Audit Policy After Enable
Without ImportExcel, the script produces AD-Change-Audit-Report.csv with a ReportSection column identifying each logical section.
Audit-policy-only execution also creates AD-Audit-Policy-Status.csv.
Events Collected
•	User account lifecycle and password activity: 4720, 4722, 4723, 4724, 4725, 4726, 4738, 4740, 4767.
•	Security group creation, deletion, modification, and membership activity: 4727-4735, 4737, 4754-4758, 4764.
•	Directory object modifications: 5136, 5137, 5138, 5139, 5141.
Security Considerations
•	Run with the least privilege required to read domain controller Security logs and query Active Directory.
•	Protect exported reports because they can contain privileged group membership, distinguished names, actor names, and change history.
•	Use a secured administrative workstation or equivalent privileged access device for production execution.
•	Prefer Group Policy for audit configuration instead of local auditpol.exe changes.
•	Validate Security log sizing and retention before relying on historical coverage.
Known Limitations
•	The reachability test validates DNS and TCP 135 only; it does not guarantee dynamic RPC, WinRM, authorization, or Security log access.
•	Audit policy parsing expects English subcategory names and may not parse localized auditpol.exe output correctly.
•	Directory Service Changes events require applicable SACLs and may generate high event volume.
•	Results are limited by Security log retention and the domain controllers queried.
•	Tier-0 identification is based on the built-in group-name list defined in the script; extend it for custom privileged groups.
•	Local audit policy changes can be overwritten by Group Policy.
•	CSV output cannot preserve worksheet separation or conditional formatting.
Troubleshooting
No Events Returned
•	Run the audit policy check and confirm Success auditing is enabled.
•	Increase DaysBack if expected activity is older than the current window.
•	Confirm the change was processed by a queried domain controller.
•	Review Security log retention and verify the expected event IDs are present.
•	For events 5136-5141, verify that the relevant objects and attributes have appropriate SACLs.
Domain Controller Skipped
•	Verify DNS resolution.
•	Confirm TCP 135 and required dynamic RPC ports are allowed.
•	Confirm the account has remote Security log access.
•	Provide known targets with DomainControllers when automatic discovery is unavailable.
Audit Policy Check Fails Remotely
•	Confirm PowerShell remoting and WinRM are configured.
•	Confirm the session is elevated.
•	Validate the subcategory names against the operating system language.
•	Increase DcQueryTimeoutSeconds for slower or remote sites.
Excel Workbook Is Not Created
Install the ImportExcel module or use the consolidated CSV fallback generated by the script.
Install-Module ImportExcel -Scope CurrentUser
Project Layout
. ├── AD-Change-Audit.ps1 ├── README.md └── LICENSE
Contributing
Issues and pull requests are welcome. Include the PowerShell version, Windows Server version, execution mode, relevant error output, and whether ImportExcel is installed. Do not include production identities, domain names, or Security event data.
Disclaimer
The sample script is not supported under any Microsoft standard support program or service. It is provided AS IS without warranty of any kind. Test all changes in a non-production environment and review the code before execution. The user assumes all risk arising from use or performance of the script and documentation.
Microsoft Documentation
•	Advanced Audit Policy Configuration settings
•	System Audit Policy recommendations
•	ActiveDirectory PowerShell module
•	Audit Directory Service Changes

