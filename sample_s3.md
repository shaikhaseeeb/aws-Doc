

# AWS Simple Storage Service - S3


* Amazon S3 is a simple key, value object store designed for the Internet
* S3 provides unlimited storage space and works on the pay as you use model. Service rates gets cheaper as the usage volume increases
* S3 is an Object level storage (not a Block level storage) and cannot be used to host OS or dynamic websites
* S3 resources _for e.g. buckets and objects_ are private by default

## S3 Buckets & Objects

### **Buckets**

* A bucket is a container for objects stored in S3 and help organize the S3 namespace.
* A bucket is owned by the AWS account that creates it and helps identify the account responsible for storage and data transfer charges. Bucket ownership is not transferable
* S3 bucket names are globally unique, regardless of the AWS region in which you create the bucket
* Even though S3 is a global service, buckets are created within a region specified during the creation of the bucket
* Every object is contained in a bucket
* There is no limit to the number of objects that can be stored in a bucket and no difference in performance whether you use many buckets to store your objects or a single bucket to store all your objects
* S3 data model is a flat structure i.e. there are no hierarchies or folders within the buckets. However, logical hierarchy can be inferred using the keyname prefix e.g. Folder1/Object1
* Restrictions 
    * 100 buckets (soft limit) can be created in each of AWS account
    * Bucket names should be globally unique and DNS compliant
    * Bucket ownership is not transferable
    * Buckets cannot be nested and cannot have bucket within another bucket
* You can delete a empty or a non-empty bucket
* S3 allows retrieval of 1000 objects and provides pagination support

### **Objects**

* Objects are the fundamental entities stored in S3 bucket
* Object is uniquely identified within a bucket by a keyname and version ID
* Objects consist of object data, metadata and others 
    * **Key** is object name
    * **Value** is data portion is opaque to S3
    * **Metadata** is the data about the data and is a set of name-value pairs that describe the object _for e.g. content-type, size, last modified_. Custom metadata can also be specified at the time the object is stored.
    * **Version ID **is the version id for the object and in combination with the key helps to unique identify an object within a bucket
    * **Subresources** helps provide additional information for an object
    * **Access Control Information** helps control access to the objects
* Metadata for an object cannot be modified after the object is uploaded and it can be only modified by performing the copy operation and setting the metadata
* Objects belonging to a bucket reside in a specific AWS region never leave that region, unless explicitly copied using Cross Region replication
* Object can be retrieved as a whole or a partially
* With Versioning enabled, you can retrieve current as well as pervious versions of an object

## Bucket & Object Operations

* Listing 
    * S3 allows listing of all the keys within a bucket
    * A single listing request would return a max of 1000 object keys with pagination support using an indicator in the response to indicate if the response was truncated
    * Keys within a bucket can be listed using Prefix and Delimiter.
    * Prefix limits results to only those keys (kind of filtering) that begin with the specified prefix, and delimiter causes list to roll up all keys that share a common prefix into a single summary list result.
* Retrieval 
    * Object can be retrieved as a whole
    * Object can be retrieved in parts or partially (specific range of bytes) by using the Range HTTP header.
    * Range HTTP header is helpful 
        * if only partial object is needed for e.g. multiple files were uploaded as a single archive
        * for fault tolerant downloads where the network connectivity is poor
    * Objects can also be downloaded by sharing Pre-Signed urls
    * Metadata of the object is returned in the response headers
* Object Uploads 
    * Single Operation – Objects of size 5GB can be uploaded in a single PUT operation
    * Multipart upload – can be used for objects of size > 5GB and supports max size of 5TB can is recommended for objects above size 100MB
    * Pre-Signed URLs can also be used shared for uploading objects
    * Uploading object if successful, can be verified if the request received a success response. Additionally, returned ETag can be compared to the calculated MD5 value of the upload object
* Copying Objects 
    * Copying of object up to 5GB can be performed using a single operation and multipart upload can be used for uploads up to 5TB
    * When an object is copied 
        * user-controlled system metadata _e.g. storage class_ and user-defined metadata are also copied.
        * system controlled metadata _e.g. the creation date etc _is reset
    * Copying Objects can be needed 
        * Create multiple object copies
        * Copy object across locations
        * Renaming of the objects
        * Change object metadata _for e.g. storage class, server-side encryption etc_
        * **Updating any metadata for an object requires all the metadata fields to be specified again**
* Deleting Objects 
    * S3 allows deletion of a single object or multiple objects (max 1000) in a single call
    * For Non Versioned buckets, 
        * the object key needs to be provided and object is permanently deleted
    * For Versioned buckets, 
        * if an object key is provided, S3 inserts a delete marker and the previous current object becomes non current object
        * if an object key with a version ID is provided, the object is permanently deleted
        * if the version ID is of the delete marker, the delete marker is removed and the previous non current version becomes the current version object
    * Deletion can be MFA enabled for adding extra security
* Restoring Objects from Glacier 
    * Objects must be restored before you can access an archived object
    * Restoration of an Object can take about 3 to 5 hours
    * Restoration request also needs to specify the number of days for which the object copy needs to be maintained.
    * During this period, the storage cost for both the archive and the copy is charged

### **Pre-Signed URLs**

* All buckets and objects are by default private
* Pre-signed URLs allows user to be able download or upload a specific object without requiring AWS security credentials or permissions
* Pre-signed URL allows anyone access to the object identified in the URL, provided the creator of the URL has permissions to access that object
* Creation of the pre-signed urls requires the creator to provide his security credentials, specify a bucket name, an object key, an HTTP method (GET for download object & PUT of uploading objects), and expiration date and time
* Pre-signed urls are valid only till the expiration date & time

### Multipart Upload

* Multipart upload allows the user to upload a single object as a set of parts. Each part is a contiguous portion of the object's data.
* Multipart uploads supports 1 to 10000 parts and each Part can be from 5MB to 5GB with last part size allowed to be less than 5MB
* Multipart uploads allows max upload size of 5TB (10000 parts * 5GB/part theoretically)
* Object parts can be uploaded independently and in any order. If transmission of any part fails, it can be retransmitted without affecting other parts.
* After all parts of the object are uploaded and complete initiated, S3 assembles these parts and creates the object.
* Using multipart upload provides the following advantages: 
    * Improved throughput – parallel upload of parts to improve throughput
    * Quick recovery from any network issues – Smaller part size minimizes the impact of restarting a failed upload due to a network error.
    * Pause and resume object uploads – Object parts can be uploaded over time. Once a multipart upload is initiated there is no expiry; you must explicitly complete or abort the multipart upload.
    * Begin an upload before the final object size is known – an object can be uploaded as is it being created
* Three Step process 
    * **Multipart Upload Initiation**
        * Initiation of a Multipart upload request to S3 returns a unique ID for each multipart upload.
        * This ID needs to be provided for each part uploads, completion or abort request and listing of parts call.
        * All the Object metadata required needs to be provided during the Initiation call
    * **Parts Upload**
        * Parts upload of objects can be performed using the unique upload ID
        * A part number (between 1 – 10000) needs to be specified with each request which identifies each part and its position in the object
        * If a part with the same part number is uploaded, the previous part would be overwritten
        * After the part upload is successful, S3 returns an ETag header in the response which must be recorded along with the part number to be provided during the multipart completion request
    * **Multipart Upload Completion or Abort**
        * On Multipart Upload Completion request, S3 creates an object by concatenating the parts in ascending order based on the part number and associates the metadata with the object
        * Multipart Upload Completion request should include the unique upload ID with all the parts and the ETag information
        * S3 response includes an ETag that uniquely identifies the combined object data
        * On Multipart upload Abort request, the upload is aborted and all parts are removed. Any new part upload would fail. However, any in progress part upload is completed and hence and abort request must be sent after all the parts upload have been completed
        * S3 should receive a multipart upload completion or abort request else it will not delete the parts and storage would be charged

## Virtual Hosted Style vs Path-Style Request

S3 allows the buckets and objects to be referred in Path-style or Virtual hosted-style URLs

### Path-style

* Bucket name is not part of the domain (unless you use a region specific endpoint)
* the endpoint used must match the region in which the bucket resides
* _for e.g, if you have a bucket called mybucket that resides in the EU (Ireland) region with object named puppy.jpg, the correct path-style syntax URI is http://s3-eu-west-1.amazonaws.com/mybucket/puppy.jpg._
* A "PermanentRedirect" error is received with an HTTP response code 301, and a message indicating what the correct URI is for the resource if a bucket is accessed outside the US East (N. Virginia) region with path-style syntax that uses either of the following: 
    * http://s3.amazonaws.com
    * An endpoint for a region different from the one where the bucket resides. For example, if you use http://s3-eu-west-1.amazonaws.com for a bucket that was created in the US West (N. California) region

### Virtual hosted-style

* S3 supports virtual hosted-style and path-style access in all regions.
* In a virtual-hosted-style URL, the bucket name is part of the domain name in the URL
* for e.g. http://_bucketname_.s3.amazonaws.com/objectname
* S3 virtual hosting can be used to address a bucket in a REST API call by using the HTTP Host header
* Benefits 
    * attractiveness of customized URLs,
    * provides an ability to publish to the "root directory" of the bucket's virtual server. This ability can be important because many existing applications search for files in this standard location.
* S3 updates DNS to reroute the request to the correct location when a bucket is created in any region, which might take time.
* S3 routes any virtual hosted-style requests to the US East (N.Virginia) region, by default, if the US East (N. Virginia) endpoint s3.amazonaws.com is used, instead of the region-specific endpoint (for example, s3-eu-west-1.amazonaws.com) and S3 redirects it with HTTP 307 redirect to the correct region.
* When using virtual hosted-style buckets with SSL, the SSL wild card certificate only matches buckets that do not contain periods.To work around this, use HTTP or write your own certificate verification logic.
* If you make a request to the http://bucket.s3.amazonaws.com endpoint, the DNS has sufficient information to route your request directly to the region where your bucket resides.

## S3 Pricing

* Amazon S3 costs vary by Region
* Charges in S3 are incurred for 
    * Storage – cost is per GB/month
    * Requests – per request cost varies depending on the request type GET, PUT
    * Data Transfer 
        * data transfer in is free
        * data transfer out is charged per GB/month (except in the same region or to Amazon CloudFront)
