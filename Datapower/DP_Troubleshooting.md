DP Runtime Observations:
=========================
* Web Service Proxy: if you send a SOAP request with and endpoint that does not match the WSP, the WSP sends SOAP fault immediately once the first part of the HTTP request is received that has the URI( phrase line and headers).
It does not wait until the body is received.

* When sending a HTTP request from datapower and the server sends RST packet, datapower shows error "cannot parse http headers" considering the response is empty.
