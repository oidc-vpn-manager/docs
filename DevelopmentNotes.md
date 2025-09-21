# Development Notes for OpenVPN Manager

## Clarifications on the roles of each of the components

### Overall Service

This is an application to provide a secure access to computers and services in a managed environment by issuing configuration files for OpenVPN.

It offers semi-automatic issuance of configuration files for user devices, such as computers, phones and tablet devices. It also provides automated issuance of server configuration files for the entry points into the network.

Once certificates are issued for use in the configuration files, the fact that the certificate is issued is recorded in an append-only "Certificate Transparency" (CT) log. Only specific components are allowed to request a "Write Event" on that log. Once an event is recorded, it will NEVER be tampered with. It is preferrable to issue two or more "Write Events" for a loggable event, in the event that a failure occurs, than it is for a single record to be lost.

The application is also designed to be scalable. There may be one or more horizontal components at each layer, so this needs to be considered when writing code and considerations for the selections made.

Security is in the forefront of design intentions. We aim for full coverage against the OWASP top 10, the API Security top 10 and Infrastructure top 10 lists. When making a coding choice, ensure you fully consider the implications of the coding choice against those standards.

The application uses the TimeZone UTC at all times, and uses ISO 8601 style date and timestamps.

## Development Documentation Standards

Following comprehensive docstring audit and function mapping analysis:

### Function Documentation Requirements
- **All functions require comprehensive docstrings** including purpose, parameters, return values, exceptions, and examples
- **Security considerations** must be documented for cryptographic, authentication, and input validation functions
- **Timing attack prevention** and constant-time comparison usage must be explained
- **Cryptographic algorithm choices** require rationale documentation

### Code Traceability
- **Complete function call mapping** is maintained in `docs/Function Call Mapping.md`
- **Route execution chains** from API endpoints to leaf functions are documented in `docs/Function Calls From Routes.md`
- **Function usage analysis** shows 96%+ utilization rate indicating excellent code health
- **Technical debt management** through periodic unused function analysis

### Documentation Quality Metrics
- Critical API endpoints have comprehensive documentation with examples
- Security functions include timing attack prevention explanations
- Template rendering functions document injection prevention measures
- Database models include relationship and constraint documentation

### Frontend

This service serves as the "Front Door" to all applications. It offers a web interface which is authenticated to via OpenID Connect (OIDC). There are multiple roles for the Web Interface.

* A User who can request new configuration files (a "Configuration Profile") and revoke their own certificates on a one-by-one basis, or all of their issued certificates at once.
* An Auditor, who, in addition to the role of the User, can READ-ONLY the CT Log.
* A Certificate Administrator, who in addition the role of the Auditor, can also revoke individual certificates, revoke all certificates issued to an individual user
* A System Administrator, who in addition to the role of the User, can also create a "Pre-Shared Key" (PSK) that can be used to allow an OpenVPN server to request a "Server Configuration Bundle" (sometimes shortened to "Server Bundle")
* A Service Administrator, who can perform all of these roles.

There is also an API which offers three functions, the ability to request a Server Bundle by using the PSK, to request, unauthenticated, a certificate revocation list (CRL) which can be used by the OpenVPN servers (this functionality will be explained later) and finally, to service the request a semi-automated request for a new Configuration Profile (again explained later).

#### Configuration Files

The service issues two types of configuration files. The first, and simplest, is the "Configuration Profile", which is requested by a user. The service uses the OIDC "group" allocation to determine which configuration template to use, and mixes in any number of "Options" from the user interface (or none if being serviced by the API). The templates are stored in a directory that the service can access in the format "0000.GroupName.ovpn", where the digits at the start indicates the priority with 0 being selected first and a higher number being selected last. The first matching group name (or "Default" if none match), ordered by those digits, is retrieved and combined with the options using the Jinja2 Templating Language, and is then returned to the user. If the service has been configured with a "TLSCert" value that matches the TLSCert V1 format, this will also be mixed into the template. If the service has been configured with a TLSCert value which matches the TLSCert V2 format, the TLSCert value will be mixed with the issued client certificate to generate a TLSCert V2 Client value.

The second type of configuration file is the "Server Bundle" which is a TAR file containing one or more configuration templated files. These are stored in a directory the service has access to in the format "Selection.0000.ovpn", where the server will index all the template files, group them by the string before the digits (so in the case mentioned before "Selection" would be the string) and make that list of templates a drop-down when creating the PSK. When the server connects to the Bundle API endpoint, the PSK authentication allows the service to build the all the configuration files which match the selected template and put them in a file with the Certificate Authority chain PEM files, the server's unique certificate and key, and, if one is configured, the TLSCert key.

The service also issues a CRL file, which is obtained by reading the CT records, recreating the index.txt file, and then requesting the Signing Service sign the CRL, at which point it can be issued to servers or auditors.

### Signing Service

This service has very few actions; a request to sign a Certificate Signing Request and the Revocation action. This is because the Signing Service is the only service which can request the CT Log perform a write action. The Signing Service has an API Key which is shared with the Frontend to trigger these actions, and a separate API Key which is shared with the CT Service to request write actions.

### Certificate Transparency (CT) Service

This is a READ-ONLY service, except for Signing Service processes which have an API key that permits them to use the APPEND-ONLY service. It's sole purpose is to ensure that any signing actions by one of the Signing Service processes is recorded into the CT log along with relevant metadata. There must be no prohibition on writing entirely or partially duplicated data, for example, the CT service may receive 5 requests to create the same record, and should record all 5 requests. When the read request is submitted to the CT service, the CT service will collapse any duplicated record following the rules;

1. A revoke action takes priority over an issue action
2. Newest record of that action type takes priority when returning data.

### Public/Private Key Cryptography Issuing (PKI) Tool

To ensure that the Signing Service Certificate Authority (CA) chain can be replaced at any time, an Offline Root CA will be used. The PKI Tool is used to create the initial CA certificate and key and then generate one or more Intermediate CA certificates and keys, which can be distributed to the Signing Service.

### Get OpenVPN Config Tools

These command line tools are used by server administrators to automatically provision OpenVPN servers, and by OpenVPN users and server administrators to automatically request new OpenVPN Configuration Files for themselves or the computers they manage.

When used with a PSK there is no user interaction required, so it can be used in post-provisioning services like Ansible or Puppet, or during machine initialization using services like Cloud-Init.

When used without a PSK, the tool invokes the user's browser to perform an OIDC authentication, and then redirects the completed OIDC authentication flow back to a specified open port where a token can be used to request an un-optioned OpenVPN profile for semi-automatic provisioning on a regular basis.

## Expected workflows

### User requests a Configuration Profile in a web browser.

1. User accesses the OpenVPN Manager "Frontend" front page.
2. If the user has not authenticated, an OIDC authentication flow is started.
3. Following a successful authentication request from an authorized user in the OIDC service, the user is redirected back to the OpenVPN Manager Frontend. The OIDC request returns the user's details and one or more groups they are affiliated to. These group affiliations are used to determine what, if any, role above "User" they are granted.
4. On the page they are returned to, the user is able to generate a Configuration Profile. They optionally select zero or more options from the provided list, and click "Generate". The supplied file has the relevant MIME Type to allow automatic ingestion into an OpenVPN application, should one exist.

### User's device obtains a Configuration Profile using the Get OpenVPN Config script

1. The script is triggered by a provisioning service (like Ansible, Puppet), an MDM service (like Kandji) or a service which monitors the half-life of the certificate and identifies it is required to be renewed.
2. The script uses the browser to access a token-issuing endpoint on the Frontend service with relevant parameters to request a response to the script. This may trigger an OIDC authentication flow, or may simply redirect to the token receiving endpoint on the local machine, started by the script, which in turn closes the web browser tab.
3. This token is used to access the profile API endpoint and request an unmodified Configuration Profile, which is then stored in the correct location on disk for that user to install it into their OpenVPN client.

## API Expectations

All API endpoints should be versioned, and should be presented at the URL path starting `/api/vX/` where "X" is the version number. APIs should be documented using Swagger files, exposed on the API Version endpoint, and an index page at `/api` should point to the version of the API which is implemented and the swagger document for that version.

Swagger files should contain examples of both successful and unsuccessful responses, which should be derived from testing results.

## Testing Methodologies

This suite of services and tools were designed to be highly tested, and all tests must pass 100% of the time, where a pass means no errors, warnings or critical issues and that none of the tests were skipped. There must be 100% coverage of the code by tests. LIMITED and EXPLICITLY AGREED exceptions MAY be delivered, but ONLY with the EXPLICIT APPROVAL of the project lead. Exceptions MUST be documented.

### Security Testing Standards

Beyond functional testing, all services include comprehensive security testing:
- **Red Team Testing**: Adversarial attack simulation against all services
- **Blue Team Testing**: Defensive security validation and monitoring
- **Bug Bounty Testing**: Common vulnerability patterns and edge cases
- **OWASP Top 10 Coverage**: Complete testing against web application security risks
- **Input Validation**: Edge case testing including Unicode, oversized payloads, and malformed data
- **Timing Attack Prevention**: Constant-time comparison algorithms with variance testing
- **Authentication Security**: Bypass attempts and session management validation

Current security test coverage: 44 comprehensive tests across CT and Signing services, with real vulnerability discovery and remediation during development.

The test types are identified as follows:

1. Unit test: a discrete test of individual functions in scripts and tools.
2. Functional or Integration tests: a discrete test of couplings between small chains of functions, which includes barrier traversal between the outside and the inside of the service or tool, e.g. API barriers, database interactions
3. End to End (E2E) or "smoke" tests: Once the application is built and running in a testing environment with running and configured test databases, a test of each user or service interaction must be made. Web interface tests are performed using Playwright, while API requests are made using the python "Requests" library and tools are executed in subprocesses.

These tests MUST be run in full and MUST return 100% green, 100% coverage before features are declared "Complete". If a test repeatedly fails, you MUST seek advice from the project lead on how to proceed.

A full test suite run will often take a long time to run, particularly if the full E2E tests are run too. In this case, requesting the project lead to run the tests and notifying you once it is complete is a perfectly acceptable request. Results from the tests will be stored in the `suite_test_results` directory, often with the time the test was triggered stored in the log file (e.g. e2e_tests.HHMM.log). Following the test run, the docker logs for the duration of the full E2E test run will also be stored in the same directory.

Individual tests outside the E2E tests, as well as the full suite of Unit and Functional/Integration tests can be run, either by running one of these commands from the root directory of the project:

```
make test_certtransparency
make test_frontend
make test_signing
make test_get_openvpn_config
make test_pki_tool
```

Or a wider test of all of those components can be issued by running `make just_test_without_e2e`.

Individual E2E tests CAN be run at any time, but if code in the services (not the testing code) has been altered, a full restart of the Docker containers must be triggered by running `make rebuild_docker` first, followed by the individual E2E test.

If you have permission to run the full E2E suite, you will run one of these commands:

* `make test` - run all tests, including the unit and functional/integration tests for services and tools
* `make just_test_e2e` - rebuild the docker containers and then run the full E2E tests

You may find it best to run either the full test suite or the E2E test suite and put it in the "Background" while you perform other activities until it is completed.

API endpoints should be tested thoroughly too, whether that's via the Integration or E2E testing when APIs are called across service boundaries (e.g. reading the CT log from Frontend) and with pytest using requests to confirm that the Swagger documentation is accurate.
