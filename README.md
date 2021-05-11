# ifit-brightai-tv-openapi-spec
OpenAPI contract between iFit &amp; BrightAI

## Handling errors
Shamelessly copied and adapted from the [tvOS Validate PR](https://github.com/ifit/ifit/pull/17003#discussion_r627050336)

General convention for requests:
- when a validation/problem processing entitlement/internal server error occurs
  - respond with one of the non-200 status code responses
- when a `validateWithIAPPlatform` request succeeds (2xx)
  - check `entitlement` (platform dependent: boolean, date comparison, etc.)
    - `entitlement == true`, do-stuff-on-user, return `res.send({ isEntitled: true })`
    - `entitlement == false`, return `res.send({ isEntitled: false, { ...error, retryable: false } })`
      - generally, we'll try to give a little more context when `isEntitled = false`,
        e.g. `retryable`, error info.
- when a `validateWithIAPPlatform` throws REST error (non-2xx):
  - `res.send({ isEntitled: false, { ...restError, retryable: true } })`

the `res.send`'s all being 200 responses back to BrightAPI when they make the request to `v1/tv-app/validate`.

Essentially just proxying back the non-2xx errors for BrightAPI to decide if they want to retry a validation request.  Otherwise it's a hard yes/no on entitlement because the respective `validateWithIAPPlatform` gives us that piece of info in it's respective 200 response(s).