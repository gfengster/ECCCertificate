# ECCCertificate

openssl ecparam -list_curves

mkdir tls
cd tls

mkdir private certs

touch index.txt
echo 01 > serial



openssl.cnf
#
# OpenSSL example configuration file.
# This is mostly being used for generation of certificate requests.
#

# This definition stops the following lines choking if HOME isn't
# defined.
HOME                    = .
RANDFILE                = $ENV::HOME/.rnd

# Extra OBJECT IDENTIFIER info:
#oid_file               = $ENV::HOME/.oid
oid_section             = new_oids

# To use this configuration file with the "-extfile" option of the
# "openssl x509" utility, name here the section containing the
# X.509v3 extensions to use:
# extensions            =
# (Alternatively, use a configuration file that has only
# X.509v3 extensions in its main [= default] section.)

[ new_oids ]

# We can add new OIDs in here for use by 'ca', 'req' and 'ts'.
# Add a simple OID like this:
# testoid1=1.2.3.4
# Or use config file substitution like this:
# testoid2=${testoid1}.5.6

# Policies used by the TSA examples.
tsa_policy1 = 1.2.3.4.1
tsa_policy2 = 1.2.3.4.5.6
tsa_policy3 = 1.2.3.4.5.7

####################################################################
[ ca ]
default_ca      = CA_default            # The default ca section

[ CA_default ]
dir             = /root/tls             # Where everything is kept
certs           = $dir/certs            # Where the issued certs are kept
database        = $dir/index.txt        # database index file.
                                        # several certs with same subject.
new_certs_dir   = $dir/certs            # default place for new certs.
certificate     = $dir/certs/ec-cacert.pem       # The CA certificate
serial          = $dir/serial           # The current serial number
crlnumber       = $dir/crlnumber        # the current crl number
                                        # must be commented out to leave a V1 CRL
private_key     = $dir/private/ec-cakey.pem # The private key

name_opt        = ca_default            # Subject Name options
cert_opt        = ca_default            # Certificate field options

default_days    = 365                   # how long to certify for
default_crl_days= 30                    # how long before next CRL
default_md      = sha256                # use SHA-256 by default
preserve        = no                    # keep passed DN ordering
policy          = policy_match

# For the CA policy
[ policy_match ]
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ policy_anything ]
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

####################################################################
[ req ]
default_bits            = 2048
default_md              = sha256
default_keyfile         = privkey.pem
distinguished_name      = req_distinguished_name
attributes              = req_attributes
x509_extensions = v3_ca # The extentions to add to the self signed cert

[ req_distinguished_name ]
countryName                     = Country Name (2 letter code)
countryName_default             = IN
countryName_min                 = 2
countryName_max                 = 2
stateOrProvinceName             = State or Province Name (full name)
stateOrProvinceName_default     = Some-State
localityName                    = Locality Name (eg, city)
localityName_default            = BANGALORE
0.organizationName              = Organization Name (eg, company)
0.organizationName_default      = GoLinuxCloud
organizationalUnitName          = Organizational Unit Name (eg, section)
commonName                      = Common Name (eg, your name or your server\'s hostname)
commonName_max                  = 64
emailAddress                    = Email Address
emailAddress_max                = 64

[ req_attributes ]
challengePassword               = A challenge password
challengePassword_min           = 4
challengePassword_max           = 20
unstructuredName                = An optional company name


[ v3_req ]
# Extensions to add to a certificate request
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment

[ v3_ca ]
# Extensions for a typical CA
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer
basicConstraints = critical,CA:true

[ crl_ext ]
# issuerAltName=issuer:copy
authorityKeyIdentifier=keyid:always


openssl ecparam -out private/ca.key -name prime256v1 -genkey
openssl ecparam -in private/ca.key -text -noout

openssl req -new -x509 -days 3650 -config openssl.cnf -extensions v3_ca -key private/ca.key -out certs/ca.crt
openssl x509 -noout -text -in certs/ca.crt | grep -i algorithm

openssl x509 -noout -pubkey -in certs/ca.crt
openssl pkey -pubout -in private/ca.key

mkdir server_certs
cd server_certs/

openssl ecparam -out server.key -name prime256v1 -genkey
openssl ecparam -in server.key -text -noout


openssl req -new -key server.key -out server.csr -sha256 -config ../openssl.cnf

openssl ca -keyfile ../private/ca.key -cert ../certs/ca.crt -in server.csr -out server.crt -config ../openssl.cnf

openssl verify -CAfile ../certs/ca.crt server.crt
openssl x509 -noout -text -in server.crt | grep -i algorithm

cat ../index.txt


mkdir client_certs
cd client_certs/

openssl ecparam -out client.key -name prime256v1 -genkey
openssl ecparam -in client.key -text -noout

openssl req -new -key client.key -out client.csr -sha256 --config ../openssl.cnf 

openssl ca -keyfile ../private/ca.key -cert ../certs/ca.crt -in client.csr -out client.crt -config ../openssl.cnf

cat ../index.txt


