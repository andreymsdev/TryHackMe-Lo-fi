# Lo-fi — TryHackMe

---

Want to hear some lo-fi beats, to relax or study to? We’ve got you covered!

---

## Website

Navigate to the following URL using the **AttackBox**: `http://MACHINE_IP` and find the flag in the root of the filesystem.

---

## Reconnaissance

I started with an **Nmap** scan against the target `10.65.166.175`:

`nmap -T4 -n -sC -sV -Pn 10.65.166.175`

### Findings:

* **22/tcp** → OpenSSH 8.2p1 (Ubuntu).
* **80/tcp** → Apache 2.2.22 serving the “Lo-Fi Music” site.

This confirmed the host was alive and running a potentially **vulnerable web application**.

---

## Spotting the Vulnerability

Inspecting the HTML, we can notice links like:

`/?page=relax.php`

This suggested the site was using PHP's **`include()`** function with **user-controlled input**. That's a classic **Local File Inclusion (LFI)** vector.

---

## Exploiting LFI

We tested **path traversal** with:

`http://10.65.166.175/?page=../../../../../etc/passwd`

The server responded with the contents of `/etc/passwd`, **proving the LFI worked**.

---

## Hunting the Flag 

The challenge stated the flag was located at the **root of the filesystem**. So I tried **`curl`** on the terminal.

```bash
curl "[http://10.65.166.175/?page=../../../../../flag.txt](http://10.65.166.175/?page=../../../../../flag.txt)"
curl "[http://10.65.166.175/?page=../../../../../flag](http://10.65.166.175/?page=../../../../../flag)"
curl "[http://10.65.166.175/?page=../../../../../.flag](http://10.65.166.175/?page=../../../../../.flag)"
```
 By adjusting the number of ```../,``` we escaped the web root (/var/www/html) and reached /. Once the correct path was hit, the flag appeared in the HTTP response.
 Understanding

Each ../ means “go up one directory.”

Example:

The script is likely located at: ```/var/www/html/index.php```
The payload ```../../../../../etc/passwd``` climbs back to the root directory (/) and then enters ```/etc/passwd```.

  This is how we broke out of the web directory and accessed system files.
