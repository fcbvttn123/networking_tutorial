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