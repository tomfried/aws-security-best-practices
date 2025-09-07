# AWS Best Practices
For all new and existing AWS projects, the following are the minimum best practices I can think for access.

## 1. Immediately Remove Access Key
When you first create an account, if it creates an access key, delete it at:
https://console.aws.amazon.com/iam/home#security_credential

<ol>
  <li>If link does not work, this is found under the user link in the header then selecting "<b>Security Credentials</b>".
    <details>
      <summary><i>(view Screenshot)</i></summary>
      <img alt="screenshot of how to navigate to security credentials" src="./images/1.Navigating-to-Security-Credentials.png" width="50%"/>
    </details>
  </li>
  <li>Then go down to AWS "<b>Access Keys</b>" and ensure it is deleted (if one was made automatically).
    <details>
      <summary><i>(view Screenshot)</i></summary>
      <img alt="screenshot of Access Keys section" src="./images/2.Access-Keys-section.png"/>
    </details>
  </li>
</ol>

## 2. Setup MFA (Multi-Factor Authentication)
While still on the "[Security Credentials page](https://console.aws.amazon.com/iam/home#security_credential)" setup 1.) both a <b>Passkey</b> for the computer you used most and 2.) MFA through the "Google <b>Authenticator app</b>" or similar".
<details>
  <summary><i>(view Screenshot)</i></summary>
  <img alt="screenshot of MFAs setup" src="./images/3.Screenshot-of-MFAs.png"/>
</details>

## 3. Setup CloudTrail to Track/Record Account Activity
<ol>
  <li>Navigate to **[AWS CloudTrail](https://console.aws.amazon.com/cloudtrail)**.</li>
  <li>Select "<b>Create Trail</b>"</li>
  <li>Give it name, ex "management-events"</li>
  <li>Create new S3 bucket to store the logs</li>
  <li><b>Save</b> then can see all actions that happen on your account whenever you return back to **[AWS CloudTrail](https://console.aws.amazon.com/cloudtrail)**.
    <details>
      <summary><i>(view Screenshot)</i></summary>
      Example after doing a few things:
      <img alt="screenshot of CloudTrail home page and events showing up" src="./images/7.CloudTrail.png" width="100%"/>
    </details>
  </li>
</ol>

## 4. Create IAMM Role for all dev/IT work
The Idea/Thinking:
- **Root User** - will have access to billing, route53, control of users, and literally everything else but won't have keys setup to SSH into your EC2 resources.
- **IAM User** - will be the only account you ever use regularly and is the account that will have SSH access to the EC2 resources.

<ol>
<li>Go to "<b>[AWS IAM](https://console.aws.amazon.com/iam/home#/groups)</b>" > <b>User Groups</b> > then "<b>Create Group</b>".</li>
<li>Call it ex. "EC2PowerUser"</li>
<li>Add Permissions:
  <ul>
    <li>AmazonEC2FullAccess</li>
    <li>EC2InstanceConnect</li>
  </ul>
</li>
<li>Then "<b>Create new [IAM User](https://console.aws.amazon.com/iam/home#/users)</b>"</li>
<li>Assign user to the group you made, ex: "EC2PowerUser"</li>
<li>After saving, grant user console access so they can turn on/off the EC2 instances when needed BUT be sure to <b>select that the password must change on login</b>.
  <details>
    <summary><i>(view Screenshot)</i></summary>
    <img alt="screenshot of enabling console access to IAM user" src="./images/4.Enabling-Console-Access-to-User.png" width="75%"/>
  </details>
</li>
<li>Then give the user (if not yourself) the credentials.
  <details>
    <summary><i>(view Screenshot)</i></summary>
    <img alt="screenshot of saving off password and other login info for user" src="./images/5.Saving-Password-And-Login-Information-to-give-user.png" width="75%"/>
  </details>
</li>
<li>"<b>If it is yourself</b>", then setup Multi-Factor-Authentication for this user too. Ex. both for the most common computer used and on an authenticator app.
  <details>
    <summary><i>(view Screenshot)</i></summary>
    <img alt="screenshot of setting up MFA for the new IAM user if that user is also you" src="./images/6.Setting-up-MFA-for-IAM-user-if-self.png" width="100%"/>
  </details>
</li>
</ol>

## 5. Create Key Pair for EC2 for IAM user
1. Go to **Key Pairs**, then select to create one.
2. Give it a name, select "**RSA**", "**.pem**".
3. Then based on the state of your EC2, do one of the following:

### OPTION 1 - Add Cert to new EC2
<details>
  <summary><i>(view Steps)</i></summary>
  <ol>
    <li>When you launch the new EC2 instance, there will be an option there to assign it to a cert.
    <img alt="screenshot of the assigning key at launch" src="./images/8.Assigning-key-pair.png" width="100%"/></li>
    <li>Then when you launch it, will always shown "<b>Key pair assigned at launch</b>".
    <img alt="screenshot of the key assigned at launch" src="./images/9.Key-assigned-at-launch-showing-new-cert.png" width="100%"/></li>
  </ol>
</details>

### OPTION 2a - Add Cert to existing EC2
<details>
  <summary><i>(view Steps)</i></summary>
  1. SSH into it using the key it already supports (if applicable)
  2. Add new .pem cert to _________
</details>

### OPTION 2b - Add Cert to existing EC2 if you deleted old SSH cert
<details>
  <summary><i>(view Steps)</i></summary>
  Either clone it via AMI or detach EBS by doing ________
</details>
