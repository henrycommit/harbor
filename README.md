# harbor



Harbor is an open source trusted cloud native registry project that stores, signs, and scans content. Harbor extends the open source Docker Distribution by adding the functionalities usually required by users such as security, identity and management. Having a registry closer to the build and run environment can improve the image transfer efficiency. Harbor supports replication of images between registries, and also offers advanced security features such as user management, access control and activity auditing.
# features
- Cloud native registry: With support for both container images and Helm charts, Harbor serves as registry for cloud native environments like  container runtimes and orchestration platforms.

- Role based access control: Users access different repositories through 'projects' and a user can have different permission for images or Helm charts under a project.

- Policy based replication: Images and charts can be replicated (synchronized) between multiple registry instances based on policies with using filters (repository, tag and label). Harbor automatically retries a replication if it encounters any errors. This can be used to assist loadbalancing, achieve high availability, and facilitate multi-datacenter deployments in hybrid and multi-cloud scenarios.

- Vulnerability Scanning: Harbor scans images regularly for vulnerabilities and has policy checks to prevent vulnerable images from being deployed.

- Image deletion & garbage collection: System admin can run garbage collection jobs so that images(dangling manifests and unreferenced blobs) can be deleted and their space can be freed up periodically.
Refered https://goharbor.io/docs/1.10/install-config/configure-https/
# Harbor installation prerequisites
| Software       | Version                       | 
| -------------  | ----------------------------- |
| Docker Engine  | Version 17.06.0-ce+ or higher | 
| Docker Compose | Version 1.18.0 or higher      |
| Openssl        |  Latest is preferred          |

# Install and configuration
Download online or offline version on _https://github.com/goharbor/harbor/releases_
+ Online installer: The online installer downloads the Harbor images from Docker hub. For this reason, the installer is very small in size.
+ Offline installer: Use the offline installer if the host to which are are deploying Harbor does not have a connection to the Internet. The offline installer contains pre-built images, so it is larger than the online installer.

**Extract package**:
tar -xvf harbor-offline-installer-version.tgz

**Configure HTTPS access to Harbor**
- Create a certificate directory: **mkdir /certs**
- Generate a Certificate Authority Certificate
1. Generate a CA certificate private key: **_openssl genrsa -out ca.key 4096_**
2. Generate the CA certificate: 
**_openssl req -x509 -new -nodes -sha512 -days 3650 \
-subj "/C=VN/ST=Hanoi/L=Hanoi/O=example/OU=Personal/CN=harbor.server.local" \
-key ca.key \
-out ca.crt_**

- Generate Server Certificate
1. Generate a private key: **_openssl genrsa -out harbor.server.local.key 4096_**
2. Generate a certificate signing request (CSR):
_**openssl req -sha512 -new \
-subj "/C=VN/ST=Hanoi/L=Hanoi/O=example/OU=Personal/CN=harbor.server.local" \
-key harbor.server.local.key \
-out harbor.server.local.csr**_

3. Generate an x509 v3 extension file:
cat > v3.ext <<-EOF \
authorityKeyIdentifier=keyid,issuer \
basicConstraints=CA:FALSE \
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment \
extendedKeyUsage = serverAuth \
subjectAltName = @alt_names
[alt_names]
DNS.1=harbor.server.local \
EOF

4. Use the v3.ext file to generate a certificate for your Harbor host: \
_**openssl x509 -req -sha512 -days 3650 \
-extfile v3.ext \
-CA ca.crt -CAkey ca.key -CAcreateserial \
-in harbor.server.local.csr \
-out harbor.server.local.crt**_
- Provide the Certificates to Docker
1. Convert harbor.server.local.crt to harbor.server.local.cert for use by Docker:\
_**openssl x509 -inform PEM -in harbor.server.local.crt -out harbor.server.local.cert**_
2. Create certs directory for Docker:\
 _**mkdir -p /etc/docker/certs.d/harbor.server.local** _
3. Copy the server certificate, key and CA files into the Docker certificates folder on the Harbor host: \
cp /certs/harbor.server.local.cert /etc/docker/certs.d/harbor.server.local \
cp /certs/harbor.server.local.key /etc/docker/certs.d/harbor.server.local  \
cp /certs/ca.crt /etc/docker/certs.d/harbor.server.local 
4. Restart Docker Engine: \
systemctl restart docker
5. Create log file: \
mkdir -p /var/log/harbor
7. Edit harbor.yml file
   hostname:
   certs: /certs/harbor.server.local.cert
   private_key: /certs/harbor.server.local.key
9. Run install script on harbor directory \
./install.sh





