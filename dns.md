# Table of contents

- [Table of contents](#table-of-contents)
- [DNS Lookup Process](#dns-lookup-process)
  - [Check Local Cache](#check-local-cache)
  - [Check Host Files](#check-host-files)
  - [Query `DNS Resolver`](#query-dns-resolver)
  - [Resolver contacts the `Root Server`](#resolver-contacts-the-root-server)
  - [Resolver contacts the `TLD Server`](#resolver-contacts-the-tld-server)
  - [Resolver contacts the `Authoritative Server` (Nameserver)](#resolver-contacts-the-authoritative-server-nameserver)
- [Nameserver, Registrar and NS Record](#nameserver-registrar-and-ns-record)
  - [Registrar and Nameserver](#registrar-and-nameserver)
  - [NS Records](#ns-records)
- [DNS Packet Analysis](#dns-packet-analysis)
  - [Basic Packet Information](#basic-packet-information)
  - [DNS Header Fields](#dns-header-fields)
  - [Question Section](#question-section)
  - [Answer Section](#answer-section)




# DNS Lookup Process

## Check Local Cache

- Browser Cache

- Operating System (OS) Cache

- Router Cache

## Check Host Files

- Host File is manually configured mappings of domain names to IP addresses

## Query `DNS Resolver`

- If no IP address is found locally, the request is sent to a DNS Resolver

- The Resolver is a server from our ISP or a public DNS service like Google DNS (`8.8.8.8`) or Cloudflare (`1.1.1.1`)

## Resolver contacts the `Root Server`

- **Resolver contacts the Root DNS Server** which is the starting point for DNS lookups

- The Root server doesn’t know the exact IP address

- It directs the query to the `Top-Level Domain (TLD)` Server responsible for `.org` or `.com`

## Resolver contacts the `TLD Server`

- **Resolver sends the query to the TLD Server** for `.org` domains

- The TLD server handles domain names ending in `.org` and knows where to find the `authoritative nameserver`

## Resolver contacts the `Authoritative Server` (Nameserver)

- Nameserver is responsible for storing `DNS records` for the domain, including the mapping of the domain name to its IP address




# Nameserver, Registrar and NS Record

## Registrar and Nameserver

- A registrar (Domain.com, GoDaddy, Namecheap, etc.) is where you register ownership of the domain (`example.com`)

- The registrar maintains the domain registration and communicates with the registry for the TLD (`.com`)

- At the registrar, you can specify which **nameservers** are authoritative for your domain

    - `ns1.cloudflare.com`
    
    - `ns2.cloudflare.com`

    - The registrar publishes these nameservers to the `.com` registry

## NS Records

- There are actually two places where NS records exist

- Delegation NS records at the parent zone (TLD)

    - The .com registry stores: `example.com NS ns1.cloudflare.com`

    - You do not manually create delegation NS records at the TLD yourself

- NS records inside the authoritative zone itself

    - The authoritative DNS servers (nameserver) for `example.com` also contain NS records: `example.com NS ns1.cloudflare.com`




# DNS Packet Analysis

## Basic Packet Information

- Protocol: DNS

- Transport Layer: UDP Port 53

## DNS Header Fields

- Transaction ID (TXID)

    - Used to match a response with its query

    - Example - Transaction ID: 0x4f32

    - Query and response should have the same TXID

- Flags

    - These reveal the packet's purpose

    - 0 = Query, 1 = Response

## Question Section

- Query Name: `www.google.com`

- Query Type: MX, AAAA, A, NS, CNAME,...

## Answer Section

- Returned Record: `www.google.com A 142.250.190.68`

- TTL (Time To Live): 300 means Resolver can cache the answer for 300 seconds

- Authority Section: contains NS Records and Nameserver (`google.com NS ns.google.com`)