# My First Blog
>> "I've been getting into this personal blog thing. It's been really fun but apparently you're not supposed to post certain info on the interwebs. Flag is at /flag.txt".
---

This challenge involved exploiting a blog site that hosted different blog pages and photos.
I needed to exploit:

1. IDOR
2. Exposed API Keys
3. Directory Path Traversal
---

## Writeup
### Step 1: Initial Site Exploration:

First, the blog page is a simple page with 3 links
1. Blog 1
2. Blog 2
3. Blog 4
---

### Step 2: IDOR:
Obviously, blog 3 is missing, changing the url to `/blog 3` shows a hidden page with a resume. Vieiwing the page source gives some hints:

```
"for documents in the 'other' folder only people with the api key has access"
```

### Step 3: Exposed API Key:
There is also an api key in the url of the pdf embedded on the page:

```
file=resume.pdf&apiKey=906392d25b3bd7d3af491799f89f6620
```

I noticed the other files on the website, like images, are served through the endpoint `/attachment?file=` so I figured this must be the intended exploit path.

Trying things like: 
`/attachment?file=flag.txt&apiKey` and `/attachment?file=flag.txt` returned 404 pages. I remembered the hint: 

```
"documents in the 'other' folder."
```
What if we tried to move into the 'other' folder with directory traversal?<br>
This reveals the flag!

```
/attachment?file=../../../../flag.txt&apiKey=906392d25b3bd7d3af491799f89f6620
```

---

## Flag
```
bkctf{k3ys_in_th3_l0ck5}
```

## Real World Lessons Learned & Connecting to OWASP Top 10
1. IDOR/Changing the URL to Find Hidden Pages - A01 2025: Broken Access Control
   > Force browsing (guessing URLs) to authenticated pages as an unauthenticated user or to privileged pages as a standard user
2. Exposed API Key/Comments Exposing File System - A02 2025: Security Misconfiguration
   > Unnecessary features are enabled or installed (e.g., unnecessary ports, services, pages, accounts, testing frameworks, or privileges).
3. Directory Traversal - A01 2025: Broken Access Control
   > Bypassing access control checks by modifying the URL (parameter tampering or force browsing), internal application state, or the HTML page, or by using an attack tool that modifies API requests.
