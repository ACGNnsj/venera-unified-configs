# venera-unified-configs

Self-hosted catalog + book sources for venera-unified (B4).

Layout plan:
- `catalog/index.json` - versioned source catalog (see SourceCatalogEntry)
- `sources/*.js` - pinned book-source scripts

Rules: catalog URLs pin to tags/commits, never floating branches.