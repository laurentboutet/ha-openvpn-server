# Changelog

## [1.0.0] - 2026-01-25

### Added
- Initial release
- Complete PKI infrastructure with EasyRSA 3.x
- OpenVPN server with industry-standard security
- Client certificate generation tool
- Client certificate revocation with CRL
- Client listing and management
- Automatic client configuration file generation
- Inline certificate embedding
- IPv4 forwarding and NAT configuration
- Configurable encryption (AES-256-GCM, AES-128-GCM)
- DNS push configuration
- Route push configuration
- Connection monitoring and logging
- S6 supervision for service management
- Comprehensive documentation

### Security
- TLS 1.2+ enforcement
- 2048-bit RSA certificates
- HMAC authentication (SHA256/SHA384/SHA512)
- TLS-auth for additional security layer
- Certificate revocation list support

## Future Enhancements

### Planned for 1.1.0
- Web UI for certificate management
- Automatic backup integration
- Email notifications for new connections
- Geographic IP restriction options
- 2FA integration (Google Authenticator)

### Under Consideration
- IPv6 full support
- OpenVPN 3.x client profile format
- Integration with Home Assistant authentication
- Mobile push notifications via HA
- Automatic DDNS integration
