# `x-bake` extension field with Compose

## Resources

* https://github.com/docker/buildx/pull/721
* https://github.com/docker/buildx/blob/master/docs/reference/buildx_bake.md#extension-field-with-compose

## Usage

Create and start the container with compose-cli as usual:

```shell
$ docker compose up -d --build
#20 exporting to image
#20 sha256:e8c613e07b0b7ff33893b694f7759a10d42e180f2b4dc349fb57dc6b71dcab00
#20 exporting layers done
#20 writing image sha256:e075e57ce65d9a593efb4fbef05a67d62f72811ef7096a4a83fb14d074f66aa2 done
#20 naming to docker.io/crazymax/xbake-demo:foo
#20 naming to docker.io/crazymax/xbake-demo:foo done
#20 DONE 0.0s
[+] Running 1/0
 ⠿ Container cloudflared  Running
```

Let's print the bake definition of the compose file with bake:

```shell
$ docker buildx bake --print
{
  "group": {
    "default": [
      "cloudflared"
    ]
  },
  "target": {
    "cloudflared": {
      "context": ".",
      "dockerfile": "./Dockerfile",
      "args": {
        "CLOUDFLARED_VERSION": "2021.8.2"
      },
      "tags": [
        "crazymax/xbake-demo:foo",
        "crazymax/xbake-demo:bar"
      ],
      "cache-to": [
        "type=registry,ref=crazymax/xbake-demo:cache"
      ],
      "platforms": [
        "linux/amd64",
        "linux/arm64"
      ]
    }
  }
}
```

`x-bake` is taken into account, let's build our multi-platform image and push:

```shell
$ docker buildx bake --push
#29 [auth] crazymax/xbake-demo:pull,push token for registry-1.docker.io
#29 DONE 0.0s

#27 exporting to image
#27 pushing layers 14.4s done
#27 pushing manifest for docker.io/crazymax/xbake-demo:foo@sha256:dbf53d691559f37e7b13a872b2c24c21b8d4f50c0fb56fe1e3e6ef6d7df6ba82
#27 pushing manifest for docker.io/crazymax/xbake-demo:foo@sha256:dbf53d691559f37e7b13a872b2c24c21b8d4f50c0fb56fe1e3e6ef6d7df6ba82 0.9s done
#27 pushing layers 0.6s done
#27 pushing manifest for docker.io/crazymax/xbake-demo:bar@sha256:dbf53d691559f37e7b13a872b2c24c21b8d4f50c0fb56fe1e3e6ef6d7df6ba82
#27 pushing manifest for docker.io/crazymax/xbake-demo:bar@sha256:dbf53d691559f37e7b13a872b2c24c21b8d4f50c0fb56fe1e3e6ef6d7df6ba82 0.7s done
#27 DONE 18.2s

#30 exporting cache
#30 preparing build cache for export
#30 preparing build cache for export 0.2s done
#30 writing layer sha256:1ba3ca939a193631c4d50a75b9880058e3a8bfe6b99d1a246b7b5eee4d4b3b7d
#30 ...

#31 [auth] crazymax/xbake-demo:pull,push token for registry-1.docker.io
#31 DONE 0.0s

#30 exporting cache
#30 writing layer sha256:1ba3ca939a193631c4d50a75b9880058e3a8bfe6b99d1a246b7b5eee4d4b3b7d 1.7s done
#30 writing layer sha256:52356da8cb69de73ac817266fd6ecc8acb132fe15fefddecc4bce85ff71f7845
#30 writing layer sha256:52356da8cb69de73ac817266fd6ecc8acb132fe15fefddecc4bce85ff71f7845 0.1s done
#30 writing layer sha256:552d1f2373af9bfe12033568ebbfb0ccbb0de11279f9a415a29207e264d7f4d9
#30 writing layer sha256:552d1f2373af9bfe12033568ebbfb0ccbb0de11279f9a415a29207e264d7f4d9 0.2s done
#30 writing layer sha256:5d65ffdb58fccd87d626d3099a3045acdda96490c3489062ea8417aa6aa259a1
#30 writing layer sha256:5d65ffdb58fccd87d626d3099a3045acdda96490c3489062ea8417aa6aa259a1 0.1s done
#30 writing layer sha256:9590ba3cabc3753cce1a1bf8cc95c43742cfdb4843cd57b7023523abde843b39
#30 writing layer sha256:9590ba3cabc3753cce1a1bf8cc95c43742cfdb4843cd57b7023523abde843b39 0.2s done
#30 writing layer sha256:a0d0a0d46f8b52473982a3c466318f479767577551a53ffc9074c9fa7035982e
#30 writing layer sha256:a0d0a0d46f8b52473982a3c466318f479767577551a53ffc9074c9fa7035982e 0.1s done
#30 writing config sha256:a4e1d820397dd1cc77811d0cdb3d6f256ef50505dedb2529d9c880ff79fb6fac
#30 writing config sha256:a4e1d820397dd1cc77811d0cdb3d6f256ef50505dedb2529d9c880ff79fb6fac 2.9s done
#30 writing manifest sha256:6b6d1f93277588cb96761dc38b5670e945887b9fd52ae46d80492fc22b4e576c
#30 writing manifest sha256:6b6d1f93277588cb96761dc38b5670e945887b9fd52ae46d80492fc22b4e576c 0.6s done
#30 DONE 6.1s
```

Let's see if the manifest is ok on the registry:

```shell
$ docker buildx imagetools inspect crazymax/xbake-demo:foo
Name:      docker.io/crazymax/xbake-demo:foo
MediaType: application/vnd.docker.distribution.manifest.list.v2+json
Digest:    sha256:dbf53d691559f37e7b13a872b2c24c21b8d4f50c0fb56fe1e3e6ef6d7df6ba82

Manifests:
  Name:      docker.io/crazymax/xbake-demo:foo@sha256:7e08cd71b3d95fca9b84a8e8fbeadce019afdf25f55f9ffd58845b54f4b70056
  MediaType: application/vnd.docker.distribution.manifest.v2+json
  Platform:  linux/amd64

  Name:      docker.io/crazymax/xbake-demo:foo@sha256:ba5f32c9026b82d13acb289363a136a627436e2c688580b13de749cea0ec1868
  MediaType: application/vnd.docker.distribution.manifest.v2+json
  Platform:  linux/arm64
```
