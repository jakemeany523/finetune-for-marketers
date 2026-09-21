# Data rights, before anything is downloaded

The guide's step 2 is the step everyone skips, and it is the most dangerous one. Public
availability is NOT training permission.

For every source, record this BEFORE downloading a single file:

    source_id, source_url, owner, rights_status (approved|denied|unreviewed),
    permission_scope, commercial_use_permitted, redistribution_permitted,
    rights_reviewed_at, private_evidence_ref

Public to the repo: metadata, file hashes, processing code, schemas, eval code.
Never public: raw sources, OCR output, permission correspondence, anything user-submitted.

Task B note: startup pitch decks are exactly the case the guide warns about. If rights_status is
not "approved" for a deck, it does not enter the dataset. No exceptions, no "it was public".
