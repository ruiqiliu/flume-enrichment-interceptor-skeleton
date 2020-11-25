# Flume Interceptor: Enriched Event

[![Build Status](https://travis-ci.com/ruiqiliu/flume-enrichment-interceptor-skeleton.svg?branch=master)](https://travis-ci.com/ruiqiliu/flume-enrichment-interceptor-skeleton)

This project provides an interceptor to enrich event body with custom data. The implementation consist of:

- EnrichedEventBody: represents enriched event body. The enriched event will have 2 attributes: extraData and message.
    extraData will be fed with the custom data set in the configuration file (see below) while message will preserve
    the original event body string. The enriched event is then serialized to a JSON object.
- EnrichmentInterceptor: implements the Flume interceptor interface. The intercept() method will add the custom data
    to the original event body and serialize it back as a JSON string.
- JSONStringSerializer: the JSON serializer

## How to use

Clone the project:

```sh
$ git clone https://github.com/keedio/flume-enrichment-interceptor-skeleton.git
```

Build with Maven:

```sh
$ mvn clean install
```

The jar file will be installed in your local maven repository and can be found in the target/ subdirectory also. Add it
to your Flume classpath.

Configure your agent to use this interceptor, setting the following options in your configuration file:

```ini
# interceptor
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = org.keedio.flume.interceptor.enrichment.interceptor.EnrichmentInterceptor$EnrichmentBuilder
# Full path to the properties file that contains the extra data to enrich the event with
a1.sources.r1.interceptors.i1.properties.filename = /path/to/filename.properties
# The format of incoming events ( DEFAULT | enriched )
a1.sources.r1.interceptors.i1.event.type = DEFAULT



# A map of regexps, where each regexp may be composed of Named captured groups according syntax (?<name>regex).
# On applying regexps to the message, the first match will enrich de data.
a1.sources.r1.interceptors.i1.custom.regexp.1 = (?<name>regex)
a1.sources.r1.interceptors.i1.custom.regexp.2 = (?<nameA>regex)\\METACHARACTER(?<nameB>regex)\\..
.......................
a1.sources.r1.interceptors.i1.custom.regexp.n = 



# In case that the input is a (optionally multiple) key-value pair string, there's no need to use named groups. It also supports regexps where one group will act as the key (default index: 1), and other group will act as the value (default index: 2)
a1.sources.r1.interceptors.i1.custom.regexp.1 = (\w+)=([^\"\s]+)
a1.sources.r1.interceptors.i1.custom.regexp.2 = (\w+)=[\"]([^\"]+)[\"]
.......................
a1.sources.r1.interceptors.i1.custom.regexp.n = 
# If there are multiple regexps, the capture groups for key and value will be the same for all
# The capture group indexes for key and value can be modified by the next configuration
a1.sources.r1.interceptors.i1.custom.group.regexp.key = 1
a1.sources.r1.interceptors.i1.custom.group.regexp.value = 2



# If the regexps are using some reserved characters which are wrongly handled by some tool administrator (for example, Ambari with character "=" inside a property value), there's the possibility to encode the regexps in order to avoid problems
# In order to be recognized as UrlEncoded, the property "regexp.is.urlencoded" must be true
a1.sources.r1.interceptors.i1.custom.regexp.1 = %28%5Cw%2B%29%3D%28%5B%5E%5C%22%5Cs%5D%2B%29
a1.sources.r1.interceptors.i1.custom.regexp.2 = %28%5Cw%2B%29%3D%5B%5C%22%5D%28%5B%5E%5C%22%5D%2B%29%5B%5C%22%5D
a1.sources.r1.interceptors.i1.regexp.is.urlencoded = true
```

Example of custom properties:
```ini
hostname = localhost
domain = localdomain
```

The enriched event body will contain:
```json
{
 "extraData":{"hostname": "localhost", "domain": "localdomain"},
 "message": "the original body string"
}
```

Besides the properties of the file, the fields extracted by regexp are added.
The fields of the first regexp having correspondence with the log entry are
appended as new pairs in extradata and if any correspondence regexp has no match,
nothing is added:

Example of regexp:
```ini
a1.sources.r1.interceptors.i1.custom.regexp.1 = (?<date>\\d{4}-\\d{2}-\\d{2}+)\\s(?<time>\\d{2}:\\d{2}:\\d{2}+)\\s
```
Example of log to match:
```ini
"2015-04-23 07:16:08 (whatever)"
```

The enriched event body will contain two new pairs in the extradata:
```json
{
 "extraData":{"hostname": "localhost", "domain": "localdomain", "date": "2015-04-23", "time": "07:16:08"},
 "message": "the original body string"
}
```



Thanks to tony19: https://github.com/tony19/named-regexp :

Althoug Java 7 allows named captured groups, flume-enrich-interceptor is using named-regexp 0.2.3 tony19's library because it adds
interesting features, example  (?\<foo_foo\>regex) or (?\<foo foo\>regex) are not allowed in Java 7.

## Needed dependencies

You need to deploy the following jars in order to use this interceptor:

* http://search.maven.org/remotecontent?filepath=com/github/tony19/named-regexp/0.2.3/named-regexp-0.2.3.jar
* http://search.maven.org/remotecontent?filepath=com/googlecode/juniversalchardet/juniversalchardet/1.0.3/juniversalchardet-1.0.3.jar
* http://search.maven.org/remotecontent?filepath=org/codehaus/jackson/jackson-mapper-asl/1.9.13/jackson-mapper-asl-1.9.13.jar
* http://search.maven.org/remotecontent?filepath=org/codehaus/jackson/jackson-core-asl/1.9.13/jackson-core-asl-1.9.13.jar

## Changes from version 0.0.1

Update package's name convention.
Added regexp in flume's context.

EnrichedEventBody.message is now a String (was byte[]).
