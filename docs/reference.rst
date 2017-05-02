


Public API Reference
========================================

The base URL for all API requests is **https://api.pointivo.com/**

All requests to the Pointivo API must include a request header named **authorization** containing a valid API authorization token.


=================
Projects
=================

Projects are used to organized resources and outputs for a given customer scene.   Projects will have input resources created via the API, and over the course of processing output resources will be made available via the API.

Projects have a field named *status* that can have one of the following values :

* **CREATED** - Project has been created and is waiting for resources or processing requests
* **PROCESSING** - Project is processing
* **COMPLETE_INITIAL** - Project is processing, but initial outputs are available
* **COMPLETE** - Project is complete, and no further outputs will be generated


--------------
Create project
--------------

This call creates a new project within the Pointivo API.

The fields shown in the POST call are all optional, but recommended.   If provided the *statusCallbackUrl* will be called by the Pointivo API when the project transitions into a completed state.   Currently supported callback mechanisms are HTTP/HTTPS urls, and Amazon SNS arns.

*POST* /v2/projects

.. code-block:: javascript

    {
        "name": "Project Name",
        "description": "Description"
        "statusCallbackUrl": "http://callback.url"
    }

The response will include the newly created project, including it's assigned id.  This project id will be needed for all subsequent API calls which reference this project.

.. code-block:: javascript

    {
        "id": 1234,
        "name": "Project Name",
        "description": "Description",
        "statusCallbackUrl": "http://callback.url",
        "status": "CREATED",
        "resourceStatus": "OK"
    }

--------------
Update project
--------------

Project data can be updated using this API method.    Only the fields shown below may be modified.

*PUT* /v2/projects/{projectId}

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
        "id": 1234,
        "name": "Modified Project Name",
        "description": "Description",
        "statusCallbackUrl": "http://callback.url"
    }




=================
Create Resource
=================
Project resources may be defined via this API call.

*POST* /v2/projects/{projectId}/resources

.. code-block:: javascript

    {
        "name": "Modified Project Name",
        "description": "Description",
        "resourceType": {

        }
    }




=================
Callbacks
=================
Callbacks