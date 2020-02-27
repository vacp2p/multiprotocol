# multiprotocol: multiformat inspired self-describing protocol identifiers

@TODO MOVE TABLE TO VAC SPECIFIC REPO

**:warning: THIS IS STILL AN EARLY DRAFT :warning:**

In this specification we describe a simple method for nodes to advertise their capabilities. 
The protocol is heavily inspired by [multiformats](https://multiformats.io/) and provides both a human and machine readable representation.

The goal is to provide node identification beyond the [multiaddr](https://github.com/multiformats/multiaddr) 
connection info which can be appended to the end of the address.

This repository contains the [multiprotocol definition](./multiprotocol.csv) used by [vac](https://vac.dev), 
the [go implementation](https://github.com/vacp2p/go-multiprotocol) however is generic and therefore anyone can implement their own table.

The table is represented using the following CSV format:

```csv
code, size, name, comment
1,    0,    vac,  namespace
2,    V,    waku,
```

| field       | description                                                                                                                     |
| :---------: | :------------------------------------------------------------------------------------------------------------------------------ |
| **code**    | This field contains the code identifying the key.                                                                               |
| **size**    | This field identifies the expected keys size, it can be any number or `V`, indicating that the value itself is length prefixed. |
| **name**    | The human readable name of the field.                                                                                           |
| **comment** | Any developer related comments for the field.                                                                                   |

**Namespaces should be generic to not cause overlap, vac uses `42`**

Human-readable:

```
/<namespace>/<protocol>/<version>(/<capability>/<version>|<capability>)+
```

Examples:

```
/vac/waku/2
/vac/waku/2/relay/2
/vac/waku/2/store/1
```

Machine-readable:

```
<namespace uvarint><protocol uvarint><version uvarint>(<protoCode uvarint><value []byte>)+
```

Examples:

```python
0x2a 0x2 0x1 0x32 # /vac/waku/2
0x2a 0x2 0x1 0x32 0x4 0x1 0x32 # /vac/waku/2/relay/2
0x2a 0x2 0x1 0x32 0x3 0x1 0x32 # /vac/waku/2/store/1
```


With multiaddr:

```
/ip4/127.0.0.1/tcp/9000/vac/waku/0.2/relay/0.2
```
