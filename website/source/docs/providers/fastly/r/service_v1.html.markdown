---
layout: "fastly"
page_title: "Fastly: aws_vpc"
sidebar_current: "docs-fastly-resource-service-v1"
description: |-
  Provides an Fastly Service
---

# fastly\_service\_v1

Provides a Fastly Service, representing the configuration for a website, app,
api, or anything else to be served through Fastly. A Service encompasses Domains
and Backends.

The Service resource requires a domain name that is correctly set up to direct
traffic to the Fastly service. See Fastly's guide on [Adding CNAME Records][fastly-cname]
on their documentation site for guidance. 

## Example Usage

Basic usage:

```
resource "fastly_service_v1" "demo" {
  name = "demofastly"

  domain {
    name    = "demo.notexample.com"
    comment = "demo"
  }

  backend {
    address = "127.0.0.1"
    name    = "localhost"
    port    = 80
  }

  force_destroy = true
}

```

Basic usage with an Amazon S3 Website, and removes the `x-amz-request-id` header:

```
resource "fastly_service_v1" "demo" {
  name = "demofastly"

  domain {
    name    = "demo.notexample.com"
    comment = "demo"
  }

  backend {
    address = "demo.notexample.com.s3-website-us-west-2.amazonaws.com"
    name    = "AWS S3 hosting"
    port    = 80
  }

  header {
    destination = "http.x-amz-request-id"
    type        = "cache"
    action      = "delete"
    name        = "remove x-amz-request-id"
  }

  gzip {
    name          = "file extensions and content types"
    extensions    = ["css", "js"]
    content_types = ["text/html", "text/css"]
  }

  default_host = "${aws_s3_bucket.website.name}.s3-website-us-west-2.amazonaws.com"

  force_destroy = true
}

resource "aws_s3_bucket" "website" {
  bucket = "demo.notexample.com"
  acl    = "public-read"

  website {
    index_document = "index.html"
    error_document = "error.html"
  }
}
```

**Note:** For an AWS S3 Bucket, the Backend address is
`<domain>.s3-website-<region>.amazonaws.com`. The `default_host` attribute
should be set to `<bucket_name>.s3-website-<region>.amazonaws.com`. See the
Fastly documentation on [Amazon S3][fastly-s3].

## Argument Reference

The following arguments are supported:

* `name` - (Required) The unique name for the Service to create
* `domain` - (Required) A set of Domain names to serve as entry points for your
Service. Defined below.
* `backend` - (Required) A set of Backends to service requests from your Domains.
Defined below.
* `gzip` - (Required) A set of gzip rules to control automatic gzipping of
content. Defined below.
* `header` - (Optional) A set of Headers to manipulate for each request. Defined
below.
* `default_host` - (Optional) The default hostname
* `default_ttl` - (Optional) The default Time-to-live (TTL) for requests
* `force_destroy` - (Optional) Services that are active cannot be destroyed. In
order to destroy the Service, set `force_destroy` to `true`. Default `false`.


The `domain` block supports:

* `name` - (Required) The domain that this Service will respond to
* `comment` - (Optional) An optional comment about the Domain

The `backend` block supports:

* `name` - (Required, string) Name for this Backend. Must be unique to this Service
* `address` - (Required, string) An IPv4, hostname, or IPv6 address for the Backend
* `auto_loadbalance` - (Optional, boolean) Denote if this Backend should be
included in the pool of backends that requests are load balanced against.
Default `true`
* `between_bytes_timeout` - (Optional) How long to wait between bytes in milliseconds. Default `10000`
* `connect_timeout` - (Optional) How long to wait for a timeout in milliseconds.
Default `1000`
* `error_threshold` - (Optional) Number of errors to allow before the Backend is marked as down. Default `0`
* `first_byte_timeout` - (Optional) How long to wait for the first bytes in milliseconds. Default `15000`
* `max_conn` - (Optional) Maximum number of connections for this Backend.
Default `200`
* `port` - (Optional) The port number Backend responds on. Default `80`
* `ssl_check_cert` - (Optional) Be strict on checking SSL certs. Default `true`
* `weight` - (Optional) The [portion of traffic](https://docs.fastly.com/guides/performance-tuning/load-balancing-configuration.html#how-weight-affects-load-balancing) to send to this Backend. Each Backend receives `weight / total` of the traffic. Default `100`

The `gzip` block supports:

* `name` - (Required) A unique name
* `content_types` - (Optional) content-type for each type of content you wish to 
have dynamically gzipped. Ex: `["text/html", "text/css"]`
* `extensions` - (Optional) File extensions for each file type to dynamically 
gzip. Ex: `["css", "js"]`


The `Header` block supports adding, removing, or modifying Request and Response
headers. See Fastly's documentation on 
[Adding or modifying headers on HTTP requests and responses](https://docs.fastly.com/guides/basic-configuration/adding-or-modifying-headers-on-http-requests-and-responses#field-description-table) for more detailed information on any 
of the properties below.

* `name` - (Required) A unique name to refer to this header attribute
* `action` - (Required) The Header manipulation action to take; must be one of
`set`, `append`, `delete`, `regex`, or `regex_repeat`
* `type` - (Required) The Request type to apply the selected Action on
* `destination` - (Required) The name of the header that is going to be affected 
by the Action
* `ignore_if_set` - (Optional) Do not add the header if it is already present. 
(Only applies to `set` action.). Default `false`
* `source` - (Optional) Variable to be used as a source for the header content 
(Does not apply to `delete` action.)
* `regex` - (Optional) Regular expression to use (Only applies to `regex` and `regex_repeat` actions.)
* `substitution` - (Optional) Value to substitute in place of regular expression. (Only applies to `regex` and `regex_repeat`.)
* `priority` - (Optional) Lower priorities execute first. (Default: `100`.)

## Attributes Reference

The following attributes are exported:

* `id` - The ID of the Service
* `name` – Name of this service
* `active_version` - The currently active version of your Fastly Service
* `domain` – Set of Domains. See above for details
* `backend` – Set of Backends. See above for details
* `header` – Set of Headers. See above for details
* `default_host` – Default host specified
* `default_ttl` - Default TTL
* `force_destroy` - Force the destruction of the Service on delete


[fastly-s3]: https://docs.fastly.com/guides/integrations/amazon-s3
[fastly-cname]: https://docs.fastly.com/guides/basic-setup/adding-cname-records
