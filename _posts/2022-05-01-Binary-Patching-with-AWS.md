## Binary Patching

In mobile gaming, we deal with a lot of files. The game client, in addition to the application code, can have dozens to thousands of media, data, and asset files. 

<<<<<<< HEAD
This presents two immediate issues when serving files to players. One, is the cost of data transfer for the provider. In AWS for example we pay for "data out" and every client (which can number in the millions) needs to download those files (though this is largely mitigated using CloudFront). Issue two, is the the player and their bandwidth, including additional costs like time, data caps. 
=======
This presents two immediate issues when serving files to players. One, is the cost of data transfer for the provider. In AWS for example we pay for "data out" and every client (which can number in the millions) needs to download those files. Issue two, is the the player and their bandwidth, including additional costs like time, data caps. 
>>>>>>> caf5e6f4ba00b7817d3d55753fef79b8c38c60ce

Common solutions include not doing anything and eating those costs, or using a zip file to deliver the client and unpacking. The following describes a solution for detecting patches using Amazon CloudFront@Edge, in tandem with SQS, and S3. There will be a brief psedocode section on implementing a patching algorithm.

## Solution

The following diagram expresses the sequence of events to detect and generate a patch. The core mechanism being leveraged here is [Cloudfront@Edge Functions](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-cloudfront-trigger-events.html). A cloudformation template for the distribution can be found [here](https://github.com/iancullinane/cloud-templates/blob/master/templates/cloudfront/s3-distribution.yaml), and a cloudformation template for the lambda can be found [here](https://github.com/iancullinane/cloud-templates/blob/master/templates/lambda/sqs-lambda.yaml). 

The basic flow of the above files, is we utilize the s3 distribution event, which will fire a [Lambda@Edge event](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-event-structure.html#example-origin-request) for various actions and requests to the distribution. In our case, we want to trigger a response when a file which has not been cached has been requested. This lambda will add an SQS message to a queue with the necessary information to complete the work. 

![Figure 1](/assets/bin-patching-1.jpg)

One of the main benefits of this design, is the the end user never sees it. Should the patch not exist, nothing will happen and the client can fall back on default behavior. Additionally, since the new patch is only made on request, the process is entirely event driven. It is extremely rare for a file to "go out into the wild" without a patch, because QA generates them through their normal workflow. However should a user request a file that is not yet patched, that is okay because we will make it now and the next user doesn't need to worry.

## Implementation

Now, this does leave out some implementations you will need to work out. For my part I did all of the following using the [AWS SDK for Go](https://docs.aws.amazon.com/sdk-for-go/api/). However the pseudocode would be as follow:

```
where msg = [
  filename,
  current_version,
  requested_version
]

Watch SQS Queue for Messages:
  Receive Message:
    let old = Check s3 curr_version storage for file
    let requested = Check s3 requested_version storage for file
    
    if requested_already_patched() == false {
      generate new patch
      push to s3
    }

    delete_message_from_SQS
```

## The actual patching code is quite simple

If you are familiar with Go, the following is sufficient to generate patches for any file type using an implementation of [bsdiff](https://www.freebsd.org/cgi/man.cgi?query=bsdiff&sektion=1):

```
package patch

import (
	"github.com/gabstv/go-bsdiff/pkg/bsdiff"
)

// CreatePatch will return a byte string containing the diff
func CreatePatch(oldfile, newfile []byte) ([]byte, error) {

	patch, err := bsdiff.Bytes(oldfile, newfile)
	if err != nil {
		return nil, err
	}

	return patch, nil

}
```

**Note:** `bsdiff` will use memory at an exponential rate where exponent value is the file size. So for larger files you need to provide appropriate memory. 
