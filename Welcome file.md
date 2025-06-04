---


---

<p>2025-06-01 09:13</p>
<p>Status:</p>
<p>Tags: [[Pentesting]] [[WriteUp]]</p>
<h1 id="ballpointpentest">BallpointPentest</h1>
<h1 id="ai-innovations---initial-pentest-report">AI Innovations - Initial Pentest Report</h1>
<p><strong>Target IP:</strong> <code>146.59.219.77</code><br>
<strong>Scan Date:</strong> 2025-05-30<br>
<strong>Pentester:</strong> [Tao Mills]<br>
<strong>Assessment Type:</strong> Black-box Web-focused Enumeration</p>
<hr>
<h2 id="nmap-scan-results">Nmap Scan Results</h2>
<pre class=" language-bash"><code class="prism  language-bash">nmap -sCV -p- 146.59.219.77
</code></pre>
<pre><code>PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 9.7p1 Ubuntu 7ubuntu4.3 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http       Caddy httpd
443/tcp  open  ssl/http   Gunicorn
5432/tcp open  postgresql PostgreSQL DB 9.6.0 or later
</code></pre>
<p><strong>Service Info:</strong> Linux system with common services running. Web services appear to be hosted on both ports 80 (Caddy) and 443 (Gunicorn). PostgreSQL is exposed on port 5432.</p>
<hr>
<h2 id="web-enumeration-port-80---http">Web Enumeration (Port 80 - HTTP)</h2>
<h3 id="http-header-analysis">HTTP Header Analysis</h3>
<pre class=" language-bash"><code class="prism  language-bash">curl -I http://146.59.219.77
</code></pre>
<pre><code>HTTP/1.1 200 OK
Server: Caddy
Content-Type: text/html; charset=utf-8
Content-Length: 302
</code></pre>
<ul>
<li>The server responds with a static page.</li>
<li>No redirects or cookies.</li>
<li>Possible minimal landing page.</li>
</ul>
<hr>
<h2 id="web-enumeration-port-443---https--gunicorn">Web Enumeration (Port 443 - HTTPS / Gunicorn)</h2>
<p>Gunicorn is commonly used to host Python-based applications such as Flask or Django.</p>
<h3 id="angularjs-detected">AngularJS Detected</h3>
<ul>
<li>The application uses <strong>AngularJS v1.3.0</strong> (legacy, known to be vulnerable to sandbox escapes if used improperly)</li>
<li>File found: <code>/static/js/angular.min.js</code></li>
</ul>
<h3 id="javascript-insights">JavaScript Insights</h3>
<ul>
<li><code>/static/js/query.js</code> contains logic for validating invitation codes.</li>
<li>Possible user code injection vector via <code>AI-XXXX-YYYY-ZZZZ</code> pattern. Will test for client-side trust or server reflection.</li>
</ul>
<hr>
<h2 id="directory-brute-forcing">Directory Brute-Forcing</h2>
<h3 id="port-80---http">Port 80 - HTTP</h3>
<pre class=" language-bash"><code class="prism  language-bash">ffuf -u http://146.59.219.77/FUZZ -w /usr/share/wordlists/dirb/common.txt -fc 404
</code></pre>
<h3 id="port-443---https">Port 443 - HTTPS</h3>
<pre class=" language-bash"><code class="prism  language-bash">ffuf -u https://146.59.219.77/FUZZ -w /usr/share/wordlists/dirb/common.txt -fc 404 -k
</code></pre>
<p>Fuzzing is in progress to identify:</p>
<ul>
<li>Hidden directories or files (e.g. <code>.env</code>, <code>backup.zip</code>, <code>api/</code>)</li>
<li>Admin panels or upload points</li>
<li>Development/debug endpoints</li>
</ul>
<hr>
<h2 id="web-vulnerability-scanning-tools-to-be-used">Web Vulnerability Scanning Tools to Be Used</h2>

<table>
<thead>
<tr>
<th>Tool</th>
<th>Functionality</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>OWASP ZAP</strong></td>
<td>Active/passive scans, XSS, SQLi, CSRF, headers</td>
</tr>
<tr>
<td><strong>Burp Suite</strong></td>
<td>Manual + scanner, parameter fuzzing, upload testing</td>
</tr>
<tr>
<td><strong>Nikto</strong></td>
<td>Outdated server versions, dangerous files</td>
</tr>
<tr>
<td><strong>Arachni</strong></td>
<td>XSS, SQLi, RFI, LFI, CSRF, unvalidated redirects</td>
</tr>
<tr>
<td><strong>Wapiti</strong></td>
<td>SQLi, XSS, SSRF, file disclosure</td>
</tr>
<tr>
<td><strong>Nuclei</strong></td>
<td>CVE matching, fast scanning with community templates</td>
</tr>
<tr>
<td><strong>Dirsearch</strong></td>
<td>Brute-forcing hidden routes and extensions</td>
</tr>
<tr>
<td><strong>SQLMap</strong></td>
<td>Automated SQLi discovery and exploitation</td>
</tr>
<tr>
<td><strong>Feroxbuster</strong></td>
<td>Fast content discovery and fuzzing</td>
</tr>
</tbody>
</table><p>These tools will be run in sequence or tandem with fuzzing results and login forms.</p>
<hr>
<h2 id="manual-testing-targets">Manual Testing Targets</h2>
<ul>
<li><strong>SQL Injection</strong> on invitation forms (<code>/auth</code>, <code>/signup</code>) with tools like Burp and SQLMap.</li>
<li><strong>XSS Injection</strong> attempts on any input forms or code entry fields.</li>
<li><strong>AngularJS sandbox escape</strong> if user-controlled input ends up in <code>ng-bind</code> or similar directives.</li>
<li><strong>Check for open debug routes</strong>, e.g., <code>/debug</code>, <code>/flask-debug</code>, <code>/admin</code>.</li>
</ul>
<hr>
<h2 id="postgresql-port-5432">PostgreSQL (Port 5432)</h2>
<ul>
<li>Version: 9.6 or later</li>
<li>Will attempt connection via <code>psql</code> if creds are obtained</li>
<li>Target for SQL injection pivoting</li>
<li>Possible local privilege escalation if compromised</li>
</ul>
<hr>
<h2 id="ssh-port-22">SSH (Port 22)</h2>
<ul>
<li>OpenSSH detected</li>
<li>No login attempts planned unless credentials are recovered via SQLi or misconfig</li>
</ul>
<hr>
<h2 id="next-planned-actions">Next Planned Actions</h2>
<ol>
<li>Complete full directory brute-force on ports 80/443</li>
<li>Use Burp/ZAP to test form behavior and inject payloads</li>
<li>Use SQLMap if parameter injection is detected</li>
<li>Try bypassing AngularJS protections</li>
<li>Launch Nuclei scan with CVE detection templates</li>
<li>Try PostgreSQL version fingerprinting with <code>nmap -sV --script=pgsql*</code></li>
</ol>
<hr>
<h2 id="references">References</h2>
<ul>
<li>OWASP Testing Guide</li>
<li>PayloadsAllTheThings</li>
<li>GTFOBins (in case local shell obtained)</li>
</ul>

