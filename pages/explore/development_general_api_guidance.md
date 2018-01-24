---
title: General API guidance
keywords: fhir development
tags: [fhir,development]
sidebar: overview_sidebar
permalink: development_general_api_guidance.html
summary: "Implementation guidance for developers - focusing on general API implementation guidance"
---

## Purpose ##

This implementation guide is intended for use by software developers looking to build a conformant NRLS API interface using the FHIR&reg; standard with a focus on general API implementation guidance.

### Notational conventions ###

The keywords ‘**MUST**’, ‘**MUST NOT**’, ‘**REQUIRED**’, ‘**SHALL**’, ‘**SHALL NOT**’, ‘**SHOULD**’, ‘**SHOULD NOT**’, ‘**RECOMMENDED**’, ‘**MAY**’, and ‘**OPTIONAL**’ in this implementation guide are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).


<!--
## Maturity roadmap ##

At a high level, the maturity roadmap of a compliant principal GP system is expected to follow the following FHIR and business capability maturity stages.

Refer to [Design - design principles - maturity model](designprinciples_maturity_model.html) for full details.

## General standards ##

Information on the technical standards that SHALL be conformed to can be found in the sections below and throughout the GP Connect specification.

{% include important.html content="Any additional principles highlighted in the GP Connect specification MUST take precedence over those defined in these technical standards." %}

## Internet standards ##

Clients and servers SHALL be conformant to the following Internet Engineering Task Force (IETF) request for comments (RFCs), which are the principal technical standards that underpin the design and development of the internet and thus FHIR's APIs.

- transport level integration SHALL be via HTTP as defined in the following RFCs: [RFC 7230](https://tools.ietf.org/html/rfc7230), [RFC 7231](https://tools.ietf.org/html/rfc7231), [RFC 7232](https://tools.ietf.org/html/rfc7232), [RFC 7233](https://tools.ietf.org/html/rfc7233), [RFC 7234](https://tools.ietf.org/html/rfc7234) and [RFC 7235](https://tools.ietf.org/html/rfc7235)
- transport level security SHALL be via TLS/HTTPS as defined in [RFC 5246](https://tools.ietf.org/html/rfc5246) and [RFC 6176](https://tools.ietf.org/html/rfc6176)
- HTTP Strict Transport Security (HSTS) as defined in [RFC 6797](https://tools.ietf.org/html/rfcy 6797) SHALL be employed to protect against protocol downgrade attacks and cookie hijacking

{% include roadmap.html content="NHS Digital is currently evaluating how [cross-origin resource sharing](http://www.w3.org/TR/cors/) (CORS) will be handled for web and mobile based applications." %}

## Endpoint resolution ##

Clients SHALL perform a sequence of query operations against existing Spine services to enable FHIR endpoint resolution.

1. Clients SHALL perform (or have previously performed) a Personal Demographics Service (PDS) lookup for a patient.
	1. Using the PDS results, the client SHALL determine the patient's primary GP organisation. 
2. Clients SHALL perform (or have previously performed) a Spine Directory Service (SDS) lookup using the Organisation Data Service (ODS) code of the patient's primary GP organisation.
	1. Using the SDS results the client SHALL determine the principal GP system responsible for hosting the most up to date GP care record.
		1. [EMIS Health](http://www.emishealth.com/)
		2. [INPS](http://www.inps.co.uk/)
		3. [Microtest](http://www.microtest.co.uk/)
		4. [TPP](http://www.tpp-uk.com/)
2. Clients SHALL construct a [FHIR service root URL](#ServiceRootURL) suitable for access to a GP vendor's FHIR server. For GP Connect, access to the principal GP systems will be via the [Spine Security Proxy](#SpineSecurityProxy) (SSP) and as such the URL will need to be pre-pended with a proxy service root URL.

{% include tip.html content="Where a practitioner (with a valid SDS user ID) or organisation (with a valid ODS code) record already exists within the local system, the details associated with these existing records may be used for display purposes." %}
-->

## RESTful API ##

<!--
The [RESTful API](https://www.hl7.org/fhir/STU3/http.html) described in the FHIR&reg; standard is built on top of the Hypertext Transfer Protocol (HTTP) with the same HTTP verbs (`GET`, `POST`, `PUT`, `DELETE`, and so on) commonly used by web browsers. Furthermore, FHIR exposes resources (and operations) as Uniform Resource Identifiers (URIs). For example, a `Patient` resource `/fhir/Patient/1`, can be operated upon using standard HTTP verbs such as `DELETE /fhir/Patient/1` to remove the patient record.

The FHIR RESTful API style guide defines the following URL conventions which are used throughout the remainder of this page:

- URL pattern content surrounded by **[ ]** are mandatory
- URL pattern content surrounded by **{ }** are optional

### Service root URL ###

The [service root URL](https://www.hl7.org/fhir/STU3/http.html#general) is the address where all the resources defined by this interface are found. 

The service root URL is the `[base]` portion of all FHIR APIs.

{% include important.html content="All URLs (and IDs that form part of the URL) defined by this specification are case sensitive." %}

### FHIR API Versioning ###
FHIR APIs SHALL be versioned according to [Semantic Versioning](http://semver.org/spec/v2.0.0.html) in the server's service root URL to provide a clear distinction between API versions that are incompatible (that is, contain breaking changes) versus backwards-compatible (that is, contain no breaking changes).

Provider systems are required to publish service root URLs for major versions of FHIR APIs in the Spine Directory Service in the following format:

{% include callout.html content="`https://[FQDN of FHIR Server]/[ODS_CODE]/[FHIR_VERSION_NAME]/{API_MAJOR_VERSION}/{PROVIDER_ROUTING_SEGMENT}`" %}


- `[FQDN_OF_FHIR_SERVER]` is the fully qualified domain name where traffic will be routed to the logical FHIR server for the organisation in question

- `[ODS_CODE]` is the [Organisation Data Service](https://digital.nhs.uk/organisation-data-service) code which uniquely identifies the GP practice organisation

- `[FHIR_VERSION_NAME]` refers to the textual name identifying the major FHIR version, examples being `DSTU2` and `STU3`. The FHIR version name SHALL be specified in upper case characters

- `{API_MAJOR_VERSION}` identifies the major version number of the provider API. Where the provider API version number is omitted, the major version SHALL be assumed to be 1

- `{PROVIDER_ROUTING_SEGMENT}` enables providers to differentiate between GP Connect and non-GP Connect requests (for example, via a load balancer). If included, this optional provider routing segment SHALL be static across all the provider's GP Connect API endpoints.
  
- The FHIR base URL SHALL NOT contain a trailing `/`

#### Example server root URL

The provider will publish the server root URL to Spine Directory Services as follows:

`https://provider.nhs.uk/GP0001/DSTU2/2/gpconnect`

Consumer systems are required to construct a [service root URL containing the SSP URL followed by the FHIR Server Root URL of the logical practice FHIR server](integration_spine_security_proxy.html#proxied-fhir-requests) that is suitable for interacting with the SSP service. API provider systems will be unaware of the SSP URL prefix as this will be removed prior to calling the provider API endpoint.

The consumer system would therefore issue a request to the new version of the provider FHIR API to the following URL:

`https://[ssp_fqdn]/https://provider.nhs.uk/GP0001/STU3/2`


### Resource URL ###

The [Resource URL](http://www.hl7.org/implement/standards/fhir/STU3/http.html) will be in the following format:

	VERB [base]/[type]/[id] {?_format=[mime-type]}

Clients and servers constructing URLs SHALL conform to [RFC 3986 Section 6 Appendix A](https://tools.ietf.org/html/rfc3986#appendix-A) which requires percent-encoding for a number of characters that occasionally appear in the URLs (mainly in search parameters).

### HTTP verbs ###

The following [HTTP verbs](http://hl7.org/fhir/STU3/valueset-http-verb.html) SHALL be supported to allow RESTful API interactions with the various FHIR resources:

- **GET**
- **POST**
- **PUT**
- **DELETE**

{% include tip.html content="Please see later sections for which HTTP verbs are expected to be available for specific FHIR resources." %}

<p/>

{% include roadmap.html content="In a future version of the FHIR&reg; standard, it is expected that the **PATCH** verb will also be supported." %}

#### Resource types ####

GP Connect provider systems SHALL support FHIR [resource types](http://hl7.org/fhir/STU3/resourcelist.html) as profiled within the [GP Connect FHIR Resource Definitions](http://developer.nhs.uk/downloads-data/fhir-resource-definitions-library/). 

#### Resource ID ####

This is the [logical Id](http://hl7.org/fhir/STU3/resource.html#id) of the resource which is assigned by the server responsible for storing it. The logical identity is unique within the space of all resources of the same type on the same server, is case sensitive and can be up to 64 characters long.

Once assigned, the identity SHALL never change. `logical Ids` are always opaque, and external systems need not and should not attempt to determine their internal structure.

{% include important.html content="As stated above and in the FHIR&reg; standard, `logical Ids` are opaque and other systems should not attempt to determine their structure (or rely on this structure for performing interactions). Furthermore, as they are assigned by each server responsible for storing a resource they are usually implementation specific. For example, NoSQL document stores typically preferring a GUID key (for example, 0b28be67-dfce-4bb3-a6df-0d0c7b5ab4) while a relational database stores typically preferring a integer key (for example, 2345)." %} 

For further background, refer to principles of [resource identity as described in the FHIR standard](http://www.hl7.org/implement/standards/fhir/STU3/resource.html#id)  

#### External resource resolution ####

In line with work being undertaken in other jurisdictions (see the [Argonaut Implementation Guide](http://argonautwiki.hl7.org/index.php?title=Implementation_Guide) for details) GP Connect provider systems are not expected to resolve full URLs that are external to their environment.
-->
### Content types ###

- The NRLS Server SHALL support both formal [MIME-types](https://www.hl7.org/fhir/STU3/http.html#mime-type) for FHIR resources:
  - XML: `application/fhir+xml`
  - JSON: `application/fhir+json`
  
- The NRLS Server SHALL also support DSTU2 [MIME-types](https://www.hl7.org/fhir/DSTU2/http.html#mime-type) for backwards compatibility:
  - XML: `application/xml+fhir`
  - JSON: `application/json+fhir`
  
- The NRLS Server SHALL also gracefully handling generic XML and JSON MIME types:
  - XML: `application/xml`
  - JSON: `application/json`
  - JSON: `text/json`
  
- The NRLS Server SHALL support the optional `_format` parameter in order to allow the client to specify the response format by its MIME-type. If both are present, the `_format` parameter overrides the `Accept` header value in the request.

<!--- The NRLS Server SHALL prefer the encoding specified by the `Content-Type` header if no explicit `Accept` header has been provided by a client system.-->

- If neither the `Accept` header nor the `_format` parameter are supplied by the client system the NRLS Server SHALL return data in the default format of `application/fhir+json`.


<!--
### Wire format representations ###

Servers should support two [wire formats](https://www.hl7.org/fhir/STU3/formats.html#wire) as ways to represent resources when they are exchanged:

- Servers SHALL support [JSON](https://www.hl7.org/fhir/STU3/json.html)
- Servers SHOULD support [XML](https://www.hl7.org/fhir/STU3/xml.html)

{% include important.html content="The FHIR standard outlines specific rules for formatting XML and JSON on the wire. It is important to read and understand in full the differences between how XML and JSON are required to be represented." %}

Consumers SHALL ignore unknown extensions and elements in order to foster [forwards compatibility](https://www.hl7.org/fhir/STU3/compatibility.html#1.10.3) and declare this by setting [CapabilityStatement.acceptUnknown](https://www.hl7.org/fhir/STU3/capabilitystatement-definitions.html#CapabilityStatement.acceptUnknown) to 'both' in their capability statement.

Systems SHALL declare which format(s) they support in their CapabilityStatement. If a server receives a request for a format that it does not support, it SHALL return an HTTP status code of `415` indicating an `Unsupported Media Type`.

### Transfer encoding ###

Clients and servers SHALL support the HTTP [Transfer-Encoding](https://www.hl7.org/fhir/STU3/http.html#mime-type) header with a value of `chunked`. This indicates that the body of a HTTP response will be returned as an unspecified number of data chunks (without an explicit `Content-Length` header).

### Character encoding ###

Clients and servers SHALL support the `UTF-8` [character encoding](https://www.hl7.org/fhir/STU3/http.html#mime-type) as outlined in the FHIR standard.

> FHIR uses `UTF-8` for all request and response bodies. Since the HTTP specification (section 3.7.1) defines a default character encoding of `ISO-8859-1`, requests and responses SHALL explicitly set the character encoding to `UTF-8` using the `charset` parameter of the MIME-type in the `Content-Type` header. Requests MAY also specify this charset parameter in the `Accept` header and/or use the `Accept-Charset` header.

### Content compression ###

To improve system performances clients/servers SHALL support GZIP compression.

Compression is requested by setting the `Accept-Encoding` header to `gzip`.

{% include tip.html content="Applying content compression is key to reducing bandwidth needs and improving battery life for mobile devices." %} 

### [Inter-version compatibility](https://www.hl7.org/fhir/STU3/compatibility.html) ###

Unrecognized search criteria SHALL always be ignored. As search criteria supported in a query are echoed back as part of the search response there is no risk in ignoring unexpected search criteria.

### HTTP headers ###

#### Proxying headers ####

Additional HTTP headers SHALL be added into the HTTP request/response for allowing the proxy system to disclose information lost in the proxying process (for example, the originating IP address of a request). Typically, this information is added to proxy forwarding headers as defined in [RFC 7239](http://tools.ietf.org/html/rfc7239).

#### Cross-organisation provenance and audit headers ####

To meet auditing and provenance requirements (which are expected to be closely aligned with the IM1 requirements), clients SHALL provide an oAuth 2.0 Bearer token in the HTTP Authorization header (as outlined in [RFC 6749](http://tools.ietf.org/html/rfc6749)) in the form of a JSON Web Token (JWT) as defined in [RFC 7519](http://tools.ietf.org/html/rfc7519).

{% include tip.html content="We are using an open standard (JWT) to provide a container for the provenance and audit data for ease of transport between the consumer and provider systems. It is important to note that these tokens (for GP Connect FoT) will **not** be centrally issued and are not signed or encrypted (that is, they are constructed of plain text). There are JWT libraries available for most programming languages simplifying the generation of this data in JWT format." %}

Refer to [Integration - cross-organisation audit and provenance](integration_cross_organisation_audit_and_provenance) for full details of the JWT claims that SHALL be used for passing audit and provenance details between systems.

{% include important.html content="We have defined a small number of additional headers which are also required to be included in NHS Digital defined custom headers." %}

Clients SHALL add the following Spine proxy headers for audit and security purposes:

- `Ssp-TraceID` - TraceID (generated per request) which identifies the sender's message/interaction (for example, a GUID/UUID).
- `Ssp-From` - ASID which identifies the sender's FHIR endpoint.
- `Ssp-To` - ASID which identifies the recipient's FHIR endpoint.
- `Ssp-InteractionID` - identifies the FHIR interaction that is being performed <sup>1</sup>

<sup>1</sup> please refer to the [Development - FHIR API guidance - operation guidance](development_fhir_operation_guidance.html) for full details.

The SSP SHALL perform the following checks to authenticate client request:

- get the common name (CN) from the TLS session and compare the host name to the declared endpoint
- check that the client/sending endpoint has been registered (and accredited) to initiate the given interaction
- check that the server/receiving endpoint has been registered (and accredited) to receive/process the given interaction   

#### Caching headers ####

Providers SHALL use the following HTTP header to ensure that no intermediaries cache responses: `Cache-Control: no-store`


### [Managing Return Content](https://www.hl7.org/fhir/STU3/http.html#return) ###

Provider SHALL maintain resource state in line with the underlying system, including the state of any associated resources.

For example: 

_If the practitioner associated with a schedule is changed on the provider's system, such as when a locum is standing in for a regular doctor, this should be reflected in all associated resources to that schedule. The diagram below shows the expected change to the appointment resources for this scenario._

_When the appointment is booked, the appointment resource is associated with a slot resource and references the practitioner resource associated with the schedule in which the slot resides. If the schedule is then updated within the provider system to reflect the change of practitioner from the original doctor to a locum doctor, then the practitioner reference with the schedule will be updated. If a consumer then performs a read of the appointment the returned appointment resource should reflect the updated practitioner on the schedule._
-->

<!--[Diagram of reflection of state](images/development/Reseource Reflection of state.png)-->

<!--
Servers SHALL default to the `return=representation` behaviour (that is, returning the entire resource) for interactions that create or update resources.

Servers SHOULD honour a `return=minimal` or `return=representation` preference indicated in the `Prefer` request header, if present.

### Demographic cross-checking ###

Consumer systems SHALL compare the returned structured patient demographic data (supplied by the provider system as structured data) against the demographic data held in the consumer system.

The following data SHALL be cross-checked between consumer and returned provider data. Any differences between these fields SHALL be brought to the attention of the user.   

| Item | Resource field |
| ---- | -------------- | 
| Family name | patient.name.family |
| Given name | patient.name.given |
| Gender | patient.gender |
| Birth date | patient.birthDate |

Additionally, the following data MAY be displayed if returned from the provider to assist a visual cross-check and for safe identification, but should not be part of the automatic comparison:
* Address and postcode
* Contact (telephone, mobile, email)

All above may be redacted if patient is flagged on Spine as sensitive demographics.

### Managing resource contention ###

To facilitate the management of [resource contention](http://hl7.org/fhir/STU3/http.html#concurrency), servers SHALL always return an `ETag` header with each resource including the resource’s `versionId`:

```http
HTTP 200 OK
Date: Sat, 09 Feb 2013 16:09:50 GMT
Last-Modified: Sat, 02 Feb 2013 12:02:47 GMT
ETag: W/"23"
Content-type: application/json+fhir
```

`ETag` headers which denote resource `version Id`s SHALL be prefixed with `W/` and enclosed in quotes, for example:

```http
ETag: W/"3141"
```

Clients SHALL submit update requests with an `If-Match` header that quotes the `ETag` from the server.

```http
PUT /Patient/347 HTTP/1.1
If-Match: W/"23"
```

If the `version Id` given in the `If-Match` header does not match, the server returns a `409` **Conflict** status code instead of updating the resource.

For servers that don't persist historical versions of a resource (that is, any resource other than the currently available/latest version) then they SHALL operate in line with the guidance provided in the following [Hay on FHIR - FHIR versioning with a non-version capable back-end](https://fhirblog.com/2013/11/21/fhir-versioning-with-a-non-version-capable-back-end/) blog post. This is to ensure that GP Connect servers will be compatible with version-aware clients, even though the server itself doesn't support the retrieval of historical versions.

### Managing return errors ###

To [manage return errors](http://hl7.org/fhir/STU3/http.html#2.1.0.4), FHIR defines an [OperationOutcome](http://hl7.org/fhir/STU3/operationoutcome.html) resource that can be used to convey specific detailed processable error information. An `OperationOutcome` may be returned with any HTTP `4xx` or `5xx` response, but is not always required.
-->