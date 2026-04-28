# Cal.com Booking Page — Setup Prompt for Claude Code

Paste this whole document into Claude Code when CD'd into the Jehovah Jireh Photography website repo. It will install the cal.com embed, build a `/book` page that matches the existing site design, and wire up the existing "BOOK A SESSION" buttons.

---

## Before You Start

Fill in this value below — Claude Code needs it to plug into the embed:

- **CAL_USERNAME**: `truepathstudios` (booking URL: `cal.com/truepathstudios`)

Event type slugs already created in cal.com (don't change these):
- `consultation` — Free 20-min intro call
- `portrait-session` — 90-min portrait session
- `family-event` — 2-hour family/event session
- `wedding-inquiry` — 30-min wedding consultation call

---

## Prompt for Claude Code

> I want to add a booking page to this site using cal.com's embeddable React component. The cal.com account already exists and four event types are configured. Build it to match the existing site design (dark near-black background, gold `#C9A961` accents, serif display headings, italic gold script accent words — same vibe as the homepage hero).
>
> **My cal.com username is:** `truepathstudios` — use this anywhere you see `CAL_USERNAME` below.
>
> ### 1. Install the dependency
>
> ```bash
> npm install @calcom/embed-react
> ```
>
> ### 2. Create `app/book/page.tsx`
>
> A server component that renders the page chrome (hero, service descriptions, FAQ section) and includes the client-side embed component. Match the site's existing fonts and colors. Reference the homepage for the typography treatment — large serif headline with one italic gold word for visual rhythm.
>
> The page needs:
> - Page metadata (title: "Book a Session — Jehovah Jireh Photography", description for SEO)
> - A hero section with a small gold tracked-out label "— BOOK A SESSION", a serif headline like *"Let's plan **your** session."* (italic gold on the accent word), and a short supporting paragraph
> - The four services rendered below the hero as a list/grid the user can pick from to swap which event type the calendar shows
> - Below the calendar, a short "A few things to know" section — three reassurance bullets about timezone, confirmation email, and "not sure which to pick? Start with the consultation"
>
> ### 3. Create `app/book/BookingEmbed.tsx`
>
> A client component (`"use client"`) that:
> - Renders four service tabs/cards the user picks between (consultation, portrait-session, family-event, wedding-inquiry)
> - Renders the cal.com embed below, swapping which event type loads when the user changes the selection
> - Themes the cal.com embed to match the site (dark theme, gold brand color, near-black background)
>
> Service definitions:
>
> | Slug | Name | Duration | Price label | Short description |
> |---|---|---|---|---|
> | `consultation` | Free Consultation | 20 min | Free | A no-pressure intro call to talk through your vision and see if we're a good fit. |
> | `portrait-session` | Portrait Session | 90 min | Starting at $450 | Individuals and couples. Direction, edited gallery, delivered in two weeks. |
> | `family-event` | Family & Events | 2 hours | Starting at $650 | Family portraits and milestone events. Travel within Florida included. |
> | `wedding-inquiry` | Wedding Inquiry | 30 min | Custom | A consultation call to discuss your day. A custom proposal follows. |
>
> Use this as the embed component skeleton (adjust styling to match the rest of the site):
>
> ```tsx
> "use client";
>
> import { useEffect, useState } from "react";
> import Cal, { getCalApi } from "@calcom/embed-react";
>
> const CAL_USERNAME = "truepathstudios";
>
> const SERVICES = [
>   { slug: "consultation", name: "Free Consultation", duration: "20 min", price: "Free",
>     description: "A no-pressure intro call to talk through your vision and see if we're a good fit." },
>   { slug: "portrait-session", name: "Portrait Session", duration: "90 min", price: "Starting at $450",
>     description: "Individuals and couples. Direction, edited gallery, delivered in two weeks." },
>   { slug: "family-event", name: "Family & Events", duration: "2 hours", price: "Starting at $650",
>     description: "Family portraits and milestone events. Travel within Florida included." },
>   { slug: "wedding-inquiry", name: "Wedding Inquiry", duration: "30 min", price: "Custom",
>     description: "A consultation call to discuss your day. A custom proposal follows." },
> ];
>
> export default function BookingEmbed() {
>   const [selected, setSelected] = useState(SERVICES[0].slug);
>
>   useEffect(() => {
>     (async () => {
>       const cal = await getCalApi({ namespace: selected });
>       cal("ui", {
>         theme: "dark",
>         cssVarsPerTheme: {
>           dark: {
>             "cal-brand": "#C9A961",
>             "cal-bg": "#0A0A0A",
>             "cal-bg-muted": "#141414",
>             "cal-text": "#FFFFFF",
>             "cal-text-emphasis": "#FFFFFF",
>             "cal-border": "rgba(255,255,255,0.1)",
>             "cal-border-subtle": "rgba(255,255,255,0.06)",
>           },
>         },
>         hideEventTypeDetails: false,
>         layout: "month_view",
>       });
>     })();
>   }, [selected]);
>
>   return (
>     <section>
>       {/* Service picker — render SERVICES as tabs/cards. The selected one should
>           highlight with the gold accent. Clicking sets `selected`. */}
>
>       {/* Cal embed — re-mounts when `selected` changes via the `key` prop */}
>       <Cal
>         key={selected}
>         namespace={selected}
>         calLink={`${CAL_USERNAME}/${selected}`}
>         style={{ width: "100%", minHeight: "700px" }}
>         config={{ layout: "month_view", theme: "dark" }}
>       />
>     </section>
>   );
> }
> ```
>
> Style the service picker to match the existing site — bordered cards with the selected one showing the gold accent, similar to how interactive elements are styled elsewhere on the site. Inspect existing components for the pattern.
>
> ### 4. Wire up the existing "BOOK A SESSION" buttons
>
> Find every place these buttons appear (the navbar, the homepage hero — there are at least two from the screenshot I shared earlier) and point them at `/book`. They should be `<Link href="/book">` from `next/link`, not anchor tags. Search the codebase for "BOOK A SESSION" or "book a session" to find them all.
>
> ### 5. Verification steps
>
> Before considering this done:
> - Run the dev server, navigate to `/book`, confirm the page renders without console errors
> - Confirm all four service tabs swap the calendar correctly
> - Confirm the calendar styling matches the site (dark background, gold accents, no jarring white surfaces)
> - Confirm the existing "BOOK A SESSION" buttons in the nav and hero now navigate to `/book`
> - Confirm the page is responsive on mobile (service tabs should stack, calendar should fit)
> - Run the project's lint and typecheck commands and resolve any issues
>
> ### Notes
>
> - The cal.com account is set up under my email for testing. When the client approves the design, I'll create a new cal.com account under her email with the same event type slugs and just swap the `CAL_USERNAME` constant. No other code changes will be needed.
> - Don't add any backend logic — cal.com handles the bookings, confirmation emails, calendar invites, reschedules, and reminders entirely on their end.
> - If the project uses a design system or shared UI components (Button, Card, etc.), use those rather than building from scratch.

---

## After Claude Code finishes

1. `npm run dev` and visit `http://localhost:3000/book`
2. Submit a test booking on yourself for the consultation event type — confirm you get the email and the booking shows up in your cal.com dashboard
3. Push to a preview deploy and send the link to the client for approval
4. When she creates her own cal.com account, swap `CAL_USERNAME` in `app/book/BookingEmbed.tsx` from `truepathstudios` to her username and redeploy
