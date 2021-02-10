# Personal Container Image Signature Store

## Setup

1. configure signature store location in `/etc/containers/registries.d/fphammerle.yaml`:
```yaml
docker:
  quay.io/fphammerle:
    sigstore: https://raw.githubusercontent.com/fphammerle/container-image-sigstore/master/quay.io
```

2. download key
```sh
$ gpg --recv-key 8D2902FE7DF47DDEDA2802F9456B9A0399A5DA2F
$ gpg --export --armor --output /some/where/pgp/fphammerle 8D2902FE7DF47DDEDA2802F9456B9A0399A5DA2F
```

3. enable verification in `policy.json`:
```json
{
  "default": [
    {
      "type": "reject"
    }
  ],
  "transports": {
    "docker": {
      "quay.io/fphammerle": [
        {
          "type": "signedBy",
          "keyType": "GPGKeys",
          "keyPath": "/some/where/pgp/fphammerle",
          "signedIdentity": {
            "type": "matchRepoDigestOrExact"
          }
        }
      ]
    }
  }
}
```

### Verify with Podman

```sh
$ podman image trust show
default             reject
quay.io/fphammerle  signedBy  fabian@hammerle.me  https://raw.githubusercontent.com/fphammerle/container-image-sigstore/master/quay.io
$ podman --log-level debug run --rm quay.io/fphammerle/systemctl-mqtt:0.5.0-amd64
[...]
DEBU[0000] Using registries.d directory /etc/containers/registries.d for sigstore configuration
DEBU[0000]  Using "docker" namespace quay.io/fphammerle
DEBU[0000]   Using https://raw.githubusercontent.com/fphammerle/container-image-sigstore/master/quay.io
[...]
DEBU[0002] GET https://raw.githubusercontent.com/fphammerle/container-image-sigstore/master/quay.io/fphammerle/systemctl-mqtt@sha256=34dcb878dbd66315de6fbf97ceb29e8fec549b7269c6c828c4c889a54a091f14/signature-1
DEBU[0002] GET https://raw.githubusercontent.com/fphammerle/container-image-sigstore/master/quay.io/fphammerle/systemctl-mqtt@sha256=34dcb878dbd66315de6fbf97ceb29e8fec549b7269c6c828c4c889a54a091f14/signature-2
DEBU[0002]  Requirement 0: allowed
DEBU[0002] Overall: allowed
[...]
DEBU[0004] Starting container 4e46cbe4e982ee84bcff54092146fb0442bb346e451cfb14e2e7f491bc886b88 with command [tini -- systemctl-mqtt --help]
[...]
$ podman --log-level debug run --rm quay.io/fphammerle/systemctl-mqtt:0.5.0-amd64-unsigned
[...]
DEBU[0000] Using registries.d directory /etc/containers/registries.d for sigstore configuration
DEBU[0000]  Using "docker" namespace quay.io/fphammerle
DEBU[0000]   Using https://raw.githubusercontent.com/fphammerle/container-image-sigstore/master/quay.io
[...]
DEBU[0002] GET https://raw.githubusercontent.com/fphammerle/container-image-sigstore/master/quay.io/fphammerle/systemctl-mqtt@sha256=34dcb878dbd66315de6fbf97ceb29e8fec549b7269c6c828c4c889a54a091f14/signature-1
DEBU[0002] GET https://raw.githubusercontent.com/fphammerle/container-image-sigstore/master/quay.io/fphammerle/systemctl-mqtt@sha256=34dcb878dbd66315de6fbf97ceb29e8fec549b7269c6c828c4c889a54a091f14/signature-2
DEBU[0002] Requirement 0: denied, done
DEBU[0002] Error pulling image ref //quay.io/fphammerle/systemctl-mqtt:0.5.0-amd64-unsigned: Source image rejected: Signature for identity quay.io/fphammerle/systemctl-mqtt:0.5.0-amd64 is not accepted
  Signature for identity quay.io/fphammerle/systemctl-mqtt:0.5.0-amd64 is not accepted
Error: unable to pull quay.io/fphammerle/systemctl-mqtt:0.5.0-amd64-unsigned: Source image rejected: Signature for identity quay.io/fphammerle/systemctl-mqtt:0.5.0-amd64 is not accepted
```

## References

* [containers/registries.d: sigstore configuration](https://github.com/containers/image/blob/v5.5.2/docs/containers-registries.d.5.md#individual-configuration-sections)
* [containers/policy.json: `signedBy` configuration](https://github.com/containers/image/blob/v5.5.2/docs/containers-policy.json.5.md#signedby)
* [How to sign and distribute container images using Podman](https://github.com/containers/podman/blob/v2.0.6/docs/tutorials/image_signing.md)
* [Verifying signatures of Red Hat container images](https://developers.redhat.com/blog/2019/10/29/verifying-signatures-of-red-hat-container-images/)
  ([archived](https://web.archive.org/web/20210210072204/https://developers.redhat.com/blog/2019/10/29/verifying-signatures-of-red-hat-container-images/))
