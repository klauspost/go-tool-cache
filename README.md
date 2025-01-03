# go-tool-cache

Like Go's built-in build/test caching but wish it weren't purely stored on local disk in the `$GOCACHE` directory?

Want to share your cache over the network between your various machines, coworkers, and CI runs without all that GitHub actions/caches tarring and untarring?

Along with a [modification to Go's `cmd/go` tool](https://go-review.googlesource.com/c/go/+/486715) ([open proposal](https://github.com/golang/go/issues/59719)), this repo lets you write
custom `GOCACHE` implementations to handle the cache however you'd like.

## Status

Currently you need to build your own Go toolchain to use this. As of 2023-04-24 it's still an open proposal & work in progress.

## Using

First, build your cache child process. For example,

```sh
$ go install github.com/bradfitz/go-tool-cache/cmd/go-cacher
```

Then tell Go to use it:

```sh
$ GOCACHEPROG=$HOME/go/bin/go-cacher go install std
```

See some stats:

```sh
$ GOCACHEPROG="$HOME/go/bin/go-cacher --verbose" go install std
Defaulting to cache dir /home/bradfitz/.cache/go-cacher ...
cacher: closing; 548 gets (0 hits, 548 misses, 0 errors); 1090 puts (0 errors)
```

Run it again and watch the hit rate go up:

```sh
$ GOCACHEPROG="$HOME/go/bin/go-cacher --verbose" go install std
Defaulting to cache dir /home/bradfitz/.cache/go-cacher ...
cacher: closing; 808 gets (808 hits, 0 misses, 0 errors); 0 puts (0 errors)
```

## S3 Support

We support S3 backend for caching.

You can connect to S3 backend by setting the following parameters:
- `GOCACHE_S3_BUCKET` - Name of S3 bucket (required)
- `GOCACHE_S3_ACCESS_KEY` + `GOCACHE_S3_SECRET_KEY` - Use static credentials.
- `GOCACHE_S3_SESSION_TOKEN` - Session token to use for alternative authentication.
- `GOCACHE_S3_REGION` - AWS Region of bucket. Default is `us-east-1`.
- `GOCACHE_S3_URL` - specify a custom endpoint. Will switch to path-style requests.
- `GOCACHE_S3_PREFIX` - Use a custom prefix for all entries. Default is `go-cacher`.

- Direct credentials or creds profile to use:
  - `GOCACHE_AWS_CREDS_PROFILE` 

The cache would be stored to `s3://<bucket>/cache/<cache_key>/<architecture>/<os>/<go-version>`
