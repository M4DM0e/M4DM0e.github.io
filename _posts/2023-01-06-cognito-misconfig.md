---
layout: post
title:  "How I was able to takeover any account (Zero-click ATO)"
categories: blog
tags: bugbounty
author:
  - Mohammed Al-Barbari
---
 

### السلام عليكم ورحمة الله وبركاتة 
## Who am I?
I am Mohammed Fadhl Al-Barbari, a cybersecurity researcher, tool developer, and bug hunter from Yemen, active on platforms such as HackerOne (H1), BugCrowd, and SRT.

## Connect with me on:
**Twitter**:     [Mohammed Al-Barbari](https://twitter.com/m4dm0e)
**LinkedIn**:     [Mohammed Al-Barbari](https://www.linkedin.com/in/albarbari/)

**It's my first writeup, so excuse any mistakes (thanks, ChatGPT for having my back)!**

## Diving into the Discovery: A December to Remember
In December, I discovered a critical bug on HackerOne. A private program was vulnerable to a full account takeover due to misconfigurations in Amazon Cognito.

Why is this significant? Well, even after the account owner changes their password, an attacker can still access the account with the initial password set. Pretty cool, huh?

![Alt Text](https://media4.giphy.com/media/v1.Y2lkPTc5MGI3NjExY281M3hpdjBzd2hqM2RhbGMzYWg2azE0dGwyNXphemptaTRjZHg2ciZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/SQQ5VpVKhCM9O/giphy.gif)


## Introduction:
While testing an app, which I'll refer to as `redacted.com`, focused on cashback and vouchers, I noticed it used Amazon Cognito-IDP for login and sign-up processes. Cognito-IDP is known for some misconfiguration vulnerabilities, as highlighted in several other write-ups:

[Amazon cognito misconfiguration](https://systemweakness.com/amazon-cognito-misconfiguration-4e90d14377c7)
[Hunting For AWS Cognito Security Misconfigurations](https://www.yassineaboukir.com/talks/NahamConEU2022.pdf)
[Hacking AWS Cognito Misconfigurations](https://notsosecure.com/hacking-aws-cognito-misconfigurations)
[Flickr Account Takeover](https://security.lauritz-holtmann.de/advisories/flickr-account-takeover/)

### What is Cognito-IDP?
>Amazon Cognito provides authentication, authorization, and user management for customer's web and mobile applications. Users can sign in directly with a user name and password, or through a third party such as Facebook, Amazon, Google, Apple, or enterprise identity providers via SAML 2.0 and OpenID Connect.

**Note:** There was two apps in scope were using cognito-idp. let's call them `redacted.com` and `redacted2.com` 

After seeing Cognito-IDP,  I spent a couple of days checking it out. Most of the things I found didn't apply to what I was testing.

I spent most the time testing `redacted.com` and there was only one bug I was able to find but unfortunatlly it was almost impossible to get an account takeover. 
Moreover, if you gain access to the victim's account, you can render it inaccessible to the victim, preventing their login.

### Changing account's email without OTP code:

While logged in, changing the email address typically requires OTP verification. 
![image](../../../assets/images/emailchanging.png)


However, I discovered that using AWS CLI commands to retrieve user information and modify user attributes bypassed this requirement.

Example AWS CLI command to view user attributes:
```bash
aws cognito-idp get-user --access-token [ACCESS_TOKEN] --region REGION
```
You will need the access token of the account to view or modify user attributes. (Also, Region is needed but most of the time you will find it out from the app requests)
![image](../../../assets/images/getuserinfo.png)


After viewing the user attributes, I tried the first scenario, changing the email to an existing email. (It worked, but you can't verify it, so it is useless)

To change the email:
```bash
aws cognito-idp update-user-attributes   --user-attributes Name=email,Value="newEmail@mailna.co" --access-token  [ACCESS_TOKEN] --region REGION
```
This command didn't show any output for me, so right after executing the command, I tried to retrieve users' info using the command I used before.
![image](../../../assets/images/emailischanged.png)

Though this changes the email, it remains unverified, preventing login or password reset.
If you log out, you can't log in again or even reset your password. You have to contact support to delete your account and make a new one.

![Alt Text](https://i.imgur.com/PxO2Qv9.gif)

### Working harder
A day later, I thought, let's try `redacted2.com`. Maybe it's different. I created a test account with email `testpoc1@mailna.co` and started playing with Burp Suite, 

The first thing that came to my mind is to try changing the email just like I did with `redacted.com`, and this time it didn't work because it will not let you change the password without the OTP, so OTP is required, and you can't change anything without it.


Then one idea came to my mind; I saw before that Cognito-IDP sometimes where the email address is case-sensitive, so I tried to register a new account with an existing email.
I tried with Burp Suite, but it didn't work because requests were sent to `cognito-idp.redacted.com`, so it was showing that the email address already exists.

I started looking in requests, trying to find a solution, and I saw `cognito-idp.eu-east-1.amazonaws.com/eu-east-1_XXXXXXXX.`

Then I tried to send the same registration request to `cognito-idp.eu-east-1.amazonaws.com/eu-east-1_XXXXXXXX` instead of `cognito-idp.redacted.com`

##### Note that I'm able to use AWS CLI, but I wanted to write a script because it is too much effort to take over an account using AWS CLI.

َ
#### Request:
```
POST /eu-east-1_XXXXXXXX HTTP/2
Host: cognito-idp.eu-east-1.amazonaws.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:121.0) Gecko/20100101 Firefox/121.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://www.redacted.com/
Content-Type: application/x-amz-json-1.1
X-Amz-Target: AWSCognitoIdentityProviderService.SignUp
X-Amz-User-Agent: aws-amplify/5.0.4 auth framework/0
Cache-Control: no-store
Content-Length: 213
Origin: https://www.redacted.com
Dnt: 1
Sec-Gpc: 1
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-site
Te: trailers


{
  "ClientId": "CLIENT_ID",
  "Username": "tEstPoc1@mailna.co",
  "Password": "Password!@#_",
  "UserAttributes": [],
  "ValidationData": [
    {
      "Name": "password",
      "Value": "Password!@#_"
    }
  ]
}
```

### Response:
```
{"UserConfirmed":true,"UserSub":"b627........"}
```

![Alt Text](https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExYnJoNWdmdXpybnB0OWQyd253czhoamF3cWtyand5ZGR1ZGJyd2ttYiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/Sqfu14lSonVN219Zb6/giphy.gif)

I was happy that the account is created, but it didn't last so long, lol. When I tried to log in to redacted2.com, it shows that the email or password is incorrect :(

![Alt Text](https://media1.giphy.com/media/3o7btYLAW7doynq3p6/giphy.gif?cid=ecf05e47xfgoyljpem9hieta41yq3mjeci4e6ux5sv42h67a&ep=v1_gifs_search&rid=giphy.gif&ct=g)


I took a short break and then came again to see what is the problem. Then I tried to log in using AWS CLI using `tEstPoc1@mailna.co` and the password I set, and it worked!?!?!

It responded with AccessToken, and refreshToken.

I realized that the application is not considering the uppercase in the email address, and it will only log you in if you have the real password. However, using AWS CLI, the email is case-sensitive, and it sees `tEstPoc1@mailna.co` and `testpoc1@mailna.co` as different accounts. But it gives the same AccessToken, so the AccessToken I got from AWS CLI I used in the application, and it logged me into the victim's account.

So, an attacker can basically get an AccessToken of the victim's account from the AWS CLI while the victim can log in to the account using the real password. No matter how many times the victim changes the password, an attacker will always be able to log in using the case-sensitive email and password set while taking the victim's account over!


![Alt Text](https://media0.giphy.com/media/25JGQ0SPpafi8/giphy.gif?cid=ecf05e477jlkbjzeus3osgri2a7rwtyj2pxnd5quolrd5ucu&ep=v1_gifs_search&rid=giphy.gif&ct=g)
