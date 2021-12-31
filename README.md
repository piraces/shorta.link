# shorta.link

Custom link shortener service using Cloudflare Workers + KV store on your domain. The Workers free tier is quite generous and perfectly suited for this since KV is optimized for high reads and infrequent writes, which is our use case.

This API is easily consumed programmatically, such as through shell functions, making it trivial to shorten links on the fly.

⚡ TBD: Additionally, it is possible to make a simple form to create short links right from a webpage if that is more to your liking. 

## Usage

### Creating short links

Send a POST request with form data `url` and (optional) `slug` to redirect [shorta.link](https://shorta.link) to url.

For authentication, pass an API key in a `X-Api-Key` header.

If `slug` exists already, the value will NEVER be overwritten.

#### API Example

```bash
curl --location --request POST "https://shorta.link" \
    -H "X-Api-Key: your api key goes here" \
    -H "Content-Type: application/x-www-form-urlencoded" \
    --data-urlencode "url=$URL" \
    --data-urlencode "slug=$NAME"
```
passing https://github.com/piraces to url, and gh to slug will make https://shorta.link/gh redirect to it (this is a real example).

### Deleting short links

Send DELETE request to the shortlink which should be deleted.

Authentication is required; pass the API key in the `X-Api-Key` header.

This method is idempotent, being that successive attempts to delete an already-deleted shortlink
will result in status 200 (OK).

#### API Example

```bash
curl --location --request DELETE "https://shorta.link/gh" \
    -H "X-Api-Key: ${SECRET_API_KEY}"
```

Will delete the shortlink available at https://shorta.link/gh

### Listing short links

Sending an authenticated GET request to the shorta.link domain root will respond with a list of all
shortlinks maintained by the service.

Authentication is required; pass the API key in the `X-Api-Key` header.

#### API Example

```bash
curl --location --request GET "https://shorta.link" \
    -H "X-Api-Key: ${SECRET_API_KEY}"
```

Will return a JSON array of keys with one to three properties:

```json
  [{ name: "gh", expiration: null, metadata: "https://github.com/piraces"}, ...]
```
(From https://developers.cloudflare.com/workers/runtime-apis/kv#more-detail)

`expiration` and `metadata` are optional.

### Consuming

This is the easy part, simply open the shortened link in your browser of choice!

## Deploying

Automatically deploys to Cloudflare Workers on push using GitHub Actions. You will only need to modify the account and kv namespace values in [wrangler.toml](wrangler.toml), and set the repo secrets `CF_API_TOKEN` and `SECRET_API_KEY` (this is the preshared header authentication key) used in the [workflow](.github/workflows/main.yml). 

Oh, and run the worker on the route you want your shortener service to be on of course.

# License

Licensed under [MIT license](LICENSE).

# Acknowledgments

Based on the awesome work by Aadi Bajpai <hello@aadibajpai.com> in [VandyHacks/vhl.ink](https://github.com/VandyHacks/vhl.ink).