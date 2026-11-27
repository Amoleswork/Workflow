# Excel range to PowerPoint & PDF

> Reference copy as of 2026-11-27. The live version is the project doc in the Claude project, which may be updated without this copy being updated.

**Goal:** put an Excel range (usually a sheet's print area) on a PowerPoint slide, or on a PDF page, as editable, native PowerPoint objects. Numbers stay editable text, stay sharp when zoomed and in PDF exports, and use the workbook's real fonts. The workbook is only read, never edited.

## Fonts

* The workspace used for this has no Microsoft fonts, and it resets between sessions. The real font files are in the connected folder at Automation\Misc\Fonts: `trebuc.ttf`, `trebucbd.ttf`, `arial.ttf`, `arialbd.ttf`.
* At the start of any task that creates a PDF or PowerPoint, load those files into the workspace (copy them into `~/.fonts` and refresh the font cache). Then both the LibreOffice reference print and any PDF made here use the real fonts. Without them, Trebuchet MS falls back to a heavier-looking stand-in.
* If a workbook uses a font that isn't in that folder, ask the user to add it there: copy it from C:\Windows\Fonts into Automation\Misc\Fonts.
* Whenever a PDF or PowerPoint made here used a substitute for any font, tell the user which font was replaced and with what. They can then examine it and decide whether the look is acceptable, or replace it manually. Check the fonts embedded in a PDF before reporting it as done.

## Process

1. Read the workbook. Get the range (usually the print area). Note each visible column's width, each row's height, merged ranges, and each cell's value, font, size, bold, alignment (horizontal and vertical), wrap setting, indent level and number format. Skip hidden rows and columns. If the request uses the sheet's print area, also note the tab's page setup: margins, horizontal/vertical centering, orientation and scaling.
2. Build the geometry from Excel's own measurements, not from a printout.
   * Column width in pixels = 7 × the stored width, rounded. The 7 is the widest digit of the workbook's default font, Arial 10; recheck it if a workbook uses a different default font. Points = pixels × 0.75.
   * Row heights are already stored in points.
   * This keeps Excel's proportions and text size. A LibreOffice print is about 6% wider than Excel, which makes the text look smaller and heavier.
3. Use a LibreOffice print to PDF only as an inventory of which text, shading and borders exist, with the real fonts loaded (see Fonts). Map each item into Excel's geometry column by column and row by row.
4. Rebuild as native objects.
   * Shading becomes rectangles with no outline, and borders become lines with the same color and thickness.
   * Each piece of text becomes its own text box with no inner margins and no wrapping. Use the real font name from the cell (e.g. Arial, Trebuchet MS) at the cell's font size and bold setting.
   * Group everything into one object (like select all + Ctrl+G) so it moves and resizes as one piece.
5. Place text the way Excel does. One Excel indent level = 2 × the widest digit of the default font, about 10.5 pt for Arial 10, plus about 0.75 pt of cell padding.
   * Right-aligned numbers: right edge = column's right edge − padding − (indent levels × 10.5 pt) − the width of ")" if the number format ends in `_)` (about ⅓ of the font size).
   * Left-aligned text: starts at the column's left edge + padding + (indent levels × 10.5 pt).
   * Centered text: centered in its cell, or across merged cells.
   * Accounting formats (those containing `*`): keep the printed position; they already line up.
   * Wrapped text in vertically centered cells: center the block of lines in the cell (or merged range). Excel spaces wrapped Trebuchet MS lines at 1.5 × the font size, which is wider than LibreOffice's spacing.
6. Size and place it.
   * On a presentation slide: use judgment. A large range fills most of the area below the slide title and above the footer. A small range stays proportionally small. Keep Excel's width-to-height ratio. The user will check and adjust it.
   * As a standalone PDF: build it as a one-page PowerPoint on an 8.5x11 page, then convert that page to PDF here with the real fonts loaded.
     * If the request uses the sheet's print area: before building, confirm with the user that the page should match the worksheet tab's own page setup, i.e. its margins, centering, orientation and scaling. If yes, lay out the page from those settings so it matches what printing the tab from Excel would give.
       * Printable area = page size minus the tab's left/right/top/bottom margins.
       * For fit-to-page (1 page wide by 1 tall), zoom = the smaller of printable width ÷ range width and printable height ÷ range height, rounded down to a whole percent, as Excel does.
       * Center horizontally and/or vertically within the printable area only if the tab has those options on; otherwise start at the left/top margin.
       * Tested on an AA tab (landscape, margins 0.25" sides and bottom, 1.75" top, centered horizontally, fit to page): 64% zoom, the table spanning the page width starting 1.75" from the top.
     * Otherwise (any other range): don't apply the tab's margins. Use landscape for wide ranges and center the range on the page.
7. Check against the workbook.
   * Every value on the slide matches the cell's value and formatting (commas, percentages, parentheses).
   * Bold and indents match the cell settings.
   * Nothing is missing or cut off at the edges.
   * Zoom in to confirm borders are sharp and the text is real text, not an image.
   * For PDFs, confirm the embedded fonts are the real ones (see Fonts).
8. Save and report.
   * Before every save, ask the user to close the file if it's open on their computer, even if the save looks like it would go through. Saving over a file that's open makes PowerPoint or the folder's sync (SharePoint/OneDrive) keep both versions when the user later closes or saves it. The extra copy is named "-1" (e.g. `Name-1.pptx`), sometimes several minutes later.
   * Don't attach copies of the file in the chat. The user works from File Explorer, and a copy opened from the chat can be edited by mistake or confuse which file is the real one. A chat attachment with the same name as an open file has also turned up in the folder as a "-1" file. Save directly into the folder, then tell the user the edits are done, naming the folder and file.
   * Verify the save. A save made right after the file is copied into the session's output folder can pick up the previous version, and it did once. Wait a few seconds after copying before saving. Then read the saved file back from the user's folder and confirm it matches the new file (same size and contents) before reporting it done. If it doesn't match, save again.
   * Link the saved file, not the folder. Include one `computer://` link to the saved file's path on the user's computer. In the desktop app it shows as a file tile with a preview and a "Show in Folder" button, which the user prefers. Don't link the folder separately; a folder link opens to the parent folder rather than inside it, and the file's "Show in Folder" already covers that.
   * After any save made while a file may have been open, check the folder for a "-1" file. If there is one:
     * Compare it with the main file to see what it holds.
     * Tell the user what happened: which save collided with the open file, and that the duplicate came from the sync or an attachment, not a new version.
     * Confirm whether anything in the "-1" is missing from the main file. If something is, move it into the main file first.
     * Ask the user to delete the "-1" file. This session can't delete files on their computer.

## What worked and what didn't

* Flat picture (PNG): blurred when zoomed and in PDFs.
* SVG/EMF picture: sharp, but the numbers couldn't be edited, and the stand-in fonts showed (Trebuchet MS headers looked noticeably different).
* Native rebuilt objects: sharp, editable, and in the real fonts. This is the method to use.
* Using a printout's layout: made text about 6% too small relative to the grid, so headers looked bolder, and it lost Excel's indents. Switching to Excel's own column widths, row heights, font sizes and indent settings fixed all three. The result is within about 1% of Excel's text size, and most text lands within about 2 points of Excel's position.
* PDFs made here without the real fonts: Trebuchet MS headers looked bolder because a heavier stand-in font was used. Loading the user's own font files fixed it, matching the Adobe PDF printouts.
* Converting Excel straight to PDF here (no PowerPoint step): fine for a rough copy, but it keeps LibreOffice's wider columns, drops indents and spaces wrapped headers too tightly, even with the real fonts. Its one advantage, the tab's own margins, is kept by laying out the PowerPoint page from the tab's page setup.
* Tip for manual resizing in PowerPoint: hold Alt while dragging to turn off snapping, and Shift to keep proportions.
