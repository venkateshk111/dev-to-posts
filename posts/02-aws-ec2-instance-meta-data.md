---
title: Amazon EC2 Instance Metadata
published: false
description: Data about your EC2 instance
tags: 'AWS, ec2, metadata'
cover_image: ./assets/01-aws-site-to-site-vpn/00-aws-site-to-site-vpn-architecture-1000x420-devto.png
canonical_url: null
id: 1987080
---

# Amazon EC2 Instance Metadata

## Instance metadata

- *Instance metadata* is **data about your instance** that you can use to configure or manage the running instance
- *Instance metadata* is accessible from within the instance itself
- The metadata is available at a special URL
    - IPv4
        - `http://169.254.169.254/latest/meta-data/`
    - IPv6
        - `http://[fd00:ec2::254]/latest/meta-data/`

- To get instance id : 

    ```
    curl http://169.254.169.254/latest/meta-data/instance-id
    ```

    ```
    curl http://[fd00:ec2::254]/latest/meta-data/instance-id

    ```
- Instance metadata includes the following:
    - Instance [**metadata properties**](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html#instancedata-data-categories)  : Example `ami-id`, `hostname`, and `local-ipv4`.
    - [**Dynamic data**](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html#dynamic-data-categories) : metadata that's generated when the instance is launched, example : `instance identity document`.
    - **User data** : bootstrap script used during the instance launch

- Instance Metadata Service (IMDS) Versions:
    - **IMDSv1**: A request/response method where you can directly query the metadata URL.
    - **IMDSv2**: A more **secure**, **session-oriented** method that requires a session token for accessing metadata. This version is resistant to SSRF (Server-Side Request Forgery) attacks that can exploit metadata.


## IMDSv2

- IMDSv2 operates with **session-oriented requests**.
- A **session token** is created to define the session duration.
- Session duration can be:
    - Minimum: **1 second**
    - Maximum: **6 hours**
- The same session token can be used for multiple requests during the session duration.
- After the session duration expires, a new session token must be created for future requests.

- Lets try to generate token and fetch instance metadata

    - Generate a token with a valid time to live value (max 6 hours - 21600 seconds):

        ```
        curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 10000"

        ```
    - Then, we re-use the acquired token to send out a request to the IMDS: (assuming I’ve stored the token in the $TOKEN variable)

        ```
        curl -H “X-aws-ec2-metadata-token: $TOKEN” http://169.254.169.254/latest/meta-data/instance-id
        
        ```


## IMDSv1 Vs IMDSv2

| Feature                  | IMDSv1                          | IMDSv2                          |
|--------------------------|---------------------------------|---------------------------------|
| **Access Method**        | Simple request/response         | Session-oriented with token     |
| **Security**             | Basic, vulnerable to SSRF       | Enhanced, mitigates SSRF        |
| **Session Token**        | Not required                    | Required                        |
| **Implementation**       | Direct access via URL           | Requires obtaining a token first|
| **Usage**                | Easier to implement             | More secure, slightly complex   |


- Common Uses:
    - Fetching IAM role credentials for the instance.
    - Retrieving dynamic information such as public and private IP addresses.
    - Bootstrapping and configuring instances using user data


## examples 

- Get the top-level metadata items

    ```
    curl http://169.254.169.254/latest/meta-data/ 

    ```

    ```
    TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"` \
    && curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/    

    ```
- Get AMI ID

    ```
    curl http://169.254.169.254/latest/meta-data/ami-id

    ```

    ```
    curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/ami-id

    ```

## References

- https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html#instancedata-data-categories

- https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html#instance-metadata-retrieval-examples

