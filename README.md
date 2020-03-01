# multiprotocol: multiformat inspired self-describing protocol identifiers

@TODO MOVE TABLE TO VAC SPECIFIC REPO

This specification describes a simple method for nodes to describe the set of their capabilities.
The protocol is inspired by other [multiformats](https://multiformats.io/), it provides both human and machine-readable
representations.

The goal is to provide more granular identification for nodes beyond their connection information as provided by
[multiaddr](https://github.com/multiformats/multiaddr). 

Multiprotocol is generic in that any protocol can adapt the `code`s used for their own protocol.
**A namespace is used to differentiate between protocols, this number should be arbitrary enough to not cause overlap**.

<!--
This repository contains the [multiprotocol definition](./multiprotocol.csv) used by [vac](https://vac.dev), 
the [go implementation](https://github.com/vacp2p/go-multiprotocol) however is generic and therefore anyone can implement their own table.
-->

## Protocol Definitions

Protocol values are defined using a csv table, current implementations support this standard. 
The CSV file MUST contain a header of the fields defined, these are `code`, `size`, `name` and `comment`.
Their values MUST be as follows:

| field       | description                                                                                                                     |
| :---------: | :------------------------------------------------------------------------------------------------------------------------------ |
| **code**    | This field contains the code identifying the key.                                                                               |
| **size**    | This field identifies the expected keys size, it can be any number or `V`, indicating that the value itself is length prefixed. |
| **name**    | The human readable name of the field.                                                                                           |
| **comment** | Any developer related comments for the field.                                                                                   |

The valid `size` values are:
 - `0` - there is no value.
 - `V` - meaning the value is length prefixed.
 - `>0` - any number above 0 is the explicit length of the field.

Below is an example valid csv table, the values in it will be used further in the examples within this document.

```csv
code, size, name, comment
42,   0,    vac,  namespace
2,    V,    waku,
3,    V,    store,
4,    V,    relay, 
```

## Specification

A multiprotocol, like a multiaddr is a recursive (TLV)+ (type-length-value repeating) encoding, 
except we add the addition of prefixes such as the `namespace`, `protocol` and `version`. It has two forms:
  - a human-readable version to be used when printing to the user (UTF-8)
  - a binary-packed version to be used in storage, transmissions on the wire, and as a primitive in other formats.
  
### Human-readable

Below is a psuedo regex of the encoding itself.

```regexp
/<namespace>/<protocol>/<version>(/<capability>/<version>|<capability>)+
```

 - `namespace` - the namespace represents the protocol namespace. In our case this would be `vac`.
 - `protocol` - the protocol represents the specific protocol we are identifying, in our case `waku`.
 - `version` - the version represents the global protocol version, this can be any integer.

Next we have our repeating fields:

 - `capability` - this represents a specific supported capability, for example `store` or `relay`.
 - `version` - this field is not required, if the `size` for a specific capability is `0`, it represents the supported version,
 this can either be latest or earliest, we leave this to implementers to decide.

Now, let's look at some human-readable examples:

```
/vac/waku/2
/vac/waku/2/relay/2
/vac/waku/2/store/1
```

### Machine-readable

Below is a psuedo regex of the binary-packed encoding itself.

```regexp
<namespace uvarint><protocol uvarint><version uvarint>(<protoCode uvarint><version uvarint>|<protoCode uvarint>)+
```

@TODO

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

## Implementations

 - [go-multiprotocol](https://github.com/vacp2p/go-multiprotocol)
 
## Credits

This protocol was inspired by [@oskarth](https://github.com/oskarth) and based on the work of [multiformats](https://github.com/multiformats).
Its current form was conceived by [decanus](https://github.com/decanus).
