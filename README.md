# Connected-HTB

Port 80 shows this website:
<img width="1919" height="800" alt="image" src="https://github.com/user-attachments/assets/f6b63442-4ec9-4ef1-a13a-4ab5dd61ad5f" />

At the bottom of the page we can see that there is a version number for Free at the bottom of the page, so let's try to find some CVE's for it:
<img width="832" height="159" alt="image" src="https://github.com/user-attachments/assets/02504d48-c947-4d99-b192-9fbd59d9dbf0" />

If we google it we can find a PoC for it: https://github.com/b4sh2/CVE-2025-57819-poc download it.

Set up the listener:
<img width="467" height="161" alt="image" src="https://github.com/user-attachments/assets/a0a9ace6-120d-41c2-b199-b97e1d4559e5" />

Execute the PoC:
<img width="764" height="247" alt="image" src="https://github.com/user-attachments/assets/fd3216e0-8d8b-4898-a30e-4f4b370248e6" />

And get shell:
<img width="778" height="124" alt="image" src="https://github.com/user-attachments/assets/df9080a4-1d0d-4a0b-86b4-37ada2e70eb0" />







