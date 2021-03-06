# Enlist a Cloudflare Worker as your Secure Password Scoring and Pwnage Protection API

(Prefer serverless? [See the AWS Lambda version here](https://github.com/detroitenglish/haveibeenpwned-zxcvbn-lambda-api))

Deploy a private, secure and serverless RESTful endpoint for sanely scoring users' new passwords using Dropbox's `zxcvbn` library while (k-)anonymously querying Troy Hunt's [`haveibeenpwned`](https://haveibeenpwned.com/) collection of +5.1 *billion* breached accounts.

![API in Action](.github/pwnage.gif?raw=true "API in Action")

&nbsp;&nbsp;&nbsp;&nbsp;*Example: handling results with [VuetifyJS](https://github.com/vuetifyjs/vuetify)*
<br>
<br>
## Motivation
<a href="https://twitter.com/DetroitEnglish/status/1008276231199055874" target="_blank">People seemed to think this concept was neat</a>. And it turns out, a tiny RESTful API also happens to be a perfect use case for a mighty little **Cloudflare Worker**. And because roughly 92% of `pwnedpassword` queries are cached on Cloudflare's edge network anyways, it's _spooky_ fast! 👻


---

## Quick Start

1. Rename `example.cloudflare.env` to `cloudflare.env` and edit the values as needed.
2. Configure the route matching pattern via the CF browser dash or some other means (route pattern management is not baked-in - yet!)
3. Install deps with `npm install`
4. Launch 🚀 with `npm run deploy`

### Configuration
**NOTE**: You MUST configure your request route matching pattern manually for now! Full auto-deployment with route patterns is in the works...

The following options are configurable via `cloudflare.env`:

- `ALLOWED_ORIGIN`: Whitelisted origin for Cross Origin Resource Sharing. If not provided, all origins are allowed (default: `*`)

- `CORS_MAXAGE`: Value in seconds for the `Access-Control-Max-Age` CORS header (default: `"300"`)

- `ALWAYS_RETURN_SCORE`: Return the `zxcvbn` score even if the `pwnedpasswords` match value is > 0. See [Response](#Response) for details (default: `undefined`)

### Updating

Update configuration à la changes to `cloudflare.env` by re-running `npm run deploy`.

### Request

POST user password input to your route as JSON with field `password` like so:

```javascript
// pwned password
{
  "password": "monkey123"
}
```
```javascript
// stronger password
{
  "password": "wonderful waffles"
}
```

### Response

Our little worker-bro will reply with an appropriate status code, and JSON body with `ok` indicating successful scoring and range search, a strength estimation `score` of 0 through 4 per `zxcvbn`, and `pwned` matches, indicating the number times the input appears in the `haveibeenpwned` database.

```javascript
// pwned password
{
    "ok": true,
    "score": 0,
    "pwned": 56491
}
```
```javascript
// stronger password
{
    "ok": true,
    "score": 3,
    "pwned": 0
}
```
By default, if `pwned` is greater than 0, then `score` will **always** be 0. You can override this behavior by settings `"ALWAYS_RETURN_SCORE"` to `true` in `cloudflare.env`

#### Errors

Failure will return JSON to inform you that something's not `ok` and a `message` as to why.

```json
{
    "ok": false,
    "message": "It went kaput 💩"
}
```

### Good to Know
Send a `GET` request to act as a little health-check. Useful for testing CORS madness 👍


## Because Software

### Disclaimer
I am not affiliated with Cloudflare, Troy Hunt, Dropbox, haveibeenpwned, good software development in general, or any combination thereof.

Handling user passwords is no laughing matter, so handle them with care and respect.

Just like your own users, assume that I have no idea what I'm doing. This part is important, because I have no idea what I'm doing.

**REVIEW THE SOURCE**, and use at your own risk 🙈

### License
MIT
