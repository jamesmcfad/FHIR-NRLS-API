---
title: Pointer Types
keywords: structured rest documentreference
tags: [fhir,pointers,record_retrieval]
sidebar: accessrecord_rest_sidebar
permalink: supported_pointer_types.html
summary: NRL supported pointer types.
---

The following table outlines the currently supported NRL information categories and types; this list will be updated as other information types are supported by the NRL. The codes for each information category and type can be found in the relevant value sets detailed on the [FHIR Resource Mapping](pointer_fhir_resource.html) page.

The table outlines each `Retrieval Format` supported for each pointer type. Details about the retrieval formats can be found on the [information retrieval overview](information_retrieval_overview.html) page.

<table style="width:100%;">
    <tr>
        <th><a href="pointer_fhir_resource.html#information-category">Information Category</a></th>
        <th><a href="pointer_fhir_resource.html#information-type">Information Type</a></th>
        <th><a href="pointer_fhir_resource.html#retrieval-format">Retrieval Format(s)</a></th>
	  </tr>
    <tr>
        <td rowspan="3">Care plan</td>
        <td>End of life care coordination summary</td>
        <td>
            <ul>
                <li>
                    <a href="retrieval_contact_details.html">"Contact Details"</a>
                </li>
                <li>
                    <a href="retrieval_unstructured_document.html">"Unstructured Document (PDF)"</a>
                </li>
            </ul>
        </td>
    </tr>
    <tr>
        <td>Mental health crisis plan</td>
        <td>
            <ul>
                <li>
                    <a href="retrieval_contact_details.html">"Contact Details"</a>
                </li>
                <li>
                    <a href="retrieval_unstructured_document.html">"Unstructured Document (PDF)"</a>
                </li>
            </ul>
        </td>
    </tr>
    <tr>
        <td>Emergency health care plan</td>
        <td>
            <ul>
                <li>
                    <a href="retrieval_contact_details.html">"Contact Details"</a>
                </li>
                <li>
                    <a href="retrieval_unstructured_document.html">"Unstructured Document (PDF)"</a>
                </li>
            </ul>
        </td>
    </tr>
    <tr>
        <td rowspan="2">Record headings</td>
        <td>Services and care</td>
        <td>
            <ul>
                <li>
                    <a href="retrieval_contact_details.html">"Contact Details"</a>
                </li>
                <li>
                    <a href="retrieval_unstructured_document.html">"Unstructured Document (PDF)"</a>
                </li>
            </ul>
        </td>
    </tr>
    <tr>
        <td>Immunisations</td>
        <td>
            <ul>
                <li>
                    <a href="retrieval_vaccinations_fhir_stu3.html">"Vaccination List - FHIR STU3 v1"</a>
                </li>
            </ul>
        </td>
    </tr>
</table>