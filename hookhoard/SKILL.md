---
name: hookhoard
description: Make, fill, label and use mood boards and swipe files in the user's HookHoard (a Chrome extension) through its MCP tools. Use this whenever the user wants to make, start, fill or label a mood board, swipe file, ad board, shoot board or brand board, says "break down this ad" or "why does this ad work", wants a board turned into a creative brief or shot list, or wants to find ads on their boards that use a technique, even if they never say "HookHoard". Do not use it for unrelated image or design tasks such as resizing, editing, generating or converting images.
---

# HookHoard

HookHoard keeps the user's boards in their own Chrome browser. You reach them through the `hookhoard` MCP tools: `list_boards`, `get_board`, `get_item_image`, `create_board`, `add_images`, `set_item_tags`, `add_note`, `add_shot_list`, `search_items`. The MCP prompts `tag-my-swipes`, `shot-list-from-board`, `brief-from-board` and `board-from-brief` carry the label lists and step-by-step flows.

If these tools are not available in this session, say the HookHoard connection is not set up (the HookHoard side panel, "Connect your AI") and stop. Do not pretend.

## ⛔ STOP rules (read before every call)

⛔ **Never invent image URLs or ad facts.** Use only links and pages the user gave you or that you actually found. If you have none, ask the user for links, a page, or local image files. Do not state an advertiser, offer, spend or result unless it is on screen or in the item data.
⛔ **Never say something was saved, added or labelled unless that tool call succeeded.** Read each result. Report every error and skipped image to the user.
⛔ **Ask before adding more than 20 images.** `add_images` takes at most 20 per call; for more, get the user's yes first, then split.
⛔ **Labels use taxonomy ids only.** Read the ids for the board's type from the MCP (the `tag-my-swipes` prompt, or the `set_item_tags` tool description and its error message) every time. Never write ids from memory, never invent one; use null when nothing fits. The lists differ per board type and change between versions, which is why they are not copied here.

## Making a board from a brief

1. Pick the type: `ads` (ads and swipes to learn from), `shoot` (frames to borrow for a shoot), `brand` (identity and style references). Ask only if it truly could be any.
2. `create_board` with a short name and the type. Keep the returned `boardId`.
3. Gather sources the user gave: image links, local image files, or a public web page. `add_images` takes `{url}`, `{path}` or `{pageUrl}`. Pages are fetched from the user's computer without their logins, so pages behind a login will not work; say so and ask for the image links instead. Only real images are accepted (jpg, png, webp, gif, avif; 15 MB each).
4. Read the `add_images` result: `added`, `errors`, `skipped`. Tell the user what was skipped and why. It also returns small thumbnails of the first 8 new items; for more items call `get_item_image` per item.
5. Label every added item with `set_item_tags` (ids from the board type's list, plus a "why it works" sentence).
6. Finish with `add_note` titled "Brief" (see below), then tell the user briefly what is on the board.

The board fills in live if it is open in Chrome. If the user has not opened HookHoard they will find the board there.

## Breaking down an ad

Read the ad in the order a viewer meets it. Look at the image first (`get_item_image`, or the thumbnail from `add_images`), then read the item data from `get_board` (title, source, and for Ad Library items the copy, call to action and days running).

1. **First frame, the 3-second hook.** What would stop a thumb? A face, a bold line, a surprising object, a before and after, motion. Say what it is, not that it is "eye-catching".
2. **Visual and format.** How is it built: a real photo or a designed graphic, a screenshot, a split screen, a testimonial, a demo, a carousel card? Who or what fills the frame, and where does the eye land?
3. **Copy.** Read every word on screen and in the ad copy. What is the headline promising, to whom, and in how many words? Quote the key phrase.
4. **Offer and call to action.** What is being asked of the viewer, and what do they get? If there is no offer on screen, say so rather than guessing one.
5. **Proof.** What makes the claim believable: a number, a real person, a result, a demo, a logo, a review? Name it, or note that none is shown.

Then label it. Read the closed id lists for this board type from the MCP now (the `tag-my-swipes` prompt, or the error from a bad `set_item_tags` call lists them), pick the single best id per field from what you saw in steps 1 to 3, and use null when nothing fits honestly. Choosing from the list is a judgement about the ad, so use the definitions that come with the ids, not just the names.

## Writing "why it works"

One sentence, specific to this ad. Name what is on screen and the move it makes on the viewer. A reader who has not seen the ad should be able to picture it. Draw it from steps 1 to 5 above: usually the hook plus the proof, or the hook plus the offer.

- Good: "A woman holds the half-empty serum bottle up to the camera while the caption reads 'week 6', so the proof is the bottle in her hand, not a claim."
- Good: "The headline is just '12 minutes' in huge type over a clean kitchen counter, so the viewer reads the promise before they see the product."
- Bad: "Strong visual that grabs attention and builds trust." (fits any ad)
- Bad: "Uses social proof to drive conversions." (names a label, shows nothing on screen)

Test: if the sentence would fit any other ad on the board, rewrite it. State only what you can see or read. Do not guess results, audience or spend. If the image is unclear, say what is unclear.

## Turning a board into a brief

`get_board`, then look at the most representative images (up to 8). Write at most 200 to 250 words: what the images share (formats, hooks, visual style), the angle to borrow, the audience and vibe the user gave, and 3 concrete ideas. Build it only from what is on the board and what the user told you. Save it with `add_note` on that board. For a shoot board, `add_shot_list` turns the images into a shoot plan (see the `shot-list-from-board` prompt for the fields).

## Other requests

- "Break down this ad" or "why does this ad work": follow "Breaking down an ad" above, answer in a few lines plus the one-sentence why, and offer to save the labels with `set_item_tags`. Deeper strategic breakdowns are available in HookHoard with credits.
- "Find ads that use X": `search_items`; for long-running ads use its `minDaysRunning` filter. Summarise what the matches share.
- Tagging a board's untagged items: follow the `tag-my-swipes` prompt.

## When HookHoard is not connected

A tool answering that HookHoard is not connected means Chrome is closed, or the HookHoard side panel is not open or not connected. Tell the user exactly that: open Chrome, click the HookHoard icon, and check it says "Connected" (Settings, Connect your AI), then try again. Do not retry in a loop and do not claim anything was done.

## Install (for the user)

- Claude Code: put this folder at `~/.claude/skills/hookhoard`.
- Claude Desktop and claude.ai: Settings > Skills, upload `hookhoard-skill.zip`.
- Codex, Cursor, Antigravity: `npx skills add <source>` (see HookHoard's Connect your AI page for the exact command).
