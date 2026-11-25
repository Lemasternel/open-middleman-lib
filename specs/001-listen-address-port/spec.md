# Feature Specification: Listen on User-Specified Address and Port

**Feature Branch**: `claude/project-thread-m9d4c6`

**Created**: 2026-11-25

**Status**: Draft

**Input**: User description: "Abrir uma comunicação utilizando a porta e o endereço que o usuário especificou. Este endereço (IP) e porta, servirão para que, futuramente, as requisições sejam redirecionadas nele."
(Open a communication endpoint using the port and address the user specified. This IP address and port will be used so that, later, requests are directed to it.)

## Overview

open-middleman is a forward web debugging proxy. Before it can inspect any
HTTP or HTTPS traffic, it needs a place where clients (browsers, operating
system proxy settings, other applications) can send their requests. This
feature lets the consuming application ask the library to open a listening
endpoint on an IP address and port chosen by the end user, keep it open, and
close it on request. Processing the traffic that arrives at the endpoint
(parsing, forwarding, TLS) is out of scope and belongs to later features.

"User" in this spec means the consuming application (for example the
open-middleman command-line tool) acting on behalf of the person who chose the
address and port.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Open the endpoint on the chosen address and port (Priority: P1)

The person running the proxy chooses an IP address and a port (for example
`127.0.0.1` and `8080`). The application passes them to the library, which
opens the endpoint there. From that moment, a client on the same machine or
network can connect to that address and port, and the application is told
that the endpoint is ready.

**Why this priority**: Without an open endpoint, no request can ever reach
the proxy. Every later feature depends on this one.

**Independent Test**: Ask the library to open an endpoint on a free port of
the loopback address, then connect to that address and port from a test
client; the connection is accepted.

**Acceptance Scenarios**:

1. **Given** a valid IPv4 address assigned to the machine and a free port, **When** the application asks to open the endpoint, **Then** the library reports success and a client can connect to that address and port.
2. **Given** a valid IPv6 address assigned to the machine and a free port, **When** the application asks to open the endpoint, **Then** the library reports success and a client can connect to that address and port.
3. **Given** the "all interfaces" address (`0.0.0.0` or `::`) and a free port, **When** the application asks to open the endpoint, **Then** clients can connect through any of the machine's addresses on that port.
4. **Given** port `0`, **When** the application asks to open the endpoint, **Then** the operating system chooses a free port and the library reports the actual port in use.
5. **Given** an open endpoint, **When** the application asks which address and port it is using, **Then** the library returns the exact address and port that clients must use.

---

### User Story 2 - Clear failure when the endpoint cannot be opened (Priority: P1)

The person may choose an address that does not belong to the machine, a port
already used by another program, or a port the current account is not allowed
to use. The library must not crash or pretend it succeeded; it tells the
application exactly why it failed so the person can pick another value.

**Why this priority**: Wrong or busy ports are the most common setup mistake
for a local proxy; a silent or vague failure leaves the person stuck. It is
equally critical to Story 1.

**Independent Test**: Occupy a port with a test listener, then ask the library
to open an endpoint on the same address and port; the library returns a
"port in use" error and the process keeps running.

**Acceptance Scenarios**:

1. **Given** a port already in use on the chosen address, **When** the application asks to open the endpoint, **Then** the library returns an "address already in use" error naming the address and port.
2. **Given** an IP address that is not assigned to any interface of the machine, **When** the application asks to open the endpoint, **Then** the library returns an "address not available" error.
3. **Given** a port the current account is not allowed to use, **When** the application asks to open the endpoint, **Then** the library returns a "permission denied" error.
4. **Given** a text that is not a valid IP address (for example `999.1.1.1` or `localhostt`) or a port outside 0 to 65535, **When** the application asks to open the endpoint, **Then** the library returns an "invalid address" or "invalid port" error before trying to open anything.
5. **Given** any failure above, **When** the error is returned, **Then** no endpoint is left half-open and the application can retry with other values.

---

### User Story 3 - Accept incoming connections and close the endpoint (Priority: P2)

While the endpoint is open, it keeps accepting client connections, one after
another and several at the same time, so later features can read the requests
from them. When the person stops the proxy, the application asks the library
to close the endpoint; the address and port become free again.

**Why this priority**: Opening the endpoint is only useful if connections can
be handed on and if the port can be released cleanly. It builds on Stories 1
and 2.

**Independent Test**: Open an endpoint, connect several test clients, check
that each connection is handed to the application, close the endpoint, then
open a new endpoint on the same address and port; it succeeds.

**Acceptance Scenarios**:

1. **Given** an open endpoint, **When** a client connects, **Then** the library hands the new connection, with the client's address and port, to the application.
2. **Given** an open endpoint, **When** several clients connect at the same time, **Then** every connection is accepted and handed on.
3. **Given** an open endpoint, **When** the application asks to close it, **Then** the library stops accepting new connections and the address and port are released.
4. **Given** a closed endpoint, **When** a client tries to connect to its former address and port, **Then** the connection is refused.
5. **Given** an endpoint that was just closed, **When** the application opens a new endpoint on the same address and port, **Then** it succeeds.

---

### Edge Cases

- The address is given with surrounding spaces or in IPv6 bracket notation (`[::1]`): spaces are rejected as invalid; brackets are accepted for IPv6.
- A host name such as `localhost` is given instead of an IP address: rejected as "invalid address" (only IP addresses are accepted, see Assumptions).
- The application asks to open a second endpoint while one is already open on the same address and port: the second request fails with "address already in use" and the first endpoint is not affected.
- The application asks to close an endpoint that is already closed: the call succeeds without effect and is logged at debug level.
- A failure happens while accepting one connection (for example the client disconnects mid-handshake or the system runs out of handles): the error is logged and reported, and the endpoint keeps accepting other connections.
- On Windows 8 Embedded, the firewall may block inbound connections on a non-loopback address; the endpoint still opens, and the library does not change firewall settings.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The library MUST let the application open a listening endpoint by giving an IP address and a port.
- **FR-002**: The library MUST accept IPv4 and IPv6 addresses, including the "all interfaces" addresses `0.0.0.0` and `::`.
- **FR-003**: The library MUST validate the address and port before opening anything and MUST reject invalid values with a typed error that says which value is wrong.
- **FR-004**: The library MUST accept port `0` as "let the operating system choose" and MUST report the port actually assigned.
- **FR-005**: The library MUST let the application read the exact address and port the open endpoint uses.
- **FR-006**: When the endpoint cannot be opened, the library MUST return a typed error that distinguishes at least: invalid address, invalid port, address already in use, address not available on this machine, permission denied, and other operating system errors (carrying the system's error detail).
- **FR-007**: The library MUST NOT abort or crash the process on any failure in this feature, and MUST NOT leave a partly opened endpoint after a failure.
- **FR-008**: While open, the endpoint MUST accept incoming client connections and hand each one, together with the client's address and port, to the application.
- **FR-009**: The endpoint MUST accept several simultaneous client connections.
- **FR-010**: A failure on a single incoming connection MUST NOT close the endpoint or affect other connections.
- **FR-011**: The library MUST let the application close the endpoint; after closing, no new connection is accepted and the address and port are released so they can be opened again immediately.
- **FR-012**: Closing an endpoint that is already closed MUST succeed without effect.
- **FR-013**: The library MUST NOT change system proxy settings or firewall rules; pointing clients at the endpoint is the person's or the application's job.
- **FR-014**: The feature MUST work on Windows 8 Embedded (x86_64), the main target.

### Logging Requirements

Per Constitution Principle VI, the library MUST emit these records:

- **LOG-001** (info): endpoint opened, with the requested and the actual address and port.
- **LOG-002** (error): endpoint failed to open, with the address, port and error kind, before the error is returned.
- **LOG-003** (warning): address or port rejected by validation, with the rejected value.
- **LOG-004** (debug): connection accepted, with the client's address and port.
- **LOG-005** (warning): failure while accepting a single connection, with the error detail; the endpoint stays open.
- **LOG-006** (info): endpoint closed, with its address and port.
- **LOG-007** (debug): close requested on an endpoint that was already closed.
- **LOG-008** (trace): each open and close request as received from the application, with its arguments.

### Key Entities

- **Listen Address**: the IP address (IPv4 or IPv6) and port the person chose. The port may be `0` to let the system choose.
- **Listening Endpoint**: an open endpoint bound to a Listen Address. Has a state (open or closed) and the actual address and port in use.
- **Client Connection**: a connection accepted by the endpoint, with the client's address and port. Handed to the application; reading and forwarding requests on it are later features.
- **Endpoint Error**: the typed reason an open, accept or close operation failed (see FR-006).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: On a free port, the endpoint is ready to accept connections in under 1 second after the application asks for it, on Windows 8 Embedded.
- **SC-002**: 100% of the failure cases listed in User Story 2 return the matching error kind, and the process keeps running in every case.
- **SC-003**: The endpoint accepts at least 100 simultaneous client connections without refusing any of them.
- **SC-004**: After the endpoint is closed, the same address and port can be opened again within 1 second, in 100% of attempts.
- **SC-005**: Every operation and failure path in this feature produces the log record listed in the Logging Requirements, verified by tests.
- **SC-006**: A person who picked a busy or wrong port can tell from the error alone which value to change, without reading logs.

## Assumptions

- "Requests are directed to it" means the endpoint is where clients send their traffic to the proxy (the address set as the proxy in a browser or in the system settings). The upstream servers that requests are forwarded to are a later feature.
- Only literal IP addresses are accepted; resolving host names such as `localhost` is out of scope for this feature.
- One endpoint per open request; the application may open several endpoints on different addresses or ports if it wants to.
- No authentication or access control on incoming connections in this feature; restricting who may connect is done by choosing the address (for example loopback only).
- What happens to the data sent on accepted connections (HTTP parsing, forwarding, TLS interception) is defined by later features.
- Reading the address and port from the person (command-line arguments, configuration files) is the consuming application's job, not the library's.
- Default values (such as `127.0.0.1:8080`) are chosen by the consuming application; the library requires both values to be given.
