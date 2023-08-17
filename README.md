# CAR Mirror over HTTP v0.2.0

## Authors

* [Brooklyn Zelenka], [Fission]

## Dependencies

- [CAR Mirror]
- [CARv1]
- [CBOR]
- [HTTP/2]
- [IPLD Schema]

## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14] when, and only when, they appear in all capitals, as shown here.

# 0 Abstract

This specification describes the protocol for synchronous [CAR Mirror] over [HTTP/2].

# 1 Introduction

[CAR Mirror] describes a generic method extending CAR files to efficiently mirror some IPLD graph state on a peer machine. It is transport agnostic.

HTTP is a very mature protocol that connects the applciations and devices that developers — especially web developers — use most fequently. This spec defines how to perform multi-round, syncronous CAR Mirror over HTTP/2.

# 2. Pull

## 2.1 Endpoint

The endpoint MAY be placed at any route, but it is RECOMMENDED to be exposed at:

```http
POST /api/v0/dag/pull
```

Since multiple CID roots MAY be requested at once, this information is instead located in the Client Payload.

## 2.2 Client Pull Request

The request MUST be serialized as [CBOR]. 

```ipldsch
type PullRequest struct {
  rs [&Any]  -- Requested CID roots
  bk Integer -- Bloom filter hash count
  bb Bytes   -- Bloom filter Binary
}
```

## 2.3 Server Pull Response

The response MUST be given as a [CARv1]. If the streaming flag was set in the URL, then the CAR MUST be a [streaming CARv1].

All DAG roots SHOULD be included in the CAR header. There MUST be at least one root.

## 2.4 Status Codes

Status codes are [as defined in RFC2616 §10][RFC2616 #10], with no additional special meaning. For example, the common cases of success and lack of further CID roots would be:

* Success: `200`
* Unable to find any new root CIDs: `404`

# 3 Push

## 3.1 Endpoint

The endpoint MAY be placed at any route, but it is RECOMMENDED to be exposed at:

```http
POST /api/v0/dag/push&diff={ipns | dnslink | cid}
```

### 3.1.2 `diff` Parameter

The `diff` field is OPTIONAL. It represents a related CID to the one being pushed, and MAY be an IPNS record, DNSLink, or CID. The complete URI MUST be provided.

While a mutable pointer (IPNS or DNSLink) MAY be resolved to a CID before the request is constructed, it is RECOMMENDED that the pointer be given when available. This strategy gives the Server more flexibility for tracking the last value that they've seen for that pointer, and conveys more about the Client's intention.

This field is primarily useful for the narrowing step, and especially during a cold call. Since the Client does not have knowledge of everything in the Server's store, this field provides a hint for the Server of what (finite) context to include as part of the Bloom in their response.

This field MUST NOT be interpreted as a CID root being sent.

## 3.2 Client Push Request

```ipldsch
type PushRequestHeader struct {
  bk Integer -- Bloom filter hash count
  bb Bytes   -- Bloom filter Binary
}

type PushRequest struct {
  hd PushRequestHeader -- Nested (extensible) header map
  pl CARv1             -- Data payload
} representation tuple
```

The data payload (`pl`) MUST be encoded as a [CARv1]. If the streaming flag was set in the URL, then the CAR MUST be a [streaming CARv1].

All DAG roots SHOULD be included in the CAR header. There MUST be at least one root.

## 3.3 Server Push Response

The response MUST be serialized as [CBOR]. 

```ipldsch
type PushResponse struct {
  sr [Link]  -- Incomplete subgraph roots
  bk Integer -- Bloom filter hash count
  bb Bytes   -- Bloom filter Binary
}
```

## 3.4 Server Status Codes

Status codes are [as defined in RFC2616 §10][RFC2616 #10], with no additional special meaning. For example, a few common cases would include:

* Complete: `200`
* Success: `202`

<!-- External Links -->

[Brooklyn Zelenka]: https://github.com/expede
[CAR Mirror]: https://github.com/wnfs-wg/car-mirror
[CARv1]: https://ipld.io/specs/transport/car/carv1/
[CBOR]: https://cbor.io/
[Fission]: https://fission.codes
[HTTP Streaming]: https://datatracker.ietf.org/doc/html/rfc7540#section-5
[HTTP/2]: https://datatracker.ietf.org/doc/html/rfc7540
[IPLD Schema]: https://ipld.io/docs/schemas/
[RFC2119]: https://datatracker.ietf.org/doc/html/rfc2119
[RFC2616 #10]: https://www.rfc-editor.org/rfc/rfc2616#section-10
[streaming CARv1]: https://ipld.io/specs/transport/car/carv1/#performance
