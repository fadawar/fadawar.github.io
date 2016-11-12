Toto je ta chyba:

TASK [setup] *******************************************************************
fatal: [127.0.0.1]: UNREACHABLE! => {"changed": false, "msg": "SSH Error: data could not be sent to the remote host. Make sure this host can be reached over ssh", "unreachable": true}
	to retry, use: --limit @/home/fadawar/Dokumenty/PycharmProjects/spinehero-website/ansible/vagrant.retry

Musel som sa rucne prihlasit
ssh vagrant@127.0.0.1 -p 2222

A potvrdit ze chcem tento fingerprint

The authenticity of host '[127.0.0.1]:2222 ([127.0.0.1]:2222)' can't be established.
ECDSA key fingerprint is SHA256:An0SmqEv4WWyXqjDISSg5WNWF/NlEx8oHDIMcK5eqec.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[127.0.0.1]:2222' (ECDSA) to the list of known hosts.
vagrant@127.0.0.1's password:
