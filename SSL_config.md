# SSL and CSR for openshift apps
A step by step guide to getting your SSL configured for an Openshift app in the BC Public Service.

(if any of this information is out of date, or has broken links please open an issue with the [Digital.gov.bc.ca Repos](https://github.com/bcgov/digital.gov.bc.ca).)

## Step 1: Requesting your certificates

You will need to complete the request form: https://ssbc-client.gov.bc.ca/services/isr_forms/hosting_ssl_site_reverse.docx

The form is submitted through the iStore by attaching it to a new ticket. https://imbsd.gov.bc.ca/.  You will need to be connected to the VPN to access this site.

### Things to keep in mind while ordering SSL's

1) SSLs for all urls of the form `*.apps.silver.devops.gov.bc.ca` are the responsibility of the platform services team, not the project teams.
2) If your project requires subject alternate names (SANs), the number entered in the form field `SANs Quantity`, will need to match the number created in your generated CSR in part 2 of this guide.
3) For simple apps a 'Standard SSL' should be fine.
4) More reading on SSL best practices can be found [Here](https://ssbc-client.gov.bc.ca/services/SSLCert/documents/Best_Practices_SSL_Certificates.pdf).

## Step 2:  Generate a Certificate Signing Request (CSR) 

The iStore will require a CSR to be generated. This can be done using the script:

[csr_details.txt](./csr_details.txt)

Once customized, this script can be stored in your codebase, allowing you to easily regenerate the CSRs when the SSL needs to be renewed.  Two examples can be found:
- [Rocket Chat (uses 3 SANs)](https://github.com/bcgov/rocketchat/blob/master/docs/csr_details.txt)

- [Digital.gov (uses 0 SANs)](https://github.com/bcgov/digital.gov.bc.ca/blob/develop/docs/csr_details.txt)

The alt names allows you to customise the SSL for multiple urls. With openshift this is useful when you want to secure dev, test, and prod namespaces under the same SSL.

This script will generate a CSR (NAME.csr) and a Key (NAME.key).  Send the CSR to the iStore when it is requested.  Keep the key secret and safe. **DO NOT COMMIT THESE TO YOUR REPOS BY ACCIDENT.**

## Step 3 Install the SSL in openshift.

Once the iStore has sent you your SSL certs, you can install it in Openshift. 

### a. Create Secret
Store your certs in the relevant namespace (probably prod) as a secret. Run the following OC command:
```
oc -n <<NAMESPACE>> create secret generic <<NAME>>-ssl.<<YEAR>> \
 --from-file=private-key=<<NAME>>.key \
 --from-file=certificate=<<NAME>>.txt \
 --from-file=csr=<<NAME>>.csr \
 --from-file=ca-chain-certificate=L1K-for-certs.txt \
 --from-file=ca-root-certifcate=L1K-root-for-certs-G2.txt
```
For digital.gov.bc.ca the command is:
```
oc -n c0cce6-prod create secret generic digital.gov.bc.ca-ssl.2021 \
 --from-file=private-key=digital.gov.bc.ca.key \
 --from-file=certificate=digital.gov.bc.ca.txt \
 --from-file=csr=digital.gov.bc.ca.csr \
 --from-file=ca-chain-certificate=L1K-for-certs.txt \
 --from-file=ca-root-certifcate=L1K-root-for-certs-G2.txt
 ```

### b. Create Front-End Route - External Traffic

Use the following settings:

* TLS Termination: Edge
* Insecure Traffic: Redirect

| Route field    | Created secret          | Source file                      |
| -------------- | ----------------------- |--------------------------------- |
| Certificate    | certificate             | NAME.txt                         | 
| Private Key    | private-key             | NAME.key                         | 
| CA Certificate | ca-chain-certificate    | L1K-for-certs.txt                | 


### c. Verify SSO (KeyCloak) settings

If your app uses keycloak, make certain that it still works.

## Step 4. Confirm with the iStore
Once your SSL has been installed and everything is working confirm with the iStore so they can close the ticket... also say thank you... 
