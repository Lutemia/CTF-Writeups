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
### Step 1: Initial Site Exploration

First, the blog page is a simple page with 3 links
1. Blog 1
2. Blog 2
3. Blog 4
---

### Step 2: IDOR:
Obviously, blog 3 is missing, changing the url to `/blog 3` shows a hidden page with a resume. Vieiwing the page source gives some hints:

`"for documents in the 'other' folder only people with the api key has access"`
---

### Step 3: Exposed API Key
There is also an api key in the url of the pdf embedded on the page:
`file=resume.pdf&apiKey=906392d25b3bd7d3af491799f89f6620` <br>
---

I noticed the other files on the website, like images, are served through the endpoint `/attachment?file=` so I figured this must be the intended exploit path.
Trying things like `/attachment?file=flag.txt&apiKey` and `/attachment?file=flag.txt` returned 404 pages. I remembered the hint `documents in the 'other' folder`. What if we tried to move into the 'other' folder with directory traversal?
---
