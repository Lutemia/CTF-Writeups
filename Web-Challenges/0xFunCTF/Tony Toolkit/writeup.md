
# Tony Toolkit 

>> Tony decided to launch bug bounties on his website for the first time, so it's likely to have some very common vulnerabilities.
---
This challenge involved exploiting a vulnerable item search web page to gain unathorized access to user credentials and access the admin account.
<br><br>
To be successful I needed to combine:

1. SQL Injection
2. Password hash cracking
3. Cookie manipulation (Broken access control)
---

## Writeup
### Step 1: Initial Site Exploration

The home and login pages are straightforward, just a search feature and standard username and password fields, respectively. I try searching an item and see `/search?item=xxxx`. My first thought tells me the site may be vulnerable to SQL injection.<br>

### Step 2 SQL Injection 
I decide to test for SQL injection with `' ORDER BY 1 --`. 
<br>
The response returns a list of items, and just to be safe I also test `' ORDER BY 2 --` and `' ORDER BY 3 --`. <br><br>
And `' ORDER BY 3 --` returns the error "1st ORDER BY term out of range - should be between 1 and 2." Which tells me there are two columns worth of items and I'd reached the limit of items. 
I also remembered that retrieving data from other tables could be possible so I injected

```
UNION SELECT ‘username’,NULL--
```
The result was a table with the columns `name` and `price` like the other tools, but the name in this instance was `username` with a price of `$None`. <br><br>
Bingo! I take it a step further and try to pull usernames and passwords:
```
' UNION SELECT username, password FROM users—
```
The list now includes tools as well as two usernames and passwords:

| Username | Password ('Price')|
| ----------- | ----------- |
| Admin | `$0000000000000000000000000000000000000000000000000000000000000000` |
| Jerry | `$059a00192592d5444bc0caad7203f98b506332e2cf7abb35d684ea9bf7c18f08` = `1qaz2wsx`|

I suspected these passwords were hashed SHA-256 and using a hash cracker, the password was revealed to be `1qaz2wsx`.
Now that I have a set of credentials I can try logging in and see what the website presents to me. <br>
### Step 3: Login and Exploit IDOR (Broken Access Control)
The page is very simple, a profile page with a list of 'What to Buy' and 3 items. I also see the url has changed to `/user` so naturally I attempt to change it to `/admin`, to see if an admin page will be revealed, but no luck there. <br>
I inspect the page and take a look at the application tab and see: <br>
| Name | Value |
| ----------- | ----------- |
| user | `0cea94be4ad3fc313cee0f65c3fd5dbc5dcf93d7e1bb337f2ecac06e52f29c28` |
| userID | `2`|

Meaning Jerry's userID value is equal to 2 and we know there are only 2 users, so what if we try changing that number to 1 instead and refreshing the page? <br>
The page reloads and presents the flag!

---
## Flag
```
0xfun{T0ny'5_T00ly4rd._1_H0p3_Y0u_H4d_Fun_SQL1ng,_H45H_Cr4ck1ng,_4nd_W1th_C00k13_M4n1pu74t10n}
```

## Real World Lessons Learned 
1. SQL Injection - Database discovery and credential leaks
2. Secure Password Storage - Jerry's password was hashed with SHA-256 but still crackable
3. Client-Side Data - Able to manipulate local cookies to access admin account

## Connecting to OWASP Top 10 
1. SQL Injection - A05:2025 - Injection
   > An injection vulnerability is an application flaw that allows untrusted user input to be sent to an interpreter (e.g. a browser, database, the command line) and causes the interpreter to execute parts of that input as commands.
   
2. Password Exposure - A04:2025 - Cryptographic Failures
   > This weakness focuses on failures related to the lack of cryptography, insufficiently strong cryptography, leaking of cryptographic keys, and related errors.
   
3. Accessing the Admin Account - A01:2025 - Broken Access Control
   > Access control enforces policy such that users cannot act outside of their intended permissions. Failures typically lead to unauthorized information disclosure, modification or destruction of all data, or performing a business function outside the user's limits.




