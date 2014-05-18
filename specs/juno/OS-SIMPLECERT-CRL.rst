..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================================
Serve Certificate Revocation Lists from OS-SIMPLECERT
=====================================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/keystone/+spec/example

Support Cert



Problem description
===================

Keystone Client validates PKI Tokens using X509 Certificates.  Certificate 
Revocation Lists provide a means to revoke certificates.   Currently there is
no way to distribute CRLs.


Proposed change
===============

Add an API for uploading and fetching CRLs from Keystone, and the corresponding Command line tools the manage them.


Alternatives
------------


While the URL of the 
original signer is in the certificate, it would cause a stampeding herd for all of the services in OpenStack to attempt to fetch the CRL on a regular basis.
For the certificates provisioned via `keystone-manage ssl_setup` there is no 
real CA, and no published CRL.

Data model impact
-----------------

The OS-SIMPLECERT extension currently has no Database.  This extension


REST API impact
---------------

The OS-SIMPLECERT extension will contain a new resource;

GET /OS-SIMPLE-CERT/crls

Will Return the list of available CRLs.

{ crsl: [
      { 
          signer: http//:signer1/crl1,
          published: 14JUN2022
      },
      { 
          signer: http//:signer2/crl1,
          published: 15JUN2022
      }

}



GET /OS-SIMPLE-CERT/crls?signer=<id>

Will Return all CRLs for a specific signer.

GET /OS-SIMPLE-CERT/crls?since=<datetime>

Will return all CRLs that have been issued at or after the time specified.

The signer and since values can both be applied.


PUT /OS-SIMPLE-CERT/crls

Will all the upload of a CRL.  The CRL will be verified against the set of CA certificates.  
*  If it is valid, this returns OK 201 Created.  
*  If the CRL is not valid, or if the CRL refers to a CA that Keystone does not have a CA certificate for, this returns 400 Bad Request.  
* If the user is not authorized to upload CRLs this returns 403






Security impact
---------------

* Does this change touch sensitive data such as tokens, keys, or user data?
  
  Yes:  Certificate sare used to signe tokens, and this addresses the ability to check for certificate revocation.  Since a CRL must be signed, it can only strengthen the security of the system to have them available.    Note that Revocation checking is not currently implemetned in keystoneclient.  
   

* Does this change alter the API in a way that may impact security, such as
  a new way to access sensitive information or a new way to login?

  yes, as CRLs can be used as part of the toekn signing process.

* Does this change involve cryptography or hashing?

  Yes, X509 certificates signatures, the same way that 

* Does this change require the use of sudo or any elevated privileges?

  No

* Does this change involve using or parsing user-provided data? This could
  be directly at the API level or indirectly such as changes to a cache layer.

  Yes, but the parsing would be done by the cryptographic libraries, and not
  in Keystone code.

* Can this change enable a resource exhaustion attack, such as allowing a
  single API interaction to consume significant server resources? Some examples
  of this include launching subprocesses for each connection, or entity
  expansion attacks in XML.

  These will be Admin only APIs.  It would require significant priviledge in 
  order to abuse this API.


Notifications impact
--------------------

Please specify any changes to notifications. Be that an extra notification,
changes to an existing notification, or removing a notification.

*  A future enhance would have new CRLs published via notifications.

Other end user impact
---------------------

Aside from the API, are there other ways a user will interact with this
feature?

Keystone client will have to change in three ways:
  
*  Provide an API to allow uploading a CRL
*  Provide an API to allow downloading CRLs
*  provide a means to check the certificate used for document signing against
   the CRL  

keystone manage will  provide a means to generate a CRL from a list of
revoked certificates.


Performance Impact
------------------

Fetching CRLs should be done at a scheduled interval, comparable to how Token
Revocation lists are fetched.  It will be up to the deployer to set an
acceptable schedule for checking for CRLs, but a rule of thumb is that it
should be less often than for token revocation lists.

The default will be one hour.


Other deployer impact
---------------------

Discuss things that will affect how you deploy and configure OpenStack
that have not already been mentioned, such as:

*  Currently Keystone only supports a single CA cert.  this extends it to
   handling several.  The CA cert for CMS is currently named explicitly.
   Openssl and NSS both support pointing to a bundle of CA certs for validation.
   

* In the auth_token middleware,  the time to fetch the CRL needs to either be
  added, or we need to piggyback it on top of the TRL fetch.

* Is this a change that takes immediate effect after its merged, or is it
  something that has to be explicitly enabled?


Developer impact
----------------

Discuss things that will affect other developers working on OpenStack,
such as:

* If the blueprint proposes a change to the driver API, discussion of how
  other hypervisors would implement the feature is required.


Implementation
==============

Assignee(s)
-----------

Who is leading the writing of the code? Or is this a blueprint where you're
throwing it out there to see who picks it up?

If more than one person is working on the implementation, please designate the
primary author and contact.

Primary assignee:
  <launchpad-id or None>

Other contributors:
  <launchpad-id or None>

Work Items
----------

Work items or tasks -- break the feature up into the things that need to be
done to implement it. Those parts might end up being done by different people,
but we're mostly trying to understand the timeline for implementation.


Dependencies
============

* Include specific references to specs and/or blueprints in keystone, or in
  other projects, that this one either depends on or is related to.

* If this requires functionality of another project that is not currently used
  by Keystone (such as the glance v2 API when we previously only required v1),
  document that fact.

* Does this feature require any new library dependencies or code otherwise not
  included in OpenStack? Or does it depend on a specific version of library?


Testing
=======

Please discuss how the change will be tested. We especially want to know what
tempest tests will be added. It is assumed that unit test coverage will be
added so that doesn't need to be mentioned explicitly, but discussion of why
you think unit tests are sufficient and we don't need to add more tempest
tests would need to be included.

Is this untestable in gate given current limitations (specific hardware /
software configurations available)? If so, are there mitigation plans (3rd
party testing, gate enhancements, etc).


Documentation Impact
====================

What is the impact on the docs team of this change? Some changes might require
donating resources to the docs team to have the documentation updated. Don't
repeat details discussed above, but please reference them here.


References
==========

Please add any useful references here. You are not required to have any
reference. Moreover, this specification should still make sense when your
references are unavailable. Examples of what you could include are:

* Links to mailing list or IRC discussions

* Links to notes from a summit session

* Links to relevant research, if appropriate

* Related specifications as appropriate (e.g.  if it's an EC2 thing, link the
  EC2 docs)

* Anything else you feel it is worthwhile to refer to
