# Jenkins RBAC — 

## 1. Start with the Problem Statement



> **Problem Statement:**
> Our company has one Jenkins server used by Developers, Testers and DevOps team.
>
> We cannot give everyone Administrator access because users may accidentally or intentionally modify jobs, credentials, or Jenkins configuration.
>
> We need to control **who can see, build, configure, or administer Jenkins jobs**.
>
> Therefore, we implement **Role-Based Access Control (RBAC)**.

### Real-time example

Imagine this Jenkins server:

```text
                    Jenkins Server
                         |
          +--------------+--------------+
          |              |              |
      Developer        Tester         DevOps
          |              |              |
       Build Jobs     Test Jobs      Full Access
       View Jobs      View Jobs      Admin
```

The basic idea is:

**User → Role → Permissions → Jenkins Resources**

---

# 2. What is RBAC?

**RBAC = Role-Based Access Control**

Instead of giving permissions individually to every user, we create **roles**.

Example:

```text
Developer Role
      |
      +-- View Jenkins
      +-- View Jobs
      +-- Build Jobs
      +-- View Console Output
```

Then:

```text
Developer
    ↓
Developer Role
    ↓
Developer Permissions
```

This is easier to manage.

---

# 3. Why do we need RBAC in Jenkins?

Without RBAC:

```text
Developer
    ↓
Administrator
    ↓
Can modify Jenkins
Can delete jobs
Can modify credentials
Can install plugins
Can change security
```

This is risky.

With RBAC:

```text
Developer
    ↓
Limited Permissions
    ↓
Can build application
Can see build logs
Cannot modify Jenkins security
Cannot manage credentials
Cannot delete production jobs
```

This follows the important security principle:

### Least Privilege

> Give users only the permissions they need to perform their job.

---

# 4. Real-Time Company Roles

For your class, I recommend using **3 roles**.

| Role         | Typical Access                              |
| ------------ | ------------------------------------------- |
| Developer    | Build and view development jobs             |
| Tester       | Build/view test jobs and see console output |
| DevOps/Admin | Full Jenkins administration                 |

You can explain that actual permissions vary from company to company.

---

# 5. What Does a Developer Usually Need?

Suppose we have:

```text
Jenkins
│
├── DEV
│   ├── EmployeeApp-DEV
│   └── PaymentApp-DEV
│
├── QA
│   ├── EmployeeApp-QA
│   └── PaymentApp-QA
│
└── PROD
    ├── EmployeeApp-PROD
    └── PaymentApp-PROD
```

A developer generally needs:

### Developer can:

* Login to Jenkins
* View permitted jobs
* Build DEV jobs
* Stop their builds
* View console output
* View build history

### Developer normally should NOT:

* Manage Jenkins
* Manage users
* Manage credentials
* Install plugins
* Change Jenkins security
* Modify production jobs
* Delete important jobs

For a classroom lab, you can make this even stricter:

```text
Developer
   ↓
DEV jobs only
   ↓
View + Build + Console Output
```

---

# 6. What Does a Tester Usually Need?

Tester may need:

```text
Tester
   ↓
QA Jobs
```

Typical permissions:

* Login
* View QA jobs
* Build QA jobs
* View console output
* View build history
* Stop builds if required

Tester normally should NOT:

* Change Jenkins global configuration
* Manage credentials
* Create users
* Install plugins
* Modify security
* Access production deployment configuration

For your beginner lab:

```text
Tester
   ↓
QA jobs only
   ↓
View + Build + Console Output
```

---

# 7. What Does DevOps Need?

DevOps has broader access.

```text
DevOps
   ↓
Jenkins Administration
```

Typical permissions:

* Create jobs
* Configure jobs
* Delete jobs
* Manage plugins
* Manage credentials
* Manage users
* Configure Jenkins
* Manage nodes/agents
* Configure security
* Troubleshoot pipelines

In many companies, **Jenkins Administrator access is restricted to a small number of people**.

---

# 8. Simple Permission Matrix

This is a good slide for your students:

| Permission         | Developer |    Tester | DevOps |
| ------------------ | --------: | --------: | -----: |
| Login              |         ✅ |         ✅ |      ✅ |
| View Jenkins       |         ✅ |         ✅ |      ✅ |
| View DEV Jobs      |         ✅ | ❌/Limited |      ✅ |
| Build DEV Jobs     |         ✅ | ❌/Limited |      ✅ |
| View QA Jobs       | ❌/Limited |         ✅ |      ✅ |
| Build QA Jobs      | ❌/Limited |         ✅ |      ✅ |
| Configure Jobs     |         ❌ |         ❌ |      ✅ |
| Delete Jobs        |         ❌ |         ❌ |      ✅ |
| Manage Credentials |         ❌ |         ❌ |      ✅ |
| Manage Plugins     |         ❌ |         ❌ |      ✅ |
| Manage Users       |         ❌ |         ❌ |      ✅ |
| Manage Jenkins     |         ❌ |         ❌ |      ✅ |

**Important:** In real companies, exact permissions depend on security policy and the Jenkins authorization model.

---

# 9. Hands-On Lab Architecture

For the classroom, create:

```text
Jenkins Server
│
├── Users
│   ├── developer1
│   ├── tester1
│   └── devops1
│
├── Roles
│   ├── Developer
│   ├── Tester
│   └── DevOps
│
└── Jobs
    ├── DEV-EmployeeApp
    ├── QA-EmployeeApp
    └── PROD-EmployeeApp
```

The goal:

```text
developer1
     ↓
Developer Role
     ↓
DEV-EmployeeApp
     ↓
Can Build


tester1
     ↓
Tester Role
     ↓
QA-EmployeeApp
     ↓
Can Build


devops1
     ↓
DevOps Role
     ↓
All Jenkins Resources
```

---

# 10. Step 1 — Install Role-Based Authorization Plugin

Login to Jenkins as Administrator.

Go to:

```text
Dashboard
   ↓
Manage Jenkins
   ↓
Plugins
   ↓
Available Plugins
```

Search:

```text
Role-based Authorization Strategy
```

Install the plugin.

Depending on your Jenkins version/plugin UI, you may be asked to restart Jenkins.

---

# 11. Step 2 — Create Users

Go to:

```text
Manage Jenkins
      ↓
Users
      ↓
Create User
```

Create:

```text
Username: developer1
Password: ********
Full Name: Developer User
```

Create another:

```text
Username: tester1
Password: ********
Full Name: Tester User
```

And:

```text
Username: devops1
Password: ********
Full Name: DevOps User
```

For the lab, keep your original admin account separately so you don't lock yourself out.

---

# 12. Step 3 — Create Test Jobs

Create three simple Freestyle jobs.

### Job 1

```text
DEV-EmployeeApp
```

Build step:

```bash
echo "Building Employee Application"
echo "Environment: DEV"
```

### Job 2

```text
QA-EmployeeApp
```

Build step:

```bash
echo "Running Employee Application QA Build"
echo "Environment: QA"
```

### Job 3

```text
PROD-EmployeeApp
```

Build step:

```bash
echo "Deploying Employee Application"
echo "Environment: PROD"
```

Now students have something visible to test RBAC against.

---

# 13. Step 4 — Enable Role-Based Authorization

As Administrator:

```text
Manage Jenkins
      ↓
Security
```

Find:

```text
Authorization
```

Select:

```text
Role-Based Strategy
```

Then:

```text
Save
```

Depending on the Jenkins version, the exact location/name may differ slightly.

---

# 14. Step 5 — Open Manage Roles

Go to:

```text
Manage Jenkins
      ↓
Manage and Assign Roles
      ↓
Manage Roles
```

You will see sections for roles and permissions.

Now create:

```text
Developer
Tester
DevOps
```

---

# 15. Step 6 — Create Developer Role

Create:

```text
Role Name:

Developer
```

Give the Developer role basic permissions such as:

```text
Overall
 └── Read              ✅

Job
 ├── Read              ✅
 ├── Build             ✅
 ├── Cancel            ✅
 └── Configure         ❌
```

Depending on your Jenkins/plugin version, permissions may be displayed differently.

The important teaching concept is:

```text
Developer

Overall/Read       → Login and access Jenkins
Job/Read           → See jobs
Job/Build          → Start builds
Job/Cancel         → Stop builds
Job/Configure      → ❌
```

---

# 16. Step 7 — Create Tester Role

Create:

```text
Tester
```

Give:

```text
Overall
 └── Read              ✅

Job
 ├── Read              ✅
 ├── Build             ✅
 ├── Cancel            ✅
 └── Configure         ❌
```

For your classroom example:

```text
Tester
    ↓
QA jobs
    ↓
Build + View
```

---

# 17. Step 8 — Create DevOps Role

Create:

```text
DevOps
```

For a lab, give DevOps the required administrative permissions.

For example:

```text
Overall
 ├── Read
 └── Administer

Job
 ├── Read
 ├── Build
 ├── Configure
 ├── Create
 ├── Delete
 └── Cancel
```

Plus the appropriate permissions required to manage:

```text
Credentials
Nodes
Plugins
Users
Jenkins configuration
```

**Teaching point:** Don't simply say "DevOps always gets everything." In production, even DevOps access should follow least privilege.

---

# 18. Step 9 — Assign Roles to Users

Go to:

```text
Manage Jenkins
      ↓
Manage and Assign Roles
      ↓
Assign Roles
```

You should see something similar to:

```text
Users             Developer     Tester     DevOps
----------------------------------------------------
developer1           ✓
tester1                            ✓
devops1                                         ✓
```

Assign:

```text
developer1 → Developer

tester1 → Tester

devops1 → DevOps
```

Save.

---

# 19. Step 10 — Test Developer Access

Logout from Administrator.

Login as:

```text
developer1
```

Ask students:

### Question:

Can developer1 see Jenkins?

Expected:

```text
YES
```

Can developer1 build a permitted DEV job?

```text
YES
```

Can developer1 view console output?

```text
YES
```

Can developer1 configure the Jenkins job?

```text
NO
```

Can developer1 manage Jenkins?

```text
NO
```

Can developer1 manage credentials?

```text
NO
```

This is where students understand RBAC practically.

---

# 20. Step 11 — Test Tester Access

Logout.

Login:

```text
tester1
```

Test:

```text
QA-EmployeeApp
```

Tester should be able to:

```text
View job       ✅
Build job      ✅
View console   ✅
```

But:

```text
Configure job       ❌
Manage Jenkins      ❌
Manage Credentials  ❌
Manage Plugins      ❌
```

---

# 21. Step 12 — Test DevOps Access

Login:

```text
devops1
```

DevOps should have access to the administrative functions assigned to the DevOps role.

For your classroom:

```text
Developer
    ↓
DEV

Tester
    ↓
QA

DevOps
    ↓
DEV + QA + PROD + Jenkins Administration
```

---

# 22. Very Important: Role vs Job Access

This is where beginners often get confused.

There are **two concepts**:

### Role

Answers:

> **What permissions does this person have?**

Example:

```text
Developer
   ↓
Read + Build + Console
```

### Resource/Job pattern

Answers:

> **Which jobs can this person access?**

Example:

```text
DEV-.*
```

means the role can match DEV jobs.

So you can explain:

```text
Role
  +
Job Pattern
  +
Permissions
  =
RBAC Access
```

---

# 23. Real Company Example

Suppose a company has:

```text
DEV-EmployeeApp
DEV-PaymentApp

QA-EmployeeApp
QA-PaymentApp

PROD-EmployeeApp
PROD-PaymentApp
```

### Developer

```text
Pattern:

DEV-.*
```

Permissions:

```text
Read
Build
Console Output
Cancel
```

Result:

```text
Developer
    |
    +---- DEV-EmployeeApp       ✅
    +---- DEV-PaymentApp        ✅
    |
    +---- QA-EmployeeApp        ❌
    +---- QA-PaymentApp         ❌
    |
    +---- PROD-EmployeeApp      ❌
    +---- PROD-PaymentApp       ❌
```

---

# 24. Tester

Pattern:

```text
QA-.*
```

Result:

```text
Tester
    |
    +---- DEV-EmployeeApp       ❌
    +---- DEV-PaymentApp        ❌
    |
    +---- QA-EmployeeApp        ✅
    +---- QA-PaymentApp         ✅
    |
    +---- PROD-EmployeeApp      ❌
    +---- PROD-PaymentApp       ❌
```

---

# 25. DevOps

DevOps may have access to:

```text
DEV-.*
QA-.*
PROD-.*
```

plus administrative permissions.

```text
DevOps
   |
   +---- DEV      ✅
   +---- QA       ✅
   +---- PROD     ✅
   |
   +---- Jenkins Administration
   +---- Credentials
   +---- Nodes
   +---- Plugins
```

---

# 26. Real-Time Deployment Flow

This is an excellent example to explain **why RBAC is required**.

Imagine:

```text
Developer
     |
     | Push code
     ↓
GitHub
     |
     ↓
Jenkins DEV Pipeline
     |
     ↓
Build
     |
     ↓
Test
```

Developer should be able to trigger the DEV pipeline.

But:

```text
Developer
     X
     |
     X
PROD Deployment
```

Production deployment may require:

```text
Tester / QA
       ↓
Testing
       ↓
Approval
       ↓
DevOps
       ↓
Production Deployment
```

This is a much more realistic explanation than simply saying "Developer doesn't have admin access."

---

# 27. Important Security Example

Ask students:

> What could happen if every developer has Jenkins Administrator access?

They might:

```text
Delete jobs
Modify pipeline
Access credentials
Change Jenkins configuration
Install plugins
Change security settings
Trigger production deployments
```

Therefore:

```text
No RBAC
   ↓
Everyone gets excessive access
   ↓
Security Risk
```

With RBAC:

```text
Developer → DEV
Tester    → QA
DevOps    → Administration
```

---

# 28. Simple Hinglish Explanation for Students

You can say this in class:

> **RBAC ka simple meaning hai — har user ko uske role ke according permission dena.**
>
> Developer ko development application build karni hai, toh usko Jenkins mein build permission denge.
>
> Tester ko QA application test karni hai, toh usko QA jobs ka access denge.
>
> Developer ko Jenkins configuration ya credentials manage karne ki zarurat nahi hai, toh hum usko woh permission nahi denge.
>
> DevOps team ko Jenkins manage karna hai, isliye unko additional administrative permissions denge.
>
> Isko hum **Least Privilege** bolte hain — jitni permission required hai, utni hi permission deni hai.

---

# 29. Final Classroom Exercise

Give students this problem statement:

### Jira Task: Implement Jenkins RBAC

**Title:** Configure Role-Based Access Control in Jenkins

**Problem:**

The organization has three teams:

```text
Development
Testing
DevOps
```

The Jenkins administrator wants to restrict access according to team responsibilities.

### Requirements

**Developer**

* Login to Jenkins
* View DEV jobs
* Build DEV jobs
* View console output
* Cannot configure jobs
* Cannot manage credentials
* Cannot administer Jenkins

**Tester**

* Login to Jenkins
* View QA jobs
* Build QA jobs
* View console output
* Cannot configure jobs
* Cannot manage credentials
* Cannot administer Jenkins

**DevOps**

* Access DEV/QA/PROD jobs
* Configure jobs
* Manage Jenkins
* Manage credentials
* Manage required plugins/nodes

### Expected Result

```text
                Jenkins
                   |
        +----------+----------+
        |          |          |
    Developer    Tester     DevOps
        |          |          |
       DEV        QA       DEV/QA/PROD
        |          |          |
      Build      Build      Admin
```

---

## 30. The One Diagram Students Should Remember

```text
                 USER
                   |
                   ↓
                 ROLE
                   |
          +--------+--------+
          |        |        |
       Developer Tester   DevOps
          |        |        |
          ↓        ↓        ↓
         DEV       QA      PROD
          |        |        |
       Build      Test     Deploy
```

### One-line interview answer

> **Jenkins RBAC allows us to assign permissions based on user roles so that users get only the access required for their responsibilities, following the principle of least privilege.**






# Jenkins Email Notification — Beginner-Friendly Hands-On

## 1. Start with the Real-Time Problem Statement



> **Problem Statement:**
> Developers push code to GitHub. Jenkins automatically starts the build.
>
> But developers don't continuously monitor Jenkins.
>
> If the build succeeds or fails, the developer should automatically receive an email.
>
> Therefore, we configure **email notifications in Jenkins**.

### Real-time flow

```text
Developer
    |
    | git push
    ↓
GitHub
    |
    | Webhook
    ↓
Jenkins
    |
    +---- Build
    |
    +---- Test
    |
    +---- Package
    |
    ↓
SUCCESS / FAILURE
    |
    ↓
Email Notification
    |
    ↓
Developer
```

---

# 2. What Are We Going to Build?

At the end of the hands-on:

```text
Developer pushes code
        ↓
GitHub
        ↓
Jenkins Pipeline
        ↓
Build
        ↓
SUCCESS
        ↓
📧 Email to Developer
```

If the build fails:

```text
Developer pushes code
        ↓
GitHub
        ↓
Jenkins
        ↓
Build
        ↓
❌ FAILURE
        ↓
📧 Failure Email
```

---

# 3. What Do We Need?

For the basic lab, students need:

1. Jenkins server
2. Jenkins job/pipeline
3. Internet connectivity
4. Gmail/SMTP account or another SMTP provider
5. Jenkins email plugins

For a classroom demo, **Gmail SMTP** is easy to understand, but don't teach students to put their normal Gmail password directly into Jenkins. Use an **App Password** where applicable.

---

# 4. Understand SMTP First

Before configuration, explain:

### SMTP

**SMTP = Simple Mail Transfer Protocol**

It is used to **send emails**.

Simple example:

```text
Jenkins
   |
   | SMTP
   ↓
Gmail Mail Server
   |
   ↓
Developer's Inbox
```

Jenkins doesn't directly "send Gmail."

Jenkins connects to the SMTP server.

---

# 5. Gmail SMTP Details

For Gmail SMTP, commonly used settings are:

```text
SMTP Server:
smtp.gmail.com

Port:
465  → SSL
or
587  → STARTTLS

Username:
your Gmail address

Password:
Gmail App Password
```

For your beginner lab, I recommend using:

```text
smtp.gmail.com
Port: 587
STARTTLS
```

---

# 6. Important: Gmail App Password

Tell students:

> Don't use your normal Gmail password in Jenkins.

Instead, when using a Google account with the appropriate security setup, create an **App Password** and use that as the SMTP password.

The concept is:

```text
Gmail Account
      |
      ↓
2-Step Verification
      |
      ↓
App Password
      |
      ↓
Jenkins SMTP Password
```

---

# 7. Step 1 — Install Email Plugin

Login to Jenkins as Administrator.

Go to:

```text
Dashboard
   ↓
Manage Jenkins
   ↓
Plugins
   ↓
Available Plugins
```

Search for:

```text
Email Extension Plugin
```

Install it.

You may also see Jenkins' built-in mailer functionality depending on your Jenkins version.

For teaching more useful notifications, I recommend demonstrating the **Email Extension Plugin**.

---

# 8. Step 2 — Configure SMTP

Go to:

```text
Manage Jenkins
       ↓
System
```

Find:

```text
Extended E-mail Notification
```

Configure:

```text
SMTP server:
smtp.gmail.com

SMTP Port:
587
```

Enable the appropriate TLS/STARTTLS option shown by your Jenkins/plugin version.

Then configure authentication:

```text
Username:
your-email@gmail.com

Password:
App Password
```

---

# 9. Example Configuration

For example:

```text
SMTP Server
    smtp.gmail.com

SMTP Port
    587

Username
    devopslab@gmail.com

Password
    **************

Use TLS / STARTTLS
    Enabled
```

Don't put your real password in screenshots or classroom notes.

---

# 10. Step 3 — Configure Jenkins Default Email

In the same Jenkins System configuration, find the mailer/email section.

Configure:

```text
System Admin Email:
devopslab@gmail.com
```

And configure the default sender/reply address as appropriate for your Jenkins version.

Conceptually:

```text
Jenkins
   |
   | From
   ↓
devopslab@gmail.com
```

---

# 11. Step 4 — Test SMTP Configuration

This is an important step.

Don't immediately create a pipeline.

First test email connectivity if your Jenkins/plugin version provides a **Test configuration by sending a test email** option.

Enter:

```text
Recipient:
your-email@gmail.com
```

Click:

```text
Test configuration
```

Expected:

```text
Email sent successfully
```

Then check Inbox/Spam.

---

# 12. If Email Doesn't Arrive

Teach students to check:

```text
Jenkins Console Output
Jenkins System Log
Gmail Spam/Junk
SMTP username
SMTP port
TLS/STARTTLS
App Password
```

A common mistake is:

```text
Normal Gmail Password
       ↓
Jenkins
       ↓
Authentication Failure
```

Use:

```text
Gmail App Password
       ↓
Jenkins
       ↓
SMTP Authentication
```

---

# 13. Step 5 — Create a Jenkins Pipeline

Create:

```text
New Item
   ↓
Email-Demo
   ↓
Pipeline
```

Use:

```groovy
pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                echo 'Building application...'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
            }
        }

        stage('Package') {
            steps {
                echo 'Packaging application...'
            }
        }
    }
}
```

Run:

```text
Build Now
```

The build should succeed.

---

# 14. Step 6 — Add Email Notification

Now add the notification section.

A simple Email Extension example:

```groovy
post {

    success {
        emailext(
            to: 'developer@example.com',
            subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
Build Successful!

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Build URL: ${env.BUILD_URL}
""",
            recipientProviders: [
                [$class: 'DevelopersRecipientProvider']
            ]
        )
    }

    failure {
        emailext(
            to: 'developer@example.com',
            subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
Build Failed!

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Build URL: ${env.BUILD_URL}
""",
            recipientProviders: [
                [$class: 'DevelopersRecipientProvider']
            ]
        )
    }
}
```

For beginners, however, I recommend first teaching the simpler version.

---

# 15. First Teach the Simple Version

Use:

```groovy
post {

    success {
        emailext(
            to: 'developer@example.com',
            subject: "BUILD SUCCESS: ${env.JOB_NAME}",
            body: "Build ${env.BUILD_NUMBER} was successful."
        )
    }

    failure {
        emailext(
            to: 'developer@example.com',
            subject: "BUILD FAILED: ${env.JOB_NAME}",
            body: "Build ${env.BUILD_NUMBER} failed."
        )
    }
}
```

Explain:

```text
post
 ↓
Runs after pipeline finishes

success
 ↓
Runs when build succeeds

failure
 ↓
Runs when build fails

emailext
 ↓
Sends email
```

---

# 16. Complete Beginner Pipeline

Give students this as their hands-on pipeline:

```groovy
pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                echo 'Building application...'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
            }
        }

        stage('Package') {
            steps {
                echo 'Creating package...'
            }
        }
    }

    post {

        success {
            emailext(
                to: 'developer@example.com',
                subject: "BUILD SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Hello Developer,

Your Jenkins build was successful.

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Build URL: ${env.BUILD_URL}

Regards,
Jenkins
"""
            )
        }

        failure {
            emailext(
                to: 'developer@example.com',
                subject: "BUILD FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Hello Developer,

Your Jenkins build has failed.

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Build URL: ${env.BUILD_URL}

Please check the Jenkins console output.

Regards,
Jenkins
"""
            )
        }
    }
}
```

---

# 17. Step 7 — Test SUCCESS Email

Run:

```text
Build Now
```

Expected:

```text
Build
  ↓
SUCCESS
  ↓
post
  ↓
success
  ↓
emailext
  ↓
📧 Developer receives email
```

Example email:

```text
Subject:
BUILD SUCCESS: Email-Demo #1
```

Body:

```text
Hello Developer,

Your Jenkins build was successful.

Job: Email-Demo
Build Number: 1
Build URL: http://jenkins-server/job/Email-Demo/1/

Regards,
Jenkins
```

---

# 18. Step 8 — Test FAILURE Email

Now intentionally create a failure.

For example:

```groovy
stage('Test') {
    steps {
        sh 'exit 1'
    }
}
```

Flow:

```text
Build
 ↓
Test
 ↓
exit 1
 ↓
FAILURE
 ↓
post
 ↓
failure
 ↓
Email
```

Developer receives:

```text
Subject:
BUILD FAILED: Email-Demo #2
```

---

# 19. Now Explain Real-Time Developer Notification

This is the important part of your question.

In a real project, you don't necessarily want to hard-code:

```groovy
to: 'developer1@gmail.com'
```

for every pipeline.

Instead, the team can configure recipients based on the project/team setup.

Conceptually:

```text
Git Commit
    ↓
Jenkins Build
    ↓
Who is responsible for this job?
    ↓
Developer / Team
    ↓
Email Notification
```

The Email Extension Plugin also supports recipient providers that can derive recipients from Jenkins build information, depending on how the Jenkins job and SCM are configured.

---

# 20. Real-Time Example

Suppose:

```text
Project: EmployeeApp

Developers:
Amit
Rahul
Pravin
```

Jenkins pipeline:

```text
GitHub
   ↓
EmployeeApp Pipeline
   ↓
Build
   ↓
Test
   ↓
Package
```

If successful:

```text
📧 SUCCESS
```

If failed:

```text
📧 FAILURE
```

The team gets notified without someone manually checking Jenkins.

---

# 21. What Should a Failure Email Contain?

Teach students that a useful notification should contain:

```text
Project Name
Build Number
Build Status
Branch
Commit
Build URL
Reason / Console URL
```

Example:

```text
Subject:
FAILED: EmployeeApp #25

Project:
EmployeeApp

Branch:
main

Build:
#25

Status:
FAILED

Jenkins:
http://jenkins/job/EmployeeApp/25/

Action:
Please check Console Output.
```

This is much more useful than simply:

```text
Build Failed.
```

---

# 22. Real-Time DevOps Flow

Now connect this with the topics you have already taught:

```text
Developer
    |
    | git push
    ↓
GitHub
    |
    | webhook
    ↓
Jenkins
    |
    ├── Checkout
    ├── Maven Build
    ├── Unit Test
    ├── SonarQube
    ├── Docker Build
    └── Deployment
            |
            ↓
       SUCCESS / FAILURE
            |
            ↓
       Email Notification
            |
            ↓
        Developer
```

For your fresher class, you can simplify it to:

```text
GitHub
   ↓
Jenkins
   ↓
Build
   ↓
Test
   ↓
Success / Failure
   ↓
Email
```

---

# 23. Combine RBAC + Email Notification

This makes an excellent **real-time Jenkins scenario** for your students.

```text
                    Jenkins
                       |
        +--------------+--------------+
        |              |              |
    Developer        Tester         DevOps
        |              |              |
       DEV             QA            PROD
        |              |              |
      Build           Test          Deploy
        |              |              |
        +--------------+--------------+
                       |
                       ↓
                Email Notification
```

Example:

```text
Developer builds DEV job
        ↓
Build fails
        ↓
Developer receives email
```

Tester:

```text
Tester builds QA job
        ↓
Test fails
        ↓
Tester receives email
```

DevOps:

```text
Production deployment fails
        ↓
DevOps receives notification
```

---

# 24. 1-Hour Teaching Plan

Since you're teaching beginners, I would structure this as:

| Time      | Topic                              |
| --------- | ---------------------------------- |
| 0–10 min  | Problem statement + SMTP concept   |
| 10–20 min | Gmail/App Password + SMTP          |
| 20–30 min | Jenkins email configuration        |
| 30–40 min | Pipeline + `post` + `success`      |
| 40–50 min | Intentionally fail build           |
| 50–60 min | Failure email + real-time scenario |

### Students should remember only these 4 concepts:

```text
SMTP
  ↓
Jenkins Email Configuration

post
  ↓
Runs after pipeline

success
  ↓
Success notification

failure
  ↓
Failure notification
```

And the final real-world flow:

```text
Developer → GitHub → Jenkins → Build/Test → Email
```

**One-line interview answer:**

> "We configure SMTP in Jenkins and use the Email Extension Plugin with pipeline `post` conditions to automatically notify developers when a build succeeds or fails."
