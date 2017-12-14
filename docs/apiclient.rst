Pointivo API Client Example
=============================

.. code-block:: python

    #!/usr/bin/python

    import json
    import requests
    import sys
    import time
    import boto3

    API_TOKEN = '<API token>'
    API_BASE_URL = 'https://api.pointivo.com/'
    HEADERS = {'Authorization': API_TOKEN, 'content-type': 'application/json'}
    AWS_HEADERS = {"content-type": "application/octet-stream"}


    def create_project():
        url = API_BASE_URL + 'v2/projects'
        data = {
            "name": "Project Name",
            "description": "Project Description",
        }
        resp = requests.post(url, data=json.dumps(data), headers=HEADERS, verify=False)
        resp.raise_for_status()
        project_id = resp.json().get('data', {}).get('id')
        return str(project_id)

    def get_project(project_id):
        url = API_BASE_URL + 'v2/projects/' + project_id
        resp = requests.get(url, headers=HEADERS, verify=False)
        resp.raise_for_status()
        return resp.json()

    def get_credentials(project_id):
        url = API_BASE_URL + 'v2/projects/' + project_id + '/credentials'
        resp = requests.get(url, headers=HEADERS, verify=False)
        resp.raise_for_status()
        return resp.json()

    def create_resource(project_id, resource_type, rersource_filename):
        url = API_BASE_URL + 'v2/projects/' + project_id + '/resources'
        data = {
            # "resourceType": resource_type
            "type": resource_type,
            "filename": rersource_filename
        }
        resp = requests.post(url, data=json.dumps(data), headers=HEADERS, verify=False)
        resp.raise_for_status()
        return resp.json()


    def upload_resource_http(datafile, resource_response):
        with open(datafile, mode='rb') as f:
            files = {
                'files': f
            }
            resp = requests.put(resource_response["uploadInfo"]["uploadUrl"], data=f, verify=False, headers=AWS_HEADERS)
            resp.raise_for_status()
            return resp


    def upload_resource_aws(datafile, resource_response, aws_creds):
        s3 = boto3.client("s3",
            aws_access_key_id = aws_creds["accessKeyId"],
            aws_secret_access_key = aws_creds["secretAccessKey"],
            aws_session_token = aws_creds["sessionToken"]
        )
        s3.upload_file(datafile, resource_response["uploadInfo"]["bucket"], resource_response["uploadInfo"]["key"])

    def process_project(project_id):
        url = API_BASE_URL + 'v2/projects/' + project_id + '/processRequest'
        data = {
            "outputRequests": [
                {"resourceType": "GEOJSON"},
                {"resourceType": "DXF"}
            ]
        }
        resp = requests.post(url, data=json.dumps(data), headers=HEADERS, verify=False)
        resp.raise_for_status()
        return resp.json()

    if __name__ == '__main__':

        # create the project
        project_id = create_project()

        # retrieve the project details
        get_project(project_id)

        # upload a camera file
        cams_file = 'test_calibrated_camera_parameters.txt'
        resource_response = create_resource(project_id, 'CAMERA_VIEWS', cams_file.split('/')[-1])
        # upload using vanilla HTTP PUT
        upload_resource_http(cams_file, resource_response)

        # get temporary AWS credentials for the next two uploads
        aws_creds = get_credentials(project_id)

        # upload image zip file
        image_file = 'frames.zip'
        resource_response = create_resource(project_id, 'FRAME', image_file.split('/')[-1])
        # upload using AWS SDK
        upload_resource_aws(image_file, resource_response, aws_creds)

        # upload a point cloud
        points_file = 'PointDense.ply'
        resource_response = create_resource(project_id, 'POINT_DENSE', points_file.split('/')[-1])
        # upload using AWS SDK
        upload_resource_aws(points_file, resource_response, aws_creds)

        # submit processing request
        process_project(project_id)

        while True:
            # poll project status until complete
            print ("Polling for project completion")
            time.sleep(10)
            projectInfo = get_project(project_id)
            if projectInfo['data']['status'] == 'COMPLETE_FINAL':
                break
