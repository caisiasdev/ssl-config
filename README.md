# SECURING LOCAL WEB SERVER/API USING SELF SIGNED SSL CERTIFICATE 

Open Command Prompt and Type `openssl version` to check whether openssl is install in your system or not. You can download OpenSSL from https://sourceforge.net/projects/openssl-for-windows/

### 1. Generete Private Key

Create new folder and name it however you like. Open command prompt as Administration and run the following command

```openssl genrsa -out selfsigned.key 2048```

_This command will generate a new Private Key_

### 2. Create a Config file for Open SSL

Open notepad and paste the following code and save it as `openssl.cnf` in the same folder. 

```text
    [req]
    default_bits = 2048
    default_md = sha256
    distinguished_name = req_distinguished_name
    req_extensions = v3_req
    prompt = no

    [req_distinguished_name]
    C  = IN
    ST = Mizoram
    L  = Aizawl
    O  = Example Organisation
    CN = Example CN

    [v3_req]
    keyUsage = critical, digitalSignature, keyEncipherment
    extendedKeyUsage = serverAuth
    subjectAltName = @alt_names

    [alt_names]
    DNS.1 = localhost
    IP.1  = 100.64.0.9 //IP Address of Web-server
    IP.2  = 100.64.0.10 //IP Address of Another Web-server
    IP.3  = 127.0.0.1
```

### 3. Create a SSL Certificate

Your folder should contain two files.
```text
    selfsigned.key
    openssl.cnf
```
Now run the following command in the Command Prompt
```text
openssl req -new -x509 -days 365 -key selfsigned.key -out selfsigned.crt -config openssl.cnf -extensions v3_req
```
_This will create new SSL Certificate with 1 Year Validity. Here name of the file is `selfsigned.crt`_

### 4. Checking the Certificate and Convert .CRT to .CER

Now, your folder should contain three files.
```text
    selfsigned.crt
    selfsigned.key
    openssl.cnf
```
Now run the following command in the Command Prompt to check the Certificate `Subject Alternative Name`
```text
openssl x509 -in selfsigned.crt -noout -text | findstr /C:"DNS"
```
This result should looks similar to the following
```text
DNS:localhost, IP Address:100.64.0.9, IP Address:100.64.0.10, IP Address:100.64.0.11, IP Address:127.0.0.1
```
New run the following command to Convert .CRT file to .CER file
```text
openssl x509 -in selfsigned.crt -out selfsigned.cer -outform DER
```
_New .CER file will be generated_

Now, your folder should contain four files.
```text
    selfsigned.cer
    selfsigned.crt
    selfsigned.key
    openssl.cnf
```
### 5. Web Server - SSL configuration (NGINX)

Upload `selfsigned.key` and `selfsigned.crt` to your web-server. You can save this 2 files to any location.

_In this example the files are saved at `/home/ssl/` directory_

Now open nginx defaulf sites configuration file and add the following code.

_In this example the web-server is running on IP 100.64.0.9_
```text
server {
    listen 443 ssl;
    server_name 100.64.0.9; #Web server IP address
    ssl_certificate /home/ssl/selfsigned.crt;
    ssl_certificate_key /home/ssl/selfsigned.key;
    location / {
        root /var/www/html;
        index index.html;
    }
}
```
Save the file and restart the nginx service
```text
sudo systemctl restart nginx
```

### 6. SSL Installation on User Computer (Windows)
_Create a folder on User Computer, name it **ssl** or give any name. Copy the file ` selfsigned.cer ` from **STEP 4** to this folder._

Open Notepad, Paste the following code and Save it as ` installcert.bat ` inside this folder.
```text
@echo off
SET "CERT_NAME=selfsigned.cer"
SET "CERT_PATH=%~dp0%CERT_NAME%"
echo Adding the certificate to Trusted Root Certification Authorities...
certutil -addstore "Root" "%CERT_PATH%"
echo Certificate added successfully.
pause
```
_Run the `installcert.bat` as an Administrator. (i.e. Right click on the file and select **Run as Administrator**)_

_Browse to https://100.64.0.9_

