---
title: Retrieval Formats
keywords: structured, rest, documentreference
tags: [rest,fhir,api,noccprofile]
sidebar: accessrecord_rest_sidebar
permalink: retrieval_formats.html
summary: Support formats for record and document retrieval
---

{% include custom/search.warnbanner.html %}


## Retrieval Formats ##


The NRL may support retrieval of records and documents in a range of formats, this could include both unstructured documents and structured data.

The format of the referenced record is detailed in two meta-data fields:
 - Record format - describes the technical structure and the rules of the document/record and its retrieval route
 - Record mime type - describes the data type of the document/record

See [FHIR Resources & References](explore_reference.html) for further detail on the data model. 

The combination of these two meta-data fields describes to a Consumer system the type and structure of the content that will be returned. This gives the Consumer system the information it needs to know to render the referenced record. For example, whether the referenced content is a publicly accessible web page, an unstructured PDF document or specific FHIR profile. See Retrieval Formats for further detail and the list of currently supported formats. 

## Supported Formats ##

The table below describes the formats that are currently supported:

| Format | Description |
|-----------|----------------|
|HTML Web Page (Publicly accessible)|A publicly accessible HTML web page detailing contact details for retrieving a record.|
|PDF (Publicly accessible)|A publicly accessible PDF detailing contact details for retrieving a record.|
|PDF|A PDF document. For guidance see the [CareConnect GET Binary specification](https://nhsconnect.github.io/CareConnectAPI/api_documents_binary.html).|

Please see the [format code value set](https://fhir.nhs.uk/STU3/ValueSet/NRL-FormatCode-1) for the list of codes to use. 

Consumers and Providers SHOULD support PDF as a minimum.

The prefix of `direct` in a Format code MUST be used for publicly accessible contact details only.

{% include note.html content="Format codes related to profiles that support structured data are not currently listed in the above referenced valueset and table. These will be added in due course." %}

Note that the NRL supports referencing multiple formats of a record document on a single pointer. 

## Multiple Formats ##

Multiple formats of a record or document can be made available through a single pointer on the NRL. For example a pointer can contain a reference to retrieve a record in PDF format and as a structured FHIR resource. Each format must be detailed in a separate content element on the DocumentReference (pointer).

### Multiple Format Example ###
The example below shows a pointer for a Mental Health Crisis Plan that can be retrieved over the phone (using the contact details listed on the referenced HTML web page) and directly as a PDF document.

<div class="github-sample-wrapper scroll-height-350">
{% highlight xml %}
{% include /examples/retrieval_multiple_formats.xml %}
{% endhighlight %}
</div>

## Direct vs Proxy ##
The record format code also indicates whether the record should be retrieved directly (for publicly accessible URLs) or is secured via the SSP. 
Where a record is to be secured via the SSP, in a response to a search or read interaction the NRL will pre-fix the Pointer URL property with the proxy server URL as follows: 

```
https://[proxy_server]/[record_url]
```

## Interaction ID ##

The SSP requires an interaction ID for retrieval of records, which Consumers are required to include in the SSP-InteractionID HTTP header as part of the HTTP request.  

The interaction ID is specific to the format of the record. The association between format codes and interaction IDs are detailed in the table below:

| Format code | Interaction ID |
|-----------|----------------|
|direct:https://www.w3.org/TR/html/|Not applicable (direct retrieval)|
|direct:https://www.iso.org/standard/63534.html|Not applicable (direct retrieval)|
|proxy:https://www.iso.org/standard/63534.html|urn:nhs:names:services:nrls:binary.read|

Endpoints for retrieval will need to be registered under the associated interaction ID on Spine Directory Services (SDS). For further details on registering endpoints, see [Provider Retrieval Guidance](retrieval_provider_guidance.html).  