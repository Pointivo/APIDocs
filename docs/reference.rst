


Public API Reference
========================================

The base URL for all API requests is **https://api.pointivo.com/**

All requests to the Pointivo API must include a request header named **authorization** containing a valid API authorization token.

All responses from the Pointivo API have a common set of fields indicating the status and success of the request:

.. code-block:: javascript

    {
        "success": true,
        "message": "", // additional detail, if available
        "data": { } // request-specific response data
    }



=================
Projects
=================

Projects are used to organize resources and outputs for a given customer scene.   Projects will have input resources created via the API, and over the course of processing output resources will be made available via the API.

Projects have a field named **status** that can have one of the following values :

* **CREATED** - Project has been created and is awaiting resources uploads or processing requests
* **PROCESSING** - Project is processing
* **COMPLETE_INITIAL** - Project is processing, but initial outputs are available
* **COMPLETE** - Project is complete, and no further outputs will be generated


--------------
Create project
--------------

This call creates a new project within the Pointivo API.

The fields shown in the POST call are all optional, but recommended.   If provided the **statusCallbackUrl** will be called by the Pointivo API when the project transitions into a completed state.   Currently supported callback mechanisms are HTTP/HTTPS urls, and Amazon SNS arns.

**POST** /v2/projects

.. code-block:: javascript

    {
        "name": "Project Name",
        "description": "Description"
        "statusCallbackUrl": "http://callback.url"
    }

The response will include the newly created project, including its assigned id.  This project id will be needed for all subsequent API calls which reference this project.

.. code-block:: javascript

    {
        "success": true,
        "message": null,
        "data": {
            "id": 1234,
            "name": "Project Name",
            "description": "Description",
            "statusCallbackUrl": "http://callback.url",
            "status": "CREATED",
            "resourceStatus": "OK"
        }
    }

.. _getprojectlabel:

--------------
Get project
--------------

Project data can be retrieved using a GET request :

**GET** /v2/projects/{projectId}

The response will include the current project data :

.. code-block:: javascript

    {
        "success": true,
        "message": null,
        "data": {
            "id": 1234,
            "name": "Project Name",
            "description": "Description",
            "statusCallbackUrl": "http://callback.url",
            "status": "CREATED",
            "resourceStatus": "OK"
        }
    }


--------------
Update project
--------------

Project data can be updated using this API method.    Only the fields shown below may be modified.

**PUT** /v2/projects/{projectId}

.. code-block:: javascript

    {
        "id": 1234,
        "name": "Modified Project Name",
        "description": "Description",
        "statusCallbackUrl": "http://callback.url"
    }

The response will return the modified project data :

.. code-block:: javascript

    {
        "success": true,
        "message": null,
        "data": {
            "id": 1234,
            "name": "Modified Project Name",
            "description": "Description",
            "statusCallbackUrl": "http://callback.url"
        }
    }




=================
Resources
=================

Resources are used to represent file content in the Pointivo API.    A resource must be created on a project before providing its file content to the API, and output content generated for the project is similarly represented as resources.

Once an input resource is defined on a project, the file content may then be uploaded.    The response to the resource create API call includes a temporary URL for the file upload.

The Pointivo API handles a defined set of resource types, each given a unique numeric identifier.

* **1  - Frame/Image Archive** (zip, rar)
* **12 - Point Cloud** (ply, las)
* **94 - Camera View Definitions** (Pix4D, Agisoft)
* **96 - GEOJSON**
* **97 - DXF**


-----------------
Create resource
-----------------

This call creates a new resource within the Pointivo API.

The only required field in the create resource endpoint is **resourceType**.

**POST** /v2/projects/{projectId}/resources

.. code-block:: javascript

    {
        "name": "Pointcloud Resource",
        "description": "Description"
        "resourceType": { id: 12 } // Point Cloud resource type
    }

The response will include the newly created resource, including its assigned id.  This resource id will be needed for all subsequent API calls which reference this resource.

.. code-block:: javascript

    {
        "success": true,
        "message": null,
        "data": {
            "id": 2345,
            "name": "Pointcloud Resource",
            "description": "Description",
            "resourceType": { id: 12 },
            "status": "OK",
        },
        "uploadUrl": "https://upload.here"
    }

The response includes a field named **uploadUrl**.   It is to this URL that the file content associated with this resource should be uploaded to, via a POST operation.  Further detail on how to perform this upload is provided `here <http://docs.aws.amazon.com/AmazonS3/latest/dev/PresignedUrlUploadObject.html>`_.


===================
Wireframe Detection
===================

The Pointivo API supports automatic wireframe detection for structures in point clouds.   Wireframe detection requires that a project have three input resources created and uploaded :

* **1 - Frame/Image Archive** (zip, rar)
* **12 - Point Cloud** (ply, las)
* **94 - Camera View Definitions** (Pix4D, Agisoft)


The wireframe detection request must include the resource ids for all three resources.

**POST** /v2/projects/{projectId}/wireframe

.. code-block:: javascript

    {
        "frameZipResourceId": 1001,
        "pointCloudResourceId": 1002,
        "cameraViewResourceId": 1003
    }

Once submitted, processing will begin immediately.   Project status can be obtained by querying the `Get project`_ API endpoint.




=================
Callbacks
=================

If a callback is defined for a project, the callback will be invoked once the project reaches a state of **COMPLETED_INITIAL** or **COMPLETED**.   The callback body includes the current project data and a list of resources available for the project :

