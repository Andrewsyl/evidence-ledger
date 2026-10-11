# Evidence ledger

SHA-256 hashes of archived copies of public web pages and public datasets (property listings, the Property Price Register, PSRA sanctions and licence register, CRO company records), published here on the day each copy was made.

The archived files themselves are not published here. This repository proves that a given file existed, byte for byte, no later than the date shown: change one byte and its hash no longer matches.

## Layout

- `manifests/<date>/run-<n>.gz.sha256`: one line per archived page from that day's listing run (`shasum` format). `run-<n>.raw.sha256` hashes the same pages uncompressed, exactly as served.
- `manifests/{ppr,psra,register,cro}/<date>.sha256`: the other daily or weekly downloads.
- `manifests/psra/wayback-<date>.sha256` and `.sources.txt`: copies fetched from the Internet Archive, with the capture URL for each, so they can be re-fetched independently.
- `seals/<date>.txt`: the hash of every manifest written that day, plus the hash of the previous day's seal. The seals form a chain: altering any earlier day breaks every later seal.
- `seals/<date>.txt.digicert.tsr`, `.freetsa.tsr`: RFC 3161 timestamps of the seal from two independent timestamp authorities.
- `seals/<date>.txt.ots`: an OpenTimestamps proof anchoring the seal in the Bitcoin blockchain.
- `tsa/`: the timestamp authorities' certificates, for verification.

Each day's seal is also saved to the Internet Archive (web.archive.org), and the repository is archived weekly by Software Heritage.

## Verifying a file

```sh
shasum -a 256 <file>                                   # compare with its line in manifests/...
grep <hash> -r manifests/                              # find which manifest lists it
shasum -a 256 manifests/<...>                          # compare with its line in seals/<date>.txt
```

## Verifying the date of a seal

```sh
openssl ts -verify -in seals/<date>.txt.freetsa.tsr  -data seals/<date>.txt -CAfile tsa/freetsa-cacert.pem -untrusted tsa/freetsa-tsa.crt
openssl ts -verify -in seals/<date>.txt.digicert.tsr -data seals/<date>.txt -CAfile tsa/digicert-roots.pem
openssl ts -reply  -in seals/<date>.txt.digicert.tsr -text | grep "Time stamp"
pip install opentimestamps-client && ots verify seals/<date>.txt.ots   # needs a Bitcoin node, or use https://opentimestamps.org
```

## Verifying the chain

The second line of each seal is the hash of the previous seal file. Recompute it with `shasum -a 256 seals/<previous date>.txt`; every link must match.
