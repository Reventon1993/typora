work



```python
import os
import six
from cryptography import fernet
import msgpack
import uuid
import time
import base64
import sys
 
token= sys.argv[1] if len(sys.argv) > 1 else None
if not token:
    print("please input a token")
    exit()
 
def load_keys():
    # build a dictionary of key_number:encryption_key pairs
    keys = dict()
    for filename in os.listdir("/etc/keystone/fernet-keys/"):
        path = os.path.join("/etc/keystone/fernet-keys/", str(filename))
        if os.path.isfile(path):
            with open(path, 'r') as key_file:
                try:
                    key_id = int(filename)
                except ValueError:
                    pass
                else:
                    keys[key_id] = key_file.read()
 
    return [keys[x] for x in sorted(keys.keys(), reverse=True)]
 
def convert_uuid_bytes_to_hex(uuid_byte_string):
    uuid_obj = uuid.UUID(bytes=uuid_byte_string)
    return uuid_obj.hex
 
def base64_encode(s):
    """Encode a URL-safe string."""
    return base64.urlsafe_b64encode(s).rstrip('=')
 
 
def disassemble(payload):
    (is_stored_as_bytes, user_id) = payload[0]
    if is_stored_as_bytes:
        user_id = "user_id = " + convert_uuid_bytes_to_hex(user_id)
    methods = "method = " + str((payload[1]))
    (is_stored_as_bytes, project_id) = payload[2]
    if is_stored_as_bytes:
        project_id = "project_id = " + convert_uuid_bytes_to_hex(project_id)
    expires_at_str = "expired_at = " + time.strftime("%Y-%m-%d %H:%M:%S",time.localtime((payload[3])))
    audit_ids =  list(map(base64_encode,payload[4]))
    return (user_id, project_id, expires_at_str,audit_ids)
 
 
def decoder(token):
    token = six.binary_type(token)
    mod_returned = len(token) % 4
    if mod_returned:
        missing_padding = 4 - mod_returned
        token += b'=' * missing_padding
    keys = load_keys()
    fernet_instances = [fernet.Fernet(key) for key in keys]
    serialized_payload =  fernet.MultiFernet(fernet_instances).decrypt(token)
    # payload format: (version,user_id,method,vertion_data,audit_ids)
    payload = msgpack.unpackb(serialized_payload)
    print("raw token info:\n%s" % payload)
    # version 2 is ProjectScopedPayload ,see keystone/token/providers/fernet/token_formatters.py
    if payload[0] == 2:
        return disassemble(payload[1:])
    else:
        return None
 
res = decoder(token)
if not res:
    print("could only resolve ProjectScopedPayload!")
    exit()
print("=================================================\nunpack token:")
for i in res:
    print(i)
```

