

========================================
Public API Reference
========================================

--------------
Project Flow
--------------

Pointivo API interaction will generally follow this sequence :

.. graphviz::

   digraph {

	size="6,4"
	node [shape = box, href="#create-project"]; "Create Project";
    node [shape = box, href="#create-resource"]; "Create Resources";
    node [shape = box, href="#wireframe-generation"]; "Submit Request";
    node [shape = box, href="#get-project"]; "Monitor Project Status";
    node [shape = box, href="#get-resources"]; "Download Output Resources";

    "Create Project" -> "Create Resources";
    "Create Resources" -> "Submit Request";
    "Submit Request" -> "Monitor Project Status";
    "Monitor Project Status" -> "Monitor Project Status";
    "Monitor Project Status" -> "Download Output Resources";

   }

Monitoring for project status can be performed by polling the `Get Project`_ endpoint, or by registering a callback URL on the project.   This callback will be called when the project reaches completion.   See `Callbacks`_ for further detail.


============
API Overview
============

The base URL for all API requests is **https://api.pointivo.com/**

All requests to the Pointivo API must include a request header named **authorization** containing a valid API authorization token.   A **content-type** request header must also be set, with value **application/json**.

All responses from the Pointivo API include a common set of fields indicating the status and success of the request:

.. code-block:: javascript
    :caption: API Response

        {
            "success": true, // indicates success or failure of the request
            "message": "", // additional detail, if available
            "data": { } // request-specific response data
        }


A working sample Pointivo API integration is provided  `here <apiclient.html>`_.

=================
Projects
=================

Projects are used to organize resources and outputs for a given customer scene.   Projects will have input resources created via the API, and over the course of processing output resources will be made available via the API.

Projects have a field named **status** that can have one of the following values :

* **CREATED** - Project has been created and is awaiting resources uploads or processing requests
* **PROCESSING** - Project is processing
* **COMPLETE_INITIAL** - Project is processing, but initial outputs are available
* **COMPLETE_FINAL** - Project is complete, all outputs are available, and no further outputs will be generated

Projects also have a field **resourceStatus**, which indicates the most severe problem detected with Resources associated with the Project.   See `Resource Status`_ for further informaiton.

The **measurementUnit** field can be set to control the units used in generated outputs for the project.    Valid values are **M** for meters (default), **FT** for feet, and **FFI** for feet/fractional inches.

--------------
Create Project
--------------

This call creates a new project within the Pointivo API.

The fields shown in the POST call are all optional, but recommended.   If provided the **statusCallbackUrl** will be called by the Pointivo API when the project transitions into a completed state.   Currently supported callback mechanisms are HTTP/HTTPS URLs, and Amazon SNS ARNs.

**POST** /v2/projects

.. code-block:: javascript
    :caption: API Request

        {
            "name": "Project Name",
            "description": "Description",
            "statusCallbackUrl": "http://callback.url",
            "measurementUnit": "M"
        }

The response will include the newly created project, including its assigned id.  This project id will be needed for all subsequent API calls which reference this project.

.. code-block:: javascript
    :caption: API Response

        {
            "success": true,
            "data": {
                "id": 1234,
                "name": "Project Name",
                "description": "Description",
                "statusCallbackUrl": "http://callback.url",
                "status": "CREATED",
                "resourceStatus": "OK",
                "measurementUnit": "M"
            }
        }

.. _getprojectlabel:

--------------
Get Project
--------------

Project data can be retrieved using a GET request :

**GET** /v2/projects/{projectId}

The response will include the current project data :

.. code-block:: javascript
    :caption: API Response

        {
            "success": true,
            "data": {
                "id": 1234,
                "name": "Project Name",
                "description": "Description",
                "statusCallbackUrl": "http://callback.url",
                "status": "CREATED",
                "resourceStatus": "OK",
                "measurementUnit": "M"
            }
        }


--------------
Update Project
--------------

Project data can be updated using this API method.    Only the fields shown below may be modified.

**PUT** /v2/projects/{projectId}

.. code-block:: javascript
    :caption: API Request

        {
            "id": 1234,
            "name": "Modified Project Name",
            "description": "Modified Description",
            "statusCallbackUrl": "http://modified.callback.url",
            "measurementUnit": "FT"
        }

The response will return the modified project data :

.. code-block:: javascript
    :caption: API Response

        {
            "success": true,
            "data": {
                "id": 1234,
                "name": "Modified Project Name",
                "description": "Modified Description",
                "statusCallbackUrl": "http://modified.callback.url",
                "measurementUnit": "FT"
            }
        }


----------------------
Region of Interest
----------------------

A region of interest can be specified for a project.   If provided, any area outside the region will be ignored during processing.

The region of interest is defined as an array of `GeoJSON-style polygons <https://tools.ietf.org/html/rfc7946#section-3.1.6>`_.   Bottom and top values can be provided, which will cap the 3D volume defined by the extruded 2D polygon.   If bottom and top are set to the same value then the region of interest will not be capped.

A region of interest with an empty polygon array will be ignored during processing, effectively disabling the function.

**PUT** /v2/projects/{projectId}/regionOfInterest

.. code-block:: javascript
    :caption: API Request

        {
            "bottom": 0,
            "top": 10,
            "polygons": [
                [
                    [ [ 0, 0 ], [ 0, 4 ], [ 4, 4 ], [ 4, 0 ], [ 0, 0 ] ],
                    [ [ 1, 1 ], [ 1, 3 ], [ 3, 3 ], [ 3, 1 ], [ 1, 1 ] ]
                ]
            ]
        }



The region of interest for a project can be retrieved via the following call :

**GET** /v2/projects/{projectId}/regionOfInterest

.. code-block:: javascript
    :caption: API Response

        {
            "success": true,
            "data": {
                "bottom": 0,
                "top": 10,
                "polygon": [
                    [
                        [ [ 0, 0 ], [ 0, 4 ], [ 4, 4 ], [ 4, 0 ], [ 0, 0 ] ],
                        [ [ 1, 1 ], [ 1, 3 ], [ 3, 3 ], [ 3, 1 ], [ 1, 1 ] ]
                    ]
                ]
            }
        }



=================
Resources
=================

Resources are used to represent file content in the Pointivo API.    A resource must be created on a project before providing its file content to the API, and output content generated for the project is similarly represented as resources.

Once an input resource is defined on a project, the file content may then be uploaded.    The response to the resource create API call includes a temporary URL for the file upload.

-----------------
Resource Status
-----------------

Resources have a **status** field which indicates whether the file content was usable during processing.   The **status** field can have the following values :

* **OK** - There were no issues processing the resource
* **PROBLEM** - A problem with the resource was detected, but the system was able to continue processing
* **UNUSABLE** - The system was unable to process the resource, and the system was unable to continue processing

-----------------
Resource Types
-----------------

The Pointivo API handles a defined set of resource types, each given a unique identifier.

* **FRAME**  - Frame/Image Archive (zip, rar)
* **POINT_DENSE** - Point Cloud (ply, las)
* **CAMERA_VIEWS** - Camera View Definitions (Pointivo, Pix4D, Agisoft)
* **ORTHO_MOSAIC** - Orthomosaic image
* **GEOJSON** - GEOJSON format
* **DXF** - DXF format


.. Point Clouds
.. -------------

.. Point clouds are supported in ASCII and binary PLY format, and in LAS format.

.. A region of interest within the point cloud can be specified by providing a **metadata.regionOfInterest** field in the resource object.   The content of this field must be a GEOJSON Polygon or MultiPolygon object :

.. .. code-block:: javascript
    :caption: API Request

..    {
..      "name": "Pointcloud Resource",
..      "description": "Description",
..      "type": "POINT_DENSE",
..      "metadata": {
..        "regionOfInterest": {
..          "type": "Polygon",
..          "coordinates": [
..            [ [ 100, 0 ], [ 101, 0 ], [ 101, 1 ], [ 100, 1 ], [ 100, 0 ] ]
..          ]
..        }
..      }
..    }


-----------------
Create Resource
-----------------

This call creates a new resource within the Pointivo API.

The only required fields in the create resource endpoint are **type** and **filename**.

**POST** /v2/projects/{projectId}/resources

.. code-block:: javascript
    :caption: API Request

        {
            "name": "Pointcloud Resource",
            "description": "Description"
            "type": "POINT_DENSE" // Point Cloud resource type,
            "filename": "pointDense.ply" // Filename with extension
            "metadata": {} // optional resource metadata
        }

The response will include the newly created resource, including its assigned id.  This resource id will be needed for all subsequent API calls which reference this resource.

.. code-block:: javascript
    :caption: API Response

        {
            "success": true,
            "projectResource": {
                "id": 2345,
                "name": "Pointcloud Resource",
                "description": "Description",
                "type": "POINT_DENSE",
                "filename": "pointDense.ply",
                "flowType": "IN",
                "metadata": {},
                "status": "OK",
            },
            "uploadInfo": {
                "uploadUrl": "https://pointivo-projects.s3.amazonaws.com/1234/in/pointDense.ply?...",
                "key": "1234/in/pointDense.ply",
                "bucket": "pointivo-projects"
            }
        }

The **uploadUrl** field is a temporary URL for uploads.   It is to this URL that the file content associated with this resource should be uploaded to, via a PUT operation.  Further detail on how to perform this upload is provided `here <http://docs.aws.amazon.com/AmazonS3/latest/dev/PresignedUrlUploadObject.html>`_.

Alternatively, resource content can be uploaded using the AWS SDK, which supports > 5GB file sizes and increased performance.    The bucket name and object key for AWS SDK uploads are provided in the **uploadInfo** part of the response.   AWS SDK credentials can be obtained through the `AWS Credentials`_ API call.

The **flowType** field indicates whether the resource was provided to the API, or produced by the API.   Possible values are **IN** and **OUT** respectively.

-----------------
Get Resources
-----------------

This call returns all resources associated with a project.


**GET** /v2/projects/{projectId}/resources


.. code-block:: javascript
    :caption: API Response

        {
            "success": true,
            "data": [
                {
                    "id": 2345,
                    "name": "Pointcloud Resource",
                    "description": "",
                    "type": "POINT_DENSE",
                    "filename": "pointDense.ply",
                    "flowType": "IN",
                    "metadata": {},
                    "status": "OK",
                    "downloadUrl": "https://download.url"
                },
                {
                    "id": 2346,
                    "name": "GEOJSON",
                    "description": "",
                    "type": "GEOJSON",
                    "filename": "wireframe.geojson",
                    "flowType": "OUT",
                    "metadata": {},
                    "status": "OK",
                    "downloadUrl": "https://download.url"
                }
            ]
        }

The **downloadUrl** field is a temporary URL provided to download the file content associated with each resource.


-----------------
AWS Credentials
-----------------

Temporary AWS credentials for project resource S3 uploads can be obtained through the following API call :

**GET** /v2/projects/{projectId}/credentials


.. code-block:: javascript
    :caption: API Response

            {
                "success": true,
                "accessKeyId": "ASIAXXXXXXXX",
                "secretAccessKey": "HKbDrGS+T1OQt4eCa5roRqhg9lcgyFn2jNgx2Z6b",
                "sessionToken": "FQoDYXdzEBkaDLBjiOKQlaIFAQ2XiiK8ArQqN9D",
                "expiration": 1513098783000
            }


These credentials can then be used with the Amazon AWS SDK to upload file content to the S3 locations provided in `Create Resource`_ API call responses.   The AWS SDK must be used if file sizes exceed 5GB.    The AWS SDK may also achieve faster transfer rates through its use of multi-part uploads.

AWS documentation for S3 file uploads can be found `here <http://docs.aws.amazon.com/AmazonS3/latest/dev/uploadobjusingmpu.html>`_.

====================
Process Requests
====================

The Pointivo API supports the generation of various output types based on provided input resources.   These outputs are generated by submitting processing requests specifying the desired output types.

Exterior structure wireframe detection requires that a project have three input resources created and uploaded :

* **FRAME** - Frame/Image Archive (zip, rar)
* **POINT_DENSE** - Pointcloud (ply, las)
* **CAMERA_VIEWS** - Camera View Definitions (Pix4D, Agisoft)

The process request body must include an array field **outputRequests**, containing a list of the desired outputs for the project.

Currently supported output resource types include the following :

* **GEOJSON** - GeoJSON representation of the scene
* **DXF** - AutoDesk DXF file format
* **DATA_PACKAGE** - Raw data and image bundle of resources associated with the scene

**POST** /v2/projects/{projectId}/processRequest

.. code-block:: javascript
    :caption: API Request

        {
            "outputRequests": [
                { "resourceType": "GEOJSON" },
                { "resourceType": "DATA_PACKAGE" }
            ]
        }

Once submitted, processing will begin immediately.   Processing status can be obtained by querying the `Get Project`_ API endpoint.



=================
Callbacks
=================

If a callback is defined for a project, the callback will be invoked once the project reaches a state of **COMPLETE_INITIAL** or **COMPLETE_FINAL**.   The callback body includes the current project data and a list of resources available for the project :


.. code-block:: javascript
    :caption: Callback POST body

        {
          "project": {
            "id": 5847,
            "name": "Project Name",
            "description": "Project Description",
            "statusCallbackUrl": "https://callback.url",
            "resourceStatus": "OK",
            "status": "COMPLETE_INITIAL"
          },
          "resources": [
            {
              "id": 2345,
              "name": "Pointcloud Resource",
              "description": "",
              "type": "POINT_DENSE",
              "filename": "pointDense.ply",
              "flowType": "IN",
              "metadata": {},
              "status": "OK",
              "downloadUrl": "https://download.url"
            },
            {
              "id": 2346,
              "name": "GEOJSON",
              "description": "",
              "type": "GEOJSON",
              "filename": "wireframe.geojson",
              "flowType": "OUT",
              "metadata": {},
              "status": "OK",
              "downloadUrl": "https://download.url"
            }
          ]
        }
