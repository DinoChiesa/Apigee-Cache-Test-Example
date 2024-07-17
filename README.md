# Cache Test proxy for Apigee

This API proxy demonstrates the ability to manipulate the Apigee cache, from
the LookupCache PopulateCache, and InvalidateCache policies.

## The way it works

The API Proxy allows an API caller to
- populate entries into the cache under a specific cache key
- lookup entries from the cache under a specific cache key
- invalidate entries under a specific cache key

This example uses Application Scope for the cache, but since this is a single
API proxy, that is mostly irrelevant.

Consult the documentation on these policies for full details on how they work.

## Instructions for use

1. On Apigee Edge: Create a named cache resource called `cache1` in your environment.
   On Apigee X or hybrid, this is not necessary.

2. Import and Deploy the apiproxy to the environment.  You can use tools like
   [apigeecli](https://github.com/apigee/apigeecli) or the [importAndDeploy script](https://github.com/DinoChiesa/apigee-edge-js-examples/blob/main/importAndDeploy.js).

   Or, you can import and deploy the proxy via the UI wizard.

3. Open a terminal, and set the `endpoint` shell variable:

   ```
   # for Apigee Edge
   ORG=myorg
   ENV=test
   endpoint=https://$ORG-$ENV.apigee.net
   
   # for Apigee X
   endpoint=https://your-api-endpoint-name.net
   ```
   If you're on Windows, then open a powershell terminal and:
   ```
   $endpoint='https://your-api-endpoint-name.net'
   ```

4. Load some values into the cache via policy with `POST /populate-no-prefix`.
   This uses the queryparam `id` as the cache key.  When you use a different
   value for `id` the proxy will populate different entries in the cache.

   On a Mac or *nix machine:
   ```
   curl -i -X POST $endpoint/cache-test/populate-no-prefix\?id=5 -d plaintext
   curl -i -X POST $endpoint/cache-test/populate-no-prefix\?id=7 -d this.is.some.other.value
   curl -i -X POST $endpoint/cache-test/populate-no-prefix\?id=9 -d '{"key":"value"}'
   ```

   In Powershell, you would use something like this:
   ```
   Invoke-WebRequest -Uri $endpoint/cache-test/populate-no-prefix?id=5 -Method Post -Body something-here \
        -ContentType "text/plain"
   ```
   You may have to add an extra option,  `-UseBasicParsing` , to that line.

   Or you can invoke the API via a graphical tool like Postman or Paw.


5. Read from the cache via nodejs with `GET /lookup-no-prefix`

   On a Mac or *nix machine:
   ```
   curl -i $endpoint/cache-test/lookup-no-prefix\\?id=5
   curl -i $endpoint/cache-test/lookup-no-prefix\\?id=7
   curl -i $endpoint/cache-test/lookup-no-prefix\\?id=9
   ```

   Or, for powershell users:
   ```
   Invoke-WebRequest -Uri $endpoint/cache-test/populate-no-prefix?id=5 -Method Get
   ```

   You should see the values you populated earlier.


5. invalidate a specific element in the cache with the invalidate policy, and then try a few lookups:

   ```
   curl -i $endpoint/cache-test/lookup-no-prefix\?id=5
   curl -i -X POST $endpoint/cache-test/invalidate-no-prefix\?id=5 -d ''
   curl -i $endpoint/cache-test/lookup-no-prefix\?id=5
   curl -i $endpoint/cache-test/lookup-no-prefix\?id=6
   ```

   You should see the cache entries for the previously-invalidated key, as empty.

   NB: Some of the lookups may succeed; if this happens it is because the invalidation has not propagated yet to all nodes.

6. try a lookup on a different resouce.

   ```
   curl -i $endpoint/cache-test/lookup\?id=7
   ```

   If you're fast enough, you should see the data that you populated previously.

7. Wait 3 minutes, then try more lookups.  You should see that all the cache entries are gone.
   They've automatically expired.


## Explore a little

There are numerous flows in the API Proxy. For example, in some cases the
policies use a prefix in the `CacheKey` element. In other cases, the InvalidateCache policy uses `PurgeChildEntries`.

Feel free to explore. You can try them all out to see how they behave.


## Clean Up

When you're finished, you can undeploy and delete the API Proxy. If you like
you can delete the cache resource too. This is necessary on Apigee Edge only.

## Disclaimer

This example is not an official Google product, nor is it part of an official Google product.
