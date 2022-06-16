# cms

## Problem

Traditional content management systems do not semantically distinguish their content versioning from their schema versioning.

Semantic versioning is an established practice in software development, that guarantees compatibility between versions of some artifact towards consumers of that artifact.

In software development, this means if you had relied on an artifact on one version A, and you evaluate your potential reliance on a future version B, if:

- it is a patch version update, changes are incorporated without a change,
- it is a minor version update, changes are incorporated without change, but some changes are potentially missed,
- it is a major version update, changes cannot be incorporated without change, even if there are no changes.

Applying this to a content management system we observe the following facts:

- each version of *data* implicitly relies on a (different) version of its *schema*,
- contemporary content management systems deal poorly with making this implicit reliance explicitly available: either data version and schema version are not explicit at all, or they are forcefully made compatible in either:
  - matching data versions with incompatible schema versions, or
  - change the schema version ad-hoc without any version guarantees to match the data, or
  - have a schema that is so general, that it encompasses potentially all data versions.

This leads to the following significant downsides in software development and content development:

- Mismatches between content version and schema version lead to bugs, which are not automatically discoverable.
- It places unnecessary burden on software developers to account for schemas that are far to general for the content they aim to represent.

## Solution

We propose a content management system that make the implicit connection of content version and schema version explicit.

In it, each content version automatically determines its schema version and schema versions are an explicit notion.

This means, that access to content is restricted by a given schema version, and only compatible content up to that schema version can be requested.

This effectively solves all problems outlined above, while not impairing content creator's and content developer's workflows. Any potential impairments are already present; it is only in their implicitly that states problems arise.

## Implementation Proposal

We propose a content management system which explicitly versions its content along with its schema.

To be practical for contemporary software development and content creation workflows, and to facilitate a first proof-of-concept, we propose the following qualities and restrictions:

- The state of the content management system is stored in two distinct Git repositories: `schema` and `content`.
- Both the schema and content files are expressed in the JSON file format.
- Both changes to the schema repository and the content repository are possible, with automatic and guided manual migration capabilities of either schema or content.
- From any content version, it is possible to determine an exact schema representation.
- Any two schema representations can be compared in their version compatibility.

This would enable us to maintain a linear history of semantically versioned, exact schema definitions, along with separate linear histories of compatible content.

The result will be a set of command line tools to effectively manage the union of `schema` and `content` repositories as well as a web server that provides REST access to semantically versioned content (and their schema definition).

## Detailed Design

JSON is a plain-text file format that supports a small set of types relevant in software development and content creation: while it has support for, say, strings, records and arrays, it lacks support for, say, date and time types.

To be useable for our purpose, the JSON file format used has to be extended by relevant types necessary for content creation and software development. Date types for instance would be embedded in JSON as conventions over string types, ie. the string "2022-06-16T09:25:00Z" could be interpreted as a UTC date time type.
Any optional type can be embedded by the convention of the presence of the type, or a field being `null`.

This naturally leads to the ambiguity of data with respect to exact schema definitions: is a string a date time string, or an arbitrary string? Is it required, or is it optional?

We will define a set of relevant data types embeddable in JSON by a canonical representation, as well as rules that determine what representation can be interpreted as what types, and provide a default interpretation in the unconstrained (see below) scenario.
This allows us to automatically generate meaningful schemas, just by looking at the content that is present.

Workflows the schema development and content development differ throughout the life-cycle of a software product. In its inception, it is desirable to start with content creation, and have the schema automatically follow. Soon in development, it makes sense to artifically constrain the schema to guide future content development. In production software, schema and content are likely developed in unison, while not affecting the semantic guarantees of the production state.
Thus, our proposed systems allows for both changes to the content and to the schema and it offers migration paths for each scenario. For example:
Should a content change incur a change in the schema, its semantic version change is automatically determined and the schema updated.
Should a schema change incur changes in the present content, tools aid for the discoverability and migration of such content instances.
Of course, should either change be compatible with the existing content or schema, no action is necessary, and the compatibility is automatically and explicitly maintained.

During software development, the schema version is known, and thus can be passed to the provided REST API. The REST API only serves the most recent content, compatible with that schema version.
Content creation and schema development are unaffected by this restriction, as they can still happen. But they happen in future versions, not placing any burden on software development. By serving content compatible up to a schema version, content changes cannot introduce bugs, and software can explicitly be updated to future versions of content at their own pace.
Additionally, because content creation does not place constraints on software development anymore, schema definitions can be as exact as desirable for software development.

## Future Work

We state the following goals for possile future work:

- API access can be provided by other means than REST, for instance GraphQL or gRPC.
- Content can be created in file formats different than JSON, for instance YAML, images or documents.
- State can be backed by a database instead of the file system.
- A web interface can be built around the content creation process, and/ or the schema creation process.
