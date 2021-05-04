.. _doc_routinator_interactive:

Running Interactively
=====================

Routinator can perform RPKI validation as a one-time operation and print a
Validated ROA Payload (VRP) list in various formats, or it can return the
validity of a specific announcement. These functions are accessible on the
command line via the following subcommands:

:subcmd:`vrps`
     Fetches RPKI data and produces a Validated ROA Payload (VRP) list in the
     specified format.

:subcmd:`validate`
     Outputs the RPKI validity for a specific announcement by supplying
     Routinator with an ASN and a prefix.

Printing a List of VRPs
-----------------------

.. versionadded:: 0.9
   The ``jsonext`` format

Routinator can produce a Validated ROA Payload (VRP) list in many different
formats, which are either printed to standard output or saved to a file:

csv
      The list is formatted as lines of comma-separated values of the prefix in
      slash notation, the maximum prefix length, the autonomous system number,
      and an abbreviation for the trust anchor the entry is derived from. The
      latter is the name of the TAL file  without the extension *.tal*. This is
      the default format used if the :option:`--format` option is not provided.      
csvcompat
       The same as csv except that all fields are embedded in double quotes and
       the autonomous system number is given without the prefix AS. This format
       is pretty much identical to the CSV produced by the RIPE NCC RPKI 
       Validator.
csvext
      This is an extended version of the *csv* format, which was used by the
      RIPE NCC RPKI Validator 1.x. Each line contains these comma-separated
      values: the rsync URI of the ROA the line is taken from (or "N/A" if it
      isn't from a ROA), the autonomous system number, the prefix in slash
      notation, the maximum prefix length, and lastly the not-before and
      not-after date of the validity of the ROA.
json
      The list is placed into a JSON object with a single element *roas* which
      contains an array of objects with four elements each: The autonomous
      system number of the network authorised to originate a prefix in *asn*,
      the prefix in slash notation in *prefix*, the maximum prefix length of the
      announced route in *maxLength*, and the trust anchor from which the
      authorisation was derived in *ta*. This format is identical to that
      produced by the RIPE NCC Validator except for different naming of the
      trust anchor. Routinator uses the name of the TAL file without the
      extension *.tal* whereas the RIPE NCC Validator has a dedicated name for
      each.
jsonext
      The list is placed into a JSON object with a single element *roas* which
      contains an array of objects with four elements each: The autonomous
      system number of the network authorized to originate a prefix in *asn*,
      the prefix in slash notation  in *prefix*, the maximum prefix length of
      the announced route  in *maxLength*.

      Extensive information about the source of the object is given in the
      array *source*. Each item in that array is an object providing details of
      a source of the VRP. The object will have a type of roa if it was derived
      from a valid ROA object or exception if it was an assertion in a local
      exception file.

      For ROAs, *uri* provides the rsync URI of the ROA, *validity* provides the
      validity of the ROA itself, and *chainValidity* the validity considering
      the validity of the certificates along the validation chain.

      For assertions from local exceptions, *path* will provide the path of
      the local exceptions file and, optionally, *comment* will provide the
      comment if given for the assertion.

      Please note that because of this additional information, output in
      :option:`jsonext` format will be quite large.
      
openbgpd
      Choosing this format causes Routinator to produce a *roa-set*
      configuration item for the OpenBGPD configuration.
bird1
      Choosing this format causes Routinator to produce a roa table
      configuration item for the BIRD1 configuration.

bird2
      Choosing this format causes Routinator to produce a route table
      configuration item for the BIRD2 configuration.
rpsl
      This format produces a list of RPSL objects with the authorisation in the
      fields *route*, *origin*, and *source*. In addition, the fields *descr*,
      *mnt-by*, *created*, and *last-modified*, are present with more or less
      meaningful values.
      
      .. code-block:: text
         
         route: 93.175.146.0/24
         origin: AS12654
         descr: RPKI attestation
         mnt-by: NA
         created: 2021-05-03T20:53:20Z
         last-modified: 2021-05-03T20:53:20Z
         source: ROA-RIPE-RPKI-ROOT
      
summary
      This format produces a summary of the content of the RPKI repository. For
      each trust anchor, it will print the number of verified ROAs and VRPs.
      Note that this format does not take filters into account. It will always
      provide numbers for the complete repository.

For example, to get the validated ROA payloads in CSV format, run:

.. code-block:: text

   routinator vrps --format csv
   ASN,IP Prefix,Max Length,Trust Anchor
   AS55803,103.14.64.0/23,23,apnic
   AS267868,45.176.192.0/24,24,lacnic
   AS41152,82.115.18.0/23,23,ripe
   AS28920,185.103.228.0/22,22,ripe
   AS11845,209.203.0.0/18,24,afrinic
   AS63297,23.179.0.0/24,24,arin
   ...

To generate a file with with the validated ROA payloads in JSON format, run:

.. code-block:: text

   routinator vrps --format json --output authorisedroutes.json

ASN and Prefix Selection
""""""""""""""""""""""""

.. deprecated:: 0.9
   ``--filter-asn`` and ``--filter-prefix``

In case you are looking for specific information in the output, Routinator
allows you to add selectors to see if a prefix or ASN is covered or matched by a
VRP. You can do this using the :option:`--select-asn` and
:option:`--select-prefix` options.

When using :option:`--select-asn`, you can use both ``AS64511`` and ``64511`` as
the notation. With :option:`--select-prefix`, the result will include VRPs
regardless of their ASN and MaxLength. Both selector flags can be combined and
used multiple times in a single query and will be treated as a logical *"or"*.

A validation run will be started before returning the result, making sure you
get the latest information. If you would like a result from the current cache,
you can use the :option:`--noupdate` or :option:`-n` option.

Here are some examples selecting an ASN and prefix in CSV and JSON format:

.. code-block:: text

   routinator vrps --format csv --select-asn 196615
   ASN,IP Prefix,Max Length,Trust Anchor
   AS196615,2001:7fb:fd03::/48,48,ripe
   AS196615,93.175.147.0/24,24,ripe

.. code-block:: text

   routinator vrps --format json --select-prefix 93.175.146.0/24
   {
     "roas": [
       { "asn": "AS12654", "prefix": "93.175.146.0/24", "maxLength": 24, "ta": "ripe" }
     ]
   }

.. _doc_routinator_validity_checker:

Validity Checker
----------------

You can check the RPKI origin validation status of one or more BGP announcements
using the :subcmd:`validate` subcommand and by supplying the ASN and prefix. A
validation run will be started before returning the result, making sure you get
the latest information. If you would like a result from the current cache, you
can use the :option:`--noupdate` option.

.. code-block:: text

   routinator validate --asn 12654 --prefix 93.175.147.0/24
   Invalid

When providing the :option:`--json` option, a detailed analysis of the reasoning
behind the validation outcome is printed in JSON format. In case of an Invalid
state, whether this because the announcement is originated by an unauthorised
AS, or if the prefix is more specific than the maximum prefix length allows.
Lastly, a complete list of VRPs that caused the result is included.

.. code-block:: text

   routinator validate --json --asn 12654 --prefix 93.175.147.0/24
   {
     "validated_route": {
      "route": {
        "origin_asn": "AS12654",
        "prefix": "93.175.147.0/24"
      },
      "validity": {
        "state": "Invalid",
        "reason": "as",
        "description": "At least one VRP Covers the Route Prefix, but no VRP ASN matches the route origin ASN",
        "VRPs": {
         "matched": [
         ],
         "unmatched_as": [
           {
            "asn": "AS196615",
            "prefix": "93.175.147.0/24",
            "max_length": "24"
           }

         ],
         "unmatched_length": [
         ]      }
      }
     }
   }

If you run the HTTP service in daemon mode, validation information is also
available via the :ref:`user interface <doc_routinator_ui>` and at the
``/validity`` API endpoint.

Reading Input From a File
"""""""""""""""""""""""""

.. versionadded:: 0.9

Routinator can also read input to validate from a file using the
:option:`--input` option. If the file is given as a single dash, input is
read from standard input. You can also save the results to a file using the
:option:`--output` option.

You can provide a simple plain text file with the routes you would like to have
verified by Routinator. The input file should have one route announcement per
line, provided as a prefix followed by an ASCII-art arrow => surrounded by white
space and followed by the AS number of the originating autonomous system.

For example, let's provide Routinator with this file, saved as ``beacons.txt``:

.. code-block:: text

   93.175.147.0/24 => 12654
   2001:7fb:fd02::/48 => 12654

When referring to the file with the :option:`--input` option Routinator
provides the RPKI validity state in the output:

.. code-block:: text

   routinator validate --input beacons.txt 
   93.175.147.0/24 => AS12654: invalid
   2001:7fb:fd02::/48 => AS12654: valid


With the :option:`--json` option you can provide a file in JSON format. It
should consist of a single object with one member *routes*  which contains an
array of objects. Each object describes one route announcement through its
*prefix* and *asn* members which contain a prefix and originating AS number as
strings, respectively.

For example, let's provide Routinator with this ``beacons.json`` JSON file:

.. code-block:: json

  {
    "routes": [{
        "asn": "AS12654",
        "prefix": "93.175.147.0/24"
      },
      {
        "asn": "AS12654",
        "prefix": "2001:7fb:fd02::/48"
      }
    ]
  }

When referring to the file with the :option:`--json` and :option:`--input`
options, Routinator produces a JSON object that includes the validity state and
a detailed analysis of the reasoning behind the outcome of each route.

.. code-block:: text

  routinator validate --json --input beacons.json
  {
    "validated_routes": [
      {
        "route": {
          "origin_asn": "AS12654",
          "prefix": "93.175.147.0/24"
        },
        "validity": {
          "state": "invalid",
          "reason": "as",
          "description": "At least one VRP Covers the Route Prefix, but no VRP ASN matches the route origin ASN",
          "VRPs": {
            "matched": [
            ],
            "unmatched_as": [
              {
                "asn": "AS196615",
                "prefix": "93.175.147.0/24",
                "max_length": "24"
              }
            ],
            "unmatched_length": [
            ]
          }
        }
      },
      {
        "route": {
          "origin_asn": "AS12654",
          "prefix": "2001:7fb:fd02::/48"
        },
        "validity": {
          "state": "valid",
          "description": "At least one VRP Matches the Route Prefix",
          "VRPs": {
            "matched": [
              {
                "asn": "AS12654",
                "prefix": "2001:7fb:fd02::/48",
                "max_length": "48"
              }
            ],
            "unmatched_as": [
            ],
            "unmatched_length": [
            ]
          }
        }
      }
    ]
  }

