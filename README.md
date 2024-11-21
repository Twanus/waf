# TEAM COOKERS: Antoon Vereecken, Wesley Van Hoof, Monowal Shrestha, Stanley Okomhen

### Verslag: Opzetten van een Web Application Firewall (WAF) met ModSecurity en Nginx

#### Inleiding
Een Web Application Firewall (WAF) beschermt webapplicaties door HTTP-verzoeken te monitoren en te filteren. In deze setup gebruiken we ModSecurity als WAF in combinatie met Nginx als reverse proxy. Dit verslag beschrijft de configuratie en implementatie van deze setup.

#### Benodigde Componenten
- **Nginx**: Fungeert als reverse proxy server.
- **ModSecurity**: Een open-source WAF die HTTP-verkeer inspecteert en filtert.
- **OWASP Core Rule Set (CRS)**: Een set van regels die veelvoorkomende aanvallen detecteren en blokkeren.

#### Configuratieoverzicht

1. **Docker Compose Setup**:
   - De `docker-compose.yml` file definieert de services en netwerken. De `reverseproxy` service gebruikt het `owasp/modsecurity-crs:nginx-alpine` image om Nginx en ModSecurity te draaien.
   - Volumes worden gemount voor configuratiebestanden en logbestanden.

2. **Nginx Configuratie**:
   - De `default.conf` file configureert de Nginx server om te luisteren op poort 80 en definieert proxy-instellingen voor de `juice-shop` applicatie.
   - Beveiligingsheaders worden toegevoegd om de beveiliging van de applicatie te verbeteren.

3. **ModSecurity Configuratie**:
   - De `modsec-override.conf` file overschrijft de standaard ModSecurity instellingen. Het specificeert de locaties voor audit- en debuglogs en stelt het logformaat in op JSON.

#### Logbestanden
- **Audit Log**: Bevat gedetailleerde informatie over HTTP-verzoeken die door ModSecurity zijn verwerkt.
- **Debug Log**: Helpt bij het oplossen van problemen door gedetailleerde foutopsporingsinformatie te bieden.

#### Veelvoorkomende Problemen en Oplossingen
- **403 Fouten**: Deze worden vaak veroorzaakt door regels in de OWASP CRS die legitieme verzoeken blokkeren. Het is belangrijk om de regels te evalueren en aan te passen aan de specifieke behoeften van de applicatie.
- **Timeouts en Verbinding geweigerd**: Deze kunnen optreden als de upstream server niet bereikbaar is of als er netwerkproblemen zijn. Controleer de verbindingen en zorg ervoor dat de `juice-shop` service correct draait.

#### Conclusie
Het opzetten van een WAF met ModSecurity en Nginx biedt een robuuste beveiligingslaag voor webapplicaties. Door gebruik te maken van de OWASP CRS kunnen veelvoorkomende aanvallen effectief worden gedetecteerd en geblokkeerd. Het is echter cruciaal om de configuratie regelmatig te evalueren en aan te passen aan veranderende bedreigingen en applicatiebehoeften.
