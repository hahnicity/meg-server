# meg-server
Server side actions for MEG.

## Installation
Installation can be done using ansible. Since megserver is built with python3 but
ansible does not support python3 you must create a new virtualenv for python2 and
install ansible

### Install required dependencies
    apt-get install build-essential sudo virtualenv libssl-dev libffi-dev postgresql-server-dev-all python-dev python3-dev python-pip ansible
    
### Setup Ansible in the virtual environment
    virtualenv venv27
    source venv27/bin/activate
    pip install ansible==2.1.1.0 markupsafe
    cd ansible
    ansible-galaxy install -r requirements.yml

### Setup Ansible Targets
Change prod.inv targets for your server. Generate ssh keys and add them to your target server

### Install MEG
Replace with your preferred database password, GCM API key, and Sendgrid API key.

    ansible-playbook -i prod.inv deploy.yml --extra-vars 'meg_user_password=<meg db pw> megserver_gcm_api_key=<gcm api key> sendgrid_api_key=<sendgrid api secret>'

## General documentation
[Flowcharts](docs/flowcharts.md)

## Testing
Testing is important if you want to ensure that the changes you made to the server
did not break anything. We are performing all testing through `nose`. First you
will need to install nose in your virtualenv

    pip install nose

Then you need to run the tests

    nosetests

Failing tests are indicative of some breakage! **DO NOT** ignore failing tests.
## API
megserver provides a wrapper around PGP keyserver APIs and adds additional APIs for
MEG specific functionality as well.

### addkey
Add an armored PGP public key.

    PUT addkey

    body:
    keydata=<armored PGP public key>

### getkey
Get a key by an 8 char key id

    GET /getkey/<keyid>


### get_trust_level
Get the trust level of another key in relation to our web of trust.

    GET /get_trust_level/<our key id>/<contact key id>

Where

 * `our key id`: Our key id
 * `contact key id`: The key id of the party we are in communication with

Returns:

 * `0` if we directly trusts the contact
 * `1` if we trust contact through web of trust
 * `2` if we do not trust contact

### store_revocation_cert
Store a revocation certificate on the server

    PUT /store_revocation_cert/

    body:
    keydata=<revocation certificate>

### request_revoke
Request that a users public key be revoked

    POST /request_revoke/?keyid=<8 digit public key id>

### revoke
Revoke a users public key

    GET /revoke/?keyid=<8 digit public key id>&token=<revocation token>

### search
Search for a users public key by some string, maybe by email address or name.
This will get all non-revoked keys matching that string.

    GET /search/<search string>

### getkey_by_message_id
Get a users public key of the user we want to send a message to by the id of a
message awaiting pick up from the server. This API is only called by the phone
when it wants to encrypt a message.

    GET /getkey_by_message_id/?associated_message_id=<message id>

note: This API should probably go away in the future for simplicity sake

### decrypted_message
Get a message (that is encrypted by AES symmetric key) that is PGP decrypted. There are two cases here.

1. Get a message by message id. This is used by the phone

    `GET /decrypted_message/?message_id=<message id>`

2. Get a message by to and from email address

    `GET /decrypted_message/?email_to=<recipient email address>&email_from=<origin email address>`

Put a PGP decrypted email on the server. The message must be base64 encoded.

    PUT /decrypted_message/?action=<action>&email_to=<recipient address>&email_from=<origin address>
    Content-Type: text/plain

    data:
    <base64 encoded message>

#### actions
The list of actions available for this API are

 * toclient
 * encrypt
 * decrypt

### encrypted_message
Get a message that is PGP encrypted from the server. The methods here are almost the exact same as the actions that we can perform for a PGP decrypted message. In fact it would probably just be easier to consolidate this API with the `decrypted_message` one and just name the API `message`.

1. Get a message by message id. This is used by the phone

    `GET /encrypted_message/?message_id=<message id>`

2. Get a message by to and from email address

    `GET /encrypted_message/?email\_to=<recipient email address>&email_from=<origin email address>`

Put a PGP encrypted email on the server. The message must be base64 encoded

    PUT /encrypted_message/?action=<action>&email_to=<recipient address>&email_from=<origin address>
    Content-Type: text/plain

    data:
    <base64 encoded message>

## Acknowledgements
To SunDwarf's work done on Skier that serves as a basis and inspiration for
the code in this repo
