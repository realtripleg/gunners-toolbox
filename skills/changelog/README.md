# changelog

Turns git history into release notes. Run `/changelog` and Claude reads the commits since the last tag (or a range you give it), works out what changed for the people using the software, and adds an entry to the top of `CHANGELOG.md` under Breaking, Added, Changed, Fixed, Removed, and Security.

It reads the diff when a commit message is useless, flags breaking changes it only finds in the code, and suggests the next version number.

`/changelog client` writes plain-language notes for a non-technical client in chat instead.

It never tags, commits, or pushes.
