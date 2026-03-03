---
layout: post
title: "Secure Temporary File Upload Service. PHP. NextCloud. WebDAV. Upload files form."
date: 2024-11-02 22:39:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/11/secure-temporary-file-upload-service.html
---

Завершив невеликий проєкт зі створення форми для завантаження файлів у хмарне
сховище.   
Проєкт реалізовано за допомогою PHP, Redis та WebDAV для
забезпечення ефективного зберігання та обробки файлів.

[![](/assets/images/blog/A%20professional,%20modern%20illustration%20of%20a%20secure%20file-upload%20service%20with%20key%20features%20highlighted.%20The%20design%20shows%20a%20user-friendly%20interface%20for%20file.webp)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhkaajkOl-jOuIySsKX6psbE8AkIS9AQy-HpH92EWhlWU5S07GjYUL1HTZWCyudBKzH3gApQOoxvSab77K7y0bwQQyRVkv2Az01HK0BtGXqAAYkUvnO97_2-956W5GyDqBR_hgA7uqVTN8K_BUMHUjwQDSE8bXhUPnhxhUbCQUSBXrVvJaAitPcxMJxWkx7/s1024/A%20professional,%20modern%20illustration%20of%20a%20secure%20file-upload%20service%20with%20key%20features%20highlighted.%20The%20design%20shows%20a%20user-friendly%20interface%20for%20file.webp)

#### Overview:

This project provides a secure, temporary file-upload service with time-limited access. It generates a unique URL for file uploads, includes CAPTCHA protection, and sends the URL to the user via email. The following is a list of key functionalities and features implemented in this project.

#### Key Features:

### Secure URL Registration

* Generates a unique URL for each request.
* Protects URL generation with CAPTCHA to prevent abuse.

### Time-Limited URL Access

* URL expires 48 hours after creation.
* URL is sent only to the provided email address.
* After the URL is activated, the expiration time reduces to 1 hour for security.

### User File Upload

* Allows users to upload files up to 1GB.
* Displays upload progress, including elapsed and remaining time estimates.

### File Storage on NextCloud

* Uploaded files are stored in a fixed folder within NextCloud storage.
* Original filenames are replaced with a new generated name for privacy.

### Metadata Generation

* A `.txt` metadata file is created for each uploaded file.
* The metadata file contains the original filename and the sender’s email address.

#### Basic:

* [Accessing Nextcloud files using WebDAV](https://docs.nextcloud.com/server/latest/user_manual/da/files/access_webdav.html#accessing-files-using-curl)

```
$ curl -u user:pass -T error.log "https://example.com/nextcloud/remote.php/dav/files/USERNAME/$(date '+%d-%b-%Y')/error.log"
```

#### 

#### Solution:

GitHub: <https://github.com/lexxai/php_form_upload_files_to_wevdab>

#### 

#### Examples:

[![](/assets/images/blog/%D0%97%D0%BD%D1%96%D0%BC%D0%BE%D0%BA%20%D0%B5%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-10-18%20014201.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEi_qlwgQLPrZQWeQws_P2XlVMSTngNWN5stbUkW88GBxSxLCgwDYx5rN7Hhlb9Vlm0B-GpHal10XKma8g3FXNNE8u-KlzdRHTg3xOep275tQY4A3FBjoEePReKJaE41WDzzFXYoNeF5v7ycspVftceiGzNJWy588keWtzWNfOtf-tbD276_Bh4vinIptQIc/s908/%D0%97%D0%BD%D1%96%D0%BC%D0%BE%D0%BA%20%D0%B5%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-10-18%20014201.png)

  

[![](/assets/images/blog/%D0%97%D0%BD%D1%96%D0%BC%D0%BE%D0%BA%20%D0%B5%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-10-18%20013839.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhJO0qbfPBJFfGtRrHvBIh2_pKdeHCKGCkf_ju6SJ7lM98C93IccBPhQhU1srBcR2eCD6excDyKEy3otcGVz7p8tYfEWlKUHqyZ2BHJSjpuzJS2A5273Px9eL2dP9qTasfv5wRRjaHj6KpBiegIRFYpDo0igFHvK_A_pZOz2sE2GOOv7MKnbyObiHKxH6g-/s886/%D0%97%D0%BD%D1%96%D0%BC%D0%BE%D0%BA%20%D0%B5%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-10-18%20013839.png)

  

####
