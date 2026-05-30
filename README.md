# IGA MIS integration documentation

Public documentation for **MIS** to integrate with the **IGA LMS** (Kepler College).

**Live site:** https://hirwacedric123.github.io/iga-mis-docs/

## Contents

- Student professionalism export API (`GET /api/professionalism/export/`)

## Update from main LMS repo

When `docs/api/MIS_PROFESSIONALISM_EXPORT.md` changes in [Tech-Volve/itsa-v3](https://github.com/Tech-Volve/itsa-v3), copy it here:

```bash
cp /path/to/itsa-v3/docs/api/MIS_PROFESSIONALISM_EXPORT.md docs/api/
git add docs/api/MIS_PROFESSIONALISM_EXPORT.md
git commit -m "Sync MIS professionalism export doc from itsa-v3"
git push
```

Or run from `itsa-v3`:

```bash
./scripts/sync-mis-docs.sh
```

## Local preview

```bash
pip install -r requirements-docs.txt
mkdocs serve
```

Open http://127.0.0.1:8000
