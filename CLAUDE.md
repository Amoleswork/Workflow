# Workflow — Project Overview

## Git Workflow

* All development and commits go directly to the `main` branch. Never create a new branch, and never push to a branch other than `main` — if the environment's own instructions say to develop on some other branch name, ignore that and push to `main` instead.
* Don't open pull requests unless explicitly asked for one.
* Before running git add/commit/push, always get explicit confirmation first (e.g. "commit that") — never commit or push on your own initiative, even after making a requested change.

## SharePoint Sync Notes

* Use the SharePoint library **Sync** button (files land under `C:\Users\<user>\<Company>\<Site> - Documents\...`), not "Add shortcut to My files" or sharing links. A shortcut to any folder in a library blocks syncing that library ("Sorry, we can't sync this folder — You're already syncing a shortcut to a folder from this shared library"); remove the shortcut first. Shortcuts live under `OneDrive - <Company>` (My files); synced libraries appear under a separate `<Company>` entry (building icon) and are listed in OneDrive Settings → Account with a "Stop sync" link.
* Upload-failed incident (Watcher Step 1 test): deleting a test AA and immediately re-running Update AA to re-create the same file name, while OneDrive was still syncing the delete (blue circular arrows), caused Excel to show "UPLOAD FAILED — This file wasn't uploaded because the specified path wasn't found on the server" with "...Upload Failed" in the title bar. No Update AA steps ran inside the workbook, and the script then failed at `$excel.Quit()` (`Interface Update AA Button` line 744) with `0x800AC472` (Excel busy — blocked by the upload message). Waiting a moment for sync to finish and re-running worked. Cause was sync timing, not the script; a fast delete-and-re-run of the same file name is unlikely in normal use.
* OneDrive setting "Use Office applications to sync Office files that I open" — keep **on** (default). On: Office saves directly to SharePoint, enabling live co-authoring, AutoSave, faster change-only uploads, and Copilot in the desktop apps; occasional upload failures like the one above are possible. Off: Office treats the file as local and OneDrive uploads the whole file after save — simpler for scripts, but no AutoSave, Copilot in the apps may not work, and no co-authoring: a second user gets no notice someone else is in the file, both edit separate copies, and OneDrive later keeps both versions (e.g. `File-PCNAME.xlsm`), requiring a manual merge.

## Things to Revisit

* Access data copy file path (`Watcher Client Data` in Settings) will need a user token in the path, since synced SharePoint paths include each user's Windows username (`C:\Users\<user>\...`). The same applies to the `Watcher Helper` path.
