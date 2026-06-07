## Lab Writeup: AD Administration — Guided Lab Part II

---

## Lab Setup

I spawned the target machine and connected via RDP using the following credentials:

```
Username: image
Password: Academy_student_AD!
```

The goal of this part of the lab was to join a Windows host to the INLANEFREIGHT domain and then move it into the correct Organizational Unit so that the Group Policy I configured in Part I could apply to it.

---

## Task 4: Add and Remove Computers to the Domain

### Overview

The helpdesk had just finished provisioning new computers for the Security Analyst hires. My job was to join the host `ACADEMY-IAD-W10` to the `INLANEFREIGHT.LOCAL` domain using the admin account `htb-student_adm`, and then make sure it ended up in the correct OU.

---

### Joining the Host to the Domain

#### Method 1: PowerShell (Remote Join)

From a PowerShell session running as administrator, I used the following command to join the remote computer to the domain:

```powershell
Add-Computer -ComputerName ACADEMY-IAD-W10 -LocalCredential ACADEMY-IAD-W10\image -DomainName INLANEFREIGHT.LOCAL -Credential INLANEFREIGHT\htb-student_adm -Restart
```

Here is what each part of the command does:
- `-ComputerName` — specifies the name of the machine I want to join
- `-LocalCredential` — the local account on that machine used to authenticate locally
- `-DomainName` — the domain I am joining the machine to
- `-Credential` — the domain admin account authorizing the join
- `-Restart` — restarts the machine automatically so the domain join takes effect and the machine picks up Group Policy

Alternatively, if I was running the command directly on the host itself:

```powershell
Add-Computer -DomainName INLANEFREIGHT.LOCAL -Credential INLANEFREIGHT\HTB-student_adm -Restart
```

---

#### Method 2: GUI (Control Panel)

I also practiced doing this through the Windows GUI:

1. Opened **Control Panel > System and Security > System**
2. Clicked **Change Settings** in the Computer Name section
3. In the System Properties window, clicked the **Change** button next to "To rename this computer or change its domain or workgroup, click Change"
4. Under the **Member of** section, selected **Domain** and typed `INLANEFREIGHT.LOCAL`, then clicked **OK**
5. When prompted, entered the domain admin credentials:
   - Username: `htb-student_adm`
   - Password: `Academy_student_DA!`
6. Received the **"Welcome to the INLANEFREIGHT.LOCAL domain"** popup confirming the join was successful
7. Restarted the machine to apply the changes and receive Group Policy settings from the domain

> **Note:** Before joining, I verified the computer's name matched the naming standard (`ACADEMY-IAD-W10`) to avoid having to rename it later after it was already domain-joined — which is a much more complex process.

---

### Moving the Computer to the Correct OU

When a computer joins a domain without a pre-staged AD object, it lands in the default **Computers** container rather than the OU I needed it in. I had to move it to the **Security Analysts** OU so it could receive the correct Group Policy.

#### Checking Where the Computer Landed (PowerShell)

First, I verified the current location of the computer in AD:

```powershell
Get-ADComputer -Identity "ACADEMY-IAD-W10" -Properties * | select CN,CanonicalName,IPv4Address
```

The `CanonicalName` field showed the full path in the format `Domain/OU/Name`, confirming the machine was sitting in the default Computers container.

#### Moving the Computer (GUI via ADUC)

1. Opened **Active Directory Users and Computers (ADUC)**
2. Navigated to the **Computers** container and located `ACADEMY-IAD-W10`
3. Right-clicked the computer object and selected **Move**
4. In the popup window, drilled down through the OU structure to:
   `inlanefreight.local > Corp > Employees > HQ-NYC > IT > Security Analysts`
5. Selected the **Security Analysts** OU and clicked **OK**
6. Navigated to the Security Analysts OU to confirm the computer object now appeared there

The computer was now in the correct OU, meaning the **Security Analysts Control** GPO I linked in Part I would automatically apply to it on the next Group Policy refresh.

---

## Summary

This lab covered the full lifecycle of bringing a new machine into an Active Directory environment:

- Joined a non-domain-joined Windows host (`ACADEMY-IAD-W10`) to the `INLANEFREIGHT.LOCAL` domain using both PowerShell and the GUI
- Used domain admin credentials (`htb-student_adm`) to authorize the domain join
- Verified the computer's OU placement using `Get-ADComputer` in PowerShell
- Moved the computer object from the default Computers container into the **Security Analysts** OU using ADUC
- Confirmed the computer would now receive the correct Group Policy settings configured in Part I

Together, Parts I and II of this lab covered the most common day-to-day AD administration tasks — managing users, groups, Group Policy, and computers. These same tasks are what attackers look to abuse during a penetration test, so understanding how they work from an admin's perspective is essential for both offense and defense.