Create nice directory listings for s3 buckets using only javascript and HTML.

The listing can be deployed on any site and can also be deployed into a bucket.

Inspiration from http://aws.amazon.com/code/Amazon-S3/1713

## Live Demo

If you want to see an example of this script in action check out:

<http://data.openspending.org/>

## Usage

Copy these 3 lines into the HTML file where you want the listing to show up:

    <div id="listing"></div>

    <!-- add jquery - if you already have it just ignore this line -->
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.9.0/jquery.min.js"></script>

    <!-- the JS variables for the listing -->
    <script type="text/javascript">
      // var S3BL_IGNORE_PATH = true;
      // var BUCKET_NAME = 'BUCKET';
      // var BUCKET_URL = 'https://BUCKET.s3-REGION.amazonaws.com';
      // var S3B_ROOT_DIR = 'SUBDIR_L1/SUBDIR_L2/';
    </script>

    <!-- the JS to the do the listing -->
    <script src="https://rgrp.github.io/s3-bucket-listing/list.js"></script>

We've provided an example [index.html][index] file you can just copy if you want.

[index]: https://github.com/rgrp/s3-bucket-listing/blob/gh-pages/index.html


## How it works
The script downloads your XML bucket listing, parses it and simulates a webserver's text-based directory browsing mode.


#### S3BL_IGNORE_PATH variable
Valid options = `false` (default) or `true`

Setting this to false will cause URL navigation to be in this form:
- _`http://data.openspending.org/worldbank/cameroon/`_

You will have to put the html code in your page html AND your error 404 document.

Setting this to true will cause URL navigation to be in this form:
- _`http://data.openspending.org/index.html?prefix=worldbank/cameroon/`_


#### BUCKET_NAME variable
Valid options = `''` (default) or your _bucket name_, e.g.

`BUCKET`

This option is designed to support access to S3 buckets in non-website mode,
via both path-style and virtualhost-style urls. See the [Amazon Documentation](
http://docs.aws.amazon.com/AmazonS3/latest/dev/VirtualHosting.html) for details
on the difference.


#### BUCKET_URL variable
Valid options = `''` (default) or your _bucket URL_, e.g.

`https://BUCKET.s3-REGION.amazonaws.com` (both http & https are valid)

- Do __NOT__ put a trailing '/', e.g. `https://BUCKET.s3-REGION.amazonaws.com/`
- Do __NOT__ put S3 website URL, e.g. `https://BUCKET.s3-website-REGION.amazonaws.com`

This variable tells the script where your bucket XML listing is, and where the files are.
If the variable is left empty, the script will use the same hostname as the _index.html_.


#### S3B_ROOT_DIR variable
Valid options = `''` (default) or `'SUBDIR_L1/'` or `'SUBDIR_L1/SUBDIR_L2/'` or etc.

- Do __NOT__ put a leading '/',     e.g. `'/SUBDIR_L1/'`
- Do __NOT__ omit the trailing '/', e.g. `'SUBDIR_L1'`

This will disallow navigation shallower than your set directory.

Note that this only disallows navigation to shallower directories, but __NOT__ access. Any person with knowledge of the existence of bucket XML listings will be able to manually access those files.

Use Amazon S3 permissions to set granular file permissions.


## Four Valid Configurations
1. Embed into your website
2. Use Amazon S3 in website mode with URL navigation
3. Use Amazon S3 in website mode with prefix mode (ignore_path mode)
4. Use Amazon S3 in non-website mode


#### 1. Embed into your website
Mandatory settings:
```
      var S3BL_IGNORE_PATH = true;
      var BUCKET_URL = 'https://BUCKET.s3-REGION.amazonaws.com';
```
Copy the code into whatever file you want to act as your listing page.


#### 2. Use Amazon S3 in website mode with URL navigation
Mandatory settings:
```
      var S3BL_IGNORE_PATH = false;
      var BUCKET_URL = 'https://BUCKET.s3-REGION.amazonaws.com';
```
- Enable website hosting under `Static website hosting` in your S3 bucket settings.
- Enter `index.html` as your `Index Document` and `Error Document`.
- Put _index.html_ in your bucket.
- Navigate to _`http://BUCKET.s3-website-REGION.amazonaws.com`_ to access the script.

The _`-website-`_ in the URL is important, as the non-website URL is what serves your XML Bucket List.

<http://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteEndpoints.html#WebsiteRestEndpointDiff>

A specific example for the EU west region:

* Website endpoint: _`http://example-bucket.s3-website-eu-west-1.amazonaws.com/`_
* S3 bucket endpoint (for RESTful calls): _`http://example-bucket.s3-eu-west-1.amazonaws.com/`_

Note that US east region is **different** in that the S3 bucket endpoint does not include a location spec but the website version does:

* Website endpoint: _`http://example-bucket.s3-website-us-east-1.amazonaws.com/`_
* S3 bucket endpoint (for RESTful calls): _`http://example-bucket.s3.amazonaws.com/`_


#### 3. Use Amazon S3 in website mode with prefix mode (ignore_path mode)
Mandatory settings:
```
      var S3BL_IGNORE_PATH = true;
      var BUCKET_URL = 'https://BUCKET.s3-REGION.amazonaws.com';
```
- Enable website hosting under `Static website hosting` in your S3 bucket settings.
- Enter `index.html` as your `Index Document` (Error Document is not required).
- Put _index.html_ in your bucket.
- Navigate to _`http://BUCKET.s3-website-REGION.amazonaws.com`_ to access the script.


#### 4. Use Amazon S3 in non-website mode
Mandatory settings:
```
      var S3BL_IGNORE_PATH = true;
      var BUCKET_NAME = 'BUCKET';
```
- Put _index.html_ in your bucket.


## S3 website bucket permissions

You must setup the S3 website bucket to allow public read access. 

* Grant `Everyone` the `List` and `View` permissions:
![List & View permissions](https://f.cloud.github.com/assets/227505/2409362/46c90dbe-aaad-11e3-9dee-10e967763770.png) 
 * Alternatively you can assign the following bucket policy if policies are your thing:

```
{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Sid": "AllowPublicRead",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::{your-bucket-name}/*"
        }
    ]
}
```
* Assign the following CORS policy
```
<CORSConfiguration>
 <CORSRule>
   <AllowedOrigin>*</AllowedOrigin>
   <AllowedMethod>GET</AllowedMethod>
   <AllowedHeader>*</AllowedHeader>
 </CORSRule>
</CORSConfiguration>
```


## Enabling HTTPS
You MUST use config 1 or 4. Amazon S3 doesn't support HTTPS in website mode.

Use https for your BUCKET_URL.

For config 4, navigate to your index.html's full path using https, e.g. _`https://BUCKET.s3-REGION.amazonaws.com/index.html`_

To stop browser warnings about displaying insecure content in secure mode:
- Host the following 3 files in your website/bucket:
  - _list.js_
  - http://ajax.googleapis.com/ajax/libs/jquery/1.9.0/jquery.min.js
  - http://assets.okfn.org/images/icons/ajaxload-circle.gif
- Edit _index.html_ to point to your bucket's `jquery.min.js` & `list.js` file (using relative paths)
- Edit _list.js_ to point to your bucket's `ajaxload-circle.gif`

With config 4, you will then be utilising AmazonAWS' wildcard SSL (unfortunately it is SHA1 only).


### S3 Bucket https only permissions (ie. deny http access)

This is only possible for config 1 or 4.

Set the following bucket policy
```
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "HTTPSOnly",
			"Effect": "Deny",
			"Principal": "*",
			"Action": "s3:*",
			"Resource": "arn:aws:s3:::{your-bucket-name}/*",
			"Condition": {
				"Bool": {
					"aws:SecureTransport": false
				}
			}
		},
		{
			"Sid": "AllowPublicRead",
			"Effect": "Allow",
			"Principal": "*",
			"Action": "s3:GetObject",
			"Resource": "arn:aws:s3:::{your-bucket-name}/*"
		},
		{
			"Sid": "AllowPublicList",
			"Effect": "Allow",
			"Principal": "*",
			"Action": "s3:ListBucket",
			"Resource": "arn:aws:s3:::{your-bucket-name}"
		}
	]
}
```


## Copyright and License

Copyright 2012-2013 Rufus Pollock.

Licensed under the MIT license:

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

