# 🖼️ Extract Image Links from Markdown (Bash)

## 📘 Overview
This portfolio entry documents a LabEx challenge where the goal was to build a Bash script that **extracts image URLs from a Markdown file**. This type of task is common in documentation QA, content migration, and automation pipelines where you need to parse Markdown and collect assets.

The script (`getimage.sh`) accepts a Markdown file path as an argument and prints **one image URL per line**.

---

## 🎯 Objectives
- Read a Markdown file passed as a parameter (example: `./getimage.sh labex_lab1.md`)
- Extract image URLs from Markdown image syntax:

```md
![alt text](https://example.com/image.png)
```
Output only the URL portion, one per line



## 🧠 Skills Practiced
Pattern matching with grep


Text extraction and cleanup with sed


Bash scripting + argument handling


Regular expressions (regex) for Markdown parsing



## 🛠️ Script Used (getimage.sh)
File: getimage.sh
```bash
#!/bin/bash

# Extract image URL
image_urls=$(grep -o "\!\[.*]\(.*\)" "$1" | sed -E "s/(\!\[.*]\()(.+)(.*\))/\2/g")

# Print image URL
echo "$image_urls"
```

## ▶️ How to Run
```bash
chmod +x getimage.sh
./getimage.sh labex_lab1.md
```
## ✅ Expected Output (Example)
```bash
https://doc.shiyanlou.com/document-uid13labid292timestamp14677222211211.png
https://doc.shiyanlou.com/document-uid13labid292timestamp14672311234511.png
https://doc.shiyanlou.com/document-uid13labid292timestamp14677029556772.png
```

## 🔍 Breakdown of the Approach
Step 1 — Find Markdown image patterns
```bash
grep -o "\!\[.*]\(.*\)" "$1"
```
-o prints only the matching part


Matches lines containing the pattern ![...](...)


Step 2 — Strip everything except the URL inside ( )
```bash
sed -E "s/(\!\[.*]\()(.+)(.*\))/\2/g"
```
Captures:


(\!\[.*]\() → everything up to the opening parenthesis


(.+) → the URL (what we want)


(.*\)) → the rest, including closing )


Replaces the full match with only capture group \2



## 🔐 Security / Practical Use Cases
Inventorying external image dependencies before publishing docs


Detecting broken or suspicious external links in documentation


Migrating Markdown content to a new platform/CDN


Building automated documentation pipelines



## 🧾 Key Takeaway
Using lightweight CLI tools (grep + sed) is a fast, portable way to parse structured text like Markdown and extract useful data without needing heavy scripting languages.

