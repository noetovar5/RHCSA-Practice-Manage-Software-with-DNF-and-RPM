# RHCSA-Practice-Manage-Software-with-DNF-and-RPM
RHCSA Practice — Manage Software with DNF and RPM
Today’s focused objective: Inspect enabled repositories, install a package with DNF, examine it with RPM, verify its files, review transaction history, and safely remove only software added by this lab.
Estimated time: 45 minutes
Lab system: RHEL 9 or RHEL 10
Privileges required: Root or sudo
Primary lab package: tree
This lesson builds on our process and service-administration work. Red Hat’s current EX200 objectives include configuring access to RPM repositories and installing or removing RPM packages. The published exam is based on RHEL 10 and remains hands-on and performance-based. EX200
Software-change warning
Installing or removing packages changes the operating system. Removing the wrong package—or accepting an unexpected dependency-removal list—can disrupt networking, authentication, services, boot behavior, or system availability.
This lab uses tree, a small command-line utility that does not install or control a system service.
Before approving any installation or removal:
- Read the transaction summary.
- Verify the exact package name.
- Review dependencies.
- Do not remove packages such as systemd, dnf, rpm, kernel, NetworkManager, openssh-server, sudo, or selinux-policy.
- Use only a disposable RHEL lab VM.
1. DNF and RPM in plain language — 7 minutes
RHEL software normally arrives as RPM packages.
RPM is both:
- The package format, such as tree-2.1.0-3.el10.x86_64.rpm
- A command for querying and verifying installed packages
DNF is the higher-level package manager. It works with repositories and handles dependencies.
A practical distinction is:
Tool	Best use
dnf	Search repositories, resolve dependencies, install, update, remove, and review transactions
rpm	Query installed packages, list package files, identify file ownership, and verify installed files


Useful commands
Command	Purpose
dnf repolist	List enabled repositories
dnf info PACKAGE	Display package information
dnf search TERM	Search package names and descriptions
dnf install PACKAGE	Install a package and dependencies
dnf remove PACKAGE	Remove a package
dnf history	Review package transactions
rpm -q PACKAGE	Determine whether a package is installed
rpm -qi PACKAGE	Show installed-package information
rpm -ql PACKAGE	List files installed by a package
rpm -qf PATH	Identify which package owns a file
rpm -V PACKAGE	Verify installed files against RPM metadata


A good exam habit is:
Inspect → Execute → Verify

Never assume a successful-looking command produced the required final state.
2. Preflight checks — 5 minutes
Enter a root shell:
sudo -i

Confirm your operating system and identity:
whoami
cat /etc/redhat-release

Expected username:
root

Create a workspace:
mkdir -p /tmp/rhcsa-software-lab
cd /tmp/rhcsa-software-lab
pwd

Expected:
/tmp/rhcsa-software-lab

Confirm that DNF and RPM are available:
command -v dnf
command -v rpm

Typical results:
/usr/bin/dnf
/usr/bin/rpm

Record whether tree was installed before the lab:
if rpm -q tree > /dev/null 2>&1; then
    touch tree-was-preinstalled
    echo "NOTICE: tree was already installed before this lab"
else
    echo "PASS: tree is not currently installed"
fi

This marker protects a preexisting package from accidental cleanup.
3. Inspect repositories — 6 minutes
Display enabled repositories:
dnf repolist

Expected output varies according to your RHEL version and subscription. On a registered RHEL system, repository names commonly reference BaseOS and AppStream.
Display enabled and disabled repositories:
dnf repolist --all

Show detailed repository information:
dnf repoinfo

Important fields include:
- Repository ID
- Repository name
- Enabled status
- Repository URL or source
- Package count
- Cache expiration
Check Red Hat registration
Run:
subscription-manager identity
subscription-manager status

Possible healthy output includes a system identity and registration information.
Some Developer Subscription systems may display an overall status that is not worded simply as Current, even while repositories work. The practical verification is whether enabled repositories can return metadata and packages.
Refresh repository metadata:
dnf makecache

Expected final message:
Metadata cache created.

If no repositories are enabled
Do not continue to package installation until repository access is fixed.
Diagnose with:
dnf repolist
subscription-manager identity
subscription-manager repos --list-enabled

Do not randomly enable repositories or place unknown .repo files under /etc/yum.repos.d.
4. Search for the package — 4 minutes
Search by word:
dnf search tree

This searches package names and descriptions, so it may return multiple results.
Display exact package information:
dnf info tree

Look for fields such as:
Name
Version
Release
Architecture
Size
Repository
Summary

Ask which package supplies /usr/bin/tree:
dnf provides '*/tree'

The result should identify the tree package.
Why dnf provides matters: during troubleshooting, I may know a missing command or file but not the package that supplies it.
Check the current installed state:
rpm -q tree

Possible results are:
package tree is not installed

or an installed package name such as:
tree-VERSION-RELEASE.x86_64

5. Preview the transaction — 4 minutes
Software-change warning
The following command calculates the installation transaction but deliberately answers “no.” It should not install anything.
dnf install --assumeno tree

Review the transaction summary carefully.
Look for:
- Package name
- Architecture
- Version and release
- Repository
- Download size
- Installed size
- Additional dependencies
Expected ending:
Operation aborted.

or equivalent wording indicating that the transaction was not performed.
Confirm that nothing changed if the package was originally absent:
if test ! -e tree-was-preinstalled; then
    rpm -q tree
fi

Expected for an initially absent package:
package tree is not installed

This preview-before-change habit is especially important before package removal.
6. Install or reinstall tree — 5 minutes
Software-change warning
The next transaction changes installed software. It affects only the non-service tree utility.
If tree was not installed before the lab, install it:
if test ! -e tree-was-preinstalled; then
    dnf install -y tree
else
    dnf reinstall -y tree
fi

Why the conditional matters:
- A previously absent package is installed.
- A preexisting package is reinstalled rather than removed.
- Cleanup can later preserve the machine’s original software state.
Expected output ends with:
Complete!

Immediately verify:
rpm -q tree
command -v tree
tree --version

Expected command location:
/usr/bin/tree

Use it in the lab workspace:
mkdir -p sample/{config,logs,reports}
touch sample/config/application.conf
touch sample/logs/application.log
touch sample/reports/status.txt
tree sample

Expected structure:
sample
├── config
│   └── application.conf
├── logs
│   └── application.log
└── reports
    └── status.txt

7. Examine the installed RPM — 6 minutes
Display detailed package information:
rpm -qi tree

Useful fields include:
- Name
- Version
- Release
- Architecture
- Install date
- License
- Signature
- Description
List all paths installed by the package:
rpm -ql tree

Expected entries include:
/usr/bin/tree

Documentation and manual-page locations may also appear.
Display documentation paths only:
rpm -qd tree

Display configuration files:
rpm -qc tree

tree may have no package-managed configuration files. No output is therefore a valid result.
Determine which package owns the executable:
rpm -qf /usr/bin/tree

Expected: the installed tree package.
Determine which package owns a common system file:
rpm -qf /usr/bin/bash

Expected: a bash package name.
This is useful when:
- A file is damaged
- A command behaves unexpectedly
- I need to reinstall the package that owns a particular file
8. Verify package integrity — 3 minutes
Verify tree against the RPM database:
rpm -V tree

Expected:
No output

No output means RPM found no differences significant enough to report.
rpm -V compares installed files with package metadata, including attributes such as:
- Size
- Mode
- Checksum
- Owner
- Group
- Modification time
- Symbolic-link target
It does not prove that the application’s business logic is correct. It verifies installed files and metadata.
Check the executable directly:
test -x /usr/bin/tree \
  && echo "PASS: tree executable exists" \
  || echo "FAIL: tree executable missing"

9. Review DNF history — 3 minutes
Display recent transactions:
dnf history list

The newest transaction normally appears near the top.
Display the latest transaction in detail:
dnf history info last

Look for:
- Transaction ID
- Begin and end time
- Command line
- User
- Packages altered
- Final transaction state
Filter history for the package:
dnf history list tree

DNF history is valuable when investigating:
- Who installed or removed a package
- When an update occurred
- What changed immediately before a problem began
Do not automatically use dnf history undo on a production server. Reversing an old transaction can affect dependencies or downgrade packages.
Independent challenge — 8 minutes
Complete this without copying the guided installation commands.
Select a package that is not already installed
Run this preparation block:
challenge_pkg=""

for candidate in lsof jq; do
    if ! rpm -q "$candidate" > /dev/null 2>&1; then
        challenge_pkg="$candidate"
        break
    fi
done

printf '%s\n' "$challenge_pkg" > challenge-package.txt
echo "Challenge package: $challenge_pkg"

If the result is blank, both packages are already installed. In that unusual case, use bash-completion if it is absent:
if test -z "$challenge_pkg" &&
   ! rpm -q bash-completion > /dev/null 2>&1; then
    challenge_pkg="bash-completion"
    printf '%s\n' "$challenge_pkg" > challenge-package.txt
fi

echo "Challenge package: $challenge_pkg"

Assignment
Using the selected package:
1. Prove that it is not currently installed.
2. Display its repository information.
3. Preview its installation without performing it.
4. Install it using DNF.
5. Verify it using RPM.
6. Display detailed package information.
7. List the files it installed.
8. Select one installed file and determine which package owns it.
9. Verify package integrity.
10. Find the transaction in DNF history.
11. Remove only the challenge package.
12. Prove that it is no longer installed.
13. Do not remove tree as part of the challenge.
14. Review the removal summary before approving it.
Independent verification
Restore the package variable:
challenge_pkg=$(cat challenge-package.txt)

After installation, this test should pass:
rpm -q "$challenge_pkg" > /dev/null 2>&1 \
  && echo "PASS: challenge package installed" \
  || echo "FAIL: challenge package not installed"

After removal, this test should pass:
if rpm -q "$challenge_pkg" > /dev/null 2>&1; then
    echo "FAIL: challenge package remains installed"
else
    echo "PASS: challenge package removed"
fi

Confirm that tree remains available:
rpm -q tree \
  && echo "PASS: guided package remains available"

Review the challenge transactions:
dnf history list "$challenge_pkg"

Knowledge check
Answer these without reviewing the lesson:
1. What is the difference between DNF and RPM?
2. Which command lists enabled repositories?
3. Which command refreshes repository metadata?
4. How do I search repository descriptions for a package?
5. How do I find which package supplies a missing path?
6. What does rpm -q tree determine?
7. What does rpm -qi tree display?
8. What does rpm -ql tree display?
9. What does rpm -qf /usr/bin/tree determine?
10. What does no output from rpm -V tree normally mean?
11. Why should I preview a removal transaction?
12. What does --assumeno accomplish?
13. Why is blindly using -y dangerous?
14. Which command displays recent DNF transactions?
15. How do I inspect the newest transaction in detail?
16. Why might removing one package remove additional dependencies?
17. Why should a package that existed before a lab be preserved during cleanup?
18. Does a successful installation alone prove that the required command works?
19. What three-stage habit should I use for exam tasks?
20. What should I verify after every package installation or removal?
Mastery standard
I would consider this objective mastered when I can independently:
- Identify enabled repositories
- Search for a package
- Identify which package provides a file
- Preview a transaction
- Install and remove a package
- Query package information with RPM
- List installed files
- Determine file ownership
- Verify package integrity
- Review DNF transaction history
- Preserve packages that existed before the lab
Safe cleanup
Remove the sample files:
Filesystem cleanup warning
Confirm the path is exactly /tmp/rhcsa-software-lab:
cd /tmp
rm -rf /tmp/rhcsa-software-lab

Verify:
test ! -e /tmp/rhcsa-software-lab \
  && echo "PASS: temporary software-lab files removed"

Restore the original tree state
If tree was installed before the lab, keep it.
If it was absent before the lab, remove only that package.
Because the marker was inside the temporary directory, determine this before deleting the directory if you want automatic cleanup. A safer complete sequence is:
cd /tmp/rhcsa-software-lab

if test -e tree-was-preinstalled; then
    echo "KEEP: tree existed before the lab"
else
    echo "REMOVE: tree was added by this lab"
    dnf remove --assumeno tree
fi

Review the removal preview. It should target tree and no important system packages.
If the preview is correct:
if test ! -e tree-was-preinstalled; then
    dnf remove -y tree
fi

Verify the restored state:
if test -e tree-was-preinstalled; then
    rpm -q tree \
      && echo "PASS: preexisting tree package preserved"
else
    if rpm -q tree > /dev/null 2>&1; then
        echo "CHECK: lab-added tree package remains"
    else
        echo "PASS: original absent state restored"
    fi
fi

Now remove the workspace:
cd /tmp
rm -rf /tmp/rhcsa-software-lab

Leave the privileged shell:
exit
