# NAME

Plack::App::RDF::Files - serve RDF data from files

# STATUS

[![Build Status](https://travis-ci.org/nichtich/Plack-App-RDF-Files.png)](https://travis-ci.org/nichtich/Plack-App-RDF-Files)
[![Coverage Status](https://coveralls.io/repos/nichtich/Plack-App-RDF-Files/badge.png)](https://coveralls.io/r/nichtich/Plack-App-RDF-Files)
[![Kwalitee Score](http://cpants.cpanauthors.org/dist/Plack-App-RDF-Files.png)](http://cpants.cpanauthors.org/dist/Plack-App-RDF-Files)

# SYNOPSIS

    my $app = Plack::App::RDF::Files->new(
        base_dir => '/path/to/rdf/
    );

    # Requests URI            =>  RDF files
    # http://example.org/     =>  /path/to/rdf/*.(nt|ttl|rdfxml)
    # http://example.org/foo  =>  /path/to/rdf/foo/*.(nt|ttl|rdfxml)
    # http://example.org/x/y  =>  /path/to/rdf/x/y/*.(nt|ttl|rdfxml)

# DESCRIPTION

This [PSGI](https://metacpan.org/pod/PSGI) application serves RDF from files. Each accessible RDF resource
corresponds to a (sub)directory, located in a common based directory. All RDF
files in a directory are merged and returned as RDF graph.

# CONFIGURATION

- base\_dir

    Mandatory base directory that all resource directories are located in.

- base\_uri

    The base URI of all resources. If no base URI has been specified, the
    base URI is taken from the PSGI request.

- file\_types

    An array of RDF file types (extensions) to look for. Set to
    `['rdfxml','nt','ttl']` by default.

- include\_index

    By default a HTTP 404 error is returned if one tries to access the base
    directory. Enable this option to also serve RDF data from this location.

- index\_property

    RDF property to use for listing all resources connected to the base URI (if
    <include\_index> is enabled).  Set to `rdfs:seeAlso` by default. Can be
    disabled by setting a false value.

- path\_map

    Optional code reference that maps a local part of an URI to a relative
    directory. Set to the identity mapping by default.

- namespaces

    Optional namespaces for serialization, passed to [RDF::Trine::Serializer](https://metacpan.org/pod/RDF::Trine::Serializer).

# METHODS

## files( $env | $req | $str )

Get a list of RDF files that will be read for a given request. The request can
be specified as [PSGI](https://metacpan.org/pod/PSGI) environment, as [Plack::Request](https://metacpan.org/pod/Plack::Request), or as partial URI
that follows `base_uri` (given as string). The requested URI is saved in field
`rdf.uri` of the request environment.  On success returns the base directory
and a list of files, each mapped to its last modification time.  Undef is
returned if the request contained invalid characters (everything but
`a-zA-Z0-9:.@/-` and the forbidden sequence `../` or a sequence starting with
`/`) or if the request equals ro the base URI and `include_index` was not
enabled.

## call( $env )

Core method of the PSGI application.

The following PSGI environment variables are read and/or set by the
application.

- rdf.uri

    The requested URI

- rdf.iterator

    The [RDF::Trine::Iterator](https://metacpan.org/pod/RDF::Trine::Iterator) that will be used for serializing, if
    `psgi.streaming` is set. One can use this variable to catch the RDF
    data in another post-processing middleware.

- rdf.files

    An hash of source filenames, each with the number of triples (on success)
    as property `size`, an error message as `error` if parsing failed, and
    the timestamp of last modification as `mtime`. `size` and `error` may
    not be given before parsing, if `rdf.iterator` is set.

If an existing resource does not contain triples, the axiomatic triple
`$uri rdf:type rdfs:Resource` is returned.

## negotiate( $env )

This internal methods selects an RDF serializer based on the PSGI environment 
variable `negotiate.format` (see [Plack::Middleware::Negotiate](https://metacpan.org/pod/Plack::Middleware::Negotiate)) or the 
`negotiate` method of [RDF::Trine::Serializer](https://metacpan.org/pod/RDF::Trine::Serializer). Returns first a 
[RDF::Trine::Serializer](https://metacpan.org/pod/RDF::Trine::Serializer) on success or `undef` on error) and second a 
(possibly empty) list of HTTP response headers.

# SEE ALSO

Use [Plack::Middleware::Negotiate](https://metacpan.org/pod/Plack::Middleware::Negotiate) to add content negotiation based on
an URL parameter and/or suffix.

See [RDF::LinkedData](https://metacpan.org/pod/RDF::LinkedData) for a different module to serve RDF as linked data.
See also [RDF::Flow](https://metacpan.org/pod/RDF::Flow) and [RDF::Lazy](https://metacpan.org/pod/RDF::Lazy) for processing RDF data.

See [http://foafpress.org/](http://foafpress.org/) for a similar approach in PHP.

# COPYRIGHT AND LICENSE

Copyright Jakob Voss, 2014-

This library is free software; you can redistribute it and/or modify it under
the same terms as Perl itself.
