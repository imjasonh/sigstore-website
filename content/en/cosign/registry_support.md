---
title: "Registry Support"
category: "Cosign"
position: 116
---

`cosign` uses [go-containerregistry](https://github.com/google/go-containerregistry) for registry
interactions, which has generally excellent compatibility, but some registries may have quirks.

Today, `cosign` has been tested and works against the following registries:

* AWS Elastic Container Registry
* GCP's Artifact Registry and Container Registry
* Docker Hub
* Azure Container Registry
* JFrog Artifactory Container Registry
* The CNCF distribution/distribution Registry
* GitLab Container Registry
* GitHub Container Registry
* The CNCF Harbor Registry
* Digital Ocean Container Registry
* Sonatype Nexus Container Registry
* Alibaba Cloud Container Registry

We aim for wide registry support. To `sign` images in registries which do not yet fully support [OCI media types](https://github.com/sigstore/cosign/blob/main/SPEC.md#object-types), one may need to use `COSIGN_DOCKER_MEDIA_TYPES` to fall back to legacy equivalents. For example:
```shell
COSIGN_DOCKER_MEDIA_TYPES=1 cosign sign --key cosign.key legacy-registry.example.com/my/image
```

Please help test and file bugs if you see issues!
Instructions can be found in the [tracking issue](https://github.com/sigstore/cosign/issues/40).

## Rekor Support
_Note: this is an experimental feature_

To publish signed artifacts to a Rekor transparency log and verify their existence in the log
set the `COSIGN_EXPERIMENTAL=1` environment variable.

```shell
COSIGN_EXPERIMENTAL=1 cosign sign --key cosign.key dlorenc/demo
COSIGN_EXPERIMENTAL=1 cosign verify --key cosign.pub dlorenc/demo
```

`cosign` defaults to using the public instance of rekor at [rekor.sigstore.dev](https://rekor.sigstore.dev).
To configure the rekor server, use the -`rekor-url` flag

## Registry Details

`cosign` signatures are stored as separate objects in the OCI registry, with only a weak
reference back to the object they "sign".
This means this relationship is opaque to the registry, and signatures *will not* be deleted
or garbage-collected when the image is deleted.
Similarly, they **can** easily be copied from one environment to another, but this is not
automatic.

Multiple signatures are stored in a list which is unfortunately "racy" today.
To add a signature, clients orchestrate a "read-append-write" operation, so the last write
will win in the case of contention.

## Specifying Registry
`cosign` will default to storing signatures in the same repo as the image it is signing.
To specify a different repo for signatures, you can set the `COSIGN_REPOSITORY` environment variable.

This will replace the repo in the provided image like this:
```
export COSIGN_REPOSITORY=gcr.io/my-new-repo
gcr.io/dlorenc-vmtest2/demo -> gcr.io/my-new-repo/demo:sha256-DIGEST.sig
```
So the signature for `gcr.io/dlorenc-vmtest2/demo` will be stored in `gcr.io/my-new-repo/demo:sha256-DIGEST.sig`.