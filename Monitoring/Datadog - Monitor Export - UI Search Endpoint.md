
Initially we used the documented [[Datadog API]] endpoint:

```text
/api/v1/monitor/search
```

with the same search query shown in the UI:

```text
tag:"customer:rogers" OR tag:"project:*rogers*"
```

However, the results did not match. The UI showed **138 monitors**, while the API returned significantly fewer.

To investigate, we opened the browser Developer Tools (**Network** tab), re-ran the search in the Datadog UI, and inspected the actual request being sent by the page.

==The UI was not calling the public API endpoint at all.== Instead, it was using:

```text
/monitor/search
```

with parameters such as:

```text
start=0
count=50
text=tag:"customer:rogers" OR tag:"project:*rogers*"
```

Using that same endpoint reproduced the exact UI results, including all **138 monitors**. The script below simply automates those same UI requests, paginates through all pages, and combines the results into a single JSON export.

In short: we **"sniffed" the browser traffic, found the real endpoint used by the UI, and automated it.** 😄

---

## Export Script

```bash
#!/usr/bin/env bash
set -euo pipefail

DD_API_KEY=$(security find-generic-password -a "$USER" -s datadog-api-key -w)
DD_APP_KEY=$(security find-generic-password -a "$USER" -s datadog-app-key -w)

OUT="rogers_monitors.json"
COUNT=50
START=0
TOTAL=-1

# Change this query when needed.
QUERY='tag:"customer:rogers" OR tag:"project:*rogers*"'

TEXT=$(python3 -c 'import urllib.parse,sys; print(urllib.parse.quote(sys.argv[1]))' "$QUERY")

TMPDIR=$(mktemp -d)
trap 'rm -rf "$TMPDIR"' EXIT

echo "Query: $QUERY"
echo "Output: $OUT"
echo

while true; do
  echo "Fetching start=${START}, count=${COUNT}..." >&2

  RESPONSE_FILE="$TMPDIR/page_${START}.json"

  curl -s "https://app.datadoghq.com/monitor/search?start=${START}&count=${COUNT}&sort=name%2Casc&text=${TEXT}&counts_size=500&counts=%7B%7D&with_groups=false&include_synthetics=true&case_insensitive_tags=true&internal_tags_to_include=managed_by%3Adatadog%2Cbitsai%3Aautomatic" \
    -H "DD-API-KEY: ${DD_API_KEY}" \
    -H "DD-APPLICATION-KEY: ${DD_APP_KEY}" \
    -H "Accept: application/json" \
    > "$RESPONSE_FILE"

  if [ "$TOTAL" -eq -1 ]; then
    TOTAL=$(jq -r '.total // 0' "$RESPONSE_FILE")
    echo "Total reported by Datadog UI endpoint: $TOTAL" >&2
  fi

  START=$((START + COUNT))

  if [ "$START" -ge "$TOTAL" ]; then
    break
  fi
done

jq -s '[.[].monitors[]] | unique_by(.id)' "$TMPDIR"/page_*.json > "$OUT"

echo
echo "Done."
echo "Exported monitors: $(jq 'length' "$OUT")"
echo "Saved to: $OUT"
```

### Usage

```bash
chmod +x export-dd-monitors.sh
./export-dd-monitors.sh
```

### Verify Export

```bash
jq length rogers_monitors.json
```

Expected result for the Rogers query:

```text
138
```
