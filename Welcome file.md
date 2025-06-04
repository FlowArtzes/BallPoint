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
<h2 id="web-enumeration-port-80---http">Web Enumeration (Port 80 - HTTP)</h2>
<h3 id="http-header-analysis">HTTP Header Analysis</h3>
<pre class=" language-bash"><code class="prism  language-bash">curl -I http://146.59.219.77
</code></pre>
<pre class=" language-makefile"><code class="prism  language-makefile">HTTP/1.1 200 OK
<span class="token symbol">Server</span><span class="token punctuation">:</span> Caddy
<span class="token symbol">Content-Type</span><span class="token punctuation">:</span> text/html<span class="token punctuation">;</span> charset<span class="token operator">=</span>utf-8
<span class="token symbol">Content-Length</span><span class="token punctuation">:</span> 302

</code></pre>
<ul>
<li>
<p>The server responds with a static page.</p>
</li>
<li>
<p>No redirects or cookies.</p>
</li>
<li>
<p>Possible minimal landing page.</p>
</li>
</ul>
<h2 id="web-enumeration-port-443---https--gunicorn">Web Enumeration (Port 443 - HTTPS / Gunicorn)</h2>
<p>Gunicorn is commonly used to host Python-based applications such as Flask or Django.</p>
<h3 id="next-planned-actions">Next planned actions:</h3>
<ul>
<li>
<p>Attempt connecting to <code>https://146.59.219.77</code> using <code>curl</code> and browser.</p>
</li>
<li>
<p>Try fuzzing routes such as <code>/admin</code>, <code>/debug</code>, <code>/api</code>, and inspect for Flask/Werkzeug traces.</p>
</li>
<li>
<p>Investigate potential virtual host requirement (e.g. <code>Host: ai-innovations.htb</code>).</p>
</li>
</ul>
<h2 id="directory-brute-forcing">Directory Brute-Forcing</h2>
<h3 id="port-80---http">Port 80 - HTTP</h3>
<pre class=" language-bash"><code class="prism  language-bash">ffuf -u http://146.59.219.77/FUZZ -w /usr/share/wordlists/dirb/common.txt -fc 404
</code></pre>
<h3 id="port-443---https">Port 443 - HTTPS</h3>
<pre class=" language-bash"><code class="prism  language-bash">ffuf -u https://146.59.219.77/FUZZ -w /usr/share/wordlists/dirb/common.txt -fc 404 -k
</code></pre>
<p>Fuzzing is in progress to identify:</p>
<ul>
<li>
<p>Hidden directories or files (e.g. <code>.env</code>, <code>backup.zip</code>, <code>api/</code>)</p>
</li>
<li>
<p>Admin panels or upload points</p>
</li>
<li>
<p>Development/debug endpoints</p>
</li>
</ul>
<h2 id="postgresql-port-5432">PostgreSQL (Port 5432)</h2>
<p>Exposed PostgreSQL service:</p>
<ul>
<li>
<p>Version: 9.6 or later</p>
</li>
<li>
<p>No credentials discovered yet</p>
</li>
<li>
<p>Will attempt login if credentials are identified from web sources (<code>.env</code>, hardcoded creds, SQLi)</p>
</li>
</ul>
<h2 id="ssh-port-22">SSH (Port 22)</h2>
<p>OpenSSH detected:</p>
<ul>
<li>
<p>Not a target for brute-force at this stage</p>
</li>
<li>
<p>Will try credentials if found through other means (e.g., DB creds or web leakage)</p>
</li>
</ul>
<h2 id="references">References</h2>

