SIP Test Intermediate CA
------------------------
This is an example CA for testing.

The goal here is to create a CA and an Intermediate CA and use that to sign
various keys for testing

Run "make testsuite" to generate a lot of stuff.

make ca		       Creates the root CA
make intermediateca    Creates and intermediate CA under the root CA
make fakeca            Creates a fake CA not to be trusted
make testcerts         Creates test certificates for servers in the "servercerts" directory

Note that this repository has no CA or certificates - you create them using
the makefile.
