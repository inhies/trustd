Trustd
======

A peer to peer framework for disseminating information based on a web of trust. Currently this is aimed at providing a DNS solution for the meshnet, however it is designed to be flexible enough for use with any type of data or resource.

Goals
-----

  1. Store the location of a resource by distributing it among a network of peers so that no central point of failure exists.
  2. Utilize dynamically generated trust ratings of peers to ensure that users are directed to their most trusted instance of the resource.
  3. Secure the pointers to the resources location using crytpographically signed documents.
  4. Enable warnings and alerts to be propagated through the network in the event a server is seized or becomes malicious.

Key Points
----------

  * Clients choose which servers they trust and how much they trust them. Servers choose which location for the resource they trust the most and provide that to the client. The client then chooses which server it trusts the most and accepts the location for the resource from that single server.
  * Every client MAY run a server instance and maintain their own database but it is not a requriement.
  * Every server node maintains a local copy of the location of the most trusted source for the requested resource.
    * Resources may be referenced by a pointer stored in the database, cached locally, or stored via DHT with the hash stored in the database.
    * In the event of caching, it should be refreshed periodically and if an update comes through the network then it immediately overwrites the cache once the updated is accepted.

Network Operation
-----------------

### Resource Creation

When a client submits a request to create a new database entry, the following steps are taken:

 * Server checks the local database first and if no reference is found it then sends out a request to the entire network for the resource. If no pre-existing version is found, accept the resource as new and enter it in the database. The server will then broadcast the record to the network so that the other systems will update themselves.
   * If the resource already exists on the network, the server SHOULD refuse to create a new record. 
     * This doesn't prevent the record from being created through other means or other servers, but  this maintains the servers integrity by refusing to create duplicate content. 
     * The creator could go elsewhere and try again and perhaps find a node willing to create it, or he could create his own server and attempt to peer with the existing network. In this case, the duplicate content will be created, but the new server will have a non existant trust rating so his content will always be overridden by the preexisting version from other servers.
   * **If the server receives a notification of new resource for an existing record, it should....?** (This is still up for debate)
     * Lower the trust rating of the node that sent the request AND refuse to accept it?
     * Inform the sending server that we already have a record for that entry and let them figure it out?
     * Check the trust rating of the node that sent the request against the node that sent the initial resource for the already existing resource and update the record if the new record comes from a more trusted source?
     * Something else entirely? 


### Resource Updates

 * Resource updates are pushed around the network. If a client receives an update for which it has more than one trusted source, it will wait until the most trusted source has the update, then accept it and forward it on to peers. (pending simulation) (**note:** I no longer believe this solution will work)
 * Alternatively, it may accept the update once it comes from a node with trust above a certain threshold, even if it is not the most trusted node available.


### Resource Expiration

 * Servers will periodically check on the resource.
   * Verify that it still exists.
     * Resources will expire to keep from having a needlessly large database.
   * Verify that the resource data you have matches the data your peers have.
 * Allow withdrawl of the resource from the system.
   * Resource servers will use periodically rotated keys to sign their information. These periodic keys will be signed by a master key that SHOULD be stored offline in a secure location and only accessed when required.
   * Both the master key and the periodic key SHALL be accepted to sign documents for the server, however the server SHOULD prefer to use the periodic keys in case of a security breach. 
   * In the event of a hostile server takeover, the server operator SHALL access their master key and broadcast an alert message. Handling of this message is TBD.


### Resource Revocation

 * In the event of an emergency, for example a compromised server, the owner may revoke the subkeys to prevent further damage to the network. 
   * Server sends a signed revocation method for each subkey it wants revokes using its master key.
   * All other servers see this and immediately refuse to use the requested subkeys to update records.
   * Server generates new subkeys, signs them with the master key, and re-distributes the information.
   * All other servers update their records to use the new subkeys for the applicable records.
