name: Update satellite data

on:
  workflow_dispatch:
  schedule:
    # 과도한 요청을 피하기 위해 6시간마다 실행합니다.
    - cron: "17 */6 * * *"

permissions:
  contents: write

concurrency:
  group: update-satellite-data
  cancel-in-progress: false

jobs:
  update-data:
    runs-on: ubuntu-latest

    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Download and validate Starlink OMM JSON
        shell: bash
        run: |
          set -euo pipefail
          mkdir -p data

          TEMP_FILE="$(mktemp)"
          URL="https://celestrak.org/NORAD/elements/gp.php?GROUP=STARLINK&FORMAT=JSON"

          if ! curl \
            --fail \
            --location \
            --retry 3 \
            --retry-delay 10 \
            --connect-timeout 20 \
            --max-time 120 \
            --user-agent "satellite-observation-guard/1.0 educational project" \
            "$URL" \
            --output "$TEMP_FILE"
          then
            if [ -s data/starlink.json ]; then
              echo "Download failed; keeping the previous valid file."
              exit 0
            fi

            echo "Download failed and no previous data file exists."
            exit 1
          fi

          python - "$TEMP_FILE" <<'PY'
          import json
          import sys
          from pathlib import Path

          source = Path(sys.argv[1])
          output = Path("data/starlink.json")

          with source.open("r", encoding="utf-8") as handle:
              data = json.load(handle)

          if not isinstance(data, list) or not data:
              raise SystemExit("Downloaded JSON is empty or is not a list.")

          required = {
              "OBJECT_NAME",
              "EPOCH",
              "MEAN_MOTION",
              "ECCENTRICITY",
              "INCLINATION",
              "RA_OF_ASC_NODE",
              "ARG_OF_PERICENTER",
              "MEAN_ANOMALY",
              "NORAD_CAT_ID",
          }

          missing = required.difference(data[0])
          if missing:
              raise SystemExit(
                  "Downloaded OMM is missing fields: "
                  + ", ".join(sorted(missing))
              )

          with output.open("w", encoding="utf-8") as handle:
              json.dump(
                  data,
                  handle,
                  ensure_ascii=False,
                  separators=(",", ":"),
              )

          print(f"Validated {len(data)} Starlink objects.")
          PY

      - name: Commit updated data
        shell: bash
        run: |
          set -euo pipefail

          git config user.name "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"

          git add data/starlink.json

          if git diff --cached --quiet; then
            echo "Satellite data has not changed."
            exit 0
          fi

          git commit -m "Update Starlink orbit data"
          git push
