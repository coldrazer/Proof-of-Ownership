# Disclosures

Linked from [the Skill](../SKILL.md), as required by the UGREEN Hackathon 2026 rules.

## Third-party components and open source libraries

None. The package contains only Markdown files. It bundles no code, libraries, models, fonts, images or data from any third party.

During development the author used the UGREEN UGOS Skill Creation Tool (uliya-skill-kit version 0.7.3, provided by UGREEN through the official hackathon starter kit) to read the public capability catalog and to run its static checker. The tool is not included in this package.

## Principal functionality developed for this hackathon

All of the following was newly developed during the hackathon:

- The proof-of-ownership workflow: turning everyday photos and receipts already on the NAS into a room-by-room, claim-ready inventory.
- The last-seen date per item and the days-from-last-seen-to-loss column for loss mode.
- The proof strength rubric (Strong, Moderate, Weak) and the "gaps to fix now" list.
- The SHA-256 evidence fingerprint appendix.
- Optional theft-window camera review that never identifies people.
- The category keyword set, label and receipt OCR terms, and the anti-guessing vision prompt.
- The test plan and synthetic test data set.

## AI tools used and their principal uses

- **Claude (Anthropic)**: read the capability catalog and the Skill development standard, drafted the SKILL.md workflow and these reference files from the author's idea and direction, and ran the kit's static checker.
- **UGOS Skill Creation Tool checker**: static validation of the package format and orchestration rules.

## External services and data flows

- The Skill makes no direct calls to any external API or website.
- All work is delegated to the registered NAS experts (file-agent, photo-agent, document-agent, knowledge-agent, security-monitor-agent). Vision and document understanding use the models the Uliya platform provides, under the platform's own permissions and data rules. The Skill does not choose an external model.
- The Skill reads only folders the user names and verifies. The only writes are the report file (Step 8) and the optional album (Step 9), both after explicit user confirmation and platform approval.
- No credentials, API keys, account numbers or personal data are stored in this package. Test data is synthetic.

## License

This Skill is the original work of its author and is submitted under the UGREEN Hackathon 2026 official rules.
