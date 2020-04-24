# Makefile to create 
# * CA
# * Intermediate Certificate Authority and to sign a test cert
# * Fake CA
# * Fake Intermediate CA
# * Various test certificates for server and client
#
#  Created by Olle E. Johansson, Edvina.net - oej@edvina.net

domain:=tls-o-matic.com
.PHONY=all
OPENSSL=openssl 
OSSLCONF=-config etc/openssl.cnf
COMPANYNAME:=SIP TLS testing Inc
FAKECOMPANYNAME:="SIP fake news testing LLC"
CACERT=ca/cacert.pem
CACSR=ca/cacert.csr
ICACERT=ica/cacert.pem
ICACSR=ica/cacsr.pem
FAKECACERT=fakeca/cacert.pem
FAKECACSR=fakeca/cacert.csr
FAKEICACERT=fakeica/cacert.pem
FAKEICACSR=fakeica/cacsr.pem
# Server name for all the server tests
SERVERNAME=sip.tls-o-matic.com

export COMPANYNAME
export COMMONNAME

all:
	@echo "Targets:"
	@echo "make ca          Create CA - ordinary "
	@echo "make intermediateca  Create Intermediate CA - ordinary "
	@echo "make fakeca      Create Fake CA"
	@echo "make fakeica     Create Fake intermediate CA"
	@echo "make testclient  Create test client CSR and sign it with the CA"
	@echo "make clean	Remove all traces of what you've done"

ca/private/cakey.pem:
	@echo "==> Magically creating private key for the ordinary CA"
	@mkdir -p ca/private
	# Generate key
	@$(OPENSSL) ecparam -genkey -name secp521r1 -out ca/private/cakey.pem
	@echo "...done!"

ica/private/cakey.pem:
	@echo "==> Magically creating private key for the intermediate  CA"
	@mkdir -p ica/private
	# Generate key
	@$(OPENSSL) ecparam -genkey -name secp521r1 -out ica/private/cakey.pem
	@echo "...done!"

fakeca/private/cakey.pem:
	@echo "==> Magically creating private key for the fake CA"
	@mkdir -p fakeca/private
	# Generate key
	@$(OPENSSL) ecparam -genkey -name secp521r1 -out fakeca/private/cakey.pem
	@echo "...done!"

fakeica/private/cakey.pem:
	@echo "==> Magically creating private key for the fake Intermediate CA"
	@mkdir -p fakeica/private
	# Generate key with another curve, just for fun
	@$(OPENSSL) ecparam -genkey -name prime192v1 -out fakeica/private/cakey.pem
	@echo "...done!"


ca/cacert.csr: ca/private/cakey.pem
	@echo "==> Creating CSR for the CA"
	COMMONNAME="SIP TLS-o-matic test CA" $(OPENSSL) req -new -batch  -nodes -key ca/private/cakey.pem -days 1800 -extensions v3_intermediate_ca -out $(CACSR) $(OSSLCONF)

ica/cacsr.pem: ica/private/cakey.pem
	@echo "==> Creating CSR for the Intermediate CA"
	COMMONNAME="SIP TLS-o-matic test Intermediate CA" $(OPENSSL) req -new -batch  -nodes -key ica/private/cakey.pem -days 800 -extensions v3_intermediate_ca -out $(ICACSR) $(OSSLCONF)

fakeca/cacert.csr: fakeca/private/cakey.pem
	@echo "==> Creating CSR for the Fake CA"
	COMMONNAME="Evil SIP CA" $(OPENSSL) req -new -batch  -nodes -key fakeca/private/cakey.pem -days 1800 -extensions v3_intermediate_ca -out $(FAKECACSR) $(OSSLCONF)

fakeica/cacsr.pem: fakeica/private/cakey.pem
	@echo "==> Creating CSR for the Fake Intermediate CA"
	COMMONNAME="SIP TLS-o-matic test Fake Intermediate CA" $(OPENSSL) req -new -batch  -nodes -key fakeica/private/cakey.pem -days 800 -extensions v3_intermediate_ca -out $(FAKEICACSR) $(OSSLCONF)

fakeca/serial:
	@echo 1000 > fakeca/serial
	@touch fakeca/index.txt
	@touch fakeca/index.txt.attr

fakeica/serial:
	@echo 1000 > fakeica/serial
	@touch fakeica/index.txt
	@touch fakeica/index.txt.attr

ca/serial:
	@echo f000 > ca/serial
	@touch ca/index.txt
	@touch ca/index.txt.attr

ica/serial:
	@echo f000 > ica/serial
	@touch ica/index.txt
	@touch ica/index.txt.attr

ca/cacert.pem: ca/serial ca/private/cakey.pem
	@echo "==> Signing key for the CA and creating cert"
	COMMONNAME="SIP TLS-o-matic test CA"  $(OPENSSL) req -new -x509 -sha512 -batch  -nodes -key ca/private/cakey.pem -days 1800 -utf8 -extensions v3_intermediate_ca -out $(CACERT) $(OSSLCONF)

ica/cacert.pem: ica/serial ica/private/cakey.pem ica/cacsr.pem
	@echo "==> Signing key for the Intermediate CA and creating cert"
	@$(OPENSSL) ca -batch $(OSSLCONF) -name SIP_CA -utf8 -extensions v3_intermediate_ca -cert $(CACERT) -out $(ICACERT) -infiles ica/cacsr.pem

fakeca/cacert.pem: fakeca/serial fakeca/private/cakey.pem
	@echo "==> Signing key for the FAKE CA and creating cert"
	COMMONNAME="Evil SIP  CA" COMPANYNAME=$(FAKECOMPANYNAME)  $(OPENSSL) req -new -extensions fake_usr_cert -utf8 -x509 -sha512 -batch  -nodes -key fakeca/private/cakey.pem -days 1800 -utf8 -extensions v3_intermediate_ca -out $(FAKECACERT) $(OSSLCONF)

fakeica/cacert.pem: fakeica/serial fakeica/private/cakey.pem fakeica/cacsr.pem
	@echo "==> Signing key for the Fake Intermediate CA and creating cert"
	@$(OPENSSL) ca -batch $(OSSLCONF) -name FAKE_CA -utf8 -extensions v3_intermediate_ca -cert $(FAKECACERT) -out $(FAKEICACERT) -infiles fakeica/cacsr.pem

	
csr:
	@mkdir -p csr

client:
	@mkdir -p client

private:
	@mkdir -p private

newcerts:
	@mkdir -p newcerts

fakeca: fakeca/cacert.pem csr newcerts
	@echo "===> Creating Fake Evil CA"
	@echo 1000 > fakeca/serial
	@touch index.txt
	@mkdir -p fakeca/private
	@echo "Done!"
	@$(OPENSSL) x509 -noout -in $(FAKECACERT) -text
	@echo "===> Created FAKE CA"

ca: ca/cacert.pem csr newcerts
	@echo "===> Creating CA"
	@echo 1000 > ca/serial
	@touch ca/index.txt
	@mkdir -p ca/private
	@echo "Done!"
	@$(OPENSSL) x509 -noout -in $(CACERT) -text
	@echo "===> Created CA"

fakeica: fakeica/cacert.pem
	@echo "===> Creating FAKE Intermediate CA"
	@echo 1000 > fakeica/serial
	@touch fakeica/index.txt
	@mkdir -p fakeica/private
	@echo "Done!"
	@$(OPENSSL) x509 -noout -in $(FAKEICACERT) -text
	@echo "===> Created FAKE Intermediate CA"

intermediateca: ica/cacert.pem
	@echo "===> Creating Intermediate CA"
	@echo 1000 > ica/serial
	@touch ica/index.txt
	@mkdir -p ica/private
	@echo "Done!"
	@$(OPENSSL) x509 -noout -in $(ICACERT) -text
	@echo "===> Created Intermediate CA"

testclient: TESTCLIENT:=client$(shell cat ca/serial)@$(domain)
testclient: KEYFILE:=private/$(TESTCLIENT).key
testclient: COMMONNAME:=client$(shell cat ca/serial)@$(domain)
testclient:	ca/cacert.pem csr client private ca/private/cakey.pem
	@echo "===> Creating cert for test client"
	@$(OPENSSL) ecparam -out $(KEYFILE) -genkey  -name secp521r1 -noout
	COMMONNAME=$(COMMONNAME) $(OPENSSL) req -new -batch -nodes -key $(KEYFILE) -out csr/$(TESTCLIENT).csr  $(OSSLCONF) 
	@$(OPENSSL) ca -batch $(OSSLCONF) -utf8 -extensions client_cert -cert $(CACERT) -out client/$(TESTCLIENT).cert -infiles csr/$(TESTCLIENT).csr 

faketestclient: TESTCLIENT:=fake$(shell cat fakeca/serial)@fake.$(domain)
faketestclient: KEYFILE:=private/$(TESTCLIENT).key
faketestclient: COMMONNAME:=fake$(shell cat fakeca/serial)@fake.$(domain)
faketestclient:	fakeca/cacert.pem csr client private fakeca/private/cakey.pem
	@echo "===> Creating fake cert for test client"
	@$(OPENSSL) ecparam -out $(KEYFILE) -genkey  -name secp521r1 -noout
	COMPANYNAME=$(FAKECOMPANYNAME) COMMONNAME=$(COMMONNAME) $(OPENSSL) req -new -batch -nodes -key $(KEYFILE) -out csr/$(TESTCLIENT).csr  $(OSSLCONF) 
	@$(OPENSSL) ca -batch $(OSSLCONF) -name FAKE_CA -utf8 -extensions client_cert -cert $(FAKECACERT) -out client/$(TESTCLIENT).cert -infiles csr/$(TESTCLIENT).csr 

faketestserver: TESTSERVER:=fake$(shell cat fakeca/serial).fake.$(domain)
faketestserver: KEYFILE:=private/$(TESTSERVER).key
faketestserver: COMMONNAME:=$(SERVERNAME)
faketestserver:	fakeca/cacert.pem csr client private fakeca/private/cakey.pem
	@echo "===> Creating fake cert for test server"
	@$(OPENSSL) ecparam -out $(KEYFILE) -genkey  -name secp521r1 -noout
	COMPANYNAME=$(FAKECOMPANYNAME) COMMONNAME=$(COMMONNAME) $(OPENSSL) req -new -batch -nodes -key $(KEYFILE) -out csr/$(TESTSERVER).csr -extensions fake_server_cert  $(OSSLCONF) 
	@$(OPENSSL) ca -batch $(OSSLCONF) -name FAKE_CA -utf8 -extensions fake_server_cert -cert $(FAKECACERT) -out client/$(TESTSERVER).cert -infiles csr/$(TESTSERVER).csr 

# Valid server cert
testserver: TESTCLIENT:=server$(shell cat ca/serial)@$(domain)
testserver: KEYFILE:=private/$(TESTCLIENT).key
testserver: COMMONNAME:=$(SERVERNAME)
testserver:	ca/cacert.pem csr client private ca/private/cakey.pem
	@echo "===> Creating cert for test server"
	@$(OPENSSL) ecparam -out $(KEYFILE) -genkey  -name secp521r1 -noout
	COMMONNAME=$(COMMONNAME) $(OPENSSL) req -new -batch -nodes -key $(KEYFILE) -out csr/$(TESTCLIENT).csr  $(OSSLCONF) 
	@$(OPENSSL) ca -batch $(OSSLCONF) -utf8 -extensions server_cert -cert $(CACERT) -out client/$(TESTCLIENT).cert -infiles csr/$(TESTCLIENT).csr 

testsuite: ca fakeca testcerts fakeica intermediateca
.PHONY: testsuite testcerts

servercerts/CERTSRV01/CERTSRV01.cert.pem: 
	@bin/server-cert $SERVERNAME CERTSRV01 "CERTSRV01: Korrekt cert"  NORMAL

servercerts/CERTSRV02/CERTSRV02.cert.pem: 
	@bin/server-cert $SERVERNAME CERTSRV02 "CERTSRV02: Expired cert. Not ok."  EXPIRED

servercerts/CERTSRV03/CERTSRV03.cert.pem:
	@bin/server-cert $SERVERNAME CERTSRV03 "CERTSRV03: Future cert. Not ok."  NOTVALIDYET

servercerts/CERTSRV04/CERTSRV04.cert.pem:
	@bin/server-cert $SERVERNAME CERTSRV04 "CERTSRV04: Bad SubjectAltName. Not ok."  BADSAN

servercerts/CERTSRV05/CERTSRV05.cert.pem:
	@bin/server-cert $SERVERNAME CERTSRV05 "CERTSRV05: Bad usage (Client). Not ok."  CLIENTCERT

servercerts/CERTSRV06/CERTSRV06.cert.pem:
	@bin/server-cert $SERVERNAME CERTSRV06 "CERTSRV06: Bad cert (512bits). Not ok."  RSA512

servercerts/CERTSRV07/CERTSRV07.cert.pem:
	@bin/server-cert $SERVERNAME CERTSRV07 "CERTSRV07: Bad cert (CodeSign, not server). Not ok."  CODESIGN

servercerts/CERTSRV10/CERTSRV10.cert.pem:
	@bin/server-cert $SERVERNAME CERTSRV10 "CERTSRV10: Signed by I-CA. ok."  ICASIGN

servercerts/CERTSRV15/CERTSRV15.cert.pem: fakeca/cacert.pem
	@bin/server-cert $SERVERNAME CERTSRV15 "CERTSRV15: Signed by Fake CA. Not ok."  FAKECASRV

clientcerts/CERTCLIENT01/CERTCLIENT01.cert.pem: ca/cacert.pem
	@bin/client-cert tlsclient-001  CERTCLIENT01 "CERTCLIENT01: Signed by real CA. ok."  NORMAL

clientcerts/CERTCLIENT02/CERTCLIENT02.cert.pem: ca/cacert.pem
	@bin/client-cert tlsclient-002  CERTCLIENT02 "CERTCLIENT02: Signed by real CA. Not valid any more. Not ok."  EXPIRED

clientcerts/CERTCLIENT03/CERTCLIENT03.cert.pem: ca/cacert.pem
	@bin/client-cert tlsclient-003  CERTCLIENT03 "CERTCLIENT03: Signed by real CA. Not valid yet. Not ok."  NOTVALIDYET

testcerts: ca/cacert.pem csr client fakeca fakeica intermediateca private ca/private/cakey.pem \
	servercerts/CERTSRV01/CERTSRV01.cert.pem \
	servercerts/CERTSRV01/CERTSRV01.cert.pem \
	servercerts/CERTSRV03/CERTSRV03.cert.pem \
	servercerts/CERTSRV04/CERTSRV04.cert.pem \
	servercerts/CERTSRV05/CERTSRV05.cert.pem \
	servercerts/CERTSRV06/CERTSRV06.cert.pem \
	servercerts/CERTSRV07/CERTSRV07.cert.pem \
	servercerts/CERTSRV10/CERTSRV10.cert.pem \
	servercerts/CERTSRV15/CERTSRV15.cert.pem \
	clientcerts/CERTCLIENT01/CERTCLIENT01.cert.pem \
	clientcerts/CERTCLIENT02/CERTCLIENT02.cert.pem \
	clientcerts/CERTCLIENT03/CERTCLIENT03.cert.pem 
	@echo "Done."

clean:
	@echo "Cleaning up everything not belonging here"
	@-rm ca/cacert.*
	@-rm ca/private/*
	@-rm fakeca/cacert.*
	@-rm fakeca/private/*
	@-rm fakeica/cacert.*
	@-rm fakeica/private/*
	@-rm ica/cacert.*
	@-rm ica/private/*
	@-rm -rf csr
	@-rm -rf client
	@-rm -rf private
	@-rm -rf newcerts
	@-rm -rf servercerts/*
	@-rm -rf clientcerts/*
	@-rm ca/index.txt
	@-rm ca/index.txt.attr
	@-rm ca/serial
	@-rm ica/index.txt
	@-rm ica/index.txt.attr
	@-rm ica/serial
	@-rm fakeica/index.txt
	@-rm fakeica/index.txt.attr
	@-rm fakeica/serial
	@-rm fakeca/index.txt
	@-rm fakeca/index.txt.attr
	@-rm fakeca/serial

