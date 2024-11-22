# 1. Brute Force aanval;
  ### De input die je gebruikt om de WAF-regel te testen;

  ```bash
    #!/bin/bash

    # Path to the wordlist
    WORDLIST="/usr/share/wordlists/rockyou.txt"

    # Target URL and email
    URL="http://10.2.0.145/rest/user/login"
    EMAIL="a@a.a"

    # Loop through each password in the wordlist
    while IFS= read -r PASSWORD; do
        echo "Trying password: $PASSWORD"
        
        # Send the curl request
        RESPONSE=$(curl -s -X POST "$URL" \
            -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0' \
            -H 'Accept: application/json, text/plain, */*' \
            -H 'Accept-Language: en-US,en;q=0.5' \
            -H 'Accept-Encoding: gzip, deflate' \
            -H 'Referer: http://10.2.0.145/' \
            -H 'Content-Type: application/json' \
            -H 'Origin: http://10.2.0.145' \
            -H 'Connection: keep-alive' \
            -H 'Cookie: language=en; welcomebanner_status=dismiss' \
            -H 'Priority: u=0' \
            --data-raw "{\"email\":\"$EMAIL\",\"password\":\"$PASSWORD\"}")

        # Check the response for success (adjust the condition based on the application's behavior)
        if [[ "$RESPONSE" != *"Invalid email or password"* ]]; then
            echo "Success! Password found: $PASSWORD"
            echo "Response: $RESPONSE"
            break
        fi
    done < "$WORDLIST"
  ```

  ### De WAF-regel die de aanval zou moeten blokkeren;

  ```bash
    SecRule IP:BLOCKED "@eq 1" "id:1005,phase:1,de ny,status:403,msg:'Repeat offender blocked'"
    SecAction "id:1006,phase:5,deprecatevar:ip.blocked=1"
  ```

  ### Een log entry (of een screenshot hiervan) van de WAF waaruit duidelijk blijkt dat er een aanval werd tegengehouden;



  ### Een uitleg waarom de WAF-regel al dan niet werkt;

  Tijdens een manuele

# 2. SQL injection;
  ### De input die je gebruikt om de WAF-regel te testen;

  ```sql
    ' OR 'a'='a
  ```

  ### De WAF-regel die de aanval zou moeten blokkeren;

  ```bash
    SecRule ARGS "@contains sql" "id:1002,phase:2,deny,status:403,msg:'SQL injection blocked'"
  ```

  ### Een log entry (of een screenshot hiervan) van de WAF waaruit duidelijk blijkt dat er een aanval werd tegengehouden;

  ![sql injection blocked](sql_injection_blocked.png)

  ### Een uitleg waarom de WAF-regel al dan niet werkt;

  We hebben geprobeerd om een SQL aanval manueel uit te voeren. De SQL injection wordt geblokkeerd.


# 3. Cross site scripting;
  
  ### De input die je gebruikt om de WAF-regel te testen;

  ```javascript
    <script>alert('XSS attack')</script>
  ```

  ### De WAF-regel die de aanval zou moeten blokkeren;

  ```bash
    SecRule ARGS "@contains script" "id:1001,phase:2,deny,status:403,msg:'XSS attack blocked'"
  ```

  ### Een log entry (of een screenshot hiervan) van de WAF waaruit duidelijk blijkt dat er een aanval werd tegengehouden;

  ![xss attack blocked](xss_attack_blocked.png)

  ### Een uitleg waarom de WAF-regel al dan niet werkt;

  We hebben een XSS aanval geprobeerd uit te voeren. Deze wordt geblokkeerd.

# 4. CSRF aanval;
  ### De input die je gebruikt om de WAF-regel te testen;

  ```html
  <html>
    <body>
      <h1>🎉 Win een gratis Juice Shop waardebon! 🎉</h1>

      <form
        id="csrf-form"
        action="http://localhost:80/#/privacy-security/change-password"
        method="POST"
      >
        <input type="hidden" name="new" value="gehackt123" />
        <input type="hidden" name="repeat" value="gehackt123" />
        <input type="hidden" name="current" value="" />
      </form>

      <script>
        window.onload = function () {
          // Methode 1: Form submit
          document.getElementById("csrf-form").submit();

          // Methode 2: Fetch API
          fetch("http://localhost:80/#/privacy-security/change-password", {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify({
              new: "gehackt123",
              repeat: "gehackt123",
              current: "",
            }),
            credentials: "include",
          });
        };
      </script>
    </body>
  </html>
  ```

  ### De WAF-regel die de aanval zou moeten blokkeren;

  ```bash
    SecRule REQUEST_METHOD "POST" "id:2001,phase:2,chain,deny,status:403,msg:'CSRF token missing or invalid'"
    SecRule ARGS_NAMES "!@contains csrf_token"
  ```
   
  ### Een log entry (of een screenshot hiervan) van de WAF waaruit duidelijd blijkt dat er een aanval werd tegengehouden;

  ![alt text](csrf_attack-blocked.png)

  ### Een uitleg waarom de WAF-regel al dan niet werkt;

  Het wachtwoord dat we probeerden te wijzigen was "gehackt123" en deze wordt geblokkeerd.

# 5. GeoIP blocking (buiten België)
  ### De input die je gebruikt om de WAF-regel te testen;

  ```bash (van buiten België)
    curl -v http://example.com
  ```

  ### De WAF-regel die de aanval zou moeten blokkeren;

  ```bash
    SecRule GEO:COUNTRY_CODE "!@streq BE" "id:1004,phase:1,deny,status:403,msg:'Non-BE IP blocked'"
  ```

  ### Een log entry (of een screenshot hiervan) van de WAF waaruit duidelijk blijkt dat er een aanval werd tegengehouden;

  ![non-be_ip_blocked](non_be_ip_blocked.png)

  ### Een uitleg waarom de WAF-regel al dan niet werkt;

  We hebben een VPS in Brazilie gebruikt dus de request werd geblokkeerd.

# 5. Blokkeren van ‘repeat offenders’
  ### De input die je gebruikt om de WAF-regel te testen;

  ```bash
    curl -v http://example.com
  ``` 

  ### De WAF-regel die de aanval zou moeten blokkeren;

  ```bash
    SecRule IP:BLOCKED "@eq 1" "id:1005,phase:1,deny,status:403,msg:'Repeat offender blocked'"
    SecAction "id:1006,phase:5,deprecatevar:ip.blocked=1"
  ```

  ### Een log entry (of een screenshot hiervan) van de WAF waaruit duidelijk blijkt dat er een aanval werd tegengehouden;
  ### Een uitleg waarom de WAF-regel al dan niet werkt;
