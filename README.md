# Yubikey-OpenPGP
Yubikey+OpenPGP configuration notes
Great repo with a lot of info: https://github.com/drduh/YubiKey-Guide

## OpenPGP

Validate information:
```
certutil -scinfo
gpg --card-status
```

Available store for 3 keys:
 - Encryption key
 - Signing key
 - Authentication key


Generating new Cert: 
```
gpg --expert --full-generate-key
8 	<- to config RSA cert
#Now select only Certify action (adding/removing actions by selecting their IDs)
```

Adding key to certificate:
```
gpg --expert --edit-key <email>

Encryption key:
	addkey
	6	<- for encryption key type
	
Signing key:
	addkey
	4 	<- for signing key type
	
Authentication key:
	addkey
	8 	<- for configurable key
	#Select only Authenticate action (adding/removing actions by selecting their IDs)
	
	save
```

**Important**: Export private and public keys + store them in the safe location (in case of losing primary and secondary yubikey keys) - this can be done from Kleopatra GUI. 

Adding keys to card
```
gpg --expert --edit-key <email>
key <number> 	<- number between 1 and 3 to select given key (also to unmark given key just select its number once again).
keytocard
#Select key type to export
	
#Repeat whole operation 3 times (to import all keys).
```

**Important**: If you have 2 physical keys (primary/backup) it'd be nice to import keys to each of them. If keys are imported following above process, leave the gpg using Ctrl+C (or quit, but without saving), swap physical keys in USB port and then repeat steps from <Adding keys to card> module. 
It's required to not save the state when swapping keys, because when saved, you won't be able to import keys to second yubikey. More info: https://lists.archive.carbon60.com/gnupg/users/76764
"It copies, but if you then save the changes to your local disk, the
original copy on local disk is deleted - so calling it a "move"
operation is correct. If you want to keep a backup copy on local disk,
you need to quit *without saving* immediately after running 'keytocard'.
This behaviour is a well-known gotcha. "


## Reset Admin PIN Counter

Source: https://support.yubico.com/hc/en-us/articles/360013761339-Resetting-the-OpenPGP-Application-on-the-YubiKey


To check PIN Counter status: 
```
gpg --card-status
```
  
If counter == 3 0 0 
Then card is probably blocked (all 3 attempts failed). 
If counter == 3 0 3 
All 3 attempts are available (for Yubikey 5)

To reset counter: 
```
gpg-connect-agent --hex

#Standard PIN reset: 
scd apdu 00 20 00 81 08 40 40 40 40 40 40 40 40
#Repeat until response is like: D[0000]  69 83 	<-for Yubikey 5

#Admin PIN reset: 
scd apdu 00 20 00 83 08 40 40 40 40 40 40 40 40
#Repeat until response is like: D[0000]  69 83 	<-for Yubikey 5

#Terminate card: 
scd apdu 00 e6 00 00

#Reactive card:
scd apdu 00 44 00 00
```

Easier method: 
```
cd "C:\Program Files\Yubico\YubiKey Manager"
ykman openpgp reset
```

## SSH
Configure Kleopatra: 
	Go to Kleopatra Settings -> GnuPG System -> Private Keys -> Enable SSH Support + Enable Putty Support
	
Before connection run agent:
```
gpg-connect-agent 
```
	
Export SSH key: 
```
gpg --export-ssh-key <email>
```
^Generates a key that needs to be placed in ~/.ssh/authorized_keys on the target host. 


## Git
Configure Git to use Signing key:
```
git config --global gpg.program <path to gpg executable>
git config --global commit.gpgsign true
git config --global user.signingkey <Signing Key ID>
```

