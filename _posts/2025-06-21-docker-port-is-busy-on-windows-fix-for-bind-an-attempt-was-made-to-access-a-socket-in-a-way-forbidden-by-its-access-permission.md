---
layout: post
title: "Docker Port Is Busy on Windows? Fix for: bind: An attempt was made to access a socket in a way forbidden by its access permission."
date: 2025-06-21 19:27:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2025/06/docker-port-is-busy-on-windows-fix-for.html
---

## If you're getting a Docker error like:

```
(HTTP code 500) server error - ports are not available: exposing port TCP 0.0.0.0:8182 -> 127.0.0.1:0: listen tcp 0.0.0.0:8182: bind: An attempt was made to access a socket in a way forbidden by its access permissions.
```

But nothing is using the port, it’s likely because Windows has excluded the port dynamically, especially after Docker was previously using it.

|  |
| --- |
|  |
| (HTTP code 500) server error - ports are not available |

#### 1. Verify Nothing Is Listening on the Port

```
netstat -aon | findstr :8181
```

If you see no output, the port is not in use by any application.

#### 2. Check Excluded Port Ranges:

```
netsh interface ipv4 show excludedportrange protocol=tcp

Protocol tcp Port Exclusion Ranges

Start Port    End Port
----------    --------
      6575        6674
      7162        7261
      7262        7361
      7362        7461
      7462        7561
      7562        7661
      7681        7780
      7781        7880
      7881        7980
      7981        8080
      8081        8180
      8181        8280
      8281        8380
      8381        8480
      8481        8580
     30604       30703
     49690       49789
     49790       49889
     49890       49989
     50000       50059     *
     50160       50259
     50360       50459
     50460       50559
     50660       50759
     50760       50859
     50860       50959
     51064       51163
     51243       51342
     51344       51443
```

You’ll notice 8182 falls within the 8181–8280 range - without a \* at the end, meaning it's a dynamically excluded port.  
  
You cannot remove dynamic exclusions manually.

#### 3. Restart winnat Service to Clear Dynamic Exclusions:

This step clears dynamic port reservations created by services like Docker:

```
net stop winnat
net start winnat
```

\* Run the above in Administrator Command Prompt

#### 4. Recheck Excluded Ports:

```
netsh int ipv4 show excludedportrange protocol=tcp

Protocol tcp Port Exclusion Ranges

Start Port    End Port
----------    --------
     50000       50059     *

* - Administered port exclusions.
```

If 8182 is no longer in the list, you're good to go.
