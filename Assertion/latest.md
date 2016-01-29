# Divergence from the Mozilla Open Badges specification.

This specifaction is based on the [Open Badges
Specification](https://github.com/mozilla/openbadges-specification) originally
authored by Mozilla in 2013.  It diverges from that specification to ensure
that badges can be completely portable.  This involves the removal of any
dependency on a webserver or other outside entity to verify the authenticity of
a badge.  The following primary changes have been made.

* All images are referred to as URI with Data URI as the recommended format.
* All previous references to URL are now URI.  This allows linking to any type
  of resource such as an IPFS link.
* All properties which required an HTTP request have been replaced with the
  object that would have been returned by that request.  (Ex. the
  **BadgeAssertion.badge** property now embeds the **BadgeClass** object
  instead of requesting it from a server).

The largest change is that this specification now **requires** tokens to be in
the **signed assertion** format.


# Assertion

Assertions are representations of an awarded badge, used to share information about badges that you've earned with the Backpack. With the 1.0 release of the OBI, there are two types of assertions: hosted and signed. For information regarding the spec prior to the latest release including instructions regarding backwards compatibility see the [spec changes](https://github.com/mozilla/openbadges/wiki/Assertion-Specification-Changes).


## Assertion Types

### Hosted

A hosted assertion is a file containing a well-formatted badge assertion in JSON served with the content-type `application/json`. This should live at a stable URL on your server (for example, [https://example.org/beths-robotics-badge.json](https://example.org/beths-robotics-badge.json)) -- it is the source of truth for the badge and any future verification attempt will hit that URL to make sure the badge exists and was issued by you.

### Signed

A signed badge is in the form of a [JSON Web Signature](http://self-issued.info/docs/draft-ietf-jose-json-web-signature.html). Signed badges use the Backpack javascript [issuer API](https://github.com/mozilla/openbadges/wiki/Issuer-API) to pass a signature rather than a URL. The JSON representation of the badge assertion should be used as the JWS payload.

## Assertion Specification

### Structures

Fields marked **in bold letters** are mandatory.


#### BadgeAssertion

Property | Expected Type | Description
-------- | ------------- | -----------
**uid** | Text | Unique Identifier for the badge. This is expected to be **locally** unique on a per-origin basis, not globally unique.
**recipient** | [IdentityObject](#identityobject) | The recipient of the achievement.
**badge** | URI or [BadgeClass](#badgeclass) object | The badge being awarded.
**verify** | [VerificationObject](#verificationobject) | Cryptographic key to verify this assertion.
**issuedOn** | [DateTime](#datetime) | Date that the achievement was awarded.
image | URI | URI of an image representing this user's achievement. This must be a PNG or SVG image, and if possible, the image should be prepared via the [Baking specification](https://github.com/mozilla/openbadges/wiki/Badge-Baking).  A [Data URI](http://en.wikipedia.org/wiki/Data_URI_scheme) is the preferred format due to portability.
evidence | Text or URI | The work that the recipient did to earn the achievement.  This may either be embedded text or a URI that points to the evidence.  Using URIs to non-permanent or mutable resources is discouraged in favor of either embedding the evidence as text or using an IPFS URI.
expires | [DateTime](#datetime) | If the achievment has some notion of expiry, this indicates when a badge should no longer be considered valid.


#### <a id="identity-object"></a>IdentityObject

Property | Expected Type | Description
--------|------------|-----------
**identity** | [IdentityHash](#identityhash) or Text | Either the hash of the identity or the plaintext value. If it's possible that the plaintext transmission and storage of the identity value would leak personally identifiable information, it is strongly recommended that an IdentityHash be used.
**type** | [IdentityType](#identitytype) | The type of identity.
**hashed** | Boolean | Whether or not the `id` value is hashed.
salt | Text | If the recipient is hashed, this should contain the string used to salt the hash. If this value is not provided, it should be assumed that the hash was not salted.


#### <a id="verification-object"></a>VerificationObject

Property | Expected Type | Description
--------|------------|-----------
**type** | VerificationType | The type of verification method.
**public_key** | Public Key | This must be the issuer's public key.



#### <a id="badge-class"></a>BadgeClass

Property | Expected Type | Description
--------|------------|-----------
**name** | Text | The name of the achievement.
**description** | Text | A short description of the achievement.
**image** | URI | URI of an image representing the achievement. This must be a PNG or SVG image.  A [Data URI](http://en.wikipedia.org/wiki/Data_URI_scheme) is the preferred format due to portabilitis the preferred format due to portability.
**criteria** | TEXT | The criteria for earning the achievement.
**issuer** | URL or **IssuerOrganization** objet | URL of the organization that issued the badge. Endpoint should be an [IssuerOrganization](#issuerorganization)
alignment | Array of [AlignmentObject](#alignmentobject)s | List of objects describing which educational standards this badge aligns to, if any.
tags | Array of Text | List of tags that describe the type of achievement.


#### <a id="issuer-organization"></a>IssuerOrganization

Property | Expected Type | Description
--------|------------|-----------
**name** | Text | The name of the issuing organization.
**url** | URI | URI of the institution
description | Text | A short description of the institution
image | URI | URI of an image representing the institution. A [Data URI](http://en.wikipedia.org/wiki/Data_URI_scheme) is the preferred format due to portability.
email | Text | Contact address for someone at the organization.
revocationList | URI |  URI of the Badge Revocation List. This resource should be a JSON representation of an object where the keys are the **uid** a revoked badge assertion, and the values are the reason for revocation.


#### <a id="alignment-object"></a>AlignmentObject

Property | Expected Type | Description
--------|------------|-----------
**name** | Text | Name of the alignment.
**url** | URI | URI linking to the official description of the standard.
description | Text | Short description of the standard


### <a id="additional-properties"></a>Additional Properties

Additional properties are allowed so long as they don't clash with specified
properties. **Processors should preserve all properties when rehosting or
retransmitting**.

Any additional properties SHOULD be namespaced to avoid clashing with
future properties. For example, if the issuer at **example.org** wants
to add a `foo` property to the assertion, the property name should be
`example.org:foo`. This will help prevent unforseen errors should an
`foo` property be defined in a later version of the specification.

If a property would be useful beyond internal use, proposals for
standardizing can be sent to
[the openbadges-dev mailing list](https://groups.google.com/forum/?fromgroups#!forum/openbadges-dev).

### Primitives

* Boolean
* Text
* Array
* <a id="date-time"></a>DateTime - Either an [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) date or a standard 10-digit Unix timestamp.
* URI - Fully qualified URI, including protocol, host, port if applicable, and path.
* <a id="identity-type"></a>IdentityType - Type of identity being represented. Currently the only supported value is "email"
* <a id="identity-hash"></a>IdentityHash - A hash string preceded by a dollar sign ("$") and the algorithm used to generate the hash. For example: `sha256$28d50415252ab6c689a54413da15b083034b66e5` represents the result of calculating a SHA256 on the string "mayze". For more information, see [how to hash & salt in various languages](https://github.com/mozilla/openbadges/wiki/How-to-hash-&-salt-in-various-languages.).
* <a id="verification-type"></a>VerificationType - Type of verification. Currently the only supported value is "signed".

## JSON Example

There are three JSON files necessary to create a valid assertion:

* _Badge Assertion:_ contains information regarding a specific badge that was awarded to a user
https://example.org/beths-robotics-badge.json
```json
{
  "uid": "f2c20",
  "recipient": {
    "type": "email",
    "hashed": true,
    "salt": "deadsea",
    "identity": "sha256$c7ef86405ba71b85acd8e2e95166c4b111448089f2e1599f42fe1bba46e865c5"
  },
  "image": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4//8/w38GIAXDIBKE0DHxgljNBAAO9TXL0Y4OHwAAAABJRU5ErkJggg==",
  "evidence": "Demonstrated understanding of the coursework.",
  "issuedOn": 1359217910,
    "badge": {
    "name": "Awesome Robotics Badge",
    "description": "For doing awesome things with robots that people think is pretty great.",
    "image": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4//8/w38GIAXDIBKE0DHxgljNBAAO9TXL0Y4OHwAAAABJRU5ErkJggg==",
    "criteria": "Completed the Robotics coursework",
    "tags": ["robots", "awesome"],
    "issuer": {
      "name": "An Example Badge Issuer",
      "image": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4//8/w38GIAXDIBKE0DHxgljNBAAO9TXL0Y4OHwAAAABJRU5ErkJggg==",
      "url": "https://example.org",
      "email": "steved@example.org",
      "revocationList": "https://example.org/revoked.json"
    },
    "alignment": [
      { "name": "CCSS.ELA-Literacy.RST.11-12.3",
        "url": "http://www.corestandards.org/ELA-Literacy/RST/11-12/3",
        "description": "Follow precisely a complex multistep procedure when carrying out experiments, taking measurements, or performing technical tasks; analyze the specific results based on explanations in the text."
      },
      { "name": "CCSS.ELA-Literacy.RST.11-12.9",
        "url": "http://www.corestandards.org/ELA-Literacy/RST/11-12/9",
        "description": " Synthesize information from a range of sources (e.g., texts, experiments, simulations) into a coherent understanding of a process, phenomenon, or concept, resolving conflicting information when possible."
      }
    ]
  },
  "verify": {
    "type": "signed",
    "public_key": "-----BEGIN PGP PUBLIC KEY BLOCK-----\nVersion: PGPfreeware 6.5.8 for non-commercial use <http://www.pgp.com>\n\nmQGiBDheqqARBAD//2FUIkCc9ITtszMh70nFmTOj/YWWi3Kk4aumxuAhgGeEwAFX\n...\n-----END PGP PUBLIC KEY BLOCK-----"
  }
}
```


# <a id="implementation"></a> Implementation

## Signed Badges

A signed badge is in the form of a
[JSON Web Signature](http://self-issued.info/docs/draft-ietf-jose-json-web-signature.html):

```
<encoded JWS header>.<encoded JWS payload>.<encoded JWS signature>
```

The JSON representation of the badge assertion should be used as the JWS
payload. For compatibility purposes, using an RSA-SHA256 is highly
recommended.

An example, with linebreaks for display purposes:

```
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
.
eyJpc3N1ZWRPbiI6MTM1OTIxNzkxMCwidWlkIjoiZjJjMjAiLCJyZWNpcGllbnQiOnsic2FsdCI6ImRlYWRzZWEiLCJ0eXBlIjoiZW1haWwiLCJoYXNoZWQiOnRydWUsImlkZW50aXR5Ijoic2hhMjU2JGM3ZWY4NjQwNWJhNzFiODVhY2Q4ZTJlOTUxNjZjNGIxMTE0NDgwODlmMmUxNTk5ZjQyZmUxYmJhNDZlODY1YzUifSwiaW1hZ2UiOiJkYXRhOmltYWdlL3BuZztiYXNlNjQsaVZCT1J3MEtHZ29BQUFBTlNVaEVVZ0FBQUFVQUFBQUZDQVlBQUFDTmJ5YmxBQUFBSEVsRVFWUUkxMlA0Ly84L3czOEdJQVhESUJLRTBESHhnbGpOQkFBTzlUWEwwWTRPSHdBQUFBQkpSVTVFcmtKZ2dnPT0iLCJldmlkZW5jZSI6IkRlbW9uc3RyYXRlZCB1bmRlcnN0YW5kaW5nIG9mIHRoZSBjb3Vyc2V3b3JrLiIsInZlcmlmeSI6eyJ1cmwiOiItLS0tLUJFR0lOIFBHUCBQVUJMSUMgS0VZIEJMT0NLLS0tLS1cblZlcnNpb246IFBHUGZyZWV3YXJlIDYuNS44IGZvciBub24tY29tbWVyY2lhbCB1c2UgPGh0dHA6Ly93d3cucGdwLmNvbT5cblxubVFHaUJEaGVxcUFSQkFELy8yRlVJa0NjOUlUdHN6TWg3MG5GbVRPai9ZV1dpM0trNGF1bXh1QWhnR2VFd0FGWFxuLi4uXG4tLS0tLUVORCBQR1AgUFVCTElDIEtFWSBCTE9DSy0tLS0tIiwidHlwZSI6InNpZ25lZCJ9LCJiYWRnZSI6eyJuYW1lIjoiQXdlc29tZSBSb2JvdGljcyBCYWRnZSIsInRhZ3MiOlsicm9ib3RzIiwiYXdlc29tZSJdLCJpbWFnZSI6ImRhdGE6aW1hZ2UvcG5nO2Jhc2U2NCxpVkJPUncwS0dnb0FBQUFOU1VoRVVnQUFBQVVBQUFBRkNBWUFBQUNOYnlibEFBQUFIRWxFUVZRSTEyUDQvLzgvdzM4R0lBWERJQktFMERIeGdsak5CQUFPOVRYTDBZNE9Id0FBQUFCSlJVNUVya0pnZ2c9PSIsImRlc2NyaXB0aW9uIjoiRm9yIGRvaW5nIGF3ZXNvbWUgdGhpbmdzIHdpdGggcm9ib3RzIHRoYXQgcGVvcGxlIHRoaW5rIGlzIHByZXR0eSBncmVhdC4iLCJjcml0ZXJpYSI6IkNvbXBsZXRlZCB0aGUgUm9ib3RpY3MgY291cnNld29yayIsImFsaWdubWVudCI6W3sidXJsIjoiaHR0cDovL3d3dy5jb3Jlc3RhbmRhcmRzLm9yZy9FTEEtTGl0ZXJhY3kvUlNULzExLTEyLzMiLCJuYW1lIjoiQ0NTUy5FTEEtTGl0ZXJhY3kuUlNULjExLTEyLjMiLCJkZXNjcmlwdGlvbiI6IkZvbGxvdyBwcmVjaXNlbHkgYSBjb21wbGV4IG11bHRpc3RlcCBwcm9jZWR1cmUgd2hlbiBjYXJyeWluZyBvdXQgZXhwZXJpbWVudHMsIHRha2luZyBtZWFzdXJlbWVudHMsIG9yIHBlcmZvcm1pbmcgdGVjaG5pY2FsIHRhc2tzOyBhbmFseXplIHRoZSBzcGVjaWZpYyByZXN1bHRzIGJhc2VkIG9uIGV4cGxhbmF0aW9ucyBpbiB0aGUgdGV4dC4ifSx7InVybCI6Imh0dHA6Ly93d3cuY29yZXN0YW5kYXJkcy5vcmcvRUxBLUxpdGVyYWN5L1JTVC8xMS0xMi85IiwibmFtZSI6IkNDU1MuRUxBLUxpdGVyYWN5LlJTVC4xMS0xMi45IiwiZGVzY3JpcHRpb24iOiIgU3ludGhlc2l6ZSBpbmZvcm1hdGlvbiBmcm9tIGEgcmFuZ2Ugb2Ygc291cmNlcyAoZS5nLiwgdGV4dHMsIGV4cGVyaW1lbnRzLCBzaW11bGF0aW9ucykgaW50byBhIGNvaGVyZW50IHVuZGVyc3RhbmRpbmcgb2YgYSBwcm9jZXNzLCBwaGVub21lbm9uLCBvciBjb25jZXB0LCByZXNvbHZpbmcgY29uZmxpY3RpbmcgaW5mb3JtYXRpb24gd2hlbiBwb3NzaWJsZS4ifV0sImlzc3VlciI6eyJ1cmwiOiJodHRwczovL2V4YW1wbGUub3JnIiwiaW1hZ2UiOiJkYXRhOmltYWdlL3BuZztiYXNlNjQsaVZCT1J3MEtHZ29BQUFBTlNVaEVVZ0FBQUFVQUFBQUZDQVlBQUFDTmJ5YmxBQUFBSEVsRVFWUUkxMlA0Ly84L3czOEdJQVhESUJLRTBESHhnbGpOQkFBTzlUWEwwWTRPSHdBQUFBQkpSVTVFcmtKZ2dnPT0iLCJyZXZvY2F0aW9uTGlzdCI6Imh0dHBzOi8vZXhhbXBsZS5vcmcvcmV2b2tlZC5qc29uIiwibmFtZSI6IkFuIEV4YW1wbGUgQmFkZ2UgSXNzdWVyIiwiZW1haWwiOiJzdGV2ZWRAZXhhbXBsZS5vcmcifX19
.
B-2gMBi-AYLQm2VihGk9YuBZwT_gNwY57SAjNcInnJi4XqbolQArYRpOO-K8WbaRFl10FiMEJ-mkEarJ0VA2TA
```

This token was created using the following key pair.

```
-----BEGIN RSA PRIVATE KEY-----
MIIBPAIBAAJBALLxerJNZXiZmenWOtiU/3G97M2gjFVMXGlS05i8VQPF6XZ2/lyO
vHeooh+YDwmb0x+eSEHOfOvgz4gJuPHuQ1ECAwEAAQJBAJxEZXHwRPzcppyeiSU6
eRlLUtD/s42J8enIeyCW12dCqIxPWCtYUVvwOh1+UM6fOGkGjqxbMZ4nOpV6f2kN
6UECIQDZ8inNOD4yCCqcM0a6hk7Mvy8nMXZ1dK9ZsWM0VIhSWwIhANIv9W3bBsS+
o8i+C/ztSyH3FxDOBwXZar9amYrTTRjDAiAIq3BsQHOA7AA97HBA1TznOie3CGms
7HJZQAwxNbeihwIhALwGoRSEIhrwu83BjTHXCSY6R00GMawe4eqKXt6cxdRHAiEA
qbPrAQt9QmRPpLFIFBpAvQHmb3vDvG13L+JbH0zphRk=
-----END RSA PRIVATE KEY-----
```

```
-----BEGIN PUBLIC KEY-----
MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBALLxerJNZXiZmenWOtiU/3G97M2gjFVM
XGlS05i8VQPF6XZ2/lyOvHeooh+YDwmb0x+eSEHOfOvgz4gJuPHuQ1ECAwEAAQ==
-----END PUBLIC KEY-----
```

The public key corresponding to the private key used to the sign the
badge should be publicly accessible and specified in the `verify.url`
property of the badge assertion.

### Revoking

To mark a badge as revoked, add an entry to the resource pointed at by
the IssuerOrganization `revocationList` URI with the **uid** of the
badge and a reason why the badge is being revoked.

For example, to mark a badge with the uid "abc-1234" as revoked, the
`revocationList` URI would respond with

```json
{"abc-1234" : "Issued in error"}
```

## Badge Verification

An assertion will either be a JWS object (signed assertion) or raw JSON (hosted assertion).

It is STRONGLY RECOMMENDED that issuers use the **signed assertion** format
since it is much more portable than a hosted token.


### Structural Validity

* `badge`: must be a valid **URI** or **BadgeClass** object.
* `recipient`: must be an object
  * `type`: must be a valid type (currently, only "email" is supported)
  * `identity`: must be a **text**
  * `hashed` (optional): must be **boolean**
  * `salt` (optional): must be **text**
* `image` (optional): must be a valid **Data URI** or **URL**.
* `evidence` (optional): must be a valid **URI** or Text
* `issuedOn` (optional): must be a valid [**DateTime**](#datetime)
* `expires` (optional): must be a valid [**DateTime**](#datetime)
* `verify`: must be an object
  * `type`: must be a valid type (currently, only "signed" is supported)
  * `public_key`: must be the public key which can be used to verify the assertion


### Signed Assertion

1. Unpack the JWS payload. This will be a JSON string representation of
the badge assertion.

2. Parse the JSON string into a JSON object. If the parsing operation
fails, assertion MUST be treated as invalid.

3. Assert structural validity.

4. Extract public key from the `verify.public_key` property from the JSON
object. If their is no `verify.public_key` property, or the `verify.public_key`
property does not contain a valid public key the assertion MUST be treated as
invalid.

5. With the public key, perform a JWS verification on the JWS object. If
the verification fails, assertion MUST be treated as invalid.

6. Retrieve the revocation list from the IssuerOrganization object and
ensure the `uid` of the badge does not appear in the list.

7. If the above steps pass, assertion MAY BE treated as valid.
