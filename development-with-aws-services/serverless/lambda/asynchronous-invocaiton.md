# Asynchronous invocaiton

```javascript
export const handler = async (event) => {
  console.log('Received event:', JSON.stringify(event, null, 2));

  const bucketName = event.Records[0].s3.bucket.name;
  const objectKey = decodeURIComponent(event.Records[0].s3.object.key.replace(/\+/g, ' ')); // Decoding URL-encoded key
  const objectSize = event.Records[0].s3.object.size;
  const eventTime = event.Records[0].eventTime;
  const sourceIP = event.Records[0].requestParameters.sourceIPAddress;

  console.log(`Bucket: ${bucketName}`);
  console.log(`Object Key/: ${objectKey}`);
  console.log(`Object Size: ${objectSize}`);
  console.log(`Event Time: ${eventTime}`);
  console.log(`Source IP: ${sourceIP}`);

  // Validate file type
  if (!/\.(jpeg|jpg|png)$/i.test(objectKey)) { // Using regex to check file extension
    console.error(`Invalid file type for object: ${objectKey}`);
    throw new Error('Uploaded file must be of type .jpeg or .png');
  }

  const response = {
    statusCode: 200,
    body: JSON.stringify('S3 event processed successfully!'),
  };

  return response;
};

```

### Destinations

OK: success\_s3\_upload

ERROR: Invalid\_s3\_uploadtype

<figure><img src="../../../.gitbook/assets/Screenshot 2024-11-22 at 21.46.33.png" alt=""><figcaption></figcaption></figure>

### VPC + ENI

```javascript
import dns from 'dns';

export const handler = async (event) => {
    const hostname = event.hostname || 'go.com';
    console.log(`Looking up DNS for hostname: ${hostname}`);

    try {
        const addresses = await new Promise((resolve, reject) => {
            dns.resolve4(hostname, (err, addresses) => {
                if (err) reject(err);
                else resolve(addresses);
            });
        });

        console.log(`Addresses for ${hostname}: ${addresses}`);
        return {
            statusCode: 200,
            body: JSON.stringify({ hostname, addresses }),
        };
    } catch (error) {
        console.error(`DNS lookup failed: ${error.message}`);
        return {
            statusCode: 500,
            body: JSON.stringify({ error: error.message }),
        };
    }
};

```

**Test**

`{ "hostname": "amazon.com" }`

<figure><img src="../../../.gitbook/assets/eni.png" alt=""><figcaption></figcaption></figure>

