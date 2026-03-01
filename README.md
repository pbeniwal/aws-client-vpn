# AWS Client VPN Connection

## Architecture Diagram:

![draw](https://github.com/pbeniwal/aws-client-vpn/blob/main/client-vpn-arch.png)

### Step 1: Certificate creation
 1. Clone Easy RSA Git Repo
 ```
 git clone https://github.com/OpenVPN/easy-rsa.git 
 ```
 2. Initialize Public Key Infrastructure (PKI)
 ```
 cd easy-rsa/easyrsa3
 ./easyrsa init-pki
 ```

 3. Build Certificate Authority
 ```
 ./easyrsa build-ca nopass
 ```
 4. Build Server Certificate
 ```
 ./easyrsa build-server-full clientvpndemo.com nopass
 ```
 5. Build Client Certificate
 ```
 ./easyrsa build-client-full user.clientvpndemo.com nopass
 ```
 6. Copy required certificates in to a single folder
 ```
 mkdir acm
cp pki/ca.crt acm
cp pki/issued/clientvpndemo.com.crt acm
cp pki/issued/pdomala.clientvpndemo.com.crt acm
cp pki/private/clientvpndemo.com.key acm
cp pki/private/pdomala.clientvpndemo.com.key acm
 ```
 7. Upload to AWS Certificate Manager (ACM
 ```
cd acm
aws acm import-certificate --certificate fileb://clientvpndemo.com.crt --private-key fileb://clientvpndemo.com.key --certificate-chain fileb://ca.crt --region us-east-1
aws acm import-certificate --certificate fileb://user.clientvpndemo.com.crt --private-key fileb://user.clientvpndemo.com.key --certificate-chain fileb://ca.crt --region us-east-1
 ```

### Create Client VPN Endpoint
 8. Create a Client VPN Endpoint Connection
 ```bash
 Name: vpn-aws-azure
 Target gateway type: Virtual private gateway (Select your Virtual private gateway created in 7)
 Customer gateway: Existing (Select your VCustomer gateway created in 6)
 Routing options: Static
 Static IP prefixes: 10.1.0.0/16
 Leave rest of them as default
 ```
 9. Download the configuration file
 ```bash
 Vendor: Generic
 Platform: Generic
 Software: Vendor Agnostic
 In this configuration file you will note that there are the Shared Keys and the Public Ip Address for each of one of the two IPSec tunnels created by AWS.
 ```
### Connecting Azure and AWS
 
 10. Create the Local Network Gateway in Azure
 ```bash
 Name: lng-azure-aws
 Resource Group Name: rg-azure-aws
 Region: East-US
 IP address: Get the Outside IP address from the configuration file downloaded in 9.
 Address Space(s): 10.0.0.0/16
 ```
 11. Create the connection on the Virtual Network Gateway in Azure
 ```bash
 Name: connection-azure-aws
 Connection Type: Site-to-Site
 Local Network Gateway: Select the Local Network Gateway which you created in 10.
 Shared Key: Get the Shared Key from the configuration file downloaded in 9.
 Wait till the Connection Status changes to - Connected
 In the same way, check in AWS Console wheather the 1st tunnel of Virtual Private Gateway UP.
 ```
 12. Create Internet Gateway and Attach it to VPC in AWS:\
 ```bash
 Name: my-internet-gateway
 ```
 13. Now let's edit the route table associated with our VPC
 ```bash
 Add the route to Azure subnet through the Virtual Private Gateway
 Destination: 10.1.0.0/16
 Target: Virtual Private Gateway that we created.
 also add,
 Destination: 0.0.0.0/0
 Target: Internet Gateway that we created in 12.
 ```
14. Create VMs in both Azure and AWS and Test the connection.
