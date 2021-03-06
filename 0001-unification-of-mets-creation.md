# Unification of METS creation

* Status: accepted

Note: this record uses a different structure based on
[Michael Nygard's blog post][nygard].

[nygard]: http://thinkrelevance.com/blog/2011/11/15/documenting-architecture-decisions

## Context

### Technological forces

* METS Reader & Writer (henceforth [metsrw][0] already exists, is being used in
  Archivematica, and seems like a good tool for abstracting METS file creation.
* Currently, [Archivematica][1] (AM) and the [Archivematica Storage Service][2]
  (SS) both create or modify METS files (pointer files are METS files too),
  using mets-rw or various lxml APIs. This inconsistency makes it difficult to
  reliably create predictable METS files.
* Artefactual has created a [METS validator][3] project, which contains a
  Schematron file that validates AM-generated AIP METS files.
* metsrw (v. 0.1.1) cannot create pointer files and it does not support METS
  file validation, either via XMLSchema or Schematron.
* metsrw (v. 0.2.0) does have support for pointer file creation and for METS
  file validation generally, both via XMLSchema and Schematron.
* Changing how we write METS files may inadvertently affect the METS files
  produced; METS validation and related testing will need to prevent this, or
  formally recognize it if it is intentional.
* At present in AM and SS, METS/PREMIS data structures are encoded at various
  points as XML strings, MySQL/SQLite database rows, and Python objects (i.e.,
  metsrw or lxml instances).

### Project forces

* In the context of the project "AIP Encryption via Mirror Locations", mirror
  locations in the Storage Service must replicate stored AIPs and give them
  their own pointer files; if replica pointer files are to be generated in the
  SS then it makes sense to generate all pointer files there.
* An [analysis of pointer file creation in Archivematica][4] has been performed.

### Political forces

* Other projects under discussion or on the horizon will involve the need to
  recognize various "flavours" or versions of AM-generated and
  externally-generated (e.g., DSpace, from-LOCKSS) METS files and process them
  accordingly. One such project relates to re-ingesting old AM-generated AIPs
  whose METS files may be significantly different from their current
  counterparts. This implies more sophisticated METS file validation and
  recognition capabilities than AM currently implements.
* The Mirror Locations project’s budget does not cover wholesale refactoring of
  how AM creates METS files, or even significant alterations to metsrw.

## Decision

We will use metsrw to create and modify METS
files---including pointer files---in all future code. We will not rewrite
existing METS creation code to use metsrw unless a clear opportunity arises to
do so, e.g., direct funding for this purpose or a project that requires
non-trivial changes to such code.

In the AIP Mirror project, metsrw will be used to create METS pointer files,
both when AIPs are stored and when they are replicated. This will involve
removing the Create Pointer File micro-service in AM and calling a
`create_pointer_file` method of Package at the end of AIP storage, reingest and
replication.

## Status

Proposed.

## Consequences

### Consequences to metsrw

* Since it will now be more crucial to AM, metsrw will have to be better
  maintained by Artefactual, e.g., moved to the artefactual repository proper
  and given more thorough unit tests, documentation and user contribution
  guidelines.
* Metsrw will have to fully support pointer file creation.
* Metsrw will need better support for METS file validation and recognition of
  METS varieties.

### Consequences to Archivematica and the Storage Service

* AIP compression is still performed in AM (and stored in AM’s db) but now
  pointer file creation will be performed by metsrw in the SS. This means that
  either AM will somehow have to communicate compression event facts to SS or
  AIP compression should be moved to the SS.
* If AM and the SS are both performing "preservation events"---which should be
  documented in METS/PREMIS, e.g., compression for AM and encryption for
  SS---then it may make sense to enhance SS's domain model to include Events.
  On the other hand, maybe the existence of database records of PREMIS events
  is weakening the role of the AIP METS files as the ultimate sources of truth
  for preservation events in Archivematica.

[0]: https://github.com/artefactual-labs/mets-reader-writer
[1]: https://github.com/artefactual/archivematica
[2]: https://github.com/artefactual/archivematica-storage-service
[3]: https://github.com/artefactual/mets-validator
[4]: https://docs.google.com/document/d/1iyEz47TN0zmhPiOi8QVDfy0HxJPK-Ar8-8S9IR0lRC4/
