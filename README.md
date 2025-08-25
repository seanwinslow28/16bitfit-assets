# 16BitFit Assets Repository

**Repo:** https://github.com/seanwinslow28/16bitfit-assets.git  
**Owner:** @seanwinslow28

This repository is the **source of truth for generated 16BitFit art assets** and run logs.  
It is fed by a **Custom GPT** that renders sprites **one at a time** (512×512 PNG, transparent) and commits them directly here via the GitHub Contents API.

---

## What lives here

Assets/
Sprites/
Combat/
<FighterName>/
combat_<fighter><pose>.png
Bosses/
<BossName>/
boss<boss><pose>.png
Runs/
Logs/
run*.csv


- **Sprites**: 512×512 **single-character** masters (no sprite sheets), transparent background.
- **Logs**: Append-only CSVs written by the Custom GPT after each batch (status, drift %, links).

> Folders may include a `.gitkeep` file so empty directories can be committed.

---

## File conventions (strict)

- **One image = one character = one pose.** No multi-character frames, no grids, no sheets.
- **Size:** 512×512 PNG, **no downscale**.
- **Transparency:** background alpha strictly 0 or 255 (no halos).
- **Safe body:** all opaque pixels must fit in a **centered 448×448 box** (32 px margins).
- **Style:** Street Fighter II / SNES; **thick outlines**, **hard-edged shading**; **no** gradients / glow / motion blur / anti-aliasing.
- **Palette:** only the hex codes specified in the prompt for that row (skin, hair, eyes, clothing, props).  
  - Special rule: **Marcus** always includes **gold boxing gloves `#FFD700`**.
- **Naming:**
  - Combat: `Assets/Sprites/Combat/<Name>/combat_<name>_<pose>.png`
  - Bosses: `Assets/Sprites/Bosses/<Name>/boss_<name>_<pose>.png`
  - Lowercase snake_case for `<name>` and `<pose>`.

---

## How these files are created

The Custom GPT:
1. Loads two Knowledge CSVs (combat + bosses).
2. **Picks rows where `route ∈ {"gpt-image","dalle"}`** (skips `comfyui`).
3. **Renders each sprite individually** (one subject only).
4. Runs QC at 512×512:
   - alpha-only background,
   - ≤ **2.0%** palette drift (vs. prompt hex set),
   - safe-fit inside 448×448,
   - **single-subject check** (regenerate once if sheet/multi-subject).
5. **Commits to this repo** (idempotent—skips if file exists).
6. Appends a CSV log in `Runs/Logs/`.

Two scheduled prompts typically run:
- **Nightly**: up to 25 rows (1:15 AM local).
- **Catch-up**: up to 10 rows (6:30 AM local).

---

## GitHub API details (used by the Custom GPT)

The uploader uses the **Contents API**:

- **Check if file exists**  
  `GET /repos/{owner}/{repo}/contents/{path}`
- **Create/update file (base64 JSON)**  
  `PUT /repos/{owner}/{repo}/contents/{path}`

**Required headers**
Authorization: token <YOUR_PAT>
Accept: application/vnd.github+json
X-GitHub-Api-Version: 2022-11-28
User-Agent: 16bitfit-gpt-uploader


**Create example (curl)**
```bash
curl -X PUT \
  -H "Authorization: token TOKEN" \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  -H "User-Agent: 16bitfit-gpt-uploader" \
  https://api.github.com/repos/seanwinslow28/16bitfit-assets/contents/Assets/.gitkeep \
  -d '{
    "message": "chore(scaffold): add .gitkeep to Assets",
    "content": "cGxhY2Vob2xkZXI=",  // base64("placeholder")
    "branch": "main"
  }'


