# Terraform provider for Google's site verification

A simple provider hitting this API: https://developers.google.com/site-verification

## How to install it?

Quick answer, if you are in an amd64 Linux: download the binary from https://github.com/hectorj/terraform-provider-googlesiteverification/releases and `mv` it to `~/.terraform.d/plugins/terraform-provider-googlesiteverification`.

Long answer: See https://www.terraform.io/docs/extend/how-terraform-works.html#discovery

## Usage

It requires Google credentials to be provided the same way as described as in this document: https://www.terraform.io/docs/providers/google/guides/provider_reference.html#credentials-1

```hcl
# We get the verification token from Google
data "googlesiteverification_dns_token" "example" {
  domain = "yourdomain.example.com"
}

# We put it in our DNS records
# Here is an example with Cloudflare, but you should be able to adapt it to any DNS provider
resource "cloudflare_record" "verification" {
  zone_id = "{your zone ID}"
  name    = data.googlesiteverification_dns_token.example.record_name
  value   = data.googlesiteverification_dns_token.example.record_value
  type    = data.googlesiteverification_dns_token.example.record_type # "TXT" only for now, but it's better to use the data value for future-proofing.
}

# *After* that, we submit our verification request to Google.
# Might take some times, depending on Google's DNS caching.
resource "googlesiteverification_dns" "example" {
  domain = "yourdomain.example.com"
  depends_on = [cloudflare_record.verification] 
}
```
