# Narci link page

## Files
- index.html: the whole site (styles and scripts inline)
- assets/: logo, [N] mark, favicon, flyers
- netlify/functions/next-event.mjs: feeds the "Next up" section from @dj_narci on Instagram

## To do before launch
1. **Music:** `assets/mix.mp3` is the Afrobeat loop (62 seconds, repeats). To swap in one of Narci's mixes,
   replace that file or change `MIX_SRC` at the top of the script. Keep it on the same domain so the record effects work.
2. **Links:** ONE Link events, SoundCloud, YouTube and WhatsApp (+1 813 297 2534) are live.

3. **Next event:** connect Instagram (below). Until then the Rum in the Sun placeholder shows.

## Instagram: Next up from @dj_narci (Narci never logs in)
Uses Meta's Business Discovery, which lets YOUR Instagram Business account read the public posts
of another Business or Creator account. @dj_narci must be public and set to Business or Creator.

### One-time setup (about 15 minutes)
1. **Your Instagram:** your O.I.M. Instagram must be a Business account, linked to a Facebook Page you manage
   (Facebook Page > Settings > Linked accounts > Instagram).
2. **Meta app:** at developers.facebook.com, create an app (Business type) and add the Instagram product,
   choosing the "API setup with Facebook login" option. You are the admin, so it can stay in Development mode.
3. **User token:** open Tools > Graph API Explorer, pick your app, add these permissions:
   `instagram_basic`, `pages_show_list`, `pages_read_engagement`, `business_management`.
   Click Generate Access Token and approve with YOUR Facebook account.
4. **Make it long-lived:** paste that token into Tools > Access Token Debugger and click "Extend Access Token".
5. **Page token (does not expire):** in Graph API Explorer, with the long-lived token, run `GET me/accounts`.
   Copy your Page's `access_token` and `id`.
6. **Your Instagram ID:** run `GET {page-id}?fields=instagram_business_account`. Copy the `id`.
7. **Netlify:** Site configuration > Environment variables:
   - `FB_TOKEN` = the Page access token from step 5
   - `IG_USER_ID` = the Instagram ID from step 6
8. **Deploy, then test:** open `https://yoursite.com/api/next-event`. You should see JSON with an image link,
   or `{"event":null}` if no post has #event yet.

### Claude reads the flyers (step 7b)
Add a third Netlify environment variable:
- `ANTHROPIC_API_KEY` = an API key from console.anthropic.com (API usage is billed there, separate from a Claude subscription)

What it does:
- Looks at each of @dj_narci's recent photo posts ONCE (results are saved per post with Netlify Blobs) and decides
  whether it's a flyer for an event. Recaps, photos, mixes and Reels are skipped.
- For event flyers it reads the name, date, start/end time, venue and address, and writes a 1-2 sentence description.
- The card shows the soonest event that hasn't happened yet.
- The countdown is only for BIG events (anniversaries, birthdays, festivals, concerts, special guests, holiday parties,
  launches). Claude flags those; the countdown runs to the next big one, even if a smaller event is on the card first
  (then it's labelled with that event's name). No big event coming up: the countdown hides.
- Cost: a fraction of a cent per new post (Claude Haiku). Model can be changed with `CLAUDE_MODEL`.
- Times are read as Eastern; change with `EVENT_TZ` (e.g. `America/Chicago`).

### For Narci
Nothing to do: just post flyers as usual. Tips that help:
- Put the ticket link in the caption (e.g. `Tickets: posh.vip/e/...`). **Get tickets** uses the first link in the caption;
  with no link it goes to onelinkentertainment.com. **Learn more** opens the Instagram post.
- `#event` in a caption forces a post to count as an event (handy if Claude ever misses one).
- Without `ANTHROPIC_API_KEY`, only posts tagged `#event` are used, and the card shows the caption.

### If it stops working
- Check Netlify > Logs > Functions > next-event for the error.
- Token errors: repeat steps 3 to 5 and paste the new Page token into `FB_TOKEN`.
- "Cannot find user": @dj_narci was switched to private or a personal account.
- "Reading post ... failed": check `ANTHROPIC_API_KEY` and that the Console account has credit. Failed reads retry on the next visit.
- If the Instagram function fails for any reason, the site falls back to the placeholder event.

## How the record works
- The mix is decoded into memory once and the record drives its speed directly (smooth on phones).
- Drag it: scratches. Speed and pitch follow your hand; dragging backwards plays the music in reverse.
- Play/pause: the record spins up or down and the music speeds up or slows down with it.
- Hold it still: the record stops and the music dies, like a real deck.
- Tap it: echo throw.
- Scrolling spins it visually. On phones it slides to the center and tucks up at the top (80% of the screen wide, 30% showing; change FINAL_WIDTH and SHOWING in the script) while the page scrolls under it.
- Air horn: the gold button next to Now playing.
- Background: gradient blobs, light leaks, and particle dot waves (the `bands` list in the script sets where they go).

## Countdown clock colors
The Nixie clock glows orange under every tube. For no colored glow at all,
remove `color` from `<div class="nixie color">` in index.html.

## iPhone silent switch
The page sets `navigator.audioSession.type = "playback"` so music and the horn still play when an
iPhone is on silent (iOS 17+). Older iPhones fall back to `assets/silence.mp3`, a silent track that
loops quietly so iOS treats the page as music. Keep that file.

## Performance notes (phones)
- Background blobs are soft gradients (no blur filter) and only move with scrolling on phones.
- Film grain has no blend mode; glows that pulse with the bass are opacity layers.
- No backdrop blur or clock reflection on phones; dot waves draw at lower resolution there.
- Animations always run (the reduce-motion setting is ignored on purpose).
- Phones run the spikes and dot waves at 30fps and use 4 background blobs and 3 light leaks.
- When scrolled, the record's size is capped by screen height (phones held sideways) and at 520px (tablets).
