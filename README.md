# IRETOMIWA-CYBERSEC-JOURNEY-FOURTH-WEEK-CAPSTONE-PROJECT
CONFIDENTIAL PENTEST OF MEDIROZAHOSPITAL.COM
PENETRATION TESTING REPORT
Mediroza General Hospital
Web Application Security Assessment
Prepared by: OWOLABI IRETOMIWA
Cybersecurity Mentor: Waqas Karim, CCIE
Organisation: Networkwalks
Batch: B083 | Week 4 Capstone Project
Target: https://medirozahospital.com
Classification: Confidential
1. Executive Summary
I was assigned to conduct a black-box penetration test on the web infrastructure of Mediroza General Hospital at https://medirozahospital.com. The client provided written authorisation for this assessment. The goal was to identify vulnerabilities, demonstrate their real-world impact through exploitation, and provide recommendations to improve the security posture of the organisation.
During the assessment I identified seven vulnerabilities ranging from Medium to Critical severity. The most significant finding was a SQL injection vulnerability on the patient portal login page, which allowed me to bypass authentication entirely without knowing any credentials. This gave me access to three confidential patient lab report PDFs. After cracking the PDF encryption using password cracking tools, I discovered sensitive metadata inside one of the files that pointed to a forgotten database backup stored in a publicly accessible folder on the server. This backup contained the monthly salaries of all 30 hospital employees and the shareholding details of 10 shareholders.
The overall security posture of the target is poor. Multiple critical vulnerabilities exist that would allow an unauthenticated attacker to access confidential patient data, staff financial records, and corporate ownership information with minimal effort and no specialised equipment.
Overall Risk Rating: CRITICAL — Immediate remediation is recommended.
2. Scope and Methodology
2.1 Scope
The assessment was limited to the following target as agreed with the client.
Target domain: https://medirozahospital.com
The following were excluded from scope: social engineering, denial of service attacks, and any testing outside the agreed domain.
2.2 Methodology
I followed a structured black-box penetration testing methodology consisting of four phases.
• Reconnaissance: Passive information gathering using publicly available information and web-based tools.
• Vulnerability Identification: Analysing the application behaviour to find weaknesses in authentication and input handling.
• Exploitation: Demonstrating the real impact of each vulnerability through controlled exploitation.
• Documentation: Recording all findings, evidence, and remediation recommendations in this report.
2.3 Tools Used
• curl — command line tool for sending HTTP requests and reading web server responses
• Browser developer tools — for inspecting page source and login form behaviour
• Networkwalks Hash Calculator — for extracting password hashes from PDF files
• Networkwalks Password Cracker — for cracking PDF password hashes using wordlists
• qpdf — for decrypting password-protected PDF files after cracking
• exiftool — for reading hidden metadata from PDF files
• wget — for downloading files from the web server
• ChatGPT — for converting raw SQL data into readable tables<img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h13m59s794" src="https://github.com/user-attachments/assets/e3942916-76e8-4a6c-9a3b-f3659dab5796" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h14m38s255" src="https://github.com/user-attachments/assets/5a2c87c4-70ed-4b81-8019-cda959e018b1" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h14m53s117" src="https://github.com/user-attachments/assets/f7e1d8fc-a06a-409c-931a-750ecc133789" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h15m17s267" src="https://github.com/user-attachments/assets/1bf4957f-e2b7-49be-a31a-b1737e0ce3c7" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h15m28s822" src="https://github.com/user-attachments/assets/720aa7a9-44a6-4a4f-8a36-6a4b234b96ec" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h15m28s822" src="https://github.com/user-attachments/assets/85d4c886-9572-411c-b112-f3ef81d84ded" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h16m00s269" src="https://github.com/user-attachments/assets/3f26b68b-9c6b-44cc-bf8c-ce137f5632cc" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h16m16s765" src="https://github.com/user-attachments/assets/664c563b-c7dd-48bd-9675-dee6f1339276" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h16m32s065" src="https://github.com/user-attachments/assets/e68b26e9-172b-435d-88fe-ed30c6285af1" />
3. Findings and Proof of Exploitation
3.1 Summary Table
# 	Vulnerability 	Location 	Risk
1 	Username enumeration on login page 	patient/login.php 	Medium
2 	SQL injection login bypass 	patient/login.php 	Critical
3 	Encrypted PDFs accessible after login bypass 	patient/reports/ 	High
4 	Weak PDF passwords crackable with a wordlist 	patient_report_*.pdf 	High
5 	Sensitive metadata left in patient PDF files 	patient_report_3.pdf 	Medium
6 	Forgotten backup folder with directory listing enabled 	old/ 	Critical
7 	Confidential staff salaries and shareholder data in plain text 	old/mediroza_db_backup_2019.sql 	Critical
<img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h17m51s480" src="https://github.com/user-attachments/assets/41955ff8-fe88-45ed-ba32-c90d1961ad87" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h18m10s320" src="https://github.com/user-attachments/assets/9140ed49-15f0-4ef5-9ecc-179dba946895" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h18m20s885" src="https://github.com/user-attachments/assets/3ebe4b2b-a433-4685-9d71-dc417ae77a98" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h18m54s712" src="https://github.com/user-attachments/assets/45579c4c-6d04-49a5-ad40-76ebef2f9e63" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h19m17s247" src="https://github.com/user-attachments/assets/331038d7-1edd-47c2-b5bf-c24b2498b1f1" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h16m00s269" src="https://github.com/user-attachments/assets/f6bb0fca-0f84-4da9-a9c3-e46474da331e" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h16m16s765" src="https://github.com/user-attachments/assets/86df1226-9fd0-4a6f-8efa-45cfea230f53" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h16m32s065" src="https://github.com/user-attachments/assets/f7e2121b-0d67-474e-b034-0eb38748cc62" />
3.2 Finding 1 — Username Enumeration
Risk Rating: Medium
Location: patient/login.php
Description
Username enumeration occurs when a login page reveals whether a username exists or not by showing different error messages for a wrong username versus a wrong password. A secure login should always show the same generic message for both cases to prevent an attacker from confirming valid usernames.
Steps Taken
I opened the patient portal login page and tested the error messages by entering different inputs.
First I entered a username that I assumed would not exist.
username: bob password: test123
RESPONSE
Username not found
Then I tried a common default username.
username: admin password: test123
RESPONSE
Incorrect password
The two different messages confirmed that admin is a valid account on the system. I now had the username and only needed to find the password.
[ Screenshot: Screenshot: Browser showing "Username not found" for bob ]
[ Screenshot: Screenshot: Browser showing "Incorrect password" for admin ]<img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h17m51s480" src="https://github.com/user-attachments/assets/f37a0b38-d1da-472c-a49c-931f576ba11b" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h18m10s320" src="https://github.com/user-attachments/assets/0ceb20ef-dcdb-4111-8a70-95e20d2baafa" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h18m20s885" src="https://github.com/user-attachments/assets/2056e72c-29e7-4263-b77e-763f53451ef2" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h18m54s712" src="https://github.com/user-attachments/assets/ca2c1cb6-b058-4d37-90cb-be68af2bbcb2" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h19m17s247" src="https://github.com/user-attachments/assets/ccea6c5c-a231-4961-928c-4be663a9b503" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h20m05s589" src="https://github.com/user-attachments/assets/d1bf9c46-fbbc-4af5-a459-fd8331c5ebee" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h20m35s007" src="https://github.com/user-attachments/assets/ba2f9d9a-14ff-406b-9f2f-e32266072466" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h21m54s683" src="https://github.com/user-attachments/assets/2836f313-de2a-48cf-bea7-ff49f0dc5f41" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h22m32s749" src="https://github.com/user-attachments/assets/b0e831cc-f8e6-490e-8810-98bbf16cdc3c" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h22m48s983" src="https://github.com/user-attachments/assets/833df97d-bbd3-4511-af5a-06c1869efa52" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h24m07s826" src="https://github.com/user-attachments/assets/10e73005-edd9-452b-bf48-a01551fc5462" />
3.3 Finding 2 — SQL Injection Login Bypass
Risk Rating: Critical
Location: patient/login.php
Description
SQL injection is a vulnerability where an application places user input directly inside a database query without sanitising it first. An attacker can insert SQL code into the input field to change the behaviour of the query. In this case, I was able to comment out the password check entirely and log in as admin without knowing the password.
Steps Taken
I tested the username field for SQL injection by typing a single quote.
username: admin' password: test123
RESPONSE
Warning: mysqli_query(): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''' at line 1
The database error confirmed the field was injectable. The application was building its SQL query like this behind the scenes.
SELECT * FROM users WHERE username='admin'' AND password='test123'
The extra quote broke the query and caused the error. I then crafted the classic SQL injection bypass payload.
username: admin' -- password: anything
The -- comments out everything after it in SQL, so the query becomes.
SELECT * FROM users WHERE username='admin'
The password check is ignored completely. The query returns the admin row and I am logged in.
[ Screenshot: Screenshot: Database error when single quote is entered ]
[ Screenshot: Screenshot: Logged in portal page after SQL injection ]<img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h24m20s835" src="https://github.com/user-attachments/assets/de233dd3-0c91-4bb5-b3d5-dfaaa2bc3b51" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h24m55s636" src="https://github.com/user-attachments/assets/575a1415-347d-435a-bb86-11f5457b0a5d" />
3.4 Finding 3 — Confidential PDFs Accessible After Login Bypass
Risk Rating: High
Location: patient/reports/
Description
After gaining unauthorised access to the portal through SQL injection, I found three patient lab report PDFs available for download. These are confidential medical documents that should only be accessible to the named patients and their doctors.
Steps Taken
After logging in with the SQL injection payload I was directed to the patient portal which listed three downloadable PDF files.
patient_report_1.pdf patient_report_2.pdf patient_report_3.pdf
I downloaded all three files.
[ Screenshot: Screenshot: Portal page showing the 3 downloadable reports ]
[ Screenshot: Screenshot: Downloaded files in the Downloads folder ]<img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h25m20s037" src="https://github.com/user-attachments/assets/91b71c0e-2380-4fb4-ad87-15f64a126f8c" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h26m39s550" src="https://github.com/user-attachments/assets/f93beecb-66f1-41f3-8518-59e9c19b4dc8" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h26m50s287" src="https://github.com/user-attachments/assets/2c1695e7-b42f-4405-89df-090503ea2005" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h27m27s839" src="https://github.com/user-attachments/assets/4b786f9e-3898-481e-ab06-d58975dd02b1" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h27m40s056" src="https://github.com/user-attachments/assets/e8c1ed18-ff7f-46e7-b57a-3a84524dbbdc" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h28m35s667" src="https://github.com/user-attachments/assets/c5f65d6c-a7cb-4c7b-85f2-30f006f696ce" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h28m51s155" src="https://github.com/user-attachments/assets/c65bc47d-97de-4f01-943a-0410711c6067" />
3.5 Finding 4 — Weak PDF Passwords Crackable with a Wordlist
Risk Rating: High
Description
All three PDFs were password protected. However the passwords were weak and appeared in commonly available password wordlists, making them trivial to crack using automated tools. Using simple passwords on sensitive medical documents does not provide meaningful protection.
Steps Taken
I used the Networkwalks Hash Calculator to extract a crackable hash from each PDF, then ran each hash through the Networkwalks Password Cracker.
Reports 1 and 2 cracked immediately using the built-in 100 word default wordlist.
RESULTS
patient_report_1.pdf → 123456
patient_report_2.pdf → password
Report 3 did not crack with the built-in list. I switched to a larger wordlist (JTR default password list) and ran the attack again.
patient_report_3.pdf → !@#$%^&
I then opened each PDF with its cracked password and read the confidential patient medical data inside.
[ Screenshot: Screenshot: Networkwalks Password Cracker showing cracked password for report 1 ]
[ Screenshot: Screenshot: Networkwalks Password Cracker showing cracked password for report 3 with JTR list ]
[ Screenshot: Screenshot: Opened PDF showing patient medical data ]<img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h29m10s946" src="https://github.com/user-attachments/assets/7ec6140a-d6d8-4431-9fd5-a2cbae2e27c0" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h29m21s958" src="https://github.com/user-attachments/assets/a7fb51fb-cafb-4e6e-b662-8676649d2ebe" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h29m47s974" src="https://github.com/user-attachments/assets/cac3f837-adeb-4a9c-9b2d-7c16bbe95829" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h30m02s459" src="https://github.com/user-attachments/assets/8cd6c040-accf-47f4-be5a-6934fa5b97e2" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h30m20s534" src="https://github.com/user-attachments/assets/1ed311e2-3a37-403a-b9ef-f9f53b966430" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h30m34s284" src="https://github.com/user-attachments/assets/088282f3-9e79-4968-8660-cb98555c5f94" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h31m49s920" src="https://github.com/user-attachments/assets/2b2048df-3f14-4b33-ac57-067e7f68a7f4" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h32m25s954" src="https://github.com/user-attachments/assets/cd27908b-741f-4cd2-b8de-995a7d6af2c9" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h32m52s528" src="https://github.com/user-attachments/assets/3b4a4cf7-cd91-4504-a396-268da754e545" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h32m52s528" src="https://github.com/user-attachments/assets/88215118-f234-46f5-be64-b9762140b277" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h33m07s411" src="https://github.com/user-attachments/assets/71f9b244-0f10-4955-8aa0-69ed2d35e939" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h33m32s375" src="https://github.com/user-attachments/assets/c2621584-ebfc-41b4-8aaa-de64783e7576" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h33m43s056" src="https://github.com/user-attachments/assets/779771a1-63ca-4641-ade2-a3f96462dc00" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h34m15s398" src="https://github.com/user-attachments/assets/c4e483fe-060c-4a89-813e-a5b139e38711" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h34m15s398" src="https://github.com/user-attachments/assets/8e00d7f1-1ede-4509-ae5b-8a406d6e94c7" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h34m34s629" src="https://github.com/user-attachments/assets/8a25ffb4-23dd-4986-96bf-ba9f7d6c4d40" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h34m46s859" src="https://github.com/user-attachments/assets/b1e18891-0ed9-468d-83be-6d33c1508756" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h35m13s499" src="https://github.com/user-attachments/assets/dca010df-401b-477f-a946-a61a5cca82d3" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h35m35s629" src="https://github.com/user-attachments/assets/81c16efd-6f81-4b0e-bfba-26195eb385e1" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h35m57s820" src="https://github.com/user-attachments/assets/565be6ea-6bf5-44cf-814d-d19f4b0df347" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h35m57s820" src="https://github.com/user-attachments/assets/17d29b45-770a-48a2-aeda-fac4985e7ee7" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h36m07s080" src="https://github.com/user-attachments/assets/f33e4d54-5159-44e1-b03f-060bcaa0399e" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h36m19s089" src="https://github.com/user-attachments/assets/26120b6f-8bc6-4807-9fb3-9294f7411494" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h36m33s568" src="https://github.com/user-attachments/assets/d263459e-5506-4d28-ba5a-721271639ea1" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h37m15s299" src="https://github.com/user-attachments/assets/75102d02-dddb-4553-b5a9-9dc9abce8a30" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h37m29s272" src="https://github.com/user-attachments/assets/46b3f4b0-f018-4022-a4b7-764ce3f22272" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h37m29s272" src="https://github.com/user-attachments/assets/66b0281d-f3eb-484c-b6b9-22b9fc970727" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h37m40s262" src="https://github.com/user-attachments/assets/b6b17517-30f4-479f-83e2-43f6fb1bde08" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h44m37s540" src="https://github.com/user-attachments/assets/a0e3f50a-f0c3-4862-ab08-b6862f683813" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h44m53s489" src="https://github.com/user-attachments/assets/c4dbaa5e-4f47-4d14-90ec-b25fea8552ef" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h45m09s287" src="https://github.com/user-attachments/assets/43d4d249-5e97-4f1d-9f05-e9919e39ec4d" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h45m36s923" src="https://github.com/user-attachments/assets/54055552-6603-44d6-a5da-03422b449859" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h45m36s923" src="https://github.com/user-attachments/assets/e0dbf145-b0da-48f7-9244-b18c06b2088e" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h45m47s682" src="https://github.com/user-attachments/assets/a98bb847-5325-4efc-8cbe-08cb352d1299" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-13h47m00s607" src="https://github.com/user-attachments/assets/08eb8d5b-d237-4daa-8706-f093a3173875" />
3.6 Finding 5 — Sensitive Metadata in Patient PDF Files
Risk Rating: Medium
Description
PDF files store hidden metadata such as the author name, creation date, and custom comments. This metadata is not visible when simply reading the document but can be extracted using tools like exiftool. In this case, report 3 contained an internal staff comment left by the IT administrator that pointed directly to a confidential backup file on the server.
Steps Taken
I first decrypted report 3 using qpdf so that exiftool could read all metadata fields. A locked PDF only shows the encryption type.
qpdf --password='!@#$%^&' --decrypt patient_report_3.pdf report3_open.pdf
Then I ran exiftool on the unlocked file.
exiftool report3_open.pdf
KEY FIELDS IN THE OUTPUT
Author : j.malik
Comments : DB backup moved to /old before site migration, do not delete
This comment was left by the IT administrator Jameel Malik (j.malik) and revealed the exact location of a database backup on the server.
[ Screenshot: Screenshot: exiftool output showing Author j.malik and Comments field ]<img width="3840" height="2400" alt="vlcsnap-2026-09-29-14h20m00s912" src="https://github.com/user-attachments/assets/d571f033-8d52-465d-b0e3-faa5c5e7960d" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-14h20m16s332" src="https://github.com/user-attachments/assets/3d664dee-97fa-477f-9b91-194855741df3" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-14h20m33s272" src="https://github.com/user-attachments/assets/46ee6d9e-192f-42d2-a2c3-3363ab4710a1" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-14h20m43s652" src="https://github.com/user-attachments/assets/0c414940-9f68-4734-ad0c-17bbe4affb63" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-14h20m43s652" src="https://github.com/user-attachments/assets/97101dcf-5760-45f9-aacd-f5ed0803e0ca" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-14h20m54s538" src="https://github.com/user-attachments/assets/627b9c78-4b82-4789-bf6c-8c34e9745009" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-14h21m09s994" src="https://github.com/user-attachments/assets/703837e1-30ad-4886-83d8-bd505b0563e0" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-14h21m52s262" src="https://github.com/user-attachments/assets/cfd46361-3652-4f0d-b673-c98d12c71b69" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-14h26m07s907" src="https://github.com/user-attachments/assets/7408fa3b-c9e7-4544-a8c1-ca2671fe5838" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-14h26m33s423" src="https://github.com/user-attachments/assets/f2e9c83e-8b75-486f-9143-bd2a04dad9d2" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-14h26m55s669" src="https://github.com/user-attachments/assets/33fa0441-232e-4d68-9434-7f469aa89cee" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-14h27m06s173" src="https://github.com/user-attachments/assets/fbcd238f-ca18-47cc-a7b3-6cd8766c0669" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-14h30m09s360" src="https://github.com/user-attachments/assets/646b1075-a527-42f1-91ca-fac60ba596a0" /><img width="3840" height="2400" alt="vlcsnap-2026-09-29-14h30m27s053" src="https://github.com/user-attachments/assets/e1323221-9572-4d52-a608-664cc8257058" />
3.7 Finding 6 — Forgotten Backup Folder with Directory Listing Enabled
Risk Rating: Critical
Description
Directory listing is a web server misconfiguration that displays the contents of a folder like a file browser when no index page exists. The /old folder on the server had directory listing enabled, exposing a database backup file to anyone who visited that URL. I had already found this folder during my initial recon from the robots.txt file, which listed it as a disallowed path.
Steps Taken
During recon I noted /old in the robots file. The metadata clue from Finding 5 confirmed this was the location of the backup. I opened the folder in the browser.
https://medirozahospital.com/old/
The directory listing was enabled and I could see the backup file immediately.
mediroza_db_backup_2019.sql
I downloaded it using wget.
wget https://medirozahospital.com/old/mediroza_db_backup_2019.sql
[ Screenshot: Screenshot: Browser showing /old/ directory listing with the SQL file visible ]<img width="3840" height="2400" alt="vlcsnap-2026-09-29-14h38m00s280" src="https://github.com/user-attachments/assets/3950b261-4938-4553-97d5-d8050bff1563" />
3.8 Finding 7 — Confidential Staff Salaries and Shareholder Data in Plain Text
Risk Rating: Critical
Description
The database backup contained highly sensitive information in plain text including the full name, job title, email address, phone number, national ID number, and monthly salary in ZAR for all 30 hospital employees. It also contained the names and shareholding percentages of all 10 shareholders of the hospital. This data was completely unprotected and accessible to anyone who visited the /old folder.
Steps Taken
I opened the SQL file in a text editor. The data was in raw SQL format. I copied the staff INSERT INTO rows and used ChatGPT to convert them into a readable table.
Present this SQL data as a readable table showing name, job title, department and monthly salary.
Then I did the same for the shareholders.
Present this SQL data as a readable table showing shareholder name, share percentage and share class.
I was able to read the private salary of every employee and the ownership structure of the entire hospital.
I also confirmed that the Author field in the PDF metadata, j.malik, corresponds to Jameel Malik, the IT Systems Administrator listed in the staff table. This connects all three milestones in a single attack chain.

4. Full Attack Chain Summary
The following is the complete path from zero access to full data exposure, showing how each finding led to the next.
• Step 1: Robots.txt revealed three hidden folders: /patient, /staff, and /old.
• Step 2: The patient login page showed different error messages for wrong username and wrong password, confirming admin as a valid account. (Finding 1)
• Step 3: A single quote in the username field triggered a database error, confirming SQL injection. (Finding 2)
• Step 4: The payload admin' -- bypassed the login entirely and gave access to the patient portal. (Finding 2)
• Step 5: Three confidential patient PDF lab reports were downloaded from the portal. (Finding 3)
• Step 6: The Networkwalks Password Cracker cracked the PDF passwords using wordlists. Reports 1 and 2 used the built-in list. Report 3 required the larger JTR list. (Finding 4)
• Step 7: exiftool on the unlocked report 3 revealed a hidden comment by j.malik pointing to /old. (Finding 5)
• Step 8: The /old folder had directory listing enabled, exposing the database backup. (Finding 6)
• Step 9: The backup contained salaries of 30 employees and shareholding details of 10 shareholders in plain text. (Finding 7)
5. Recommendations and Remediation
5.1 Fix Username Enumeration
Change the login page to show a single generic message for any failed login attempt, regardless of whether the username or password was wrong. For example: "Invalid credentials. Please try again."
5.2 Fix SQL Injection
Replace the current login query with a parameterised query or prepared statement. This separates the SQL code from the user input so that no amount of crafted input can change the query structure. This is the most important fix in this report.
// Safe example using PHP PDO prepared statement $stmt = $pdo->prepare("SELECT * FROM users WHERE username = ? AND password = ?") $stmt->execute([$username, $password])
5.3 Fix PDF Access and Password Strength
Move the PDF files outside the web root so they cannot be served directly by the web server. Enforce access through a server-side script that checks authentication before serving any file. If passwords are used, enforce a minimum length of 12 characters with a mix of uppercase, lowercase, numbers, and symbols.
5.4 Strip PDF Metadata
Remove all metadata from files before distributing them. Staff notes or internal comments must never be stored inside files that leave the organisation. Use the following command to strip all metadata.
exiftool -all= patient_report_3.pdf
5.5 Fix Directory Listing and Remove the Backup
Disable directory listing on all folders by adding Options -Indexes to the Apache configuration or .htaccess file. Delete the database backup from the /old folder immediately. Database backups must never be stored inside the web root. Store them in a private, access-controlled location outside the publicly accessible part of the server.
6. Conclusion
This assessment found a complete attack chain from the login page to highly sensitive internal data. No sophisticated tools or knowledge were required. All vulnerabilities found in this report are well-known and have well-established fixes. I recommend the client address all Critical and High findings immediately before the system is used to store or serve real patient data.
Submitted by: OWOLABI IRETOMIWA
Cybersecurity Mentor: Waqas Karim, CCIE
Organisation: Networkwalks
Batch B083 | Week 4 Capstone Project
This report is produced as part of a controlled educational exercise by Networkwalks. The target has been authorised for security testing. These techniques must never be applied to any system without explicit written permission from the owner.
This report is submitted as part of the Networkwalks B082 Cybersecurity Internship Week 4 Capstone Project. All testing was conducted in a controlled environment with written authorisation from the client. These techniques must never be applied to any system without explicit written permission from the owner.
