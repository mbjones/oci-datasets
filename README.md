# OCI as a data package storage layer

While Open Container Initiative (OCI) registries have enabled widespread, cross-platform, and vendor-neutral storage and sharing of compute images for containers, they also have utility for storing other objects. Over the last several years, people have demonstrated storing other types of artifacts in these registries, including Helm charts, brew packages, SBOMs, and others. The OCI specifications have evolved to support a more general `artifact` type in addition to the traditional `image` format used for containers.

Public storage on many OCI-complient registries is at least partially free, including on the Github Packages (ghcr.io), the quay.io registry, and others. These registries are also generally tied into content distribution networks (CDNs), so accessing data from them tends to be faster than from other locations. The OCI has created the "OCI Registry as Storage" `oras` commandline tool (https://oras.land/docs/quickstart) for interacting with any OCI-compliant registry. Try `brew install oras` for a quick launch.

We can leverage the OCI Registry system to store data packages, at least replicas of public data packages. DataONE tracks the individual objects in a data package, including the data, scientific metadata, system metadata, and the package manifest (ORE), and each of these can be retrieved independently from the DataONE API. We also have a package download API, which supports downloading all of these components in a package format such as the BagIt format. We could leverage the OCI registries to store individual objects, or to store entire package directory structures in BagIt and other forms.

Below are some experiments in using the oras tool for uploading and downloading objects and packages from OCI.

## Uploading a BagIt directory as a package to GHCR:

```zsh
❯ oras push ghcr.io/mbjones/d1obj:dataset_10.18739_A2D795C92 --artifact-type application/bagit-1.0 ./dataset_10.18739_A2D795C92
✓ Uploaded  application/vnd.oci.empty.v1+json                                                                                                2/2  B 100.00%  564ms
  └─ sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a
✓ Uploaded  dataset_10.18739_A2D795C92                                                                                                 5.11/5.11 MB 100.00%     4s
  └─ sha256:45be1bc87b4fd9dabfa2f3d7eeaee0d95daefafb0ee35d8b6ebabf73f203f1ed
✓ Uploaded  application/vnd.oci.image.manifest.v1+json                                                                                   742/742  B 100.00%  390ms
  └─ sha256:3b772c982c6126457667ce01e7696953affb55d8fedbfd9fd6030121bc080846
Pushed [registry] ghcr.io/mbjones/d1obj:dataset_10.18739_A2D795C92
ArtifactType: application/bagit-1.0
Digest: sha256:3b772c982c6126457667ce01e7696953affb55d8fedbfd9fd6030121bc080846
```

## Finding manifest for an artifact

```zsh
❯ oras discover ghcr.io/mbjones/d1obj:dataset_10.18739_A2D795C92
ghcr.io/mbjones/d1obj@sha256:3b772c982c6126457667ce01e7696953affb55d8fedbfd9fd6030121bc080846
```

## Fetch the manifest for an artifact

Run `oras manifest fetch ghcr.io/mbjones/d1obj:dataset_10.18739_A2D795C92 --pretty`:

```json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "artifactType": "application/bagit-1.0",
  "config": {
    "mediaType": "application/vnd.oci.empty.v1+json",
    "digest": "sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a",
    "size": 2,
    "data": "e30="
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "digest": "sha256:45be1bc87b4fd9dabfa2f3d7eeaee0d95daefafb0ee35d8b6ebabf73f203f1ed",
      "size": 5358776,
      "annotations": {
        "io.deis.oras.content.digest": "sha256:992756058e199a728863a942d454832d2f47652f6b5590a92df3609cca78a787",
        "io.deis.oras.content.unpack": "true",
        "org.opencontainers.image.title": "dataset_10.18739_A2D795C92"
      }
    }
  ],
  "annotations": {
    "org.opencontainers.image.created": "2025-11-07T02:19:02Z"
  }
}
```

## Pulling an artifact to the local host

```zsh
❯ oras pull ghcr.io/mbjones/d1obj:dataset_10.18739_A2D795C92
✓ Pulled      dataset_10.18739_A2D795C92                                                                                               5.11/5.11 MB 100.00%  587ms
  └─ sha256:45be1bc87b4fd9dabfa2f3d7eeaee0d95daefafb0ee35d8b6ebabf73f203f1ed
✓ Pulled      application/vnd.oci.image.manifest.v1+json                                                                                 742/742  B 100.00%  434µs
  └─ sha256:3b772c982c6126457667ce01e7696953affb55d8fedbfd9fd6030121bc080846
Pulled [registry] ghcr.io/mbjones/d1obj:dataset_10.18739_A2D795C92
Digest: sha256:3b772c982c6126457667ce01e7696953affb55d8fedbfd9fd6030121bc080846
```

## Pushing to Quay, pushing a whole dataset BagIt directory

```zsh
❯ echo ${QUAY_PAT} | oras login quay.io -u 'dataone+dataone' --password-stdin
Login Succeeded

❯ oras push quay.io/dataone/objects:latest --artifact-type text/csv data/CP_2024_25m_Temp.csv
✓ Uploaded  application/vnd.oci.empty.v1+json                                                                                                2/2  B 100.00%     2s
  └─ sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a
✓ Uploaded  data/CP_2024_25m_Temp.csv                                                                                                  11.1/11.1 MB 100.00%     6s
  └─ sha256:3de00b8e974c4ae3714809fbe56d10df22519e54d820eab387858e577ce1443e
✓ Uploaded  application/vnd.oci.image.manifest.v1+json                                                                                   583/583  B 100.00%  443ms
  └─ sha256:5036460570e339b022e8d43035843e7b377aec512dab35df43c80d86f11fa441
Pushed [registry] quay.io/dataone/objects:latest
ArtifactType: text/csv
Digest: sha256:5036460570e339b022e8d43035843e7b377aec512dab35df43c80d86f11fa441

❯ oras manifest fetch quay.io/dataone/objects:dataset_10.18739_A2D795C92 --pretty
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "artifactType": "application/bagit-1.0A",
  "config": {
    "mediaType": "application/vnd.oci.empty.v1+json",
    "digest": "sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a",
    "size": 2,
    "data": "e30="
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "digest": "sha256:82148bbc3ca5af7be2ed078239cfeb7e91645d16200376b4a3736127a92b688a",
      "size": 5358777,
      "annotations": {
        "io.deis.oras.content.digest": "sha256:aeaa99e88264d56a9da362b03857e7848b970de8e698fac2ec59f31f80f86d4a",
        "io.deis.oras.content.unpack": "true",
        "org.opencontainers.image.title": "dataset_10.18739_A2D795C92"
      }
    }
  ],
  "annotations": {
    "org.opencontainers.image.created": "2025-11-07T07:13:58Z"
  }
}
```

# Example: Individual object by tag and digest, with directory path

```zsh
❯ oras push quay.io/dataone/objects:urn_uuid_bd3d29b6-b25d-46ab-91a0-1938cd7ccadd --artifact-type text/csv ./dataset_10.18739_A2D795C92/data/T3_2024_15m_Temp.csv
✓ Exists    application/vnd.oci.empty.v1+json                                                                                                2/2  B 100.00%     0s
  └─ sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a
✓ Uploaded  dataset_10.18739_A2D795C92/data/T3_2024_15m_Temp.csv                                                                       10.9/10.9 MB 100.00%     7s
  └─ sha256:fc3436daeeeb2db0bb7955b4dbe4e2220a5e7ed1f26593f1b2d69a171699de00
✓ Uploaded  application/vnd.oci.image.manifest.v1+json                                                                                   610/610  B 100.00%  405ms
  └─ sha256:103d6fb2cc7f1ba883f63862dc87af7e57a333b86e58338a53aba11f782ee9ed
Pushed [registry] quay.io/dataone/objects:urn_uuid_bd3d29b6-b25d-46ab-91a0-1938cd7ccadd
ArtifactType: text/csv
Digest: sha256:103d6fb2cc7f1ba883f63862dc87af7e57a333b86e58338a53aba11f782ee9ed

❯ oras manifest fetch quay.io/dataone/objects:urn_uuid_bd3d29b6-b25d-46ab-91a0-1938cd7ccadd | jq .
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "artifactType": "text/csv",
  "config": {
    "mediaType": "application/vnd.oci.empty.v1+json",
    "digest": "sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a",
    "size": 2,
    "data": "e30="
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar",
      "digest": "sha256:fc3436daeeeb2db0bb7955b4dbe4e2220a5e7ed1f26593f1b2d69a171699de00",
      "size": 11379409,
      "annotations": {
        "org.opencontainers.image.title": "dataset_10.18739_A2D795C92/data/T3_2024_15m_Temp.csv"
      }
    }
  ],
  "annotations": {
    "org.opencontainers.image.created": "2025-11-07T07:47:52Z"
  }
}

❯ oras discover quay.io/dataone/objects:urn_uuid_bd3d29b6-b25d-46ab-91a0-1938cd7ccadd
quay.io/dataone/objects@sha256:103d6fb2cc7f1ba883f63862dc87af7e57a333b86e58338a53aba11f782ee9ed

❯ oras pull quay.io/dataone/objects@sha256:103d6fb2cc7f1ba883f63862dc87af7e57a333b86e58338a53aba11f782ee9ed
✓ Pulled      dataset_10.18739_A2D795C92/data/T3_2024_15m_Temp.csv                                                                     10.9/10.9 MB 100.00%     1s
  └─ sha256:fc3436daeeeb2db0bb7955b4dbe4e2220a5e7ed1f26593f1b2d69a171699de00
✓ Pulled      application/vnd.oci.image.manifest.v1+json                                                                                 610/610  B 100.00%    1ms
  └─ sha256:103d6fb2cc7f1ba883f63862dc87af7e57a333b86e58338a53aba11f782ee9ed
Pulled [registry] quay.io/dataone/objects@sha256:103d6fb2cc7f1ba883f63862dc87af7e57a333b86e58338a53aba11f782ee9ed
Digest: sha256:103d6fb2cc7f1ba883f63862dc87af7e57a333b86e58338a53aba11f782ee9ed
```

# Individual object without directory path

```zsh
❯ oras push quay.io/dataone/objects:urn_uuid_1f5274ce-8f8a-4b89-bd8a-e61b93072c55 --artifact-type text/csv UP18_2024_25m_Temp.csv
✓ Uploaded  UP18_2024_25m_Temp.csv                                                                                                     10.8/10.8 MB 100.00%    12s
  └─ sha256:247f1fc78ba5d9bc2de7f453fb479d078eef109ade2080149ecf2edc69c3c510
✓ Exists    application/vnd.oci.empty.v1+json                                                                                                2/2  B 100.00%     0s
  └─ sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a
✓ Uploaded  application/vnd.oci.image.manifest.v1+json                                                                                   580/580  B 100.00%  365ms
  └─ sha256:2d08ec5cd3a0f1f5bd41c0ac654db54d1f8eb9cebe236ed9ead06dc771b4c5f7
Pushed [registry] quay.io/dataone/objects:urn_uuid_1f5274ce-8f8a-4b89-bd8a-e61b93072c55
ArtifactType: text/csv
Digest: sha256:2d08ec5cd3a0f1f5bd41c0ac654db54d1f8eb9cebe236ed9ead06dc771b4c5f7

❯ oras discover quay.io/dataone/objects:urn_uuid_1f5274ce-8f8a-4b89-bd8a-e61b93072c55
quay.io/dataone/objects@sha256:2d08ec5cd3a0f1f5bd41c0ac654db54d1f8eb9cebe236ed9ead06dc771b4c5f7
❯ oras manifest fetch quay.io/dataone/objects:urn_uuid_1f5274ce-8f8a-4b89-bd8a-e61b93072c55 | jq .
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "artifactType": "text/csv",
  "config": {
    "mediaType": "application/vnd.oci.empty.v1+json",
    "digest": "sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a",
    "size": 2,
    "data": "e30="
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar",
      "digest": "sha256:247f1fc78ba5d9bc2de7f453fb479d078eef109ade2080149ecf2edc69c3c510",
      "size": 11293998,
      "annotations": {
        "org.opencontainers.image.title": "UP18_2024_25m_Temp.csv"
      }
    }
  ],
  "annotations": {
    "org.opencontainers.image.created": "2025-11-07T08:04:31Z"
  }
}
❯ oras pull quay.io/dataone/objects@sha256:2d08ec5cd3a0f1f5bd41c0ac654db54d1f8eb9cebe236ed9ead06dc771b4c5f7 -o downloads
✓ Pulled      UP18_2024_25m_Temp.csv                                                                                                   10.8/10.8 MB 100.00%     1s
  └─ sha256:247f1fc78ba5d9bc2de7f453fb479d078eef109ade2080149ecf2edc69c3c510
✓ Pulled      application/vnd.oci.image.manifest.v1+json                                                                                 580/580  B 100.00%    1ms
  └─ sha256:2d08ec5cd3a0f1f5bd41c0ac654db54d1f8eb9cebe236ed9ead06dc771b4c5f7
Pulled [registry] quay.io/dataone/objects@sha256:2d08ec5cd3a0f1f5bd41c0ac654db54d1f8eb9cebe236ed9ead06dc771b4c5f7
Digest: sha256:2d08ec5cd3a0f1f5bd41c0ac654db54d1f8eb9cebe236ed9ead06dc771b4c5f7
❯ tree downloads
downloads
└── UP18_2024_25m_Temp.csv

1 directory, 1 file
```
