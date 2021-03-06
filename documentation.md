---
# Page settings
layout: default
keywords:
comments: true

# Micro navigation
micro_nav: true

# Hero section

title: API Documentation (v0.5)
description: This API aims to provide an independent, collective, shared resource by which researchers working in the field can collaborate to push the state of the art in realtime factcheck detection.

# Page navigation
page_nav:
    prev:
        content: Measuring Truth
        url: '/measuring_truth'
    next:
        content: Members
        url: '/members'
---


# Fact Benchmark API (v0.5)

This API aims to provide an independent, collective, shared resource by which researchers working in the field can collaborate to push the state of the art in realtime factcheck detection.

## Inviting feedback

This API is currently a work in progress. Given our desire to support and facilitate the development of research in this space we are currently inviting and would appreciate feedback from our member organisations on points major or minor.

We invite your feedback on this api at feedback@factbenchmark.org

## Resources 

### Claim 

A claim is a short statement which should be falsifiable and of general interest (important).

**A claim should, ideally, be 'falsifiable' in that it:**
* is phrased in an non-ambiguous way
* is not a prediction about the future
* relates to (potentially) verifiable information 
* is not phrased as a question or inquiry
* is not a normative statement ("our policy is small business focused")

**Claims should, ideally, be 'of general interest' due to one or more of the following reasons:**
* it relates to a generally active public discussion or rumor .. or ..
* it is attributed to an important person of interest .. or ..
* it is otherwise important enough to be of importance to a number of people

The ```claim-text``` is the definitive text that should be evaluated for truth or falsity. It doesn't *necessarily* exactly match the text from the source (if any), but should be close to it. 

The ```timestamp``` is the time of attribution to the original claim (if any) 

The ```attribution``` is a URI, being the username of the person who made the claim. In the case of well known individuals it is acceptable to use a wikipedia url. *Is there a better way to do this?*

Example (with some expansion)
```json
{
	"claim-text": "Student arrested for shouting slogans against BJP in Tamil Nadu",
	"timestamp": "2018-09-04T04:29:00.000Z",
	"attribution": "https://twitter.com/LKC1965",
	"evidence": [{
	  "source-url": "https://twitter.com/anindita-guha/status/1036924232595382273",
	  "snapshot": { "hash": "3d8c26e642e3b" }
	}],
	"submitted-by": { "href": "agent/faktist16"},
	"created": "2019-01-01T01:23:00.000Z",
	"self": { "href": "claim/5355b6c6-b620-411b-9eb5-7e73f7146cbf"}
}
```

### Claim-Response 

In this version of the benchmark check-worthy claims are submitted by members and then, also, evaluated by members.

The first step is to decide whether a claim is check-worthy, then to evaluate its truth or falsehood. 

<img src="../theme/assets/images/claim_lifecycle.png" align="right" width="60%">

It is expected, but not required, that a claim will tend to be evaluated for check-worthiness prior to being evaluated for truthiness, but this is not required. (In some cases the check-worthiness will be self evident from source.)

There are three parts of a claim response, any of which can be provided at any time.

- ```check-worthy``` is a section indicating whether or not the claim is check-worthy.
- ```truth-rating``` is a section indicating whether the claim is true, false, or partially true.
- ```annotation``` is a section for arbitrary data.

A given user can respond multiple times to the same claim (and often will).

Submitting a claim (or providing a truth-rating without a check-worthy section) is taken to be implcit acknowledgement that the claim is check-worthy (equivalent to ```"check-worthy": {"importance":1}```).

#### Check-worthy

The simplest way to respond to a claim is to simply post ```"check-worthy": {"importance":1}``` - which is essentially the same as saying ```"check-worthy": true```.

For example:

```json
{ 
	"ref": { "href": "claim/7dc0051f" },
	"check-worthy": { "importance": 1},
	"submitted-by": { "href": "agent/joef2016"},
	"created": "2019-01-01T01:23:00.000Z"
} 
```

The ```importance``` indicates the degree to which the claim is, in fact, "check-worthy". If it summarizes a wider discussion, it may be the degree to which it nicely summarizes that discussion, or if it's phrasing could be improved, it might be the degree to which it's phrasing clearly supports falsifiability. 

Benchmarks will include claims with the highest, 'weighted, consensus importance' in a given time step. So it may also be taken to be an indication of the importance for the benchmark.

Where a ```better-option``` is supplied within a decline-to-rate section, it indicates a related, more check-worthy claim.

Where ```support``` is supplied it indicates content that supports the view put forward.

For example:
```json
{ 
	"ref": { "href": "claim/7dc0051f" },	
	"check-worthy": {
		"importance": 0.3,
		"decline-to-rate": {
			"reason": "not falsifiable - better options exist",
			"support": { "hash": "3d8c26e642e3b" },
			"ref": { "href": "claim/5d81f18d" }
		}
	},
	"submitted-by": { "href": "agent/joef2016" },
	"created": "2019-01-01T01:23:00.000Z"
}
```

A response may simply indicate that the agent chooses not to rate this claim. In such cases the ```importance``` is implicitly assumed to be 0.

```json
{ 
	"ref": { "href": "claim/7dc0051f" },
	"check-worthy": {
		"decline-to-rate": {
			"reason": "not falsifiable - statement of opinion"
		}
	},
	"submitted-by": { "href": "agent/joef2016"},
	"created": "2019-01-01T01:23:00.000Z"
} 
```

#### Truth-rating 

If an agent views a claim as check-worthy they may then assign a 'truth-rating' to it.  Apart from declining to rate, or annotating the claim there are two ways that a truth-rating can be indicated:
* A boolean indicating whether the claim is 'true' or 'false'.
* A boolean true, with a weighting between 0 and 1 indicating 'how true'

The ```weighting``` if any, should be interpreted as the degree to which the statement itself is true, not the confidence that the member has in their rating. 

For example:
```json
{ 
	"ref": { "href": "claim/7dc0051f" },	
	"truth-rating": {
		"call": false,
	},
	"submitted-by": { "href": "agent/joef2016"},
	"created": "2019-01-01T01:23:00.000Z"
} 
```

Or:
```json
{ 
	"ref": { "href": "claim/7dc0051f" },	
	"truth-rating": {
		"call": true,
		"weighting": 0.3,
		"support": { "hash": "3d8c26e642e3b" }
	},
	"submitted-by": { "href": "agent/joef2016"},
	"created": "2019-01-01T01:23:00.000Z"
} 
```

#### Annotation

Another form of response to a claim is an annotation.

Amongst other things, this provides a method for responding to a claim, and having this information independently recorded and timestamped, which can allow the agents to store their own data, and make their own intepretations. (In such cases the "Fact Benchmark" service acts a little like a simple block chain - only without the block or the chain).

Arbitrary json can be posted in an annotation, for instance:

```json
{ 
	"ref": { "href": "claim/7dc0051f" },	
	"annotation": {"seq":1,"id":"fresh","changes":[{"rev":"1-967a00dff5e02add41819138abb3284d"}]},
	"submitted-by": { "href": "agent/joef2016"},
	"created": "2019-01-01T01:23:00.000Z"
} 
```

This json will be stored (and, by default returned) as a content-hash, like this:

```json
{ 
	"ref": { "href": "claim/7dc0051f" },	
	"annotation": { "hash": "a2f12df5" },
	"submitted-by": { "href": "agent/joef2016"},
	"created": "2019-01-01T01:23:00.000Z"
} 
```

One use for the annotations field may be to indicate groups of related claims, but the schama and mechanism for this remains to be determined.

#### 24Hr Time Delay

Claim-responses, like all data, eventually become available to all members of the benchmark. However, to ensure independence of submissions, detailed claim-responses are not available to other agents, until after a 24hr time delay.

### Benchmark

A benchmark consists of a set of check-worthy claims chosen by some objective criteria. 

One format for benchmarks is to choose one  check-worthy claim each hour and at each stage choose the claim with the highest, weighted importance not included in the benchmark already.

It is not necessary for a claim to be included in the benchmark at the time that ratings are provided, and in many cases it will be added to the benchmark retroactively.

```json
{ 
	"name": "December english twitter benchmark",
	"description": "",
	"items": [
		{
			"timestamp": "2018-12-11T02:00:00.000Z",
			"ref": { "href": "claim/1ab3f" }
		},
		{
			"timestamp": "2018-12-12T03:00:00.000Z",
			"ref": { "href": "claim/c283" }
		},
		{
			"timestamp": "2018-12-13T04:00:00.000Z",
			"ref": { "href": "claim/4b40" }
		}
	],
	"submitted-by": { "href": "agent/factbenchmark"},
	"created": "2018-12-10T01:23:00.000Z",
}

```

### Agent (Member)

For transparency, (almost) every piece of data in the API is attached to an agent record, including, wherever possible, data created by the system.

```json
{
	"username": "joef2016",
	"url": "https://xyz.com",
	"affiliation": "XYZ Inc",
	"description": "XYZ Agent 2016",
	"profile_image_url":
"http://abs.twimg.com/sticky/default_profile_images/default_profile_normal.png",
	"submitted-by": { "href": "agent/joef2016"},
	"created": "2019-01-01T01:23:00.000Z",
	"self": { "href": "agent/joef2016" },	
} 
```

### Content (Content Cache)

In order to allow and sharing of associated data against claims an end point exists for posting arbitrary data. This data is retrieved by a content-hash of the concetenation of mime-type and data field (not including the submitted-by etc data). If two agents post the same data the content will only be stored once, using the first submission. 

```json
{
	"hash": "a2f12df5",
	"mime-type": "text/html",
	"data": {"..."},
	"submitted-by": { "href": "agent/joef2016"},
	"created": "2019-01-01T01:23:00.000Z"
}
```

It is also possible to request that content be retrieved by the server, timestamped, cached and signed with the server key,

```json
{
	"hash": "a2f12df5",
	"mime-type": "text/html",
	"data": {"..."},
	"source_url": "https://twitter.com/anindita-guha/status/1036924232595382273",
	"timestamp-captured": "2019-01-01T01:02:00.000Z",
	"signature": "d76a0ee281f84e08b04f73670122f4c9",
	"submitted-by": { "href": "agent/joef2016"},
	"created": "2019-01-01T01:23:00.000Z"
}
```

The mime-type hash-html should be treated in a special way. When retrieving content of this type any urls or hrefs of the form "hash:xxxx" can be de-referenced by recursive requests to this end-point to retrieve the associated content for that hash, along with the corresponding mime-type. 

## Cross cutting concerns 

### Idempotent, CRDT data model

Wherever possible the design of this API is designed to allow for idempotent (or at least CRDT, Conflict-Free Replicated Data) data update models. That means that records, once submitted, cannot be updated, only appended to.

### Submitted-by 

For almost all resources there is a ```submitted-by``` entry automatically created on server with the user id of the authenticated user.

```json
"submitted-by": { "href": "agent/joef2016"},
```

### Created 

For almost all resources there is a ```created``` entry recorded as ISO-8601 formatted UTC indicating the time on the server at the moment the entry was submitted.

```json
"created": "2019-01-01T01:23:00.000Z"
```

### Self ref 

The server will allocate ids to objects on submission. In read responses it may include a self reference as
follows (inspired by, but not quite HATEOS).

```json
"self": { "href": { "item-type/6b051a34-e4b8-4c4f-83a2-de2c3c2f144e" } }
```

### Hidden log

As part of our guiding principles we aim to:
- have attributions for all data
- allow idempotent data schemas
- not delete any data

To do this, while still allow graceful handling of errors we have a hidden log, with values like the following. 

All end points, by default, hide data with a hidden log entry.

For now, hidden log entries will only be accepted by the user with the corresponding submitted-by reference or by a special factbenchmark admin user.

```json
{
	"ref": { "href": "claim/1ab3f" },
	"reason": "accidental submission",
	"submitted-by": { "href": "agent/joef2016"},
	"created": "2019-01-01T01:23:00.000Z"
}
```

Special handling is used for hidden responses, when dealing with computation of agent reputations.

## Future directions 

After the first round of feedback we have attempted to simplify our API to the bare essentials. There are, however, some features we would like to build back in when we can. 

### Mixed media and longer form 'claims' 

In the above, we only allow:
* Claims that can be represented in short text form. 

In future iterations we'd like to extend this model to include:
* Mixed media posts, such as urban rumors
* Article length content
* Quotes from well known persons, in speeches or other venues (these can be handled, but could do with special handling)

### Collections of related claims that comprise a 'rumor'. 

We would also like to address the relationship between specific wordings of a claim and the existence (in some cases) of a storm of related tweets and representations of the 'same' rumor or idea. 

### GraphQL

We may build a GraphQL front end on top of the read-only aspects of this API (especially if there is interest in this). This will naturally fall out of the basic design of the REST API. 

--

If you have any further thoughts about the design of this API we invite our member organisations to please contact us at feedback@factbenchmark.org 
