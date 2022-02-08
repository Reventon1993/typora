amqp



amqp vs http

- REST, RPC - architecture patterns, AMQP is wire-level protocol and HTTP is application protocol which run on top of TCP/IP
- AMQP is a specific protocol while HTTP is general-purpose protocol, thus, HTTP has damn high overhead comparing to AMQP
- AMQP nature is asynchronous where HTTP nature is synchronous
- both REST and RPC use data serialization, which format is up to you and it depends of infrastructure. If you are using python everywhere I think you can use python native serialization - `pickle` which should be faster than JSON or any other formats.
- both HTTP+REST and AMQP+RPC can run in heterogeneous and/or distributed environment

Advantages of RESTful services are:

- they well-mapped on web interface
- people are familiar with them
- easy to debug (due to general purpose of HTTP)
- easy provide API to third-party services.

Advantages of AMQP-based solution:

- damn fast
- flexible
- cost-effective (in resources usage meaning)
- reliable