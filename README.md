# AD Learning Path 48 — Seize FSMO Roles in a Failure Lab

## Goal
Practice the documented disaster-recovery decision process when `DC01` is permanently unavailable and cannot be returned to service.

## Lab boundary
Perform this exercise only in a disposable isolated lab with `DC02` healthy and a current backup. A domain controller whose roles are taken over during permanent-failure recovery must not later be reconnected in its old state.

## Prerequisites
- Two-domain-controller lab
- Healthy `DC02`
- Recent replication and system-state evidence
- Console access and a recovery record

## Steps
1. Record the current role owners and replication state.
2. Shut down and isolate `DC01` to represent a permanent failure.
3. Prove that restoring service from `DC01` is not possible within the exercise scenario.
4. Follow Microsoft's supported FSMO recovery procedure to assign the unavailable roles to `DC02`.
5. Verify all role owners through independent tools.
6. Remove the failed DC's stale AD metadata, DNS records, server object, and references only after the recovery decision is final.
7. Validate DNS, SYSVOL, Netlogon, replication, time, RID allocation, and directory writes on `DC02`.
8. Rebuild any replacement domain controller from a clean operating-system installation.

## Validation
```powershell
Get-ADForest | Select-Object SchemaMaster,DomainNamingMaster
Get-ADDomain | Select-Object PDCEmulator,RIDMaster,InfrastructureMaster
netdom.exe query fsmo
Get-ADDomainController -Filter *
dcdiag.exe /s:DC02 /v
repadmin.exe /replsummary
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.corp.lab
w32tm.exe /query /status
```

## Evidence
Store the declared failure condition, role ownership before/after, recovery authorization, metadata/DNS cleanup checklist, health checks, rebuilt-DC plan, errors/remediation, and final pass/fail status under `evidence/`.

## Troubleshooting
- Stale failed-DC references remain: review Sites and Services, Domain Controllers OU, DNS, DFSR, and replication metadata.
- Time becomes unreliable: configure and validate the new PDC Emulator time hierarchy.
- Replication reports the removed DC: complete supported metadata cleanup and allow convergence.

## Security notes
This is a disaster-recovery action, not a routine administrative shortcut. Preserve logs, require approval, and never reconnect the original failed DC after the recovery decision.

## Cleanup
Keep `DC01` isolated. Rebuild it cleanly if it will return to the lab.

## References
- Microsoft Learn: Transfer or seize operations master roles
- Microsoft Learn: Clean up AD DS server metadata

## Next activity
`AD-Learning-Path-49-Deploy-a-Read-Only-Domain-Controller`
