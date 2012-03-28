sparql.js
=========

Features
--------

* issuing SPARQL SELECT or ASK queries using the SPARQL Protocol for RDF extended with a parameter named output. Joseki as deployed on SPARQLer currently supports:
    * No output specified; results are returned in the SPARQL Query Results XML Format with a MIME type of application/sparql-results+xml.
    * output=xml or output=sparql; results are returned in the SPARQL Query Results XML Format with a MIME type of text/plain.
    * output=json; results are returned via the JSON serialization with a MIME type of text/javascript.
    * output=any-other-value; results are returned in RDF/XML with MIME type text/plain as a graph using the DAWG's result-set vocabulary for test cases.
* automatically validating and parsing JSON return values into JavaScript objects.
* providing several query wrapper methods and accompanying result transformations to enable direct access to single-valued query results, vectors of query results, and boolean results (for ASK queries). This mechanism could be easily extended to support parsing the XML result format.
* allowing either HTTP GET or HTTP POST to be used when sending queries.
* providing distinct service and query objects such that dataset graphs, prefixes, and other settings can be set service-wide or on a per-query basis.

Example Code
------------

    var sparqler = new SPARQL.Service("http://sparql.org/sparql");
    
    // graphs and prefixes defined here
    // are inherited by all future queries
    sparqler.addDefaultGraph("http://thefigtrees.net/lee/ldf-card");
    sparqler.addNamedGraph("http://torrez.us/elias/foaf.rdf");
    sparqler.setPrefix("foaf", "http://xmlns.com/foaf/0.1/"); 
    sparqler.setPrefix("rdf", "http://www.w3.org/1999/02/22-rdf-syntax-ns#");
    	
    // "json" is the default output format
    sparqler.setOutput("json");
    
    var query = sparqler.createQuery();
    
    // these settings are for this query only
    query.addDefaultGraph(...);
    query.addNamedGraph(...);
    query.setPrefix(...);
    
    // query wrappers:
    
    // passes standard JSON results object to success callback
    query.setPrefix("ldf", "http://thefigtrees.net/lee/ldf-card#");
    query.query(
        "SELECT ?who ?mbox WHERE { ldf:LDF foaf:knows ?who . ?who foaf:mbox ?mbox }",
        {failure: onFailure, success: function(json) { for (var x in json.head.vars) { ... } ...}}
    );
    
    // passes boolean value to success callback
    query.ask(
        "ASK ?person WHERE { ?person foaf:knows [ foaf:name "Dan Connolly" ] }",
        {failure: onFailure, success: function(bool) { if (bool) ... }}
    ); 
    
    // passes a single vector (array) of values 
    // representing a single column of results 
    // to success callback
    query.setPrefix("ldf", "http://thefigtrees.net/lee/ldf-card#");
    var addresses = query.selectValues(
        "SELECT ?mbox WHERE { _:someone foaf:mbox ?mbox }",
        {failure: onFailure, success: function(values) { for (var i = 0; i < values.length; i++) { ... values[i] ...} } }
    ); 
    
    // passes a single value representing a single 
    // row of a single column (variable) to success callback
    query.setPrefix("ldf", "http://thefigtrees.net/lee/ldf-card#");
    var myAddress = query.selectSingleValue(
        "SELECT ?mbox WHERE {ldf:LDF foaf:mbox ?mbox }",
        {failure: onFailure, success: function(value) { alert("value is: " + value); } }
    ); 
    	
    // shortcuts for all of the above 
    // (w/o ability to set any query-specific graphs or prefixes)
    sparqler.query(...);
    sparqler.ask(...);
    sparqler.selectValues(...);
    sparqler.selectSingleValue(...);
