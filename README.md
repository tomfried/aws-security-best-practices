# AWS Best Practices
For all new and existing AWS projects, the following are the minimum best practices I can think for access.

## 1. Immediately Remove Access Key
When you first create an account, if it creates an access key, delete it at:
https://console.aws.amazon.com/iam/home#security_credential

If link does not work, this is found under the user link in the header then selecting "**Security Credentials**".
<details>
  <summary><i>(view Screenshot)</i></summary>
  ![screenshot of how to navigate to security credentials](./1.-Navigating-to-Security-Credentials.png)
</details>

Then go down to AWS "**Access Keys**" and ensure it is deleted (if one was made automatically).
<details>
  <summary><i>(view Screenshot)</i></summary>
  ![screenshot of Access Keys section](./2.-Access-Keys-section.png)
</details>
