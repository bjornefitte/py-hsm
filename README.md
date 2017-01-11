# pihsm

## Overview
The pihsm module enables Python users simplified access to any PKCS11 standards compliant Hardware Security Module (HSM).  The PKCS#11 API is an OASIS vendor-neutral open standards API for interfacing with Hardware Security Modules (HSMs).  The Python class pihsm.HsmClient, through the companion libhsm library, gives easy-to-use access to any HSM vendor PKCS11 compliant interface.  

## What is an HSM?
Hardware Security Modules (HSMs) are physical, electronic black box devices designed to provide secure creation, management and storage of cryptographic keys and secrets.  Most HSMs physical devices go through US and foreign government certification programs such as the US governent's FIPS program which rates the security and compliance level for a specific HSM product.

## What is PKCS#11?
Pyhsical HSMs are built by a variety of 3rd party vendors and come in a variety of form factors.  Yet, all mainstream HSM devices implement the industry OASIS C-based API called PKCS#11.  The PKCS#11 API was first an industry defacto standard API originally developed by RSA Security for HSM security tokens.  Later EMC acquired RSA Security.  Shortly after the acquisition, the OASIS standards body took control of the PKCS #11 Cryptographic Token Interface Base Specification standard and made it a true industry standard API.  Many existing software applications use the PKCS#11 API to interface with a variety of Hardware Security Modules in a vendor neutral manner.  Although it is possible for developers to directly interact with a vendor's PKCS#11 API implemenation, the API is very complex and full of trip-ups and pitfalls.  The goal of the pihsm and libhsm modules is to provide Python users a simplified HSM interface, without sacrificing performance by abstracting away many of the painful complexities of the PKCS#11 API.

## Supported HSMs
The pihsm module has been tested to work with the following HSM devices and software based testbed HSMs.
- SafeNet / Gemalto Luna SA-4
- SafeNet / Gemalto Luna SA-5
- SafeNet / Gemalto Luna PCIe K5/K6
- SafeNet / Gemalto Luna CA-4
- Utimaco Security Server Simulator (SMOS Ver. 3.1.2.3)
- OpenDNSSEC SoftHSM 2.2.0

### Installation Prerequisites
- Python 3.5.x
- libhsm https://github.com/bentonstark/libhsm

**Note:** If your server already has specific versions of these components installed, you can use a **virtualenv** to create an isolated python environment.  If there is enough demand requests, future versions may be back supported to Python 2.7.x

Tested Platforms.
- Fedora 25
- CentOS 6
- CentOS 7

## Usage Examples
### Login / Logout
```python
# long method - use with keyword for short method
c = HsmClient(pkcs11_lib="/usr/lib/vendorp11.so")
c.open_session(slot=1)
c.login(pin="partition_password")
c.logout()
c.close_session()
```
### List Slots
```python
from pihsm.hsmclient import HsmClient

# note: listing slot information does not require a login
with HsmClient(pkcs11_lib="/usr/lib/vendorp11.so") as c:
  for s in c.get_slot_info():
    print("----------------------------------------")
    print(s.to_string())
```
### List Objects
```python
from pihsm.hsmclient import HsmClient

with HsmClient(slot=1, pin="partition_password", pkcs11_lib="/usr/lib/vendorp11.so") as c:
  for s in c.get_slot_info():
    obj_list = c.get_objects()
    for obj in obj_list:
      print(obj.to_string())
```
### Sign
```python
from pihsm.hsmclient import HsmClient
from pihsm.hsmenums import HsmMech
from pihsm.convert import bytes_to_hex

with HsmClient(slot=1, pin="partition_password", pkcs11_lib="/usr/lib/vendorp11.so") as c:
  sig = c.sign(handle=1, data=data_to_sign, mechanism=HsmMech.SHA256_RSA_PKCS)
  print(bytes_to_hex(sig))
```
### Verify
```python
from pihsm.hsmclient import HsmClient
from pihsm.hsmenums import HsmMech

with HsmClient(slot=1, pin="partition_password", pkcs11_lib="/usr/lib/vendorp11.so") as c:
  result = c.verify(handle=1, data=sig_to_verify, mechanism=HsmMech.SHA256_RSA_PKCS)
  print(str(result))
```
### Encrypt
```python
from pihsm.hsmclient import HsmClient
from pihsm.hsmenums import HsmMech
from pihsm.convert import bytes_to_hex

with HsmClient(slot=1, pin="partition_password", pkcs11_lib="/usr/lib/vendorp11.so") as c:
  ciphertext = c.encrypt(handle=aes_key_handle, data=cleartext, mechanism=HsmMech.AES_CBC_PAD, iv=init_vector)
  print(bytes_to_hex(ciphertext))
```
### Decrypt
```python
from pihsm.hsmclient import HsmClient
from pihsm.hsmenums import HsmMech
from pihsm.convert import bytes_to_hex

with HsmClient(slot=1, pin="partition_password", pkcs11_lib="/usr/lib/vendorp11.so") as c:
  cleartext = c.decrypt(handle=aes_key_handle, data=ciphertext, mechanism=HsmMech.AES_CBC_PAD, iv=init_vector)
  print(bytes_to_hex(cleartext))
```
### Create AES Key
```python
from pihsm.hsmclient import HsmClient
from pihsm.hsmenums import HsmSymKeyGen

with HsmClient(slot=1, pin="partition_password", pkcs11_lib="/usr/lib/vendorp11.so") as c:
  key_handle = c.create_secret_key(key_label="my_aes_key",
                      HsmSymKeyGen.AES,
                      key_size_in_bits=256,
                      token=True,
                      private=True,
                      modifiable=False,
                      extractable=False,
                      sign=True,
                      verify=True,
                      decrypt=True,
                      wrap=True,
                      unwrap=True,
                      derive=False)    
  print(key_handle)
```
### Create RSA Key Pair
```python
from pihsm.hsmclient import HsmClient

with HsmClient(slot=1, pin="partition_password", pkcs11_lib="/usr/lib/vendorp11.so") as c:
  key_handles = c.create_rsa_key_pair(public_key_label="my_rsa_pub",
                                      private_key_label="my_rsa_pvt",
                                      key_length=2048,
                                      public_exponent=b"\x01\x00\x01",
                                      token=True,
                                      private=True,
                                      modifiable=False,
                                      extractable=False,
                                      sign_verify=True,
                                      encrypt_decrypt=True,
                                      wrap_unwrap=True,
                                      derive=False)
  print("public_handle: " + key_handles[0])
  print("private_handle: " + key_handles[1])
```
### Create EC Key Pair
```python
from pihsm.hsmclient import HsmClient
from pihsm.convert import hex_to_bytes

with HsmClient(slot=1, pin="partition_password", pkcs11_lib="/usr/lib/vendorp11.so") as c:
  # prime256v1
  ec_oid = hex_to_bytes("06082a8648ce3d03010")
  key_handles = c.create_ecc_key_pair(public_key_label="my_ec_pub",
                                      private_key_label="my_ec_pvt",
                                      curve_parameters=ec_oid,
                                      token=True,
                                      private=True,
                                      modifiable=False,
                                      extractable=False,
                                      sign_verify=True,
                                      encrypt_decrypt=True,
                                      wrap_unwrap=True,
                                      derive=False)
  print("public_handle: " + key_handles[0])
  print("private_handle: " + key_handles[1])
```
### Wrap Key (AES wrapped with AES)
```python
from pihsm.hsmclient import HsmClient
from pihsm.hsmenums import HsmMech
from pihsm.convert import bytes_to_hex

with HsmClient(slot=1, pin="partition_password", pkcs11_lib="/usr/lib/vendorp11.so") as c:
  wrapped_key_bytes = c.wrap_key(key_to_wrap, wrapping_key_handle, HsmMech.AES_CBC_PAD, iv)
  print(bytes_to_hex(wrapped_key_bytes))
```
### Unwrap Key (AES wrapped with AES)
```python
from pihsm.hsmclient import HsmClient
from pihsm.hsmenums import HsmMech
from pihsm.convert import bytes_to_hex

with HsmClient(slot=1, pin="partition_password", pkcs11_lib="/usr/lib/vendorp11.so") as c:
  hkey = c.unwrap_secret_key(wrap_key_handle=wraping_key_handle,
                             wrap_key_mech=HsmMech.AES_CBC_PAD,
                             wrap_key_iv=iv,
                             key_label="my_key",
                             key_data=wrapped_key_bytes,
                             key_type=HsmSymKeyType.AES,
                             key_size_in_bits=key_size,
                             token=True,
                             private=True,
                             modifiable=False,
                             extractable=False,
                             sign=True,
                             verify=True,
                             encrypt=True,
                             decrypt=True,
                             wrap=True,
                             unwrap=True,
                             derive=False)
```
### Generate Random
```python
from pihsm.hsmclient import HsmClient
from pihsm.convert import bytes_to_hex

with HsmClient(slot=1, pin="partition_password", pkcs11_lib="/usr/lib/vendorp11.so") as c:
  rnd_bytes = c.generate_random(16)
  print(bytes_to_hex(rnd_bytes))
```
### Get Object Handle by Label
```python
from pihsm.hsmclient import HsmClient

with HsmClient(slot=1, pin="partition_password", pkcs11_lib="/usr/lib/vendorp11.so") as c:
  handle = c.get_object_handle(args.label)
  print(str(handle))
```
### Change Object Label
```python
from pihsm.hsmclient import HsmClient
from pihsm.hsmenums import HsmAttribute
from pihsm.convert import str_to_bytes

with HsmClient(slot=1, pin="partition_password", pkcs11_lib="/usr/lib/vendorp11.so") as c:
  c.set_attribute_value(handle=object_handle, attribute_type=HsmAttribute.LABEL, attribute_value=str_to_bytes(new_label))
```



=== INSTALLATION STEPS
This package supports the Linux 64-bit and optionally Windows 64-bit.  It have been
tested with the SafeNet/Gemalto Luna SA-5, Luna K5 PCIe card, Utimaco Simulator,
and DNSSec's SoftHSM2.  

== PRE-REQUISITS

* Install libhsm shared library on the host system.
  The libhsm is a companion shared library needed by pihsm to connect to the
  vendor specific PKCS#11 library implemenation. 
  
  https://github.com/bentonstark/libhsm

* Verify Python 3+ is installed.
    $ python -V

* Uninstall any previous pihsm version.
    $ pip uninstall pihsm

== MANUAL INSTALL
    $ cd pihsm
    $ python setup.py install





	